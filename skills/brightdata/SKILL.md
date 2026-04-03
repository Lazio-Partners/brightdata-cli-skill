---
name: brightdata
description: "Use when the user wants to scrape walled-garden platforms that block normal scrapers: LinkedIn, Instagram, TikTok, Facebook, X/Twitter, YouTube, Crunchbase, Glassdoor, Indeed, Yelp, Zillow, or news sites behind paywalls (Reuters, BBC, CNN, Google News). Also use when the user explicitly mentions Bright Data or BrightData, or when Firecrawl has failed on a specific site and you need anti-bot bypass as a fallback. For general open-web scraping, use Firecrawl first — Bright Data is the fallback. For rich LinkedIn data, prefer Apify over Bright Data."
---

# Bright Data CLI

Scrape, search, and extract structured web data from the terminal using `brightdata` (alias: `bdata`). Handles CAPTCHAs, JavaScript rendering, anti-bot protections, and geo-targeting automatically.

## When to Use (and When Not To)

Bright Data is a **last-resort** scraping tool for general websites, but it is the **go-to tool for walled-garden platforms** that block normal scrapers entirely.

### Decision Order

1. **Firecrawl (self-hosted)** — Default for all general/open web scraping. Runs locally on ovhcloud (port 3002), no per-request cost.
2. **Apify** — Use for rich LinkedIn data when you need detailed enrichment.
3. **Bright Data** — Use in these cases:
   - **Walled-garden platforms** (see table below) — Firecrawl can't access these at all
   - **Web Unlocker** (`brightdata scrape`) — when Firecrawl fails on a specific site's anti-bot protections
   - **Cheap LinkedIn pulls** (`brightdata pipelines linkedin_*`) — basic profile/company data when Apify's richness isn't needed

### Walled-Garden Sites — Use Bright Data Pipelines

These platforms require authenticated/specialized scrapers that Firecrawl cannot handle. Bright Data pipelines are the right tool here.

| Platform | Pipeline Types | Use For |
|----------|---------------|---------|
| **linkedin.com** | `linkedin_person_profile`, `linkedin_company_profile`, `linkedin_job_listings`, `linkedin_posts`, `linkedin_people_search` | Basic profiles, company pages, job posts (use Apify for rich/enriched data) |
| **instagram.com** | `instagram_profiles`, `instagram_posts`, `instagram_reels`, `instagram_comments` | Profiles, posts, engagement data |
| **tiktok.com** | `tiktok_profiles`, `tiktok_posts`, `tiktok_shop`, `tiktok_comments` | Creator profiles, video data, shop listings |
| **facebook.com** | `facebook_posts`, `facebook_marketplace_listings`, `facebook_company_reviews`, `facebook_events` | Marketplace, reviews, events, posts |
| **x.com (Twitter)** | `x_posts` | Post data, engagement metrics |
| **youtube.com** | `youtube_profiles`, `youtube_videos`, `youtube_comments` | Channel data, video metadata, comments |
| **crunchbase.com** | `crunchbase_company` | Startup/company funding, investors, team |
| **glassdoor.com** | Check `brightdata pipelines list` | Company reviews, salary data |
| **indeed.com** | Check `brightdata pipelines list` | Job listings, company info |
| **yelp.com** | Check `brightdata pipelines list` | Business reviews, ratings |
| **zillow.com** | `zillow_properties_listing` | Property listings |
| **news sites** (Reuters, BBC, CNN, Google News) | `reuter_news` + scrape for others | News articles behind paywalls |

### Open Web — Use Firecrawl First

For everything else (blogs, docs, company websites, public pages, e-commerce product pages on open sites), use Firecrawl. Only fall back to Bright Data's Web Unlocker if Firecrawl fails.

## Prerequisites

Requires Node.js >= 20. Install globally:
```bash
npm install -g @brightdata/cli
```

Verify: `brightdata --version` or `bdata --version`

## Authentication

The API key is stored in `~/.brightdata/.env`. Source it before running commands:

```bash
source ~/.brightdata/.env
```

The env file contains `BRIGHTDATA_API_KEY=<key>`. The CLI reads this from the environment automatically. You can also pass `-k <key>` or `--api-key <key>` on any command.

To set up a new key:
```bash
mkdir -p ~/.brightdata
echo 'BRIGHTDATA_API_KEY=your-key-here' > ~/.brightdata/.env
chmod 600 ~/.brightdata/.env
```

Get your API key from: https://brightdata.com/cp/setting/users

On first login the CLI checks for required zones (`cli_unlocker`, `cli_browser`) and creates them automatically.

## Output Formats

All commands support these flags:

| Flag | Effect |
|------|--------|
| `--json` | Compact JSON to stdout |
| `--pretty` | Indented JSON to stdout |
| `-o <path>` | Write to file (format inferred from extension: .json, .md, .html, .csv) |

When stdout is not a TTY, colors and spinners are disabled automatically. Pipe-friendly by default.

## Core Workflows

### Scraping Any URL

Use `brightdata scrape` to extract content from any URL. Bypasses CAPTCHAs, JS rendering, and anti-bot protections via Web Unlocker.

```bash
# Scrape as markdown (default)
brightdata scrape https://news.ycombinator.com

# Scrape as raw HTML
brightdata scrape https://example.com -f html

# US geo-targeting, save to file
brightdata scrape https://amazon.com -f json --country us -o product.json

# Take a screenshot
brightdata scrape https://example.com -f screenshot -o page.png

# Async mode — returns a snapshot ID to poll with `status`
brightdata scrape https://example.com --async

# Use mobile user agent
brightdata scrape https://example.com --mobile
```

| Flag | Description |
|------|-------------|
| `-f, --format <fmt>` | `markdown` / `html` / `screenshot` / `json` (default: markdown) |
| `--country <code>` | ISO country code for geo-targeting (e.g. `us`, `de`, `jp`) |
| `--zone <name>` | Web Unlocker zone name |
| `--mobile` | Use a mobile user agent |
| `--async` | Submit async, return snapshot ID |
| `-o, --output <path>` | Write output to file |

### Searching Google / Bing / Yandex

Use `brightdata search` for structured SERP results. Google results include organic, ads, people-also-ask, and related searches.

```bash
# Default formatted table output
brightdata search "web scraping best practices"

# German localized results
brightdata search "restaurants berlin" --country de --language de

# News search
brightdata search "AI regulation" --type news

# Page 2 of results
brightdata search "web scraping" --page 1

# Bing search
brightdata search "bright data pricing" --engine bing

# Extract just URLs
brightdata search "open source projects" --json | jq -r '.organic[].link'
```

| Flag | Description |
|------|-------------|
| `--engine <name>` | `google` / `bing` / `yandex` (default: google) |
| `--country <code>` | Localized results |
| `--language <code>` | Language code (e.g. `en`, `fr`) |
| `--page <n>` | Page number, 0-indexed |
| `--type <type>` | `web` / `news` / `images` / `shopping` |
| `--device <type>` | `desktop` / `mobile` |
| `--zone <name>` | SERP zone name |

### Structured Data Extraction (Pipelines)

Extract structured data from 40+ platforms using the Web Scraper API. Triggers an async job, polls until ready, returns results.

```bash
# List all available dataset types
brightdata pipelines list

# LinkedIn profile
brightdata pipelines linkedin_person_profile "https://linkedin.com/in/username"

# Amazon product as CSV
brightdata pipelines amazon_product "https://amazon.com/dp/B09V3KXJPB" --format csv -o product.csv

# Instagram profile
brightdata pipelines instagram_profiles "https://instagram.com/username"

# Amazon search by keyword
brightdata pipelines amazon_product_search "laptop" "https://amazon.com"

# Google Maps reviews
brightdata pipelines google_maps_reviews "https://maps.google.com/..." 7

# YouTube comments (top 50)
brightdata pipelines youtube_comments "https://youtube.com/watch?v=..." 50
```

| Flag | Description |
|------|-------------|
| `--format <fmt>` | `json` / `csv` / `ndjson` / `jsonl` (default: json) |
| `--timeout <seconds>` | Polling timeout (default: 600) |
| `-o, --output <path>` | Write output to file |

For the full list of 40+ dataset types (Amazon, LinkedIn, Instagram, TikTok, YouTube, Reddit, etc.), see [pipelines.md](references/pipelines.md).

### Browser Automation

Control a real browser session via Bright Data's Scraping Browser. A local daemon holds the connection open between commands for persistent state.

```bash
# Open a URL (starts session automatically)
brightdata browser open https://example.com

# Read the page (accessibility tree with refs)
brightdata browser snapshot --compact

# Interact using refs from the snapshot
brightdata browser click e3
brightdata browser type e5 "search query" --submit

# Take a screenshot
brightdata browser screenshot ./result.png

# Close session
brightdata browser close
```

The browser supports named sessions for parallel work:
```bash
brightdata browser open https://amazon.com --session us --country us
brightdata browser open https://amazon.com --session de --country de
brightdata browser snapshot --session us
brightdata browser close --all
```

For the full browser command reference (snapshot modes, scroll, fill, select, cookies, network, etc.), see [browser.md](references/browser.md).

### Account & Zones

```bash
# Check account balance
brightdata budget
brightdata budget balance

# Zone costs and bandwidth
brightdata budget zones
brightdata budget zone my_zone --from 2026-01-01T00:00:00 --to 2026-04-01T00:00:00

# List proxy zones
brightdata zones
brightdata zones info cli_unlocker --pretty

# Check async job status
brightdata status <job-id> --wait --pretty
```

## Common Patterns

### Piping with jq
```bash
# Get all organic search URLs
brightdata search "nodejs tutorials" --json | jq -r '.organic[].link'

# Chain search → scrape
brightdata search "top open source projects" --json \
  | jq -r '.organic[0].link' \
  | xargs brightdata scrape
```

### Async Jobs
For large scraping or pipeline jobs, use async mode and poll:
```bash
brightdata scrape https://example.com --async
# Returns snapshot ID like s_abc123xyz

brightdata status s_abc123xyz --wait --timeout 300
```

### Geo-Targeting
All scrape/search/browser commands support `--country <code>`:
```bash
brightdata scrape https://amazon.com --country jp  # Japan
brightdata search "restaurants" --country de        # Germany
brightdata browser open https://example.com --country gb  # UK
```

## Command Quick Reference

| Group | Key Commands | Details |
|-------|-------------|---------|
| `scrape` | scrape any URL with anti-bot bypass | Above + [scraping.md](references/scraping.md) |
| `search` | Google/Bing/Yandex structured search | Above + [scraping.md](references/scraping.md) |
| `pipelines` | extract data from 40+ platforms | [pipelines.md](references/pipelines.md) |
| `browser` | control a real browser session | [browser.md](references/browser.md) |
| `budget` | balance, zone costs, bandwidth | [account.md](references/account.md) |
| `zones` | list/inspect proxy zones | [account.md](references/account.md) |
| `status` | poll async job status | [account.md](references/account.md) |
| `config` | CLI configuration | [account.md](references/account.md) |
| `skill` | install AI agent skills | [account.md](references/account.md) |
| `add mcp` | add MCP server to coding agents | [account.md](references/account.md) |

For full flag details on any command group, read the corresponding reference file.

## Safety Notes

- **Scraping costs credits.** Each scrape, search, or pipeline job consumes Bright Data credits. Use `brightdata budget` to monitor balance.
- **Pipeline jobs are async.** They poll until completion (default 600s timeout). Large extractions can take several minutes.
- **Browser sessions cost bandwidth.** Close sessions when done (`brightdata browser close --all`).
- **Geo-targeting affects pricing.** Some regions cost more than others.

## Troubleshooting

**"No Web Unlocker zone specified":** Run `brightdata config set default_zone_unlocker <zone>` or set `BRIGHTDATA_UNLOCKER_ZONE` env var.

**"Invalid or expired API key":** Run `brightdata login` or check `~/.brightdata/.env`.

**"Access denied":** Check zone permissions in the Bright Data control panel.

**"Rate limit exceeded":** Wait and retry. Use `--async` for large jobs.

**"No active browser session":** Start one first with `brightdata browser open <url>`.

**Element ref not found:** Refs change on every `snapshot`. After clicking or navigating, take a fresh snapshot before using new refs.
