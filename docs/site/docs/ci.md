# Continuous integration

Every repository in the fleet runs the **same CI model**, and it is
**version-agnostic by design**: the set of language versions a repo tests
against is configuration, not workflow code, and the merge gates never name a
version. Changing what a repo tests against is a one-line edit; nothing about
the workflows or the branch protection has to move with it.

This page is the fleet-level summary. The authoritative mechanics — job names,
the reusable-workflow inputs, the evidence bundle format — live in
`vergil-tooling` and are linked below rather than restated here:

- [CI Architecture](https://vergil-project.github.io/vergil-tooling/guides/ci-architecture/)
- [CI Evidence Convention](https://vergil-project.github.io/vergil-tooling/guides/ci-evidence-convention/)

## The version set lives only in `vergil.toml`

A repo's CI version matrix is stored in exactly one place: `[ci].versions` in
its `vergil.toml`. It is **not** embedded in `ci.yml` and **not** passed as a
workflow input. The `vergil-actions` reusable workflows read `[ci].versions`
from the consuming repo at run time and fan their matrix out over that list —
a **dynamic matrix**.

The consequence is that changing what a repo tests against — adding a new
language version, or dropping an end-of-life one — is a **one-line
`vergil.toml` edit**. No `ci.yml` is hand-edited, so the matrix cannot drift
from the stored version set.

## Thin-caller `ci.yml`

A consuming repo's `ci.yml` is a **thin caller**. Each job simply `uses:` a
`vergil-actions` reusable workflow at the pinned `@v2.1` tag and passes only
`language:` and `container-suffix:`. It passes **no** version matrix and **no**
container tag — both resolve dynamically: the matrix from `[ci].versions`, and
the single-container jobs' image tag from the primary version (below).

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
per-version legs. For each matrixed kind — audit, quality (lint + typecheck),
and tests — the required check is the `<kind> / evidence` aggregate the
reusable workflow emits (`audit / evidence`, `quality / evidence`,
`test / evidence`), never the individual `… / 3.12`, `… / 3.13` legs. Each
`evidence` job depends on the whole matrix, so one required context covers
every version.

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
**highest** entry of `[ci].versions` (so `3.14` for
`["3.12", "3.13", "3.14"]`). `primary-version` is the escape hatch for the rare
case where the highest version is not the right single-container default.

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
[CI Architecture](https://vergil-project.github.io/vergil-tooling/guides/ci-architecture/)
guide and the
[Package Config Reference](https://vergil-project.github.io/vergil-tooling/reference/package-config/#ci-and-cd).

## Nightly governance (`ops.yml`)

Keeping this model canonical across the fleet is itself automated. Every
managed repo carries an `ops.yml` workflow with a **nightly config-audit
caller** (a scheduled `vergil-actions` reusable workflow). It keeps each
repo's GitHub configuration canonical — reconciling branch protection against
the derived required-check set, and flagging drift such as a required context
that no workflow can produce. A repo whose `ops.yml` is missing or has no
schedule is treated as non-compliant.
