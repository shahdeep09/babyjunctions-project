# DESIGN.md — Baby Junctions
> Defines every visual decision. Never deviate. Update here first, then build.

## THEME
- **Theme:** Kadence (free) — never edit parent directly
- **Child theme:** Create via Child Theme Configurator before any customisation
- **Builder:** Kadence Blocks only — never Elementor, never Divi

## COLORS
```css
:root {
  --bj-primary:     #5BC8F5;  /* Baby blue — logo match */
  --bj-secondary:   #F26B8A;  /* Coral pink — logo match */
  --bj-dark:        #1A2A3A;  /* Body text */
  --bj-white:       #FFFFFF;
  --bj-cta:         #E67E22;  /* CTA buttons */
  --bj-gold:        #F39C12;  /* Price badges */
  --bj-green:       #27AE60;  /* In stock */
  --bj-red:         #E74C3C;  /* OOS / warning */
  --bj-grey-light:  #F4F6F7;
  --bj-grey-mid:    #7F8C8D;
  --bj-strip:       #154360;  /* Footer */
  --bj-primary-bg:  #E8F4FD;  /* Page backgrounds */
}
```

## TYPOGRAPHY
```php
// Add to child theme functions.php
function babyjunctions_fonts() {
    wp_enqueue_style('bj-fonts',
        'https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800&family=Source+Sans+3:wght@400;600&display=swap',
        array(), null);
}
add_action('wp_enqueue_scripts', 'babyjunctions_fonts');
```
| Element | Size | Font | Weight |
|---|---|---|---|
| H1 | 32px | Nunito | 700 |
| H2 | 24px | Nunito | 700 |
| H3 | 20px | Nunito | 600 |
| Body | 16px | Source Sans 3 | 400 |
| Small | 13px | Source Sans 3 | 400 |
| Button | 15px | Nunito | 700 |

## SPACING
8px base unit. Use multiples: 4 · 8 · 16 · 24 · 32 · 48 · 64px

## COMPONENTS

### Product Card HTML
```html
<div class="bj-product-card">
  <div class="bj-product-rank">#1</div>
  <div class="bj-product-image">
    <img src="{amazon_image_url}" alt="{title}" loading="lazy" />
  </div>
  <div class="bj-product-info">
    <h3 class="bj-product-title">{title}</h3>
    <div class="bj-stars">
      <span class="bj-star-visual">★★★★☆</span>
      <span class="bj-star-number">{rating}/5</span>
      <span class="bj-review-count">({reviews} reviews)</span>
    </div>
    <div class="bj-price-box">
      <span class="bj-current-price">₹{price}</span>
      <span class="bj-price-note">on Amazon.in</span>
    </div>
    <div class="bj-oos-badge" style="display:none;">⚠️ Currently unavailable</div>
    <div class="bj-why-picked"><strong>Why we picked:</strong> {text}</div>
    <div class="bj-pros-cons">
      <div class="bj-pros"><strong>✅ Pros</strong><ul>{pros}</ul></div>
      <div class="bj-cons"><strong>⚠️ Cons</strong><ul>{cons}</ul></div>
    </div>
    <a href="{link}" class="bj-buy-button" target="_blank" rel="nofollow noopener">🛒 Check Price on Amazon →</a>
    <div class="bj-price-updated">Updated: {date}</div>
  </div>
</div>
```

### Other Components
```html
<!-- Quick Answer -->
<div class="bj-quick-answer">
  <div class="bj-quick-answer-label">⚡ Quick Answer</div>
  <p>{2-3 sentence answer}</p>
</div>

<!-- Seasonal Banner -->
<div class="bj-seasonal-alert">🌸 {seasonal context}</div>

<!-- Expert Pick -->
<div class="bj-expert-pick">🏆 Baby Junctions Expert Pick — {month} {year}</div>

<!-- Disclosure -->
<div class="bj-disclosure">
  <strong>Disclosure:</strong> Baby Junctions earns from Amazon affiliate links at no extra cost to you.
</div>

<!-- FAQ -->
<div class="bj-faq-item">
  <div class="bj-faq-question">{question}</div>
  <div class="bj-faq-answer">{answer}</div>
</div>
```

## CSS (Add to child theme style.css)
```css
.bj-product-card { background:#fff; border:1px solid #DCE8F0; border-radius:12px; padding:24px; margin:24px 0; position:relative; box-shadow:0 2px 8px rgba(0,0,0,0.06); }
.bj-product-rank { position:absolute; top:-12px; left:20px; background:var(--bj-primary); color:#fff; font-family:'Nunito',sans-serif; font-weight:800; font-size:13px; padding:4px 12px; border-radius:20px; }
.bj-product-image img { max-width:200px; height:auto; display:block; margin:0 auto 16px; }
.bj-product-title { font-family:'Nunito',sans-serif; font-size:18px; font-weight:700; color:var(--bj-dark); margin:0 0 8px; }
.bj-stars { display:flex; align-items:center; gap:8px; margin:8px 0; }
.bj-star-visual { color:var(--bj-gold); font-size:16px; }
.bj-star-number { font-weight:700; font-size:14px; }
.bj-review-count { color:var(--bj-grey-mid); font-size:13px; }
.bj-price-box { margin:12px 0; display:flex; align-items:baseline; gap:8px; }
.bj-current-price { font-family:'Nunito',sans-serif; font-size:24px; font-weight:800; color:var(--bj-primary); }
.bj-price-note { font-size:13px; color:var(--bj-grey-mid); }
.bj-oos-badge { background:#FDEDEC; border:1px solid var(--bj-red); color:var(--bj-red); padding:8px 12px; border-radius:6px; font-size:13px; font-weight:600; margin:8px 0; }
.bj-why-picked { background:var(--bj-primary-bg); border-left:3px solid var(--bj-primary); padding:12px 16px; border-radius:0 8px 8px 0; font-size:14px; margin:16px 0; line-height:1.6; }
.bj-pros-cons { display:grid; grid-template-columns:1fr 1fr; gap:16px; margin:16px 0; font-size:14px; line-height:1.7; }
.bj-buy-button { display:block; background:var(--bj-cta); color:#fff!important; text-align:center; padding:14px 24px; border-radius:8px; font-family:'Nunito',sans-serif; font-weight:700; font-size:15px; text-decoration:none!important; margin:16px 0 8px; transition:background 0.2s; }
.bj-buy-button:hover { background:#D35400; }
.bj-price-updated { font-size:12px; color:var(--bj-grey-mid); text-align:right; }
.bj-quick-answer { background:linear-gradient(135deg,#E8F4FD,#D4E6F1); border:1px solid var(--bj-primary); border-radius:10px; padding:20px 24px; margin:0 0 32px; }
.bj-quick-answer-label { font-family:'Nunito',sans-serif; font-weight:800; font-size:12px; color:var(--bj-primary); text-transform:uppercase; letter-spacing:1px; margin-bottom:8px; }
.bj-expert-pick { display:inline-block; background:var(--bj-gold); color:var(--bj-dark); font-family:'Nunito',sans-serif; font-weight:800; font-size:13px; padding:6px 14px; border-radius:20px; margin:0 0 16px; }
.bj-seasonal-alert { background:#FEF9E7; border:1px solid #F39C12; border-radius:8px; padding:12px 16px; font-size:14px; font-weight:600; color:#7D6608; margin:0 0 24px; }
.bj-disclosure { background:var(--bj-grey-light); border-radius:8px; padding:16px; font-size:13px; color:var(--bj-grey-mid); margin:40px 0 24px; line-height:1.6; }
.bj-faq-item { border-bottom:1px solid #DCE8F0; padding:16px 0; }
.bj-faq-question { font-family:'Nunito',sans-serif; font-weight:700; font-size:16px; margin-bottom:8px; }
.bj-faq-answer { font-size:15px; line-height:1.7; }
@media(max-width:768px){.bj-pros-cons{grid-template-columns:1fr}.bj-product-card{padding:16px}.bj-current-price{font-size:20px}h1{font-size:24px}h2{font-size:20px}}
@media(max-width:480px){.bj-product-image img{max-width:140px}.bj-buy-button{font-size:14px;padding:12px 16px}}
```

## IMAGES (Python/Pillow)
| Type | Size |
|---|---|
| Featured | 1200×628 |
| Banner | 1200×400 |
| Pinterest | 1000×1500 |
| Instagram | 1080×1080 |
| Mini | 400×300 |

## HEADER
- Logo: Baby Junctions logo image
- Tagline: "India ke parents ka trusted guide"
- Sticky: Yes | Mobile: Hamburger
- Background: #FFFFFF | Border-bottom: 2px solid #E8F4FD

## FOOTER
4 columns: Logo+about · Quick links · Categories · Newsletter
Background: #1A2A3A | Text: #FFFFFF

## NEVER DO
- Edit parent theme files
- Install Elementor alongside Kadence
- Save Amazon images to server
- Use inline styles in article HTML
- Add CSS anywhere except child theme style.css
- Use Google Fonts @import in HTML

*v1.0 | April 2026*
