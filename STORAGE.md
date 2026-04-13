# STORAGE.md — Baby Junctions
> Google Sheets tabs + Google Drive folders. Scripts read/write only here.

## GOOGLE SHEETS — "Baby Junctions — Master Dashboard"

### TAB 1: Daily Keywords
| Column | Type | Notes |
|---|---|---|
| date | YYYY-MM-DD | |
| keyword | text | |
| search_volume | number | |
| competition | LOW/MED/HIGH | |
| subcategory | text | |
| blog_type | text | |
| seasonal_relevance | text | |
| status | PENDING/ASSIGNED/USED | |
| topic_id | text | |

### TAB 2: Daily Topics
| Column | Type | Notes |
|---|---|---|
| topic_id | BJ-YYYYMMDD-01 | |
| date | YYYY-MM-DD | |
| title | text | |
| blog_type | text | |
| subcategory | text | |
| primary_keyword | text | |
| secondary_keywords | comma-separated | |
| target_audience | text | |
| seasonal_angle | text | |
| revenue_potential | HIGH/MEDIUM/LOW | |
| estimated_aov_inr | number | |
| status | QUEUED/RESEARCHING/WRITING/READY/PUBLISHED | |
| scheduled_slot | time | e.g. 10:00 AM |
| notes | text | |

### TAB 3: Content Queue
| Column | Type | Notes |
|---|---|---|
| topic_id | text | |
| date | YYYY-MM-DD | |
| title | text | |
| slug | text | |
| word_count | number | |
| primary_keyword | text | |
| meta_description | text | |
| product_count | number | |
| docx_file_path | text | Drive path |
| image_thumbnail | text | |
| image_pinterest | text | |
| image_instagram | text | |
| status | READY/PUBLISHING/PUBLISHED/FAILED | |
| scheduled_time | datetime | |

### TAB 4: Published (deduplication source — Stage 1 reads this first)
| Column | Type | Notes |
|---|---|---|
| post_id | number | WP post ID |
| topic_id | text | |
| published_date | YYYY-MM-DD | |
| title | text | |
| slug | text | |
| url | text | full live URL |
| primary_keyword | text | |
| blog_type | text | |
| subcategory | text | |
| word_count | number | |
| last_price_update | datetime | |
| status | LIVE/UPDATED/OOS_DETECTED | |

### TAB 5: Product Registry
| Column | Type | Notes |
|---|---|---|
| product_id | BJ-PROD-0001 | |
| asin | text | |
| title | text | |
| subcategory | text | |
| brand | text | |
| current_price_inr | number | |
| star_rating | decimal | |
| review_count | number | |
| image_url | text | Amazon CDN — hotlink only |
| affiliate_link | text | |
| availability | IN_STOCK/OOS/UNAVAILABLE | |
| last_checked | datetime | |
| oos_since | YYYY-MM-DD | blank if in stock |
| passes_quality_filter | TRUE/FALSE | 3.8★ + 100 reviews |

### TAB 6: Price History
| date | asin | price_inr | previous_price | price_change | direction UP/DOWN/SAME |

### TAB 7: OOS Log
| date_detected | asin | product_title | article_url | days_oos | action BADGE/REPLACED/PENDING | replacement_asin | replacement_date |

### TAB 8: GSC Performance
| report_date | article_url | primary_keyword | avg_position | total_clicks | total_impressions | ctr_pct | position_change | action_needed YES/NO |

### TAB 9: SM Performance
| date | platform INSTAGRAM/PINTEREST/WHATSAPP | article_title | article_url | post_type | impressions | reach | clicks | saves | engagement_rate | best_performing YES/NO |

### TAB 10: Product Performance
| month | asin | product_title | subcategory | affiliate_clicks | season | festival_period | trend UP/DOWN/STABLE |

### TAB 11: Analytics Feed (Stage 1 reads this every morning)
| insight_date | insight_type KEYWORD/CATEGORY/SEASON/PLATFORM/PRODUCT/BLOGTYPE | insight | data_source | action_for_stage1 | valid_until | priority HIGH/MEDIUM/LOW |

### TAB 12: SEO Loop Results
| report_month | article_url | position_before | issues_found | improvements_made | position_after | improvement_date |

### TAB 13: Weekly SM Report
| week_ending | platform | top_post_title | top_metric | top_metric_value | insight | replicate YES/NO |

### TAB 14: Monthly P&L
| month | amazon_commission_inr | adsense_inr | email_revenue_inr | total_revenue | claude_api_cost | other_costs | total_costs | net_profit | articles_live | monthly_visits |

---

## GOOGLE DRIVE — "Baby Junctions — Content Archive"
```
/Baby Junctions — Content Archive/
├── /Daily Content/
│   └── /YYYY-MM-DD/
│       ├── keywords.txt
│       ├── topics.txt
│       └── /articles/
│           ├── {slug}.docx
│           └── ...
├── /SEO-Reports/
│   └── /YYYY-MM/
│       └── {month}-seo-report.docx
├── /Social Media Reports/
│   └── week-{YYYY-MM-DD}.docx
└── /System/
    ├── error-log.txt
    └── pipeline-status.txt
```

### Article .docx Contents
```
Title | Published URL | Date | Keyword | Blog Type | Subcategory | Word Count
Products: [ASIN - Title list]
Meta Description
---
[FULL ARTICLE CONTENT]
---
Performance Note (added by Stage 7 if updated): "Updated {date} — added {X}"
```

### keywords.txt Format
```
Date: YYYY-MM-DD | Season: {season} | Festivals: {festivals}
1. {keyword} | {volume}/mo | {competition} | {subcategory} | {blog_type}
...
Selected for today: [list]
Deferred (duplicate): [list]
```

### topics.txt Format
```
Date: YYYY-MM-DD | Total: 15 | Season: {season}
TOPIC 1
Title: {title} | Type: {type} | Keyword: {kw}
Revenue: {HIGH/MED/LOW} | Slot: {time} | Status: {status}
...
```

## SHEETS ACCESS (Python)
```python
import gspread, json
from google.oauth2.service_account import Credentials

def get_sheet(tab_name):
    creds = Credentials.from_service_account_info(
        json.loads(os.environ["GOOGLE_SERVICE_ACCOUNT_JSON"]),
        scopes=["https://spreadsheets.google.com/feeds","https://www.googleapis.com/auth/drive"])
    return gspread.authorize(creds).open(os.environ["GOOGLE_SHEETS_NAME"]).worksheet(tab_name)
```

## DRIVE — CREATE DAILY FOLDER
```python
def create_daily_folder(parent_id):
    from googleapiclient.discovery import build
    today = datetime.now().strftime("%Y-%m-%d")
    svc = build("drive","v3",credentials=get_drive_creds())
    res = svc.files().list(q=f"name='{today}' and '{parent_id}' in parents and mimeType='application/vnd.google-apps.folder'",fields="files(id)").execute()
    if res["files"]: return res["files"][0]["id"]
    f = svc.files().create(body={"name":today,"mimeType":"application/vnd.google-apps.folder","parents":[parent_id]},fields="id").execute()
    # Also create /articles subfolder
    svc.files().create(body={"name":"articles","mimeType":"application/vnd.google-apps.folder","parents":[f["id"]]},fields="id").execute()
    return f["id"]
```

*v1.0 | April 2026*
