# Logbook

Vergil does a large share of its work unattended: lifecycle and
upstream-refresh PRs open and merge themselves, CD builds publish images, and
scheduled and event-driven jobs run whether or not anyone is looking. The
logbook is the written record of the **management cadence** that looks at all
of it: a fixed set of reviews at the start and end of each day, and at week,
month, quarter and year boundaries, run with `/vergil:hoshin`. Every review
writes one dated entry here, even when it finds nothing, so the record proves
each review ran and later reviews can read earlier ones.

- [Hoshin kanri](../management/hoshin-kanri.md): the method behind the
  reviews, and why they are shaped as they are.
- [Hoshin](hoshin.md): the direction document, holding the fleet's vital few.
- [Fleet](fleet.md): the fleet roster, each org active or inactive.
- The entries, by level: [daily](daily/index.md), [weekly](weekly/index.md),
  [monthly](monthly/index.md), [quarterly](quarterly/index.md) and
  [yearly](yearly/index.md).

## Time zone

Every date and time in the logbook is computed in this IANA time zone, never
from the host clock:

Time zone: America/New_York

## File names

Every path is ASCII and names the period the review covers, plus the mode,
`start` or `end`:

| Level | File name | Example |
| --- | --- | --- |
| Daily | `daily/YYYY-MM-DD-<mode>.md` | `daily/2026-11-18-start.md` |
| Weekly | `weekly/<Monday's YYYY-MM-DD>-<mode>.md` | `weekly/2026-11-16-end.md` |
| Monthly | `monthly/YYYY-MM-<mode>.md` | `monthly/2026-11-start.md` |
| Quarterly | `quarterly/YYYY-Qn-<mode>.md` | `quarterly/2026-Q4-end.md` |
| Yearly | `yearly/YYYY-<mode>.md` | `yearly/2026-start.md` |

Weeks run Monday to Sunday, and a week is named by its Monday's date. Every
period comes from the week: a week belongs to the month its Monday falls in,
and so to that month's quarter and year. The week of Monday 2025-12-29
belongs to December 2025, Q4 2025 and the year 2025.

## Entries

An entry is free-form Markdown under a minimal header, YAML front matter with
exactly these keys:

```yaml
level: daily            # daily | weekly | monthly | quarterly | yearly
mode: start             # start | end
period: 2026-11-18      # the file name's period part
window_from: 2026-11-17T18:42-05:00   # omitted on a not-performed entry
window_to: 2026-11-18T08:05-05:00     # omitted on a not-performed entry
ran_at: 2026-11-18T08:05-05:00
late: false
not_performed: false
```

The body is the review: what was found, what was decided, the follow-up issues
filed, and the kaizen notes. A run where nothing happened still writes an
entry, saying so. A "not performed" entry is a few lines: that the review was
not done, and the human's note on why, if any. The format is kept loose on
purpose.

A run never edits an entry already merged by an earlier run. A late or "not
performed" close is its own new file.

## Indexes

Each level's `index.md` lists that level's entries, newest first, under its
level heading, one line per entry:

```markdown
- [<period> <mode>](<file name>)
```

An entry that ran late or was not performed adds a space and `(late)` or
`(not performed)` at the end of its line. A run adds its own entries to the
indexes in the same PR as the entries.
