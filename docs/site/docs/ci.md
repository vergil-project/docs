# Continuous integration

Every repository in the fleet runs the **same CI model**, and it is
**version-agnostic by design**: the set of language versions a repo tests
against is configuration, not workflow code, and the merge gates never name a
version. The versions come from a fleet-wide version window, so a new or
dropped language version reaches every repo without an edit; nothing about the
workflows or the branch protection has to move with it.

This page is the fleet-level summary. The authoritative mechanics — job names,
the reusable-workflow inputs, the evidence bundle format — live in
`vergil-tooling` and are linked below rather than restated here:

- [CI Architecture](https://vergil-project.github.io/vergil-tooling/latest/guides/ci-architecture/)
- [CI Evidence Convention](https://vergil-project.github.io/vergil-tooling/latest/guides/ci-evidence-convention/)

## The version set comes from the version window

A repo's CI version matrix is **not** embedded in `ci.yml` and **not** passed
as a workflow input. It comes from the **version window**, the set of language
versions Vergil supports, through one chain:

1. **The manifest.** `vergil-containers` records each language's window in
   `docker/versions/manifest.yml`. Every version list in that repo is generated
   from it.
2. **The published window.** The generator also writes
   `docker/versions/window.json`, which consumers read at the `v2.1` ref.
3. **The resolver.** `vrg-ci-resolve`, in `vergil-tooling`, reads the window
   and the repo's `vergil.toml` and returns the versions to build. Every CI run
   uses it, through the `vergil-actions` CI setup, and so does local
   validation. The matrix fans out over what it returns — a **dynamic
   matrix**.

What the resolver returns depends on the repo:

- **No pin.** With no `[ci].versions`, a repo follows the window. A `library`
  (`[project].repository-type`) builds every version in the window. Any other
  repository type is an app, and builds one version: the newest.
- **`[ci].versions` is an optional pin.** A pinned version that is not in the
  window is a fatal error, and so is an app pin with more than one version. A
  pin is visible technical debt, not the normal way to choose versions.

The window moves on its own. A nightly **lifecycle job** in `vergil-containers`
compares each language's window with its upstream's release and end-of-life
data, and opens the add, drop and runtime pull requests that change it (see
[Nightly governance](#nightly-governance-opsyml)). A repo that does not pin
picks up the change on its next run, with no edit of its own, and no `ci.yml`
is hand-edited, so the matrix cannot drift from the window.

The window, the pin rules and the lifecycle rules are documented once, in
vergil-containers'
[Language version lifecycle](https://vergil-project.github.io/vergil-containers/latest/languages/lifecycle/)
page.

## Thin-caller `ci.yml`

A consuming repo's `ci.yml` is a **thin caller**. Each job simply `uses:` a
`vergil-actions` reusable workflow at the pinned `@v2.1` tag and passes only
`language:` and `container-suffix:`. It passes **no** version matrix and **no**
container tag — both resolve dynamically: the matrix from the version window
(or from the `[ci].versions` pin, if one is set), and the single-container
jobs' image tag from the primary version (below).

Because the matrix and the container tag are derived rather than written down,
the same short workflow serves every repo, and a version change takes effect
fleet-wide without touching a single `ci.yml`.

One exception to "pass only `language:` and `container-suffix:`": a repo that
publishes packages (see [Release packaging](#release-packaging)) must call the
release workflow with `secrets: inherit`. Its signing secrets are
**environment** secrets, and an explicit `secrets:` map does not deliver
environment secrets to a cross-repo reusable workflow — only `inherit` does.
Semgrep flags `secrets: inherit`, so the line carries an inline suppression:

```yaml
    secrets: inherit  # nosemgrep: yaml.github-actions.security.secrets-inherit.secrets-inherit
```

## Version-agnostic evidence gates

Branch protection requires **stable, version-agnostic aggregate checks**, not
per-version legs. For each matrixed kind — audit, quality, and tests — the
required check is the `<kind> / evidence` aggregate the reusable workflow emits
(`audit / evidence`, `quality / evidence`, `test / evidence`), never the
individual `… / 3.12`, `… / 3.13` legs. Each `evidence` job depends on the
whole matrix, so one required context covers every version.

`quality / evidence` also covers the quality workflow's two single-container
jobs: it aggregates `quality / common` (the common checks), `quality / custom`
(the repo's custom validator), and the matrixed lint and typecheck legs. It
runs even when one of them fails, so any failure fails the gate. It is required
in every repo's CI-gates ruleset, whatever the language; `quality / common` is
no longer required on its own.

The custom validator (`scripts/bin/validate-custom`) therefore runs in PR CI as
well as in local `vrg-validate`, and a failing validator blocks the merge. A
repo with no custom validator gets a passing no-op `quality / custom` job.

Because the required-check *names* carry no version, a **matrix change merges
through the normal gate** — including a matrix *reduction*. This closes the old
deadlock, where branch protection required a per-version leg that a reduced
matrix could never produce, leaving the PR "expected, never reported" and
permanently blocked with no `--admin` escape. A version change is now an
ordinary PR.

The non-matrixed checks (the security scanners, the version-bump gate, the
docs build) keep their fixed, version-free names, and they are required too —
there are no report-only PR gates.

## `[ci].primary-version` for single-container jobs

Some jobs are not matrixed — they run once in a single container (for example
the security scan, the version-bump gate, and the docs build). These run on
the **primary version**: `[ci].primary-version` if it is set, otherwise the
**newest** resolved version (for Python, `3.15` today). `primary-version` is
the escape hatch for the rare case where the newest version is not the right
single-container default.

## Release packaging

A repo whose `vergil.toml` has a `[package]` table also builds signed `.deb` /
`.rpm` packages and publishes them to the org's
[package repository](packages.md). Three `vergil-actions` reusable workflows
carry it; repos without `[package]` call none of the packaging jobs and see no
change.

- **`ci-package.yml` (PR CI).** Builds every package and install-tests it in a
  clean container of each target OS, rolling up into the single
  `package / evidence` gate. It runs one of two tiers: **full** (every build
  cell and every test cell) for release PRs into `main` and non-PR events, and
  **reduced** (every build cell, but only one test cell per format — the oldest
  release, on amd64) for other feature PRs.
- **`cd-release.yml` (release).** `package-matrix` resolves the cells,
  `package-build` builds them unsigned, and `package-sign` — in the main-only
  `package-signing` environment — signs every `.rpm` and attests every artifact.
  The `release` job then attaches the signed packages and
  `packages-manifest.json` to the GitHub Release and dispatches to `packages`.
  Any packaging failure stops the release before it is tagged.
- **`publish-index.yml` (the `packages` repo).** Rebuilds the apt and dnf
  index from the product releases. Before indexing anything it verifies every
  asset's build attestation, **pinned to `cd-release.yml` on
  `refs/heads/main`**, and every `.rpm`'s signature, then signs the metadata in
  the develop-only `index-signing` environment and deploys to GitHub Pages.

The release caller needs `secrets: inherit` (see
[Thin-caller `ci.yml`](#thin-caller-ciyml)). The full mechanics are in
vergil-tooling's
[CI Architecture](https://vergil-project.github.io/vergil-tooling/latest/guides/ci-architecture/)
guide and the
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/latest/reference/package-config/#ci-and-cd).

## Nightly governance (`ops.yml`)

Checking this model across the fleet is itself automated. Every managed repo
carries an `ops.yml` workflow that calls the `vergil-actions` reusable
workflow `ops-github-config.yml` on a nightly schedule. A repo whose `ops.yml`
is missing or has no schedule is treated as non-compliant. The nightly run has
three jobs:

- **Config audit.** `vrg-github-repo-config audit` compares the repo's GitHub
  configuration and its checked-in files against the canonical Vergil
  configuration. It reports drift, such as branch protection that does not
  match the derived required-check set or a required context that no workflow
  can produce, and any drift fails the run. The nightly job only audits; it
  never changes the configuration. Reconciling a repo (`apply`) is a human
  action. See the
  [GitHub Config Audit reference](https://vergil-project.github.io/vergil-tooling/latest/reference/config-audit/).
- **Dependabot audit.** `vrg-dependabot-audit` fails the run when the repo has
  any open Dependabot alert, described below.
- **Branch sweep.** `vrg-branch-sweep` deletes branches whose pull-request
  work is finished. It is the safety net behind the delete-on-merge and
  delete-on-close rules. See the
  [vrg-branch-sweep reference](https://vergil-project.github.io/vergil-tooling/latest/reference/dev/branch-sweep/).

The nightly `ops.yml` in `vergil-containers` also runs the **language
lifecycle job**, which keeps the version window current. For Python and Go it
opens one pull request per change: an add, a drop, or (for Python) a runtime
change that records a newly published `vergil-python` runtime. For the other
languages it opens an issue instead. It never enables auto-merge: a person
reviews and merges every lifecycle pull request.

### Report-only Dependabot

Dependabot alerts are on for every repo, and Dependabot never opens a pull
request. The config audit enforces both settings: alerts on, Dependabot
security updates off. Every repo also carries a canonical
`.github/dependabot.yml` in which every entry sets
`open-pull-requests-limit: 0`. `vrg-validate` rejects an entry without that
limit, and the config audit flags a missing or non-canonical file. The file
exists so that the OpenSSF Scorecard Dependency-Update-Tool check passes.

The nightly `dependabot-audit` job fails on any open alert, at any severity.
`vrg-dependabot-audit` exits 1 when the repo has open alerts and 2 when it
could not check them, and both fail the run. The log of a public repo shows
alert counts only. The only way to clear an alert is to fix the dependency or
dismiss the alert on GitHub; nothing in the repo can suppress one.

This gate is defense in depth, aimed at dormant repos. The dependency update
at release time and the language audits that run on every change remain the
main dependency hygiene. See the
[Dependabot Alert Audit reference](https://vergil-project.github.io/vergil-tooling/latest/reference/dependabot-audit/).

## Python toolchain selection

Vergil owns which Python interpreter runs. A repo's own files may declare only
a floor, such as `requires-python`:

- **`.python-version` is an error.** It chooses the interpreter version behind
  Vergil's back, so validation fails until it is deleted.
- **uv's `python-preference` and `python-downloads` are allowed.** They choose
  where an interpreter comes from, not which version it is.
- **Vergil sets the interpreter itself.** Wherever it runs Python, it sets
  `UV_PYTHON` to the resolved version and `UV_PYTHON_PREFERENCE=only-system`,
  so uv always uses the image's interpreter.

See rule 13 of the
[Language version lifecycle](https://vergil-project.github.io/vergil-containers/latest/languages/lifecycle/#the-rules)
and vergil-tooling's
[Container Config Reference](https://vergil-project.github.io/vergil-tooling/latest/reference/container-config/).
