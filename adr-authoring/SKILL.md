---
name: adr-authoring
description: >-
  Author a new Architecture Decision Record (ADR) that captures the reasoning
  behind a significant, expensive-to-reverse technical decision — grill the
  thinking to the ground first, then draft from the bundled template in the
  author's voice. Use when the user wants to write, author, or draft an ADR or
  Architecture Decision Record, record why a technical decision was made, or
  when they invoke an `/adr` command with a decision topic. This is for a
  decision already made and being recorded; an open decision that still needs
  reviewer input belongs in an RFC instead.
---

# ADR Authoring

An ADR captures the reasoning behind a significant technical decision — the
*why*, not just the *what*. Code shows what was built; the ADR explains why it
was built this way and what alternatives were rejected. This is the
highest-value documentation there is: a 10-minute ADR prevents a 2-hour debate
about the same decision six months later, and it stops future engineers (and
agents) from re-deciding something already settled.

ADRs are for decisions that are **expensive to reverse** — choosing a framework,
library, or major dependency; designing a data model; selecting an auth
strategy; picking an API architecture (REST vs. GraphQL vs. tRPC); or committing
to a build tool, hosting platform, or real-time transport. If the choice is
cheap to change or has no real alternatives, it probably doesn't warrant an ADR.

This is **human-led authoring**: the user owns the understanding and the
decision; you assist. **Do not draft ADR prose until the thinking has been
grilled to the ground.** Follow the phases in order.

The exact output shape lives in [`references/adr-template.md`](references/adr-template.md) —
read it before drafting. If the working repo has its own `templates/adr.md`,
prefer that copy so the ADR stays in sync with local conventions; otherwise use
the bundled template.

## 1. Orient (do this quietly, then confirm with the user)

- Read the template (see above) — that is the exact shape of the output.
- Find where ADRs live (commonly `docs/decisions/` or `docs/adr/`). Scan for
  existing `ADR-NNNN-*.md` files and determine the **next sequential number**,
  zero-padded to four digits (e.g. `ADR-0004`). Numbers are monotonic and never
  reused, even for superseded decisions. Tell the user the number you'll use.
- If a `README.md` (or equivalent) sits alongside the ADRs, read it for the
  project's conventions and any "likely decisions to capture" list.
- Check whether this decision touches an existing open question, dependency, or
  risk the project tracks — an ADR records a decision **already made**. If it is
  really still open and needs reviewer input, say so: it may belong in an RFC
  instead.

## 2. Grill relentlessly (before writing anything)

Interview the user one question at a time, giving your recommended answer each
time, and don't move on from a branch until it's resolved. If a `grilling` (or
`grill-me`) skill is available, invoke it to run this; otherwise conduct the
interview directly. Ground it in the Nygard ADR discipline
(https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) and
the community ADR hub
(https://github.com/architecture-decision-record/architecture-decision-record).
Pin down specifically:

- **Context / forces in tension** — the constraints, requirements, and competing
  forces. Keep these value-neutral: facts, not opinions. What is actually
  pushing on this decision? Which open question does it unblock?
- **The decision itself** — what exactly will be done, precise enough to act on.
- **Alternatives** — what else was on the table and the concrete reason each
  lost. If you can't name real alternatives, the decision may not be significant
  enough to warrant an ADR — say so.
- **Consequences** — the full picture: trade-offs being *accepted*, follow-on
  work, new risks, and the **integration impact** on downstream consumers of the
  system this decision touches.

If a question can be answered from the project's source materials or existing
deliverables, go read them instead of asking.

## 3. Draft — only after the user confirms the thinking is settled

- Write to `<adr-location>/ADR-NNNN-<short-kebab-title>.md` using the template's
  sections exactly: YAML frontmatter (`status`, `date`) plus Context, Decision,
  Alternatives considered, and Consequences. Make the `<short-kebab-title>` a
  present-tense imperative verb phrase — `choose-realtime-transport`, not
  `realtime-transport-decision` — so the filename reads as the decision itself.
- **One decision per ADR.** Each record captures a single architecture decision.
  If the grill surfaces a second decision riding along, split it into its own
  ADR (next number) and cross-reference — one ADR triggering follow-on ADRs is
  normal, not a smell.
- **Timestamp anything that will age.** Costs, quotas, SLAs, pricing tiers, and
  version numbers drift; note the date a fact was true as of, so a future reader
  knows whether to re-check it rather than trust it blindly.
- **Apply the `voice-profile` skill** if available — this is prose authored on
  the user's behalf; it must read as theirs, not generic AI output.
- Write it as **a conversation with a future developer**: full sentences in
  paragraphs, active voice ("We will …"), not fragments. Keep it to one or two
  pages.
- Set frontmatter `status: draft` and `date` to today; status moves
  **draft → ready-to-review → done**.
- **Never rewrite or delete a settled ADR to change a past decision.** If this
  decision reverses an earlier one, write it as a new ADR that references and
  supersedes the old one, and add a forward-pointing note
  (`Superseded by ADR-NNNN`) to the old ADR. Old ADRs stay in place — they
  capture historical context.
- This is a delivered document: no authoring scaffolding. Resolve or drop
  `(illustrative)` and `> Expected here:` prompts. Defined cross-reference
  conventions (`ADR-NNNN`, risk/dependency IDs) are fine to keep.
- If drafting surfaces a gap the sources don't cover, log it wherever the
  project tracks open questions (append-only).
- Show the user the draft for review. Don't commit unless asked.

## 4. Verify before handing back

- **Context is value-neutral** — facts and forces, not opinions or the
  conclusion in disguise.
- **The decision is actionable** — someone could implement it from the ADR alone.
- **One decision only** — a second decision hiding inside gets split into its own
  record.
- **At least two real alternatives** are named, each with the concrete reason it
  lost. If you can't, flag that this may not merit an ADR.
- **Consequences are honest** — the trade-offs being *accepted*, not just the
  upsides, plus follow-on work, new risks, and integration impact on downstream
  consumers.
- **No red flags**: no decision left without written rationale, no restating
  what the code does instead of why, no leftover scaffolding.
