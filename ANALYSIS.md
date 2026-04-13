# ANALYSIS.md — Baby Junctions
## Performance Tracking, SM Analysis & Feedback Loop

> This file defines HOW we measure performance, WHAT we look for,
> and HOW findings feed back into Stage 1 to make tomorrow's topics smarter.
> This is the intelligence layer that separates Baby Junctions from every other affiliate site.

---

## 1. THE CORE PRINCIPLE

Most affiliate sites publish and forget.
Baby Junctions publishes, measures, learns, and improves — automatically.

Every month, the system asks:
- What worked? → Do more of it
- What didn't work? → Do less of it or fix it
- What surprised us? → Understand why and replicate it
- What's coming? → Seasonal/festival opportunities ahead

The answers go into Google Sheets → Analytics Feed tab.
Stage 1 reads this tab every morning before generating topics.

---

## 2. DATA SOURCES WE TRACK

| Source | What It Tells Us | How We Get It | Frequency |
|---|---|---|---|
| Google Search Console | Which keywords rank, which pages get clicks, avg position | GSC API | Monthly (Stage 7) |
| Google Analytics 4 | Sessions, bounce rate, time on page, traffic source | GA4 API | Monthly |
| Instagram Insights API | Impressions, reach, link clicks, saves per post | Instagram Graph API | Weekly |
| Pinterest Analytics API | Pin impressions, saves, outbound clicks per pin | Pinterest API | Weekly |
| WhatsApp Channel | Views per broadcast, link clicks (UTM tracked) | GA4 UTM data | Weekly |
| Amazon PA API | Affiliate click-through (estimated via link clicks) | GA4 + PA API | Monthly |
| WordPress | Internal — which related posts get clicked | GA4 Events | Monthly |

---

## 3. WHAT WE MEASURE — THE 6 LENSES

### LENS 1: Keyword Performance
What to look for:
- Keywords ranking positions 1-10 → content is working, create more on same topic
- Keywords ranking positions 11-20 → near miss, needs improvement (Stage 7 acts on these)
- Keywords ranking positions 21+ → either too competitive or content too thin
- New keywords appearing in GSC that we haven't targeted → opportunity list

Questions to answer:
- Which subcategory has the most ranking keywords?
- Which blog types rank fastest (within 60 days)?
- Which keywords brought the most clicks last month?

Output to Analytics Feed tab:
```
"Baby monitor keywords rank faster than diaper keywords — avg 45 days vs 90 days"
"Buying Guide blog types reach top 20 in 30 days on average"
"Keywords with ₹ price in title get 2x more clicks"
```

---

### LENS 2: Blog Type Performance
What to look for:
- Which of the 21 blog types generates most clicks?
- Which blog types convert to affiliate clicks most?
- Which blog types rank fastest in Google?
- Which blog types get most social shares?

Questions to answer:
- Are Top 10 Lists outperforming Buying Guides?
- Are Myth Busters getting more shares but fewer affiliate clicks?
- Are Gift Guides spiking in October-December only?

Output to Analytics Feed tab:
```
"Top 10 Lists get 3x more Pinterest saves than Buying Guides"
"Myth Busters rank in 30 days but convert at 0.5x rate — use for SEO not revenue"
"Gift Guides spike 500% in Oct-Dec — start publishing in September"
```

---

### LENS 3: Seasonal & Festival Performance
What to look for:
- Which subcategories spike in which months?
- Which festivals drive the biggest traffic surges?
- How far in advance should we start publishing seasonal content?

India's seasonal calendar to track:
```
January:    Republic Day sales, winter baby care
February:   Valentine's gifting, spring gear
March:      Holi, FY end sales
April:      Summer begins, heat rash products spike
May:        Mother's Day, peak summer products
June:       Monsoon begins, waterproof + fungal care
July:       Monsoon deep, indoor toys spike
August:     Independence Day sales, Raksha Bandhan
September:  Navratri prep, Durga Puja gifting
October:    Diwali, Big Billion Days — BIGGEST MONTH
November:   Post-Diwali, Children's Day (Nov 14)
December:   Christmas, winter baby care, year-end lists
```

Output to Analytics Feed tab:
```
"Diaper rash content spikes 400% in June-August — start publishing in May"
"October affiliate revenue is 3x average — focus high-ticket items in Sept-Oct"
"'Best of 2026' year-end lists should be published in November"
```

---

### LENS 4: Social Media Performance
What to look for across Instagram, Pinterest, WhatsApp:

**Instagram:**
- Which caption formats get most link clicks? (price-first vs question-first vs tip-first)
- Which image types perform best? (product-only vs lifestyle vs comparison)
- Which hashtag combinations drive most reach?
- What posting time gets most engagement? (7-9 AM vs 7-9 PM IST)
- Which blog types make best Instagram content?

**Pinterest:**
- Which pin formats get most saves? (price in title vs benefit in title)
- Which subcategories get most saves on Pinterest?
- Which image orientation performs better (1000x1500 vertical)?
- Which boards are growing fastest?

**WhatsApp Channel:**
- Which broadcast formats get most views? (deal alert vs tip vs new article)
- Which time of day gets most opens?
- Which topics generate most link clicks from WhatsApp?

Output to Analytics Feed tab:
```
"Instagram posts with '₹X' price in caption get 40% more link clicks"
"Pinterest pins in Baby Safety subcategory get 2x more saves than Baby Monitors"
"WhatsApp broadcasts sent at 8 PM get 3x more clicks than 10 AM"
"Question-format Instagram captions get 2x more comments → more reach"
```

---

### LENS 5: Product Performance
What to look for:
- Which products get most affiliate link clicks?
- Which price ranges convert best? (under ₹500 / ₹500-₹2000 / ₹2000-₹5000 / ₹5000+)
- Which brands get most clicks? (Pigeon vs Mee Mee vs Chicco vs international brands)
- Which products go OOS most frequently? (avoid featuring unreliable inventory)
- Which products have best click-to-purchase ratio?

Output to Analytics Feed tab:
```
"Products ₹800-₹2500 range get 2x more clicks than products above ₹5000"
"Pigeon brand products get highest CTR — Indian parents trust this brand"
"Baby monitors go OOS frequently in October — find reliable ASINs in September"
"Products with 200+ reviews convert at 2x rate vs products with 100-200 reviews"
```

---

### LENS 6: Content Quality Performance
What to look for:
- Which articles have best time-on-page? (indicates quality)
- Which articles have highest bounce rate? (indicates mismatch)
- Which articles get most internal link clicks? (indicates interest in topic)
- Which articles get most return visitors?

Output to Analytics Feed tab:
```
"Articles above 2500 words have 40% better time-on-page"
"Articles with comparison tables have 25% lower bounce rate"
"Articles with Quick Answer box at top have 15% lower bounce rate"
```

---

## 4. THE FEEDBACK LOOP — HOW IT WORKS

```
STEP 1: DATA COLLECTION (Weekly + Monthly)
Weekly:  Instagram API + Pinterest API + WhatsApp UTM → SM Performance tab
Monthly: GSC API + GA4 → GSC Performance + Product Performance tabs

STEP 2: ANALYSIS (Monthly — 1st of month, Stage 7)
Claude reads all Performance tabs
Identifies patterns across all 6 lenses
Generates plain-English insights

STEP 3: WRITE TO ANALYTICS FEED TAB
Each insight written as a row:
- What the insight is
- Which data source proved it
- What Stage 1 should do with it
- How long it's valid

STEP 4: STAGE 1 READS ANALYTICS FEED (Every morning at 6 AM)
Before generating 15 topics, Stage 1:
1. Reads all HIGH priority insights from Analytics Feed
2. Reads all insights valid for current month
3. Adjusts topic selection accordingly

STEP 5: SMARTER TOPICS
Tomorrow's 15 topics are informed by:
- What ranked well last month
- What sold well last month
- What got shared on social last month
- What season/festival is coming
- What blog types are underperforming (need fewer)
- What subcategories haven't been covered recently
```

---

## 5. CLAUDE'S ANALYSIS PROMPT (Used in Stage 7)

When Stage 7 runs on the 1st of every month, Claude receives this prompt:

```
You are the performance analyst for Baby Junctions (babyjunctions.in),
an Indian baby products affiliate site.

Analyse the following data from last month:

GSC DATA: {gsc_data}
INSTAGRAM DATA: {instagram_data}
PINTEREST DATA: {pinterest_data}
WHATSAPP UTM DATA: {whatsapp_data}
PRODUCT CLICK DATA: {product_data}
CURRENT MONTH: {current_month}
UPCOMING FESTIVALS: {festivals}
UPCOMING SEASON: {season}

Generate insights across these 6 categories:
1. Keyword Performance — what ranked, what didn't, what's near-miss
2. Blog Type Performance — which types worked best
3. Seasonal Patterns — what seasonal content to prioritise next month
4. Social Media — what worked on each platform
5. Product Performance — what products/price ranges converted best
6. Content Quality — what made the best articles better

For each insight, specify:
- The insight in one clear sentence
- The data that proves it
- What Stage 1 should do with this tomorrow
- Priority: HIGH/MEDIUM/LOW
- Valid until: which date this insight is still relevant

Return as JSON array of insight objects.
Format: [{insight, data_source, action_for_stage1, priority, valid_until}]
```

---

## 6. WEEKLY SM REPORT GENERATION

Every Sunday, a lightweight SM report is generated:

```python
# Runs every Sunday at 11 AM IST
# Reads SM Performance tab for last 7 days
# Identifies top 3 posts per platform
# Writes to Weekly SM Report tab
# Saves Word doc to Google Drive → /Social Media Reports/
```

The report answers:
- What was the best Instagram post this week and why?
- What was the best Pinterest pin this week and why?
- What was the best WhatsApp broadcast this week and why?
- What should we replicate next week?

---

## 7. METRICS THRESHOLDS — WHAT COUNTS AS "GOOD"

These thresholds define when to flag something as best_performing = YES in SM Performance tab.

| Platform | Metric | Good Threshold |
|---|---|---|
| Instagram | Link clicks | ≥ 50 clicks per post |
| Instagram | Engagement rate | ≥ 3% |
| Instagram | Reach | ≥ 500 unique accounts |
| Pinterest | Outbound clicks | ≥ 30 clicks per pin |
| Pinterest | Saves | ≥ 20 saves per pin |
| Pinterest | Impressions | ≥ 500 per pin |
| WhatsApp | Link clicks | ≥ 40 clicks per broadcast |
| WhatsApp | View rate | ≥ 30% of subscribers |
| GSC | CTR | ≥ 3% |
| GSC | Position | ≤ 20 (on first 2 pages) |

---

## 8. WHAT ANALYSIS NEVER DOES

- Never deletes published articles based on performance alone
- Never changes a live article's primary keyword
- Never removes a product that is still in stock and passes quality filters
- Never overrides seasonal urgency with past performance data
  (e.g. even if baby monitors underperformed last month, still cover them in October because Big Billion Days)

---

## 9. MONTHLY ANALYSIS TIMELINE

```
1st of month — 6:00 AM:   Stage 7 runs — GSC + GA4 data pulled
1st of month — 7:00 AM:   Claude analyses all data across 6 lenses
1st of month — 8:00 AM:   Insights written to Analytics Feed tab
1st of month — 9:00 AM:   SEO improvements queued for Stage 3 to action
1st of month — 10:00 AM:  SEO Loop Results tab updated
1st of month — 11:00 AM:  Monthly report Word doc saved to Google Drive
2nd of month onwards:     Stage 1 begins reading new Analytics Feed insights
```

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
