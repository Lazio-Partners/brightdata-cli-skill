# Pipelines (Structured Data Extraction)

Full reference for `brightdata pipelines` — extract structured data from 40+ platforms using the Web Scraper API.

---

## Usage

```bash
brightdata pipelines <type> [params...] [options]
```

| Flag | Type | Description |
|------|------|-------------|
| `--format <fmt>` | string | `json` / `csv` / `ndjson` / `jsonl` (default: json) |
| `--timeout <seconds>` | number | Polling timeout (default: 600) |
| `-o, --output <path>` | string | Write output to file |
| `--json` | boolean | JSON output |
| `--pretty` | boolean | Indented JSON output |
| `-k, --api-key <key>` | string | Override API key |

## Listing Available Types

```bash
brightdata pipelines list
```

---

## Dataset Types by Category

### E-Commerce

| Type | Platform | Parameters |
|------|----------|------------|
| `amazon_product` | Amazon product page | `<url>` |
| `amazon_product_reviews` | Amazon reviews | `<url>` |
| `amazon_product_search` | Amazon search results | `<keyword> <url>` |
| `walmart_product` | Walmart product page | `<url>` |
| `walmart_seller` | Walmart seller profile | `<url>` |
| `ebay_product` | eBay listing | `<url>` |
| `bestbuy_products` | Best Buy | `<url>` |
| `etsy_products` | Etsy | `<url>` |
| `homedepot_products` | Home Depot | `<url>` |
| `zara_products` | Zara | `<url>` |
| `google_shopping` | Google Shopping | `<url>` |

### Professional Networks

| Type | Platform | Parameters |
|------|----------|------------|
| `linkedin_person_profile` | LinkedIn person | `<url>` |
| `linkedin_company_profile` | LinkedIn company | `<url>` |
| `linkedin_job_listings` | LinkedIn jobs | `<url>` |
| `linkedin_posts` | LinkedIn posts | `<url>` |
| `linkedin_people_search` | LinkedIn people search | `<url>` |
| `crunchbase_company` | Crunchbase | `<url>` |
| `zoominfo_company_profile` | ZoomInfo | `<url>` |

### Social Media

| Type | Platform | Parameters |
|------|----------|------------|
| `instagram_profiles` | Instagram profiles | `<url>` |
| `instagram_posts` | Instagram posts | `<url>` |
| `instagram_reels` | Instagram reels | `<url>` |
| `instagram_comments` | Instagram comments | `<url>` |
| `facebook_posts` | Facebook posts | `<url>` |
| `facebook_marketplace_listings` | Facebook Marketplace | `<url>` |
| `facebook_company_reviews` | Facebook reviews | `<url>` |
| `facebook_events` | Facebook events | `<url>` |
| `tiktok_profiles` | TikTok profiles | `<url>` |
| `tiktok_posts` | TikTok posts | `<url>` |
| `tiktok_shop` | TikTok shop | `<url>` |
| `tiktok_comments` | TikTok comments | `<url>` |
| `x_posts` | X (Twitter) posts | `<url>` |
| `youtube_profiles` | YouTube channels | `<url>` |
| `youtube_videos` | YouTube videos | `<url>` |
| `youtube_comments` | YouTube comments | `<url> <count>` |
| `reddit_posts` | Reddit posts | `<url>` |

### Other

| Type | Platform | Parameters |
|------|----------|------------|
| `google_maps_reviews` | Google Maps reviews | `<url> <count>` |
| `google_play_store` | Google Play | `<url>` |
| `apple_app_store` | Apple App Store | `<url>` |
| `reuter_news` | Reuters news | `<url>` |
| `github_repository_file` | GitHub repository files | `<url>` |
| `yahoo_finance_business` | Yahoo Finance | `<url>` |
| `zillow_properties_listing` | Zillow | `<url>` |
| `booking_hotel_listings` | Booking.com | `<url>` |

---

## Examples

### E-Commerce

```bash
# Amazon product details
brightdata pipelines amazon_product "https://amazon.com/dp/B09V3KXJPB" --pretty

# Amazon product as CSV
brightdata pipelines amazon_product "https://amazon.com/dp/B09V3KXJPB" --format csv -o product.csv

# Amazon reviews
brightdata pipelines amazon_product_reviews "https://amazon.com/dp/B09V3KXJPB" -o reviews.json

# Amazon search
brightdata pipelines amazon_product_search "laptop" "https://amazon.com"

# Walmart product
brightdata pipelines walmart_product "https://walmart.com/ip/123456"

# eBay listing
brightdata pipelines ebay_product "https://ebay.com/itm/123456"
```

### Professional Networks

```bash
# LinkedIn person profile
brightdata pipelines linkedin_person_profile "https://linkedin.com/in/username"

# LinkedIn company
brightdata pipelines linkedin_company_profile "https://linkedin.com/company/brightdata"

# LinkedIn job listings
brightdata pipelines linkedin_job_listings "https://linkedin.com/jobs/view/123456"

# Crunchbase company
brightdata pipelines crunchbase_company "https://crunchbase.com/organization/bright-data"
```

### Social Media

```bash
# Instagram profile
brightdata pipelines instagram_profiles "https://instagram.com/username"

# Instagram posts
brightdata pipelines instagram_posts "https://instagram.com/p/ABC123/"

# TikTok profile
brightdata pipelines tiktok_profiles "https://tiktok.com/@username"

# YouTube video details
brightdata pipelines youtube_videos "https://youtube.com/watch?v=dQw4w9WgXcQ"

# YouTube comments (top 50)
brightdata pipelines youtube_comments "https://youtube.com/watch?v=dQw4w9WgXcQ" 50

# Reddit post
brightdata pipelines reddit_posts "https://reddit.com/r/technology/comments/abc123/"

# X/Twitter posts
brightdata pipelines x_posts "https://x.com/username/status/123456"
```

### Other

```bash
# Google Maps reviews (last 7)
brightdata pipelines google_maps_reviews "https://maps.google.com/..." 7

# Google Play app
brightdata pipelines google_play_store "https://play.google.com/store/apps/details?id=com.example"

# Zillow listings
brightdata pipelines zillow_properties_listing "https://zillow.com/homedetails/..."

# Booking.com hotel
brightdata pipelines booking_hotel_listings "https://booking.com/hotel/..."

# Yahoo Finance
brightdata pipelines yahoo_finance_business "https://finance.yahoo.com/quote/AAPL"
```

---

## Output Formats

| Format | Description |
|--------|-------------|
| `json` | JSON array of records (default) |
| `csv` | Comma-separated values |
| `ndjson` | Newline-delimited JSON |
| `jsonl` | JSON Lines (same as ndjson) |

---

## Notes

- Pipeline jobs are **async** — the CLI triggers a collection job, then polls until results are ready.
- Default timeout is **600 seconds** (10 minutes). Increase with `--timeout` for large extractions.
- Some dataset types accept additional positional parameters (e.g., count for reviews/comments).
- Use `brightdata pipelines list` to see all available types in your terminal.
- Use `brightdata status <job-id>` to manually check status of a pipeline job.
