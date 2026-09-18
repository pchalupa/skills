---
name: ticket-authoring
description: >-
  Author a self-contained ticket for any issue tracker — a bug report with
  reproduction steps, expected and actual result, and test environment, or a
  task with a description and acceptance criteria. Use when the user wants a
  ticket, issue, or backlog item written or cleaned up, or invokes `/ticket`.
---

# Ticket Authoring

A ticket is a handover. Whoever picks it up — a teammate tomorrow, the user in three months, an agent with no memory of this conversation — has only the ticket, so every ticket is **self-contained**: it carries the state to start from, the **literal** values to use, and the **observable** condition that says the work is done.

Two variants, and picking the wrong one is the most common failure. Ask whether the current behaviour is wrong or merely missing:

- **Bug** — wrong behaviour in something already built. Read [`references/bug-template.md`](references/bug-template.md).
- **Task** — behaviour that is missing and should exist. Read [`references/task-template.md`](references/task-template.md).

The templates are plain Markdown with no tracker-specific syntax, so the same ticket lands in Jira, Linear, GitHub Issues, or a Markdown file unchanged. Where the repo or the tracker has its own template, that one wins.

## 1. Ask the user for context

_Done when every field of the chosen template has an answer that came from the user or from something you read._

Read the template for the variant first — its fields are the questions. Then ask in one message, a short numbered list covering only the gaps, each with your best guess so the user corrects rather than composes.

- **Read before you ask.** The repo, the linked design, an ADR, an RFC, a Sentry issue, a sibling ticket — anything that already holds an answer.
- **Ask again when an answer is vague.** "It crashes sometimes" leaves the steps and the reproducibility number open. A second round is normal.
- **Ask which tracker and project** when the answer isn't already in the session.

## 2. Draft

_Done when every template section holds the user's specifics and no scaffolding prompt survives._

- Follow the template's sections in order. Drop one only when it genuinely does not apply.
- **One ticket, one outcome.** Two unrelated failures are two bugs; two deliverables are two tasks plus a parent. Split, cross-reference, and say so.
- **Literal.** The button label in quotes, the exact input, the exact account, the error text verbatim, the version as `4.12.0 (1180)`. Every reference names the thing it points at, so a reader never resolves "the usual test user" or "the settings thing".
- **Start from a known state.** Fresh install, logged out, seeded account — name it, so step 1 lands the same way every time.
- **Observable.** Expected result, actual result, and each acceptance criterion is a fact someone confirms true or false against the running product.
- **Written steps carry the failure.** A screenshot, recording, or log link goes in a comment as support, because nobody can grep an image.
- **Apply the Voice section below** — this is written on the user's behalf.

## 3. Trim to reading time

_Done when the ticket body is under 400 words and reads end to end in under three minutes._

- Reproduction steps: **three to eight**. Past eight, start from a closer known state.
- Acceptance criteria: **three to seven**. Past seven, split the task.
- Prose sections: **one short paragraph each**. Background that changes nothing about what someone builds becomes a link.
- Test environment: a table of values.

Still over? The ticket is too big — say so and propose the split.

## 4. Verify

_Done when every check passes._

- **The title alone triages it** — the area plus the broken behaviour or the wanted outcome.
- **The variant is right** — wrong behaviour is a bug, missing behaviour is a task.
- **A stranger reproduces it** — known starting state, one action per step, and the failure appears at the last step.
- **Expected and actual are two different observable facts.** "It doesn't work" is not the actual result.
- **The test environment is pinned** — platform and OS version, app version and build, environment (development, staging, production), reproducibility. A bug without a version is a bug nobody can close.
- **Acceptance criteria cover the error and empty states**, not only the happy path.
- **Every link resolves** and says what it shows.
- **Everything needed to start is in the ticket**, rather than in a screenshot or a chat thread.
- **Reproduction steps are actions, not diagnosis** — the theory of the cause belongs in the summary or a comment.

## 5. File it

_Done when the user has the draft, or the ticket exists in the tracker and you have given its ID and URL._

- Show the user the draft and get their go-ahead. Filing is outward-facing: it lands in someone's backlog.
- Create it with whatever this session offers for that tracker — a CLI skill, an MCP tool, a `gh`-style command.
- **Convert to the target field's format and keep the structure.** Markdown goes into Linear and GitHub Issues as it stands; a tracker with its own markup (Jira takes ADF or wiki markup) needs the headings, numbered steps, and test environment table preserved through the conversion. A table flattened into prose loses the test environment.
- Report the ticket ID and URL. Assignee, priority, sprint, and labels stay with the user unless they named them.

## Voice

Every sentence is authored on the user's behalf, so it reads as theirs:

- Friendly, confident, informal. Active voice, contractions, short sentences in paragraphs of two to four.
- Plain English a non-native speaker follows on the first read. Everyday words: "use" over "utilize", "start" over "commence", "important" over "load-bearing". Reach for a rarer word when it's a technical term or genuinely more precise, and define it on first use.
- Low jargon, concrete examples, and the why alongside the what.
- Lead with the point. The problem statement comes first, the background becomes a link.
- Keep it plain: no hype or superlatives, no filler openers, no hedging ("maybe consider possibly"), no clauses chained with dashes, no stacked emoji.
- A sentence that wouldn't change what the reader does comes out.
