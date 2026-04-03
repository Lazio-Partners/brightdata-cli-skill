# Browser Commands

Full reference for `brightdata browser` — control a real browser session via Bright Data's Scraping Browser.

A lightweight local daemon holds the browser connection open between commands, giving persistent state without reconnecting on every call.

---

## Global Flags

These work with every browser subcommand:

| Flag | Type | Description |
|------|------|-------------|
| `--session <name>` | string | Session name for parallel isolation (default: `default`) |
| `--country <code>` | string | ISO country code for geo-targeting |
| `--zone <name>` | string | Scraping Browser zone (default: `cli_browser`) |
| `--timeout <ms>` | number | IPC command timeout (default: 30000) |
| `--idle-timeout <ms>` | number | Daemon auto-shutdown after idle (default: 600000 = 10 min) |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-o, --output <path>` | string | Write output to file |
| `-k, --api-key <key>` | string | Override API key |

---

## Navigation

### open \<url\>

Navigate to a URL. Starts the daemon and browser session automatically if not already running.

```bash
brightdata browser open https://example.com
brightdata browser open https://amazon.com --country us --session shop
```

| Flag | Description |
|------|-------------|
| `--country <code>` | Reconnects browser if country changes on existing session |
| `--zone <name>` | Browser zone name |
| `--idle-timeout <ms>` | Daemon idle timeout for this session |

### back

Navigate back in browser history.

```bash
brightdata browser back
```

### forward

Navigate forward in browser history.

```bash
brightdata browser forward
```

### reload

Reload the current page.

```bash
brightdata browser reload
```

---

## Reading Page Content

### snapshot

Capture the page as a text accessibility tree. The primary way AI agents read page content — far more token-efficient than raw HTML.

```bash
brightdata browser snapshot
brightdata browser snapshot --compact          # Interactive elements + ancestors only (70-90% fewer tokens)
brightdata browser snapshot --interactive      # Interactive elements as a flat list
brightdata browser snapshot --depth 3          # Limit tree depth
brightdata browser snapshot --selector "main"  # Scope to a CSS subtree
brightdata browser snapshot --wrap             # AI-safe content boundaries
```

| Flag | Type | Description |
|------|------|-------------|
| `--compact` | boolean | Only interactive elements and their ancestors |
| `--interactive` | boolean | Only interactive elements, flat list |
| `--depth <n>` | number | Limit tree depth |
| `--selector <sel>` | string | Scope to CSS selector |
| `--wrap` | boolean | Wrap in `--- BRIGHTDATA_BROWSER_CONTENT ---` boundaries |

**Output format:**
```
Page: Example Domain
URL: https://example.com

- heading "Example Domain" [level=1]
- paragraph "This domain is for use in illustrative examples."
- link "More information..." [ref=e1]
```

Each interactive element gets a `ref` (e.g. `e1`, `e2`) to pass to `click`, `type`, `fill`, etc.

### screenshot [path]

Capture a PNG screenshot.

```bash
brightdata browser screenshot
brightdata browser screenshot ./result.png
brightdata browser screenshot --full-page -o page.png
brightdata browser screenshot --base64
```

| Flag | Description |
|------|-------------|
| `[path]` | Where to save PNG (default: temp directory) |
| `--full-page` | Full scrollable page, not just viewport |
| `--base64` | Base64-encoded PNG data instead of file |

### get text [selector]

Get text content of the page or a scoped element.

```bash
brightdata browser get text           # Full page
brightdata browser get text "h1"      # First h1
brightdata browser get text "#price"  # Element by ID
```

### get html [selector]

Get HTML of the page or a scoped element.

```bash
brightdata browser get html              # Full page outer HTML
brightdata browser get html ".product"   # innerHTML of .product
```

---

## Interaction

### click \<ref\>

Click an element by its snapshot ref.

```bash
brightdata browser click e3
brightdata browser click e3 --session shop
```

### type \<ref\> \<text\>

Type text into an element. Clears the field first by default.

```bash
brightdata browser type e5 "search query"
brightdata browser type e5 " more text" --append   # Append to existing
brightdata browser type e5 "search query" --submit  # Press Enter after
```

| Flag | Description |
|------|-------------|
| `--append` | Append to existing value (key-by-key simulation) |
| `--submit` | Press Enter after typing |

### fill \<ref\> \<value\>

Fill a form field directly (no keyboard simulation). Use `type` if you need keydown/keyup events.

```bash
brightdata browser fill e2 "user@example.com"
```

### select \<ref\> \<value\>

Select a dropdown option by visible label.

```bash
brightdata browser select e4 "United States"
```

### check \<ref\> / uncheck \<ref\>

Check or uncheck a checkbox or radio button.

```bash
brightdata browser check e7
brightdata browser uncheck e7
```

### hover \<ref\>

Hover over an element (triggers hover states, tooltips, dropdowns).

```bash
brightdata browser hover e2
```

### scroll

Scroll the viewport or an element into view.

```bash
brightdata browser scroll                         # Down 300px (default)
brightdata browser scroll --direction up
brightdata browser scroll --direction down --distance 600
brightdata browser scroll --ref e10               # Scroll element into view
```

| Flag | Type | Description |
|------|------|-------------|
| `--direction <dir>` | string | `up` / `down` / `left` / `right` (default: down) |
| `--distance <px>` | number | Pixels to scroll (default: 300) |
| `--ref <ref>` | string | Scroll this element into view instead |

---

## Session & Network

### network

Show HTTP requests captured since last navigation.

```bash
brightdata browser network
brightdata browser network --json
```

### cookies

Show cookies for the active session.

```bash
brightdata browser cookies
brightdata browser cookies --pretty
```

### status

Show current state of a browser session.

```bash
brightdata browser status
brightdata browser status --session shop --pretty
```

### sessions

List all active browser daemon sessions.

```bash
brightdata browser sessions
brightdata browser sessions --pretty
```

### close

Close a session and stop its daemon.

```bash
brightdata browser close                    # Close default session
brightdata browser close --session shop     # Close named session
brightdata browser close --all              # Close all sessions
```

---

## Workflow Examples

### AI Agent Workflow

```bash
# Open a US-targeted session
brightdata browser open https://example.com --country us

# Read page structure (compact for token efficiency)
brightdata browser snapshot --compact

# Interact using refs
brightdata browser click e3
brightdata browser type e5 "search query" --submit

# Refresh snapshot after interaction (refs change!)
brightdata browser snapshot --compact

# Save screenshot for visual verification
brightdata browser screenshot ./result.png

# Done
brightdata browser close
```

### Multi-Session Comparison

```bash
# Open parallel sessions with different geo-targeting
brightdata browser open https://amazon.com --session us --country us
brightdata browser open https://amazon.com --session de --country de

# Compare snapshots
brightdata browser snapshot --session us --json > us.json
brightdata browser snapshot --session de --json > de.json

# Cleanup
brightdata browser close --all
```

### Form Submission

```bash
brightdata browser open https://example.com/login
brightdata browser snapshot --interactive

# Fill form fields
brightdata browser fill e2 "user@example.com"
brightdata browser fill e3 "password123"
brightdata browser click e4  # Submit button

# Wait for navigation, then read result
brightdata browser snapshot --compact
```

---

## Important Notes

- **Refs change on every snapshot.** After clicking or navigating, always take a fresh snapshot before using refs.
- **Sessions persist** until explicitly closed or the daemon idle-timeout (10 min default) expires.
- **Browser costs bandwidth.** Close sessions when done.
- **Use `--compact` for AI agents** — 70-90% fewer tokens than full snapshot.
