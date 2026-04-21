# Kola CLI — AI Agent Skill

Load this file into your AI agent to give it accurate knowledge of the `kola` CLI.

**Compatible with:** Claude (CLAUDE.md / claude-code), Cursor (`.cursorrules`), GitHub Copilot (`AGENTS.md`), Windsurf, and any agent that supports instruction files.

---

## What kola is

`kola` is the CLI for the Kolaborate freelance marketplace and academy. It provides programmatic access to jobs, courses, hackathons, internships, testing, and admin operations.

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

**Security gate**: `kola admin` commands always require Firebase human login. Agents with `KOLA_API_KEY` are rejected at the gate with exit code 2 even if they have `admin:read` scope.

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

Always use `--json` to get full Convex IDs. The table view shows truncated 8-char suffixes that cannot be used as input:

```bash
# CORRECT — get full _id field
kola admin academy hackathons list --json | jq '.[0]._id'

# WRONG — the truncated ID shown in table output is not usable
```

### Using @file syntax

Commands that accept a `--spec` or `--content` argument support `@path` to read from a file:

```bash
kola admin academy hackathons create --spec @./hackathon.json
kola academy course create --description "@./description.md"
```

### Dry-run before mutations

```bash
kola admin academy hackathons create --spec @./spec.json --dry-run
kola admin academy internships publish <id> --dry-run
```

---

## Command groups and common operations

### Jobs

```bash
kola jobs list --json
kola jobs list --status open --json
kola jobs get <id> --json
kola jobs create --title "..." --description "..." --budget 500
```

### Academy — user operations

```bash
kola academy list --json
kola academy enroll <course-id>
kola academy progress <course-id> --json
```

### Academy — instructor course management

```bash
# Create
kola academy course create --title "..." --level beginner --tags "ai,tools"

# Full one-shot workflow from a manifest file
kola academy course bootstrap ./manifest.json
kola academy course bootstrap ./manifest.json --resume   # after failure

# Publish
kola academy course publish --course <id>
```

### Testing & QA

```bash
kola testing projects list --json
kola testing runs list --project <id> --json
kola testing runs create --project <id> --name "Sprint 3"
kola testing runs start <run-id>
kola testing results update <result-id> --status pass   # pass|fail|skip|block
kola testing bugs create --run <run-id> --title "..."
kola testing bugs list --run <run-id> --status open --json
kola testing agent bulk-run --project <id> --template <template-id>
```

### Admin academy — hackathons

```bash
# List (always use --json for full IDs)
kola admin academy hackathons list --json

# Get single
kola admin academy hackathons get <full-convex-id> --json

# Create from file
kola admin academy hackathons create --spec @./hackathon.json

# Update fields
kola admin academy hackathons update <id> --banner <url>
kola admin academy hackathons update <id> --prize 250 --currency KLC
kola admin academy hackathons update <id> --status active   # upcoming|active|ended
kola admin academy hackathons update <id> --title "New Title"

# Lifecycle
kola admin academy hackathons status <id> --status active
kola admin academy hackathons delete <id>

# Engagement
kola admin academy hackathons participants <id> --json
kola admin academy hackathons submissions <id> --json
kola admin academy hackathons ideas generate <id>
kola admin academy hackathons qa list <id> --json
kola admin academy hackathons qa answer <question-id> --answer "..."
```

### Admin academy — courses

```bash
kola admin academy courses list --json
kola admin academy courses publish <id>
kola admin academy courses unpublish <id>
kola admin academy courses analytics <id> --json
kola admin academy courses enrollments <id> --json

# Modules
kola admin academy courses modules create <course-id> \
  --title "Module 1" --overview "..." --order 1 --objectives "obj1,obj2"
kola admin academy courses modules update <module-id> --title "..." --published true

# Lessons
kola admin academy courses lessons create <module-id> \
  --course-id <course-id> --title "Lesson 1" --order 1
kola admin academy courses lessons update <lesson-id> \
  --video-url "https://..." --provider mux --duration 300
```

> **Role constraint:** Course mutations (publish, create module, etc.) require `academyRole: instructor`. Admin-only role is not sufficient for course mutations.

### Admin academy — internships

```bash
kola admin academy internships list --json
kola admin academy internships create \
  --title "..." --description "..." \
  --company-id <id> --deadline 2026-06-01 \
  --requirements "Req 1, Req 2" --max-interns 10
kola admin academy internships publish <id>
kola admin academy internships close <id>
kola admin academy internships applications list <id> --status pending --json
kola admin academy internships applications update <app-id> \
  --status accepted   # accepted|approved|rejected
kola admin academy internships sessions create <id> \
  --title "Week 1" \
  --start 2026-05-05T09:00:00Z \
  --end 2026-05-05T10:00:00Z \
  --meeting-link "https://meet.google.com/..."
kola admin academy internships students list <id> --json
```

### Admin academy — waitlist

```bash
kola admin academy waitlist list --status pending --json
kola admin academy waitlist approve <entry-id>
kola admin academy waitlist reject <entry-id> --notes "Outside region"
kola admin academy waitlist stats --json
```

### Admin academy — stats

```bash
kola admin academy stats dashboard --json
kola admin academy stats hackathons --json
kola admin academy stats homepage --json
kola admin academy stats homepage update <key> --value 1240 --label "Enrollments"
```

---

## Hackathon spec reference

When calling `kola admin academy hackathons create --spec @file`, the JSON must include at minimum:

```json
{
  "title": "...",
  "slug": "unique-url-slug",
  "startDate": "2026-05-01T00:00:00.000Z",
  "endDate": "2026-05-07T23:59:59.000Z",
  "registrationDeadline": "2026-05-01T23:59:59.000Z",
  "organizer": "Kolaborate Academy",
  "status": "upcoming",
  "totalPrize": 250,
  "currency": "KLC",
  "isOnline": true,
  "prizes": [
    { "rank": "1st Place", "amount": 150, "description": "..." }
  ],
  "judges": [{ "name": "...", "title": "...", "company": "..." }],
  "timeline": [
    { "phase": "...", "startDate": "...", "endDate": "...", "description": "..." }
  ],
  "rules": "...",
  "submissionGuidelines": "...",
  "contactEmail": "...",
  "socialLinks": {}
}
```

**Do not include null values** for optional fields — omit them entirely. Convex rejects `null` for optional fields.

---

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | General error (bad args, not found, validation failed) |
| `2` | Auth error — agent blocked, not logged in, or insufficient trust level |
| `4` | Access denied — admin role required |

---

## Errors to watch for

| Error | Cause | Fix |
|---|---|---|
| `Academy admin commands require Firebase user login` | Using API key for admin academy command | Run `kola auth login` as a human user |
| `No updates specified` | `hackathons update` called with no flags | Pass at least one update flag |
| `Server Error` on update | Passing wrong arg shape to mutation | Check the `--help` for the command |
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

All CLI commands become callable MCP tools. The agent cannot call `kola admin` via MCP — those require human login.

---

## Links

- npm: https://www.npmjs.com/package/@kolaborate/kola-cli
- Platform: https://kolaborate.co
- Academy: https://academy.kolaborate.co
