# VOC Scanner Setup

How to configure the Voice of Customer Scanner when you don't have access to customer call transcripts or CRM data.

If you have transcript data, use [Customer Data Setup](customer-data-setup.md) instead.

---

## What this does

The **VOC Scanner** (`skills/standalone/voc-scanner/`) mines the public web for how your ICP talks about their problems — the exact words, phrases, metaphors, and framing practitioners use when they describe their pain.

This is language mining for marketing. When a CTO says "we get 15 use case requests a month and can serve maybe 3", that's not just a signal — it's copy. When a Head of Innovation says "every new initiative feels like starting from scratch", that's a headline.

The primary output is the **language itself** — vivid, reusable phrasing captured verbatim. The secondary output is classification (who said it, which persona, which pain domain) so you can target the right language to the right audience.

---

## Prerequisites

| Requirement | Purpose |
|------------|---------|
| Web search capability | The scanner searches the public web |
| Database (Supabase) | Stores captured signals |
| Populated ICP framework | Scanner needs to know who to listen for |

---

## Setup steps

### 1. Populate your ICP framework first

The VOC Scanner uses your ICP personas to classify who is speaking. Complete `frameworks/icp-framework.md` before running the scanner.

### 2. Update the VOC Scanner references

The scanner has its own reference files in `skills/standalone/voc-scanner/references/`:

**`icp-pain-framework.md`** — condensed ICP reference for the scanner. Update with your persona types, their titles, and their pain points. The scanner uses this to identify who is speaking and classify what they care about.

**`voc-search-framework.md`** — defines what signal language looks like, where to search, and how to score results. Update the search queries with your industry context:
- Replace institution lists with your target companies
- Replace pain language with terms relevant to your market
- Replace conference/event lists with events your ICP attends

### 3. Update the skill file

Open `skills/standalone/voc-scanner/SKILL.md` and update:
- `[YOUR COMPANY]` references
- `[YOUR DATABASE PROJECT]` and `[YOUR PROJECT ID]` with your Supabase details
- Pain domains relevant to your market
- Signal types relevant to your business

### 4. Create the database table

Run the `voc_signals` CREATE TABLE from `architecture/schema.md`.

### 5. Deploy and run

Load `skills/standalone/voc-scanner/SKILL.md` into your AI platform. The scanner needs web search capability — it actively searches the internet for practitioner voice signals.

Run it:
- **Daily** for continuous language monitoring
- **Weekly** for periodic refreshes
- **On demand** when you need messaging fuel for a specific campaign

---

## What the scanner produces

Each signal captured includes:

| Field | What it is |
|-------|-----------|
| `quote_snippet` | The most vivid, reusable phrasing — exact words from the source |
| `persona_type` | Which ICP persona is speaking |
| `pain_domain` | What category of problem they're describing |
| `summary` | 2-3 sentence context |
| `relevance` | How directly relevant to your company |
| `score_specificity` | How detailed and attributable the signal is |

---

## How VOC signals feed the content pipeline

The VOC Scanner is standalone — it doesn't directly feed the main pipeline. But you can use its output to:

1. **Inform the Researcher** — use captured language patterns to guide what external signals to scan for
2. **Enrich the Evidence Curator's evidence** — manually promote the strongest signals to `market_signals` or `context_library`
3. **Fuel the Ghost Writer** — reference specific ICP language in content briefs submitted through the Router
4. **Refine your ICP framework** — update pain points and language resonance based on what the scanner captures

---

## Quality controls

The scanner enforces strict quality rules:

- **Verbatim only** — quote_snippet must be exact words from the source, never paraphrased
- **Date verification mandatory** — publication date must be confirmed, never estimated
- **24-month cutoff** — signals older than 24 months are excluded regardless of quality
- **Attribution accuracy** — who is speaking must be verified, not inferred from who the article is about
- **No vendor marketing** — only authentic practitioner voice, never disguised marketing
