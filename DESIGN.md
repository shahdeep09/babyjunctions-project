# DESIGN.md — Baby Junctions
## The Complete Visual System

> This file defines every visual decision for babyjunctions.in.
> Claude never deviates from this system. Ever.
> If a new component is needed, add it here first, then build it.

---

## 1. WHY THIS FILE EXISTS

The previous WordPress site had broken design because there was no locked system.
Code appeared in headers. Layouts broke on mobile. Fonts were inconsistent.

This file fixes that permanently. Every visual decision is made here ONCE.
Scripts and WordPress templates follow this — they never invent their own styles.

---

## 2. WORDPRESS THEME

**Theme: Kadence (Free)**
- Download from: wordpress.org/themes/kadence
- Why: Fastest free theme, full block editor support, no bloat, mobile-first
- Child theme: Create a Kadence child theme BEFORE any customisation
- Never edit the parent theme directly

**Page Builder: Kadence Blocks (Free plugin)**
- All custom layouts use Kadence Blocks only
- Never install Elementor — conflicts with Kadence and adds weight
- Never install Divi — too heavy for shared hosting

---

## 3. COLOR PALETTE

These are the ONLY colors used across the entire site.

```
PRIMARY BLUE      #2E86C1   — Main buttons, links, accents, headers
LIGHT BLUE BG     #E8F4FD   — Page backgrounds, card backgrounds
DARK NAVY         #1A2A3A   — Primary text, headings
WHITE             #FFFFFF   — Cards, content backgrounds
WARM ORANGE       #E67E22   — Call-to-action buttons, urgency badges
GOLD BADGE        #F39C12   — Price badges, "Best Pick" labels
SUCCESS GREEN     #27AE60   — In stock, verified, safe badges
WARNING RED       #E74C3C   — Out of stock, warning, avoid badges
LIGHT GREY        #F4F6F7   — Dividers, subtle backgrounds
MID GREY          #7F8C8D   — Secondary text, metadata
DARK STRIP        #154360   — Footer, image bottom strips
```

**CSS Variables (add to child theme style.css):**
```css
:root {
  --bj-primary:     #2E86C1;
  --bj-primary-bg:  #E8F4FD;
  --bj-dark:        #1A2A3A;
  --bj-white:       #FFFFFF;
  --bj-cta:         #E67E22;
  --bj-gold:        #F39C12;
  --bj-green:       #27AE60;
  --bj-red:         #E74C3C;
  --bj-grey-light:  #F4F6F7;
  --bj-grey-mid:    #7F8C8D;
  --bj-strip:       #154360;
}
```

---

## 4. TYPOGRAPHY

**Font Pair:**
- Display / Headings: **Nunito** (Google Fonts) — friendly, rounded, professional
- Body text: **Source Sans 3** (Google Fonts) — clean, highly readable at small sizes

**Why these fonts:**
Nunito feels warm and approachable — right for Indian parent audience.
Source Sans 3 is extremely readable on mobile — critical for India where 80%+ traffic is mobile.

**Load in WordPress (add to child theme functions.php):**
```php
function babyjunctions_fonts() {
    wp_enqueue_style('bj-fonts',
        'https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=Source+Sans+3:wght@400;600&display=swap',
        array(), null
    );
}
add_action('wp_enqueue_scripts', 'babyjunctions_fonts');
```

**Type Scale:**
```
H1 Article Title:    32px / Nunito Bold 700 / line-height 1.3 / color: #1A2A3A
H2 Section Headers:  24px / Nunito SemiBold 700 / color: #1A2A3A
H3 Subsections:      20px / Nunito SemiBold 600 / color: #1A2A3A
Body Text:           16px / Source Sans 3 Regular / line-height 1.7 / color: #1A2A3A
Small/Meta:          13px / Source Sans 3 Regular / color: #7F8C8D
CTA Button Text:     15px / Nunito Bold 700 / uppercase / letter-spacing 0.5px
Badge Text:          12px / Nunito Bold 700 / uppercase
```

---

## 5. SPACING SYSTEM

Everything uses multiples of 8px. No random spacing.

```
4px   — micro gap (between badge and text)
8px   — small gap (between icon and label)
16px  — component padding (inside cards)
24px  — section gap (between card sections)
32px  — large gap (between page sections)
48px  — section breathing room (above/below major sections)
64px  — hero sections
```

---

## 6. COMPONENTS

### 6.1 — Product Card (The Most Important Component)

Every article has product cards. This is the exact HTML structure.
WordPress template renders this from JSON data. Never change this structure.

```html
<div class="bj-product-card">

  <!-- Product Number Badge -->
  <div class="bj-product-rank">#1</div>

  <!-- Product Image (Amazon CDN hotlink — never save locally) -->
  <div class="bj-product-image">
    <img src="{amazon_image_url}" alt="{product_title}" loading="lazy" />
  </div>

  <!-- Product Info -->
  <div class="bj-product-info">

    <!-- Title -->
    <h3 class="bj-product-title">{product_title}</h3>

    <!-- Star Rating -->
    <div class="bj-stars">
      <span class="bj-star-visual">★★★★☆</span>
      <span class="bj-star-number">{rating} out of 5</span>
      <span class="bj-review-count">({review_count} reviews)</span>
    </div>

    <!-- Price -->
    <div class="bj-price-box">
      <span class="bj-current-price">₹{current_price}</span>
      <span class="bj-price-note">on Amazon.in</span>
    </div>

    <!-- Out of Stock Badge (shown only when OOS) -->
    <div class="bj-oos-badge" style="display:none;">
      ⚠️ Currently unavailable on Amazon
    </div>

    <!-- Why We Picked This -->
    <div class="bj-why-picked">
      <strong>Why we picked this:</strong> {why_picked_text}
    </div>

    <!-- Pros & Cons -->
    <div class="bj-pros-cons">
      <div class="bj-pros">
        <strong>✅ Pros</strong>
        <ul>{pros_list}</ul>
      </div>
      <div class="bj-cons">
        <strong>⚠️ Cons</strong>
        <ul>{cons_list}</ul>
      </div>
    </div>

    <!-- Buy Button -->
    <a href="{amazon_affiliate_link}" class="bj-buy-button" target="_blank" rel="nofollow noopener">
      🛒 Check Price on Amazon →
    </a>

    <!-- Price Last Updated -->
    <div class="bj-price-updated">
      Price last checked: {last_updated_date}
    </div>

  </div>
</div>
```

### 6.2 — Quick Answer Box (Top of Every Article)

```html
<div class="bj-quick-answer">
  <div class="bj-quick-answer-label">⚡ Quick Answer</div>
  <p>{2-3 sentence direct answer to the article's main question}</p>
</div>
```

### 6.3 — Expert Pick Badge

```html
<div class="bj-expert-pick">
  🏆 Baby Junctions Expert Pick — {month} {year}
</div>
```

### 6.4 — Seasonal Alert Banner

```html
<div class="bj-seasonal-alert">
  🌸 {seasonal_context} — Updated for {season} {year}
</div>
```

### 6.5 — Affiliate Disclosure (Every Article Footer)

```html
<div class="bj-disclosure">
  <strong>Disclosure:</strong> Baby Junctions is a participant in the Amazon Associates
  Program. If you buy through our links, we may earn a small commission —
  at no extra cost to you. We only recommend products we genuinely think are
  good for Indian babies and parents.
</div>
```

### 6.6 — FAQ Section

```html
<div class="bj-faq">
  <h2>Frequently Asked Questions</h2>
  <div class="bj-faq-item">
    <div class="bj-faq-question">{question}</div>
    <div class="bj-faq-answer">{answer}</div>
  </div>
</div>
```

---

## 7. CSS FOR ALL COMPONENTS

Add this to child theme style.css:

```css
/* ── PRODUCT CARD ─────────────────────────────── */
.bj-product-card {
  background: var(--bj-white);
  border: 1px solid #DCE8F0;
  border-radius: 12px;
  padding: 24px;
  margin: 24px 0;
  position: relative;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}
.bj-product-rank {
  position: absolute;
  top: -12px;
  left: 20px;
  background: var(--bj-primary);
  color: var(--bj-white);
  font-family: 'Nunito', sans-serif;
  font-weight: 800;
  font-size: 13px;
  padding: 4px 12px;
  border-radius: 20px;
}
.bj-product-image img {
  max-width: 200px;
  height: auto;
  display: block;
  margin: 0 auto 16px;
}
.bj-product-title {
  font-family: 'Nunito', sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: var(--bj-dark);
  margin: 0 0 8px 0;
}
.bj-stars {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 8px 0;
}
.bj-star-visual {
  color: var(--bj-gold);
  font-size: 16px;
}
.bj-star-number {
  font-weight: 700;
  color: var(--bj-dark);
  font-size: 14px;
}
.bj-review-count {
  color: var(--bj-grey-mid);
  font-size: 13px;
}
.bj-price-box {
  margin: 12px 0;
  display: flex;
  align-items: baseline;
  gap: 8px;
}
.bj-current-price {
  font-family: 'Nunito', sans-serif;
  font-size: 24px;
  font-weight: 800;
  color: var(--bj-primary);
}
.bj-price-note {
  font-size: 13px;
  color: var(--bj-grey-mid);
}
.bj-oos-badge {
  background: #FDEDEC;
  border: 1px solid var(--bj-red);
  color: var(--bj-red);
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  margin: 8px 0;
}
.bj-why-picked {
  background: var(--bj-primary-bg);
  border-left: 3px solid var(--bj-primary);
  padding: 12px 16px;
  border-radius: 0 8px 8px 0;
  font-size: 14px;
  margin: 16px 0;
  line-height: 1.6;
}
.bj-pros-cons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin: 16px 0;
}
.bj-pros, .bj-cons {
  font-size: 14px;
  line-height: 1.7;
}
.bj-pros ul, .bj-cons ul {
  margin: 8px 0 0 0;
  padding-left: 16px;
}
.bj-buy-button {
  display: block;
  background: var(--bj-cta);
  color: var(--bj-white) !important;
  text-align: center;
  padding: 14px 24px;
  border-radius: 8px;
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  font-size: 15px;
  text-decoration: none !important;
  letter-spacing: 0.3px;
  margin: 16px 0 8px 0;
  transition: background 0.2s;
}
.bj-buy-button:hover {
  background: #D35400;
}
.bj-price-updated {
  font-size: 12px;
  color: var(--bj-grey-mid);
  text-align: right;
}

/* ── QUICK ANSWER BOX ─────────────────────────── */
.bj-quick-answer {
  background: linear-gradient(135deg, #E8F4FD, #D4E6F1);
  border: 1px solid var(--bj-primary);
  border-radius: 10px;
  padding: 20px 24px;
  margin: 0 0 32px 0;
}
.bj-quick-answer-label {
  font-family: 'Nunito', sans-serif;
  font-weight: 800;
  font-size: 12px;
  color: var(--bj-primary);
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 8px;
}
.bj-quick-answer p {
  font-size: 16px;
  line-height: 1.7;
  margin: 0;
  color: var(--bj-dark);
}

/* ── EXPERT PICK BADGE ────────────────────────── */
.bj-expert-pick {
  display: inline-block;
  background: var(--bj-gold);
  color: var(--bj-dark);
  font-family: 'Nunito', sans-serif;
  font-weight: 800;
  font-size: 13px;
  padding: 6px 14px;
  border-radius: 20px;
  margin: 0 0 16px 0;
}

/* ── SEASONAL ALERT ───────────────────────────── */
.bj-seasonal-alert {
  background: #FEF9E7;
  border: 1px solid #F39C12;
  border-radius: 8px;
  padding: 12px 16px;
  font-size: 14px;
  font-weight: 600;
  color: #7D6608;
  margin: 0 0 24px 0;
}

/* ── DISCLOSURE ───────────────────────────────── */
.bj-disclosure {
  background: var(--bj-grey-light);
  border-radius: 8px;
  padding: 16px;
  font-size: 13px;
  color: var(--bj-grey-mid);
  margin: 40px 0 24px 0;
  line-height: 1.6;
}

/* ── FAQ ──────────────────────────────────────── */
.bj-faq {
  margin: 32px 0;
}
.bj-faq-item {
  border-bottom: 1px solid #DCE8F0;
  padding: 16px 0;
}
.bj-faq-question {
  font-family: 'Nunito', sans-serif;
  font-weight: 700;
  font-size: 16px;
  color: var(--bj-dark);
  margin-bottom: 8px;
}
.bj-faq-answer {
  font-size: 15px;
  color: var(--bj-dark);
  line-height: 1.7;
}

/* ── MOBILE RESPONSIVE ────────────────────────── */
@media (max-width: 768px) {
  .bj-pros-cons {
    grid-template-columns: 1fr;
  }
  .bj-product-card {
    padding: 16px;
  }
  .bj-current-price {
    font-size: 20px;
  }
  h1 { font-size: 24px; }
  h2 { font-size: 20px; }
}
@media (max-width: 480px) {
  .bj-product-image img {
    max-width: 140px;
  }
  .bj-buy-button {
    font-size: 14px;
    padding: 12px 16px;
  }
}
```

---

## 8. IMAGE DIMENSIONS (Python/Pillow Generator)

| Image Type | Dimensions | Used For |
|---|---|---|
| Featured Thumbnail | 1200×628px | WordPress featured image, OG tag |
| Article Banner | 1200×400px | Top of article |
| Pinterest Pin | 1000×1500px | Pinterest posts |
| Instagram Square | 1080×1080px | Instagram posts |
| Mini Thumbnail | 400×300px | Related posts widget |

All images use the color palette defined in Section 3.
Background: #E8F4FD | Accent: #2E86C1 | Text: #1A2A3A | Badge: #F39C12

---

## 9. HEADER & NAVIGATION

```
Logo: "🍼 Baby Junctions" — Nunito 800, #1A2A3A
Tagline below logo: "India ke parents ka trusted guide" — Source Sans 3, 13px, #7F8C8D

Navigation items:
- Best Products
- By Age (dropdown: Newborn, 0-6M, 6-12M, Toddler)
- By Category (dropdown: 50 subcategories)
- Seasonal Guides
- About Us

Header background: #FFFFFF
Header border-bottom: 2px solid #E8F4FD
Sticky header: Yes — stays at top on scroll
Mobile: Hamburger menu
```

---

## 10. FOOTER

```
Column 1: Logo + tagline + brief about (2 lines)
Column 2: Quick links (Best Products, Safety Guides, Gift Ideas, About)
Column 3: Categories (top 8 subcategories)
Column 4: Newsletter signup box

Footer background: #1A2A3A (dark navy)
Footer text: #FFFFFF and #AED6F1
Footer bottom bar: #154360 with copyright + Amazon disclosure
```

---

## 11. WHAT NEVER TO DO

These broke the previous site. Never repeat:

- ❌ Never install Elementor alongside Kadence
- ❌ Never edit parent theme files directly
- ❌ Never add custom CSS in the wrong place (always child theme style.css)
- ❌ Never use Google Fonts @import in HTML — use functions.php enqueue
- ❌ Never save Amazon product images to server — hotlink only
- ❌ Never use inline styles in article HTML — always use CSS classes
- ❌ Never paste raw PHP/JavaScript into WordPress posts — use WPCode plugin
- ❌ Never use pixel-fixed widths for images — always max-width + height:auto
- ❌ Never skip the child theme — updates will wipe all customisations

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
