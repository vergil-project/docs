# Workflow

Getting an idea into shipped, closed tasks is a pipeline — and the whole bet is
that **front-loading the analysis** makes the implementation near-bulletproof.
Hours spent up front on design and planning pay for themselves when the pull
requests just work the first time.

## `epic-create` is the entry point

For non-trivial work, the starting move is **`epic-create`**, not brainstorming
directly. A solution worth thinking through is worth recording as work, so there
is little value in designing something and then walking away from it untracked.
`epic-create` opens *into* brainstorming and orchestrates the whole sequence. If
the design collapses to a trivial, single-PR change, it is a **task**, not an
epic — filed under the target repository's ad-hoc epic, and done.

Every run is tracked in a **planning item** — an issue in the
[planning queue](intake.md) that records its own epic creation, with progress
comments so a session can pause and resume days later:

- **`epic-create <issue>`** starts or resumes that planning item (an intake or
  deferred item is first moved to the planning queue).
- **`epic-create` with a free-text idea** first turns the idea into a new
  planning item.
- **`epic-create`** with no argument summarizes the planning queue and asks
  which item to work on.

Which item to plan next is decided in **`planning-review`**: the human ranks the
planning queue by priority and importance and picks only what will be
implemented soon, since a plan left sitting goes stale.

The planning item closes once the epic exists, its docs task has merged, and
its tasks are filed — "planned as" the epic it became.

## The pipeline

`epic-create` drives these stages in order:

1. **Brainstorm** — explore intent and converge on an approved design.
2. **Initialize** — create the epic in `.github` and seed its bookend tasks
   (documentation first; documentation review and the retrospective last).
3. **Spec → spec pushback rounds → human review** — write the spec, critique
   it adversarially in spec pushback rounds, and have a human review it.
4. **Plan → plan pushback rounds → alignment rounds** — write the
   implementation plan in one automated pass, critique it in plan pushback
   rounds, then reconcile the spec and the plan in alignment rounds, which may
   change both.
5. **Docs PR** — land the spec and plan as a single documentation PR (which
   closes the documentation bookend).
6. **File tasks** — create the implementation tasks from the plan, each in the
   repository where its PR will land.

The review steps in stages 3 and 4 run in **human-gated rounds**, described
below.

## The interaction doctrine

The design and review steps are not hand-waving — each runs a concrete,
reusable **skill**,
composed from two packages: **Superpowers** (`brainstorming`, `writing-plans`)
and **PAAD** (`pushback`, `alignment`). Brainstorming produces the spec,
pushback critiques the spec, writing-plans generates the plan, pushback
critiques the plan, and alignment confirms the spec and the plan match. The
pipeline mixes interactive, human-gated and automated stages on purpose:

| Stage | Mode | Contract |
| --- | --- | --- |
| `superpowers:brainstorming` | **interactive** | Explore intent, one question at a time — produces the spec. |
| Design review (pushback on the spec, pushback on the plan, alignment) | **human-gated rounds** | A fresh review-only subagent per round. The main session walks the human through the findings, and the human decides every further round. |
| `superpowers:writing-plans` | **automated** | Generate the implementation plan in one pass; no gating. |

Each design-review round reviews the whole document, and the main session
records it as a comment on the planning item. The human decides every finding
and whether to run another round of the stage. The rules are in the
[paad review doctrine, mode 1](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/paad-review-doctrine.md#2-mode-1-human-gated-design-review).
How the workflow uses PAAD, and every place it deviates from the PAAD authors'
recommendations, is recorded in the
[paad usage policy](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/paad-policy.md).

The **human-judgment principle** governs brainstorming: stop and ask *only* for
ambiguities or judgment calls that **materially affect** the outcome; handle
minor, obvious corrections by **batching them to the end** as a single
"here are the no-brainers — correct me if I'm wrong" review, rather than gating
each one.

**Design review** walks the findings one at a time, in severity order, except
for one batch at the end: low findings, each with its own decision, and known
findings, which keep their earlier decision unless the human pulls one out
([doctrine §2.2](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/paad-review-doctrine.md#22-one-round),
step 4).

The goal of all this front-loading is blunt: plans that just work — on the order
of 90–99% of tasks green on first implementation. Real-world misses still happen
and get patched, but the more upfront refinement, the higher the success rate,
and the more mechanical work can eventually be forked off and trusted to run on
its own.

Each task is then implemented with the `issue-implement` skill, which runs an
**agentic-review gate** before it hands the PR off. The gate is advisory: when
it pauses, the human can accept the branch as it stands. The rules are in the
[paad review doctrine](https://github.com/vergil-project/vergil-claude-plugin/blob/develop/docs/paad-review-doctrine.md#3-mode-2-agentic-review-in-issue-implement).
