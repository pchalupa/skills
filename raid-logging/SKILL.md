---
name: raid-logging
description: >-
  Create and maintain a RAID log, the register of a project's risks,
  assumptions, issues, and dependencies. Use when the user wants to start or
  update a RAID log, record a risk, blocker, assumption or dependency, prepare
  the log for a status meeting, or invokes a `/raid` command. For a settled
  technical decision use an ADR; for an open one an RFC.
---

# RAID Log

A RAID log is a central register that catalogs a project's **R**isks, **A**ssumptions, **I**ssues, and **D**ependencies in one place, so the team has a single source of truth for the factors shaping the project. It is not written once — it is a **living document**: only accurate when updated regularly, and most valuable when reviewed in status meetings and reflected on in the post-mortem. This skill follows Asana's RAID framing ([asana.com/resources/raid-log](https://asana.com/resources/raid-log)).

**D is always Dependencies here** — a settled technical decision belongs in an ADR, an open one in an RFC, and duplicating either in the log gives the team two records to keep in sync. **A** is the one flexible letter; pick the meaning that fits the project and use it consistently:

- **R — Risks.** Potential problems that could negatively affect your project, identified during planning so they can be mitigated proactively before they occur. *Example: R-1 — a third-party API may be unavailable during integration testing (Likelihood MEDIUM, Impact HIGH, owned by the integration lead).*
- **A — Assumptions or Actions.** **Assumptions** are factors the team believes will hold true and plans around — best for **long-term projects that require significant forethought**. **Actions** are tasks that need completing — best for **projects with many moving parts** to track. Use ownership to keep either accountable.
- **I — Issues.** Problems that occur during a project that you did **not** anticipate. Unlike risks — which you plan for in advance and manage through mitigation — issues pop up unexpectedly and require immediate resolution.
- **D — Dependencies.** Tasks blocked until another task is completed elsewhere, usually by another team. Record what you're waiting on, who owns it on their side, and what stalls here until it lands.

The line that trips people up most: **a risk is a *potential* problem you anticipate; an issue is an *actual* problem that already occurred.** If a logged risk materialises, it becomes an issue.

Confirm which **A** variant the project uses before drafting (or follow the working repo's existing choice) — don't assume.

The exact output shape lives in [`references/raid-template.md`](references/raid-template.md) — read it before drafting. If the working repo has its own `templates/raid.md`, prefer that copy so the log stays in sync with local conventions; otherwise use the bundled template.

## 1. Orient

*Done when you have told the user which log you will write to, which of the four letters the entry belongs under, and the next free ID.*

- Read the template (see above) — that is the exact shape of the output.
- **The log always lives at `docs/raid-log.md` in the repo root.** One log per repo, one known path, so the next reader and the next agent find it without searching. Create `docs/` as you write the first file into it.
- Decide **how to present the log**: this skill produces a Markdown document, but Asana notes a RAID log can also live in a spreadsheet or PM software. If the project already tracks RAID in PM tooling, work there rather than forking a copy.
- Determine whether you are **creating a new log** or **updating an existing one**. Read `docs/raid-log.md` first if it exists and work within its structure and ID scheme — never start a parallel register. If you find a log elsewhere in the repo (`RAID.md`, a project folder), keep working in that file and tell the user it belongs at `docs/raid-log.md`; move it only if they say so.
- If creating, confirm the project name and which **A** variant applies with the user.

## 2. Capture each entry

*Done when every column of the entry's table has a value, including an owner and a status from that table's vocabulary.*

Every entry shares a small common core; each type then adds the fields its table defines (see the template for exact columns):

- **ID number** — a stable, prefixed identifier (see phase 3).
- **Description** — specific and actionable. For a **risk**, state the potential problem and its effect, not a vague label; for a **dependency**, name what you're waiting on and who owns it.
- **Owner** — a single named person (or team) accountable for the entry. Every item has exactly one owner; an unowned item is not being managed.
- **Status** — the type-specific vocabulary defined under each table.

Then, per type:

- **Risks** carry **Likelihood** and **Impact** (each `HIGH` / `MEDIUM` / `LOW`) so the team can judge severity, plus a **Mitigation**.
- **Issues** carry a **Date raised** and a **Severity** (`HIGH` / `MEDIUM` / `LOW`) — they've already happened, so there's no likelihood.
- **Assumptions** carry **Impact if wrong** and **Validated by** (who confirmed it, once confirmed).
- **Dependencies** carry **Depends on** (who we wait on), **Blocks** (what stalls without it), and a **Tracker** (who logs and chases it).

Ask only for what the user cannot point you to. If the answer is in a PRD, RFC, ADR, ticket, or the existing log, read it rather than asking.

## 3. Draft or update the log

*Done when the entry is in the right table, the header's last-reviewed date is current, and no existing ID has moved.*

- Write to `docs/raid-log.md` using the template's structure exactly: a short header, then a section (table) per category with the field set defined above.
- **Assign stable, prefixed IDs**: `R-1` (risk), `A-1` (assumption/action), `I-1` (issue), `DE-1` (dependency), monotonic **per category**. IDs are never reused and never renumbered — closed items keep their ID so cross-references (from an ADR, RFC, ticket, or status report) stay valid.
- **Update regularly — the log is only accurate when current.** When updating, touch only what changed: move a status to `In Progress` or `Closed`, add the new entry with the next ID, and refresh the "last reviewed" date in the header. Leave untouched rows exactly as they are.
- **Close entries, don't delete them.** Set status to `Closed` and keep the row; the history is what makes the post-mortem useful.
- **When a risk materialises, log it as an issue** that references the risk (`I-00N — realised from R-00N`) and close the risk. This preserves the trail from anticipated risk to actual problem.
- **Focus on high-impact items** — Asana's guidance is to log what is High priority, cross-functional, or recurring, and to align with the team on what to include so the log doesn't become unwieldy clutter. A RAID log is **supplemental** to real project-management tooling, not a replacement for it.
- **Timestamp anything that will age** — costs, dates, vendor commitments — so a reader knows whether to re-check a fact rather than trust it.
- **Apply the Voice section below** — descriptions and type-specific response details are prose authored on the user's behalf; they should read as theirs.
- This is a delivered, working document: no authoring scaffolding. Resolve or drop any `(example)` / `> Expected here:` prompts from the template. The defined ID conventions and status vocabulary stay.
- Show the user the result for review. Leave committing to them.

## 4. Verify

*Done when every check below passes.*

- **Every entry is in the correct category** — risks are anticipated potential problems; issues are actual problems that occurred; dependencies are work blocked on someone else, not decisions the team made; the A variant matches what the project chose.
- **Every entry has its type's field set** — ID, description, a single named owner, and status, plus the type-specific columns: risks have likelihood and impact; issues have a date raised and severity; assumptions have impact-if-wrong; dependencies name what they depend on and what they block.
- **Dependencies name the blocking relationship** (depends on → blocks).
- **IDs are stable and unique per category** — nothing renumbered, nothing deleted; closed items retained.
- **The header shows who owns the log and when it was last reviewed**, so staleness is visible.
- **No red flags**: no item without an owner, no risk without a likelihood and impact, no risk that has actually occurred still sitting in the risks table, no clutter of low-impact entries drowning the high-priority ones.

## 5. Point agents at it

*Done when the repo's existing instruction file names this document with a read-trigger, carries the maintenance rules, and every path in it resolves — or when there is no such file and you have said so.*

The document earns nothing if the next agent never opens it, so make the repo's agent instructions name it.

- **Update the instruction file the repo already has.** If `CLAUDE.md` is a symlink to `AGENTS.md` (`test -L`) or a short file importing it (`@./AGENTS.md`), write `AGENTS.md` only, and leave the symlink as it is.
- **Where neither file exists, leave it that way** and tell the user the document's path instead. Whether the repo wants an agent instruction file is their call, not a side effect of writing one document.
- **Keep a `## Documentation` section** holding one table row per document that exists, each naming *when* to read it rather than restating its title:

  ```markdown
  | Document | Read it when |
  |---|---|
  | `docs/raid-log.md` | you hit a risk, blocker, assumption or dependency worth recording |
  ```

- **Replace that section in place** on a re-run. Two Documentation sections is a worse outcome than a stale one, and a row pointing at a file that no longer exists teaches agents to distrust the whole table.
- **Carry the maintenance rules into the section.** They are what keeps the set true: documentation stays accurate because the agent that breaks a document fixes it in the same change, not because somebody audits it later. Write them as their own short block under the table:

  ```markdown
  Keep these documents true as you work:

  - **A document that contradicts the code, another document, or these instructions gets flagged, not quietly fixed.** Say it in your reply: name both sides, say which looks stale — usually the document, since code moves faster — and let the human choose which one changes.
  - **When your change makes a document wrong, update it in the same change.** A feature, service, or dependency you added that the architecture document does not show; a path you renamed, a command you changed, an environment you moved, a library you swapped. The document that names it — or should now name it — is part of the change, not follow-up work.
  - **Never rewrite a recorded decision or a closed entry.** A reversal is a new record that supersedes the old one; the old one stays, with a note pointing forward.
  - **Keep the table above accurate.** A new document gets a row, a deleted one loses its row, and a row whose file has moved gets the new path.
  ```

- `AGENTS.md` is tables and bullets, not prose. Keep the whole file under 100 lines.

## Voice

Every sentence you write here is authored on the user's behalf, so it reads as theirs:

- Active voice, first person plural, contractions. Short-to-medium sentences, 2-4 sentence paragraphs.
- Plain English a non-native speaker follows on the first read. Everyday words: "use" over "utilize", "start" over "commence". Direct description over idiom and metaphor. Reach for a rarer word when it's a technical term or genuinely more precise, and define it on first use.
- Cover the why and the how, not only the what.
- Lead with the point. Concrete examples over abstract description.
- Keep it plain: no hype or superlatives, no filler openers, no stacked hedges, no clauses chained with dashes, no stacked emoji.
- Write for the reader who has to build against this. A sentence that wouldn't change what they build comes out.
