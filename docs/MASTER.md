# MASTER.md — Baby Junctions
> Read this file first. Always. Everything else references this.

## PROJECT
| Field | Value |
|---|---|
| Brand | Baby Junctions |
| Domain | babyjunctions.in |
| Tagline | India ke parents ka trusted baby guide |
| Niche | Baby Products — India |
| Language | English + slight Hinglish |
| Audience | Indian parents, 25–50 yrs, tier 1/2/3 cities |
| Standard | BabyGearLab depth |
| Affiliate | Amazon Associates India (PA API 5.0) |
| CMS | WordPress on GreenGeeks |
| Automation | GitHub Actions + Python |
| AI | Claude API (claude-sonnet-4-20250514) |

## TECH STACK
| Tool | Role |
|---|---|
| GreenGeeks | WordPress hosting |
| Kadence Theme | CMS frontend |
| GitHub Actions | Free daily cron |
| Claude API | Content + strategy |
| Amazon PA API | Products + prices + images |
| Google Sheets | Live pipeline dashboard |
| Google Drive | Daily folders + Word docs |
| Python/Pillow | Image generation |
| Rank Math SEO | SEO + schema |
| Mailchimp | Email digest |
| Instagram/Pinterest/WhatsApp | Social automation |

## MD FILES
| File | Purpose |
|---|---|
| MASTER.md | This file |
| DESIGN.md | Colors, fonts, CSS, components |
| WORDPRESS.md | Plugins, fields, REST API |
| CONTENT.md | Voice, 21 blog types, JSON schema |
| WORKFLOW.md | 9-stage pipeline |
| PROMPTS.md | All Claude prompts |
| APIS.md | All API specs |
| STORAGE.md | Sheets tabs + Drive folders |
| ANALYSIS.md | Performance + feedback loop |
| ERRORS.md | Fixes + fallbacks |

## BUILD PHASES
**Phase 0:** WordPress redesign + plugin setup + Sheets/Drive structure
**Phase 1:** First 10 articles semi-manually → apply for Amazon Associates
**Phase 2:** PA API approved → buy 3 products via own links → full automation
**Phase 3:** 15 articles/day running → social posting live
**Phase 4:** Monthly SEO loop + Analytics Feed feedback active

## AMAZON RULES (Never Break)
- Show current price — refresh within 24hrs
- Never save Amazon images to server — hotlink CDN only
- Never show price without affiliate link
- PA API: 1 request/second max
- Disclosure on every page with affiliate links

## PRODUCT FILTERS (Stage 2)
- Stars: 3.8+ minimum
- Reviews: 100+ minimum
- Status: In Stock only

## GOOGLE SHEETS TABS (Quick Ref)
Daily Keywords · Daily Topics · Content Queue · Published · Product Registry · Price History · OOS Log · GSC Performance · SM Performance · Product Performance · Analytics Feed · SEO Loop Results · Weekly SM Report · Monthly P&L

## GOOGLE DRIVE STRUCTURE
```
/Baby Junctions — Content Archive/
├── /Daily Content/YYYY-MM-DD/
│   ├── keywords.txt
│   ├── topics.txt
│   └── /articles/*.docx
└── /SEO-Reports/YYYY-MM/report.docx
```

## SCHEDULE
| Time (IST) | Stage |
|---|---|
| 6:00 AM | Stage 1 — Topics |
| 6:30 AM | Stage 2 — Products |
| 7:00 AM | Stage 3 — Content |
| 8AM–11PM | Stage 4 — Publish (1/hr) |
| After each publish | Stage 6 — Social |
| 2:00 AM | Stage 5 — Prices |
| Every Friday 10AM | Stage 8 — Email |
| 1st of month | Stage 7 — SEO Loop |

## ENV VARIABLES
Store in GitHub Secrets (production) and `.env` file (local only — never commit).
```
ANTHROPIC_API_KEY · AMAZON_ACCESS_KEY · AMAZON_SECRET_KEY · AMAZON_ASSOCIATE_TAG
WP_SITE_URL · WP_USERNAME · WP_APP_PASSWORD · GOOGLE_SERVICE_ACCOUNT_JSON
GOOGLE_SHEETS_ID · GOOGLE_SHEETS_NAME · GOOGLE_DRIVE_ROOT_FOLDER_ID
INSTAGRAM_PAGE_TOKEN · INSTAGRAM_USER_ID · PINTEREST_ACCESS_TOKEN
MAILCHIMP_API_KEY · MAILCHIMP_LIST_ID · GA4_PROPERTY_ID
```

## OOS HANDLING
1. Show "Currently unavailable" badge → hide buy button
2. Log to OOS Log tab
3. After 7 days OOS → Claude finds replacement (same filters)
4. Article silently updated

## UTM TRACKING
```
Instagram:  ?utm_source=instagram&utm_medium=post&utm_campaign=YYYY-MM-DD
Pinterest:  ?utm_source=pinterest&utm_medium=pin&utm_campaign=YYYY-MM-DD
WhatsApp:   ?utm_source=whatsapp&utm_medium=channel&utm_campaign=YYYY-MM-DD
```
*v1.0 | April 2026*
