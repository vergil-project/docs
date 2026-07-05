# Tools & skills

The Management layer is operated through two surfaces:

- **`vrg-*` command-line tools** — the mechanical operations (create an epic,
  link a task, run the audit), maintained in
  [`vergil-tooling`](https://github.com/vergil-project/vergil-tooling).
- **Skills** — the agent-facing workflows that *compose* those tools into a
  process (orchestrate an epic, capture intake, groom the queue), maintained in
  the Claude Code plugin,
  [`vergil-claude-plugin`](https://github.com/vergil-project/vergil-claude-plugin).

This page explains how they fit together and **who drives what**. The dry,
per-command reference — every flag of every tool — lives in the repositories
where the code does; this is the story, not the man page.

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

An agent never merges and never opens a PR itself: it prepares the change and
hands off, and a human takes it across the line.

## The tools, by role

### Authoring & structure

- `vrg-epic-create` (mint an epic in `.github`)
- `vrg-adhoc-epic` (ensure a repo's ad-hoc epic exists)
- `vrg-issue-create` (create a task, born linked under an epic)
- `vrg-epic-link` / `vrg-epic-move` / `vrg-epic-unlink` (manage the epic↔task relationship)

### Intake

- `vrg-triage-create` (file an intake issue into `.github`; `--kind triage|idea|research`)

### Lifecycle

- `vrg-epic-rollup` (close a finite epic when its last task closes — event-driven)
- `vrg-finalize-pr` (merge + post-merge cleanup — human)

### Reporting & enforcement

- `vrg-roadmap` (the open-epic roadmap)
- `vrg-epic-audit` (drift and invariant violations — read-only, with a human-gated `--close`)
- `vrg-adhoc-migrate` (relocate legacy standing epics — dry-run by default, human-gated `--apply`)

### Pull-request flow

- `vrg-commit` (standards-compliant commits)
- `vrg-pr-workflow report-ready` (records the PR hand-off)
- `vrg-submit-pr` (open the PR — human-run)

The `vrg-gh` and `vrg-git` wrappers sit under all of this, enforcing the
allow-lists and selecting the right identity; raw `git`/`gh` are blocked in agent
sessions.

## The skills, by role

- **`epic-create`** — the outer orchestrator of the [workflow](workflow.md):
  brainstorm → epic → spec → pushback → plan → alignment → docs PR → file tasks.
- **`triage-capture`** — the lossless net: file an [intake](intake.md) issue with
  near-zero friction.
- **`triage-review`** — the periodic grooming pass that drains the intake queue
  into the model.
- **`migrate-repo`** — onboard a repository's pre-framework backlog into the
  epic/task model.
- **`issue-implement`** — the agent implements a single issue end-to-end and
  hands the PR off.
- **`pr-watch`** — the agent monitors an open PR through CI and review,
  reconciling until it is mergeable.

Together, the tools give the mechanical guarantees and the skills give the
process on top of them — which is exactly why the *story* belongs here, in one
place, while each tool's and action's mechanical detail stays in its own repo.
