---
level: yearly
mode: start
period: 2026
window_from: 2026-01-05T00:00-05:00
window_to: 2026-11-22T07:16-05:00
ran_at: 2026-11-22T07:16-05:00
late: false
not_performed: false
---

# Yearly start 2026

The first yearly review, run as part of the logbook's bootstrap. The 2026
year began on Monday 2026-01-05 and ends on Monday 2027-01-04, so this
entry sets a lasting direction and only the last six weeks of the year's
objectives; the 2027 yearly `start` reads it and sets the next year.

## Findings

No earlier logbook entries exist: this is the first run.

The fleet's shape, as it stands now:

| Org | Status | Public repos | Private repos | Archived |
| --- | --- | --- | --- | --- |
| vergil-project | active | 15 | 0 | 0 |
| logical-minds-foundry | active | 4 | 2 | 0 |
| mnemosys-project | active | 3 | 0 | 0 |
| mq-rest-admin-project | inactive | 9 | 0 | 3 |
| diogenes-project | inactive | 2 | 0 | 0 |
| vergils-nemesis | inactive | 1 | 0 | 0 |

The public repos' primary languages: 15 Shell, 11 Python, 3 Rust, 2
TypeScript, 1 Go, 1 Java and 1 Ruby (the Java, the Ruby and one Shell repo
archived).

How the fleet grew in 2026:

- **January to February:** `mq-rest-admin-project`, one library built in
  several languages, which needed shared validation across all of them.
- **January to March:** the core Vergil repos, `vergil-actions`,
  `vergil-tooling`, `vergil-claude-plugin` and `vergil-containers`.
- **May to June:** the org `.github` (the planning home), `vergil-vm` and
  the docs site.
- **June to August:** the projects the toolkit serves, in
  `logical-minds-foundry` (the MQ lab) and `mnemosys-project` (melete).
- **October to November:** `vergil-python`, the signed `packages`
  repository, and six language fixture repos that prove each release.

The pace is still rising. Commits to the planning repo went from 6 in
September to 61 in October and 125 in the first three weeks of November.
About 35 epics closed in November alone.

Open epics: 7 in vergil-project (the largest are
`vergil-project/.github#1384`, agents submit and finalize their own PRs,
1 of 59 tasks done; `vergil-project/.github#1374`, agent review against
cited coding standards, 1 of 28; and `vergil-project/.github#1440`,
resumable releases, 10 of 31), 17 in logical-minds-foundry and 1 in
mnemosys-project.

## Decisions

**The three-to-five-year direction:**

> Vergil is my personal engineering toolkit, my Jarvis: the means by which
> I do what I did for large enterprises, building tooling and software
> that automates complex problems sustainably and securely, scaled down to
> one person without lowering the bar on standards, security, quality or
> maintainability. It is a lab first: free to experiment, borrowable by
> anyone, built for no one else. It is also a factory that produces real
> things. My time goes to brainstorming designs, breaking large
> undertakings into collections of epics, and saying "go"; large pieces
> then get built with minimal intervention. The hoshin cadence, daily
> through yearly, is how one person steers that factory.

**The rest of 2026: round out Vergil 2 and its management ecosystem, and
prepare 2027.** Every remaining item counts as rounding out Vergil 2. The
biggest accelerator is `vergil-project/.github#1384`, agents submitting and
merging their own PRs into develop, which removes the main human blocker.
Releases stay with the human for now; agents gain more autonomy as better
logging and auditing earn confidence. The backlog burn-down began this
week, at two or three small or medium planning items a day. Large items
come in only when they make planning itself faster.

The year's breakthrough objectives, for the six weeks left:

1. **The management cadence runs:** hoshin every working day, and every
   period closed.
2. **The queues get under control:** planning and deferred burned down;
   intake moving quickly through planning, `epic-create` and
   `epic-implement`; items deferred only when blocked, never for lack of
   time.
3. **The fleet stays green:** every red run gets an issue and an owner
   within a day.
4. **2027 is planned before it starts:** the forge migration and Vergil 3
   specified before the new home server arrives in February.

**2027, previewed** for the 2027 yearly `start` to set:

- **Off GitHub, early in the year:** a self-hosted Forgejo on a dedicated
  home server, the foundation of a home enterprise platform, with as much
  of the core moved onto it as possible.
- **Vergil 3:** most of the Python ported to Rust, for performance and
  security, and the design revisited with what Vergil 2 taught.
- **The MQ lab, finished:** the showcase for what this tooling makes
  possible, published as a lab anyone can download.
- **The writing pipeline:** reviving The Infrastructure Mindset and its
  queue of article ideas, and possibly an old personal website.
- **The MIMIR adversary, medium to low priority:** a second workstation
  that seeks out the seams and security holes in the architecture.
- **The toolkit applied to the other projects** as time allows; melete
  stays a lifelong hobby project with no deadline.

## Follow-ups

None.

## Kaizen

- The bootstrap should run top-down, the yearly review first, and the
  daily last: direction set before the day's operations were judged worked
  better here than the skill's fixed order.
- The yearly review needs the fleet's history, not only a snapshot. A
  fleet-history check (repo creation dates, epics closed per month) would
  replace several ad hoc queries.
