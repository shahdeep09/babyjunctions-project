# WORKFLOW.md — Baby Junctions
## Complete 9-Stage Pipeline — What Runs, When, Triggered By What

> This file is the master operating manual for the entire automation system.
> Every stage, every timing, every trigger, every handoff is defined here.
> GitHub Actions reads this logic. Python scripts implement it.

---

## 1. THE BIG PICTURE

```
GitHub Actions triggers at 6:00 AM IST every day
            ↓
Stage 1: Research 15 topics (reads Google Sheets + Analytics Feed)
            ↓
Stage 2: Fetch Amazon products for all 15 topics
            ↓
Stage 3: Claude writes all 15 articles (JSON format)
            ↓
Stage 3B: Generate images for all 15 articles
            ↓
Stage 4: Publish 1 article per hour (from 8 AM to 11 PM)
            ↓  (after each publish, triggers:)
Stage 6: Post to Instagram + Pinterest + WhatsApp
            ↓
Stage 5: Nightly price + stock update at 2:00 AM
            ↓
Stage 7: Monthly SEO loop on 1st of each month
            ↓
Stage 8: Email digest every Friday at 10:00 AM
```

---

## 2. GITHUB ACTIONS — SCHEDULE FILE

Save as: `.github/workflows/daily-pipeline.yml`

```yaml
name: Baby Junctions Daily Pipeline

on:
  schedule:
    # 6:00 AM IST = 00:30 UTC
    - cron: '30 0 * * *'

    # Stage 5 — Nightly price update: 2:00 AM IST = 20:30 UTC previous day
    - cron: '30 20 * * *'

    # Stage 8 — Weekly email: Friday 10:00 AM IST = Friday 04:30 UTC
    - cron: '30 4 * * 5'

    # Stage 7 — Monthly SEO: 1st of month 6:00 AM IST = 1st 00:30 UTC
    - cron: '30 0 1 * *'

  workflow_dispatch:  # Allow manual trigger from GitHub UI

jobs:
  daily_pipeline:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install -r requirements.txt

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
          INSTAGRAM_PAGE_TOKEN: ${{ secrets.INSTAGRAM_PAGE_TOKEN }}
          INSTAGRAM_USER_ID: ${{ secrets.INSTAGRAM_USER_ID }}
          PINTEREST_ACCESS_TOKEN: ${{ secrets.PINTEREST_ACCESS_TOKEN }}
          MAILCHIMP_API_KEY: ${{ secrets.MAILCHIMP_API_KEY }}
          MAILCHIMP_LIST_ID: ${{ secrets.MAILCHIMP_LIST_ID }}
          GA4_PROPERTY_ID: ${{ secrets.GA4_PROPERTY_ID }}
          GOOGLE_SHEETS_ID: ${{ secrets.GOOGLE_SHEETS_ID }}
        run: python scripts/master_pipeline.py
```

---

## 3. STAGE 1 — TOPIC RESEARCH

**Trigger:** 6:00 AM IST daily (GitHub Actions cron)
**Script:** `scripts/stage1_topics.py`
**Runtime:** ~5-8 minutes
**Output:** 15 topics written to Google Sheets → Daily Topics tab

### What It Does

```
1. Read Google Sheets → Published tab
   → Extract all published titles + primary keywords
   → Build deduplication list

2. Read Google Sheets → Analytics Feed tab
   → Filter: priority = HIGH, valid_until >= today
   → Extract action_for_stage1 for each insight
   → Pass to Claude as context

3. Detect today's context:
   → Current date, month, Indian season
   → Upcoming festivals (next 30 days)
   → Seasonal buying angles

4. Call Claude API with:
   → Deduplication list (what NOT to repeat)
   → Analytics Feed insights (what worked before)
   → Today's seasonal context
   → Blog type distribution rules
   → CONTENT.md voice + standard

5. Claude returns 15 topics as JSON

6. Validate: no duplicate titles or keywords vs Published tab

7. Write to Google Sheets → Daily Topics tab
   → All 15 topics with status = QUEUED

8. Write to Google Drive → /YYYY-MM-DD/topics.txt
   → Full topic list with reasoning

9. Log to Google Drive → /System/pipeline-status.txt
   → "Stage 1 complete: 15 topics generated"
```

### Blog Type Distribution (Daily)
```
4 Transactional:
  → 2 × Top 10 List
  → 1 × Budget Guide
  → 1 × Gift Guide OR Platform Comparison

3 Buying Guides

2 Age-Based OR Seasonal (based on current season/festivals)

2 Problem-Solution

2 Informational (Myth Buster / Safety Alert / Scientific)

2 Flexible (Claude decides based on Analytics Feed insights)
```

### Deduplication Rules
- Never generate a topic with the same primary keyword as any published article
- Never generate a topic with title similarity > 70% to any published article
- Never cover the same subcategory more than 2× in the same day's 15 topics
- If Analytics Feed says "avoid X subcategory this month", skip it

---

## 4. STAGE 2 — AMAZON PRODUCT RESEARCH

**Trigger:** 6:30 AM IST (30 mins after Stage 1)
**Script:** `scripts/stage2_products.py`
**Runtime:** ~10-15 minutes (rate limited — 1 API call per second)
**Output:** Products written to Google Sheets → Product Registry tab

### What It Does

```
1. Read Google Sheets → Daily Topics tab
   → Filter: status = QUEUED
   → Get all 15 topics

2. For each topic:
   a. Call Amazon PA API → SearchItems
      → Keyword: topic's primary_keyword
      → SearchIndex: Baby
      → ItemCount: 10
      → Resources: Title, Price, Images, Reviews, Rating, Availability

   b. Filter results:
      → star_rating >= 3.8
      → review_count >= 100
      → availability = IN_STOCK

   c. Sort by: review_count DESC (most reviewed first)

   d. Take top 5 products that pass filters

   e. Build affiliate URL:
      https://www.amazon.in/dp/{ASIN}?tag={ASSOCIATE_TAG}

   f. Write to Google Sheets → Product Registry tab

   g. Update Daily Topics tab: status = RESEARCHED

   h. Wait 1 second (PA API rate limit)

3. If fewer than 3 products pass filters for a topic:
   → Broaden search (remove price constraint)
   → If still < 3 products: flag topic as NEEDS_REVIEW in Sheets
   → Skip that topic for today — don't write a thin article

4. Log: "Stage 2 complete: X/15 topics have sufficient products"
```

### PA API Request Format
```python
{
  "Keywords": topic["primary_keyword"],
  "SearchIndex": "Baby",
  "ItemCount": 10,
  "PartnerTag": AMAZON_ASSOCIATE_TAG,
  "PartnerType": "Associates",
  "Marketplace": "www.amazon.in",
  "Resources": [
    "ItemInfo.Title",
    "Offers.Listings.Price",
    "CustomerReviews.StarRating",
    "CustomerReviews.Count",
    "Images.Primary.Large",
    "ItemInfo.Features",
    "BrowseNodeInfo.BrowseNodes",
    "Offers.Listings.Availability.Type"
  ]
}
```

---

## 5. STAGE 3 — CONTENT GENERATION

**Trigger:** 7:00 AM IST (30 mins after Stage 2)
**Script:** `scripts/stage3_content.py`
**Runtime:** ~20-30 minutes (15 articles × ~2 mins each)
**Output:** Article JSON saved to Google Sheets → Content Queue tab + Google Drive

### What It Does

```
1. Read Google Sheets → Daily Topics tab
   → Filter: status = RESEARCHED
   → Get all topics with products ready

2. Read Google Sheets → Published tab
   → Get last 50 article titles + slugs (for internal linking)

3. For each topic:

   a. Load matching products from Product Registry tab

   b. Determine blog type → load correct JSON schema template from CONTENT.md

   c. Call Claude API:
      → System prompt: full CONTENT.md brand voice + structure rules
      → User message: topic + products + blog type + internal link candidates
      → Output format: strict JSON only — no preamble, no markdown
      → Max tokens: 4000

   d. Validate JSON response:
      → Check all required fields present
      → Check word count within target range
      → If invalid: retry once with error correction prompt

   e. Self-check (Claude scores its own output):
      → Is the intro India-specific?
      → Are prices in INR?
      → Does it pass the content quality checklist?
      → If fails: Claude revises

   f. Save to Google Sheets → Content Queue tab

   g. Save full article Word doc to Google Drive:
      → /YYYY-MM-DD/articles/{slug}.docx
      → Includes: title, URL (pending), keyword, full content

   h. Update Daily Topics tab: status = CONTENT_READY

4. Log: "Stage 3 complete: X articles written"
```

---

## 6. STAGE 3B — IMAGE GENERATION

**Trigger:** Immediately after Stage 3 completes
**Script:** `scripts/stage3b_images.py`
**Runtime:** ~5 minutes (all 15 articles)
**Output:** 3 images per article (45 images total) saved temporarily

### What It Does

```
For each article in Content Queue:

1. Generate Featured Thumbnail (1200×628px)
   → Background: #E8F4FD
   → Article title text overlay
   → Category badge
   → Baby Junctions logo strip
   → Product image (if available from Amazon CDN)

2. Generate Pinterest Pin (1000×1500px)
   → Tall format
   → Article title + price hook
   → "India ke parents ka guide" tagline
   → Baby Junctions branding

3. Generate Instagram Square (1080×1080px)
   → Product highlight
   → Price badge
   → CTA: "Link in bio"
   → Baby Junctions branding

4. Save images temporarily to /tmp/images/{slug}/
   → thumbnail.png
   → pinterest.png
   → instagram.png

5. Update Content Queue tab: image_paths filled in
```

---

## 7. STAGE 4 — WORDPRESS AUTO-PUBLISHING

**Trigger:** Every 60 minutes from 8:00 AM to 11:00 PM IST
**Script:** `scripts/stage4_publish.py`
**Runtime:** ~2-3 minutes per article
**Output:** Article live on babyjunctions.in

### What It Does

```
Every 60 minutes:

1. Read Google Sheets → Content Queue tab
   → Filter: status = READY
   → Take exactly 1 article (the oldest READY article)
   → If none ready: skip this slot silently

2. Upload images to WordPress Media Library:
   → Upload thumbnail.png → get media_id
   → Upload pinterest.png (stored for Stage 6)
   → Upload instagram.png (stored for Stage 6)

3. Render HTML from JSON:
   → Apply DESIGN.md component templates
   → Product cards with stars, reviews, price, buy button
   → Quick Answer box
   → Seasonal banner (if applicable)
   → FAQ section
   → Science section
   → Internal links embedded
   → Schema markup JSON-LD injected
   → Affiliate disclosure at footer

4. Build post payload (see WORDPRESS.md Section 9)

5. POST to WordPress REST API:
   → Status: publish
   → Set all custom fields (ACF)
   → Set Rank Math SEO fields
   → Set featured image (thumbnail media_id)
   → Set categories + tags

6. Receive WordPress post_id + published URL

7. Create Pretty Link for each product's affiliate URL

8. Update Google Sheets:
   → Content Queue tab: status = PUBLISHED
   → Published tab: add new row with all details
   → Daily Topics tab: status = PUBLISHED

9. Update Google Drive Word doc:
   → Add Published URL to the article's .docx file

10. Trigger Stage 6 (social posting) with:
    → article_title, article_url, slug
    → pinterest_image_path, instagram_image_path
    → primary_keyword, subcategory

11. Log: "Published: {title} at {time} → {url}"
```

### Publish Schedule
```
8:00 AM  → Article 1
9:00 AM  → Article 2
10:00 AM → Article 3
11:00 AM → Article 4
12:00 PM → Article 5
1:00 PM  → Article 6
2:00 PM  → Article 7
3:00 PM  → Article 8
4:00 PM  → Article 9
5:00 PM  → Article 10
6:00 PM  → Article 11
7:00 PM  → Article 12
8:00 PM  → Article 13
9:00 PM  → Article 14
10:00 PM → Article 15
```

---

## 8. STAGE 5 — NIGHTLY PRICE + STOCK UPDATER

**Trigger:** 2:00 AM IST daily
**Script:** `scripts/stage5_prices.py`
**Runtime:** ~20-30 minutes
**Output:** All live articles have fresh prices. OOS products flagged.

### What It Does

```
1. Read Google Sheets → Published tab
   → Get all live articles (status = LIVE)
   → Extract all ASINs from Product Registry

2. Batch ASINs (10 per API call — PA API limit)

3. For each batch:
   a. Call PA API → GetItems
      → Fetch: current Price, Availability, Rating

   b. Compare new price vs stored price in Product Registry

   c. If price changed:
      → Update Product Registry: current_price_inr
      → Add row to Price History tab
      → Flag WordPress post for price update

   d. If availability = OUT_OF_STOCK:
      → Update Product Registry: availability = OOS, oos_since = today
      → Add row to OOS Log tab
      → Flag WordPress post for OOS badge

   e. Wait 1 second between API calls

4. For each flagged WordPress post:
   a. Fetch post content via REST API
   b. Update price in HTML (string replace)
   c. If OOS: add .bj-oos-badge visible class
   d. Update "Price last checked" date
   e. PATCH post via REST API (content + meta fields only)

5. For products OOS 7+ consecutive days:
   a. Call PA API SearchItems for same keyword
   b. Filter: rating >= 3.8, reviews >= 100, IN_STOCK
   c. Call Claude API to write replacement product paragraph
   d. Replace OOS product section in WordPress post
   e. Update OOS Log: action_taken = REPLACED

6. Log summary: "Updated X prices, Y OOS detected, Z replaced"
```

---

## 9. STAGE 6 — SOCIAL MEDIA POSTING

**Trigger:** After each Stage 4 publish (webhook/callback)
**Script:** `scripts/stage6_social.py`
**Runtime:** ~1-2 minutes per article
**Output:** Post live on Instagram, Pinterest, WhatsApp Channel

### Pinterest

```
1. Receive: article_url, title, pinterest_image_path, subcategory

2. Add UTM to article_url:
   ?utm_source=pinterest&utm_medium=pin&utm_campaign=YYYY-MM-DD&utm_content={slug}

3. Call Claude API to generate pin description:
   → 200 chars
   → Include primary keyword
   → Include price hook if transactional
   → Include India context
   → End with: "Save for later 📌"

4. POST to Pinterest API → /v5/pins:
   → board: mapped from subcategory (50 boards)
   → title: article SEO title
   → description: Claude output
   → link: article URL with UTM
   → media: upload pinterest.png

5. Write to Google Sheets → SM Performance tab:
   → platform = PINTEREST, post_type = PIN, status = POSTED
```

### Instagram

```
1. Receive: article_url, title, instagram_image_path, subcategory

2. Add UTM to article_url:
   ?utm_source=instagram&utm_medium=post&utm_campaign=YYYY-MM-DD&utm_content={slug}

3. Call Claude API to generate caption:
   → Match blog type:
     Transactional: "₹X for [product] on Amazon.in 🛒 Worth it? Here's our honest review 👇 Link in bio"
     Informational: "5 things parents miss when buying [product] 👇 Full guide — link in bio"
     Safety: "⚠️ Stop buying this for your newborn — link in bio to see why"
   → Add hashtags: #babygear #indianmom #babyproductsindia #newborn + 5 product-specific tags
   → End with a question to drive comments

4. Upload image to Instagram:
   POST /v18.0/{ig-user-id}/media
   → image_url: public URL of instagram.png
   → caption: Claude output

5. Publish:
   POST /v18.0/{ig-user-id}/media_publish
   → creation_id: from step 4

6. Write to Google Sheets → SM Performance tab
```

### WhatsApp Channel

```
1. Receive: article_url, title, subcategory, top_product_price

2. Add UTM to article_url:
   ?utm_source=whatsapp&utm_medium=channel&utm_campaign=YYYY-MM-DD&utm_content={slug}

3. Call Claude API to generate broadcast message:
   → Max 300 characters
   → Include: article topic, top price/deal if transactional, link
   → Tone: WhatsApp message from a friend, not a newsletter
   → Example: "New guide out! 🍼 Best baby monitors under ₹5,000 — we tested 10 and only 3 are worth it. Check here 👉 [link]"

4. Send via WhatsApp Business API or Channel API

5. Write to Google Sheets → SM Performance tab
```

---

## 10. STAGE 7 — MONTHLY SEO INTELLIGENCE LOOP

**Trigger:** 1st of every month at 6:00 AM IST
**Script:** `scripts/stage7_seo.py`
**Runtime:** ~30-45 minutes
**Output:** Improved articles, updated Sheets, Word doc report in Drive

### What It Does

```
1. Pull Google Search Console data (last 28 days):
   → All pages with impressions > 50
   → Filter: avg_position > 10 AND avg_position <= 20

2. Sort by impressions DESC → top 20 near-miss articles

3. For each near-miss article:

   a. Scrape top 3 Google results for the same keyword
      (HTTP request to Google search, parse titles/meta)

   b. Call Claude API → Gap Analyser:
      → Compare our article outline vs competitor outlines
      → Identify: missing sections, missing FAQs, missing keywords
      → Return JSON: {gaps[], missing_keywords[], additional_faqs[]}

   c. Fetch our post via WordPress REST API

   d. Call Claude API → Content Patcher:
      → Write 300-500 words filling the identified gaps
      → Add 2-3 missing FAQ entries
      → Weave in missing keywords naturally

   e. PATCH post via REST API:
      → Append new content sections
      → Add new FAQ entries
      → Update "Last Updated" date

   f. Write to Google Sheets → SEO Loop Results tab

4. Pull all data for Analytics Feed update:
   → Which keywords rank 1-10? (feed to Stage 1: do more of this)
   → Which blog types rank fastest? (feed to Stage 1: use more of these)
   → Which subcategories perform best? (feed to Stage 1: prioritise)
   → Which social platform drove most traffic? (feed to Stage 6)

5. Write insights to Google Sheets → Analytics Feed tab
   → Each insight: what it is + action_for_stage1 + valid_until

6. Generate monthly SEO report:
   → Save as Word doc to Google Drive → /SEO-Reports/YYYY-MM/
   → Include: positions before/after, improvements made, traffic trend

7. Log: "Stage 7 complete: 20 articles improved, X insights added to feed"
```

---

## 11. STAGE 8 — EMAIL DIGEST

**Trigger:** Every Friday at 10:00 AM IST
**Script:** `scripts/stage8_email.py`
**Runtime:** ~3-5 minutes
**Output:** Email sent to Mailchimp list

### What It Does

```
1. Read Google Sheets → Published tab
   → Filter: published in last 7 days
   → Sort by word_count DESC (proxy for quality)
   → Take top 3 articles

2. Read Google Sheets → SM Performance tab
   → Filter: last 7 days
   → Find best_performing = YES posts
   → Take top deal/saving post if any

3. Call Claude API to generate email:
   → Subject: engaging, India-specific, this week's hook
   → Body: 3 article summaries (50 words each)
   → 1 deal highlight if price dropped on a featured product
   → 1 tip of the week (practical, for Indian parents)
   → Tone: weekly WhatsApp from a helpful friend

4. Create Mailchimp campaign:
   POST /3.0/campaigns
   → type: regular
   → recipients: full list
   → subject_line: Claude output
   → from_name: Baby Junctions
   → reply_to: hello@babyjunctions.in

5. Set campaign content:
   PUT /3.0/campaigns/{id}/content
   → html: formatted email HTML

6. Send campaign:
   POST /3.0/campaigns/{id}/actions/send

7. Write to Google Sheets → Monthly P&L tab: email_sent = YES
```

---

## 12. STAGE 9 — LLM/GEO OPTIMISATION

**Trigger:** Baked into every article at Stage 3 — no separate script needed
**Implementation:** Built into Claude's content generation prompt

### What Stage 3 Includes for GEO

```
Every article automatically includes:

1. Quick Answer Box at top
   → 2-3 sentence direct answer
   → LLMs quote these when answering user questions
   → Example: "The best baby monitor under ₹5,000 is the Motorola VM34..."

2. FAQ Schema (JSON-LD)
   → 5 question-answer pairs
   → ChatGPT, Perplexity, Gemini all extract FAQ schema

3. H2 headings are question-formatted
   → "Which baby monitor is best for Indian apartments?"
   → LLMs prefer clearly labelled sections

4. Structured comparison tables
   → LLMs extract table data easily for AI answers

5. Citable factual claims
   → Specific prices, ratings, specs — not vague descriptions
   → Perplexity specifically cites sites with hard data

6. Author expertise signals
   → "Reviewed by Baby Junctions Team"
   → Trust signals that LLMs weight
```

### One-Time Technical Setup (Stage 0)

```
robots.txt additions:
User-agent: GPTBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: ClaudeBot
Allow: /

Bing Webmaster Tools:
→ Submit sitemap (ChatGPT uses Bing index)
→ URL: bing.com/webmasters

About page:
→ Detailed "who we are" page
→ LLMs use About pages to assess credibility
```

---

## 13. MASTER PIPELINE SCRIPT

Save as: `scripts/master_pipeline.py`

```python
import os
import sys
from datetime import datetime

def get_pipeline_stage():
    """Determine which stage to run based on current time/day"""
    now = datetime.now()
    hour = now.hour
    day = now.day
    weekday = now.weekday()  # 4 = Friday

    # Stage 5: runs at 2 AM (20:30 UTC previous day)
    if hour == 2:
        return "stage5"

    # Stage 7: runs on 1st of month
    if day == 1 and hour == 6:
        return "stage7"

    # Stage 8: runs Friday 10 AM
    if weekday == 4 and hour == 10:
        return "stage8"

    # Stage 4: runs every hour 8 AM - 11 PM
    if 8 <= hour <= 23:
        return "stage4"

    # Default morning pipeline: stages 1, 2, 3, 3B
    if hour == 6:
        return "morning_pipeline"

    return None

def run_morning_pipeline():
    from stage1_topics import run as stage1
    from stage2_products import run as stage2
    from stage3_content import run as stage3
    from stage3b_images import run as stage3b

    print(f"[{datetime.now()}] Starting morning pipeline...")
    stage1()
    stage2()
    stage3()
    stage3b()
    print(f"[{datetime.now()}] Morning pipeline complete.")

if __name__ == "__main__":
    stage = get_pipeline_stage()
    if stage == "morning_pipeline":
        run_morning_pipeline()
    elif stage == "stage4":
        from stage4_publish import run
        run()
    elif stage == "stage5":
        from stage5_prices import run
        run()
    elif stage == "stage7":
        from stage7_seo import run
        run()
    elif stage == "stage8":
        from stage8_email import run
        run()
    else:
        print("No stage scheduled for this time.")
```

---

## 14. ERROR HANDLING RULES

When any stage fails:

```
1. Log error to Google Drive → /System/error-log.txt
   Format: [YYYY-MM-DD HH:MM] STAGE_X ERROR: {error_message}

2. Update pipeline-status.txt:
   Format: [YYYY-MM-DD] Stage X: FAILED — {brief reason}

3. For Stage 1-3 failures:
   → Skip today's run for that stage
   → Do NOT publish partial content
   → Retry automatically next day

4. For Stage 4 failures (individual article):
   → Log the failed article slug
   → Mark as FAILED in Content Queue tab
   → Continue to next article in next hour slot

5. For Stage 5 failures:
   → Keep existing prices (stale for 1 day is acceptable)
   → Retry in 1 hour

6. Never crash silently. Always log.
```

Full error definitions are in ERRORS.md.

---

## 15. PHASE BUILD ORDER — WHICH SCRIPTS TO BUILD FIRST

Build in this exact order. Test each before moving to next.

```
WEEK 1: Foundation
□ master_pipeline.py (skeleton only)
□ Google Sheets connection test
□ WordPress REST API connection test

WEEK 2: Content Pipeline
□ stage3_content.py (Claude writes article JSON)
□ stage3b_images.py (Pillow image generator)
□ stage4_publish.py (publish to WordPress)
→ Manually feed 10 topics → test full content → publish flow

WEEK 3: Research Pipeline
□ stage1_topics.py (topic research with deduplication)
□ stage2_products.py (Amazon PA API — use manual links until PA API approved)
→ Full pipeline test: Stage 1 → 2 → 3 → 4

WEEK 4: Social + Overnight
□ stage6_social.py (Instagram + Pinterest + WhatsApp)
□ stage5_prices.py (nightly price updater)
→ Full daily automation running

MONTH 2: Intelligence Loop
□ stage7_seo.py (Monthly SEO loop)
□ stage8_email.py (Weekly digest)
→ Analytics Feed loop feeding Stage 1
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
