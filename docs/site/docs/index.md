# Vergil

**Vergil** is a fleet-management and repository-governance system: a Python
toolchain (`vergil-tooling`), a Claude Code behavioral plugin, reusable CI/CD
actions, and isolated container/VM environments — all coordinated by a two-tier
**epic / task** planning model.

This site is the project's home: what Vergil is, **where it's going**, and what's
shipped recently.

## Where the project is going

The **[Roadmap](roadmap.md)** is generated from the open epics across every repo
in the organization — one view of every cross-cutting initiative, refreshed
automatically.

## What's shipping

The **[Activity log](activity.md)** is the by-date record of recently closed work
across the whole org.

## Supported languages

Vergil validates repositories across a set of first-class, containerized,
multi-version languages — Python, Go, Java, Rust, C++, and now **TypeScript**
(Node.js runtime, single canonical `tsc`, npm, Vitest + V8 100% coverage, a
shareable strict base tsconfig, `node-24` / `node-22` images). See
**[Supported languages](languages.md)** for the fleet-level summary.

## Getting started

New here? Start with **[Getting Started](getting-started.md)**.

## The repositories

| Repo | Delivers |
| --- | --- |
| [`vergil-tooling`](https://github.com/vergil-project/vergil-tooling) | the `vrg-*` Python CLIs, hooks, and the audit/reporting tools |
| [`vergil-claude-plugin`](https://github.com/vergil-project/vergil-claude-plugin) | Claude Code skills, hooks, agents, commands |
| [`vergil-actions`](https://github.com/vergil-project/vergil-actions) | reusable CI/CD workflows |
| [`vergil-containers`](https://github.com/vergil-project/vergil-containers) | dev / prod container images |
| [`vergil-vm`](https://github.com/vergil-project/vergil-vm) | isolated VM environments |
| [`.github`](https://github.com/vergil-project/.github) | epics + org metadata |
| `docs` | this site |

---

_The Roadmap and Activity pages are generated from GitHub issue/epic data — do
not edit them by hand._
