# WORDPRESS.md — Baby Junctions
> WordPress setup rules. Scripts follow this exactly.

## HOSTING
- Host: GreenGeeks Shared | Server: LiteSpeed
- PHP: 8.1+ (set in cPanel → PHP Manager)
- SSL must be active before setup

## THEME SETUP ORDER
1. Install Kadence (free) → Activate
2. Install Child Theme Configurator → Create child theme → Activate child
3. Add CSS from DESIGN.md → Appearance → Customise → Additional CSS
4. Add fonts to child theme functions.php (from DESIGN.md)
5. Configure Kadence settings (below)

## KADENCE SETTINGS
```
Header: Logo upload | Sticky: ON | BG: #FFFFFF | Border: 2px solid #E8F4FD
Typography: Headings=Nunito | Body=Source Sans 3 | Base=16px | Line-height=1.7
Colors: Primary=#5BC8F5 | Secondary=#F26B8A | Text=#1A2A3A
Layout: Content width=1200px | Articles=full width (no sidebar)
```

## PLUGINS (install in this order)
| Plugin | Purpose |
|---|---|
| Kadence Blocks | Page builder |
| Rank Math SEO | SEO + schema |
| LiteSpeed Cache | Speed (GreenGeeks optimised) |
| WPCode Lite | Custom code injection |
| Pretty Links Lite | Affiliate link cloaking |
| UpdraftPlus | Daily backups |
| Wordfence Security | Firewall |
| Simple Author Box | E-E-A-T author bio |
| WP Crontrol | Cron management |
| Custom Post Type UI | Baby Product Review post type |
| Advanced Custom Fields | Custom fields |

**Never install:** Elementor · Divi · WP Rocket · any other SEO plugin · any other cache plugin

## RANK MATH CONFIG
```
GSC: Connect in Settings → Search Console
Schema: Enable Product, Article, FAQ, BreadcrumbList, HowTo
Sitemap: ON — include Posts, Pages, Categories. Ping Google+Bing on publish.
Title separator: |
robots.txt additions:
User-agent: GPTBot
Allow: /
User-agent: PerplexityBot
Allow: /
User-agent: ClaudeBot
Allow: /
```

## LITESPEED CACHE CONFIG
```
Cache: ON | Logged-in users: OFF
Minify: HTML+CSS+JS ON | Lazy Load Images: ON
Image Optimization: Auto ON | QUIC.cloud CDN: ON
```

## CUSTOM POST TYPE (via CPT UI)
```
Slug: baby_product_review
Labels: Baby Product Reviews / Baby Product Review
Public: true | Archive: true | Rewrite slug: reviews
Supports: title, editor, thumbnail, excerpt, custom-fields, comments
```

## CUSTOM FIELDS (ACF — apply to all Posts)
| Field | Type |
|---|---|
| amazon_asin | Text |
| amazon_affiliate_link | URL |
| amazon_price_inr | Number |
| amazon_star_rating | Number |
| amazon_review_count | Number |
| amazon_image_url | URL |
| amazon_availability | Select: IN_STOCK/OOS/UNAVAILABLE |
| price_last_updated | Date |
| primary_keyword | Text |
| blog_type | Select (21 types) |
| subcategory | Select (50 categories) |
| topic_id | Text |
| seasonal_context | Text |
| oos_replacement_asin | Text |

## REST API SETUP
```
WP Admin → Users → Profile → Application Passwords
Name: GitHub Actions Bot → Add → Copy password → save as WP_APP_PASSWORD
Base URL: https://babyjunctions.in/wp-json/wp/v2/
Test: curl -u "username:app_password" https://babyjunctions.in/wp-json/wp/v2/posts?per_page=1
```

## POST PAYLOAD (Python → WordPress)
```python
post_payload = {
    "title": article_json["seo"]["h1_title"],
    "slug": article_json["seo"]["slug"],
    "content": rendered_html,
    "excerpt": article_json["seo"]["meta_description"],
    "status": "publish",
    "categories": [category_id],
    "tags": [tag_id_1, tag_id_2],
    "featured_media": media_id,
    "meta": {
        "rank_math_focus_kw": article_json["seo"]["focus_keyword"],
        "rank_math_title": article_json["seo"]["meta_title"],
        "rank_math_description": article_json["seo"]["meta_description"],
        "rank_math_schema_type": "Article",
        "amazon_asin": products[0]["asin"],
        "amazon_affiliate_link": products[0]["affiliate_link"],
        "amazon_price_inr": products[0]["price"],
        "amazon_star_rating": products[0]["rating"],
        "amazon_review_count": products[0]["review_count"],
        "amazon_image_url": products[0]["image_url"],
        "amazon_availability": "IN_STOCK",
        "price_last_updated": today_date,
        "primary_keyword": article_json["seo"]["focus_keyword"],
        "blog_type": article_json["meta"]["blog_type"],
        "subcategory": article_json["meta"]["subcategory"],
        "topic_id": article_json["meta"]["topic_id"],
    }
}
response = requests.post(f"{WP_SITE_URL}/wp-json/wp/v2/posts",
    json=post_payload, auth=(WP_USERNAME, WP_APP_PASSWORD))
```

## IMAGE UPLOAD
```python
def upload_image(image_path, filename, alt_text):
    with open(image_path, 'rb') as img:
        r = requests.post(f"{WP_BASE}/media",
            headers={"Content-Disposition": f"attachment; filename={filename}",
                     "Content-Type": "image/png"},
            data=img.read(), auth=(WP_USERNAME, WP_APP_PASSWORD))
    media_id = r.json()["id"]
    requests.post(f"{WP_BASE}/media/{media_id}",
        json={"alt_text": alt_text}, auth=(WP_USERNAME, WP_APP_PASSWORD))
    return media_id
```

## PRICE UPDATE
```python
def update_post_price(post_id, new_price, availability, updated_html=None):
    payload = {"meta": {"amazon_price_inr": new_price,
                        "amazon_availability": availability,
                        "price_last_updated": today_date}}
    if updated_html: payload["content"] = updated_html
    return requests.post(f"{WP_BASE}/posts/{post_id}",
        json=payload, auth=(WP_USERNAME, WP_APP_PASSWORD)).status_code == 200
```

## SCHEMA (injected into every post)
```html
<script type="application/ld+json">
{"@context":"https://schema.org","@type":"Article","headline":"{title}",
"datePublished":"{date}","dateModified":"{updated}",
"author":{"@type":"Person","name":"Baby Junctions Team"},
"publisher":{"@type":"Organization","name":"Baby Junctions","url":"https://babyjunctions.in"}}
</script>
<script type="application/ld+json">
{"@context":"https://schema.org","@type":"Product","name":"{product}",
"image":"{img}","aggregateRating":{"@type":"AggregateRating",
"ratingValue":"{stars}","reviewCount":"{count}"},
"offers":{"@type":"Offer","price":"{price}","priceCurrency":"INR",
"availability":"https://schema.org/InStock","url":"{link}"}}
</script>
<script type="application/ld+json">
{"@context":"https://schema.org","@type":"FAQPage",
"mainEntity":[{"@type":"Question","name":"{q}",
"acceptedAnswer":{"@type":"Answer","text":"{a}"}}]}
</script>
```

## CATEGORIES (parent → children)
| Parent | Subcategories |
|---|---|
| Feeding & Nutrition | Bottles, Formula, Food, Nursing Pillows, Breast Pumps, Sterilizers, Warmers, Food Makers |
| Diapering | Diapers, Changing Mats, Diaper Bags |
| Sleep & Comfort | Cribs, Mattresses, Sleeping Bags, Blankets, Night Lights, White Noise, Bouncers, Swings |
| Travel | Strollers, Car Seats, Carriers |
| Health & Safety | Monitors, Thermometers, Aspirators, Humidifiers, Gates, Proofing, First Aid, Sunscreen, Mosquito Nets |
| Bath & Grooming | Bath Tubs, Shampoo, Skincare, Grooming Kits |
| Toys & Development | Teething, Toys 0-6M, Toys 6-12M, Toddler Toys, Play Mats, Books, Mobiles |
| Clothing | Clothing, Walkers, Swaddling, Pacifiers |
| Furniture & Gear | High Chairs, Maternity Support |
| Guides | Hospital Bag, Subscription Boxes |

## DISCLOSURE (auto-add via WPCode after post content)
```html
<div class="bj-disclosure">
<strong>Affiliate Disclosure:</strong> Baby Junctions participates in the Amazon Associates Program.
If you purchase through our links, we earn a small commission at no extra cost to you.
</div>
```

## STATIC PAGES (create before publishing)
About · Contact · Privacy Policy · Disclaimer · Terms of Service

## PRE-LAUNCH CHECKLIST
```
□ SSL active | □ PageSpeed mobile >70 | □ LiteSpeed active
□ Child theme active | □ Google Fonts loading | □ Disclosure on every page
□ robots.txt has LLM rules | □ Sitemap at /sitemap_index.xml
□ GSC connected | □ REST API responding | □ App Password in GitHub Secrets
□ 5 static pages live | □ Test post published via REST API
```
*v1.0 | April 2026*
