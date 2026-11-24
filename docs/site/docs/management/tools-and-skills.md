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
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/).

## Who drives what

The dividing line is accountability. **The speed limit is human comprehension,
not code generation** — so agents do the development, and humans own the
irreversible, outward-facing steps.

| Driven by the **agent** (runs as the `user` identity) | Driven by the **human** |
| --- | --- |
| Participating in the brainstorm; running the design-review rounds | The judgment calls in the brainstorm, and the decision on each design-review finding and each further round |
| Authoring specs and plans; implementing tasks | Submitting the pull request (`vrg-submit-pr`), which turns GitHub auto-merge on |
| Commits and PR preparation / hand-off | Reviewing the change before submitting it: the submit is the decision to **merge** |
| Read-only audits and dry-runs | Settling `develop` once the PRs merge (`vrg-finalize-pr`, which the submit always runs) |
| Filing intake, epics, tasks, links | Write-gated ops: migration `--apply`, audit `--close` |
| Implementing a dependency-update fix task | Releases (`vrg-release`) and dependency updates (`vrg-update-deps`) |
| Requesting a review decision, drafting the human's reply and reading it (`vrg-ruling`) | Replying to a review-decision request, as themselves (`vrg-ruling-reply`) |

An agent never merges and never opens a PR itself: it prepares the change and
hands off, and a human takes it across the line.

## The tools, by role

### Authoring & structure

- `vrg-epic-create` (mint an epic in `.github`; with
  `--proxy-for <primary ref> --repo <owner>/<repo>` it instead finds, reopens
  or creates the [proxy epic](epics-and-tasks.md#proxy-epics) for a primary in
  another org, in the target's org `.github`, with its `Proxy:` comment on the
  primary)
- `vrg-adhoc-epic` (ensure a repo's ad-hoc epic exists)
- `vrg-issue-create` (create a task, born linked under an epic)
- `vrg-epic-link` / `vrg-epic-move` (manage the epic↔task relationship)

- `vrg-issue-create --epic adhoc|intake|planning|hoshin` (target a
  repository's standing epic by name rather than by number; `hoshin` is the
  org's hoshin epic in `<org>/.github`, for `/vergil:hoshin`'s logbook tasks)

### Intake, planning & deferred

- `vrg-intake-create` (capture an intake item, linked under its home's intake
  epic as it is created — no category to choose)
- `vrg-intake-route` (apply a review's route: drop, defer, one-off, split, or
  planning out of intake; defer or drop out of planning; and out of deferred,
  revive to intake, planning or one-off, refresh, drop, or split. Defer
  requires `--reason` and `--revisit-when`, and is the only way into or out of
  the deferred queue. `size --tasks N --reason TEXT` records a planning or
  deferred item's estimated epic size (an `epic-size:<S|M|L|XL>` label and a
  dated `Sized …` comment), `planning --tasks N --reason TEXT` sizes an item
  as it enters planning, and a one-off clears the size. A `security` item
  keeps its advisory links when it is transferred, recreated, or split; a
  split of an item naming several advisories is refused)
- `vrg-queue intake|planning|deferred` (the read-only view of one queue — intake
  oldest first, deferred least recently deferred first — or of all three with a
  bare `vrg-queue`; a planning item is held at in-progress, never shown as
  ready-to-close, while any security advisory still tracks it, or while it is
  `security`-labelled and names an advisory that cannot be read, until each
  advisory is retargeted to the task or tasks that fix it. Planning and
  deferred lines show each item's epic size (`M/14`, `?` unsized, `!` in
  error), and `vrg-queue planning` also takes
  `--sort age|size|tasks` and `--size LIST` to order and filter by it. An item
  whose sizing is in error makes the run exit 2, after the full listing
  (exit 1 is a queue that could not be read or a usage error). See the
  [`vrg-intake-route`](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#vrg-intake-route) and
  [`vrg-queue`](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#vrg-queue) reference)
- `vrg-queue adhoc` (the read-only view of the ready one-off work: the open
  tasks under every live ad-hoc epic, grouped by repository and oldest first
  within each, each with its state — `ready`, `blocked` by an open
  `Blocked-by:` dependency, or `in-flight` with the marks that show it, such as
  an open PR, a report-ready record, a feature branch, a worktree or a
  merge-review directory. `--repo` narrows it to one repository's ad-hoc epic.
  It is not a review queue, so it is shown only when named, never by a bare
  `vrg-queue`)
- `vrg-queue hoshin` (the same read-only view of the open tasks under the
  org's hoshin epic; `--repo` narrows it to one repository's tasks. Like
  `adhoc`, it is shown only when named)
- `vrg-security-record` (record a security finding as a private draft advisory
  in `<org>/.github`, paired with a neutral `security`-labelled intake issue;
  `create`, `list`, `show`, `retarget` (move an advisory's tracking to the
  tasks that fix it; when they are all in another org, the advisory moves to
  that org's `.github`), `close` (close a draft advisory with a
  reason posted on every issue it tracks), `note` (append a dated note to an
  advisory) and `sync` (the **security-records pass**: sync each advisory's
  state to its issues and report broken pairs, run by the human or the user
  agent, never on a schedule) — see vergil-tooling's
  [Security Records](https://vergil-project.github.io/vergil-tooling/latest/guides/security-records/)
  guide)

An issue still in the intake, planning or deferred queue is not buildable:
`vrg-pr-workflow report-ready` and `vrg-submit-pr` refuse it until it is routed
out.

### Lifecycle

- `vrg-epic-rollup` (close a finite epic when its last task closes — event-driven)
- `vrg-finalize-pr` (watch the submitted PRs merge, then settle `develop` once:
  sync, cleanup, validate and a CD report; it never merges — human, run by
  every `vrg-submit-pr`)

### Reporting & enforcement

- `vrg-roadmap` (the open-epic roadmap)
- `vrg-epic-audit` (drift and invariant violations — read-only, with a gated
  `--close`; under `--close`, run daily by the scheduled sweep, it also runs the
  **orphan pass** that attaches every unparented issue to its home's intake
  epic, except one an outsider opened, which it reports. With
  `--untrusted --since <time>` it is instead the read-only **outsider
  report**: what accounts outside the
  [trusted roles](epics-and-tasks.md#trusted-content) did in each Vergil org
  since then, by metadata and links only, with possible tampering first.
  `--close` makes no security-advisory call: the security-records pass is
  `vrg-security-record sync`, which the read-only `vrg-epic-audit` only
  previews)

To relocate a **finite** epic into its correct home (`.github`), move the issue
with `vrg-gh issue transfer` and re-link its tasks with `vrg-epic-link`.

### Pull-request flow

- `vrg-commit` (standards-compliant commits)
- `vrg-pr-workflow report-ready` (records the PR hand-off)
- `vrg-submit-pr` (open the PRs against `develop` with auto-merge on — squash
  for one issue, rebase for a batch — then always finalize them; human-run)

### Merge review

The `merge-review` skill's tools, run by the agent between a green
`vrg-validate` and `report-ready`:

- `vrg-merge-review-request` (build a review or check request in the task's
  private directory; it refuses unless the last `vrg-validate` was a full,
  clean, green run at `HEAD`)
- `vrg-merge-review-check` (check a stored reviewer result against its
  request; the skill acts only on a result that passes)

The request builder decides the change class itself, `docs-only` or `code`;
a `docs-only` change is reviewed only for merge quality.
`vrg-change-class` is not part of the loop: it prints the class a branch
will get, for humans and debugging.

A repository lists its documentation paths for the change class in the
`[merge-review] docs-paths` key of `vergil.toml`, read as committed at the
branch's merge base. The loop, the records and these tools are in
vergil-tooling's
[Merge Review](https://vergil-project.github.io/vergil-tooling/latest/guides/merge-review/)
guide.

### Review decisions

When a review of a task's change needs the human's decision, the decision
travels as an authenticated comment on the task issue:

- `vrg-ruling` (the agent's side, run as the `user` identity: `request` posts
  a review-decision request, `draft` relays the reply it drafts for the human,
  `read` reads the human's reply with turn checks, and `respond` posts the
  response)
- `vrg-ruling-reply` (post the human's reply to a request, as the human —
  human-run in the human's own shell, like `vrg-submit-pr`)

The `merge-review` skill sends every stop for the human through them, and
`epic-implement` drafts the human's replies to those stops. The protocol, its
checks and recovery are in vergil-tooling's
[Human Replies](https://vergil-project.github.io/vergil-tooling/latest/guides/human-replies/)
guide.

### Release

- `vrg-release` (cut a release from `develop` to `main` — human-run;
  `--resume` finishes the open release, and `--supersede X.Y.Z` replaces
  an open release that `--resume` cannot finish)
- `vrg-update-deps` (the mechanized dependency update — human-run)
- `vrg-release-record check` (read an issue's `Released:` record and
  confirm it against git: how a deployment task confirms that a
  blocked-by task is released)

A release is one workflow, tracked on its `release: <version>` issue. It
runs its stages in order: tag and publish, `update-deps`, the `install`
stage (`[publish].install`), a
[`Released:` record](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/#released-records)
on every issue the release closes, and last the closing of the tracking
issue. A failed stage does not undo what is done: the tracking issue
stays open, and the release is always finished with
[`vrg-release --resume`](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/#resuming-a-release),
which runs only the stages not yet done. A release that cannot finish is
[superseded](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/#superseding-a-release)
by a new version, which absorbs its issues.

A repository has
[one release at a time](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/#one-release-at-a-time):
a fresh `vrg-release` refuses while any release tracking issue is open.
The tracking issue is the human's: it is locked as it is created, and
its state is trusted only while every identity that can change it is a
human. Any other identity stops the release, which can then only be
superseded
([the lock and the identity rule](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/#the-lock-and-the-identity-rule)).

The `update-deps` stage runs `vrg-update-deps` once the release is tagged
and published. When anything changed, it opens a
`deps/update-<next>` pull request labelled `dependency-update`, and a green one
is merged. If validation or the merge fails, the PR stays open and a fix task is
filed. If an updater itself fails, nothing is pushed and the failure is recorded
on the release's tracking issue. Either failure leaves the tracking issue open
for `--resume`. Any open `dependency-update` PR blocks
the next release, until its fix task closes as completed — the next
`vrg-release` then closes the PR and the gate clears. Two flags change this:
`--allow-pending-deps` lets an urgent release through despite an open PR, and
the bypass is logged on the release's tracking issue; `--without-deps` skips the
`update-deps` stage. The release procedure itself is in vergil-tooling's
[Releasing guide](https://vergil-project.github.io/vergil-tooling/latest/guides/releasing/).

The `vrg-gh` and `vrg-git` wrappers sit under all of this, enforcing the
allow-lists and selecting the right identity; raw `git`/`gh` are blocked in agent
sessions. The hook guard also refuses an agent's raw issue and PR content
reads and names `vrg-issue-read`, the trusted read command, which prints only
what a [trusted](epics-and-tasks.md#trusted-content) identity wrote and
reports the rest as metadata.

## The skills, by role

- **`epic-create`** — the outer orchestrator of the [workflow](workflow.md):
  epic and documentation task → brainstorm → spec → spec pushback rounds →
  human spec review → plan → plan pushback rounds → alignment rounds → file
  tasks → docs PR, always worked through a planning item that records its
  progress. The run is nine checkpointed stages, tracked by `vrg-checkpoint`
  in the planning item's body, so a fresh session resumes it after `/clear`
  at the first unticked stage. Its
  design-review rounds need paad's unattended runner, `paad:paad-delegate`,
  which every box, Lima or cloud, gets by installing paad from the vergil
  fork through the ordinary plugin refresh. A session without it is on a
  misconfigured box and runs no review round: `epic-create` stops and tells
  the human how to fix the box
  ([paad usage policy §10](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/paad-policy.md#10-paad-from-the-fork);
  [paad review rounds: what a session needs](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#paad-review-rounds-what-a-session-needs)).
- **`intake-capture`** — the lossless net: file an [intake](intake.md) item with
  near-zero friction.
- **`epic-review`** — the periodic pass over one queue, in small batches,
  with a mode per queue:
  - **`epic-review intake`** works the intake queue oldest first and routes
    each item out of it: drop, defer, one-off, split, or planning, or leaves
    it for now. An item routed to planning is always sized as it moves.
  - **`epic-review planning`** keeps the planning queue correct, oldest
    first: each item is left (most are), deferred, dropped, or, unusually,
    sent to a repo's ad-hoc epic as a one-off. It prunes and never picks:
    the human chooses what to plan next and runs `epic-create` on it. It
    recommends a size for each waiting item that is unsized or in error, and
    ends by listing the surviving queue, by size when the human asks.
  - **`epic-review deferred`** checks each deferred item's `Revisit when:`
    conditions against their real sources and recommends keep, refresh,
    revive, or drop; an unsized item revived to planning is sized as it
    moves.
- **`epic-implement`** — the driver above the `issue-*` skills, run as often
  as needed between `epic-create` and `epic-retrospective`. On the development
  path it resumes an epic from its issue and plan, dispatches every runnable
  task to `issue-implement`, `issue-validate` or `issue-deploy` (in parallel
  sub-agents where that helps, batching small independent code tasks), brings
  everything that needs the human to one gate, and stops. Its **`adhoc`** mode
  works the ad-hoc tasks instead: org-wide with `/vergil:epic-implement adhoc`,
  or for one repository with `adhoc --repo <owner>/<repo>` or that
  repository's ad-hoc epic ref. On a development epic it also works the tasks
  of every [proxy epic](epics-and-tasks.md#proxy-epics) the epic has in
  another org. It refuses an epic it does not drive and says what does:
  given an intake, planning or deferred queue it points to `epic-review`,
  which is how a queue is worked; given the hoshin epic, to `/vergil:hoshin`;
  and given a proxy epic, to its primary
  ([epic-implement](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#epic-implement)).
- **`issue-implement`** — the agent implements a single issue end-to-end,
  validates it to green, runs `merge-review` and hands the PR off with the
  merge review's eligibility line in its notes, in a Lima or a cloud session
  alike
  ([issue-implement](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#issue-implement)).
- **`merge-review`** — the pre-PR judgement check `issue-implement` runs
  once validation is green: one review by the read-only `merge-reviewer`
  agent, at most one fix and check, every stop for the human through
  `vrg-ruling`, every raised item closed before hand-off, and an eligibility
  line for the PR notes
  ([merge-review](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#merge-review)).
- **`fixture-run`** — files a fixture run on one or more of the language
  fixture repos: per fixture, a change, release and validation trio, blocked
  by the release or deployment task passed as `--after`. It files the tasks
  and never runs them. An epic whose plan changes language support,
  container images, shared CI workflows or release tooling seeds one run per
  affected fixture
  ([fixture-run](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#fixture-run)).
- **`hoshin`** — runs the [hoshin kanri](hoshin-kanri.md) review cadence
  with the human and writes its public [logbook](../logbook/index.md):
  `/vergil:hoshin start` each working day works out which reviews are due,
  closes any period left open (late, or recorded as not performed) and runs
  the daily review and any boundary reviews; `end` or `end <level>` is
  optional. The daily review also shows two trust checks, in the terminal
  only: the outsider report (`vrg-epic-audit --untrusted`) and the result of
  the nightly trusted-authors check. Each run files the follow-ups the human decides on and hands off
  one docs PR of dated entries, after a security screen and the human's
  approval of the full text. Its `Logbook:` task lives under the org's
  hoshin epic, and it skips `merge-review` on purpose, so its handoff
  carries no eligibility line
  ([hoshin](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/site/docs/skills/index.md#hoshin)).
- **`pr-watch`** — the agent monitors an open PR through CI and review,
  reconciling until it is mergeable. It fixes a logbook PR (one titled
  `Logbook:`) like any other, but skips the merge review of its fixes, as
  the logbook run itself does: the text passed the run's security screen
  and the human's approval before the PR existed, and a fix to it is
  mechanical, never new content.

Together, the tools give the mechanical guarantees and the skills give the
process on top of them — which is exactly why the *story* belongs here, in one
place, while each tool's and action's mechanical detail stays in its own repo.
