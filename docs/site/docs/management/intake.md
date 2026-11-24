# Intake, planning & deferred

The formal path — brainstorm, spec, epic, tasks — is the preferred on-ramp, but
most work does not arrive that way. It arrives as a dog-walk epiphany, a "we
should probably…", an "oh, don't forget." **Intake** is the lossless net: it
turns raw input into a tracked issue with near-zero friction, so nothing is lost
before it can be planned.

Capturing the seed of an idea *when it is fresh* is more valuable than planning
it out on the spot. Intake optimizes for that.

## Every issue belongs to an epic

The net is not an exception to the model — it is part of it. In every managed
repository, **every issue is either a legitimate top-level kind (an epic, a
per-quarter archive bucket, or a release tracking issue) or has an epic
parent.** There is no opt-out and no "unlinked" state, and that includes a
half-formed idea captured thirty seconds ago.

What makes that workable is that **the parent is the state**. An issue's parent
epic says where it stands, so no label has to restate it:

| State | Parent | Buildable? |
| --- | --- | --- |
| **Intake** — captured, not yet reviewed | the intake epic | no |
| **Planning** — judged to need a full epic | the planning epic | no |
| **Deferred** — evaluated, but waiting on something before it can move | the deferred epic | no |
| **Ready** — a one-off, or part of a planned initiative | a repository's ad-hoc epic, a finite epic, or a proxy epic | yes |

Work moves **intake → planning → ready**. An item that has been evaluated but
cannot move yet — it waits on an upstream release, another issue, a date, or
capacity — steps aside into the **deferred** queue until its revival conditions
are met. Only ready work can be built: the pull-request tooling refuses an issue
that still sits in the intake, planning or deferred queue, and says which step
to take first.

## The standing queues

The three queues are **standing epics** — perpetual, like a repository's ad-hoc
epic: created once, reused, never auto-closed.
`Epic (intake): <name>` holds captured, unreviewed items;
`Epic (planning): <name>` holds items waiting for (or undergoing) epic
creation; `Epic (deferred): <name>` holds evaluated items parked until their
revival conditions are met.

There is one set per **queue home**, the org's `.github`. Every repository
shares it, so the whole org-wide queue is one view rather than something
scattered across repositories. Closed items drain into
per-quarter archive buckets, so the live queues show only open work.

## The lifecycle

1. **Capture.** The moment something is worth remembering, it becomes an intake
   item — filed already linked under the intake epic. A one-line "don't forget
   X" is valid, and there is no category to choose: a bug, a spark, and an
   investigation all enter the same way. No planning, no scoping; that is
   deliberately deferred.
2. **Review.** On a periodic pass, intake is reviewed **oldest first**, in small
   batches. For each item the agent checks its current state and recommends a
   route; the human confirms the batch. There are five routes:
   - **Drop** — close it with a reason (not planned). It stays searchable.
   - **Defer** — move it into the deferred queue, recording why it waits and
     what to check before it can come back.
   - **One-off** — move it into its target repository as a ready task under
     that repository's ad-hoc epic (or a named epic). The ad-hoc epic takes
     only standalone work: an item that is work of a development epic in
     another org goes under that epic's
     [proxy](epics-and-tasks.md#proxy-epics) in the target repository's org,
     never ad-hoc, and one in the primary's own org goes under the primary
     itself. A one-off never goes under the hoshin epic: only
     `/vergil:hoshin` files its tasks.
   - **Split** — replace it with several ready tasks and close it.
   - **Planning** — it needs a real design, so it moves to the planning queue,
     with an estimated epic size.
3. **Plan.** A planning review keeps the planning queue correct, oldest
   first: each item is left (most are), deferred, dropped, or, unusually,
   sent out as a one-off, to a repository's ad-hoc epic or, for another
   org's epic, to its proxy, by the same rule as from intake. It never picks. The human
   chooses what to plan next and runs `epic-create` on it, picking only what
   will be implemented soon, since a plan left sitting goes stale. A
   planning item is the record of its own epic creation: `epic-create` always
   works through one. Its body carries the run's stage checklist, written by
   `vrg-checkpoint`, and its comments record each stage's progress, so a
   session can pause and resume days later at the first unticked stage (see
   [the pipeline](workflow.md#the-pipeline)). A planning item is *waiting*, *in progress*, or
   *completed* — closed as "planned as" the epic it became, or with the reason
   it is not worth doing.

   A planning item can carry an **estimated epic size**: **S** (1–9 tasks),
   **M** (10–19), **L** (20–39) or **XL** (40+), derived from an estimated
   task count. Sizing is optional: an item stays unsized (`?` in `vrg-queue`)
   until it is sized. A size is recorded as an `epic-size:<S|M|L|XL>` label
   and a dated `Sized …` comment, both written by `vrg-intake-route`: an item
   is sized as it is routed to planning, and the planning review sizes any
   waiting item still unsized. Deferred items can carry a size too, and a
   one-off clears it. The size is there so the planning queue can be sorted
   and filtered by it: `vrg-queue planning --sort size --size S` lists the
   small items, oldest first, so small epics need not wait behind large
   ones. An item whose sizing is in error (`!`) is always listed too,
   whatever the filter.
4. **Revisit.** A deferred item is never forgotten, because every deferral
   records a reason and the conditions to check (`Revisit when:`). On a
   periodic deferred review, the agent actually checks each item's conditions
   against their real sources — a release page, an issue's state, a date — and
   recommends one outcome: **keep** it (not yet), **refresh** its record,
   **revive** it (back to intake, to planning, or as a one-off), or **drop**
   it. Only an item in a queue or an ad-hoc epic can be deferred; a task in a
   finite epic cannot, since parking one task would in effect suspend its
   whole epic.

A `security` item — the neutral issue half of a security record — is routed
like any other and keeps its advisory links through every move: every advisory
it names is retargeted when the item is transferred or recreated, and its one
advisory is retargeted when it is split. A split of an item naming several
advisories is refused, since which advisory goes to which task can't be
inferred; retarget each by hand, one at a time, with
`vrg-security-record retarget <GHSA> --from <item> --to <task> [--to …]`,
then split. The `--to` set is the complete set the advisory tracks afterwards:
a `--to` for each task that fixes it and for every other open issue it still
tracks, since `retarget` refuses a set that leaves one out and prints the
command that would succeed. See vergil-tooling's
[Security Records](https://vergil-project.github.io/vergil-tooling/latest/guides/security-records/#retargeting-by-hand)
guide.

The intake and planning queues are meant to stay drained: a growing one is a
signal to review it, not a place for work to live. The deferred queue is not
drained but re-checked — an item stays there exactly as long as its revival
conditions are unmet.

## Nothing escapes

Issues created outside the sanctioned tools — in the web UI, by a script —
are **healed, not rejected**. A daily sweep attaches every orphan it finds to
its home's intake epic, so it surfaces at the next review instead of drifting
unseen. The one exception is an issue an outsider opened: the sweep never
attaches it, and reports it by metadata only (see
[Trusted content](epics-and-tasks.md#trusted-content)). Placement stays the reviewer's job; the sweep only guarantees that
nothing is lost.

The former `triage` / `idea` / `research` kind labels are **retired** — the
parent carries the state now, and the sweep strips the old labels.

## Mechanics

The commands and their exact behavior — `vrg-intake-create`,
`vrg-intake-route`, `vrg-queue`, the deferral record, the orphan pass, and the
queue-home rules — are documented in vergil-tooling's
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#intake-planning-and-deferred-queues)
(see
[The deferred queue](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#the-deferred-queue)
for deferring and reviving, and
[Viewing the queues](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#viewing-the-queues-vrg-queue)
for epic sizes) and the
[CLI reference](https://vergil-project.github.io/vergil-tooling/latest/reference/cli-tools-overview/#vrg-intake-route).
