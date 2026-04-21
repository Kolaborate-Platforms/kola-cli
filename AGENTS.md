# Kola CLI — AI Agent Skill

Load this file into your AI agent to give it accurate knowledge of the `kola` CLI.

**Compatible with:** Claude (CLAUDE.md / claude-code), Cursor (`.cursorrules`), GitHub Copilot (`AGENTS.md`), Windsurf, and any agent that supports instruction files.

---

## What kola is

`kola` is the CLI for the Kolaborate freelance marketplace and academy. It provides programmatic access to jobs, courses, hackathons, internships, testing, and more.

Install: `npm install -g @kolaborate/kola-cli`
Package: `@kolaborate/kola-cli`

---

## Authentication

### For AI agents (no browser)

```bash
export KOLA_API_KEY=kola_live_...
kola jobs list --json
```

Create a key for an agent: `kola auth apikey create --name "agent" --scopes "jobs:read"`

### For human-facing commands

```bash
kola auth login   # interactive — prompts for email and password
```

Credentials saved to `~/.kola/credentials.json`. Token auto-refreshes.

---

## Global flags (work on every command)

```
--json      Machine-readable JSON output
--dry-run   Preview mutations — no database writes
--quiet     Suppress non-error output
```

Always use `--json` when parsing output programmatically.

---

## Key patterns

### Reading IDs from list output

Always use `--json` to get full IDs. The table view shows truncated 8-char suffixes that cannot be used as input:

```bash
# CORRECT — get full _id field
kola academy list --json | jq '.[0]._id'

# WRONG — the truncated ID shown in table output is not usable
```

### Using @file syntax

Commands that accept a `--spec` or `--content` argument support `@path` to read from a file:

```bash
kola academy course create --description "@./description.md"
```

### Dry-run before mutations

```bash
kola academy course bootstrap ./manifest.json --dry-run
kola jobs create --title "..." --dry-run
```

---

## Command groups and common operations

### Jobs

```bash
kola jobs list --json
kola jobs list --status open --json
kola jobs get <id> --json
kola jobs create --title "..." --description "..." --budget 500
kola jobs update <id> --status closed
```

### Academy — browsing and enrollment

```bash
kola academy list --json
kola academy enroll <course-id>
kola academy progress <course-id> --json
```

### Academy — instructor course management

```bash
# Create a course
kola academy course create \
  --title "AI at Work" \
  --description "@./description.md" \
  --level beginner \
  --tags "ai,productivity"

# Full one-shot workflow from a manifest file
kola academy course bootstrap ./manifest.json
kola academy course bootstrap ./manifest.json --resume   # after failure

# Publish
kola academy course publish --course <id>

# Upload a story video
kola academy course video upload \
  --course <id> --file ./intro.mp4 --title "Intro"

# Set banner
kola academy course banner set --course <id> --file ./banner.png

# Generate AI outputs
kola academy course output generate \
  --course <id> \
  --type summary --type flashcards --type quiz \
  --watch
```

Bootstrap manifest shape:

```json
{
  "course": {
    "title": "AI at Work",
    "description": "@./description.md",
    "level": "beginner",
    "tags": ["ai", "productivity"],
    "dailyChatLimit": 50
  },
  "sources": [{ "title": "Module 1", "content": "@./01.md" }],
  "storyVideos": [{ "title": "Intro", "file": "@./intro.mp4" }],
  "banner": { "file": "@./banner.png" },
  "outputs": ["summary", "study_guide", "flashcards", "quiz", "faq"],
  "publish": true
}
```

`@path` reads content relative to the manifest file.

### Testing & QA

```bash
kola testing projects list --json
kola testing runs list --project <id> --json
kola testing runs create --project <id> --name "Sprint 3"
kola testing runs start <run-id>
kola testing results update <result-id> --status pass   # pass|fail|skip|block
kola testing bugs create --run <run-id> --title "..."
kola testing bugs list --run <run-id> --status open --json
kola testing sessions create --project <id> --title "Exploratory"
kola testing agent bulk-run --project <id> --template <template-id>
```

### Notifications

```bash
kola notifications list --unread --json
kola notifications mark-read <id>
kola notifications mark-read --all
```

### Support

```bash
kola support list --json
kola support create --subject "..." --message "..."
```

---

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General error (bad args, not found, validation failed) |
| `2` | Auth error — not logged in or insufficient credentials |

---

## Errors to watch for

| Error | Cause | Fix |
|---|---|---|
| `No updates specified` | Update command called with no flags | Pass at least one update flag |
| `Server Error` on mutation | Wrong arg shape | Check `--help` for the command |
| Truncated IDs fail | Using 8-char table ID as input | Re-run with `--json` and use the full `_id` value |

---

## MCP integration

To give an AI agent access to kola as MCP tools:

```json
{
  "mcpServers": {
    "kolaborate": {
      "command": "kola",
      "args": ["serve", "mcp"],
      "env": { "KOLA_API_KEY": "your_agent_api_key" }
    }
  }
}
```

---

## Links

- npm: https://www.npmjs.com/package/@kolaborate/kola-cli
- Platform: https://kolaborate.co
- Academy: https://academy.kolaborate.co
