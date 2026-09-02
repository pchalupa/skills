# Skills

My personal [agent skills](https://code.claude.com/docs/en/skills). Each skill is a folder with a
`SKILL.md` that an agent loads on demand.

## Skills

| Skill                        | What it does                                                   |
| ---------------------------- | -------------------------------------------------------------- |
| `adr-authoring`              | Record a settled technical decision as an ADR.                 |
| `rfc-authoring`              | Propose an open technical decision for async review.           |
| `raid-logging`               | Track project risks, assumptions, issues, and dependencies.    |
| `architecture-diagramming`   | Draw C4 architecture diagrams in Mermaid.                      |
| `acli-cli`                   | Drive Jira and Confluence from the Atlassian CLI.              |
| `react-native-accessibility` | Write accessible React Native UI and tests that query by role. |
| `voice-profile`              | Write text in my voice.                                        |

## Install

### Skills CLI

Works with any agent the [skills CLI](https://github.com/vercel-labs/skills) supports.

```sh
# Pick skills interactively
npx skills add pchalupa/skills

# Or install one directly
npx skills add pchalupa/skills --skill adr-authoring
```

Add `--global` to install for every project instead of the current one.

### Claude Code plugin

The documentation skills also ship as a Claude Code plugin.

```sh
/plugin marketplace add pchalupa/skills
/plugin install Documentation@pchalupa-skills
```

## License

MIT
