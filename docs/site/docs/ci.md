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

## Nightly governance (`ops.yml`)

Keeping this model canonical across the fleet is itself automated. Every
managed repo carries an `ops.yml` workflow with a **nightly config-audit
caller** (a scheduled `vergil-actions` reusable workflow). It keeps each
repo's GitHub configuration canonical — reconciling branch protection against
the derived required-check set, and flagging drift such as a required context
that no workflow can produce. A repo whose `ops.yml` is missing or has no
schedule is treated as non-compliant.
