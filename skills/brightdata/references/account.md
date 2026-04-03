# Account, Budget, Zones & Configuration

Full reference for `brightdata budget`, `brightdata zones`, `brightdata config`, `brightdata skill`, `brightdata add mcp`, and `brightdata login/logout`.

---

## Budget

View account balance and per-zone cost and bandwidth usage. Read-only.

### budget (default)

```bash
brightdata budget                    # Quick balance view
```

### budget balance

```bash
brightdata budget balance            # Balance + pending charges
brightdata budget balance --pretty
```

### budget zones

```bash
brightdata budget zones              # Cost & bandwidth table for all zones
brightdata budget zones --from 2026-01-01T00:00:00 --to 2026-04-01T00:00:00
```

### budget zone \<name\>

```bash
brightdata budget zone my_unlocker_zone
brightdata budget zone cli_browser --pretty
```

| Flag | Type | Description |
|------|------|-------------|
| `--from <datetime>` | string | Start of date range (e.g. `2026-01-01T00:00:00`) |
| `--to <datetime>` | string | End of date range |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-k, --api-key <key>` | string | Override API key |

---

## Zones

List and inspect Bright Data proxy zones.

### zones

```bash
brightdata zones                     # List all active zones
brightdata zones --json -o zones.json
```

### zones info \<name\>

```bash
brightdata zones info my_unlocker_zone --pretty
brightdata zones info cli_browser
```

---

## Config

View and manage CLI configuration.

```bash
brightdata config                              # Show all config
brightdata config get <key>                    # Get a single value
brightdata config set <key> <value>            # Set a value
```

### Config Keys

| Key | Description |
|-----|-------------|
| `default_zone_unlocker` | Default zone for `scrape` and `search` |
| `default_zone_serp` | Default SERP zone (overrides unlocker zone for search) |
| `default_format` | Default output format: `markdown` or `json` |
| `api_url` | Override Bright Data API base URL |

```bash
brightdata config set default_zone_unlocker my_zone
brightdata config set default_format json
```

### Config Storage

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/brightdata-cli/` |
| Linux | `~/.config/brightdata-cli/` |
| Windows | `%APPDATA%\brightdata-cli\` |

Two files stored: `credentials.json` (API key) and `config.json` (zones, format, preferences).

**Priority order** (highest to lowest):
```
CLI flags → Environment variables → config.json → Defaults
```

---

## Login / Logout

```bash
brightdata login                      # Interactive — opens browser
brightdata login --api-key <key>      # Non-interactive
brightdata logout                     # Clear saved credentials
```

On first login the CLI checks for required zones (`cli_unlocker`, `cli_browser`) and creates them automatically if missing.

---

## Skill

Install Bright Data AI agent skills into coding agents (Claude Code, Cursor, Copilot, etc.).

```bash
brightdata skill add                  # Interactive picker
brightdata skill add <name>           # Install specific skill
brightdata skill list                 # List available skills
```

### Available Skills

| Skill | Description |
|-------|-------------|
| `search` | Search Google with structured JSON results |
| `scrape` | Scrape any webpage as markdown with bot bypass |
| `data-feeds` | Extract structured data from 40+ websites |
| `bright-data-mcp` | Orchestrate 60+ Bright Data MCP tools |
| `bright-data-best-practices` | Reference knowledge base for Bright Data code |

---

## Add MCP

Write a Bright Data MCP server entry into Claude Code, Cursor, or Codex config files.

```bash
brightdata add mcp                               # Interactive
brightdata add mcp --agent claude-code --global
brightdata add mcp --agent claude-code,cursor --project
brightdata add mcp --agent codex --global
```

| Flag | Type | Description |
|------|------|-------------|
| `--agent <agents>` | string | Comma-separated: `claude-code`, `cursor`, `codex` |
| `--global` | boolean | Install to global config |
| `--project` | boolean | Install to project config |

### Config Targets

| Agent | Global | Project |
|-------|--------|---------|
| Claude Code | `~/.claude.json` | `.claude/settings.json` |
| Cursor | `~/.cursor/mcp.json` | `.cursor/mcp.json` |
| Codex | `$CODEX_HOME/mcp.json` | Not supported |

Uses the API key stored by `brightdata login`. Does not read `BRIGHTDATA_API_KEY` env var, so log in first.

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `BRIGHTDATA_API_KEY` | API key (overrides stored credentials) |
| `BRIGHTDATA_UNLOCKER_ZONE` | Default Web Unlocker zone |
| `BRIGHTDATA_SERP_ZONE` | Default SERP zone |
| `BRIGHTDATA_POLLING_TIMEOUT` | Default polling timeout in seconds |
| `BRIGHTDATA_BROWSER_ZONE` | Default Scraping Browser zone (default: `cli_browser`) |
| `BRIGHTDATA_DAEMON_DIR` | Override directory for browser daemon socket/PID files |
