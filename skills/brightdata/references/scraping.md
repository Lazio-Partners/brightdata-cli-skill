# Scraping & Search Commands

Full reference for `brightdata scrape`, `brightdata search`, and `brightdata status`.

---

## scrape

Scrape any URL using Bright Data's Web Unlocker. Handles CAPTCHAs, JavaScript rendering, and anti-bot protections automatically.

```bash
brightdata scrape <url> [options]
```

| Flag | Type | Description |
|------|------|-------------|
| `-f, --format <fmt>` | string | `markdown` / `html` / `screenshot` / `json` (default: markdown) |
| `--country <code>` | string | ISO country code for geo-targeting (e.g. `us`, `de`, `jp`) |
| `--zone <name>` | string | Web Unlocker zone name |
| `--mobile` | boolean | Use a mobile user agent |
| `--async` | boolean | Submit async job, return snapshot ID |
| `-o, --output <path>` | string | Write output to file |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-k, --api-key <key>` | string | Override API key |

### Examples

```bash
# Default markdown output
brightdata scrape https://news.ycombinator.com

# Raw HTML
brightdata scrape https://example.com -f html

# US geo-targeting, JSON to file
brightdata scrape https://amazon.com -f json --country us -o product.json

# Screenshot
brightdata scrape https://example.com -f screenshot -o page.png

# Mobile user agent
brightdata scrape https://example.com --mobile

# Async mode — returns snapshot ID
brightdata scrape https://example.com --async

# Pipe to markdown viewer
brightdata scrape https://docs.github.com | glow -

# Save to markdown file
brightdata scrape https://example.com -f markdown > page.md
```

### Output Formats

| Format | Description |
|--------|-------------|
| `markdown` | Clean markdown text (default) |
| `html` | Raw HTML source |
| `screenshot` | PNG screenshot of the page |
| `json` | Structured JSON with metadata |

---

## search

Search Google, Bing, or Yandex via Bright Data's SERP API. Google results include structured data: organic results, ads, people-also-ask, related searches.

```bash
brightdata search <query> [options]
```

| Flag | Type | Description |
|------|------|-------------|
| `--engine <name>` | string | `google` / `bing` / `yandex` (default: google) |
| `--country <code>` | string | Localized results (e.g. `us`, `de`) |
| `--language <code>` | string | Language code (e.g. `en`, `fr`) |
| `--page <n>` | number | Page number, 0-indexed (default: 0) |
| `--type <type>` | string | `web` / `news` / `images` / `shopping` (default: web) |
| `--device <type>` | string | `desktop` / `mobile` |
| `--zone <name>` | string | SERP zone name |
| `-o, --output <path>` | string | Write output to file |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-k, --api-key <key>` | string | Override API key |

### Examples

```bash
# Formatted table (default)
brightdata search "typescript best practices"

# German localized results
brightdata search "restaurants berlin" --country de --language de

# News search
brightdata search "AI regulation" --type news

# Shopping results
brightdata search "bluetooth headphones" --type shopping

# Image search
brightdata search "cute puppies" --type images

# Page 2 of results
brightdata search "web scraping" --page 1

# Bing search
brightdata search "bright data pricing" --engine bing

# Yandex search
brightdata search "web scraping" --engine yandex

# Mobile results
brightdata search "restaurants near me" --device mobile

# Extract URLs with jq
brightdata search "open source projects" --json | jq -r '.organic[].link'

# Get titles and URLs
brightdata search "AI news" --json | jq -r '.organic[] | "\(.title): \(.link)"'
```

### Google SERP JSON Structure

When using `--json`, Google search results include:

```json
{
  "organic": [
    {
      "title": "Result title",
      "link": "https://example.com",
      "description": "Snippet text",
      "position": 1
    }
  ],
  "ads": [...],
  "people_also_ask": [...],
  "related_searches": [...]
}
```

---

## status

Check or poll the status of an async snapshot or pipeline job.

```bash
brightdata status <job-id> [options]
```

| Flag | Type | Description |
|------|------|-------------|
| `--wait` | boolean | Poll until the job completes |
| `--timeout <seconds>` | number | Polling timeout (default: 600) |
| `-o, --output <path>` | string | Write output to file |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-k, --api-key <key>` | string | Override API key |

### Examples

```bash
# Check current status
brightdata status s_abc123xyz

# Block until complete
brightdata status s_abc123xyz --wait --pretty

# Custom timeout (5 minutes)
brightdata status s_abc123xyz --wait --timeout 300
```

---

## Pipe-Friendly Patterns

```bash
# Chain search → scrape
brightdata search "top open source projects" --json \
  | jq -r '.organic[0].link' \
  | xargs brightdata scrape

# Save scraped content
brightdata scrape https://example.com -f markdown > page.md

# Amazon product data as CSV
brightdata pipelines amazon_product "https://amazon.com/dp/xxx" --format csv > product.csv
```
