---
name: rfc-authoring
description: >-
  Author an RFC (Request for Comments) proposing a technical decision for
  asynchronous review. Use when the user wants to write an RFC, put an open
  technical decision to reviewers, or invokes an `/rfc` command. For a decision
  already made, use an ADR.
---

# RFC Authoring

An RFC is how a decision gets made **asynchronously** — a written proposal that seeks feedback, weighs alternatives, and drives to a merge. This is **human-led authoring**: the user owns the understanding and the decision; you assist. **Do not draft RFC prose until the thinking has been grilled to the ground.** Follow the three phases in order.

The exact output shape lives in [`references/rfc-template.md`](references/rfc-template.md) — read it before drafting. If the working repo has its own `templates/rfc.md`, prefer that copy so the RFC stays in sync with local conventions; otherwise use the bundled template.

## 1. Orient

*Done when you have told the user the RFC number you will use, and they have confirmed the decision is still open and needs reviewer input.*

- Read the template (see above) — that is the exact shape of the output.
- **RFCs always live in `docs/requests-for-comments/` in the repo root.** One known directory, so the next reader and the next agent find them without searching; create it as you write the first record into it. If the repo already keeps RFCs somewhere else (`docs/rfcs/`, a project folder), keep working there and tell the user where they belong; move them only if the user says so. Scan for existing `RFC-NN-*.md` files and determine the **next sequential number**, zero-padded to two digits (e.g. `RFC-04`). Numbers are monotonic and never reused. Tell the user the number you'll use.
- If a `README.md` (or equivalent) sits alongside the RFCs, read it for the project's lifecycle and conventions.
- Check whether this decision touches an existing dependency, risk, `ADR-NN`, or RFC — an RFC is for a decision still **open** that needs reviewer input. If it is really already decided, say so: it may belong in an ADR instead.

## 2. Grill relentlessly

*Done when the proposal, its impact on every affected team, at least two alternatives with the reason each lost, and the open questions blocking acceptance all have answers the user has confirmed.*

Interview the user one question at a time, giving your recommended answer each time, and don't move on from a branch until it's resolved. If a `grilling` (or `grill-me`) skill is available, invoke it to run this; otherwise conduct the interview directly. Pin down specifically:

- **The decision & why now** — what exactly are reviewers being asked to decide, and what forces the decision now? What does it unblock?
- **Problem & constraints** — the situation today, stated as value-neutral facts. Whose problem is it, and for whom?
- **The proposal** — precise enough to act on. If it affects a system or API contract that other teams consume, pin the specifics: endpoints, auth/token flow, request/response fields & types, call sequences, error/edge-case behaviour, environments.
- **Alternatives** — what else is on the table and the concrete reason each loses. If there are no real alternatives, this may not warrant an RFC — say so.
- **Trade-offs, risks & impact** — what is accepted, what new risks appear, and how the change lands on the systems and teams affected.
- **Who reviews, and by when** — the reviewers, a realistic **Comment by** date, and a **Decision by** date.

If a question can be answered from the project's source materials or existing deliverables, go read them instead of asking.

## 3. Draft

*Only once the user confirms the thinking is settled. Done when every template section is filled, the reviewers and both dates are set, and the decision section is left for the outcome.*

- Write to `docs/requests-for-comments/RFC-NN-<short-kebab-title>.md` using the template's sections exactly. Make the `<short-kebab-title>` name the decision under review — `choose-realtime-transport`, not `transport-proposal` — so the filename tells a reader picking from a directory listing what this RFC decides.
- **Apply the Voice section below** — this is prose authored on the user's behalf; it must read as theirs, human and consistent, not generic AI output.
- Write it to be **read by a reviewer skimming for the ask, then reading for the reasoning**: a Summary graspable in one pass, full sentences in the body, diagrams (Mermaid) where a flow or state machine needs one. Not a dump of fragments.
- **Keep it to 7 pages at most.** Following Miller's law, a reviewer holds only so much in working memory; a longer decision doc costs comprehension. If it won't fit, that's a signal the RFC is bundling more than one decision — split it or tighten the reasoning, don't shrink the type.
- **Link inline, so a reviewer can click through instead of taking a claim on faith.** Anything they would otherwise go and search for becomes a link the first time it appears:
  - **A technology, product, API, capability, or sourced fact** (pricing, retirement dates, version behaviour) — its official documentation.
  - **A sibling record** — a relative link that resolves from this file: `[ADR-03](../decisions/ADR-03-adopt-expo-as-the-framework.md)`, `[RFC-02](RFC-02-build-pipeline.md)`.
  - **Code and config** — a repo-relative path in backticks, or a link where the host renders one.
  Resolve every path you write: a link into a file that does not exist is worse than plain text. Link on first mention only, and where you don't have a URL, name the thing and leave it unlinked rather than guessing at one.
- Created date is today. Fill Authors/Reviewers and the agreed dates. The accepted, rejected, or withdrawn outcome goes in _Decision & outcome_ once it is decided.
- Leave **Decision & outcome** empty — it fills in when the RFC is closed. Review feedback itself is captured wherever the project collects it (e.g. inline comments), not in the file.
- This is a document reviewers will read: no authoring scaffolding. Resolve or drop `(illustrative)` and `> Expected here:` prompts. An ID that names a file becomes a link to it; an ID that lives somewhere else, such as a RAID or ticket reference, stays as plain text.
- If drafting surfaces a gap the sources don't cover, log it wherever the project tracks open questions (append-only).
- Show the user the draft for review. Leave committing to them.

## 4. Point agents at it

*Done when the repo's existing instruction file names this document with a read-trigger, carries the maintenance rules, and every path in it resolves — or when there is no such file and you have said so.*

The document earns nothing if the next agent never opens it, so make the repo's agent instructions name it.

- **Update the instruction file the repo already has.** If `CLAUDE.md` is a symlink to `AGENTS.md` (`test -L`) or a short file importing it (`@./AGENTS.md`), write `AGENTS.md` only, and leave the symlink as it is.
- **Where neither file exists, leave it that way** and tell the user the document's path instead. Whether the repo wants an agent instruction file is their call, not a side effect of writing one document.
- **Keep a `## Documentation` section** holding one table row per document that exists, each naming *when* to read it rather than restating its title:

  ```markdown
  | Document | Read it when |
  |---|---|
  | `docs/requests-for-comments/` | a decision is still open and under review |
  ```

- **A row pointing at a directory relies on the filenames inside it.** `RFC-04-choose-realtime-transport.md` tells an agent listing the directory whether this is the record it needs; `RFC-04-proposal.md` makes it read all of them. The table gets it to the right directory, the filename gets it to the right file.
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
