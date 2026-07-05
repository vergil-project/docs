# Epics & tasks

The model has exactly two artifacts — the **task** (the unit of work) and the
**epic** (the unit of delivery) — plus one special-purpose variant of the epic
for the work that refuses to fit a plan.

## Tasks

A **task** is an issue that represents a single, self-contained change:

- It is implemented by **exactly one pull request**, in the repository where
  that PR lands (**task↔PR locality**).
- It is **born linked** to an epic. On GitHub the link is a native
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

An ad-hoc epic is **perpetual**: created once, never auto-closed, kept open even
when empty. That perpetual-ness is the *only* thing distinguishing it from a
finite epic. The name does deliberate work — "ad hoc" is the smell a serious
engineer drives toward zero. The long-term goal is for ad-hoc work to become the
rare exception as real work is increasingly shaped into finite, tidy,
self-closing epics.

## The bookend convention

An epic is never truly "done" the instant its implementation lands, because
almost no real problem is 100% closed by a single initiative. So every epic
carries **bookend** tasks:

- **It opens** with a **documentation task** — the spec and plan, born from
  planning.
- **It closes** with two kinds of task: a **follow-on brainstorm** (review what
  shipped, decide what comes next, and spawn the follow-on epic if there is one)
  and a **documentation review** (confirm the change is reflected in the docs —
  especially this site).

This rides the roll-up: because the epic cannot close until *all* its tasks
close, the closing bookends *gate* completion. You cannot finish an epic without
answering "now what?" and confirming the story is documented. Nothing is ever
done without asking what comes next.
