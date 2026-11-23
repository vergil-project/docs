# Epics & tasks

The model has exactly two artifacts — the **task** (the unit of work) and the
**epic** (the unit of delivery) — plus a family of perpetual **standing epics**
for the work that is not (or not yet) part of a plan.

## Tasks

A **task** is an issue that represents a single, self-contained change:

- It is implemented by **exactly one pull request**, in the repository where
  that PR lands (**task↔PR locality**).
- It is **born linked** to an epic — as is every issue: see
  [Every issue belongs to an epic](#every-issue-belongs-to-an-epic). On GitHub the link is a native
  *sub-issue*; on forges without sub-issues it degrades to a portable
  `Parent: <owner>/<repo>#<n>` line in the task body. Either way, the tooling
  speaks one vocabulary and never the underlying mechanism.
- It **closes when its PR merges**. A task with an epic parent links its PR with
  the `Closes` keyword, so the merge closes the task automatically; the one close
  keyword is `Closes` (`Fixes`/`Resolves` are banned to keep it unambiguous).

Because a task is exactly one PR, "done" is unambiguous: once the change is in
the trunk, the task is finished. Any later change is a *new* task, never a
reopening.

Several small tasks of one epic in one repository may share a **batch PR**:
up to 8 of them (4 by default) on one branch, each still its own commit and
its own `Closes` line. The batch PR is rebase-merged, not squashed, so each
task keeps its own commit and changelog entry on the trunk and still closes
when the PR merges. A single-task PR is squash-merged as before. The
[worktree convention](https://github.com/vergil-project/vergil-tooling/blob/develop/docs/specs/worktree-convention.md#batches)
names a batch's worktree and branch.

## Finite epics

A **finite epic** is a cross-cutting initiative with an end — a real project made
of related tasks, often spanning several repositories. It carries the `epic`
label and lives in the org's `.github`. Its spec and plan live beside it, at
`epics/<n>-<slug>/` in `.github`.

When **every** child task of a finite epic is closed, the epic **rolls up** — it
auto-closes. Roll-up is event-driven: an `on: issues.closed` Action fires the
check on every issue close, so an epic wraps itself up the moment its last task
lands, with no human bookkeeping.

## Ad-hoc epics

Not all work belongs to an initiative. The small, unplanned, standalone
things — fix a typo in a man page, bump a pin — would clutter the model if each
demanded its own epic. So every repository has exactly one **ad-hoc epic**:
`Epic (ad hoc): <repo>`, labelled `epic` + `ad-hoc`, and (like all epics) located
in `.github`. Its tasks live in the target repository, linked across.

An ad-hoc epic holds **standalone work only**: each of its tasks can be picked
up, merged and released on its own. A task owned by another epic or by a skill
never goes there. A task of another org's epic goes under that epic's
[proxy](#proxy-epics), and the `Logbook:` tasks that `/vergil:hoshin` files go
under the [hoshin epic](#standing-epics). No filing-time guard enforces this:
`vrg-epic-audit` reports an open ad-hoc task that looks misplaced (a
`Logbook:` title, or a body naming an epic in another org) for a human to rule
on, and a trusted `Standalone-confirmed` comment on a task ruled standalone
stops the report.

`ad-hoc` is the **single** marker for this perpetual variant. An earlier
`standing` alias has been retired — removed from the code and the label
registry — so `ad-hoc` is the only form the model recognises.

An ad-hoc epic is **perpetual**: created once, never auto-closed, kept open even
when empty, and it holds only work that stands alone. The name does deliberate
work — "ad hoc" is the smell a serious
engineer drives toward zero. The long-term goal is for ad-hoc work to become the
rare exception as real work is increasingly shaped into finite, tidy,
self-closing epics.

As its children close they drain into per-quarter archive buckets, so the live
ad-hoc epic shows only work still in flight.

Ready ad-hoc tasks are worked with `epic-implement` in its `adhoc` mode, which
collects them across every ad-hoc epic with `vrg-queue adhoc` and proposes one
strategy for the human to confirm; see
[Working the ad-hoc tasks](workflow.md#working-the-ad-hoc-tasks).

## Standing epics

The ad-hoc epic is one of five **standing epics** — perpetual umbrellas that
are created once, reused, and never roll up:

| Standing epic | Holds |
| --- | --- |
| `Epic (ad hoc): <repo>` | ready one-off work for one repository |
| `Epic (intake): <name>` | captured, unreviewed items — and every orphan the sweep finds |
| `Epic (planning): <name>` | items waiting for (or undergoing) epic creation |
| `Epic (deferred): <name>` | evaluated items parked until their revival conditions are met |
| `Epic (hoshin): <org>` | the `Logbook:` tasks that `/vergil:hoshin` files and works |

There is one ad-hoc epic per repository. The intake, planning and deferred
epics come as a set per **queue home**: the org's `.github`, shared by every
repository in the org. They are the three queues that hold work before it is
ready to build — see [Intake, planning & deferred](intake.md).

There is one **hoshin epic** per org, labelled `epic` + `hoshin`. It is always
homed in `<org>/.github`. It is not a queue: its tasks are ready to build, but
only `/vergil:hoshin` (the [hoshin kanri](hoshin-kanri.md) review cadence) files
and works them, so no queue review or ad-hoc run picks them up, and no intake
route moves an item into it. Its closed children drain into per-quarter
archives like the other standing epics'.

## Proxy epics

A development epic's work sometimes lands in a repository of another org. A
**proxy epic** is the same-org umbrella for those tasks: titled
`Epic (proxy): <owner>/<repo>#<N> — <primary's title without "Epic: ">` (the
primary's full ref, then its title), labelled `epic` + `proxy`, and homed in
that org's `.github`. The development epic it serves is its **primary**. The
link runs both ways: the proxy's body carries one `Primary: <owner>/<repo>#<N>`
line, and the primary records each proxy in a trusted comment carrying a
`Proxy: <owner>/.github#<N>` line.
`vrg-epic-create --proxy-for <primary ref> --repo <owner>/<repo>` finds,
reopens or creates the proxy, with its `Proxy:` comment on the primary.

A proxy is **not a development epic**. It has no spec, plan or bookends, since
the primary carries them; the roll-up never closes it; and the roadmap does not
list it. Its tasks are driven from the primary: `epic-implement` on the primary
works them beside the primary's own, and refuses the proxy itself. The proxy's
own org closes it: the daily sweep closes a proxy once its primary is closed
and it has no open task. `vrg-epic-audit --epic` reports a `Primary:` /
`Proxy:` link where the two sides disagree.

## Epic categories

Every epic carries exactly one **category label**, so one label filter in the
GitHub UI shows each category:

| Category | Epics | Category label |
| --- | --- | --- |
| Development | every [finite epic](#finite-epics): planned with `epic-create`, closed by roll-up | `epic-development` |
| Management | the intake, planning, deferred and hoshin [standing epics](#standing-epics) | `epic-management` |
| Ad hoc | each repository's [ad-hoc epic](#ad-hoc-epics) | `ad-hoc` |
| Proxy | each [proxy epic](#proxy-epics) | `proxy` |

`ad-hoc` and `proxy` are both the marker the tools read and the category
label. `epic-development` and `epic-management` are new, and they are **for
display only**: no tool reads either of them to decide behaviour or to find an
epic. An archive bucket is not an epic and has no category.

The tools apply the category label when they create an epic.
[`vrg-epic-create`](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#vrg-epic-create)
derives it from the epic's other labels, and refuses, creating nothing, a
caller-given `epic-development` or `epic-management` label and labels that
mark two categories at once (for example `ad-hoc` and `proxy`). The path that
creates a standing epic on demand adds `epic-management` to a new intake,
planning, deferred or hoshin epic.

Epics created before the category labels existed are labelled once, by the
one-off deployment
[vergil-project/.github#1460](https://github.com/vergil-project/.github/issues/1460).
[`vrg-ensure-label --repo <owner>/<repo> --label <name>`](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#vrg-ensure-label)
creates one registered label from the label registry.

## Every issue belongs to an epic

In any managed repository, **every issue is either a legitimate top-level kind
or has an epic parent.** The top-level kinds are epics (including the standing
ones), the per-quarter archive buckets, and the release tracking issue a
release opens for its duration. Everything else — tasks, operational tasks,
intake, planning and deferred items — hangs under an epic.

There is no opt-out and no "unlinked" state, and the parent is not just
bookkeeping: it *is* the issue's state. Under the intake epic it has been
captured; under the planning epic it awaits a design; under the deferred epic it
waits for its revival conditions; under an ad-hoc, finite or proxy epic it is
ready to build. The sanctioned tools create every issue already
linked, and an issue born outside them is **healed, not rejected** — the daily
sweep attaches it to its home's intake epic.

## The bookend convention

An epic is never truly "done" the instant its implementation lands, because
almost no real problem is 100% closed by a single initiative. So every epic
carries **bookend** tasks:

- **It opens** with a **documentation task** — the spec and plan, born from
  planning.
- **It closes** with two tasks, in order: a **documentation review** (confirm
  the change is reflected in the docs — especially this site) and then the
  **retrospective**, the terminal gate — a single backward-looking record that
  partners the spec and plan and records what comes next. Its docs PR closes
  the epic.
- A **follow-on brainstorm** is seeded up front only when a known follow-on
  already exists; otherwise its outcome is captured in the retrospective.

This rides the roll-up: because the epic cannot close until *all* its tasks
close, the closing bookends *gate* completion. You cannot finish an epic without
answering "now what?" and confirming the story is documented. Nothing is ever
done without asking what comes next.

## Compliance invariants

These guarantees are enforced, not just documented. `vrg-epic-audit` is a
read-only sweep (with a human-gated `--close`) that flags drift and invariant
violations:

- **Epics live in `.github`.** Every repository's epics belong in the org's
  `.github`; an epic that has leaked out is flagged.
- **No stray `.github` issues.** The epic home holds only epics, the
  per-quarter archive buckets, and issues linked under an epic — tasks and
  intake, planning and deferred queue items alike. Any other open issue there is a stray.
- **An epic is never closed while a child is open.** Roll-up closes a finite
  epic only once *all* its children are closed; a closed epic with a still-open
  child is a violation, remediated by reopening it. (Standing epics never roll
  up; their closed children drain into archives instead.)
- **Every issue has an epic parent.** An **orphan** — an open issue that is
  neither a top-level kind nor linked under an epic — is reported, along with
  any issue still carrying the retired `triage` / `idea` / `research` labels.

The daily sweep (`vrg-epic-audit --close`) does more than report: its **orphan
pass** auto-heals the model. It converts body-only parent links into native
ones, strips the retired labels, and attaches every orphan to its home's intake
epic. It never closes an item, and it stops loudly — never silently skipping —
when a queue is too full to accept more.

The sweep does not run the **security-records pass**: `vrg-epic-audit --close`
makes no security-advisory call. The pass is `vrg-security-record sync`, which
the human or the user agent runs over the private draft advisories in the org's
`.github`; nothing runs it on a schedule, and the read-only `vrg-epic-audit`
only previews it. Each advisory follows the neutral `security` issues it
tracks: closed once every one of them is closed as completed and none is a
queue item, a draft while any is open. The **close guard** keeps an advisory
`draft`, reported on a `held:` line, while any issue it tracks is a queue item
(live or archived) or was closed with a reason other than completed; retarget
it to the task that fixes it, or close it with `vrg-security-record close` and
a reason. Broken pairs are reported for a human to resolve, never repaired. See vergil-tooling's
[Security Records](https://vergil-project.github.io/vergil-tooling/latest/guides/security-records/#managing-a-record-through-its-issue)
guide.

When a **finite epic** is found outside its home, relocate it: move the issue
into `.github` with `vrg-gh issue transfer`, then re-establish its task links
with `vrg-epic-link`.

The exact rules and commands are in vergil-tooling's
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#compliance-invariants).
