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

Available scopes: `jobs:read`, `jobs:write`, `chats:read`, `chats:write`, `academy:read`, `academy:write`

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
