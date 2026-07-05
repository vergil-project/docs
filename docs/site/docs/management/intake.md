# Intake

The formal path — brainstorm, spec, epic, tasks — is the preferred on-ramp, but
most work does not arrive that way. It arrives as a dog-walk epiphany, a "we
should probably…", an "oh, don't forget." **Intake** is the lossless net: it
turns raw input into a tracked issue with near-zero friction, so nothing is lost
before it can be planned.

Capturing the seed of an idea *when it is fresh* is more valuable than planning
it out on the spot. Intake optimizes for that.

## Three queues

Intake has three shapes, each a label, and — like everything strategic — all of
them live in the org's `.github`, so the whole org-wide queue is one filtered
view rather than something scattered across repositories:

| Kind | What it captures | Graduates into |
| --- | --- | --- |
| `triage` | A problem or bug not yet understood — needs diagnosis | an epic, or a task |
| `idea` | A spark — "what if we did this?" | a feature/epic |
| `research` | An investigation meant to produce a **reproducible** result | an epic with tooling + a report |

The three are genuinely different intake shapes, not severity levels. In
particular, **research is not ad-hoc work**: a result worth having is a result
worth reproducing, so a research thread spawns automated tooling and pull
requests and becomes a proper finite epic — never something done by hand and
thrown away.

## The lifecycle

1. **Capture.** The moment something is worth remembering, it becomes an intake
   issue in `.github` with the right `--kind` — a single, fast, correctly
   labelled record. No planning, no scoping; that is deliberately deferred.
2. **Groom.** On a periodic pass, each intake item is reviewed and dispositioned:
   promoted into a finite epic, filed as a task under an existing epic, dropped
   onto a repository's ad-hoc epic, or closed as obsolete.
3. **Promote.** An item that graduates leaves the queue and enters the formal
   model; the queue is meant to stay drained.

Intake is intentionally exempt from the every-task-needs-an-epic rule *until it
is groomed* — that exemption is the whole point of a lossless net. Capture
first; impose structure later.
