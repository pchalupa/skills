---
name: changeset-authoring
description: >-
  Author or revise Changesets release entries with the affected packages,
  semantic-version bumps, user-facing summaries, and migration guidance. Use
  when a change needs a `.changeset/*.md` file or an existing changeset needs
  review. For general release notes or changelogs outside Changesets, do not use
  this skill.
---

# Changeset Authoring

A changeset is a release note with machine-readable package bumps. Write it for the package consumer: what changed, when they notice it, and what they must change. The diff already holds the implementation.

The target repository's release policy and examples take precedence. When it has no changeset template, use [`references/changeset-template.md`](references/changeset-template.md) as the output shape.

## 1. Read the change and release policy

_Done when every consumer-visible change, affected publishable package, and applicable release rule is accounted for._

- Confirm the repository uses Changesets by finding `.changeset/config.json`, a Changesets dependency, or an existing `.changeset/*.md` file. If it uses another release system, stop and name the mismatch.
- Read the repository instructions, `.changeset/README.md`, `.changeset/config.json`, package manifests, and a few recent changesets. Match local rules and house style over this skill.
- Read the diff and commits. Trace changed public behaviour, APIs, types, configuration, defaults, errors, and deprecations to the packages consumers install.
- Use exact manifest package names. Account for `fixed`, `linked`, `ignore`, private-package, and internal-dependency settings before adding related packages. Let Changesets calculate transitive dependent releases unless local policy says otherwise.
- If the change has no release impact, explain why and leave the repository unchanged. Create an empty changeset only when the repository's checks require one.

## 2. Choose package scope and bumps

_Done when each named package has the bump its consumers require, under the repository's policy._

- For a stable public API, use `patch` for a backward-compatible fix, `minor` for backward-compatible functionality or deprecation, and `major` for an incompatible change.
- Treat runtime behaviour and exported types as API. Choose from consumer impact, not diff size.
- Follow the repository's rule for `0.x` packages. SemVer does not define how their features and breaking changes map to bumps.
- Give the current change its correct bump even when another pending changeset already requests a higher one.
- One body is copied into every named package's changelog. Keep packages together only when the same summary is accurate and useful for all of them; otherwise write separate changesets.

## 3. Write the entry

_Done when every changeset is valid Markdown with resolved frontmatter and changelog-ready prose._

- Prefer the repository's Changesets command so package names and file structure come from the workspace. When direct authoring is more practical, add a descriptive kebab-case `.md` file under `.changeset/` and follow the applicable template.
- Open with one present-tense sentence that stands alone in a changelog. Name the public behaviour or API and the benefit or effect.
- Add context only when it helps a consumer decide whether or how to update. Keep implementation details out unless they are public API.
- For a breaking change or deprecation, state exactly what consumers must change. Add a short before-and-after example when prose alone leaves room for error.
- Use plain paragraphs by default. Follow local conventions before adding headings, lists, issue links, or contributor credits because changelog formatters render them differently.
- Remove every placeholder and authoring comment. An ordinary changeset has a non-empty summary.

For a required no-release entry, use the repository's empty-changeset command. Its file contains only:

```md
---
---
```

## 4. Verify

_Done when the repository accepts the files and every line is correct for each named package._

- Parse the frontmatter: each key is an exact package name and each value is the intended `patch`, `minor`, or `major` bump.
- Read each body as if it appeared unchanged in every named package's changelog. Split any entry that becomes vague, misleading, or package-irrelevant.
- Check that the first sentence stands alone and that every breaking change or deprecation has a concrete migration path.
- Run the repository's `changeset status` command. If the command is unavailable or fails for unrelated repository state, report that limitation instead of claiming validation passed.
- Report the files written, packages and bumps, and the validation result. Leave committing to the user.
