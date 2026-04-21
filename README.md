# kola-cli

Official CLI and AI agent skill for the [Kolaborate](https://kolaborate.co) platform.

```bash
npm install -g @kolaborate/kola-cli
```

---

## What is Kola CLI?

`kola` is the command-line interface for the Kolaborate freelance marketplace and academy platform. It gives humans and AI agents full programmatic access to:

- **Marketplace** — jobs, quotations, kolaborator profiles, chats
- **Academy** — courses, hackathons, internships, waitlist, stats
- **Testing & QA** — test runs, cases, bugs, assignments, sessions
- **Admin** — employees, orders, leave, requisitions, audit logs, and more

Every command supports `--json` for machine-readable output and `--dry-run` to preview mutations before they execute.

---

## Installation

```bash
# npm (Node ≥ 18)
npm install -g @kolaborate/kola-cli

# npx (no install)
npx @kolaborate/kola-cli --help
```

---

## Authentication

### Human users

```bash
kola auth login
```

Interactive email/password login. Saves a Firebase JWT to `~/.kola/credentials.json`. Token refresh is automatic.

### Agents / CI / Headless

```bash
export KOLA_API_KEY=kola_live_...
kola jobs list --json
```

Create an API key with:

```bash
kola auth apikey create --name "my-agent" --scopes "jobs:read,academy:read"
```

Scopes: `jobs:read`, `jobs:write`, `chats:read`, `chats:write`, `academy:read`, `academy:write`, `admin:read`, `admin:write`

> **Note:** Agents are blocked from `kola admin` commands by the security gate regardless of scopes. Admin operations require Firebase human login.

---

## Global Flags

These work on every command:

| Flag | Description |
|---|---|
| `--json` | Output as machine-readable JSON |
| `--dry-run` | Preview mutations — no writes to the database |
| `--quiet` | Suppress non-error output (useful in CI pipelines) |

---

## Command Reference

### `kola auth`

```bash
kola auth login                    # interactive login
kola auth logout                   # clear saved credentials
kola auth status                   # show current actor (human or agent)
kola auth apikey create \
  --name "ci" \
  --scopes "jobs:read,academy:read"  # create a scoped agent API key
kola auth apikey list
kola auth apikey revoke <key-id>
```

### `kola jobs`

```bash
kola jobs list                        # browse open jobs
kola jobs list --status open --json   # filter + JSON output
kola jobs get <id>
kola jobs create --title "..." --description "..." --budget 500
kola jobs update <id> --status closed
```

### `kola academy`

```bash
kola academy list                     # browse published courses
kola academy enroll <course-id>       # enroll in a course
kola academy progress <course-id>     # check your progress

# Instructor — create and publish courses
kola academy course create \
  --title "AI at Work" \
  --description "@./description.md" \
  --level beginner \
  --tags "ai,productivity"

kola academy course publish --course <id>

# One-shot bootstrap (create + sources + videos + outputs + publish)
kola academy course bootstrap ./manifest.json
kola academy course bootstrap ./manifest.json --resume   # resume after failure
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
  "sources": [
    { "title": "Module 1", "content": "@./01.md" }
  ],
  "storyVideos": [
    { "title": "Intro", "file": "@./intro.mp4" }
  ],
  "banner": { "file": "@./banner.png" },
  "outputs": ["summary", "study_guide", "flashcards", "quiz", "faq"],
  "publish": true
}
```

`@path` reads file content relative to the manifest.

### `kola testing`

```bash
kola testing projects list
kola testing runs list --project <id> --json
kola testing runs create --project <id> --name "Sprint 3"
kola testing runs start <run-id>
kola testing results update <result-id> --status pass
kola testing bugs create --run <run-id> --title "Login fails on Safari"
kola testing bugs list --run <run-id> --status open --json
kola testing sessions create --project <id> --title "Exploratory"
kola testing agent bulk-run --project <id> --template <template-id>
```

### `kola admin` (human login + admin role required)

```bash
# Employees
kola admin employees list --json
kola admin employees get <id>

# Leave
kola admin leave list --status pending
kola admin leave approve <request-id>

# Requisitions (4-step workflow)
kola admin requisitions list --stage prepare
kola admin requisitions authorize <id>

# Orders
kola admin orders list --status pending
kola admin orders update <id> --status fulfilled

# Audit log
kola admin audit list --actor <user-id> --json

# Users
kola admin users list --role instructor --json
```

### `kola admin academy` (requires `academyRole: instructor | admin`)

Full CLI parity with the Academy admin web panel.

#### Courses

```bash
kola admin academy courses list --json
kola admin academy courses get <id>
kola admin academy courses publish <id>
kola admin academy courses unpublish <id>
kola admin academy courses duplicate <id> --title "Copy of Course"
kola admin academy courses analytics <id> --json
kola admin academy courses enrollments <id> --json

# Modules
kola admin academy courses modules list <course-id>
kola admin academy courses modules create <course-id> \
  --title "Module 1" --overview "..." --order 1
kola admin academy courses modules update <module-id> --title "New Title"
kola admin academy courses modules delete <module-id>

# Lessons
kola admin academy courses lessons list <course-id>
kola admin academy courses lessons create <module-id> \
  --course-id <course-id> --title "Lesson 1" --order 1
kola admin academy courses lessons update <lesson-id> \
  --video-url "https://..." --provider mux --duration 300
```

#### Hackathons

```bash
# List and inspect
kola admin academy hackathons list --json
kola admin academy hackathons list --status upcoming
kola admin academy hackathons get <id>

# Create from spec file or inline JSON
kola admin academy hackathons create --spec @./hackathon.json
kola admin academy hackathons create --spec '{"title":"...","slug":"...","startDate":"...","endDate":"...",...}'

# Update individual fields
kola admin academy hackathons update <id> --title "New Title"
kola admin academy hackathons update <id> --banner <image-url>
kola admin academy hackathons update <id> --prize 250 --currency KLC
kola admin academy hackathons update <id> --status active
kola admin academy hackathons update <id> --start 2026-05-01 --end 2026-05-07

# Lifecycle
kola admin academy hackathons status <id> --status active
kola admin academy hackathons delete <id> --dry-run

# Participants and submissions
kola admin academy hackathons participants <id> --json
kola admin academy hackathons submissions <id> --json

# Ideas and Q&A
kola admin academy hackathons ideas list <id>
kola admin academy hackathons ideas generate <id>
kola admin academy hackathons qa list <id>
kola admin academy hackathons qa answer <question-id> --answer "..."
```

Hackathon spec file shape:

```json
{
  "title": "The Build Challenge",
  "shortDescription": "One-line teaser",
  "description": "Full description...",
  "organizer": "Kolaborate Academy",
  "status": "upcoming",
  "category": "AI & Productivity",
  "tags": ["AI", "builders"],
  "startDate": "2026-05-01T00:00:00.000Z",
  "endDate": "2026-05-07T23:59:59.000Z",
  "registrationDeadline": "2026-05-01T23:59:59.000Z",
  "totalPrize": 250,
  "currency": "KLC",
  "isOnline": true,
  "maxParticipants": 100,
  "requirements": ["..."],
  "prizes": [
    { "rank": "1st Place", "amount": 150, "description": "..." },
    { "rank": "2nd Place", "amount": 70,  "description": "..." },
    { "rank": "3rd Place", "amount": 30,  "description": "..." }
  ],
  "judges": [{ "name": "...", "title": "...", "company": "..." }],
  "timeline": [
    { "phase": "Build Sprint", "startDate": "...", "endDate": "...", "description": "..." }
  ],
  "rules": "...",
  "submissionGuidelines": "...",
  "contactEmail": "academy@kolaborate.co",
  "socialLinks": {},
  "slug": "unique-slug-here"
}
```

#### Internships

```bash
kola admin academy internships list --json
kola admin academy internships create \
  --title "..." --description "..." \
  --company-id <id> --deadline 2026-06-01 \
  --requirements "Requirement 1, Requirement 2"
kola admin academy internships publish <id>
kola admin academy internships close <id>
kola admin academy internships applications list <id> --status pending
kola admin academy internships applications update <app-id> --status accepted
kola admin academy internships sessions create <id> \
  --title "Week 1 Kickoff" --start 2026-05-05T09:00:00Z --end 2026-05-05T10:00:00Z
kola admin academy internships activities create <id> \
  --title "Project Brief" --type assignment --due 2026-05-10
kola admin academy internships students list <id>
```

#### Waitlist

```bash
kola admin academy waitlist list
kola admin academy waitlist list --status pending --json
kola admin academy waitlist approve <entry-id>
kola admin academy waitlist reject <entry-id> --notes "Outside target region"
kola admin academy waitlist stats --json
```

#### Stats

```bash
kola admin academy stats dashboard --json
kola admin academy stats hackathons --json
kola admin academy stats homepage --json
kola admin academy stats homepage update enrollments --value 1240 --label "Total Enrollments"
```

### `kola notifications`

```bash
kola notifications list
kola notifications list --unread --json
kola notifications mark-read <id>
kola notifications mark-read --all
```

### `kola support`

```bash
kola support list
kola support get <ticket-id>
kola support create --subject "..." --message "..."
```

---

## MCP Setup for AI Agents

`kola serve mcp` exposes all CLI commands as MCP tools, letting AI agents like Claude call them natively.

### Claude Desktop / Claude Code

```json
{
  "mcpServers": {
    "kolaborate": {
      "command": "kola",
      "args": ["serve", "mcp"],
      "env": {
        "KOLA_API_KEY": "your_agent_api_key_here"
      }
    }
  }
}
```

### Hosted MCP endpoint

```json
{
  "mcpServers": {
    "kolaborate": {
      "url": "https://mcp.kolaborate.com/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_KOLA_API_KEY"
      }
    }
  }
}
```

---

## Environment Variables

| Variable | Description |
|---|---|
| `KOLA_API_KEY` | Agent API key for headless / CI authentication |
| `KOLA_CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name for image/video uploads |
| `KOLA_CLOUDINARY_UPLOAD_PRESET` | Cloudinary unsigned upload preset |
| `KOLA_CLOUDINARY_UPLOAD_FOLDER` | Upload destination folder |

---

## AI Agent Skill

See [`AGENTS.md`](./AGENTS.md) for a structured skill file you can load into any AI agent (Claude, Cursor, Copilot, etc.) to give it accurate, up-to-date knowledge of the kola CLI.

---

## Links

- **npm package**: [npmjs.com/package/@kolaborate/kola-cli](https://www.npmjs.com/package/@kolaborate/kola-cli)
- **Platform**: [kolaborate.co](https://kolaborate.co)
- **Academy**: [academy.kolaborate.co](https://academy.kolaborate.co)
- **Issues**: [github.com/Kolaborate-Platforms/kola-cli/issues](https://github.com/Kolaborate-Platforms/kola-cli/issues)

---

## License

MIT
