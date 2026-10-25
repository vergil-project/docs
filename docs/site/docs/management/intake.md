# Intake & planning

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
| **Ready** — a one-off, or part of a planned initiative | a repository's ad-hoc epic, or a finite epic | yes |

Work moves **intake → planning → ready**, and only ready work can be built: the
pull-request tooling refuses an issue that still sits in the intake or planning
queue, and says which step to take first.

## The standing queues

The intake and planning queues are **standing epics** — perpetual, like a
repository's ad-hoc epic: created once, reused, never auto-closed.
`Epic (intake): <name>` holds captured, unreviewed items;
`Epic (planning): <name>` holds items waiting for (or undergoing) epic
creation.

There is one pair per **queue home**. Every public repository shares the org's
pair in `.github`, so the whole org-wide queue is one view rather than something
scattered across repositories. A **private** repository keeps its own pair, so
private work never surfaces in a public issue. Closed items drain into
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
   - **Defer** — leave it, with a dated note; it keeps its age and comes up
     first next time.
   - **One-off** — move it into its target repository as a ready task under
     that repository's ad-hoc epic (or a named epic).
   - **Split** — replace it with several ready tasks and close it.
   - **Planning** — it needs a real design, so it moves to the planning queue.
3. **Plan.** A planning item is the record of its own epic creation:
   `epic-create` always works through one, and leaves progress comments so a
   session can pause and resume days later. A planning item is *waiting*,
   *in progress*, or *completed* — closed as "planned as" the epic it became,
   or with the reason it is not worth doing.

The queues are meant to stay drained: a growing intake queue is a signal to
review it, not a place for work to live.

## Nothing escapes

Issues created outside the sanctioned tools — in the web UI, by a script —
are **healed, not rejected**. A daily sweep attaches every orphan it finds to
its home's intake epic, so it surfaces at the next review instead of drifting
unseen. Placement stays the reviewer's job; the sweep only guarantees that
nothing is lost.

The former `triage` / `idea` / `research` kind labels are **retired** — the
parent carries the state now, and the sweep strips the old labels.

## Visibility changes go one way

Because a private repository keeps its own queues and epics, a repository's
visibility decides where its work is tracked. **Private → public is supported**:
the sweep moves the repository's open queue items under the org's `.github`
queues, and its epics are relocated to match. **Public → private is not
supported** — work already tracked in public cannot be made private again, so
the path is to create a new private repository and migrate the code into it.
The audit reports private-repository issues found under public epics for a
human to resolve.

## Mechanics

The commands and their exact behavior — `vrg-intake-create`,
`vrg-intake-route`, `vrg-queue`, the orphan pass, and the queue-home rules —
are documented in vergil-tooling's
[GitHub Issue Standards](https://vergil-project.github.io/vergil-tooling/latest/standards/github-issues/#intake-and-planning-queues)
and
[Epic home visibility flips](https://vergil-project.github.io/vergil-tooling/latest/guides/epic-home-visibility-flip/)
guide.
