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

## The pipeline

`epic-create` drives these stages in order:

1. **Brainstorm** — explore intent and converge on an approved design.
2. **Initialize** — create the epic in `.github` and seed its bookend tasks
   (documentation first; follow-on-brainstorm and documentation-review last).
3. **Spec → pushback → review** — write the spec, adversarially critique it,
   and have a human review it.
4. **Plan → alignment** — write the implementation plan, then reconcile it
   against the spec.
5. **Docs PR** — land the spec and plan as a single documentation PR (which
   closes the documentation bookend).
6. **File tasks** — create the implementation tasks from the plan, each in the
   repository where its PR will land.

## The four-stage interaction doctrine

The four stages are not hand-waving — each is a concrete, reusable **skill**,
composed from two packages: **Superpowers** (`brainstorming`, `writing-plans`)
and **PAAD** (`pushback`, `alignment`). They run in a fixed order — brainstorming
produces the spec, pushback critiques the spec, writing-plans generates the plan,
and alignment critiques the plan and confirms it matches the spec — and the
pipeline mixes interactive and automated stages on purpose:

| Skill | Mode | Contract |
| --- | --- | --- |
| `superpowers:brainstorming` | **interactive** | Explore intent, one question at a time — produces the spec. |
| `paad:pushback` | **interactive** | Adversarially critique the spec; the human makes the judgment calls. |
| `superpowers:writing-plans` | **automated** | Generate the implementation plan; no gating. |
| `paad:alignment` | **interactive** | Critique the plan and confirm it matches the spec. |

The **human-judgment principle** governs every interactive stage: stop and ask
*only* for ambiguities or judgment calls that **materially affect** the outcome;
handle minor, obvious corrections by **batching them to the end** as a single
"here are the no-brainers — correct me if I'm wrong" review, rather than gating
each one.

The goal of all this front-loading is blunt: plans that just work — on the order
of 90–99% of tasks green on first implementation. Real-world misses still happen
and get patched, but the more upfront refinement, the higher the success rate,
and the more mechanical work can eventually be forked off and trusted to run on
its own.
