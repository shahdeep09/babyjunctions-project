# CONTENT.md — Baby Junctions
> Brand voice, all 21 blog types, JSON schema. Claude reads this before writing any article.

## VOICE RULES
- Writer persona: trusted Indian parent, 30-35, tier-1 city, done real research
- Always use ₹ — never $. Always Amazon.in — never Amazon.com
- Mention Indian brands: Pigeon, Mee Mee, LuvLap, Chicco, Himalaya Baby, Mamaearth
- Reference Indian context: cities, seasons, festivals naturally
- Hinglish: max 1-2 phrases per article, only when natural
- Never: "In this article we will discuss..." · fake superlatives · generic openers
- Honest: mention cons, not just pros

**Wrong:** "This AMAZING stroller is absolutely PERFECT for every family!"
**Right:** "The Chicco Miinimo 3 handles Mumbai potholes well. Downside: canopy is small."

## ARTICLE STRUCTURE (every article)
1. Quick Answer Box
2. Seasonal Banner (if relevant)
3. Expert Pick Badge (Top 10 only)
4. Intro — 200-250 words, keyword in first 100 words
5. Blog Type Content (varies — see below)
6. Buying Guide (transactional articles)
7. Science/Expert Section — 150-200 words
8. FAQ — exactly 5 questions
9. Affiliate Disclosure

## 21 BLOG TYPES

| # | Type | Intent | Keywords | Words | Revenue |
|---|---|---|---|---|---|
| 1 | Top 10 List | Transactional | best, top, recommended | 2000-2500 | ⭐⭐⭐⭐⭐ |
| 2 | Comparison A vs B | Transactional | vs, versus, compare | 1800-2200 | ⭐⭐⭐⭐⭐ |
| 3 | Buying Guide | Info→Trans | how to choose, guide | 2000-2500 | ⭐⭐⭐⭐ |
| 4 | What to Avoid | Informational | avoid, don't buy, mistake | 1500-1800 | ⭐⭐⭐ |
| 5 | Why Not to Buy | Informational | why not, worth it, is it good | 1200-1500 | ⭐⭐⭐ |
| 6 | Budget Guide | Transactional | under ₹, budget, affordable | 1500-1800 | ⭐⭐⭐⭐ |
| 7 | Age-Based Guide | Informational | newborn, 6 month, toddler | 2000-2500 | ⭐⭐⭐⭐ |
| 8 | Safety Alert | Informational | recall, unsafe, dangerous | 1000-1500 | ⭐⭐ |
| 9 | Organic vs Regular | Informational | organic, natural, chemical free | 1800-2000 | ⭐⭐⭐⭐ |
| 10 | Brand Deep Dive | Informational | brand name (Pigeon, Chicco…) | 2000-2500 | ⭐⭐⭐ |
| 11 | Myth Buster | Informational | myth, truth, fact, is it true | 1500-1800 | ⭐⭐ |
| 12 | Problem-Solution | Transactional | colic, rash, not sleeping | 1500-2000 | ⭐⭐⭐⭐ |
| 13 | Gift Guide | Transactional | gift, baby shower, present | 1800-2200 | ⭐⭐⭐⭐⭐ |
| 14 | Seasonal Guide | Informational | monsoon, summer, winter, Diwali | 1500-1800 | ⭐⭐⭐ |
| 15 | First-Time Parent | Informational | first time, new parent, what do I need | 2000-2500 | ⭐⭐⭐⭐ |
| 16 | Interactive Tool | Informational | calculator, tracker, planner | 800-1200 | ⭐⭐ |
| 17 | Checklist | Informational | checklist, everything needed | 1200-1500 | ⭐⭐⭐ |
| 18 | Scientific/Research | Informational | study, research, pediatrician | 2000-2500 | ⭐⭐ |
| 19 | Subscription Box | Transactional | subscription, monthly box | 1800-2200 | ⭐⭐⭐ |
| 20 | Hospital Bag | Transactional | hospital bag, what to pack | 1500-2000 | ⭐⭐⭐⭐ |
| 21 | Platform Comparison | Transactional | where to buy, best deal | 1200-1500 | ⭐⭐⭐⭐⭐ |

## DAILY BLOG TYPE MIX (15 articles)
- 2 × Top 10 List
- 1 × Budget Guide
- 1 × Gift Guide OR Platform Comparison
- 3 × Buying Guide
- 2 × Age-Based OR Seasonal
- 2 × Problem-Solution
- 2 × Myth Buster / Safety / Scientific
- 2 × Flexible (based on Analytics Feed)

## JSON SCHEMA (Claude outputs this — no free text)
```json
{
  "meta": {
    "topic_id": "BJ-YYYYMMDD-01",
    "blog_type": "top_10_list",
    "subcategory": "baby_monitors",
    "keyword": "best baby monitor india under 5000",
    "seasonal_angle": "string",
    "generated_date": "YYYY-MM-DD"
  },
  "seo": {
    "h1_title": "string",
    "meta_title": "string — under 60 chars",
    "meta_description": "string — under 155 chars",
    "focus_keyword": "string",
    "lsi_keywords": ["string"],
    "slug": "lowercase-hyphenated"
  },
  "quick_answer": "2-3 sentence direct India-specific answer",
  "seasonal_banner": { "show": true, "text": "string with emoji" },
  "expert_pick_badge": "Baby Junctions Expert Pick — Month Year",
  "intro": "200-250 words — conversational, keyword in first 100 words",
  "products": [
    {
      "rank": 1,
      "asin": "string",
      "title": "string",
      "affiliate_link": "string",
      "image_url": "string",
      "price_inr": 3499,
      "star_rating": 4.3,
      "review_count": 1247,
      "availability": "IN_STOCK",
      "badge": "Best Overall",
      "why_picked": "2-3 sentences, specific, honest, India-relevant",
      "pros": ["string","string","string"],
      "cons": ["string","string"],
      "best_for": "one sentence",
      "verdict": "10 words max"
    }
  ],
  "comparison_table": {
    "show": true,
    "headers": ["Product","Price","Rating","Reviews","Key Feature","Best For"],
    "rows": [["Name","₹X,XXX","4.3★","1,200","Feature","Use case"]]
  },
  "main_body": "600-800 words with H2 subheadings as HTML tags",
  "buying_guide": {
    "heading": "string",
    "intro": "50 words",
    "factors": [{"factor":"string","explanation":"60-80 words","what_to_look_for":"one line"}]
  },
  "science_section": {
    "heading": "What Experts Say About {topic}",
    "body": "150-200 words, plain English, reassuring",
    "key_facts": ["string","string","string"]
  },
  "faq": [{"question":"string","answer":"50-100 words"}],
  "internal_links": [{"anchor_text":"string","target_slug":"string","context_sentence":"string"}],
  "affiliate_disclosure": "Baby Junctions participates in Amazon Associates. We earn a small commission at no extra cost to you.",
  "schema_markup": {"article_type":"Article","product_schema":true,"faq_schema":true}
}
```

## SCIENCE SECTION PROMPT
```
Write 150-200 words: "What [Experts] Say About {topic}"
- Class 8 reading level
- 1 safety fact, 1 material note, 1 age guideline
- No specific study citations — write as general expert consensus
- End with 3 bullet key_facts
- Reassuring tone, Indian context where possible
```

## QUALITY CHECKLIST (pre-publish)
```
□ Keyword in first 100 words
□ H2s are question-formatted
□ Every product: image + stars + review count + price + buy button
□ Prices in ₹ only — no $ anywhere
□ Quick Answer at top
□ Disclosure at bottom
□ 3+ internal links (valid slugs only)
□ Exactly 5 FAQs
□ Science section 150-200 words
□ No "In this article we will discuss..."
□ No fake superlatives
□ India-specific reference exists
□ Word count within blog type range
```

## INTERNAL LINKING RULES
- Every new article links to 3-5 existing articles
- Anchor text: descriptive ("best baby monitors under ₹5,000" not "click here")
- Links inside natural sentences — never in a list
- Never validate links to slugs not in Published tab

*v1.0 | April 2026*
