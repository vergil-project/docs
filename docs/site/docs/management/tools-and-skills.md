# Tools & skills

The Management layer is operated through two surfaces:

- **`vrg-*` command-line tools** — the mechanical operations (create an epic,
  link a task, run the audit), maintained in
  [`vergil-tooling`](https://github.com/vergil-project/vergil-tooling).
- **Skills** — the agent-facing workflows that *compose* those tools into a
  process (orchestrate an epic, capture intake, review the queue), maintained in
  the Claude Code plugin,
  [`vergil-claude-plugin`](https://github.com/vergil-project/vergil-claude-plugin).

This page explains how they fit together and **who drives what**. The dry,
per-command reference — every flag of every tool — lives in the repositories
where the code does; this is the story, not the man page. For the issue model's
mechanics, see vergil-tooling's
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/standards/github-issues/).

## Who drives what

The dividing line is accountability. **The speed limit is human comprehension,
not code generation** — so agents do the development, and humans own the
irreversible, outward-facing steps.

| Driven by the **agent** (runs as the `user` identity) | Driven by the **human** |
| --- | --- |
| Participating in brainstorm / pushback / alignment | The judgment calls *inside* those interactive stages |
| Authoring specs and plans; implementing tasks | Submitting the pull request (`vrg-submit-pr`) |
| Commits and PR preparation / hand-off | Reviewing and **merging** |
| Read-only audits and dry-runs | Post-merge finalize (`vrg-finalize-pr`) |
| Filing intake, epics, tasks, links | Write-gated ops: migration `--apply`, audit `--close` |
| Implementing a dependency-update fix task | Releases (`vrg-release`) and dependency updates (`vrg-update-deps`) |

An agent never merges and never opens a PR itself: it prepares the change and
hands off, and a human takes it across the line.

## The tools, by role

### Authoring & structure

- `vrg-epic-create` (mint an epic in `.github`)
- `vrg-adhoc-epic` (ensure a repo's ad-hoc epic exists)
- `vrg-issue-create` (create a task, born linked under an epic)
- `vrg-epic-link` / `vrg-epic-move` / `vrg-epic-unlink` (manage the epic↔task relationship)

- `vrg-issue-create --epic adhoc|intake|planning` (target a repository's
  standing epic by name rather than by number)

### Intake & planning

- `vrg-intake-create` (capture an intake item, linked under its home's intake
  epic as it is created — no category to choose)
- `vrg-intake-route` (apply an intake-review route: drop, defer, one-off, split,
  or planning)
- `vrg-queue intake|planning` (the read-only view of either queue, oldest
  first)

An issue still in the intake or planning queue is not buildable:
`vrg-pr-workflow report-ready` and `vrg-submit-pr` refuse it until it is routed
out.

### Lifecycle

- `vrg-epic-rollup` (close a finite epic when its last task closes — event-driven)
- `vrg-finalize-pr` (merge + post-merge cleanup — human)

### Reporting & enforcement

- `vrg-roadmap` (the open-epic roadmap)
- `vrg-epic-audit` (drift and invariant violations — read-only, with a gated
  `--close`; under `--close`, run daily by the scheduled sweep, it also runs the
  **orphan pass** that attaches every unparented issue to its home's intake epic)
- `vrg-adhoc-migrate` (the one-time relocation of the legacy `standing` epics —
  **complete**; the `standing` alias is retired, so `ad-hoc` is the only
  perpetual variant)

To relocate a **finite** epic into its correct home (`.github`), move the issue
with `vrg-gh issue transfer` and re-link its tasks with `vrg-epic-link`.

### Pull-request flow

- `vrg-commit` (standards-compliant commits)
- `vrg-pr-workflow report-ready` (records the PR hand-off)
- `vrg-submit-pr` (open the PR — human-run)

### Release

- `vrg-release` (cut a release from `develop` to `main` — human-run)
- `vrg-update-deps` (the mechanized dependency update — human-run)

A release ends with an `update-deps` stage, which runs `vrg-update-deps` once
the release is tagged and published. When anything changed, it opens a
`deps/update-<next>` pull request labelled `dependency-update`, and a green one
is merged. If validation or the merge fails, the PR stays open and a fix task is
filed. If an updater itself fails, nothing is pushed and the failure is recorded
on the release's tracking issue. Any open `dependency-update` PR blocks
the next release, until its fix task closes as completed — the next
`vrg-release` then closes the PR and the gate clears. Two flags change this:
`--allow-pending-deps` lets an urgent release through despite an open PR, and
the bypass is logged on the release's tracking issue; `--without-deps` skips the
`update-deps` stage. The release procedure itself is in vergil-tooling's
[Releasing guide](https://vergil-project.github.io/vergil-tooling/guides/releasing/).

The `vrg-gh` and `vrg-git` wrappers sit under all of this, enforcing the
allow-lists and selecting the right identity; raw `git`/`gh` are blocked in agent
sessions.

## The skills, by role

- **`epic-create`** — the outer orchestrator of the [workflow](workflow.md):
  brainstorm → epic → spec → pushback → plan → alignment → docs PR → file tasks,
  always worked through a planning item that records its progress.
- **`intake-capture`** — the lossless net: file an [intake](intake.md) item with
  near-zero friction.
- **`intake-review`** — the periodic pass that reviews the intake queue oldest
  first, in small batches, and routes each item out of it.
- **`migrate-repo`** — onboard a repository's pre-framework backlog into the
  epic/task model.
- **`issue-implement`** — the agent implements a single issue end-to-end and
  hands the PR off.
- **`pr-watch`** — the agent monitors an open PR through CI and review,
  reconciling until it is mergeable.

Together, the tools give the mechanical guarantees and the skills give the
process on top of them — which is exactly why the *story* belongs here, in one
place, while each tool's and action's mechanical detail stays in its own repo.
