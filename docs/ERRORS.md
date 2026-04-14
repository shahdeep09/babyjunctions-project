# ERRORS.md — Baby Junctions
> Check here first when anything breaks. Every error has a fix and a fallback.

## ERROR LOG FORMAT
```
[YYYY-MM-DD HH:MM] STAGE-X ERROR {CODE}: {message}
Topic: {topic_id} — {title}
Action: {what system did}
Fix: {resolution needed}
```

## PIPELINE STATUS FORMAT
```
Stage 1: ✅ COMPLETE — 15 topics at 06:05 AM
Stage 2: ✅ COMPLETE — 15/15 products at 06:22 AM
Stage 3: ✅ COMPLETE — 15 articles at 07:08 AM
Stage 4: ✅ COMPLETE — 15/15 published (last 10PM)
Stage 5: ✅ COMPLETE — 127 prices updated, 2 OOS
Stage 6: ✅ COMPLETE — 15×3 social posts
Stage 7: ⏭️ SCHEDULED — 1st of next month
Stage 8: ⏭️ SCHEDULED — Friday 10AM
```

---

## STAGE 1 ERRORS

**S1-01: Claude returns invalid JSON**
Fix: `call_claude_json()` auto-strips markdown + retries once
Fallback: Use yesterday's QUEUED topics from Daily Topics tab

**S1-02: All topics are duplicates**
Fix: Trim deduplication list to last 90 days only
```python
from datetime import datetime, timedelta
cutoff = (datetime.now() - timedelta(days=90)).strftime("%Y-%m-%d")
recent = [r for r in published if r["published_date"] >= cutoff]
```

**S1-03: Google Sheets connection fails**
Fix: Check GOOGLE_SERVICE_ACCOUNT_JSON secret is valid JSON. Wait 2 min, retry.
Fallback: Generate topics without deduplication. Log warning.

---

## STAGE 2 ERRORS

**S2-01: PA API returns 0 products**
```python
def search_with_fallback(keyword, subcategory):
    results = search_products(keyword)
    if not results: results = search_products(subcategory)
    if not results: results = search_products("baby " + keyword.split()[0])
    return results
```
Fallback: Mark topic NEEDS_REVIEW. Publish 14 articles instead of 15.

**S2-02: Fewer than 3 products pass filter**
```python
# Relax review count if rating is high
min_reviews = 50 if rating >= 4.2 else 100
```
Fallback: Use 2 products. Article becomes comparison instead of Top 10.

**S2-03: PA API credentials not approved yet**
This is expected in Phase 1. Use manual product data from Product Registry tab instead.

---

## STAGE 3 ERRORS

**S3-01: JSON truncated (response cut off)**
```python
if not raw.strip().endswith("}"):
    # Request completion
    raw2 = call_claude(system, f"Complete the JSON from: ...{raw[-200:]}")
    raw = raw + raw2
```
Fallback: Mark CONTENT_FAILED. Skip today. Log error.

**S3-02: Word count too low**
```python
# Add expansion prompt
"Expand main_body by adding one more H2 section (200 words) + 2 more sentences per buying guide factor"
```
Fallback: Publish anyway. Stage 7 improves it if it underperforms.

**S3-03: Internal links point to non-existent slugs**
```python
def validate_links(article_json, published_slugs):
    article_json["internal_links"] = [l for l in article_json.get("internal_links",[]) if l["target_slug"] in published_slugs]
    return article_json
```

---

## STAGE 4 ERRORS

**S4-01: WordPress REST API 401 Unauthorized**
Fix: WP Admin → Users → Profile → App Passwords → delete + create new → update GitHub Secret
Also check: Wordfence → Firewall → add GitHub Actions IP to allowlist

**S4-02: Image upload 413 (too large)**
```python
from PIL import Image
def compress_image(path, max_kb=500):
    img = Image.open(path)
    quality = 95
    while os.path.getsize(path)/1024 > max_kb and quality >= 60:
        img.save(path, "PNG", optimize=True)
        quality -= 5
```
Fallback: Publish without featured image. Log warning.

**S4-03: Duplicate slug**
```python
if slug_exists(article["seo"]["slug"]):
    update_status("Daily Topics", "topic_id", tid, "status", "ALREADY_PUBLISHED")
    return  # skip
```

**S4-04: GreenGeeks timeout**
```python
for attempt in range(3):
    try:
        r = requests.post(url, json=payload, auth=WP_AUTH, timeout=30)
        if r.status_code == 201: return r.json()
    except requests.exceptions.Timeout:
        time.sleep(30 * (attempt+1))
raise Exception("WP publish failed after 3 retries")
```

---

## STAGE 5 ERRORS

**S5-01: Price returns 0 or None**
```python
def safe_price(item):
    try: return float(item.offers.listings[0].price.amount)
    except: return None  # keep old price, show "Price varies"
```

**S5-02: Replacement product also goes OOS**
```python
# Keep 3 replacement candidates, not just 1
replacements = [p for p in products if passes_filter(p) and p.asin not in exclude][:3]
```
Fallback: Show "No products available in this category — check back soon."

---

## STAGE 6 ERRORS

**S6-01: Instagram token expired (every 60 days)**
Fix: Facebook Dev → Graph API Explorer → generate new long-lived token → update GitHub Secret
Prevention: Set Google Calendar reminder every 50 days.
Fallback: Skip Instagram for this article. Log IG_TOKEN_EXPIRED.

**S6-02: Pinterest board ID not found**
```python
board_id = PINTEREST_BOARDS.get(subcategory, PINTEREST_BOARDS["default"])
```
Prevention: Add new subcategories to PINTEREST_BOARDS immediately.

---

## STAGE 7 ERRORS

**S7-01: GSC returns no data (site too new)**
```python
SITE_LAUNCH_DATE = datetime(2026, 4, 13)
if (datetime.now() - SITE_LAUNCH_DATE).days < 90:
    print("Site too new. Skipping Stage 7.")
    exit(0)
```

**S7-02: Competitor scraping blocked (403)**
```python
import time, random
headers = {"User-Agent": "Mozilla/5.0 (compatible; Googlebot/2.1)"}
time.sleep(random.uniform(2, 5))
```
Fallback: Run gap analysis without competitor data. Claude identifies obvious gaps.

---

## GENERAL ERRORS

**G-01: GitHub Actions runs silently but does nothing**
```python
# Always end every stage with explicit success log
print(f"[SUCCESS] Stage {N} complete at {datetime.now()}")
# In master_pipeline.py — re-raise exceptions so GitHub marks run as FAILED
try: run()
except Exception as e: log_error(e); raise
```

**G-02: GitHub Actions free minutes exceeded**
Fix: Make repo public (unlimited free minutes). Code is safe — keys are in Secrets not code.

**G-03: .env committed to git accidentally**
```bash
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "Remove .env"
git push
# IMMEDIATELY rotate ALL API keys — Anthropic, Amazon, WordPress, Google
```

*v1.0 | April 2026*
