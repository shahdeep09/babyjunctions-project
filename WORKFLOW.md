# WORKFLOW.md — Baby Junctions
> 9-stage pipeline. What runs, when, triggered by what.

## SCHEDULE
| Time IST | Stage |
|---|---|
| 6:00 AM | Stage 1 — Topics |
| 6:30 AM | Stage 2 — Products |
| 7:00 AM | Stage 3 — Content + Images |
| 8AM–11PM (every 60 min) | Stage 4 — Publish 1 article |
| After each publish | Stage 6 — Social |
| 2:00 AM | Stage 5 — Prices |
| Friday 10AM | Stage 8 — Email |
| 1st of month | Stage 7 — SEO Loop |

## GITHUB ACTIONS (.github/workflows/daily-pipeline.yml)
```yaml
name: Baby Junctions Daily Pipeline
on:
  schedule:
    - cron: '30 0 * * *'    # 6AM IST — morning pipeline
    - cron: '30 20 * * *'   # 2AM IST — price updater
    - cron: '30 4 * * 5'    # Friday 10AM IST — email
    - cron: '30 0 1 * *'    # 1st of month — SEO loop
  workflow_dispatch:
jobs:
  daily_pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt
      - name: Run pipeline
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          AMAZON_ACCESS_KEY: ${{ secrets.AMAZON_ACCESS_KEY }}
          AMAZON_SECRET_KEY: ${{ secrets.AMAZON_SECRET_KEY }}
          AMAZON_ASSOCIATE_TAG: ${{ secrets.AMAZON_ASSOCIATE_TAG }}
          WP_SITE_URL: ${{ secrets.WP_SITE_URL }}
          WP_USERNAME: ${{ secrets.WP_USERNAME }}
          WP_APP_PASSWORD: ${{ secrets.WP_APP_PASSWORD }}
          GOOGLE_SERVICE_ACCOUNT_JSON: ${{ secrets.GOOGLE_SERVICE_ACCOUNT_JSON }}
          GOOGLE_SHEETS_ID: ${{ secrets.GOOGLE_SHEETS_ID }}
          GOOGLE_SHEETS_NAME: ${{ secrets.GOOGLE_SHEETS_NAME }}
          GOOGLE_DRIVE_ROOT_FOLDER_ID: ${{ secrets.GOOGLE_DRIVE_ROOT_FOLDER_ID }}
          INSTAGRAM_PAGE_TOKEN: ${{ secrets.INSTAGRAM_PAGE_TOKEN }}
          INSTAGRAM_USER_ID: ${{ secrets.INSTAGRAM_USER_ID }}
          PINTEREST_ACCESS_TOKEN: ${{ secrets.PINTEREST_ACCESS_TOKEN }}
          MAILCHIMP_API_KEY: ${{ secrets.MAILCHIMP_API_KEY }}
          MAILCHIMP_LIST_ID: ${{ secrets.MAILCHIMP_LIST_ID }}
          GA4_PROPERTY_ID: ${{ secrets.GA4_PROPERTY_ID }}
        run: python scripts/master_pipeline.py
```

## STAGE 1 — TOPIC RESEARCH
**Script:** stage1_topics.py | **Output:** 15 topics → Sheets Daily Topics tab + Drive /YYYY-MM-DD/topics.txt
```
1. Read Sheets → Published tab → build deduplication list
2. Read Sheets → Analytics Feed tab → HIGH priority insights
3. Detect: date, Indian season, month, upcoming festivals
4. Call Claude API → generate 15 topics (follows PROMPTS.md Stage 1)
5. Validate: no duplicates vs Published tab
6. Write to Sheets → Daily Topics (status=QUEUED)
7. Write to Drive → /YYYY-MM-DD/topics.txt
```
**Blog type mix:** 2×Top10 · 1×Budget · 1×Gift/Platform · 3×Buying · 2×Age/Seasonal · 2×Problem · 2×Info · 2×Flexible

## STAGE 2 — AMAZON PRODUCT RESEARCH
**Script:** stage2_products.py | **Output:** Products → Sheets Product Registry
```
1. Read Daily Topics (status=QUEUED) — all 15
2. For each topic:
   a. Call PA API SearchItems (keyword, SearchIndex=Baby, ItemCount=10)
   b. Filter: stars≥3.8, reviews≥100, availability=IN_STOCK
   c. Take top 5 passing products
   d. Build affiliate URL: amazon.in/dp/{ASIN}?tag={TAG}
   e. Write to Product Registry
   f. Update topic status=RESEARCHED
   g. Wait 1 second (PA API rate limit)
3. If <3 products pass: broaden search → if still fails: mark NEEDS_REVIEW, skip
```

## STAGE 3 — CONTENT GENERATION
**Script:** stage3_content.py | **Output:** Article JSON → Sheets Content Queue + Drive .docx
```
1. Read topics (status=RESEARCHED) + products from Registry
2. Read Published tab → last 50 slugs (internal linking)
3. For each topic:
   a. Call Claude API (PROMPTS.md Stage 3 system + user prompt)
   b. Validate JSON — retry once if invalid
   c. Self-check quality (PROMPTS.md self-check prompt)
   d. Fix if fails self-check
   e. Write to Content Queue (status=CONTENT_READY)
   f. Save .docx to Drive → /YYYY-MM-DD/articles/{slug}.docx
```

## STAGE 3B — IMAGE GENERATION
**Script:** stage3b_images.py | **Triggered:** After Stage 3
```
For each article: generate thumbnail(1200×628) + pinterest(1000×1500) + instagram(1080×1080)
Save to /tmp/images/{slug}/ — upload to WP in Stage 4
```

## STAGE 4 — WORDPRESS PUBLISHING
**Script:** stage4_publish.py | **Trigger:** Every 60 min 8AM–11PM | **Output:** Live article
```
1. Read Content Queue (status=CONTENT_READY) — take oldest 1
2. Upload images to WP Media Library
3. Render HTML from JSON (apply DESIGN.md components)
4. Inject schema markup JSON-LD
5. POST to WP REST API
6. Create Pretty Links for affiliate URLs
7. Update Sheets: Content Queue→PUBLISHED, Published tab→add row
8. Update Drive .docx: add published URL
9. Trigger Stage 6 with article details
```
**Publish slots:** 8AM · 9AM · 10AM · 11AM · 12PM · 1PM · 2PM · 3PM · 4PM · 5PM · 6PM · 7PM · 8PM · 9PM · 10PM

## STAGE 5 — NIGHTLY PRICE UPDATER
**Script:** stage5_prices.py | **Trigger:** 2AM daily
```
1. Read all Published articles → extract ASINs
2. Batch call PA API GetItems (10 ASINs/call, wait 1s between)
3. Compare prices → if changed: flag for WP update
4. If OOS: show badge, log to OOS Log tab
5. If OOS 7+ days: find replacement, update article
6. PATCH WP posts (price fields + content if OOS replaced)
7. Update Price History tab
```

## STAGE 6 — SOCIAL POSTING
**Script:** stage6_social.py | **Trigger:** After each Stage 4 publish

**Pinterest:**
```
Add UTM → Claude writes 200-char description → POST /v5/pins
Board mapped from subcategory (PINTEREST_BOARDS dict)
```
**Instagram:**
```
Use WP media URL → Claude writes caption with hashtags → 
POST /media → POST /media_publish
```
**WhatsApp Channel:**
```
Claude writes 300-char broadcast → send via API
All links include UTM parameters
```
Write all results to SM Performance tab.

## STAGE 7 — MONTHLY SEO LOOP
**Script:** stage7_seo.py | **Trigger:** 1st of month 6AM
```
1. Pull GSC data (last 28 days) → find pages pos 11-20, impressions>50
2. Top 20 near-miss articles
3. For each: scrape top 3 competitor headings → Claude gap analysis
4. Claude writes 300-500 words filling gaps → PATCH WP post
5. Write to SEO Loop Results tab
6. Update Analytics Feed tab with new insights
7. Save monthly report .docx to Drive /SEO-Reports/YYYY-MM/
```

## STAGE 8 — EMAIL DIGEST
**Script:** stage8_email.py | **Trigger:** Friday 10AM
```
1. Read top 3 articles this week (by word count)
2. Claude writes email (subject + preview + body HTML)
3. Create + send Mailchimp campaign
```

## STAGE 9 — LLM/GEO (baked into every article)
- Quick Answer box at top
- FAQ schema JSON-LD
- H2s question-formatted
- Comparison tables (LLMs extract easily)
- Specific data: prices, ratings, specs
- robots.txt: allow GPTBot, PerplexityBot, ClaudeBot

## MASTER PIPELINE SCRIPT
```python
# scripts/master_pipeline.py
import os
from datetime import datetime

def get_stage():
    now = datetime.now()
    h, d, wd = now.hour, now.day, now.weekday()
    if h == 2: return "stage5"
    if d == 1 and h == 6: return "stage7"
    if wd == 4 and h == 10: return "stage8"
    if 8 <= h <= 23: return "stage4"
    if h == 6: return "morning"
    return None

if __name__ == "__main__":
    stage = get_stage()
    if stage == "morning":
        from stage1_topics import run as s1
        from stage2_products import run as s2
        from stage3_content import run as s3
        from stage3b_images import run as s3b
        s1(); s2(); s3(); s3b()
    elif stage == "stage4":
        from stage4_publish import run; run()
    elif stage == "stage5":
        from stage5_prices import run; run()
    elif stage == "stage7":
        from stage7_seo import run; run()
    elif stage == "stage8":
        from stage8_email import run; run()
```

## BUILD ORDER (week by week)
```
Week 1: Sheets connection test + WP REST API test
Week 2: stage3 + stage3b + stage4 → manually feed 10 topics → test publish
Week 3: stage1 + stage2 → full pipeline test
Week 4: stage6 + stage5 → full daily automation
Month 2: stage7 + stage8 → intelligence loop
```

*v1.0 | April 2026*
