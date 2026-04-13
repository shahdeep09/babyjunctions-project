# WORDPRESS.md — Baby Junctions
## WordPress Setup, Plugins, Custom Fields & REST API

> This file defines every WordPress decision — plugins, settings, custom fields,
> and how Python scripts talk to WordPress via REST API.
> Claude Code follows this exactly. Never install plugins not listed here.

---

## 1. HOSTING CONTEXT

| Item | Detail |
|---|---|
| Host | GreenGeeks Shared Hosting |
| Server | LiteSpeed Web Server (use LiteSpeed Cache — not WP Rocket) |
| PHP | 8.1+ (set in GreenGeeks cPanel → PHP Manager) |
| MySQL | 8.0+ |
| SSL | Free via GreenGeeks — must be active before any setup |
| WordPress Version | Always latest stable |

---

## 2. THEME SETUP — DO THIS FIRST

### Step 1: Install Kadence Theme
```
WordPress Admin → Appearance → Themes → Add New → Search "Kadence" → Install → Activate
```

### Step 2: Create Child Theme (CRITICAL — do before any customisation)
Install plugin: **Child Theme Configurator** (free)
```
Plugins → Add New → Search "Child Theme Configurator" → Install → Activate
Tools → Child Themes → Select Kadence → Create Child Theme
Activate the child theme (NOT the parent)
```

### Step 3: Add CSS to Child Theme
Go to: Appearance → Customise → Additional CSS
Paste the ENTIRE CSS block from DESIGN.md Section 7.
This is the only place CSS goes. Never anywhere else.

### Step 4: Add Google Fonts to Child Theme
In GreenGeeks File Manager OR via plugin WPCode:
Edit child theme functions.php → add the font enqueue code from DESIGN.md Section 4.

### Step 5: Configure Kadence Theme Settings
```
Appearance → Customise → Kadence Settings:

Header:
- Logo: Upload Baby Junctions logo (text logo: "🍼 Baby Junctions")
- Tagline: "India ke parents ka trusted guide"
- Sticky header: ON
- Header background: #FFFFFF
- Header border bottom: 2px solid #E8F4FD

Typography:
- Headings font: Nunito (loaded via functions.php)
- Body font: Source Sans 3
- Base font size: 16px
- Line height: 1.7

Colors:
- Primary: #2E86C1
- Secondary: #E67E22
- Background: #FFFFFF
- Text: #1A2A3A

Layout:
- Content width: 1200px
- Sidebar: Right sidebar (for related posts widget)
- Blog layout: No sidebar on article pages (full width content)

Footer:
- 4 columns
- Background: #1A2A3A
- Text color: #FFFFFF
```

---

## 3. PLUGINS — INSTALL IN THIS ORDER

Install ONLY these plugins. Nothing else.

| # | Plugin | Where to Get | Purpose | Free/Paid |
|---|---|---|---|---|
| 1 | **Kadence Blocks** | WP Plugin Directory | Page builder for static pages | Free |
| 2 | **Rank Math SEO** | rankmath.com | SEO meta, schema, sitemap, GSC | Free |
| 3 | **LiteSpeed Cache** | WP Plugin Directory | Speed + caching (GreenGeeks optimised) | Free |
| 4 | **WPCode Lite** | WP Plugin Directory | Add custom code without editing theme | Free |
| 5 | **Pretty Links Lite** | WP Plugin Directory | Cloak Amazon affiliate links | Free |
| 6 | **UpdraftPlus** | WP Plugin Directory | Daily automated backups | Free |
| 7 | **Wordfence Security** | WP Plugin Directory | Firewall + malware scan | Free |
| 8 | **Simple Author Box** | WP Plugin Directory | Author bio with E-E-A-T signals | Free |
| 9 | **WP Crontrol** | WP Plugin Directory | Manage WordPress cron jobs | Free |
| 10 | **Custom Post Type UI** | WP Plugin Directory | Register Baby Product Review post type | Free |
| 11 | **Advanced Custom Fields (ACF)** | WP Plugin Directory | Add custom fields to posts | Free |

**NEVER install:**
- Elementor (conflicts with Kadence)
- Divi (too heavy)
- WP Rocket (use LiteSpeed Cache instead on GreenGeeks)
- Any SEO plugin other than Rank Math
- Any caching plugin other than LiteSpeed Cache

---

## 4. RANK MATH SEO CONFIGURATION

```
Rank Math → Dashboard → Setup Wizard:
- Connect to Google Search Console: YES
- Site type: Blog
- Focus keywords: enabled

Rank Math → Titles & Meta:
- Separator: |
- Homepage title: Best Baby Products in India 2026 | Baby Junctions
- Homepage description: India's most trusted baby product guide. Expert reviews, real prices, honest recommendations for Indian parents.

Rank Math → Schema:
- Default schema: Article
- Enable: Product Schema, FAQPage Schema, BreadcrumbList, HowTo

Rank Math → Sitemap:
- Enable XML Sitemap: YES
- Include: Posts, Pages, Categories
- Ping Google + Bing on new post: YES

Rank Math → General Settings → Edit robots.txt:
Add these lines:
User-agent: GPTBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Bingbot
Allow: /
```

---

## 5. LITESPEED CACHE CONFIGURATION

```
LiteSpeed Cache → General:
- Enable Cache: ON
- Cache Logged-in Users: OFF
- Cache Commenters: OFF

LiteSpeed Cache → Page Optimization:
- Minify HTML: ON
- Minify CSS: ON
- Minify JS: ON
- Lazy Load Images: ON
- Responsive Placeholder: ON

LiteSpeed Cache → Image Optimization:
- Auto Request Cron: ON
- Optimize on Uploading: ON

LiteSpeed Cache → CDN:
- Use QUIC.cloud CDN: ON (free with GreenGeeks)
```

---

## 6. CUSTOM POST TYPE SETUP

Use Custom Post Type UI plugin to register the Baby Product Review type.

```
CPT UI → Add/Edit Post Types:

Post Type Slug: baby_product_review
Plural Label: Baby Product Reviews
Singular Label: Baby Product Review
Public: True
Has Archive: True
Rewrite Slug: reviews
Supports: title, editor, thumbnail, excerpt, custom-fields, comments
Taxonomies: category, post_tag
```

---

## 7. CUSTOM FIELDS — COMPLETE LIST

Use Advanced Custom Fields (ACF) to register these fields.
Field Group Name: **Baby Product Data**
Location: Post Type is equal to Post (applies to all posts + baby_product_review)

| Field Name | Field Type | Description |
|---|---|---|
| amazon_asin | Text | Amazon product ASIN |
| amazon_affiliate_link | URL | Full affiliate link with tag |
| amazon_price_inr | Number | Current price in INR |
| amazon_star_rating | Number | Star rating (e.g. 4.3) |
| amazon_review_count | Number | Total review count |
| amazon_image_url | URL | Amazon CDN image URL (hotlink only) |
| amazon_availability | Select | IN_STOCK / OOS / UNAVAILABLE |
| price_last_updated | Date | Date price was last refreshed |
| primary_keyword | Text | Main SEO keyword for this post |
| blog_type | Select | One of 21 blog types |
| subcategory | Select | One of 50 baby subcategories |
| topic_id | Text | Links back to Google Sheets topic ID |
| seasonal_context | Text | Why this article is relevant this season |
| oos_replacement_asin | Text | Replacement product ASIN if OOS |

---

## 8. WORDPRESS REST API SETUP

### Enable Application Passwords (One-Time Setup)
```
WordPress Admin → Users → Your Profile → scroll to "Application Passwords"
Name: GitHub Actions Bot
Click "Add New Application Password"
Copy the password — save in GitHub Secrets as WP_APP_PASSWORD
```

### REST API Base URL
```
https://babyjunctions.in/wp-json/wp/v2/
```

### Test the Connection
```bash
curl -u "your_username:your_app_password" \
  https://babyjunctions.in/wp-json/wp/v2/posts?per_page=1
```
If this returns JSON, REST API is working.

---

## 9. REST API — COMPLETE POST PAYLOAD

This is the exact JSON Python sends to WordPress to publish an article.

```python
post_payload = {
    # CORE CONTENT
    "title": article_json["seo"]["h1_title"],
    "slug": article_json["seo"]["slug"],
    "content": rendered_html,          # Full HTML rendered from JSON
    "excerpt": article_json["seo"]["meta_description"],
    "status": "publish",               # or "draft" during testing

    # TAXONOMY
    "categories": [category_id],       # Mapped from subcategory
    "tags": [tag_id_1, tag_id_2],

    # FEATURED IMAGE
    "featured_media": media_id,        # ID after uploading thumbnail

    # SEO FIELDS (Rank Math)
    "meta": {
        "rank_math_focus_kw":      article_json["seo"]["focus_keyword"],
        "rank_math_title":         article_json["seo"]["meta_title"],
        "rank_math_description":   article_json["seo"]["meta_description"],
        "rank_math_schema_type":   "Article",

        # CUSTOM FIELDS (ACF)
        "amazon_asin":             products[0]["asin"],
        "amazon_affiliate_link":   products[0]["affiliate_link"],
        "amazon_price_inr":        products[0]["price"],
        "amazon_star_rating":      products[0]["rating"],
        "amazon_review_count":     products[0]["review_count"],
        "amazon_image_url":        products[0]["image_url"],
        "amazon_availability":     "IN_STOCK",
        "price_last_updated":      today_date,
        "primary_keyword":         article_json["seo"]["focus_keyword"],
        "blog_type":               article_json["meta"]["blog_type"],
        "subcategory":             article_json["meta"]["subcategory"],
        "topic_id":                article_json["meta"]["topic_id"],
        "seasonal_context":        article_json["meta"]["seasonal_angle"],
    }
}

# Send to WordPress
response = requests.post(
    f"{WP_SITE_URL}/wp-json/wp/v2/posts",
    json=post_payload,
    auth=(WP_USERNAME, WP_APP_PASSWORD),
    headers={"Content-Type": "application/json"}
)
```

---

## 10. UPLOADING IMAGES TO WORDPRESS

Images must be uploaded to Media Library before publishing the post.

```python
def upload_image_to_wordpress(image_path, image_name, alt_text):
    with open(image_path, 'rb') as img:
        response = requests.post(
            f"{WP_SITE_URL}/wp-json/wp/v2/media",
            headers={
                "Content-Disposition": f"attachment; filename={image_name}",
                "Content-Type": "image/png",
            },
            data=img.read(),
            auth=(WP_USERNAME, WP_APP_PASSWORD)
        )
    media_id = response.json()["id"]

    # Set alt text
    requests.post(
        f"{WP_SITE_URL}/wp-json/wp/v2/media/{media_id}",
        json={"alt_text": alt_text, "caption": alt_text},
        auth=(WP_USERNAME, WP_APP_PASSWORD)
    )
    return media_id
```

---

## 11. UPDATING AN EXISTING POST (Price + OOS Updates)

Stage 5 uses this to silently update prices nightly.

```python
def update_post_price(post_id, new_price, availability, updated_html=None):
    update_payload = {
        "meta": {
            "amazon_price_inr":     new_price,
            "amazon_availability":  availability,
            "price_last_updated":   today_date,
        }
    }
    if updated_html:
        update_payload["content"] = updated_html

    response = requests.post(
        f"{WP_SITE_URL}/wp-json/wp/v2/posts/{post_id}",
        json=update_payload,
        auth=(WP_USERNAME, WP_APP_PASSWORD)
    )
    return response.status_code == 200
```

---

## 12. SCHEMA MARKUP — INJECTED PER BLOG TYPE

Claude generates this JSON-LD. Python injects it into the post content before publishing.

```html
<!-- Injected at bottom of every post content -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{article_title}",
  "datePublished": "{published_date}",
  "dateModified": "{last_updated_date}",
  "author": {
    "@type": "Person",
    "name": "Baby Junctions Team"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Baby Junctions",
    "url": "https://babyjunctions.in"
  }
}
</script>

<!-- Product schema for Top 10 / Review articles -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "{product_title}",
  "image": "{amazon_image_url}",
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "{star_rating}",
    "reviewCount": "{review_count}"
  },
  "offers": {
    "@type": "Offer",
    "price": "{price_inr}",
    "priceCurrency": "INR",
    "availability": "https://schema.org/InStock",
    "url": "{amazon_affiliate_link}"
  }
}
</script>

<!-- FAQ schema — every article -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "{question_1}",
      "acceptedAnswer": { "@type": "Answer", "text": "{answer_1}" }
    }
  ]
}
</script>
```

---

## 13. CATEGORY SETUP

Create these WordPress categories before automation starts.
Map each of the 50 subcategories to one of these parent categories.

| Parent Category | Subcategories Under It |
|---|---|
| Feeding & Nutrition | Baby Bottles, Baby Formula, Baby Food, Nursing Pillows, Breast Pumps, Bottle Sterilizers, Bottle Warmers, Baby Food Makers |
| Diapering | Baby Diapers, Changing Mats, Diaper Bags |
| Sleep & Comfort | Baby Cribs, Baby Mattresses, Baby Sleeping Bags, Baby Blankets, Baby Night Lights, White Noise Machines, Baby Bouncers, Baby Swings |
| Travel & Outdoors | Baby Strollers, Baby Car Seats, Baby Carriers |
| Health & Safety | Baby Monitors, Baby Thermometers, Nasal Aspirators, Baby Humidifiers, Baby Safety Gates, Baby Proofing, Baby First Aid, Baby Sunscreen, Baby Mosquito Nets |
| Bath & Grooming | Baby Bath Tubs, Baby Shampoo, Baby Skincare, Baby Grooming Kits |
| Toys & Development | Teething Toys, Toys 0-6M, Toys 6-12M, Toddler Toys, Baby Play Mats, Baby Books, Baby Crib Mobiles |
| Clothing & Fashion | Baby Clothing, Baby Walkers, Baby Swaddling, Baby Pacifiers |
| Furniture & Gear | Baby High Chairs, Baby Changing Mats, Maternity Support |
| Shopping Guides | Hospital Bag Essentials, Baby Subscription Boxes |

---

## 14. AFFILIATE DISCLOSURE — AUTO-ADDED TO EVERY POST

Add this via WPCode plugin → Auto Insert → After Post Content:

```html
<div class="bj-disclosure">
  <strong>Affiliate Disclosure:</strong> Baby Junctions participates in the Amazon
  Associates Program, an affiliate advertising program designed to provide a means
  for sites to earn advertising fees by advertising and linking to Amazon.in.
  If you purchase through our links, we earn a small commission at no extra cost to you.
  We only recommend products we genuinely believe are good for Indian babies and parents.
</div>
```

---

## 15. PRETTY LINKS SETUP — LINK CLOAKING

All Amazon affiliate links get cloaked through Pretty Links.
This makes links look clean and lets you change a destination without editing posts.

```
Amazon link: https://www.amazon.in/dp/B09XYZ1234?tag=babyjunctions-21
Pretty Link: https://babyjunctions.in/go/philips-avent-monitor
```

Python script creates Pretty Links via REST API after publishing:
```python
# Pretty Links has its own REST endpoint
requests.post(
    f"{WP_SITE_URL}/wp-json/pretty-links/v1/links",
    json={
        "url": amazon_affiliate_url,
        "slug": product_slug,
        "name": product_title,
    },
    auth=(WP_USERNAME, WP_APP_PASSWORD)
)
```

---

## 16. STATIC PAGES TO CREATE (One-Time, Manual)

Before publishing any articles, create these pages:

| Page | Slug | Content |
|---|---|---|
| About Us | /about | Who Baby Junctions is, mission, team |
| Contact | /contact | Simple contact form |
| Privacy Policy | /privacy-policy | Required for AdSense + Amazon |
| Disclaimer | /disclaimer | Affiliate + medical disclaimer |
| Terms of Service | /terms | Standard ToS |

These pages are required for:
- Amazon Associates approval
- Google AdSense approval
- General trust signals (E-E-A-T)

---

## 17. PERFORMANCE CHECKLIST — BEFORE GOING LIVE

Run these checks after full setup:

```
□ SSL certificate active (https:// loads without warning)
□ Google PageSpeed score > 70 on mobile
□ LiteSpeed Cache active and configured
□ All plugins active and no conflicts
□ Child theme active (not parent)
□ Google Fonts loading correctly (no console errors)
□ Amazon affiliate disclosure visible on every page
□ robots.txt includes LLM crawler rules
□ XML sitemap accessible at /sitemap_index.xml
□ Google Search Console connected + sitemap submitted
□ REST API responding correctly (test with curl)
□ Application Password saved in GitHub Secrets
□ 5 static pages live (About, Contact, Privacy, Disclaimer, Terms)
□ Pretty Links plugin active
□ At least 1 test post published via REST API successfully
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
