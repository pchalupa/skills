---
name: adr-authoring
description: >-
  Author an ADR (Architecture Decision Record) recording why an
  expensive-to-reverse technical decision was made and which alternatives lost.
  Use when the user wants to write an ADR, record the reasoning behind a
  technical decision, or invokes an `/adr` command. For a decision still open
  and needing reviewer input, use an RFC.
---

# ADR Authoring

An ADR captures the reasoning behind a significant technical decision — the *why*, not just the *what*. Code shows what was built; the ADR explains why it was built this way and what alternatives were rejected. This is the highest-value documentation there is: a 10-minute ADR prevents a 2-hour debate about the same decision six months later, and it stops future engineers (and agents) from re-deciding something already settled.

ADRs are for decisions that are **expensive to reverse** — choosing a framework, library, or major dependency; designing a data model; selecting an auth strategy; picking an API architecture (REST vs. GraphQL vs. tRPC); or committing to a build tool, hosting platform, or real-time transport. If the choice is cheap to change or has no real alternatives, it probably doesn't warrant an ADR.

This is **human-led authoring**: the user owns the understanding and the decision; you assist. **Grill the thinking to the ground, then draft.** Follow the phases in order.

The exact output shape lives in [`references/adr-template.md`](references/adr-template.md) — read it before drafting. If the working repo has its own `templates/adr.md`, prefer that copy so the ADR stays in sync with local conventions; otherwise use the bundled template.

## 1. Orient

*Done when you have told the user the ADR number you will use and they have confirmed this is a decision already made.*

- Read the template (see above) — that is the exact shape of the output.
- **ADRs always live in `docs/decisions/` in the repo root.** One known directory, so the next reader and the next agent find them without searching; create it as you write the first record into it. If the repo already keeps ADRs somewhere else (`docs/adr/`, a project folder), keep working there and tell the user where they belong; move them only if the user says so. Scan for existing `ADR-NN-*.md` files and determine the **next sequential number**, zero-padded to two digits (e.g. `ADR-04`). Numbers are monotonic and never reused, even for superseded decisions. Tell the user the number you'll use.
- If a `README.md` (or equivalent) sits alongside the ADRs, read it for the project's conventions and any "likely decisions to capture" list.
- Check whether this decision touches an existing open question, dependency, or risk the project tracks — an ADR records a decision **already made**. If it is really still open and needs reviewer input, say so: it may belong in an RFC instead.

## 2. Grill relentlessly

*Done when context, decision, at least two alternatives with the reason each lost, and consequences all have answers the user has confirmed.*

Interview the user one question at a time, giving your recommended answer each time, and don't move on from a branch until it's resolved. If a `grilling` (or `grill-me`) skill is available, invoke it to run this; otherwise conduct the interview directly. Ground it in the Nygard ADR discipline (https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) and the community ADR hub (https://github.com/architecture-decision-record/architecture-decision-record). Pin down specifically:

- **Context / forces in tension** — the constraints, requirements, and competing forces. Keep these value-neutral: facts, not opinions. What is actually pushing on this decision? Which open question does it unblock?
- **The decision itself** — what exactly will be done, precise enough to act on.
- **Alternatives** — what else was on the table and the concrete reason each lost. If you can't name real alternatives, the decision may not be significant enough to warrant an ADR — say so.
- **Consequences** — the full picture: trade-offs being *accepted*, follow-on work, new risks, and the **integration impact** on downstream consumers of the system this decision touches.

If a question can be answered from the project's source materials or existing deliverables, go read them instead of asking.

## 3. Draft

*Only once the user confirms the thinking is settled. Done when every template section is filled and the file carries no scaffolding.*

- Write to `docs/decisions/ADR-NN-<short-kebab-title>.md` using the template's sections exactly: YAML frontmatter (`date`) plus Context, Decision, Alternatives considered, and Consequences. Make the `<short-kebab-title>` a present-tense imperative verb phrase — `choose-realtime-transport`, not `realtime-transport-decision` — so the filename reads as the decision itself.
- **One decision per ADR.** Each record captures a single architecture decision. If the grill surfaces a second decision riding along, split it into its own ADR (next number) and cross-reference — one ADR triggering follow-on ADRs is normal, not a smell.
- **Link inline, so a reader can follow every claim.** Anything a reader would otherwise go and search for becomes a link the first time it appears:
  - **A sibling record** — a relative link that resolves from this file: `[ADR-03](ADR-03-adopt-expo-as-the-framework.md)`, `[RFC-05](../requests-for-comments/RFC-05-api-error-resilience.md)`.
  - **A technology, product, API, or standard** the document names — its official documentation.
  - **Code and config** — a repo-relative path in backticks, or a link where the host renders one.
  Resolve every path you write: a link into a file that does not exist is worse than plain text. Link on first mention only, and where you don't have a URL, name the thing and leave it unlinked rather than guessing at one.
- **Timestamp anything that will age.** Costs, quotas, SLAs, pricing tiers, and version numbers drift; note the date a fact was true as of, so a future reader knows whether to re-check it rather than trust it blindly.
- **Apply the Voice section below** — this is prose authored on the user's behalf; it must read as theirs, not generic AI output.
- Write it as **a conversation with a future developer**: full sentences in paragraphs, active voice ("We will …"), not fragments. Keep it to one or two pages.
- **Never rewrite or delete a settled ADR to change a past decision.** If this decision reverses an earlier one, write it as a new ADR that references and supersedes the old one, and add a forward-pointing note (`Superseded by ADR-NN`) to the old ADR. Old ADRs stay in place — they capture historical context.
- This is a delivered document: no authoring scaffolding. Resolve or drop `(illustrative)` and `> Expected here:` prompts. An ID that names a file becomes a link to it; an ID that lives somewhere else, such as a RAID or ticket reference, stays as plain text.
- If drafting surfaces a gap the sources don't cover, log it wherever the project tracks open questions (append-only).
- Show the user the draft for review. Leave committing to them.

## 4. Verify

*Done when every check below passes.*

- **Context is value-neutral** — facts and forces, not opinions or the conclusion in disguise.
- **The decision is actionable** — someone could implement it from the ADR alone.
- **One decision only** — a second decision hiding inside gets split into its own record.
- **At least two real alternatives** are named, each with the concrete reason it lost. If you can't, flag that this may not merit an ADR.
- **Consequences are honest** — the trade-offs being *accepted*, not just the upsides, plus follow-on work, new risks, and integration impact on downstream consumers.
- **Every link resolves** — sibling records by relative path, external claims to their documentation, and no invented URLs.
- **No red flags**: no decision left without written rationale, no restating what the code does instead of why, no leftover scaffolding.

## 5. Point agents at it

*Done when the repo's existing instruction file names this document with a read-trigger, carries the maintenance rules, and every path in it resolves — or when there is no such file and you have said so.*

The document earns nothing if the next agent never opens it, so make the repo's agent instructions name it.

- **Update the instruction file the repo already has.** If `CLAUDE.md` is a symlink to `AGENTS.md` (`test -L`) or a short file importing it (`@./AGENTS.md`), write `AGENTS.md` only, and leave the symlink as it is.
- **Where neither file exists, leave it that way** and tell the user the document's path instead. Whether the repo wants an agent instruction file is their call, not a side effect of writing one document.
- **Keep a `## Documentation` section** holding one table row per document that exists, each naming *when* to read it rather than restating its title:

  ```markdown
  | Document | Read it when |
  |---|---|
  | `docs/decisions/` | you are about to re-decide something an `ADR-NN` may already settle |
  ```

- **A row pointing at a directory relies on the filenames inside it.** `ADR-04-choose-realtime-transport.md` tells an agent listing the directory whether this is the record it needs; `ADR-04-notes.md` makes it read all of them. The table gets it to the right directory, the filename gets it to the right file.
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
