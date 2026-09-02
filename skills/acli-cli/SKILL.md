---
name: acli-cli
description: Atlassian CLI (acli) reference for working with Jira Cloud and Confluence Cloud from the command line — Jira tickets (work items), projects, sprints, boards, filters, plus Confluence spaces, blog posts, and page viewing. Use this skill whenever the user mentions acli, the Atlassian CLI, or asks to script anything against Jira or Confluence (creating tickets, JQL searches, bulk transitions, project management, Confluence space/blog operations, etc.) — even when they don't say "acli" by name.
---

# Atlassian CLI (acli)

Reference for `acli`, Atlassian's official CLI for Jira Cloud and Confluence Cloud.

**Version reference:** 1.3.18-stable (verify with `acli --version`).

**Scope:** Jira Cloud (work items, projects, sprints, boards, filters) and Confluence Cloud (spaces, blogs, page viewing). Org admin, Rovo Dev, custom fields, dashboards, and Bitbucket are out of scope here — for Bitbucket use `git` and the Bitbucket REST API.

## Mental model — what makes acli different from gh

A few things to keep in mind, because they trip people up coming from `gh`:

1. **Authentication is global, not per-host.** One `acli auth login` covers all your Atlassian sites and both products (Jira + Confluence). You switch between accounts/sites with `acli auth switch`, not by re-logging in.
2. **"Work item" is the new "issue."** Atlassian renamed Jira issues to work items in the CLI. Stories, tasks, bugs, epics — all are work items. The command is `acli jira workitem`, not `acli jira issue`.
3. **Bulk operations are first-class.** Most write commands (`edit`, `transition`, `assign`, `comment`, `delete`, `clone`) accept four targeting modes: a comma-separated list (`--key "K-1,K-2"`), a JQL query (`--jql "project = TEAM"`), a saved filter (`--filter 10001`), or a file (`--from-file issues.txt`). This means you can transition fifty work items in one command.
4. **JSON round-trip pattern.** For commands with many fields (create, edit, create-bulk, link create, project create, blog create), use `--generate-json` to scaffold an example payload, edit the file, then apply with `--from-json file.json`. This is the cleanest way to drive non-trivial creates and avoids juggling dozens of flags.
5. **Confluence support is asymmetric.** Spaces and blog posts have full CRUD, but `acli confluence page` only has `view` — there is no `acli confluence page create/edit/delete` today. For programmatic page management, use the Confluence REST API directly.

## Installation

```bash
# macOS
brew install --cask acli

# Verify
acli --version
```

For other platforms see https://developer.atlassian.com/cloud/acli/.

## Authentication

```bash
# Interactive global OAuth login (opens a browser)
acli auth login

# Status: which sites/accounts are authenticated
acli auth status

# Switch active account (interactive)
acli auth switch

# Switch to a specific site/email without prompts
acli auth switch --site mysite.atlassian.net --email user@example.com

# Logout from all accounts globally
acli auth logout
```

## CLI structure

```
acli
├── auth                             # Global OAuth (covers Jira + Confluence)
│   ├── login
│   ├── logout
│   ├── status
│   └── switch
├── jira
│   ├── workitem                     # Stories, tasks, bugs, epics
│   │   ├── archive / unarchive
│   │   ├── assign
│   │   ├── attachment {list,delete}
│   │   ├── clone
│   │   ├── comment {create,delete,list,update,visibility}
│   │   ├── create / create-bulk
│   │   ├── delete
│   │   ├── edit
│   │   ├── link {create,delete,list,type}
│   │   ├── search                   # JQL-driven
│   │   ├── transition
│   │   ├── view
│   │   └── watcher {list,remove}
│   ├── project {create,view,list,update,archive,restore,delete}
│   ├── sprint {view,list-workitems}
│   ├── board {search,get,list-sprints}
│   └── filter {list,get,search}
└── confluence
    ├── page {view}                  # view only — no create/edit yet
    ├── space {create,view,list,update,archive,restore}
    └── blog {create,view,list}
```

## Common flags

| Flag              | Notes                                                                                           |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `--json`          | Machine-readable output. Pipe to `jq` for queries.                                              |
| `--csv`           | CSV output (search, list, blog list).                                                           |
| `--web`           | Open the resource in a browser instead of printing.                                             |
| `--from-json F`   | Read full payload from `F`. Pair with `--generate-json` to scaffold.                            |
| `--generate-json` | Print an example JSON payload to stdout — pipe it to a file, edit, re-apply with `--from-json`. |
| `--from-file F`   | For `create`/`comment`: read summary/body from a file (plain text or ADF).                      |
| `--key`           | Targeting: comma-separated work item keys, e.g. `--key "TEAM-1,TEAM-2"`.                        |
| `--jql`           | Targeting: any JQL query, e.g. `--jql "project = TEAM AND status = 'To Do'"`.                   |
| `--filter`        | Targeting: a saved Jira filter ID.                                                              |
| `--yes` / `-y`    | Skip confirmation on bulk write operations. Use carefully.                                      |
| `--ignore-errors` | For bulk ops: continue past failures instead of aborting.                                       |
| `--paginate`      | Fetch all results across pages (search, filter list).                                           |
| `--limit N`       | Cap result count.                                                                               |

## Work items

Work items (formerly "issues") are the central resource. Most workflows route through `acli jira workitem`.

### Create

```bash
# Minimal: summary + project + type
acli jira workitem create \
  --project TEAM --type Task --summary "Wire up payments webhook"

# With description, assignee, labels, parent (epic link)
acli jira workitem create \
  --project TEAM --type Story \
  --summary "Migrate user table to UUIDs" \
  --description "Background and acceptance criteria..." \
  --assignee user@example.com \
  --label backend,migration \
  --parent TEAM-42

# Open an editor for summary + description (most ergonomic for prose)
acli jira workitem create --project TEAM --type Bug --editor

# Read summary + description from a file
acli jira workitem create \
  --project TEAM --type Task \
  --from-file ./bug-report.txt

# Use the JSON round-trip for full field control
acli jira workitem create --generate-json > workitem.json
$EDITOR workitem.json
acli jira workitem create --from-json workitem.json

# Self-assign
acli jira workitem create --project TEAM --type Task --summary "..." --assignee @me

# Output the new key as JSON (useful for scripting)
acli jira workitem create --project TEAM --type Task --summary "..." --json
```

Use `--editor` when writing real prose. Use `--from-json` when you need fields the flags don't expose (custom fields, components, fix versions, due date, priority).

### Bulk create

```bash
# Generate the example shape
acli jira workitem create-bulk --generate-json > issues.json

# Apply
acli jira workitem create-bulk --from-json issues.json

# Or from CSV (columns: summary, projectKey, issueType, description, label, parentIssueId, assignee)
acli jira workitem create-bulk --from-csv issues.csv

# Don't abort on the first failure
acli jira workitem create-bulk --from-json issues.json --ignore-errors --yes
```

### View

```bash
# Default fields
acli jira workitem view TEAM-123

# All fields as JSON
acli jira workitem view TEAM-123 --fields '*all' --json

# Just summary + comments
acli jira workitem view TEAM-123 --fields summary,comment --json

# All navigable fields except comment
acli jira workitem view TEAM-123 --fields '*navigable,-comment'

# Open in browser
acli jira workitem view TEAM-123 --web
```

### Search (JQL)

```bash
# Basic JQL
acli jira workitem search --jql "project = TEAM AND status = 'In Progress'"

# Just the count
acli jira workitem search --jql "project = TEAM AND assignee = currentUser()" --count

# Choose columns and emit CSV
acli jira workitem search \
  --jql "project = TEAM AND created >= -7d" \
  --fields "key,summary,assignee,status" \
  --csv

# Paginate through the entire result set
acli jira workitem search --jql "project = TEAM" --paginate --json

# Use a saved filter
acli jira workitem search --filter 10001 --json

# Open the search in the browser
acli jira workitem search --jql "project = TEAM AND priority = Highest" --web
```

JQL cheatsheet (most useful operators):

```
project = TEAM
status in ("To Do", "In Progress")
assignee = currentUser()        # or @me equivalent
assignee was currentUser()
labels in (backend, migration)
sprint in openSprints()
sprint in closedSprints()
"Epic Link" = TEAM-42
created >= -7d
updated >= startOfWeek()
resolved >= startOfMonth(-1) AND resolved < startOfMonth()
text ~ "webhook"                # full-text
ORDER BY priority DESC, created ASC
```

### Edit (single or bulk)

```bash
# Edit one work item
acli jira workitem edit --key TEAM-123 --summary "New summary"

# Edit many at once via JQL
acli jira workitem edit --jql "project = TEAM AND status = 'To Do'" \
  --assignee user@example.com --yes

# Replace labels (note: this overwrites — see remove/add below)
acli jira workitem edit --key TEAM-123 --labels "backend,urgent"

# Remove specific labels without touching others
acli jira workitem edit --key TEAM-123 --remove-labels "stale"

# Unassign
acli jira workitem edit --key TEAM-123 --remove-assignee

# Use JSON for any field the flags don't cover (priority, custom fields, due date)
acli jira workitem edit --generate-json > edit.json
acli jira workitem edit --key TEAM-123 --from-json edit.json
```

### Transition (status changes)

```bash
# One work item
acli jira workitem transition --key TEAM-123 --status "In Progress"

# Bulk by JQL
acli jira workitem transition \
  --jql "project = TEAM AND status = 'Code Review' AND updated < -3d" \
  --status "In Progress" --yes
```

The `--status` value must match a transition available on the work item's workflow (case-sensitive). If unsure, view the work item in the UI to see its current available transitions.

### Assign

```bash
# Self-assign
acli jira workitem assign --key TEAM-123 --assignee @me

# Assign by email
acli jira workitem assign --key TEAM-123 --assignee user@example.com

# Default project assignee
acli jira workitem assign --key TEAM-123 --assignee default

# Bulk reassignment via JQL
acli jira workitem assign \
  --jql "project = TEAM AND assignee = old.user@example.com" \
  --assignee new.user@example.com --yes
```

### Comments

```bash
# Add a comment to one work item
acli jira workitem comment create --key TEAM-123 --body "Looks good to me."

# From a file (supports plain text or Atlassian Document Format)
acli jira workitem comment create --key TEAM-123 --body-file ./review.txt

# Open editor
acli jira workitem comment create --key TEAM-123 --editor

# Edit your most recent comment instead of adding a new one
acli jira workitem comment create --key TEAM-123 --body "(updated)" --edit-last

# List comments
acli jira workitem comment list --key TEAM-123 --json

# Update or delete a specific comment
acli jira workitem comment update --key TEAM-123 --id 10001 --body "..."
acli jira workitem comment delete --key TEAM-123 --id 10001
```

### Links between work items

```bash
# See what link types exist on this site
acli jira workitem link type --json

# Create a "TEAM-123 blocks TEAM-456" link
acli jira workitem link create --out TEAM-123 --in TEAM-456 --type Blocks

# Bulk via JSON or CSV
acli jira workitem link create --generate-json > links.json
acli jira workitem link create --from-json links.json

# List existing links on an item
acli jira workitem link list --key TEAM-123 --json

# Remove a link
acli jira workitem link delete --key TEAM-123 --id <link-id>
```

### Other work item operations

```bash
# Clone (within project or to a different project)
acli jira workitem clone --key TEAM-123 --to-project NEW

# Delete (single, bulk, or by JQL)
acli jira workitem delete --key TEAM-123 --yes
acli jira workitem delete --jql "project = SCRATCH" --yes

# Archive / unarchive (recoverable, unlike delete)
acli jira workitem archive --key TEAM-123
acli jira workitem unarchive --key TEAM-123

# Attachments
acli jira workitem attachment list --key TEAM-123 --json
acli jira workitem attachment delete --key TEAM-123 --id <attachment-id>

# Watchers
acli jira workitem watcher list --key TEAM-123
acli jira workitem watcher remove --key TEAM-123 --account-id <id>
```

## Jira: projects

```bash
# List all visible projects
acli jira project list --json

# View one
acli jira project view --key TEAM --json

# Create from an existing project (clones config)
acli jira project create --from-project TEAM \
  --key NEWTEAM --name "New Team" \
  --description "..." --lead-email lead@example.com

# Create from JSON (for non-cloning creates)
acli jira project create --generate-json > project.json
acli jira project create --from-json project.json

# Update / archive / restore / delete
acli jira project update --key TEAM --name "Renamed Team"
acli jira project archive --key TEAM
acli jira project restore --key TEAM
acli jira project delete --key TEAM --yes
```

## Sprints, boards, filters (lookups for ticket workflows)

You'll often need a sprint, board, or saved filter to scope ticket queries. The relevant lookups:

```bash
# Find a board by name
acli jira board search --name "Team Alpha" --json

# View a board
acli jira board get --id 5 --json

# List the sprints on a board (useful for "what's the current sprint ID?")
acli jira board list-sprints --id 5 --json

# View a sprint
acli jira sprint view --id 37 --json

# List the work items in a sprint
acli jira sprint list-workitems --id 37 --fields "key,summary,status,assignee"

# Saved filters — use the returned ID with --filter on workitem search/edit/transition
acli jira filter list --json
acli jira filter get --id 10001 --json
acli jira filter search --name "release"
```

For creating or updating sprints, boards, or filters themselves, use the Jira UI or REST API — they're outside the ticket-management scope of this skill.

## Confluence

### Pages

Today the CLI exposes only `view` for pages. For programmatic page create/edit/delete, use the Confluence REST API directly.

```bash
acli confluence page view --id 123456789

# Specific body format (storage = Confluence XHTML, atlas_doc_format = ADF, view = rendered HTML)
acli confluence page view --id 123456789 --body-format storage

# Pull metadata along with the page
acli confluence page view --id 123456789 \
  --include-labels --include-version --include-direct-children --json
```

### Spaces

```bash
# List spaces
acli confluence space list --json
acli confluence space list --type personal           # global | personal
acli confluence space list --keys ENG,DOCS

# View
acli confluence space view --id 123456 --include-all --json

# Create
acli confluence space create --key ENG --name "Engineering" \
  --description "Engineering wiki"

# Private space
acli confluence space create --key SECRET --name "Secret" --private

# Update / archive / restore
acli confluence space update --key ENG --name "Engineering Hub"
acli confluence space archive --key OLD
acli confluence space restore --key OLD
```

### Blog posts

```bash
# List recent posts in a space
acli confluence blog list --space-id 12345 --json
acli confluence blog list --space-id 12345 --title "Release Notes"

# Create published post (storage format = Confluence XHTML)
acli confluence blog create --space-id 12345 \
  --title "v2.0 release notes" \
  --body "<p>What changed...</p>"

# Draft, or from a file
acli confluence blog create --space-id 12345 --title "WIP" --status draft \
  --from-file ./post.html

# Private published post
acli confluence blog create --space-id 12345 --title "Internal" --private \
  --body "<p>...</p>"

# View
acli confluence blog view --id 98765 --json
```

## Output handling

```bash
# JSON + jq for queries
acli jira workitem search --jql "project = TEAM" --json \
  | jq -r '.[] | "\(.key)\t\(.fields.status.name)\t\(.fields.summary)"'

# Just keys, one per line, for piping into other commands
acli jira workitem search --jql "project = TEAM AND assignee = currentUser()" --json \
  | jq -r '.[].key'

# CSV for spreadsheets / quick reports
acli jira workitem search --jql "project = TEAM AND created >= -30d" \
  --fields "key,summary,assignee,status,created" --csv > sprint-report.csv
```

## Common workflows

### Sweep stale "In Review" PRs back to "In Progress"

```bash
acli jira workitem transition \
  --jql 'project = TEAM AND status = "Code Review" AND updated < -7d' \
  --status "In Progress" --yes --ignore-errors
```

### Bulk move all open work items off a departing teammate

```bash
acli jira workitem assign \
  --jql 'assignee = leaving.user@example.com AND resolution = Unresolved' \
  --assignee new.owner@example.com --yes
```

### Create an epic + child stories from a template file

```bash
# 1. Create epic
EPIC=$(acli jira workitem create --project TEAM --type Epic \
  --summary "Auth migration" --json | jq -r '.key')

# 2. Bulk-create children with the epic as parent
acli jira workitem create-bulk --generate-json > children.json
# edit children.json: set "parentKey": "$EPIC" on each, set "issueType" to "Story"
sed -i '' "s/PARENT_PLACEHOLDER/$EPIC/g" children.json
acli jira workitem create-bulk --from-json children.json --yes
```

### Daily standup digest

```bash
acli jira workitem search \
  --jql 'assignee = currentUser() AND sprint in openSprints()' \
  --fields "key,status,summary" --json \
  | jq -r '.[] | "[\(.fields.status.name)] \(.key) — \(.fields.summary)"'
```

### Pull a Confluence page's storage format for diffing

```bash
acli confluence page view --id 123456789 --body-format storage --json \
  | jq -r '.body.storage.value' > page.html
```

## Help and feedback

```bash
acli --help                     # Top-level
acli jira workitem --help       # Any subcommand
acli jira workitem create --help

# Report a bug or send feedback to Atlassian
acli feedback
```

## References

- Official docs: https://developer.atlassian.com/cloud/acli/
- JQL reference: https://support.atlassian.com/jira-software-cloud/docs/use-advanced-search-with-jira-query-language-jql/
