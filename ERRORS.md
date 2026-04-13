# ERRORS.md — Baby Junctions
## Known Failure Modes, Fallbacks & Fixes

> When automation breaks, check this file first.
> Every known error has a defined fix and fallback.
> Scripts log to Google Drive → /System/error-log.txt

---

## HOW TO READ THIS FILE

Each error entry has:
- **What happens:** The visible symptom
- **Why it happens:** The root cause
- **Fix:** How to resolve it
- **Fallback:** What the system does automatically if no fix is applied
- **Prevention:** How to stop it happening again

---

## STAGE 1 ERRORS — TOPIC RESEARCH

---

### ERROR S1-01: Claude returns invalid JSON

**What happens:** Stage 1 crashes with `json.JSONDecodeError`

**Why it happens:**
- Claude added markdown code fences (```json) despite instructions
- Response truncated due to max_tokens too low
- Claude added explanation before the JSON

**Fix:**
```python
# scripts/stage1_topics.py — already handled in call_claude_json()
# The wrapper strips markdown and retries once automatically
# If still failing, check max_tokens — increase to 5000 for Stage 1
```

**Fallback:** If both attempts fail, Stage 1 logs the error and uses yesterday's unused topics from Daily Topics tab (status = QUEUED from yesterday).

**Prevention:** System prompt says "No markdown, no preamble, raw JSON only." Already included.

---

### ERROR S1-02: All 15 topics are duplicates of existing content

**What happens:** Every generated topic matches something in the Published tab

**Why it happens:** Published tab has grown large and Claude's deduplication list is hitting token limits

**Fix:**
```python
# Trim deduplication list to last 90 days only
published = sheet.worksheet("Published").get_all_records()
recent = [r for r in published if r["published_date"] >= ninety_days_ago]
existing_keywords = [r["primary_keyword"] for r in recent]
```

**Fallback:** Claude generates topics in underused subcategories (those not published in 30+ days)

**Prevention:** Keep deduplication list to 90 days max. Already in stage1_topics.py.

---

### ERROR S1-03: Google Sheets connection fails

**What happens:** `gspread.exceptions.APIError` — unable to connect

**Why it happens:**
- Service account JSON is malformed in GitHub Secrets
- Google Sheets quota exceeded (100 requests per 100 seconds)
- Sheet was renamed or deleted

**Fix:**
1. Check `GOOGLE_SERVICE_ACCOUNT_JSON` secret is valid JSON
2. Check sheet name matches `GOOGLE_SHEETS_NAME` environment variable
3. Wait 2 minutes and retry

**Fallback:** Stage 1 generates topics without deduplication check. Logs warning: "Deduplication skipped — Sheets unavailable"

---

## STAGE 2 ERRORS — AMAZON PRODUCT RESEARCH

---

### ERROR S2-01: Amazon PA API returns 0 products

**What happens:** `search_amazon_products()` returns empty list

**Why it happens:**
- Keyword too specific — no results on Amazon.in
- PA API credentials expired or invalid
- Rate limit hit (called faster than 1/second)

**Fix — Keyword too specific:**
```python
# Broaden the search automatically
def search_with_fallback(keyword, subcategory):
    results = search_amazon_products(keyword)
    if not results:
        # Try subcategory name only
        results = search_amazon_products(subcategory)
    if not results:
        # Try generic baby + first word of keyword
        generic = "baby " + keyword.split()[0]
        results = search_amazon_products(generic)
    return results
```

**Fix — Rate limit:**
```python
# Add retry with exponential backoff
import time
time.sleep(2)  # Wait 2 seconds before retry
```

**Fallback:** If still 0 results after 3 attempts, mark topic status = NEEDS_REVIEW in Sheets. Skip this topic for today. System publishes 14 articles instead of 15.

---

### ERROR S2-02: Fewer than 3 products pass quality filter

**What happens:** Only 0-2 products have 3.8+ stars AND 100+ reviews AND in stock

**Why it happens:** Subcategory genuinely has few good products on Amazon.in

**Fix:**
```python
# Relax review count filter if rating is high
def relaxed_filter(item):
    try:
        rating = float(item.customer_reviews.star_rating.value)
        reviews = int(item.customer_reviews.count)
        availability = item.offers.listings[0].availability.type
        # Relax to 50 reviews if rating is 4.2+
        min_reviews = 50 if rating >= 4.2 else 100
        return rating >= 3.8 and reviews >= min_reviews and availability == "Now"
    except:
        return False
```

**Fallback:** Use 2 products if only 2 pass even relaxed filter. Log warning. Article becomes a comparison article instead of top 10.

---

### ERROR S2-03: PA API credentials not yet approved

**What happens:** API returns `InvalidAssociateId` or `InvalidPartnerTag`

**Why it happens:** Amazon Associates account pending. PA API not yet activated.

**Fix (temporary until PA API approved):**
```python
# Use manual product data during Phase 1
# Store manually gathered products in Google Sheets → Product Registry
# Stage 2 reads from Sheets instead of calling PA API

def get_products_manual_fallback(topic_id):
    products = sheet.worksheet("Product Registry").get_all_records()
    return [p for p in products if p["subcategory"] == topic_subcategory]
```

**Fallback:** This is the expected Phase 1 behaviour. Not an error — it's the plan.

---

## STAGE 3 ERRORS — CONTENT GENERATION

---

### ERROR S3-01: Article JSON missing required fields

**What happens:** Self-check fails with multiple missing field errors

**Why it happens:** Claude truncated the response due to token limits

**Fix:**
```python
# Detect truncation early
raw = call_claude(system, message)
if not raw.strip().endswith("}"):
    # Response was cut — request completion
    continuation_message = f"""
    Your previous response was cut off. The JSON was incomplete.
    Please complete the JSON from where it stopped.
    Last content received: ...{raw[-200:]}
    """
    raw2 = call_claude(system, continuation_message)
    raw = raw + raw2
```

**Fallback:** If still invalid after retry, mark topic status = CONTENT_FAILED. Skip for today. Log to error-log.txt.

---

### ERROR S3-02: Word count too low (under minimum threshold)

**What happens:** Self-check detects intro + body + guide < minimum for blog type

**Why it happens:** Claude wrote a thin article

**Fix:**
```python
# Add expansion prompt
expansion_message = f"""
The article is only {actual_words} words but needs at least {min_words}.
Expand the main_body section by adding:
1. One more detailed H2 section (200 words)
2. Two more sentences to the buying guide for each factor
Keep the same JSON structure. Return complete updated JSON.
"""
```

**Fallback:** Publish anyway with a NOTE in the pipeline log. Stage 7 will improve it next month if it underperforms.

---

### ERROR S3-03: Internal links point to articles that don't exist

**What happens:** Claude generates internal links to slugs not in Published tab

**Why it happens:** Claude invented plausible slugs that don't actually exist

**Fix:**
```python
# Validate all internal links before publishing
def validate_internal_links(article_json, published_slugs):
    valid_links = []
    for link in article_json.get("internal_links", []):
        if link["target_slug"] in published_slugs:
            valid_links.append(link)
        else:
            print(f"Removed invalid internal link: {link['target_slug']}")
    article_json["internal_links"] = valid_links
    return article_json
```

**Fallback:** Remove invalid links. Publish with fewer internal links. Log the removed ones.

---

## STAGE 4 ERRORS — WORDPRESS PUBLISHING

---

### ERROR S4-01: WordPress REST API returns 401 Unauthorized

**What happens:** `requests.post()` returns status 401

**Why it happens:**
- Application Password changed or revoked
- WordPress username incorrect
- Wordfence blocked the IP of GitHub Actions runner

**Fix:**
1. Log into WordPress Admin → Users → Profile → Application Passwords
2. Delete old password, create new one
3. Update `WP_APP_PASSWORD` in GitHub Secrets
4. For Wordfence: Wordfence → Firewall → Allowlisted IPs → add GitHub Actions IP ranges

**Fallback:** Article saved to Content Queue with status = PUBLISH_FAILED. Will retry next available hour slot.

---

### ERROR S4-02: Image upload fails — file too large

**What happens:** Media upload returns 413 Request Entity Too Large

**Why it happens:** GreenGeeks shared hosting has upload limits (usually 32MB — should be fine, but Pillow generates large PNGs)

**Fix:**
```python
# Compress image before upload
from PIL import Image

def compress_image(image_path, max_size_kb=500):
    img = Image.open(image_path)
    quality = 95
    while True:
        img.save(image_path, "PNG", optimize=True)
        size_kb = os.path.getsize(image_path) / 1024
        if size_kb <= max_size_kb or quality < 60:
            break
        quality -= 5
```

**Fallback:** Publish article without featured image. Set placeholder image. Log warning.

---

### ERROR S4-03: Duplicate post — article already published

**What happens:** WordPress returns 201 but with a slug like `best-baby-monitors-india-2`

**Why it happens:** Stage 4 ran twice (GitHub Actions double-trigger), or slug collision

**Fix:**
```python
# Check if slug already exists before publishing
def check_slug_exists(slug):
    response = requests.get(
        f"{WP_BASE}/posts?slug={slug}",
        auth=WP_AUTH
    )
    return len(response.json()) > 0

# In stage4_publish.py
if check_slug_exists(article["seo"]["slug"]):
    print(f"Slug already exists: {article['seo']['slug']} — skipping")
    update_topic_status(topic_id, "ALREADY_PUBLISHED")
    return
```

**Fallback:** Skip the duplicate. Mark as ALREADY_PUBLISHED in Sheets.

---

### ERROR S4-04: GreenGeeks server timeout

**What happens:** `requests.exceptions.Timeout` or `ConnectionError`

**Why it happens:** Shared hosting server temporarily overloaded

**Fix:**
```python
# Add timeout and retry
def publish_with_retry(payload, retries=3):
    for attempt in range(retries):
        try:
            response = requests.post(
                f"{WP_BASE}/posts",
                json=payload,
                auth=WP_AUTH,
                timeout=30
            )
            if response.status_code == 201:
                return response.json()
        except requests.exceptions.Timeout:
            wait = 30 * (attempt + 1)
            print(f"Timeout. Waiting {wait}s before retry {attempt + 1}...")
            time.sleep(wait)
    raise Exception("WordPress publish failed after 3 retries")
```

**Fallback:** Mark article as PUBLISH_FAILED. Try again in next hour slot.

---

## STAGE 5 ERRORS — PRICE UPDATER

---

### ERROR S5-01: Price shows as 0 or None

**What happens:** PA API returns product but price is missing

**Why it happens:** Product has no "buyable" offer (marketplace seller only, no direct Amazon offer)

**Fix:**
```python
def safe_get_price(item):
    try:
        return float(item.offers.listings[0].price.amount)
    except (AttributeError, IndexError, TypeError):
        # Try savings basis price
        try:
            return float(item.offers.listings[0].savings_basis.amount)
        except:
            return None  # Price unavailable

# If price is None: keep old price, mark as "Price varies"
```

**Fallback:** Keep previous price in database. Add note "Price may vary" next to the price display.

---

### ERROR S5-02: OOS replacement product also goes OOS immediately

**What happens:** Replacement product is found but is also OOS when checked next cycle

**Why it happens:** High-demand product, seasonal stockout

**Fix:**
```python
# Find 3 replacement candidates, not just 1
# If primary replacement goes OOS, fall to secondary
def find_replacements(keyword, subcategory, exclude_asins):
    products = search_amazon_products(keyword)
    replacements = [
        p for p in products
        if passes_quality_filter(p) and p.asin not in exclude_asins
    ]
    return replacements[:3]  # Keep 3 in reserve
```

**Fallback:** If all replacements are OOS, show section as "Currently no products available in this category — check back soon." Log for manual review.

---

## STAGE 6 ERRORS — SOCIAL POSTING

---

### ERROR S6-01: Instagram token expired

**What happens:** Instagram API returns `#190 — Token expired`

**Why it happens:** Page Access Tokens expire every 60 days unless refreshed

**Fix:**
1. Go to Facebook Developers → Your App → Tools → Graph API Explorer
2. Generate new long-lived token (valid 60 days)
3. Update `INSTAGRAM_PAGE_TOKEN` in GitHub Secrets

**Prevention:** Set a Google Calendar reminder every 50 days to refresh the token.

**Fallback:** Skip Instagram post for this article. Log as IG_TOKEN_EXPIRED. Continue pipeline.

---

### ERROR S6-02: Instagram image URL not publicly accessible

**What happens:** Instagram rejects the image URL with error `#100`

**Why it happens:** GitHub Actions stores images at `/tmp/` — not publicly accessible. Instagram needs a public URL.

**Fix:**
```python
# Upload image to WordPress first, use WordPress URL
# This is the correct flow — image goes to WP Media Library,
# then WordPress URL is used for Instagram

def get_public_image_url(wordpress_media_id):
    response = requests.get(
        f"{WP_BASE}/media/{wordpress_media_id}",
        auth=WP_AUTH
    )
    return response.json()["source_url"]
```

**Already handled in Stage 4 → Stage 6 flow. Images uploaded to WP first.**

---

### ERROR S6-03: Pinterest board ID not found for subcategory

**What happens:** `KeyError` when looking up board ID in `PINTEREST_BOARDS` dict

**Why it happens:** New subcategory added that isn't in the mapping yet

**Fix:**
```python
# Use fallback board
board_id = PINTEREST_BOARDS.get(subcategory, PINTEREST_BOARDS["default"])
```

**Prevention:** Every time a new subcategory is used, add it to `PINTEREST_BOARDS` in APIS.md and in the script.

---

## STAGE 7 ERRORS — SEO LOOP

---

### ERROR S7-01: GSC returns no data for new site

**What happens:** GSC API returns 0 rows

**Why it happens:** Site is too new — not enough clicks/impressions yet to appear in GSC data

**Fix:** Skip Stage 7 if site is less than 3 months old. Check:
```python
site_age_days = (datetime.now() - SITE_LAUNCH_DATE).days
if site_age_days < 90:
    print("Site too new for SEO analysis. Skipping Stage 7.")
    exit(0)
```

**Fallback:** Stage 7 runs but generates 0 improvements. That is fine. It will run again next month.

---

### ERROR S7-02: Competitor scraping blocked

**What happens:** Requests to scrape competitor headings return 403 or CAPTCHA

**Why it happens:** Some sites block scrapers

**Fix:**
```python
import time
import random

headers = {
    "User-Agent": "Mozilla/5.0 (compatible; Googlebot/2.1)"
}

def safe_scrape(url):
    time.sleep(random.uniform(2, 5))
    try:
        response = requests.get(url, headers=headers, timeout=10)
        if response.status_code == 200:
            return response.text
        return None
    except:
        return None
```

**Fallback:** If all 3 competitors block scraping, run gap analysis with only our article's content. Claude will identify obvious missing sections from general knowledge.

---

## GENERAL ERRORS

---

### ERROR G-01: GitHub Actions run fails silently

**What happens:** Workflow shows green tick but no articles published

**Why it happens:** Script ran but exited early without error

**Fix:**
```python
# Add explicit success logging at end of every stage
print(f"[SUCCESS] Stage {STAGE_NUM} completed at {datetime.now()}")
print(f"[SUCCESS] Articles processed: {count}")

# Add to end of master_pipeline.py:
if __name__ == "__main__":
    try:
        run()
        log_to_drive(f"Pipeline completed successfully at {datetime.now()}")
    except Exception as e:
        log_to_drive(f"Pipeline FAILED: {str(e)}")
        raise  # Re-raise so GitHub marks the run as failed
```

---

### ERROR G-02: GitHub Actions free minutes exceeded

**What happens:** Workflow is queued but never runs

**Why it happens:** Free GitHub Actions gives 2,000 minutes/month for private repos. 15 articles × ~5 min = 75 min/day = ~2,250 min/month — slightly over limit.

**Fix options:**
1. Make repo public (unlimited free minutes for public repos)
2. Optimise scripts to run faster (target < 4 min total)
3. Use GitHub's free tier more efficiently — combine stages

**Best fix:** Make the repo public. The code doesn't contain API keys (those are in Secrets). Making it public is safe and gives unlimited minutes.

---

### ERROR G-03: .env file accidentally committed to git

**What happens:** API keys visible in GitHub repository

**Why it happens:** Developer accidentally committed `.env` file

**Immediate fix:**
```bash
# Remove from git history
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "Remove .env from tracking"
git push

# Immediately rotate ALL API keys:
# - Anthropic: console.anthropic.com → API Keys → Delete + New
# - Amazon PA API: Amazon Associates → PA API → Regenerate
# - WordPress: Users → Profile → App Passwords → Delete + New
# - Google: Service Account → IAM → Delete + New key
```

**Prevention:** `.gitignore` must include `.env` before any first commit. Already noted.

---

## ERROR LOG FORMAT

Every error written to Google Drive → /System/error-log.txt uses this format:

```
[2026-04-13 08:23:15] STAGE-4 ERROR S4-01: WordPress 401 Unauthorized
Topic: BJ-20260413-07 — Best Baby Monitors Under ₹5000
Action taken: Marked PUBLISH_FAILED in Sheets. Will retry next slot.
Resolution: Check WP_APP_PASSWORD in GitHub Secrets.
```

---

## PIPELINE STATUS FORMAT

Google Drive → /System/pipeline-status.txt updated after every stage:

```
Last updated: 2026-04-13 22:00 IST

Stage 1:  ✅ COMPLETE — 15 topics generated at 06:05 AM
Stage 2:  ✅ COMPLETE — 15/15 topics have products at 06:22 AM
Stage 3:  ✅ COMPLETE — 15 articles written at 07:08 AM
Stage 3B: ✅ COMPLETE — 45 images generated at 07:15 AM
Stage 4:  ✅ COMPLETE — 15/15 articles published (last at 10:00 PM)
Stage 5:  ✅ COMPLETE — 127 prices updated, 2 OOS detected at 02:07 AM
Stage 6:  ✅ COMPLETE — 15 Pinterest + 15 Instagram + 15 WhatsApp posts
Stage 7:  ⏭️ SCHEDULED — Runs 1st of next month
Stage 8:  ⏭️ SCHEDULED — Runs Friday 10 AM
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
