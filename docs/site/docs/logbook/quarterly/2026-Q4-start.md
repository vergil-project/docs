---
level: quarterly
mode: start
period: 2026-Q4
window_from: 2026-10-05T00:00-04:00
window_to: 2026-11-22T08:02-05:00
ran_at: 2026-11-22T08:02-05:00
late: false
not_performed: false
---

# Quarterly start 2026-Q4

The first quarterly review, run as part of the logbook's bootstrap. Q4
2026 began on Monday 2026-10-05 and ends on Monday 2027-01-04, so the
vital few set here cover its last six weeks.

## Findings

No earlier quarterly or monthly entries exist: this is the first run.

**Delivery rate,** fleet totals per Monday-to-Sunday week (the week of
2026-11-16 is partial, to Sunday morning):

| Week of | Tasks closed | Epics closed | PRs merged by the human | PRs merged by automation |
| --- | --- | --- | --- | --- |
| 2026-08-17 | 52 | 2 | 57 | 0 |
| 2026-08-24 | 80 | 2 | 86 | 0 |
| 2026-08-31 | 77 | 2 | 85 | 0 |
| 2026-09-07 | 10 | 2 | 6 | 0 |
| 2026-09-14 | 61 | 4 | 51 | 0 |
| 2026-09-21 | 5 | 0 | 5 | 0 |
| 2026-09-28 | 104 | 5 | 78 | 0 |
| 2026-10-05 | 144 | 3 | 139 | 0 |
| 2026-10-12 | 214 | 10 | 190 | 1 |
| 2026-10-19 | 384 | 14 | 408 | 3 |
| 2026-10-26 | 373 | 10 | 305 | 0 |
| 2026-11-02 | 223 | 6 | 198 | 1 |
| 2026-11-09 | 497 | 11 | 506 | 7 |
| 2026-11-16 | 584 | 14 | 539 | 20 |

Tasks closed per week rose about tenfold from late August. Over the last
four weeks the fleet closed about 420 tasks and 10 epics a week, and the
first automation-merged PRs appeared as the self-merging work landed.
Nearly all of it is in vergil-project.

**Queues:**

| Home | Queue | Items | Oldest | Median age | Sizing |
| --- | --- | --- | --- | --- | --- |
| vergil-project | intake | 42 | 4 days | 3 days | unsized |
| vergil-project | planning | 61 | 30 days | 7 days | 7 S, 19 M, 4 L, 2 XL, 29 unsized; about 470 tasks across the sized items |
| vergil-project | deferred | 37 | 185 days | 22 days | 25 unsized |
| logical-minds-foundry | intake | 23 | 158 days | 121 days | unsized |
| logical-minds-foundry | planning | 0 | | | |
| logical-minds-foundry | deferred | 4 | 137 days | 110 days | unsized |
| mnemosys-project | intake | 1 | 24 days | 24 days | unsized |

mnemosys-project has no planning or deferred queue epic.

**Open epics:** vergil-project's in-flight epics have about 140 tasks left:
`vergil-project/.github#1384` 58, `vergil-project/.github#1374` 27,
`vergil-project/.github#1440` 21, `vergil-project/.github#375` 17,
`vergil-project/.github#388` 12, `vergil-project/.github#708` 4 and
`vergil-project/.github#1318` 3. logical-minds-foundry has 17 open epics and
mnemosys-project 1.

## Decisions

**The vital few** were set for the first time, by catchball, from the
yearly `start`'s objectives for the rest of 2026; they are now in
[hoshin.md](../hoshin.md):

1. Run the cadence.
2. Drain the queues (vergil-project's, for Q4).
3. Keep the fleet green.
4. Plan 2027.

**The capacity plan:**

1. **Capacity:** about 10 epics a week, so roughly 60 in the six weeks to
   2027-01-04 if the pace holds; the pace is rising.
2. **Finish first:** `vergil-project/.github#1384`, the accelerator, then
   `vergil-project/.github#1318` and `vergil-project/.github#1440`.
   `vergil-project/.github#1374`, `vergil-project/.github#375` and
   `vergil-project/.github#388` advance as they fit.
3. **The planning burn-down:** two or three small or medium items a day,
   10 to 15 a week, enough to reach 20 or fewer net of new inflow. The risk
   is inflow: 42 intake items arrived in four days. Large and extra-large
   items wait unless they make planning itself faster.
4. **December:** brainstorm the forge migration and Vergil 3 through
   `epic-create`.
5. **Security records:** one cleanup pass first, closing the stale
   records; `vergil-project/.github#724` (hardening the security-records
   model) waits until the cleanup shows what is live.
6. **Does not fit this quarter:** the logical-minds-foundry backlog. Its
   intake gets one `epic-review` pass, deferring the MQ items with the
   blocker that MQ work resumes once Vergil 2 is rounded out.

## Follow-ups

- `mnemosys-project/.github#116`: create mnemosys-project's missing
  planning and deferred queue epics.

## Kaizen

- The delivery-rate check took 168 search calls, run from a scratch
  script. It is the clearest candidate for a `vrg-*` tool (history,
  `vergil-project/.github#1317`), and that tool should also draw the
  charts that show how the rate of progress is changing over time.
- `vrg-queue --json` gives no ages or size summary; a scratch script
  computed them. The Queue size and age check needs that summary from the
  tool.
- The queue checks should report a missing queue epic as a finding rather
  than an error, as mnemosys-project's planning and deferred queues showed.
