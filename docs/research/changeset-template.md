# Research: Changeset template

_Researched on 2026-09-21._

## Recommendation

Add any future changeset-authoring skill under `skills/` with its reusable template in a `references/` directory. The template should be a small, valid Changesets-shaped document, while the skill should hold the decision rules and tell the author to prefer the target repository's own policy and template.

The portable template should look like this:

```md
---
"<exact package.json name>": <patch | minor | major>
---

<Present-tense, user-facing summary of what changed and who it affects.>

<Optional context: why it matters or when a consumer will notice it.>

<For a breaking change or deprecation, explain exactly what consumers must change. Add a short before/after example or diff when that is the clearest migration guide.>
```

This is authoring scaffolding, not text to copy unchanged. Every placeholder must be resolved or removed before the changeset is committed because the Markdown body becomes changelog content for each package named in the frontmatter ([Changesets FAQ](https://changesets.dev/faq#what-is-a-changeset)).

## Repository context

This repository is a private catalog of agent skills, not a versioned npm package: the root package is marked `private`, has no `version` or workspaces, and contains only formatting and spelling scripts ([`package.json`](../../package.json)). It also has no `.changeset/` setup. The requested template therefore fits the repository best as a future skill reference for use in other repositories, not as a release file for this repository.

That matches the established layout: authoring skills keep exact output shapes in `references/*-template.md`, and their instructions prefer a target repository's own template when one exists ([RFC authoring](../../skills/docs/rfc-authoring/SKILL.md), [RFC template](../../skills/docs/rfc-authoring/references/rfc-template.md), [pull-request authoring](../../skills/pull-request-authoring/SKILL.md)). The repository also separates reusable writing rules from the template itself ([voice profile](../../skills/voice-profile/SKILL.md)).

There is no existing research-note directory. This note uses `docs/research/`, which keeps temporary design research outside both installable skills and `.changeset/`. Changesets deletes consumed changeset files during versioning and explicitly warns against storing other information there ([Changesets FAQ](https://changesets.dev/faq#are-changesets-removed)).

## What the format requires

A changeset is a Markdown file in `.changeset/` with YAML frontmatter followed by a Markdown summary. The frontmatter maps package names to release bump types; the summary is later written to the changelog ([Changesets FAQ](https://changesets.dev/faq#what-is-a-changeset)).

The parser requires the file to start with `---`-delimited, valid YAML whose top level is an object. Package keys must be non-empty strings, and values must be one of `major`, `minor`, `patch`, or `none`; the body is trimmed into a summary, but a non-empty summary is not enforced by the parser ([parser source](https://github.com/changesets/changesets/blob/main/packages/parse/src/index.ts)). The contributor-facing template should expose only `patch`, `minor`, and `major`: those are the release choices described by the CLI and by Changesets' design, while `none` is an internal type and an empty changeset has its own supported command ([CLI reference](https://changesets.dev/guide/cli#add), [technical decisions](https://changesets.dev/guide/technical-decisions#how-changesets-differs-from-conventional-commit-based-tools)).

Use exact workspace package names. Syntax parsing only checks that a name is a non-empty string, but release-plan assembly rejects a changeset whose package is not in the workspace ([parser source](https://github.com/changesets/changesets/blob/main/packages/parse/src/index.ts), [release-plan source](https://github.com/changesets/changesets/blob/main/packages/assemble-release-plan/src/index.ts)). Generating the file with `pnpm changeset`, `npx @changesets/cli`, or `yarn changeset` is safer than hand-writing the frontmatter because the CLI prompts from the packages it finds; authors can edit the generated filename, package map, bump types, and Markdown afterward ([Changesets FAQ](https://changesets.dev/faq#how-do-i-add-a-changeset), [manual editing](https://changesets.dev/faq#can-i-manually-edit-a-changeset)).

## Choosing the bump

For a stable public API, SemVer defines `patch` for backward-compatible bug fixes, `minor` for backward-compatible functionality or deprecation, and `major` for incompatible public API changes ([Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html#summary)). Bump selection must be made from the consumer's point of view, including behavior and types, not from the size of the diff. The template can show the three values, but the skill should read the target repository's release policy before choosing one.

Do not hardcode a special rule for `0.x` packages. SemVer says the public API at `0.y.z` should not be considered stable, but it does not map features and breaking changes to specific bumps ([Semantic Versioning 2.0.0, rule 4](https://semver.org/spec/v2.0.0.html#spec-item-4)). Established repositories add their own mapping; for example, Backstage uses `minor` for a breaking `0.x` change and `patch` for a new `0.x` feature ([Backstage review policy](https://github.com/backstage/backstage/blob/master/REVIEWING.md#reviewing-changeset-bump-levels)). This is precisely the kind of local policy that should override a bundled default.

Changesets combines accumulated entries into one release per package at the highest requested bump, while preserving the individual changelog entries ([technical decisions](https://changesets.dev/guide/technical-decisions#how-changesets-are-combined)). Authors should still choose the correct bump for the change in front of them instead of lowering it because another changeset already requests a larger release.

## Package scope and file scope

One changeset can name several packages and assign a different bump to each ([Changesets FAQ](https://changesets.dev/faq#what-is-a-changeset)). Its one Markdown body is the summary for every named package, so a combined changeset works best when the same consumer-facing explanation is true for all of them ([Changesets type definition](https://github.com/changesets/changesets/blob/main/packages/types/src/index.ts), [Changesets dictionary](https://github.com/changesets/changesets/blob/main/docs/dictionary.md)). If packages need different explanations, create separate changesets. The official FAQ explicitly supports several changesets in one pull request for different package entries or separately notable changes ([Changesets FAQ](https://changesets.dev/faq#can-i-add-more-than-one-changeset-in-a-pr)); Backstage applies the same rule to avoid unrelated text being copied into multiple changelogs ([Backstage example](https://github.com/backstage/backstage/blob/master/REVIEWING.md#changeset-example)).

Do not mechanically list every transitive dependent. Changesets can calculate required dependent releases from the workspace graph and configuration, and its own example adds a patch bump to an internal dependent when a dependency leaves its accepted range ([technical decisions](https://changesets.dev/guide/technical-decisions#how-dependencies-are-bumped)). `fixed`, `linked`, `ignore`, internal-dependency, and private-package settings alter release planning, so the skill should inspect `.changeset/config.json` before finalizing package scope ([configuration reference](https://changesets.dev/guide/config)).

## Writing the summary

The first line should stand alone as a user-facing changelog entry: name the public API or behavior that changed and the effect a consumer will observe. Follow with context only when it helps someone decide whether or how to update. This implements the official guidance to cover what changed, why, and how consumers should update ([Changesets FAQ](https://changesets.dev/faq#how-do-i-add-a-changeset)). It also works with `@changesets/changelog-github`, whose `{summary}` token is the first line and whose formatter appends continuation lines separately ([formatter tokens](https://github.com/changesets/changesets/blob/main/packages/changelog-github/README.md#tokens)).

Keep implementation details out unless they are themselves public API. Backstage's maintainer guidance asks for user impact, concise migration help, and package-specific content rather than internal symbols or a copy of the diff ([Backstage review policy](https://github.com/backstage/backstage/blob/master/REVIEWING.md#reviewing-changeset-content)). Astro follows the same consumer-first approach: it recommends a present-tense opening, one line for many patches, names and benefits for new APIs, and explicit migration guidance for breaking changes ([Astro changeset guide](https://contribute.docs.astro.build/docs-for-code-changes/changesets/)).

The portable template should not require Markdown headings. Astro permits low-level headings for long entries, while Backstage bans headings for one generated changelog shape because they break the surrounding list structure ([Astro changeset guide](https://contribute.docs.astro.build/docs-for-code-changes/changesets/#using-markdown-section-headings), [Backstage UI format](https://github.com/backstage/backstage/blob/master/REVIEWING.md#backstage-ui-changeset-format)). Plain paragraphs and an optional short code example are the safest default; a target repository's convention can allow or require more structure.

## No-release changes

Not every change needs a changeset. Changesets recommends against blocking all contributions when there is no release to describe ([getting started](https://changesets.dev/guide/getting-started#usage)). When CI nevertheless requires a file, `changeset --empty` creates an empty changeset; the documented use is to satisfy such a merge check ([CLI reference](https://changesets.dev/guide/cli#empty-changesets)). Its file shape is:

```md
---
---
```

Treat this as a separate no-release path, not another option in the normal authoring template. It avoids inventing a changelog summary or misusing a package bump merely to satisfy automation.

## Checks a future skill should perform

Before handing off a changeset, the authoring workflow should verify:

- the target repository already uses Changesets and its own instructions or `.changeset/README.md` take precedence;
- every package key exactly matches a workspace package eligible for versioning;
- each bump follows that package's current version and repository policy;
- the same summary is appropriate for every package in the file, otherwise the entry is split;
- the first line makes sense on its own in each affected package's changelog;
- breaking changes and deprecations include concrete migration guidance;
- all authoring placeholders and comments are gone; and
- `changeset status` succeeds, since the CLI uses it to report current changesets and returns a failure when changed packages lack changesets ([CLI reference](https://changesets.dev/guide/cli#status)).

These checks belong in the skill instructions rather than the reference template. That keeps the committed file valid and short while preserving the repository's established separation between workflow rules and exact output shape.
