# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/)
and this project adheres to [Semantic Versioning](https://semver.org/).

## [0.1.0] - 2026-11-10

### Bug fixes

- match org dark-mode palette (follow OS color scheme, #60) (#4)
- add build-only docs job to ci.yml so docs/docs posts on PRs (#24)
- link versioned Vergil docs sites through /latest/ (#53)

### CI

- add on: issues.closed caller for event-driven epic rollup (#8)

### Chores

- step 3 - vergil.toml
- step 4 - config files
- step 5 - CI/CD workflows
- step 6 - docs site
- correct copyright holder (#10)
- drop dead .venv-host/ line, ignore .venv/ (#20)
- reconcile .gitignore to baseline + add ops.yml (#25) (#26)
- sync .gitignore to the canonical baseline (#28)
- sync the vergil-managed .gitignore block (#30)
- turn off Claude Code co-author attribution (#40)
- add the report-only dependabot.yml (#42)
- sync the vergil-managed .gitignore block (#52)
- sync the vergil-managed .gitignore block (#60)
- sync the vergil-managed .gitignore block (#72)
- sync the managed .claude/settings.json (#95)
- sync the managed .claude/hooks/guard.sh (#96)

### Documentation

- add the Management framework guide to the docs site (#11) (#12)
- polish the framework guide — scannable tools, tighter framing, named skills (#15)
- nest Roadmap and Activity under the Management section (#13) (#16)
- reflect the compliance model and standing retirement in management docs (#18)
- TypeScript language support summary (#22)
- document dynamic version-agnostic CI gates (#338) (#34)
- add package repository page and release packaging in CI (#35) (#36)
- add vrg-release and vrg-update-deps to tools-and-skills (#38)
- describe the version window and per-series runtime packages (#45)
- describe intake -> planning -> ready and the every-issue-has-an-epic invariant (#47)
- summarise report-only Dependabot posture in nightly governance (#49)
- show the vergil-python runtime version with its release suffix (#56)
- describe quality / evidence aggregating common and custom (#58)
- link the paad usage policy from the interaction doctrine (#62)
- reflect security records in the management framework pages (#65)
- describe the three-queue model (intake, planning, deferred) (#66)
- describe several advisories per security item (#69)
- steering and settings sweep (epic 558) (#70)
- mention the agentic-review gate in the implement path (#74)
- describe human-gated design-review rounds in the epic-create workflow (#76)
- sync the template's build/ exception line (#78)
- say paad review rounds need the delegate and a Lima session (#80)
- say every box installs paad from the fork and drop the agentic-review gate (#83)
- drop removed tools and the migrate-repo skill (#85)
- list vrg-ruling and vrg-ruling-reply on the Tools & skills page (#87)
- describe merge review in the workflow and tools pages (#90)
- remove Java from the supported languages (#92)

### Features

- org site — narrative home, roadmap + activity pages, nav (#60) (#2)
- add nightly + on-demand docs-refresh caller (#6)
- publish the multi-page activity log (vrg-activity-log --out-dir) (#54)

### Refactoring

- drop the hardcoded matrix inputs from ci.yml (#32)
