# Hoshin kanri

Vergil does a large share of its work unattended, and the faster development
goes, the more the small instabilities in that work show. The fast, ad hoc
engineering work needs a **management cadence** beside it: a fixed set of
reviews at the start and end of each day, and at week, month, quarter and year
boundaries, each leaving a written record in the [logbook](../logbook/index.md).
This page explains the model those reviews follow, so a later reader
understands why they are shaped as they are.

The decision behind it: **until a better framework is found, Vergil scales
方針管理 hōshin kanri down to one person working with agents, and adopts its
practices in spirit where not to the letter.** The skill, its documents and
the logbook use the method's Japanese terms freely, in content. Vergil is a
personal, single-user stack that takes no outside contributions, so the
vocabulary is chosen for its one reader.

## The method

**The word.** 方針 *hōshin* is policy or direction; the characters are
literally "direction" and "needle", as in a compass needle. 管理 *kanri* is
management. In English it is usually called *policy deployment*.

**Origin.** It grew out of Japan's total quality control movement in the late
1950s and 1960s, building on Deming's plan–do–check–act cycle and Drucker's
management by objectives. Bridgestone Tire is usually credited with naming and
first using it in 1965 (one history dates the company's idea to systematize it
to 1962); Komatsu and Toyota were early adopters. Hewlett-Packard brought it to
the US in the 1980s, and Danaher made it central to the Danaher Business
System.

**How it works.**

1. A three-to-five-year direction yields a small number of yearly
   breakthrough objectives: **the vital few**, deliberately not a long list.
2. Each level turns the objectives above it into its own targets and the means
   of reaching them.
3. **キャッチボール catchball**: targets and means pass back and forth between
   levels until both sides agree they are achievable. Direction comes from the
   top; the means are agreed with the people doing the work.
4. Two halves: **hōshin**, which drives change toward the direction, and
   **日常管理 nichijō kanri** (daily management), which keeps normal operation
   stable and standardised. Hōshin assumes daily management is under control.
5. A nested review cadence (monthly, quarterly), and a yearly review by top
   management, 社長診断 *shachō shindan* (the president's diagnosis). Each
   review's findings feed the next cycle's plan.

## Terms

| Term | In Vergil |
| --- | --- |
| 方針管理 hōshin kanri | The whole system; the skill is `/vergil:hoshin`. |
| 日常管理 nichijō kanri | The daily review, run on every invocation. |
| 朝礼 chōrei | The daily `start` review (morning assembly). |
| 終礼 shūrei | The daily `end` review (closing assembly). |
| 方針 hōshin | The direction document, [`hoshin.md`](../logbook/hoshin.md), holding the vital few. |
| 反省 hansei | Reflection; the `end` review at the week, month and quarter levels. |
| 社長診断 shachō shindan | The yearly `end` review; in a one-person company the president is the human. |
| キャッチボール catchball | Agreeing objectives and means; the agent proposes, the human adjusts, until both accept. |
| 改善 kaizen | Notes, in each entry, on improving the reviews themselves. |

## The two halves

- **Nichijō kanri (daily)** runs on every invocation. `start` is the 朝礼
  chōrei: what ran unattended since the last review, what failed and why, and
  what needs action today. It also shows two trust checks, in the terminal
  only: the outsider report (`vrg-epic-audit --untrusted`), what accounts
  outside the [trusted roles](epics-and-tasks.md#trusted-content) did in
  each Vergil org since the last daily review, and the latest nightly
  trusted-authors check, compared with the host configuration. `end` is the 終礼 shūrei: what changed today, what
  is waiting on the human, and a note for tomorrow's chōrei.
- **Hōshin (week, month, quarter, year)** runs only at boundaries, or when the
  human asks to close a period with `end <level>`. A `start` review sets
  direction for the period; an `end` review is 反省 hansei, reflection on the
  period just finished. The yearly `end` is the 社長診断 shachō shindan.

Every review ends with a kaizen prompt: what should this review do
differently?

## The calendar in brief

- **One command each morning.** `/vergil:hoshin start` works out for itself
  which reviews are due, closes whatever period was left open, and runs them.
  `/vergil:hoshin end [<level>]` is optional: plain `end` closes the day, and
  `end week`, `end month`, `end quarter` or `end year` closes that period and
  every smaller one still open beneath it.
- **One time zone.** Every date and time is computed in the IANA time zone
  recorded in the [logbook](../logbook/index.md), never from the host clock.
- **Days.** A day with a `start` entry is a day worked; a day with none is a
  day off, and nothing about it is flagged.
- **Weeks** run Monday to Sunday, and every period comes from the week: a week
  belongs to the month its Monday falls in, and so to that month's quarter and
  year. The week of Monday 2025-12-29 belongs to December 2025, Q4 2025 and
  2025, so January 2026, Q1 2026 and 2026 begin with the week of Monday
  2026-01-05.
- **Order.** Closing reviews run first, smallest level first, since
  reflection rolls up. Then the daily chōrei runs, so operations are known to
  be stable before direction is set, and the boundary `start` reviews follow
  from the largest level down, since direction flows top-down.
- **Nothing silently skipped.** A period opened and never closed is offered
  for closing at the next `start`: run the missed `end` review late, or record
  it as not performed. At the week level and above, periods with no entries at
  all are named as gaps in the next `start` entry.
- **Windows.** Each review covers a window from the end of the last reviewed
  window, so a skipped day widens the next window rather than losing time.
- **The first run** on an empty logbook opens every level, and sets the first
  vital few by catchball.

## Scaled to one person and agents

In a company, hōshin kanri runs through layers of management, and catchball
passes between them. Vergil has one person and the agents working with them,
so the method is adopted in spirit where not to the letter:

- **The human is every level of management.** The yearly shachō shindan is the
  human's own review of the year; there is no one above them to report to.
- **Catchball is between the human and the agent.** The agent proposes
  objectives, focus and means from the durable record; the human adjusts until
  both accept.
- **The agent does the gathering.** Each review's data comes from durable
  state (git and GitHub today), through `vrg-*` tools where they exist. Every
  check the skill runs by hand is written as the brief for a tool that will
  later do it deterministically.
- **The cadence fits the human's working day.** `start` is run on most
  working days, weekends included, and carries the boundaries; `end` is the
  human's choice, because the day ends when it ends.
- **The record is public and light.** The logbook is Markdown with a minimal
  header, published on this site, and nothing a security record keeps private
  appears in it.
- **Kaizen improves the reviews themselves.** Each entry's kaizen notes feed
  back into the skill, so the first, deliberately thin checklists grow through
  use.

## References

- Akao, Y. (ed.), *Hoshin Kanri: Policy Deployment for Successful TQM*,
  Productivity Press, 1991 (English edition).
- Lean Enterprise Institute lexicon, "Hoshin Kanri":
  <https://www.lean.org/lexicon-terms/hoshin-kanri/>
- Akao's book, Appendix 1, "History of Hoshin Kanri" (2020 reissue):
  <https://www.oreilly.com/library/view/hoshin-kanri/9781000286656/xhtml/18_Appendix01.xhtml>
- All About Lean, "The Evolution of Strategic Management — Hoshin Kanri":
  <https://www.allaboutlean.com/evolution-of-strategic-management-hoshin-kanri/>
- Wikipedia, "Hoshin Kanri": <https://en.wikipedia.org/wiki/Hoshin_Kanri>

The Bridgestone dates come from the Akao and All About Lean histories; the
Lean Enterprise Institute and Wikipedia pages give no dates. The term 社長診断
for the yearly review is the human's own, from years of working in Japanese;
English sources call it the president's diagnosis.
