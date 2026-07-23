---
name: rfc-authoring
description: >-
  Author a new RFC (Request for Comments) for making a technical decision
  asynchronously with reviewers — grill the thinking to the ground first, then
  draft from the bundled template in the author's voice. Use when the user wants
  to write, author, or draft an RFC or Request for Comments, propose an open
  technical decision for review, or when they invoke an `/rfc` command with a
  decision topic. This is for decisions that are still open and need reviewer
  input; a decision already made belongs in an ADR instead.
---

# RFC Authoring

An RFC is how a decision gets made **asynchronously** — a written proposal that
seeks feedback, weighs alternatives, and drives to a merge. This is **human-led
authoring**: the user owns the understanding and the decision; you assist. **Do
not draft RFC prose until the thinking has been grilled to the ground.** Follow
the three phases in order.

The exact output shape lives in [`references/rfc-template.md`](references/rfc-template.md) —
read it before drafting. If the working repo has its own `templates/rfc.md`,
prefer that copy so the RFC stays in sync with local conventions; otherwise use
the bundled template.

## 1. Orient (do this quietly, then confirm with the user)

- Read the template (see above) — that is the exact shape of the output.
- Find where RFCs live (commonly `docs/rfcs/`). Scan for existing
  `RFC-NNNN-*.md` files and determine the **next sequential number**,
  zero-padded to four digits (e.g. `RFC-0004`). Numbers are monotonic and never
  reused. Tell the user the number you'll use.
- If a `README.md` (or equivalent) sits alongside the RFCs, read it for the
  project's lifecycle and conventions.
- Check whether this decision touches an existing dependency, risk, `ADR-NNNN`,
  or RFC — an RFC is for a decision still **open** that needs reviewer input. If
  it is really already decided, say so: it may belong in an ADR instead.

## 2. Grill relentlessly (before writing anything)

Interview the user one question at a time, giving your recommended answer each
time, and don't move on from a branch until it's resolved. If a `grilling` (or
`grill-me`) skill is available, invoke it to run this; otherwise conduct the
interview directly. Pin down specifically:

- **The decision & why now** — what exactly are reviewers being asked to decide,
  and what forces the decision now? What does it unblock?
- **Problem & constraints** — the situation today, stated as value-neutral
  facts. Whose problem is it, and for whom?
- **The proposal** — precise enough to act on. If it affects a system or API
  contract that other teams consume, pin the specifics: endpoints, auth/token
  flow, request/response fields & types, call sequences, error/edge-case
  behaviour, environments.
- **Alternatives** — what else is on the table and the concrete reason each
  loses. If there are no real alternatives, this may not warrant an RFC — say so.
- **Trade-offs, risks & impact** — what is accepted, what new risks appear, and
  how the change lands on the systems and teams affected.
- **Who reviews, and by when** — the reviewers, a realistic **Comment by** date,
  and a **Decision by** date.

If a question can be answered from the project's source materials or existing
deliverables, go read them instead of asking.

## 3. Draft — only after the user confirms the thinking is settled

- Write to `docs/rfcs/RFC-NNNN-<short-kebab-title>.md` (or the project's RFC
  location) using the template's sections exactly.
- **Apply the `voice-profile` skill** if available — this is prose authored on
  the user's behalf; it must read as theirs, human and consistent, not generic
  AI output.
- Write it to be **read by a reviewer skimming for the ask, then reading for the
  reasoning**: a Summary graspable in one pass, full sentences in the body,
  diagrams (Mermaid) where a flow or state machine needs one. Not a dump of
  fragments.
- **Keep it to 7 pages at most.** Following Miller's law, a reviewer holds only
  so much in working memory; a longer decision doc costs comprehension. If it
  won't fit, that's a signal the RFC is bundling more than one decision — split
  it or tighten the reasoning, don't shrink the type.
- **Link to documentation whenever possible** — a reviewer should be able to
  click through to the source rather than take a claim on faith. When the RFC
  names a technology, product, API, capability, or a sourced fact (pricing,
  retirement dates, version behaviour), link the official documentation the
  first time it appears.
- Status starts as **draft** (moves draft → ready-to-review → done; the
  accepted/rejected/withdrawn outcome goes in _Decision & outcome_, not the
  status). Created date is today. Fill Authors/Reviewers and the agreed dates.
- Leave **Decision & outcome** empty — it fills in when the RFC is closed.
  Review feedback itself is captured wherever the project collects it (e.g.
  inline comments), not in the file.
- This is a document reviewers will read: no authoring scaffolding. Resolve or
  drop `(illustrative)` and `> Expected here:` prompts. Defined cross-reference
  conventions (`ADR-NNNN`, risk/dependency IDs) are fine to keep.
- If drafting surfaces a gap the sources don't cover, log it wherever the
  project tracks open questions (append-only).
- Show the user the draft for review. Don't commit unless asked.
