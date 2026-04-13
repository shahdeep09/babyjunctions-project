# STORAGE.md — Baby Junctions
## Google Sheets + Google Drive — Complete Structure

> This file defines every tab in Google Sheets and every folder in Google Drive.
> The automation scripts read and write ONLY to the locations defined here.
> Never create new tabs or folders without updating this file first.

---

## PART 1 — GOOGLE SHEETS

### Sheet Name: Baby Junctions — Master Dashboard

One Google Sheet. 14 tabs. Everything tracked in one place.

---

### TAB 1: Daily Keywords

**Purpose:** Every keyword researched each day. Stage 1 writes here.

| Column | Type | Description |
|---|---|---|
| date | YYYY-MM-DD | Date keywords were researched |
| keyword | text | The exact search term |
| search_volume | number | Monthly searches (from DataForSEO or estimate) |
| competition | LOW/MED/HIGH | Keyword difficulty |
| subcategory | text | Which of the 50 subcategories |
| blog_type | text | Assigned blog type from 21 types |
| seasonal_relevance | text | Why relevant today |
| status | PENDING/ASSIGNED/USED | Whether used in a topic |
| topic_id | text | Linked topic ID if assigned |

---

### TAB 2: Daily Topics

**Purpose:** The 15 topics generated each day. Stage 1 writes here. Stage 2 reads here.

| Column | Type | Description |
|---|---|---|
| topic_id | BJ-YYYYMMDD-01 | Unique ID e.g. BJ-20260413-01 |
| date | YYYY-MM-DD | Date generated |
| title | text | Full article title |
| blog_type | text | One of the 21 blog types |
| subcategory | text | Baby subcategory |
| primary_keyword | text | Main SEO keyword |
| secondary_keywords | comma-separated | Supporting keywords |
| target_audience | text | Who this article is for |
| seasonal_angle | text | Why now |
| revenue_potential | HIGH/MEDIUM/LOW | Expected earning potential |
| estimated_aov_inr | number | Expected order value |
| status | QUEUED/RESEARCHING/WRITING/READY/PUBLISHED | Pipeline stage |
| assigned_slot | time | Which hourly slot e.g. 10:00 AM |
| notes | text | Any special instructions |

---

### TAB 3: Content Queue

**Purpose:** Articles that have been written and are ready to publish. Stage 3 writes here. Stage 4 reads here.

| Column | Type | Description |
|---|---|---|
| topic_id | text | Links back to Daily Topics tab |
| date | YYYY-MM-DD | Date article was written |
| title | text | Final article title |
| slug | text | WordPress URL slug |
| word_count | number | Total word count |
| primary_keyword | text | SEO focus keyword |
| meta_description | text | 155-char meta description |
| product_count | number | Number of products in article |
| json_file_path | text | Path to JSON in Google Drive |
| docx_file_path | text | Path to Word doc in Google Drive |
| image_thumbnail | text | Path to generated thumbnail |
| image_pinterest | text | Path to generated Pinterest pin |
| image_instagram | text | Path to generated Instagram image |
| status | READY/PUBLISHING/PUBLISHED/FAILED | Current state |
| scheduled_time | datetime | When to publish |

---

### TAB 4: Published

**Purpose:** Every live article. The deduplication source. Stage 1 reads this before generating topics.

| Column | Type | Description |
|---|---|---|
| post_id | number | WordPress post ID |
| topic_id | text | Original topic ID |
| published_date | YYYY-MM-DD | Date published |
| title | text | Article title |
| slug | text | URL slug |
| url | text | Full live URL |
| primary_keyword | text | SEO keyword |
| blog_type | text | Blog type |
| subcategory | text | Baby subcategory |
| word_count | number | Word count |
| product_count | number | Products featured |
| featured_product | text | Top recommended product |
| last_price_update | datetime | Last nightly price update |
| internal_links_count | number | How many internal links added |
| status | LIVE/UPDATED/NEEDS_REVIEW/OOS_DETECTED | Current state |

---

### TAB 5: Product Registry

**Purpose:** Every Amazon product used anywhere on the site. Stage 2 writes here. Stage 5 updates here nightly.

| Column | Type | Description |
|---|---|---|
| product_id | BJ-PROD-0001 | Internal product ID |
| asin | text | Amazon ASIN |
| title | text | Product title |
| subcategory | text | Baby subcategory |
| brand | text | Brand name |
| current_price_inr | number | Current price from last API call |
| star_rating | decimal | e.g. 4.3 |
| review_count | number | Total reviews |
| image_url | text | Amazon CDN image URL (hotlinked) |
| affiliate_link | text | Full affiliate URL with tag |
| availability | IN_STOCK/OOS/UNAVAILABLE | Current stock status |
| first_seen_date | YYYY-MM-DD | When first added |
| last_checked | datetime | Last price/stock check |
| used_in_articles | number | How many articles feature this |
| oos_since | YYYY-MM-DD | Date went OOS (blank if in stock) |
| replacement_product_id | text | If replaced, links to new product |
| passes_quality_filter | TRUE/FALSE | 3.8+ stars, 100+ reviews |

---

### TAB 6: Price History

**Purpose:** Track price changes over time per product. Useful for analysis.

| Column | Type | Description |
|---|---|---|
| date | YYYY-MM-DD | Date of price check |
| asin | text | Amazon ASIN |
| product_title | text | Product name |
| price_inr | number | Price on this date |
| previous_price_inr | number | Price on previous check |
| price_change | number | Difference (+ or -) |
| price_direction | UP/DOWN/SAME | Change direction |

---

### TAB 7: OOS Log

**Purpose:** Track out-of-stock events and replacements.

| Column | Type | Description |
|---|---|---|
| date_detected | YYYY-MM-DD | When OOS was first detected |
| asin | text | Product ASIN |
| product_title | text | Product name |
| article_url | text | Article where this product appears |
| days_oos | number | How many days OOS so far |
| action_taken | BADGE_SHOWN/REPLACED/PENDING | What system did |
| replacement_asin | text | ASIN of replacement product |
| replacement_date | YYYY-MM-DD | When replacement was made |
| notes | text | Any context |

---

### TAB 8: GSC Performance

**Purpose:** Google Search Console data. Stage 7 writes here monthly.

| Column | Type | Description |
|---|---|---|
| report_date | YYYY-MM-DD | Date of GSC pull |
| article_url | text | Page URL |
| article_title | text | Article title |
| primary_keyword | text | Focus keyword |
| avg_position | decimal | Average Google ranking position |
| total_clicks | number | Clicks in the period |
| total_impressions | number | Impressions in the period |
| ctr_pct | decimal | Click-through rate |
| position_change | number | Change from last month |
| action_needed | YES/NO | Whether improvement is queued |
| improvement_date | YYYY-MM-DD | When improvement was applied |

---

### TAB 9: SM Performance

**Purpose:** Social media metrics per post. Updated weekly via API.

| Column | Type | Description |
|---|---|---|
| date | YYYY-MM-DD | Post date |
| platform | INSTAGRAM/PINTEREST/WHATSAPP | Which platform |
| article_title | text | Linked article |
| article_url | text | Link with UTM |
| post_type | IMAGE/CAROUSEL/PIN/BROADCAST | Content format |
| impressions | number | How many times shown |
| reach | number | Unique accounts reached |
| clicks | number | Link clicks (UTM tracked) |
| saves | number | Saves/Pins saved (Pinterest) |
| comments | number | Comments received |
| shares | number | Shares/Repins |
| engagement_rate | decimal | (likes+comments+saves)/reach |
| best_performing | YES/NO | Top 10% performer flag |

---

### TAB 10: Product Performance

**Purpose:** Which products get most clicks, when, in which season.

| Column | Type | Description |
|---|---|---|
| month | YYYY-MM | Month of data |
| asin | text | Product ASIN |
| product_title | text | Product name |
| subcategory | text | Category |
| affiliate_clicks | number | Total affiliate link clicks |
| estimated_conversions | number | Estimated purchases |
| season | Summer/Monsoon/Festive/Winter | Season of performance |
| festival_period | text | Any festival that boosted it |
| rank_this_month | number | Rank among all products |
| trend | UP/DOWN/STABLE | Month over month |

---

### TAB 11: Analytics Feed

**Purpose:** The synthesised intelligence that feeds back into Stage 1 topic decisions.
Stage 7 writes here. Stage 1 reads here every morning before generating topics.

| Column | Type | Description |
|---|---|---|
| insight_date | YYYY-MM-DD | When insight was generated |
| insight_type | KEYWORD/CATEGORY/SEASON/PLATFORM/PRODUCT/BLOGTYPE | What kind of finding |
| insight | text | The actual finding in plain English |
| data_source | GSC/INSTAGRAM/PINTEREST/WHATSAPP/AMAZON | Where data came from |
| action_for_stage1 | text | How Stage 1 should use this |
| valid_until | YYYY-MM-DD | When this insight expires |
| priority | HIGH/MEDIUM/LOW | How much to weight it |

**Example rows:**
```
Baby monitor posts perform 3x better in Oct-Nov | GSC | Prioritise baby monitors in Oct | valid until Dec 31 | HIGH
Myth Buster blog type ranks within 30 days | GSC | Include 2 Myth Busters per week | MEDIUM
Pinterest pins with price in title get 40% more saves | PINTEREST | Always include price in pin title | HIGH
₹500-₹1500 range products get highest affiliate clicks | AMAZON | Prioritise mid-range products | HIGH
Instagram carousels outperform single images 2x | INSTAGRAM | Always use carousel format | MEDIUM
```

---

### TAB 12: SEO Loop Results

**Purpose:** Monthly SEO loop findings. Stage 7 writes here.

| Column | Type | Description |
|---|---|---|
| report_month | YYYY-MM | Which month |
| article_url | text | Article that was analysed |
| article_title | text | Title |
| position_before | decimal | GSC position before improvement |
| issues_found | text | What was missing |
| improvements_made | text | What was added/changed |
| position_after | decimal | Position at next check |
| improvement_date | YYYY-MM-DD | When changes were made |
| full_report_link | text | Link to Word doc in Google Drive |

---

### TAB 13: Weekly SM Report

**Purpose:** Best performing social posts each week. Auto-generated.

| Column | Type | Description |
|---|---|---|
| week_ending | YYYY-MM-DD | Sunday date |
| platform | text | Instagram/Pinterest/WhatsApp |
| top_post_title | text | Best performing article |
| top_post_url | text | Link |
| top_metric | text | What made it top (clicks/saves/reach) |
| top_metric_value | number | The number |
| insight | text | Why it performed well |
| replicate_this | YES/NO | Should we create similar content |

---

### TAB 14: Monthly P&L

**Purpose:** Revenue tracking. Manual entry by you.

| Column | Type | Description |
|---|---|---|
| month | YYYY-MM | Month |
| amazon_commission_inr | number | Amazon Associates earnings |
| adsense_inr | number | Google AdSense earnings |
| email_revenue_inr | number | Email list affiliate clicks |
| total_revenue_inr | number | Sum of above |
| claude_api_cost_inr | number | Claude API usage cost |
| other_costs_inr | number | Any other costs |
| total_costs_inr | number | Sum of costs |
| net_profit_inr | number | Revenue minus costs |
| total_articles_live | number | Articles live at end of month |
| total_monthly_visits | number | GA4 sessions |
| revenue_per_1k_visits | number | Calculated |
| notes | text | Any context for the month |

---

## PART 2 — GOOGLE DRIVE

### Root Folder: Baby Junctions — Content Archive

---

### Folder Structure:

```
📁 Baby Junctions — Content Archive/
│
├── 📁 Daily Content/
│   ├── 📁 2026-04-13/
│   │   ├── 📄 keywords.txt          ← All keywords researched today
│   │   ├── 📄 topics.txt            ← 15 topics chosen + reasoning
│   │   └── 📁 articles/
│   │       ├── 📄 best-baby-monitors-india.docx
│   │       ├── 📄 summer-diaper-rash-solutions.docx
│   │       └── ... (one Word doc per article published today)
│   │
│   ├── 📁 2026-04-14/
│   │   └── ... (same structure, auto-created every day)
│   │
│   └── 📁 2026-04-15/
│       └── ...
│
├── 📁 SEO Reports/
│   ├── 📁 2026-04/
│   │   └── 📄 april-2026-seo-report.docx
│   ├── 📁 2026-05/
│   │   └── 📄 may-2026-seo-report.docx
│   └── ...
│
├── 📁 Social Media Reports/
│   ├── 📄 week-2026-04-13.docx
│   ├── 📄 week-2026-04-20.docx
│   └── ...
│
└── 📁 System/
    ├── 📄 error-log.txt             ← Running log of automation errors
    └── 📄 pipeline-status.txt       ← Last run status of each stage
```

---

### What Goes In Each Daily Article Word Doc:

Every published article gets saved as a Word doc in its date folder.

```
Article Word Doc — Contents:
─────────────────────────────
Title
Published URL
Published Date
Primary Keyword
Blog Type
Subcategory
Word Count
Products Featured (list of ASINs + titles)
Meta Description
─────────────────────────────
[FULL ARTICLE CONTENT]
All headings, body text, product descriptions,
FAQs, buying guide — complete content exactly
as published on WordPress
─────────────────────────────
Performance Note (added by Stage 7 if updated):
"Updated on [date] — added [X] missing sections"
```

---

### Keywords.txt Format (Daily):

```
Date: 2026-04-13
Season: Summer (गर्मी)
Festivals: None this week

KEYWORDS RESEARCHED TODAY:
1. best baby monitor under 5000 | Vol: 2,400/mo | LOW competition | Baby Monitors | Top 10 List
2. summer diaper rash prevention india | Vol: 890/mo | LOW competition | Diapers | Problem-Solution
3. pigeon baby bottles review india | Vol: 1,200/mo | MED competition | Baby Bottles | Brand Deep Dive
... (all keywords)

KEYWORDS SELECTED FOR TODAY'S 15 TOPICS: [list of 15]
KEYWORDS DEFERRED (too similar to existing content): [list]
KEYWORDS DEFERRED (low quality): [list]
```

---

### Topics.txt Format (Daily):

```
Date: 2026-04-13
Total Topics: 15
Season: Summer | Festivals: None

TOPIC 1
Title: 10 Best Baby Monitors in India Under ₹5,000 (2026) — Tested
Blog Type: Top 10 List
Subcategory: Baby Monitors
Primary Keyword: best baby monitor india under 5000
Revenue Potential: HIGH
Seasonal Angle: Parents staying home in summer want audio monitoring
Urgency: Pre-monsoon purchases spike in April-May
Scheduled Slot: 9:00 AM
Status: PUBLISHED

... (all 15 topics)
```

---

## PART 3 — HOW SCRIPTS ACCESS STORAGE

### Reading from Google Sheets:
```python
import gspread
from google.oauth2.service_account import Credentials

# Authenticate using service account
creds = Credentials.from_service_account_info(
    json.loads(os.environ['GOOGLE_SERVICE_ACCOUNT_JSON']),
    scopes=['https://spreadsheets.google.com/feeds',
            'https://www.googleapis.com/auth/drive']
)
client = gspread.authorize(creds)
sheet = client.open("Baby Junctions — Master Dashboard")

# Read Published tab for deduplication
published = sheet.worksheet("Published").get_all_records()
existing_keywords = [row['primary_keyword'] for row in published]
existing_titles = [row['title'] for row in published]
```

### Writing to Google Sheets:
```python
# Write new topic to Daily Topics tab
topics_tab = sheet.worksheet("Daily Topics")
topics_tab.append_row([
    topic_id, date, title, blog_type, subcategory,
    primary_keyword, secondary_keywords, target_audience,
    seasonal_angle, revenue_potential, estimated_aov,
    "QUEUED", scheduled_time, ""
])
```

### Creating Daily Folder in Google Drive:
```python
from googleapiclient.discovery import build

drive_service = build('drive', 'v3', credentials=creds)

# Create today's folder
today = datetime.now().strftime('%Y-%m-%d')
folder_metadata = {
    'name': today,
    'mimeType': 'application/vnd.google-apps.folder',
    'parents': [DAILY_CONTENT_FOLDER_ID]
}
folder = drive_service.files().create(body=folder_metadata).execute()
today_folder_id = folder['id']

# Create articles subfolder
articles_folder_metadata = {
    'name': 'articles',
    'mimeType': 'application/vnd.google-apps.folder',
    'parents': [today_folder_id]
}
drive_service.files().create(body=articles_folder_metadata).execute()
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
