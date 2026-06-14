---
name: voc-scanner
description: "Mines the web for Voice of the Customer language — the exact words and framing practitioners at target institutions use when describing their problems. Captures ICP language so your team can market in their voice. Uses your defined ICP personas to classify who is speaking and surface vivid, reusable phrasing. Writes to voc_signals in your database."
---

# Voice of the Customer Scanner

The goal of this skill is to capture how your ICP actually talks about their problems — the exact words, phrases, metaphors, and framing that practitioners use when they describe their pain. This is language mining for marketing.

The scanner searches across your defined ICP personas and all the pain domains they face, as configured in `references/icp-pain-framework.md`.

The primary output is the language itself — the `quote_snippet` field should always capture the most vivid, reusable phrasing. The secondary output is the classification (who said it, what persona, what pain domain) so your team can target the right language to the right audience.

This is a fully standalone market research tool. It does not reference, route to, or trigger any other skill.

---

## Before you start

You must configure two reference files before this skill will work. These are driven by your answers in QUESTIONS.md.

### 1. ICP Pain Framework (`references/icp-pain-framework.md`)

Answer these questions to populate it:

- **Who are your ICP personas?** Define 3–10 persona types with their titles and pain points.
- **What problems does each persona face?** List strategic/budget pain and operational/day-to-day pain separately.
- **What industry context shapes each persona?** Different sectors may have different dynamics.

### 2. Search Framework (`references/voc-search-framework.md`)

Answer these questions to populate it:

- **What pain domains does your ICP face?** (e.g. vendor evaluation, compliance, speed-to-value, integration friction, cost pressure, talent shortage)
- **What language do practitioners use when describing these pains?** List the exact phrases, complaints, and frustration language for each domain.
- **What institutions should you listen to?** Group your target companies/institutions by type.
- **Where do your ICP personas speak publicly?** (conferences, webinars, podcasts, forums, industry publications, LinkedIn, regulatory bodies)
- **What sources should be excluded?** (your own content, vendor marketing, generic hype)

---

## Database connection

- **Project:** [YOUR DATABASE PROJECT]
- **Project ID:** [YOUR PROJECT ID]
- **Table:** voc_signals

Use `INSERT ... ON CONFLICT (source_url) DO NOTHING` to handle duplicates.

---

## Execution sequence

### Step 1 — Run institution-led searches

Search for **specific institutions** (from your search framework) + **practitioner voice language** across all pain domains. Use the search queries defined in your search framework.

The key is to find real people at real institutions describing their actual frustrations — not vendor marketing, not analyst reports, not generic thought leadership.

### Step 2 — Run context-specific searches

Conferences, webinars, podcasts, and panels are where practitioners speak candidly. Search for events and forums defined in your search framework.

### Step 3 — For each result, extract the voice signal

For each search result that contains a practitioner or executive voice signal:

1. Read the source and confirm there's a real person speaking (not marketing copy or vendor content)
2. Extract:
   - **Who** — person name (if identifiable), their title, their institution
   - **Which persona** — map them to one of your defined ICP personas using titles and context from the ICP pain framework
   - **What they said** — the pain point, frustration, wish, or criteria described. Pay particular attention to exact phrasing — vivid language, specific numbers, metaphors, and emotional framing are gold for marketing copy
   - **Which pain domain** — map to your defined pain domains
   - **How specific** — vague ("this is hard") vs detailed ("our process takes 9 months and involves 14 internal stakeholders")
   - **What context** — conference, interview, blog, forum, podcast, regulatory publication
   - **Sentiment** — positive (what works), negative (what's broken), neutral (describing process), mixed
   - **When** — publication date

Skip results that are:
- Vendor marketing disguised as customer voice
- Your own content or content from your customers promoting you
- Generic hype articles without a named practitioner perspective
- **Published more than 24 months before today's date** — hard exclusion

**Date verification is mandatory before filing any signal.** If the date cannot be confirmed, skip the result. Never estimate or infer a publication date.

---

### Quality guardrails — apply before filing any signal

#### 1. Quote integrity — verbatim only

The `quote_snippet` field must contain the exact words as they appear in the source — nothing constructed, synthesised, or paraphrased.

#### 2. Source-blocked fallback protocol

When a source is blocked, you may still file a signal — but only if:
- **Date is explicit** in the search snippet
- **Quote is verbatim** text from the snippet, not constructed
- **Author/institution is identifiable** from the snippet itself

Add `"quote_source: search_snippet — domain blocked"` to `notes` for any record filed under this fallback.

#### 3. Attribution accuracy — who is actually speaking

`source_name`, `person_name`, `person_title`, and `institution` must reflect who is speaking — not who the article is about.

### Step 4 — Classify

**Persona type** — map to your defined personas from the ICP pain framework, or `unknown` if ambiguous.

**Pain domain** — map to your defined pain domains from the search framework.

**Sentiment**: `positive`, `negative`, `neutral`, `mixed`

**Voice context**: `conference_talk`, `interview`, `blog_post`, `forum`, `social_media`, `webinar`, `podcast`, `analyst_report`, `press_coverage`, `earnings_call`, `panel_discussion`, `case_study`, `regulatory_publication`, `other`

**Person level** (if identifiable): `ceo`, `cfo`, `cto`, `cio`, `cdo`, `ciso`, `other_csuite`, `head_of`, `vp`, `director`, `practitioner`, `unknown`

**Relevance levels**:
- `direct` — mentions your company by name, or describes exactly the problem you solve
- `high` — named institution + specific pain in a relevant domain + attributable person
- `medium` — general industry discussion about relevant challenges with useful detail
- `low` — tangential mention, generic frustration without institutional context

### Step 5 — Score

Three dimensions:
- **Recency** (weight 0.25): How recent?
- **Specificity** (weight 0.45): How detailed, attributable, and specific?
- **Relevance** (weight 0.30): How relevant to your company?

### Step 6 — Write to database

```sql
INSERT INTO voc_signals (
  person_name, person_title, person_level, persona_type,
  institution, institution_type,
  signal_type, pain_domain, summary, quote_snippet, sentiment, voice_context,
  source_url, source_name, source_authority, published_date,
  relevance, categories,
  score_recency, score_specificity, score_relevance,
  institutions_mentioned, competitors_mentioned, technologies_mentioned,
  geography, routed_to, routed_at, notes
) VALUES (...)
ON CONFLICT (source_url) DO NOTHING;
```

### Step 7 — Produce the scan summary

**Best language captured** (top 5-10 quote snippets ranked by vividness and reusability):
- The exact quote, who said it, their persona type, institution, and pain domain

**Language patterns** (recurring phrases across multiple signals):
- What words keep appearing? Group by pain domain.
- Any language that would work as ad copy, landing page headlines, or social hooks?

**By persona** — how does each persona talk about their problems differently?

**By pain domain** — which domains are loudest today?

**Positive signals** (sentiment = positive) — what's working? Useful for proof points.

**Emerging themes** — cross-persona patterns, clustering, convergence

**Scan stats** — total found, new entries written, breakdowns by persona/domain/sentiment

---

## Error handling

- If web search fails for a specific query, log it and continue.
- If database write fails, report the error but continue processing.
- If the reference files cannot be read, stop.

---

## Important constraints

- Writes ONLY to `voc_signals`. Never to any other table.
- This skill is fully standalone. It never references, routes to, or triggers any other skill.
- Only write results with actual practitioner or executive voice. No vendor marketing.
- `quote_snippet` is the most important field. 10-25 words. Vivid, specific, emotional, or contains numbers.
- `summary` column: 2-3 sentences max.
- Honest scoring. A vague complaint = specificity 2. A named person with specific numbers = specificity 10.
- Deduplicate aggressively. Same talk covered by 3 outlets = keep the best source.
