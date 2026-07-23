---
name: raid-log
description: >-
  Create and maintain a RAID log — the central register tracking a project's
  Risks, Assumptions (or Actions), Issues, and Decisions (or Dependencies) —
  capturing each entry from the bundled template and keeping the log current as
  things change. Use when the user wants to start, write, or update a RAID log,
  track project risks/assumptions/actions/issues/decisions/dependencies, log a
  new risk or issue, prep the log for a status meeting, or invoke a `/raid`
  command. For recording a settled technical decision in depth use an ADR; for
  an open decision needing reviewer input use an RFC.
---

# RAID Log

A RAID log is a central register that catalogs a project's **R**isks,
**A**ssumptions, **I**ssues, and **D**ecisions in one place, so the team has a
single source of truth for the factors shaping the project. It is not written
once — it is a **living document**: only accurate when updated regularly, and
most valuable when reviewed in status meetings and reflected on in the
post-mortem. This skill follows Asana's RAID framing
([asana.com/resources/raid-log](https://asana.com/resources/raid-log)).

Two of the four letters are flexible — pick the meaning that fits the project
and use it consistently:

- **R — Risks.** Potential problems that could negatively affect your project,
  identified during planning so they can be mitigated proactively before they
  occur. *Example: R-1 — third-party payment API failure during app testing
  (Likelihood MEDIUM, Impact HIGH, owned by Alex Martinez).*
- **A — Assumptions or Actions.** **Assumptions** are factors the team believes
  will hold true and plans around — best for **long-term projects that require
  significant forethought**. **Actions** are tasks that need completing — best
  for **projects with many moving parts** to track. Use ownership to keep either
  accountable.
- **I — Issues.** Problems that occur during a project that you did **not**
  anticipate. Unlike risks — which you plan for in advance and manage through
  mitigation — issues pop up unexpectedly and require immediate resolution.
- **D — Decisions or Dependencies.** **Decisions** are concrete choices that
  advance the project; document the **what** (the decision), the **who** (person
  or team responsible), and the **why** (the rationale). **Dependencies** are
  tasks blocked until another task is completed elsewhere.

The line that trips people up most: **a risk is a *potential* problem you
anticipate; an issue is an *actual* problem that already occurred.** If a logged
risk materialises, it becomes an issue.

Confirm which **A** and **D** variants the project uses before drafting (or
follow the working repo's existing choice) — don't assume.

The exact output shape lives in [`references/raid-template.md`](references/raid-template.md) —
read it before drafting. If the working repo has its own `templates/raid.md`,
prefer that copy so the log stays in sync with local conventions; otherwise use
the bundled template.

## 1. Orient (do this quietly, then confirm with the user)

- Read the template (see above) — that is the exact shape of the output.
- Decide **how to present the log**: this skill produces a Markdown document,
  but Asana notes a RAID log can also live in a spreadsheet or PM software. If
  the project already tracks RAID elsewhere, work there rather than forking a
  copy.
- Determine whether you are **creating a new log** or **updating an existing
  one**. Look for a RAID log already in the repo (commonly `docs/raid-log.md`,
  `RAID.md`, or a project/PM folder). If one exists, read it first and work
  within its structure and ID scheme — never start a parallel register.
- If creating, confirm the location, the project name, and which **A**/**D**
  variants apply with the user.

## 2. Capture each entry

Every entry shares a small common core; each type then adds the fields its
table defines (see the template for exact columns):

- **ID number** — a stable, prefixed identifier (see phase 3).
- **Description** — specific and actionable. For a **risk**, state the potential
  problem and its effect, not a vague label; for a **dependency**, name what
  you're waiting on and who owns it.
- **Owner** — a single named person (or team) accountable for the entry. Every
  item has exactly one owner; an unowned item is not being managed.
- **Status** — the type-specific vocabulary defined under each table.

Then, per type:

- **Risks** carry **Likelihood** and **Impact** (each `HIGH` / `MEDIUM` /
  `LOW`) so the team can judge severity, plus a **Mitigation**.
- **Issues** carry a **Date raised** and a **Severity** (`HIGH` / `MEDIUM` /
  `LOW`) — they've already happened, so there's no likelihood.
- **Assumptions** carry **Impact if wrong** and **Validated by** (who confirmed
  it, once confirmed).
- **Dependencies** carry **Depends on** (who we wait on), **Blocks** (what
  stalls without it), and a **Tracker** (who logs and chases it).

Ask only for what the user cannot point you to. If the answer is in a PRD, RFC,
ADR, ticket, or the existing log, read it rather than asking.

## 3. Draft or update the log

- Write to `<raid-location>` using the template's structure exactly: a short
  header, then a section (table) per category with the field set defined above.
- **Assign stable, prefixed IDs**: `R-1` (risk), `A-1` (assumption/action),
  `I-1` (issue), `DE-1` (dependency; `D-1` if the project uses decisions),
  monotonic **per category**. IDs are never reused and never renumbered — closed items
  keep their ID so cross-references (from an ADR, RFC, ticket, or status report)
  stay valid.
- **Update regularly — the log is only accurate when current.** When updating,
  touch only what changed: move a status to `In Progress` or `Closed`, add the
  new entry with the next ID, and refresh the "last reviewed" date in the
  header. Do not renumber or reflow untouched rows.
- **Close entries, don't delete them.** Set status to `Closed` and keep the row;
  the history is what makes the post-mortem useful.
- **When a risk materialises, log it as an issue** that references the risk
  (`I-00N — realised from R-00N`) and close the risk. This preserves the trail
  from anticipated risk to actual problem.
- **Focus on high-impact items** — Asana's guidance is to log what is High
  priority, cross-functional, or recurring, and to align with the team on what
  to include so the log doesn't become unwieldy clutter. A RAID log is
  **supplemental** to real project-management tooling, not a replacement for it.
- **Timestamp anything that will age** — costs, dates, vendor commitments — so a
  reader knows whether to re-check a fact rather than trust it.
- **Apply the `voice-profile` skill** if available — descriptions and
  type-specific response details are prose authored on the user's behalf; they
  should read as theirs.
- This is a delivered, working document: no authoring scaffolding. Resolve or
  drop any `(example)` / `> Expected here:` prompts from the template. The
  defined ID conventions and status vocabulary stay.
- Show the user the result for review. Don't commit unless asked.

## 4. Verify before handing back

- **Every entry is in the correct category** — risks are anticipated potential
  problems; issues are actual problems that occurred; the A and D variants match
  what the project chose.
- **Every entry has its type's field set** — ID, description, a single named
  owner, and status, plus the type-specific columns: risks have likelihood and
  impact; issues have a date raised and severity; assumptions have impact-if-wrong;
  dependencies name what they depend on and what they block.
- **Dependencies name the blocking relationship** (depends on → blocks).
- **IDs are stable and unique per category** — nothing renumbered, nothing
  deleted; closed items retained.
- **The header shows who owns the log and when it was last reviewed**, so
  staleness is visible.
- **No red flags**: no item without an owner, no risk without a likelihood and
  impact, no risk that has actually occurred still sitting in the risks table, no
  clutter of low-impact entries drowning the high-priority ones.
