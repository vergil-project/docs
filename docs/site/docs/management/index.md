# Management

Vergil's **Management** layer is how work is organized and moved — from a raw
idea to a merged, closed change — across every repository and org in the fleet.
It is one of several Vergil frameworks (alongside Security, Testing, and the
VM/identity and container layers); this one owns the *work itself*: what we are
doing, why, in what order, and how it gets done.

It is deliberately **two-directional**. Looking *down*, it lets you manage and
zoom into the smallest unit — a single task, one pull request. Looking *up*, it
lets you zoom out to the strategic view — every cross-cutting initiative in
flight, across the whole organization, in one place. The same model serves both,
and the point is to keep climbing: once you can manage tasks, you manage epics;
once you can manage epics, you manage how epics interrelate.

## Units of work and delivery

Two definitions do the load-bearing work here:

- **The unit of *work* is the task** — an issue that represents a single task,
  implemented by a single pull request and closed the moment that PR merges.
  This one-issue-to-one-PR discipline is exactly what makes the trailing end of
  the process — closing and roll-up — automatable.
- **The unit of *delivery* is the epic** — a collection of related tasks that
  together implement a larger, more complex body of functionality. You *deliver*
  epics; the tasks are the to-dos that implement them.

Historically the unit was the *issue*: have an idea, file an issue, go do it.
That broke down — a single issue was often complex enough to need many pull
requests, sometimes across repositories. Splitting work from delivery fixes it:
the task is small and mechanical, the epic is the meaningful whole.

If this looks like Jira with a layer missing, that is deliberate. Jira would
slot a *feature* between epic and task; we skip it — "feature" is an overloaded
label — and let the epic be the first-layer container of tasks. `.github` is our
one special repository; the `epic` is our one special issue type: the container
for everything else.

## Two invariants

Everything in this layer follows from two rules:

1. **All epics live in the organization's `.github` repository** — one place to
   see every initiative in the org.
2. **Every other repository holds nothing but single-PR tasks** — each closed by
   exactly one pull request in that same repository.

The load-bearing principle behind the second rule is **task↔PR locality**: a task
lives in the repository whose pull request closes it. Cohesion between a task and
the change that completes it matters more than a pristine issue list. In practice
the `.github` list reads as "epics only" — you filter by the `epic` label — and
the few exceptions (the intake queue, and the rare task whose PR edits `.github`
itself) are self-justifying.

## The shape of the model

- **Epics & tasks** — finite epics (in `.github`), plus one perpetual **ad-hoc**
  epic per repository for the small, unplanned work that should never accrete
  into a larger initiative.
- **Intake** — three lightweight queues (`triage`, `idea`, `research`) that catch
  raw input the moment it occurs, so nothing is lost before it is planned.
- **Workflow** — how an idea becomes an epic becomes shipped tasks: the
  `epic-create` orchestrator and a front-loaded, interactive design pipeline that
  makes implementation near-bulletproof.
- **Tools & skills** — how the `vrg-*` commands and the skills fit together: what
  each does at the story level, and which are driven by a human versus an agent.
  The per-command reference lives in the repositories where the code does.

One convention is worth stating up front, because it shapes the whole lifecycle:
an epic **opens** with a documentation task and does not **close** until you have
reviewed it, decided what comes next, and confirmed the docs reflect what
changed. Nothing is ever "done" without asking what comes next.

## Why this lives in the `docs` repo

This documentation is **integrated**. The Management layer spans the tooling
(`vergil-tooling`) and the CI actions (`vergil-actions`), and the model itself is
not specific to any one repository — so it lives here, in the org-wide `docs`
site, the first place to go to understand how the whole thing fits together.
This site tells the **story** — how the pieces interact, who drives what.
Individual repositories hold only the dry, *mechanically unique* reference: the
per-command "man page" for each CLI, what a specific action does. Part of
building this section is therefore walking the other repositories and trimming
their documentation back to reference-only, so the org-wide explanation lives in
exactly one place — here.
