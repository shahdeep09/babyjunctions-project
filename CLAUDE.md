# CLAUDE.md — Baby Junctions
> Read this file first. Then read ONLY the file you need. Do not load all docs.

## Project Summary
- **Brand:** Baby Junctions (babyjunctions.in)
- **Niche:** Baby products affiliate — India
- **Stack:** WordPress (GreenGeeks) + GitHub Actions + Claude API + Amazon PA API
- **Language:** English + slight Hinglish
- **Audience:** Indian parents, 25–50 yrs, tier 1/2/3 cities

---

## Current Phase
```
Phase 1 — Semi-manual build
✅ WordPress live
✅ MD docs written
❌ Amazon PA API (pending Associates approval)
❌ GitHub Actions (not yet set up)
❌ Full automation (Phase 2)
```

---

## File Map — Load ONLY What You Need

| I need to... | Read this file |
|---|---|
| Understand full project | `docs/MASTER.md` |
| Write or fix Python scripts | `docs/WORKFLOW.md` + `docs/APIS.md` |
| Write Claude prompts | `docs/PROMPTS.md` |
| Write or edit articles | `docs/CONTENT.md` |
| Fix CSS / HTML / design | `docs/DESIGN.md` |
| Set up WordPress / plugins | `docs/WORDPRESS.md` |
| Work with Sheets or Drive | `docs/STORAGE.md` |
| Debug pipeline errors | `docs/ERRORS.md` |
| Analyse performance data | `docs/ANALYSIS.md` |

> ⚠️ Never load all files at once. One task = one or two files max.

---

## Folder Structure
```
baby-junctions/
├── CLAUDE.md                  ← you are here
├── docs/
│   ├── MASTER.md              ← project overview
│   ├── WORKFLOW.md            ← 9-stage pipeline logic
│   ├── APIS.md                ← all API code snippets
│   ├── PROMPTS.md             ← all Claude prompts
│   ├── CONTENT.md             ← voice, blog types, JSON schema
│   ├── DESIGN.md              ← colors, fonts, CSS, HTML components
│   ├── WORDPRESS.md           ← plugins, REST API, custom fields
│   ├── STORAGE.md             ← Sheets tabs + Drive folders
│   ├── ANALYSIS.md            ← performance tracking + feedback loop
│   └── ERRORS.md              ← error fixes + fallbacks
└── scripts/
    ├── master_pipeline.py
    ├── stage1_topics.py
    ├── stage2_products.py
    ├── stage3_content.py
    ├── stage3b_images.py
    ├── stage4_publish.py
    ├── stage5_prices.py
    ├── stage6_social.py
    ├── stage7_seo.py
    └── stage8_email.py
```

---

## Key Facts (No Need to Open Other Files for These)

**Claude API model:** `claude-sonnet-4-20250514`

**Amazon filters:** stars ≥ 3.8 · reviews ≥ 100 · IN_STOCK only

**PA API rate limit:** 1 request/second — always `time.sleep(1)`

**Daily schedule:**
- 6:00 AM → Stage 1 (topics)
- 6:30 AM → Stage 2 (products)
- 7:00 AM → Stage 3 (content + images)
- 8AM–11PM → Stage 4 (publish 1/hr)
- 2:00 AM → Stage 5 (prices)
- Friday 10AM → Stage 8 (email)
- 1st of month → Stage 7 (SEO loop)

**Prices:** Always ₹ — never $. Always Amazon.in — never Amazon.com.

**Never commit `.env` to git.** All secrets go in GitHub Secrets only.

---

## ENV Variables (Quick Reference)
```
ANTHROPIC_API_KEY
AMAZON_ACCESS_KEY · AMAZON_SECRET_KEY · AMAZON_ASSOCIATE_TAG
WP_SITE_URL · WP_USERNAME · WP_APP_PASSWORD
GOOGLE_SERVICE_ACCOUNT_JSON · GOOGLE_SHEETS_ID · GOOGLE_SHEETS_NAME
GOOGLE_DRIVE_ROOT_FOLDER_ID
INSTAGRAM_PAGE_TOKEN · INSTAGRAM_USER_ID
PINTEREST_ACCESS_TOKEN
MAILCHIMP_API_KEY · MAILCHIMP_LIST_ID
GA4_PROPERTY_ID
```

---

## Build Order (Week by Week)
```
Week 1 → Sheets + WP REST API connection test
Week 2 → stage3 + stage3b + stage4 (manual feed 10 topics → test publish)
Week 3 → stage1 + stage2 (full pipeline test)
Week 4 → stage5 + stage6 (prices + social live)
Month 2 → stage7 + stage8 (SEO loop + email)
```

*v1.0 | April 2026*
