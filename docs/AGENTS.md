# Agent Reference

Complete reference for all 24 GTM Content Engine agents, plus the optional VOC Scanner. Each agent has a single SKILL.md file under `skills/<category>/<name>/`.

---

## Pipeline Agents

These agents process content sequentially through the pipeline.

### 01 -- Scorer (`skills/pipeline/scorer/`)

**Purpose:** Maps content briefs to ICP personas, core beliefs, content pillars, and framing directions with confidence scores.

**Inputs:** `content_items` (status: `queued`), `icp_profiles`, `core_beliefs`, `content_pillars`, `market_signals`, `learned_adjustments`

**Outputs:** `content_items.scoring_result` (JSONB), status -> `scoring_complete`

**Schedule:** Pipeline step 1 (triggered by status change)

---

### 02 -- Format Selector (`skills/pipeline/format-selector/`)

**Purpose:** Chooses format, channel, structure, and delivery strategy based on performance data, algorithm knowledge, and cadence.

**Inputs:** `content_items` (scoring_result), `content_performance`, `algorithm_knowledge` (see frameworks/platform-algorithm.md), `format_constraints`, `learned_adjustments`

**Outputs:** `content_items.format_recommendation` (JSONB), status -> `shaping_complete`

**Schedule:** Pipeline step 2 (parallel with Evidence Curator and Repetition Monitor)

---

### 03 -- Evidence Curator (`skills/pipeline/evidence-curator/`)

**Purpose:** Selects evidence from six layers: market signals, company context, operational data, performance insights, and external sources.

**Inputs:** `content_items` (scoring_result), `market_signals`, `company_context_library`, `content_insights`, `content_performance`

**Outputs:** `content_items.evidence_package` (JSONB), status -> `curating_complete`

**Schedule:** Pipeline step 3 (parallel with Format Selector and Repetition Monitor)

---

### 04 -- Repetition Monitor (`skills/pipeline/repetition-monitor/`)

**Purpose:** Monitors five dimensions of content repetition (pillar, ICP, format, hook, structure) and flags potential echo issues.

**Inputs:** `content_items` (recent history), `content_pillars`, `content_insights`

**Outputs:** `content_items.echo_warnings` (JSONB), status -> `echo_complete`

**Schedule:** Pipeline step 4 (parallel), plus daily 7 AM sweep

---

### 07 -- Ghost Writer (`skills/pipeline/ghost-writer/`)

**Purpose:** Writes the draft in your configured voice profiles across all channels and formats — short-form posts, long-form articles, institutional content, and newsletters. Loads the relevant voice profile and applies signature devices, hooks, and structure per format.

**Inputs:** Assembled prompt from Prompt Assembler, `references/voice-profile.template.md`, `references/ghost-writer-foundation.md`, curation plan from Newsletter Curator (for newsletter formats)

**Outputs:** `content_items.draft` (text), status -> `draft_complete`

**Schedule:** Pipeline step 6 (routed by Orchestrator based on channel and format)

---

### 12 -- Validator (`skills/pipeline/validator/`)

**Purpose:** Two-layer quality gate: deterministic rules check + AI-powered dimensional assessment. Routes to approved, review_required, or revision_required.

**Inputs:** `content_items` (draft), `trust_tracker_state`, `authority_credentials`

**Outputs:** `content_items.validation_result` (JSONB), status -> `approved` / `review_required` / `revision_required`

**Schedule:** Pipeline step 7

---

## Independent Agents

These run on cron schedules or event triggers.

### 05 -- Contact Matcher (`skills/independent/contact-matcher/`)

**Purpose:** Selects target contacts from CRM and call intelligence data, matching to ICP profiles with cooldown and exclusion rules.

**Schedule:** Pipeline step 9 + weekly Monday 11 AM review

---

### 06 -- Distributor (`skills/independent/distributor/`)

**Purpose:** Builds three-stage distribution briefs (pre-publication, at-publication, post-publication) from Contact Matcher selections.

**Schedule:** Pipeline step 10

---

### 08 -- Learner (`skills/independent/learner/`)

**Purpose:** Observes human edits to drafts, detects patterns over time, and proposes system improvements at three speeds (observe, adjust, propose).

**Schedule:** Weekly Saturday + triggered by Path A completions

---

### 09 -- Engagement Collector (`skills/independent/engagement-collector/`)

**Purpose:** Ingests engagement data from scheduling tools and LinkedIn analytics uploads. Snapshots performance at 24h, 48h, and 7d.

**Schedule:** Event-driven (webhook or manual upload)

---

### 10 -- Researcher (`skills/independent/researcher/`)

**Purpose:** Daily external signal scan across industry sources. Friday convergence analysis synthesising weekly signals into market intelligence.

**Schedule:** Daily 8 AM + Friday convergence

---

### 11 -- Content Suggester (`skills/independent/content-suggester/`)

**Purpose:** Generates ranked daily content suggestions, manages content campaigns, and updates the content calendar.

**Schedule:** Daily 9 AM

---

### 13 -- Reporter (`skills/independent/reporter/`)

**Purpose:** Weekly synthesis report covering pipeline health, content performance, learning activity, distribution effectiveness, and market signals.

**Schedule:** Sunday 10 AM

---

### 14 -- Advisor (`skills/independent/advisor/`)

**Purpose:** On-demand human chat interface to query system state, ask questions about content, and route proposals to the learning queue.

**Schedule:** On-demand

---

### 15 -- Autonomy Manager (`skills/independent/autonomy-manager/`)

**Purpose:** Manages autonomy levels (1-3) for the Validator, tracking incidents, edit rates, and quality metrics. Level changes are human-gated for upgrades, automatic for safety downgrades.

**Schedule:** Sunday 8 AM

---

### 16 -- Call Analyst (`skills/independent/call-analyst/`)

**Purpose:** Processes call intelligence transcripts, extracts ICP-tagged signals, applies NDA gating, and promotes patterns to market signals.

**Schedule:** Event-driven (new transcripts) + weekly Saturday pattern review

---

### 18 -- Response Analyst (`skills/independent/response-analyst/`)

**Purpose:** Matches engagement data to distribution stages and builds segment profiles for targeting optimisation.

**Schedule:** Weekly Wednesday

---

## System Skills

Infrastructure agents with no learning loop.

### SYS-01 -- Router (`skills/system/router/`)

**Purpose:** Intake classifier. Creates `content_items` records with proper origin, input_type, and initial classification. All content enters through this front door.

---

### SYS-02 -- Signal Hygiene (`skills/system/signal-hygiene/`)

**Purpose:** Daily data hygiene: expires old signals, decays learned adjustments, archives stale suggestions, updates campaign statuses.

**Schedule:** Daily 6 AM

---

### SYS-03 -- Orchestrator (`skills/system/orchestrator/`)

**Purpose:** Coordinates the entire pipeline: manages status transitions, parallel step joins, human review paths (A and B), revision caps, and special routing for received content and newsletters.

---

### SYS-04 -- Newsletter Curator (`skills/system/newsletter-curator/`)

**Purpose:** Builds curation plans for all newsletter types from market signals and published content. Produces structured plans consumed by the Ghost Writer.

**Schedule:** Day before each newsletter send

---

### SYS-05 -- Prompt Assembler (`skills/system/prompt-assembler/`)

**Purpose:** Translates structured JSONB pipeline data (scoring, shaping, evidence, echo warnings) into prose briefs that Ghost Writers can consume directly.

---

### SYS-06 -- Approval Queue (`skills/system/approval-queue/`)

**Purpose:** Human approval gate between tentative and queued status. Manages the content calendar view and approval workflow.

---

### SYS-07 -- Visual Producer (`skills/system/visual-producer/`)

**Purpose:** Renders carousels to HTML/PDF and produces graphic briefs with metadata for visual content formats.

---

## Standalone (Optional)

Not part of the core 24. Run independently when you need it.

### VOC Scanner (`skills/standalone/voc-scanner/`)

**Purpose:** Mines the public web for how your ICP talks about their problems — the exact words, phrases, and framing practitioners use. Produces messaging fuel when you don't have customer call transcripts or CRM data to draw on.

**Schedule:** On-demand
