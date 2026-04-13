# CONTENT.md — Baby Junctions
## Brand Voice, Blog Types, JSON Schema & Content Standard

> This file defines what every article sounds like, looks like, and is structured like.
> Claude API reads this before writing any article. No exceptions.
> The standard is BabyGearLab depth — not thin, not generic, not robotic.

---

## 1. THE CONTENT STANDARD

### The BabyGearLab Benchmark
Every article must match this quality bar:
- Real product data — actual prices, real star ratings, actual review counts
- Honest assessments — if a product has a flaw, say it clearly
- Depth over length — 1,500 words of useful content beats 3,000 words of filler
- India-specific — prices in INR, Indian brands mentioned, Indian seasons referenced
- Parent-to-parent tone — like advice from a trusted didi or bhaiya, not a corporate site

### What We Are NOT
- Not a product catalogue dumped from Amazon
- Not a generic listicle that could be about any country
- Not a site that only says positive things about every product
- Not a site that uses vague language like "great quality" or "highly recommended"

---

## 2. BRAND VOICE — DETAILED RULES

### The Voice Character
Imagine the writer is:
- An Indian parent, 30-35 years old
- Lives in a tier-1 city (Mumbai, Delhi, Bengaluru, Ahmedabad)
- Has done real research before buying their own baby products
- Is sharing that research honestly with other parents
- Speaks English naturally but occasionally drops a Hindi/Hinglish phrase

### Tone Examples

**WRONG (generic, robotic):**
"In this comprehensive article, we will discuss the top 10 baby monitors available in India. These products have been carefully selected based on various factors."

**RIGHT (warm, Indian, specific):**
"Baby monitor khareedna hai? We've tested and compared 10 options under ₹5,000 — and honestly, most aren't worth it. Here's what we actually recommend."

---

**WRONG (fake enthusiasm):**
"This AMAZING stroller is absolutely PERFECT for every Indian family and is the BEST choice available!"

**RIGHT (honest, specific):**
"The Chicco Miinimo 3 is our top pick for Indian roads — it folds in one hand, handles Mumbai potholes better than most, and at ₹8,500 it's genuinely good value. The downside? The canopy is a bit small."

---

**WRONG (non-Indian):**
"Prices start at $29.99. Available on Amazon."

**RIGHT (Indian):**
"Prices start at ₹2,499 on Amazon.in. As of today, the Pigeon option is slightly cheaper on Amazon compared to last week."

---

### Language Rules
- Always use ₹ — never $, never USD
- Refer to Amazon.in not Amazon.com
- Use Indian brand names: Pigeon, Mee Mee, LuvLap, Himalaya Baby, Mamaearth, Johnson's India
- Reference Indian cities naturally: "great for Mumbai's humidity", "perfect for Delhi winters"
- Reference Indian seasons: "during monsoon", "in summer", "before Diwali"
- Occasional Hinglish is fine: "Thoda costly hai, but worth it", "Bilkul safe hai", "Sach mein kaam aata hai"
- Never overdo Hinglish — max 1-2 phrases per article, only where natural

---

## 3. ARTICLE STRUCTURE — EVERY ARTICLE HAS THESE SECTIONS

```
1. Quick Answer Box         ← 2-3 sentences answering the main question directly
2. Seasonal Alert Banner    ← Only if seasonally relevant (monsoon/summer/festive)
3. Expert Pick Badge        ← For Top 10 / Best Of articles
4. Introduction             ← 150-200 words, conversational, keyword in first 100 words
5. [Blog Type Content]      ← Main section — varies by blog type (see Section 4)
6. Buying Guide             ← 5-7 factors to consider (for transactional articles)
7. Science / Expert Section ← 150-200 words of expert-backed context
8. FAQ Section              ← 5 questions with answers (schema markup)
9. Affiliate Disclosure     ← Required by Amazon ToS
```

---

## 4. ALL 21 BLOG TYPES — COMPLETE DEFINITIONS

---

### TYPE 1: Top 10 List
**Intent:** Transactional — buyer is ready to purchase
**Keyword signals:** best, top, recommended, highest rated
**Structure:**
- Quick Answer: "Our top pick is [Product] at ₹X"
- 7-10 product cards (ranked #1 to #10)
- Each card: image, title, stars, review count, price, why we picked it, pros/cons, buy button
- Comparison table at end (all products, key specs)
- Buying guide section
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 2: Comparison (A vs B)
**Intent:** Transactional — buyer has narrowed to 2 options
**Keyword signals:** vs, versus, compare, difference between, or
**Structure:**
- Quick Answer: "For [use case], [Product A] wins. For [budget], [Product B] is better."
- Side-by-side comparison table (15 factors)
- Deep dive on Product A (card format)
- Deep dive on Product B (card format)
- Clear winner verdict with reasoning
- Who should buy each
- FAQ (5 questions)
**Word count:** 1,800-2,200

---

### TYPE 3: Buying Guide
**Intent:** Informational-to-transactional — buyer is learning before deciding
**Keyword signals:** how to choose, what to look for, guide to buying, best way to buy
**Structure:**
- Quick Answer: "The 3 most important things to look for are X, Y, Z"
- 8 factors to consider (with explanations)
- 5 mistakes to avoid
- Price range breakdown (budget / mid-range / premium)
- Top 3 recommended products
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 4: What to Avoid
**Intent:** Informational — parent wants to know what NOT to buy
**Keyword signals:** avoid, don't buy, mistakes, wrong, bad products
**Structure:**
- Quick Answer: "Avoid [specific type] — here's why"
- List of 5 products/types to avoid (with specific reasons)
- What makes a product dangerous or poor quality
- Safe alternatives for each
- Safety checklist
- FAQ (5 questions)
**Word count:** 1,500-1,800

---

### TYPE 5: Why Not to Buy
**Intent:** Informational — parent is second-guessing a popular product
**Keyword signals:** why not, should I, worth it, is it good, is it safe
**Structure:**
- Quick Answer: Direct yes/no + one-line reason
- Common reasons people buy this product
- Why those reasons may be wrong
- What to buy instead (3 alternatives with cards)
- When it IS worth buying (balanced view)
- FAQ (5 questions)
**Word count:** 1,200-1,500

---

### TYPE 6: Budget Guide
**Intent:** Transactional — price-sensitive Indian buyer
**Keyword signals:** under ₹, budget, cheap, affordable, best value
**Structure:**
- Quick Answer: "Best under ₹X is [Product] — here's why"
- Budget breakdown table (under ₹500 / ₹500-₹1500 / ₹1500-₹3000)
- Top pick per price bracket (product cards)
- What you sacrifice at each price point (honest)
- What NOT to compromise on regardless of budget
- FAQ (5 questions)
**Word count:** 1,500-1,800

---

### TYPE 7: Age-Based Guide
**Intent:** Informational — parent buying for specific age/stage
**Keyword signals:** for newborn, 0-6 months, 6 month old, 1 year, toddler
**Structure:**
- Quick Answer: "At [age], babies need X. Here's what actually works."
- Developmental stage context (brief)
- Top 5-8 recommended products for this age
- Products to avoid at this age
- When to transition to next stage
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 8: Safety Alert
**Intent:** Informational — urgent safety concern
**Keyword signals:** recall, banned, unsafe, dangerous, BIS certified, toxic
**Structure:**
- Quick Answer: "This product [is/is not] safe. Here's what you need to know."
- What the safety concern is
- Which specific products are affected
- How to check if your product is affected
- Safer alternatives (product cards)
- Official guidelines reference
- FAQ (5 questions)
**Word count:** 1,000-1,500

---

### TYPE 9: Organic vs Regular
**Intent:** Informational — health-conscious parent
**Keyword signals:** organic, natural, chemical free, non-toxic, safe ingredients
**Structure:**
- Quick Answer: "For [product type], [organic/regular] is better because..."
- What "organic" actually means for this product category in India
- Ingredient comparison (what to avoid, what's safe)
- Price difference analysis
- Top organic pick + top regular pick (product cards)
- Verdict: when it's worth paying more
- FAQ (5 questions)
**Word count:** 1,800-2,000

---

### TYPE 10: Brand Deep Dive
**Intent:** Informational — parent wants to understand a brand
**Keyword signals:** brand name (Pigeon, Chicco, Mee Mee, LuvLap, etc.)
**Structure:**
- Quick Answer: "[Brand] is [verdict] — here's what you need to know"
- Brand background (India operations, parent company)
- Full product range overview
- Quality assessment (materials, safety standards)
- Price vs value analysis
- Best products from this brand (top 3 cards)
- When to choose this brand vs alternatives
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 11: Myth Buster
**Intent:** Informational — parent has heard conflicting advice
**Keyword signals:** myth, truth, fact, is it true, really works, do I need
**Structure:**
- Quick Answer: "The truth about [topic] is..."
- 5 myths (each: the myth → the truth → the science behind it)
- Expert consensus summary
- Practical takeaway for Indian parents
- FAQ (5 questions)
**Word count:** 1,500-1,800

---

### TYPE 12: Problem-Solution
**Intent:** Transactional — parent has a specific problem right now
**Keyword signals:** not sleeping, colic, rash, teething, refuses, leaking, crying
**Structure:**
- Quick Answer: "For [problem], try [solution] first — here's why"
- Why this problem happens (brief, non-medical)
- 5 product solutions (cards with specific use case)
- 3 non-product tips (parents appreciate the honesty)
- When to see a doctor (E-E-A-T + trust signal)
- FAQ (5 questions)
**Word count:** 1,500-2,000

---

### TYPE 13: Gift Guide
**Intent:** Transactional — buyer is shopping for someone else
**Keyword signals:** gift, baby shower, present, for new parents, birthday
**Structure:**
- Quick Answer: "Best gift for [occasion] under ₹X is [Product]"
- Budget breakdown: under ₹500 / ₹500-₹1500 / ₹1500-₹3000 / ₹3000+
- 10 product recommendations with occasion context
- "What new parents actually need" honest section
- What NOT to gift (saves buyer embarrassment)
- FAQ (5 questions)
**Word count:** 1,800-2,200

---

### TYPE 14: Seasonal Guide
**Intent:** Informational — season/festival specific buying
**Keyword signals:** monsoon, summer, winter, Diwali, festive, seasonal
**Structure:**
- Quick Answer: "For [season], the most important baby product is..."
- Why [season] matters for babies specifically in India
- 8 recommended products with seasonal relevance (cards)
- Products to avoid in this season
- Season-specific care tips (non-product)
- FAQ (5 questions)
**Word count:** 1,500-1,800

---

### TYPE 15: First-Time Parent Guide
**Intent:** Informational — overwhelmed new parent
**Keyword signals:** first time, new parent, what do I need, complete list, just had baby
**Structure:**
- Quick Answer: "You actually need [X things]. Most of the rest can wait."
- What you ACTUALLY need (essentials — product cards)
- What you DON'T need (saves money — honest)
- Month-by-month buying timeline
- Budget breakdown for first 6 months
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 16: Interactive Tool
**Intent:** Informational — parent wants a calculator/tracker
**Keyword signals:** calculator, tracker, planner, how much, how many
**Structure:**
- Quick Answer: The tool result summary
- The tool itself (embedded interactive element or step-by-step guide)
- Explanation of how to use results
- Related product recommendations based on results
- FAQ (5 questions)
**Word count:** 800-1,200

---

### TYPE 17: Checklist
**Intent:** Informational — parent wants a complete list
**Keyword signals:** checklist, list of, everything needed, complete list, don't forget
**Structure:**
- Quick Answer: "You need [X] items. Here's the complete list."
- Checklist by category (20-30 items)
- For each item: why it's needed + product recommendation
- Priority level per item (must-have / nice-to-have / skip)
- Printable format note
- FAQ (5 questions)
**Word count:** 1,200-1,500

---

### TYPE 18: Scientific / Research
**Intent:** Informational — parent wants evidence-based advice
**Keyword signals:** study, research, science, what experts say, pediatrician, evidence
**Structure:**
- Quick Answer: "Research shows that..."
- What studies/experts say (plain English — no jargon)
- What this means practically for Indian parents
- Common misconceptions vs evidence
- Product recommendations based on findings
- FAQ (5 questions)
**Word count:** 2,000-2,500

---

### TYPE 19: Subscription Box Review
**Intent:** Transactional — parent considering a recurring purchase
**Keyword signals:** subscription, monthly box, curated, surprise box
**Structure:**
- Quick Answer: "[Box name] is worth it if you [condition]. Here's our honest review."
- What's inside (typical month)
- Price per item analysis (is it actually cheaper?)
- Who it's best for / who should skip it
- Alternatives comparison
- FAQ (5 questions)
**Word count:** 1,800-2,200

---

### TYPE 20: Hospital Bag Guide
**Intent:** Transactional — pregnant parent preparing for delivery
**Keyword signals:** hospital bag, delivery bag, what to pack, birth kit, labour bag
**Structure:**
- Quick Answer: "Pack [X] days before due date. Here are the 20 essentials."
- For Mom section (items + products)
- For Baby section (items + products)
- For Partner/Support section
- What NOT to pack (saves space)
- India-specific tips (what hospitals usually provide)
- FAQ (5 questions)
**Word count:** 1,500-2,000

---

### TYPE 21: Platform Comparison (Amazon vs Flipkart context)
**Intent:** Transactional — buyer wants best price
**Keyword signals:** where to buy, amazon or flipkart, best deal, cheapest
**Structure:**
- Quick Answer: "For [product], [platform] is cheaper today by ₹X"
- Price comparison table (last 30 days)
- Delivery time comparison
- Return policy comparison
- Which platform wins for this category overall
- Current best deal (product card)
- FAQ (5 questions)
**Word count:** 1,200-1,500

---

## 5. JSON SCHEMA — MASTER FORMAT

Claude API outputs ONLY this JSON structure. No free text. No markdown. Pure JSON.

```json
{
  "meta": {
    "topic_id": "BJ-20260413-01",
    "blog_type": "top_10_list",
    "subcategory": "baby_monitors",
    "keyword": "best baby monitor india under 5000",
    "target_audience": "first-time parents, India, budget-conscious",
    "seasonal_angle": "Parents staying home in summer want audio monitoring",
    "generated_date": "2026-04-13"
  },

  "seo": {
    "h1_title": "10 Best Baby Monitors in India Under ₹5,000 (2026) — Tested",
    "meta_title": "Best Baby Monitors India Under ₹5,000 | 2026 Reviews",
    "meta_description": "We tested 10 baby monitors under ₹5,000. Our top pick for Indian homes in 2026 — with real prices, honest pros and cons.",
    "focus_keyword": "best baby monitor india under 5000",
    "lsi_keywords": ["baby monitor price india", "wifi baby monitor india", "baby monitor under 5000"],
    "slug": "best-baby-monitors-india-under-5000"
  },

  "quick_answer": "The best baby monitor in India under ₹5,000 is the Motorola VM34 — reliable range, clear audio, and it works well in Indian apartments. If you want video too, the Pigeon Video Monitor is worth the extra ₹500.",

  "seasonal_banner": {
    "show": true,
    "text": "🌡️ Summer Update: Baby monitors with temperature alerts are especially useful this season — we've flagged which ones have this feature below."
  },

  "intro": "300-word intro text here — conversational, slightly Hinglish, keyword in first 100 words...",

  "products": [
    {
      "rank": 1,
      "asin": "B09XYZ1234",
      "title": "Motorola VM34 Baby Monitor",
      "affiliate_link": "https://www.amazon.in/dp/B09XYZ1234?tag=babyjunctions-21",
      "image_url": "https://m.media-amazon.com/images/I/...",
      "price_inr": 3499,
      "star_rating": 4.3,
      "review_count": 1247,
      "availability": "IN_STOCK",
      "why_picked": "Best audio range in its price class — works across 2 floors in a typical Indian apartment without signal drops.",
      "pros": ["Excellent 300m range", "Clear audio quality", "Temperature display", "Long battery — 6 hours"],
      "cons": ["No video", "App can be buggy", "Charger feels cheap"],
      "best_for": "Parents who want reliable audio without overspending",
      "verdict": "Top pick under ₹3,500"
    }
  ],

  "comparison_table": {
    "headers": ["Product", "Price", "Rating", "Range", "Video?", "Battery"],
    "rows": []
  },

  "buying_guide": {
    "heading": "What to Look for When Buying a Baby Monitor in India",
    "factors": [
      {
        "factor": "Range",
        "explanation": "For Indian apartments (2-3 BHK), 150m range is enough. For larger homes or independent houses, go for 300m+.",
        "what_to_look_for": "Minimum 150m for apartments, 300m for independent houses"
      }
    ]
  },

  "science_section": {
    "heading": "What Pediatric Experts Say About Baby Monitors",
    "body": "150-200 word evidence-based section...",
    "key_facts": [
      "No baby monitor is a substitute for supervision",
      "Keep monitor at least 1 metre from the cot",
      "Audio-only monitors are sufficient for most families"
    ]
  },

  "faq": [
    {
      "question": "Which baby monitor is best for Indian apartments?",
      "answer": "For a 2-3 BHK apartment, any monitor with 150m+ range works well. The Motorola VM34 is our top pick — it maintains signal through 2-3 walls without issues."
    }
  ],

  "internal_links": [
    {
      "anchor_text": "best baby safety gates in India",
      "target_slug": "best-baby-safety-gates-india",
      "context": "If you're also babyproofing, check our guide to..."
    }
  ],

  "schema_markup": {
    "article_type": "Article",
    "product_schema": true,
    "faq_schema": true
  }
}
```

---

## 6. SCIENCE SECTION PROMPT

Claude generates this for every article. It adds E-E-A-T signals.

```
Write a 150-200 word section titled "What [Experts/Pediatricians/Research] Say About [topic]".

Rules:
- Write at Class 8 reading level — simple words, short sentences
- Include 1 safety fact, 1 material/chemical note if relevant, 1 age-appropriateness guideline
- Do NOT cite specific studies by name — write as general expert consensus
- End with exactly 3 bullet "key facts" that a parent can remember instantly
- Tone: reassuring, not scary
- Reference Indian context where possible
```

---

## 7. CONTENT QUALITY CHECKLIST

Before publishing, every article must pass this check:

```
□ Keyword appears in first 100 words of intro
□ H1 title is the SEO title from JSON
□ H2 headings are question-formatted ("Which baby monitor is best under ₹5,000?")
□ Every product card has: image, stars, review count, price, buy button
□ Prices shown in ₹ only
□ Quick Answer box is at the very top
□ Affiliate disclosure is at the bottom
□ At least 3 internal links to other articles
□ FAQ section has exactly 5 questions
□ Science section is 150-200 words
□ No generic phrases ("In this article we will discuss")
□ No fake superlatives ("ABSOLUTELY THE BEST")
□ India-specific context exists somewhere in article
□ Word count is within target range for blog type
```

---

## 8. INTERNAL LINKING RULES

- Every new article must link to 3-5 existing articles
- Every existing article gets a retroactive link to new article (Stage 3B)
- Anchor text must be descriptive: "best baby monitors under ₹5,000" not "click here"
- Links go inside natural sentences — never in a list of links
- Never link to a competitor site in the body text

---

*Last updated: April 2026 | Version 1.0 | Baby Junctions*
