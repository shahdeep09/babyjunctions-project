# MASTER.md — Baby Junctions
## The Single Source of Truth for the Entire Project

> Every script, every Claude prompt, every API call in this project reads this file first.
> If something conflicts with MASTER.md, MASTER.md wins. Always.

---

## 1. PROJECT IDENTITY

| Field | Value |
|---|---|
| Brand Name | Baby Junctions |
| Domain | babyjunctions.in |
| Tagline | India ke parents ka trusted baby guide |
| Niche | Baby Products — India |
| Language | English with slight Hinglish warmth |
| Target Audience | Indian parents, 25–50 years, tier 1/2/3 cities |
| Content Standard | BabyGearLab-level depth — not thin, not generic |
| Affiliate Platform | Amazon Associates India (PA API 5.0) |
| CMS | WordPress on GreenGeeks shared hosting |
| Automation | GitHub Actions (free cron) + Python scripts |
| AI Engine | Claude API (claude-sonnet-4-20250514) |

---

## 2. CONTENT VOICE & TONE

This is non-negotiable. Every article must sound like this:

**Who is writing:** A trusted Indian parent — someone who has done the research so other parents don't have to. Not a salesperson. Not a robot. A didi or bhaiya who genuinely cares.

**Tone rules:**
- Warm, direct, slightly Hinglish — e.g. "Yeh product thoda costly hai, but it's worth it"
- Never pushy. Never fake. Never over-excited.
- Always honest — if a product has a downside, say it
- Use INR prices everywhere — never USD
- Reference Indian seasons, festivals, cities naturally
- Write like you're WhatsApp-messaging a friend who just asked for advice

**What to NEVER write:**
- "In this article, we will discuss..."
- "As an Amazon Associate we earn..."  (put this only in footer disclosure)
- Generic intros that could apply to any country
- Fake superlatives — "The ABSOLUTE BEST product EVER"

---

## 3. TECH STACK

| Tool | Role | Cost |
|---|---|---|
| GreenGeeks Shared Hosting | WordPress website hosting | Already owned |
| WordPress (Kadence Theme) | CMS + frontend | Free |
| GitHub Actions | Automation cron — runs all scripts daily | Free |
| Claude API | Content writing, topic research, analysis | Pay per use |
| Amazon PA API | Product data, prices, images, affiliate links | Free |
| Google Sheets | Live pipeline dashboard + all tracking tabs | Free |
| Google Drive | Daily folders + article Word docs + reports | Free |
| Python 3.x | All automation scripts | Free |
| Pillow (Python) | Blog image generation — no external API | Free |
| Rank Math SEO | WordPress SEO + schema markup | Free tier |
| Mailchimp | Weekly email digest | Free up to 500 subs |
| Instagram Graph API | Auto-post after each publish | Free |
| Pinterest API | Auto-pin after each publish | Free |
| WhatsApp Channel | Broadcast after each publish (UTM tracked) | Free |

---

## 4. THE 10 MD FILES — WHAT EACH ONE COVERS

| File | Purpose |
|---|---|
| **MASTER.md** | This file. Project identity, rules, stack. Read first always. |
| **DESIGN.md** | Visual system — colors, fonts, components, WordPress theme rules |
| **WORDPRESS.md** | Plugins, settings, custom fields, REST API payload structure |
| **CONTENT.md** | Brand voice detail, all 21 blog types, JSON schema, BabyGearLab standard |
| **WORKFLOW.md** | Complete 9-stage pipeline — what runs, when, triggered by what |
| **PROMPTS.md** | Every Claude API prompt for every stage, word for word |
| **APIS.md** | All API connection specs, endpoints, auth methods, rate limits |
| **STORAGE.md** | Google Sheets tab structure + Google Drive folder system |
| **ANALYSIS.md** | Performance tracking rules, SM analysis, feedback loop into Stage 1 |
| **ERRORS.md** | Known issues, fixes, fallback behaviour |

---

## 5. BUILD PHASES — STEP BY STEP

The system is built in phases. Do NOT skip ahead. Each phase unlocks the next.

### PHASE 0 — Foundation (Do Once, Manually)
- [ ] Redesign WordPress site using DESIGN.md
- [ ] Install and configure all plugins per WORDPRESS.md
- [ ] Create Google Sheet with all tabs per STORAGE.md
- [ ] Create Google Drive folder structure per STORAGE.md
- [ ] Set up GitHub repository for automation scripts
- [ ] Configure GitHub Actions secrets (API keys)
- [ ] Test WordPress REST API connection

### PHASE 1 — First 10 Articles (Semi-Manual)
- [ ] Run Stage 1 manually — generate 10 topics
- [ ] Run Stage 2 manually — fetch Amazon products for each
- [ ] Run Stage 3 manually — Claude writes all 10 articles
- [ ] Run Stage 3B manually — generate all images
- [ ] Publish all 10 articles to WordPress manually
- [ ] Register all 10 in Google Sheets + Google Drive
- [ ] Apply for Amazon Associates account

### PHASE 2 — Amazon PA API Unlock
- [ ] Buy 3 products through your own affiliate links (with VPN)
- [ ] Wait for PA API approval (usually 3–7 days after 3 sales)
- [ ] Add PA API credentials to GitHub Secrets
- [ ] Test Stage 2 with live PA API call

### PHASE 3 — Full Automation (15 articles/day)
- [ ] Enable GitHub Actions cron — runs at 6:00 AM IST daily
- [ ] Stage 1 through Stage 6 all running automatically
- [ ] Google Sheets updating in real time
- [ ] Google Drive saving daily folders + Word docs
- [ ] Social posting live — Instagram + Pinterest + WhatsApp

### PHASE 4 — Intelligence Loop
- [ ] Stage 7 (Monthly SEO Loop) enabled
- [ ] Stage 8 (Email Digest) enabled
- [ ] ANALYSIS.md feedback loop reading SM + GSC data
- [ ] Analytics Feed tab updating Stage 1 topic decisions

---

## 6. AMAZON ASSOCIATES — RULES TO NEVER BREAK

These are Amazon's actual Terms of Service. Breaking them = account banned.

| Rule | Detail |
|---|---|
| Price display | Must show current price. Must update within 24 hours. |
| Image hosting | Do NOT save Amazon product images to your server. Hotlink from Amazon CDN only. |
| Price without link | Never show a price without the affiliate link next to it. |
| Star ratings | Allowed to display. Fetch via PA API only. |
| Review count | Allowed to display. Fetch via PA API only. |
| Disclosure | Every page with affiliate links must have disclosure text in footer. |
| PA API rate limit | Maximum 1 request per second. Always add 1 second wait between calls. |
| 180-day rule | Must make at least 3 qualifying sales in any 180-day window or PA API revoked. |
| Price caching | Cache prices for maximum 24 hours. Stage 5 refreshes every night — this covers it. |

---

## 7. PRODUCT QUALITY FILTERS

These filters apply at Stage 2 — Amazon PA API product research.
No article is ever written about a product that fails these filters.

| Filter | Rule |
|---|---|
| Minimum star rating | 3.8 stars or above |
| Minimum review count | 100 reviews or above |
| Availability | Must be In Stock at time of writing |
| Price display | Must show current INR price |
| Image | Must have primary product image available via PA API |

---

## 8. GOOGLE SHEETS — MASTER REFERENCE

Full tab structure is in STORAGE.md. Quick reference:

| Tab | What it tracks |
|---|---|
| Daily Keywords | Keywords researched each day |
| Daily Topics | 15 topics per day — status, blog type, subcategory |
| Content Queue | Articles written, ready to publish |
| Published | Every live article — URL, date, keyword, word count |
| Product Registry | Every Amazon product — stars, reviews, price, OOS status |
| Price History | Price changes per product over time |
| OOS Log | Out of stock events + replacement actions |
| GSC Performance | Google Search Console — clicks, impressions, position |
| SM Performance | Instagram + Pinterest + WhatsApp metrics per post |
| Product Performance | Which products get most affiliate clicks, when |
| Analytics Feed | Synthesised intelligence fed back into Stage 1 |
| SEO Loop Results | Monthly SEO findings |
| Weekly SM Report | Best performing posts each week |
| Monthly P&L | Revenue tracking (manual entry) |

---

## 9. GOOGLE DRIVE — FOLDER STRUCTURE

Full structure is in STORAGE.md. Quick reference:

```
/Baby Junctions — Content Archive/
├── /2026-04-13/
│   ├── keywords.txt
│   ├── topics.txt
│   └── /articles/
│       ├── best-baby-monitors-india.docx
│       └── ...
├── /2026-04-14/
│   └── ... (new folder auto-created every day)
└── /SEO-Reports/
    └── /2026-04/
        └── april-2026-seo-report.docx
```

---

## 10. OUT OF STOCK HANDLING

When Stage 5 detects a product is out of stock on Amazon:

1. WordPress article: Show "Currently unavailable on Amazon" badge
2. Buy button: Hide or grey out
3. Google Sheets → OOS Log: Record date, product, article URL
4. If OOS for 7+ consecutive days: Claude finds replacement product
5. Replacement must pass same quality filters (3.8+ stars, 100+ reviews)
6. Article silently updated with replacement product
7. OOS Log updated with replacement action taken

---

## 11. SOCIAL MEDIA — UTM TRACKING RULES

Every link posted on social media gets a UTM tag for tracking in Google Analytics.

```
Instagram:  ?utm_source=instagram&utm_medium=post&utm_campaign=YYYY-MM-DD&utm_content=SLUG
Pinterest:  ?utm_source=pinterest&utm_medium=pin&utm_campaign=YYYY-MM-DD&utm_content=SLUG
WhatsApp:   ?utm_source=whatsapp&utm_medium=channel&utm_campaign=YYYY-MM-DD&utm_content=SLUG
```

This tells Google Analytics exactly which platform sent which visitor.
All data flows into Google Sheets → SM Performance tab via GA4 API.

---

## 12. AUTOMATION SCHEDULE — AT A GLANCE

| Time (IST) | What Runs |
|---|---|
| 6:00 AM | Stage 1 — Topic Research (15 topics generated) |
| 6:30 AM | Stage 2 — Amazon Product Research (all 15 topics) |
| 7:00 AM | Stage 3 — Content Generation begins |
| Every 60 min from 8 AM | Stage 4 — One article published per hour |
| After each publish | Stage 3B — Images generated |
| After each publish | Stage 6 — Social posting (Instagram + Pinterest + WhatsApp) |
| 2:00 AM | Stage 5 — Nightly price + stock updater |
| Every Friday 10 AM | Stage 8 — Email digest sent |
| 1st of every month | Stage 7 — Monthly SEO Intelligence Loop |

---

## 13. ENVIRONMENT VARIABLES — WHERE TO STORE KEYS

Never hardcode API keys in any script. All keys stored in:
- **GitHub Actions:** Settings → Secrets and Variables → Actions
- **Local development:** .env file (never committed to git)

```
ANTHROPIC_API_KEY=
AMAZON_ACCESS_KEY=
AMAZON_SECRET_KEY=
AMAZON_ASSOCIATE_TAG=
WP_SITE_URL=https://babyjunctions.in
WP_USERNAME=
WP_APP_PASSWORD=
GOOGLE_SERVICE_ACCOUNT_JSON=
INSTAGRAM_PAGE_TOKEN=
INSTAGRAM_USER_ID=
PINTEREST_ACCESS_TOKEN=
MAILCHIMP_API_KEY=
MAILCHIMP_LIST_ID=
GA4_PROPERTY_ID=
```

---

## 14. FIRST ACTION WHEN OPENING THIS PROJECT

Every time Claude Code opens this project, read in this order:
1. MASTER.md (this file) — get full context
2. The relevant MD file for today's task
3. Check Google Sheets → Daily Topics tab — see what's queued
4. Proceed with the task

Never assume. Never skip reading MASTER.md first.

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
