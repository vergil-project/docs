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

`ad-hoc` is the **single** marker for this perpetual variant. An earlier
`standing` alias has been retired — removed from the code and the label
registry — so `ad-hoc` is the only form the model recognises.

An ad-hoc epic is **perpetual**: created once, never auto-closed, kept open even
when empty. That perpetual-ness is the *only* thing distinguishing it from a
finite epic. The name does deliberate work — "ad hoc" is the smell a serious
engineer drives toward zero. The long-term goal is for ad-hoc work to become the
rare exception as real work is increasingly shaped into finite, tidy,
self-closing epics.

As its children close they drain into per-quarter archive buckets, so the live
ad-hoc epic shows only work still in flight.

## Standing epics

The ad-hoc epic is one of four **standing epics** — perpetual umbrellas that
are created once, reused, and never roll up:

| Standing epic | Holds |
| --- | --- |
| `Epic (ad hoc): <repo>` | ready one-off work for one repository |
| `Epic (intake): <name>` | captured, unreviewed items — and every orphan the sweep finds |
| `Epic (planning): <name>` | items waiting for (or undergoing) epic creation |
| `Epic (deferred): <name>` | evaluated items parked until their revival conditions are met |

There is one ad-hoc epic per repository. The intake, planning and deferred
epics come as a set per **queue home**: the org's `.github` for every public
repository, and the repository itself for a private one. They are the three
queues that hold work before it is ready to build — see
[Intake, planning & deferred](intake.md).

## Every issue belongs to an epic

In any managed repository, **every issue is either a legitimate top-level kind
or has an epic parent.** The top-level kinds are epics (including the standing
ones), the per-quarter archive buckets, and the release tracking issue a
release opens for its duration. Everything else — tasks, operational tasks,
intake, planning and deferred items — hangs under an epic.

There is no opt-out and no "unlinked" state, and the parent is not just
bookkeeping: it *is* the issue's state. Under the intake epic it has been
captured; under the planning epic it awaits a design; under the deferred epic it
waits for its revival conditions; under an ad-hoc or finite epic it is ready to
build. The sanctioned tools create every issue already
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

- **Epics live in `.github`.** A public repository's epics belong in the org's
  `.github`; an epic that has leaked out is flagged. (A private repository may
  legitimately home its own epics.)
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
- **No private work under public epics.** Visibility changes are supported in
  one direction only. **Private → public** is supported: the sweep moves the
  repository's queue items under the org's `.github` queues, and its epics are
  relocated. **Public → private** is **unsupported** — create a new private
  repository and migrate the code instead. A private repository's issue found
  under a public `.github` epic is reported for a human to resolve.

The daily sweep (`vrg-epic-audit --close`) does more than report: its **orphan
pass** auto-heals the model. It converts body-only parent links into native
ones, strips the retired labels, moves a now-public repository's self-homed
queue items under `.github`, and attaches every orphan to its home's intake
epic. It never closes an item, and it stops loudly — never silently skipping —
when a queue is too full to accept more.

After the orphan pass, the sweep runs a **security-records pass** over the
private draft advisories in the org's `.github`. Each advisory follows the
neutral `security` issues it tracks — closed once every one of them is closed,
a draft while any is open — and broken pairs are reported for a human to
resolve, never repaired. See vergil-tooling's
[Security Records](https://vergil-project.github.io/vergil-tooling/latest/guides/security-records/#managing-a-record-through-its-issue)
guide.

When a **finite epic** is found outside its home, relocate it: move the issue
into `.github` with `vrg-gh issue transfer`, then re-establish its task links
with `vrg-epic-link`.

The exact rules and commands are in vergil-tooling's
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#compliance-invariants)
and its
[Epic home visibility flips](https://vergil-project.github.io/vergil-tooling/latest/guides/epic-home-visibility-flip/)
guide.
