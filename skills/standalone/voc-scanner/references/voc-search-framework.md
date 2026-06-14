# Voice of the Customer Search Framework

Reference data for the VOC Scanner. Defines what practitioner voice signals look like, where to find them, and how to score them.

The core thesis: when practitioners and decision-makers at target institutions publicly describe their frustrations — in their own words — that's the most valuable market intelligence. It reveals unmet needs, messaging language, product gaps, and buying triggers that no analyst report can match.

Populate this by answering the questions below.

---

## Section 1 — Signal Language

For each pain domain your ICP faces, define the phrases that indicate a practitioner is expressing an authentic opinion. These become your search queries.

### [YOUR PAIN DOMAIN 1 — e.g. "Vendor evaluation and procurement"]

```
[YOUR SIGNAL PHRASES — e.g.]
"struggle" OR "frustrating" OR "too slow" OR "takes too long"
  OR "months to evaluate" OR "inefficient" OR "cumbersome"
  + "[YOUR DOMAIN KEYWORDS]" OR "[YOUR DOMAIN KEYWORDS]"
  + "[YOUR INDUSTRY]" OR "enterprise"
```

### [YOUR PAIN DOMAIN 2 — e.g. "Speed to value"]

```
[YOUR SIGNAL PHRASES]
  + "[YOUR DOMAIN KEYWORDS]"
  + "[YOUR INDUSTRY]" OR "enterprise"
```

### [YOUR PAIN DOMAIN 3]

```
[YOUR SIGNAL PHRASES]
```

*Add more pain domains as needed. Each should map to a real category of frustration your ICP experiences.*

---

## Section 2 — Institution-Led Queries

Group your target institutions and search for them combined with pain language. The scanner runs these as batched searches.

### [YOUR INSTITUTION GROUP 1 — e.g. "Enterprise customers"]

**Institutions:** [LIST YOUR TARGET COMPANIES/INSTITUTIONS]

**Combined search pattern:**
```
("[Institution A]" OR "[Institution B]" OR "[Institution C]")
  + [pain language from Section 1]
```

### [YOUR INSTITUTION GROUP 2 — e.g. "Mid-market targets"]

**Institutions:** [LIST]

### [YOUR INSTITUTION GROUP 3 — e.g. "Industry bodies / regulators"]

**Institutions:** [LIST]

---

## Section 3 — Context-Specific Queries

Where your ICP speaks publicly. Conferences, webinars, podcasts, and panels are where practitioners are most candid.

### Events and conferences

[LIST YOUR INDUSTRY EVENTS — e.g.]
- [Event 1] — [what it covers, when it runs]
- [Event 2]
- [Event 3]

**Search pattern:**
```
("[Event 1]" OR "[Event 2]" OR "[Event 3]")
  + ("said" OR "announced" OR "panel" OR "keynote" OR "discussed")
  + [pain language from Section 1]
```

### Podcasts and webinars

[LIST PODCASTS/WEBINARS YOUR ICP LISTENS TO OR APPEARS ON]

### Forums and communities

[LIST ONLINE COMMUNITIES WHERE YOUR ICP PARTICIPATES — e.g. specific subreddits, LinkedIn groups, Slack communities, industry forums]

### Publications

[LIST INDUSTRY PUBLICATIONS THAT COVER YOUR ICP'S WORLD]

---

## Section 4 — Scoring Tables

### Recency (weight 0.25)

| Score | Criteria |
|-------|----------|
| 10 | Published within last 30 days |
| 8 | Published within last 90 days |
| 6 | Published within last 6 months |
| 4 | Published within last 12 months |
| 2 | Published within last 24 months |
| 0 | Older than 24 months (excluded) |

### Specificity (weight 0.45)

| Score | Criteria |
|-------|----------|
| 9-10 | Named person + named institution + specific numbers/timelines + direct quote |
| 7-8 | Named institution + specific pain + attributable person |
| 5-6 | Named institution + general pain, or unnamed but very specific detail |
| 3-4 | Industry-level discussion with some useful detail |
| 1-2 | Vague frustration, no institutional context, generic language |

### Relevance (weight 0.30)

| Score | Criteria |
|-------|----------|
| 9-10 | Directly describes the problem your product solves, from a target institution |
| 7-8 | Strongly related pain domain from a target institution type |
| 5-6 | Related pain from a relevant industry |
| 3-4 | Tangentially related, but useful language or insight |
| 1-2 | Peripheral mention, limited direct relevance |

---

## Section 5 — Exclusion Rules

**Always exclude:**
- Your own company's content
- Content from your customers that promotes your product
- Vendor marketing disguised as customer voice
- Generic industry predictions without practitioner perspective
- Content older than 24 months
- AI hype articles with no institutional or practitioner context

**Source authority tiers** (for deduplication):
- **Tier 1:** Direct institutional publications, regulatory bodies, official conference recordings
- **Tier 2:** Major industry publications, established analyst firms
- **Tier 3:** Industry blogs, news aggregators, secondary coverage

When the same story is covered by multiple sources, keep the highest-tier source.
