# GTM Content Engine — Architecture Document

**Version:** 1.0  
**Date:** June 2026  
**System:** Multi-agent content operations platform for [YOUR COMPANY]

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Design Philosophy](#2-design-philosophy)
3. [Agent Inventory](#3-agent-inventory)
4. [Core Content Pipeline](#4-core-content-pipeline)
5. [Pipeline Flow: Step-by-Step](#5-pipeline-flow-step-by-step)
6. [Special Pipeline Routes](#6-special-pipeline-routes)
7. [Human Review Flow](#7-human-review-flow)
8. [Independent Agents (Cron & Event-Driven)](#8-independent-agents-cron--event-driven)
9. [Learning System](#9-learning-system)
10. [Intelligence & Data Architecture](#10-intelligence--data-architecture)
11. [Ghost Writer Routing](#11-ghost-writer-routing)
12. [Trust & Autonomy System](#12-trust--autonomy-system)
13. [Scheduling & Cron](#13-scheduling--cron)
14. [External Integrations](#14-external-integrations)
15. [Data Flow Diagrams](#15-data-flow-diagrams)
16. [Failure Handling Philosophy](#16-failure-handling-philosophy)
17. [Database & Framework Reference](#17-database--framework-reference)
18. [Missing Tables](#18-missing-tables)
19. [What's Not in This Suite](#19-whats-not-in-this-suite)

---

## 1. System Overview

GTM Content Engine is a multi-agent content operations system built for [YOUR COMPANY]. It produces ICP-targeted thought leadership content across 11 channels, with a learning feedback loop driven by [YOUR FOUNDER/CEO]'s ([YOUR COMPANY]'s CEO) editing patterns.

The system is composed of **24 agent skills** and **3 reference documents**. Agents fall into three categories: pipeline agents that process content sequentially, independent agents that run on cron schedules or event triggers, and system skills that handle infrastructure coordination.

**The core promise:** Every piece of content enters through a single front door (the Router), flows through a structured pipeline of scoring, shaping, evidence curation, drafting, and validation, exits through human review, and feeds its performance data back into the system so it gets smarter over time.

### What the System Produces

- **LinkedIn posts** for [YOUR FOUNDER] (daily, thought leadership)
- **LinkedIn posts** for [YOUR COMPANY] (daily, institutional)
- **[YOUR NEWSLETTER PLATFORM] newsletter** (weekly, [YOUR FOUNDER/CEO]'s personal)
- **[YOUR PUBLICATION] articles** (monthly, industry)
- **Blog articles** (monthly, deep-dive)
- **[YOUR NEWSLETTER] newsletter** (weekly, broad industry)
- **5 ICP pipeline newsletters** (weekly, targeted to [Persona A], [Persona B], [Persona C], [Persona D], [Persona E] Economic Buyers)

---

## 2. Design Philosophy

### Signals, Not Facts

Every confidence score in the system is a percentage. No agent claims certainty. The Scorer produces probabilistic ICP matches. The Learner writes "soft guidance" with confidence weights. When no good match exists, agents say so rather than forcing a fit.

### Agent Separation of Concerns

Each agent has a narrow, well-defined responsibility. The Scorer does not recommend formats. The Format Selector does not assess ICP fit. The Ghost Writer never touches the database or reads JSONB. The Validator evaluates but never rewrites. This separation is enforced by design: agents receive only the upstream outputs they need, never raw framework data.

### Human-in-the-Loop

The system never publishes without the possibility of human intervention. The Autonomy Manager governs autonomy levels, and even at the highest trust level, 20% of auto-approved content is spot-checked. Level upgrades require explicit human approval. Level downgrades (safety) can happen immediately.

### Learning Has Speed Limits

A single atypical edit cannot change the system. The Learner requires 5+ observations before creating a runtime adjustment, and 10+ observations over 4+ weeks before proposing a permanent skill change. Permanent changes are always human-gated.

### Honest Gaps Over Silent Failures

Every agent is instructed to flag gaps rather than force weak outputs. The Evidence Curator flags thin evidence rather than padding with irrelevant data. The Scorer flags low ICP relevance rather than forcing a fit. Gap flags travel downstream so the Ghost Writer can work around them.

---

## 3. Agent Inventory

### Pipeline Agents (Process Content Sequentially)

| ID | Agent | Category | Pipeline Step | Role |
|---|---|---|---|---|
| 01 | Scorer | B | Step 1 | ICP scoring, belief selection, pillar alignment, framing direction |
| 02 | Format Selector | B | Step 2 | Format, channel, structure, delivery strategy |
| 03 | Evidence Curator | A | Step 3 | Evidence selection, anti-repetition, [YOUR COMPANY] context, gap flagging |
| 04 | Repetition Monitor | B | Step 4 | 5-dimension repetition monitoring |
| 07 | Ghost Writer | B | Step 6 | [YOUR FOUNDER/CEO]'s LinkedIn voice |
| 07 | Ghost Writer | B | Step 6 | [YOUR FOUNDER/CEO]'s [YOUR NEWSLETTER PLATFORM] + [YOUR PUBLICATION] voice |
| 07 | Ghost Writer | B | Step 6 | [YOUR COMPANY] LinkedIn + blog voice |
| 07 | Ghost Writer | B | Step 6 | [YOUR NEWSLETTER] + 5 ICP pipeline newsletters |
| 12 | Validator | B | Step 7 | 2-layer quality gate (deterministic rules + AI assessment) |

### System Skills (Infrastructure, No Learning)

| ID | Agent | Pipeline Position | Role |
|---|---|---|---|
| SYS-01 | Router | Step 0 | Intake, classification, record creation |
| SYS-02 | Signal Hygiene | Daily 6 AM cron | Data hygiene — expiry, decay, cleanup |
| SYS-03 | Orchestrator | Continuous | Pipeline coordination, status management, review paths |
| SYS-04 | Newsletter Curator | Weekly (before send) | Curation plan for 7 newsletters |
| SYS-05 | Prompt Assembler | Step 5b | Translates structured JSONB into prose briefs for Ghost Writers |
| SYS-06 | Approval Queue | Pre-pipeline | Approval gate, calendar view, workflow surface |
| SYS-07 | Visual Producer | Step 6.5 | Carousel rendering, graphic briefs |

### Independent Agents (Cron + Event-Driven)

| ID | Agent | Category | Trigger | Role |
|---|---|---|---|---|
| 05 | Contact Matcher | A | Post-approval | Contact finding, ICP matching, comms preferences |
| 06 | Distributor | A | After Contact Matcher | 3-stage distribution briefs |
| 08 | Learner | A | Post-review + weekly | Observes edits, detects patterns, proposes skill changes |
| 09 | Engagement Collector | B | Ongoing + weekly upload | Engagement data capture from [YOUR SCHEDULING TOOL] + LinkedIn |
| 10 | Researcher | A | Daily 8 AM | External signal scanning |
| 11 | Content Suggester | A | Daily 9 AM | Content suggestions, campaigns, calendar planning |
| 13 | Reporter | B | Sunday 10 AM | Weekly intelligence report |
| 14 | Advisor | B | On-demand | Human chat interface to system state |
| 15 | Autonomy Manager | B | Sunday 8 AM | Autonomy level assessment |
| 16 | Call Analyst | A | [YOUR TRANSCRIPT PROVIDER] webhook | Sales call transcript extraction, pattern promotion |
| 18 | Response Analyst | A | Event + Wednesday | Engagement analysis, segment profiling |

### Agent Categories Explained

**Category A — Autonomous Learner:** Has its own scratchpad, runs weekly self-analysis, shares learnings automatically with other agents. Skills change only through human-gated edits. Agents: Evidence Curator, Contact Matcher, Distributor, Learner, Researcher, Content Suggester, Call Analyst, Response Analyst.

**Category B — Informed Executor:** Reads shared intelligence from Category A agents. Improves through human skill edits and richer shared data. Agents: Scorer, Format Selector, Repetition Monitor, Ghost Writers, Validator, Engagement Collector, Reporter, Advisor, Autonomy Manager.

**System Skills:** Infrastructure. No learning loop. Coordination, routing, hygiene, rendering.

---

## 4. Core Content Pipeline

```
                                  ┌──────────────────────────────────┐
                                  │         CONTENT SOURCES          │
                                  │  Human submission · Content Suggester    │
                                  │  Campaign pieces · External      │
                                  │  Newsletter Curator              │
                                  └────────────────┬─────────────────┘
                                                   │
                                                   ▼
                              ┌─────────────────────────────────────────┐
                   Step 0     │  SYS-01 ROUTER                         │
                              │  Classify origin, input type, create    │
                              │  content_items record (status: tentative)│
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 0     │  SYS-06 CALENDAR & QUEUE               │
                              │  Human approves → status: queued        │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 1     │  01 SCORER                              │
                              │  ICP match, beliefs, pillar, framing    │
                              │  → status: scoring_complete             │
                              └────────────────┬────────────────────────┘
                                               │
                          ┌────────────────────┼────────────────────┐
                          ▼                    ▼                    ▼
               ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
   Steps 2-4   │  02 SHAPER      │ │  03 CURATOR      │ │  04 ECHO WATCHER │
  (parallel)   │  Format, channel│ │  Evidence         │ │  Repetition      │
               │  structure      │ │  selection        │ │  monitoring      │
               └────────┬────────┘ └────────┬──────────┘ └────────┬─────────┘
                        │                   │                     │
                        └───────────────────┼─────────────────────┘
                                            │ (all three must complete)
                                            ▼
                              ┌─────────────────────────────────────────┐
                   Step 5a    │  SYS-03 ORCHESTRATOR                   │
                              │  Verify all fields populated            │
                              │  → status: brief_assembled             │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 5b    │  SYS-05 PROMPT ASSEMBLER               │
                              │  JSONB → prose brief for Ghost Writer   │
                              │  → status: prompt_assembled            │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 6     │  07 GHOST WRITER      │
                              │  Reads assembled_prompt, writes draft   │
                              │  → status: draft_complete              │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 6.5   │  SYS-07 VISUAL PRODUCER (conditional)  │
                              │  Carousels, graphic briefs              │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 7     │  12 VALIDATOR                           │
                              │  Rules (Layer 1) + AI (Layer 2)         │
                              │  → approved / review_required /         │
                              │    revision_required                    │
                              └──────┬─────────┬───────────┬────────────┘
                                     │         │           │
                          ┌──────────┘         │           └──────────┐
                          ▼                    ▼                      ▼
                   ┌──────────┐      ┌──────────────────┐    ┌──────────────┐
                   │ APPROVED │      │ HUMAN REVIEW     │    │ REVISION     │
                   │          │      │ Path A or Path B │    │ LOOP         │
                   └────┬─────┘      └────────┬─────────┘    │ → Prompt     │
                        │                     │              │   Assembler  │
                        │                     │              │ → Ghost      │
                        │                     ▼              │   Writer     │
                        │              ┌──────────────┐      │ → Validator  │
                        │              │ APPROVED     │      └──────────────┘
                        │              └──────┬───────┘
                        └──────────┬──────────┘
                                   │
                                   ▼
                              ┌─────────────────────────────────────────┐
                   Step 9     │  05 PEOPLE ENGINE                      │
                              │  Find ICP-matched contacts              │
                              └────────────────┬────────────────────────┘
                                               │
                                               ▼
                              ┌─────────────────────────────────────────┐
                   Step 10    │  06 DISTRIBUTOR                         │
                              │  Pre-pub + At-launch + Post-pub briefs  │
                              │  → status: published                   │
                              └────────────────────────────────────────┘
```

---

## 5. Pipeline Flow: Step-by-Step

### Step 0: Intake (Router + Approval Queue)

**Router (SYS-01)** is the front door. Every piece of content enters through it. Inputs arrive from four sources:

1. **Human submission** — raw brief, source material, or finished content via the frontend
2. **Content Suggester actioned** — a daily suggestion approved by a human
3. **Campaign piece** — a child piece from an approved campaign
4. **External submission** — guest posts, partner content

The Router classifies each submission along two dimensions:

- **Origin:** `brief` (direction to write from), `source_generate` (source material provided), `received` (finished content for scoring only)
- **Input type:** `standard` (standalone), `hero` (campaign parent), `hero_child` (campaign child piece)

It creates a `content_items` record at status `tentative` with `calendar_status: tentative`. The record includes any pre-specified attributes (channel, format, campaign ID, suggestion ID) passed through without modification.

**Approval Queue (SYS-06)** surfaces the tentative item. A team member reviews and can:

- **Approve** → `calendar_status: approved_topic`, pipeline `status: queued`. Orchestrator picks it up.
- **Dismiss** → removed from calendar. If from Content Suggester, dismissal flows back.
- **Kill** → pipeline terminates for this piece.

Edge cases: duplicate detection (similar `raw_input` within 5 minutes), empty submissions rejected, campaign children respect sequence order.

### Step 1: Scoring

**Scorer (01)** determines who the content is for, what it believes, and which content pillar it serves. It does not bulk-load all frameworks — it reads the input first, then queries 2-4 relevant ICP profiles, related core beliefs, and candidate content pillars.

Output written to `content_items.scoring_result` (JSONB):
- **Primary ICP** with confidence percentage (0-100)
- **Secondary ICP** if above 30% confidence
- **1-3 selected core beliefs** with expression strength (strong/moderate/light)
- **Pillar alignment** — against `content_pillars`
- **Framing direction** incorporating `learned_adjustments` from the Learner
- **Market context** from `market_intelligence`

Key principle: if no ICP scores above 40%, the Scorer flags `low_icp_relevance` rather than forcing a fit.

### Steps 2-4: Parallel Processing (Format Selector + Evidence Curator + Repetition Monitor)

These three agents run simultaneously after scoring completes. The Orchestrator tracks completion by field presence (not by status, since a single status field cannot hold three concurrent values).

**Format Selector (02)** decides format, channel, and delivery approach. Format follows content type and pillar, not ICP. It classifies content into one of 16 subcategories, checks format performance data, anti-repetition state, platform algorithm signals, and channel cadence. Output includes specific structural guidance (word count, slide count, hook approach, [YOUR COMPANY] integration point).

**Evidence Curator (03)** selects evidence across six layers:
1. Context Library items (with anti-repetition scoring)
2. [YOUR COMPANY] operational data (flagging anything stale >30 days)
3. [YOUR COMPANY] Context Library (platform capabilities, product claims — the Evidence Curator alone decides which [YOUR COMPANY] props enter a piece)
4. Internal market signals (sales call patterns)
5. External market signals (Researcher findings, regulatory updates)
6. Gap assessment (honest gaps flagged, never filled with weak evidence)

**Repetition Monitor (04)** monitors five dimensions of repetition:
1. Pillar concentration (strict tracking for LinkedIn channels only, 40% threshold)
2. ICP coverage (flag if untargeted for 2+ weeks)
3. Format variety (flag if 3+ consecutive same format)
4. Hook pattern repetition (flag if same hook type 3+ times in 7 days)
5. Structural monotony (flag if 3+ pieces share the same skeleton)

All flags are advisory — the Repetition Monitor reports but never blocks.

**Timeout:** 10 minutes. If any agent hasn't completed, the Orchestrator flags for investigation.

### Step 5a: Brief Assembly

The **Orchestrator (SYS-03)** verifies all three parallel outputs are present and well-formed. Status moves to `brief_assembled`. The "brief" is the combination of `scoring_result` + `format_recommendation` + `evidence_package` + `echo_warnings` already on the content record.

### Step 5b: Prompt Assembly

**Prompt Assembler (SYS-05)** is the translator between the pipeline's structured JSONB world and the Ghost Writer's prose world. It:

- Strips system noise (IDs, confidence percentages, pillar codes, database terminology)
- Resolves conflicts between agent outputs (e.g., Repetition Monitor flagging a hook the Format Selector recommended — Repetition Monitor wins)
- Checks `algorithm_knowledge` (see frameworks/platform-algorithm.md) live for current platform conventions
- Writes a prose brief structured as: AUDIENCE → ARGUMENT → EVIDENCE → MARKET CONTEXT → FORMAT → CHANNEL → PER-PIECE VOICE NOTES

Critical boundary: permanent voice rules live in the Ghost Writer skills. The Prompt Assembler delivers only what changes per piece.

### Step 6: Ghost Writer (Routed)

The Orchestrator routes to one of the Ghost Writer based on the `channel` field:

| Channel | Ghost Writer |
|---|---|
| [your-channel-1] | 07 Ghost Writer |
| [your-channel-2] | 07 Ghost Writer |
| [your-channel-3] | 07 Ghost Writer |
| [your-channel-4] | 07 Ghost Writer |

The Ghost Writer reads `assembled_prompt` only — never the structured JSONB, never the database. All four load `references/ghost-writer-foundation.md` for shared rules (UK English, banned phrases, evidence handling). Each carries its own permanent voice rules.

**Timeout:** 15 minutes (generation task, not deterministic translation).

### Step 6.5: Visual Producer (Conditional)

**Visual Producer (SYS-07)** triggers only when:
- Format is `carousel` → carousel mode
- Channel is `linkedin-[your-company]` → graphic brief mode
- `format_detail.visual_required` is true → graphic brief mode

Visual Producer failure does not block publication — it proceeds to Validator with a flag.

### Step 7: Validation

**Validator (12)** runs two independent layers:

**Layer 1 — Deterministic Rules (pass/fail):**
1. UK English spelling (scan for US variants)
2. Banned phrases (salesy, generic AI, clichés, hyperbolic)
3. Character/word count against format constraints
4. [YOUR COMPANY] mention count (max 1 unless format warrants more) + factual accuracy against `[your-company]_context_library`
5. Structure compliance against Format Selector's guidance
6. Link/CTA appropriateness per channel conventions

**Layer 2 — AI Assessment (scored 0-100 per dimension):**
1. Brand voice quality (sentence rhythm, specificity, tone)
2. [YOUR COMPANY] integration quality (natural vs forced)
3. Content differentiation from last 7 days of output
4. Brief alignment (ICP match, evidence integration, belief expression)

**Composite Score** (weighted):
- AI self-assessment: 20%
- ICP match from Scorer: 25%
- Layer 1 pass rate: 20%
- Context richness: 15%
- Differentiation: 20%

**Routing** (governed by Autonomy Manager autonomy level):

| Outcome | Level 2 (Standard) | Level 3 (High Trust) |
|---|---|---|
| Auto-approve | ≥85 + all L1 pass | ≥80 + all L1 pass (20% spot-check) |
| Human review | 60-84 or any L1 flag | 50-79 or any L1 flag |
| Revision loop | <60 or L1 hard fail | <50 or L1 hard fail |

At Level 1 (Full Review), everything goes to human review regardless.

---

## 6. Special Pipeline Routes

### Received Content (origin = "received")

Finished content submitted for scoring and validation, not generation.

```
Router → Scorer → Repetition Monitor only → Validator → Human Review
```

Skips: Format Selector, Evidence Curator, Prompt Assembler, Ghost Writer. The submitted content IS the draft — `raw_input` is copied to the `draft` field.

### Newsletter Editions (origin = "newsletter_curation")

The Newsletter Curator (SYS-04) has already handled ICP targeting, evidence selection, and content curation.

```
Router → Prompt Assembler → Ghost Writer → Validator → Human Review
```

Skips: Steps 1-4 (Scorer, Format Selector, Evidence Curator, Repetition Monitor). The Prompt Assembler reads `curation_plan` instead of the standard JSONB fields.

---

## 7. Human Review Flow

When the Validator routes to `review_required`, the human reviewer has three options:

### Path A: Direct Editing (Delta Capture)

1. Human opens draft in edit mode and makes changes
2. System preserves the original draft
3. Human saves → clicks "Complete" → status: `approved`
4. The **edit delta** (original vs edited version) flows to the Engagement Collector, which writes it to `content_performance.edit_delta`
5. The Learner (08) is triggered for Speed 1 observation — it reads both versions and writes a free-form synopsis

This is the primary learning signal. The gap between what the AI wrote and what [YOUR FOUNDER] published is the lesson.

### Path B: Verbal Notes (Re-stimulation)

1. Human writes notes: "Soften the opening hook. Move the evidence reference earlier."
2. Human clicks "Send Back" → status: `revision_required_human_notes`
3. Prompt Assembler re-triggered with: original brief + current draft + human notes (verbatim, never summarised) + Validator feedback
4. Ghost Writer produces new draft → returns to Validator (Step 7)

### Path Combinations

A single review can involve both paths: Path B first (send notes → revision), then Path A (direct final edits). The delta captured is between the Ghost Writer's final draft and the human's final edited version — the most useful signal for the Learner.

### Approve Without Changes

Human clicks "Approve" directly. No delta. The Learner logs a clean approval (a positive data point).

### Kill

Human clicks "Kill" → status: `killed`. Reason logged. Pipeline terminates.

### Revision Loop Management

- **1st-2nd revision:** Normal re-trigger
- **3rd revision:** Flag — recommend Path A editing or killing the piece
- **4th revision attempt:** Do not re-trigger. Escalate to manual writing or kill.

---

## 8. Independent Agents (Cron & Event-Driven)

### Researcher (10) — Daily 8 AM

The Researcher is the system's external sensor. Every morning it scans the outside world for signals that matter to [YOUR COMPANY]'s audience and feeds them into `market_signals` for downstream consumption.

**Input: Industry Map.** The Researcher does not work from a fixed list of URLs. It queries `industry_map` from the database — a reference that defines the terrain (sectors, signal categories, ICP-relevant themes) but not specific sources. The Researcher decides where to search based on what's happening.

Sectors monitored: defined in your industry_map table (populated from frameworks/industry-landscape.md).

Signal categories: regulatory developments, competitor activity, industry trends, technology shifts, events/conferences, market consolidation.

ICP-relevant themes: vendor management and third-party risk, innovation programme management, regulatory compliance and reporting, technology architecture and modernisation, procurement and onboarding processes.

**Context loading** is deliberately lean — ICP summaries (names and key pain points, not full profiles), pillar names from `content_pillars`, core belief names, and recent scratchpad entries for deduplication and source quality tracking.

**Process (5 steps):**

1. **Scan** — open-ended search guided by the industry map across industry press, regulatory announcements, competitor sites, analyst reports, conference announcements, technology publications, and social media signals from industry figures.

2. **Assess** each signal on three dimensions:
   - *Relevance* — which ICPs care? Which pillars? Which beliefs? Must connect to at least one ICP and one pillar or it's noise.
   - *Urgency* — classified as `time_sensitive` (days), `current` (weeks), `developing` (months), `evergreen`, or `background`.
   - *Quality* — source credibility, significance, historical reliability (checked against scratchpad notes).

3. **Filter** — pass signals that are relevant, credible, and non-duplicate. Reject noise, low-quality sources, and duplicate coverage of the same story.

4. **Write structured signals** to `market_signals` with: summary, detail, source URL/name, relevant ICPs, relevant pillars, relevant beliefs, urgency, confidence, and status. Signal types follow a locked taxonomy: `regulatory`, `competitive`, `industry_trend`, `technology`, `event`, `partnership`, `market_shift`, `pain_point`, `objection`.

5. **Flag convergence candidates** — when multiple signals point to the same theme ("third regulatory signal on [YOUR COMPLIANCE PROCESS] in two weeks"), note it in the scratchpad for the Market Analyst's Friday convergence analysis.

**Downstream consumers:**
- **Content Suggester** (runs 1 hour later at 9 AM) — reads time-sensitive signals as content opportunities
- **Evidence Curator** (next pipeline run) — selects signals as external evidence for content pieces
- **Market Analyst** (Friday) — synthesises for convergence patterns
- **Newsletter Curator** — market intelligence for newsletter editions

**Weekly self-analysis (Saturday):** Tracks signal utility (which got used downstream, which were ignored), source quality trends (reliable vs noisy), industry map coverage (any blind spots or emerging themes), and updates source quality weights.

**Learning sharing:** Signals are written directly to `market_signals` — automatic, no human gate. Source health and coverage notes are available in the scratchpad for the Advisor.

**Known functional gaps in current skill spec:**
- **No search execution spec** — describes what to look for but not how the agent actually accesses external information (no search APIs, RSS feeds, or web scraping integrations defined). Implementation blocker.
- **No signal volume guidance** — no floor or ceiling on daily output. Downstream consumers can't anticipate load.
- **No intra-day re-trigger** — runs once at 8 AM with no event-driven trigger for breaking news. Every other time-sensitive agent (Call Analyst, Engagement Collector) has event-driven capabilities.
- **No signal update/correction mechanism** — once written to `market_signals`, no process for same-day retraction or correction if information changes.
- **No coordination with Signal Hygiene (SYS-02)** — SYS-02 runs at 6 AM expiring old signals. No guidance on whether the Researcher should check expired signals when deduplicating.
- **Thin failure handling** — covers 3 scenarios vs the 5-6 most agents define. Missing: `market_signals` write failure, `core_beliefs` query failure, external source rate limiting, partial run failure.

### Content Suggester (11) — Daily 9 AM

Has the broadest view of any agent. Operates at three levels:

1. **Daily suggestions** — ranked content ideas based on signals, gaps, and performance. Reads ICP gaps, pillar gaps, format variety needs, evidence gaps, calendar gaps, and newsletter pipeline gaps. Writes to `daily_content_suggestions`.
2. **Campaign planning** — when signals converge, plans multi-piece campaigns (hero asset + child content + channel sequence + timeframe). Writes to `content_campaigns`.
3. **Content calendar** — maintains a rolling view of planned vs published content.

### Call Analyst (16) — [YOUR TRANSCRIPT PROVIDER] Webhook

Extracts structured intelligence from sales call transcripts: objections, pain points, buying signals, questions, sentiment. Maps signals to ICP framework. Promotes cross-call patterns (3+ calls, 2 weeks) to `market_signals` (source_type: "internal"). Strict NDA guardrail: never allows company names, individual names, or deal specifics into `market_signals`.

### Engagement Collector (09) — Ongoing + Weekly Upload

Captures engagement data from two sources:
1. **[YOUR SCHEDULING TOOL]** (ongoing feed) — post-level metrics
2. **Weekly LinkedIn analytics upload** (manual, richer data, authoritative for LinkedIn metrics)

Matches data to `content_items` records, normalises into `content_performance`, triggers Response Analyst. Also handles edit delta writes for Path A reviews.

### Response Analyst (18) — Event + Wednesday

Studies how people respond to content. Matches engagement to distribution briefs. Classifies engagement quality (deep/standard/passive). Builds segment engagement profiles. Identifies individual consistent engagers. Updates `content_insights` with segment patterns.

### Contact Matcher (05) — Post-Approval

Searches [YOUR CRM], LinkedIn network lists, and [YOUR TRANSCRIPT PROVIDER] data to find contacts matching the content's target ICP. Cross-references against [YOUR COMPANY] internal team (always excluded), pipeline/deal stages (flagged for sensitivity), and ecosystem position (pre-pipeline, active, post-sale). Enforces 7-day cooldown per contact. Produces ranked list with per-contact synopses.

### Distributor (06) — After Contact Matcher

Takes Contact Matcher output and builds three distribution briefs:
1. **Pre-publication** (30 min before publish) — algorithmic priming engagement list
2. **At-launch** — targeted sharing via tags (max 5 per LinkedIn post) and DMs, respecting comms preferences and ecosystem position
3. **Post-publication** (2-4 hours after) — personalised outreach digest for human distribution person

### Reporter (13) — Sunday 10 AM

Reads the entire week's system state across all agent scratchpads and data tables. Produces a single intelligence report covering: content pipeline summary, performance highlights, market intelligence, learning activity, distribution effectiveness, pipeline health metrics, ICP/pillar coverage, and pending action items.

### Advisor (14) — On-Demand

Human chat interface to system state. Answers questions grounded in system data ("What should I write about?", "Why did that post underperform?", "Should we adjust ICP targeting?"). Can write structured suggestions to `learning_staging_queue` for human approval when conversations lead to concrete proposals.

### Autonomy Manager (15) — Sunday 8 AM

Assesses system autonomy level based on edit rates, auto-approve quality, and trust incidents. Proposes level changes. Upgrades always require human approval. Downgrades (safety) can happen immediately.

### Signal Hygiene (SYS-02) — Daily 6 AM

Data hygiene: expires time-sensitive signals past their window, decays confidence on ageing intelligence, cleans up orphaned records. Runs before all other agents to ensure they work with current data.

### Newsletter Curator (SYS-04) — Weekly (Before Send)

Curates editions for 7 newsletters: [YOUR NEWSLETTERS] (configured in distribution-channels.md). Builds a `curation_plan` JSONB (edition theme, featured content, market intelligence, AI TLDR, LLM trick) that enters the pipeline via Router → Prompt Assembler → Ghost Writer.

---

## 9. Learning System

The learning system operates at three speeds with increasing impact and increasing human oversight:

### Speed 1: Observe (Per Review, Immediate)

**Trigger:** Human edits a draft (Path A completion).

The **Learner (08)** reads both the original draft and the published version. It writes a free-form synopsis — its interpretation of what changed and why. This is stored in `agent_scratchpads` (where `agent_id = 'learner'`) and the file-based memory system (`memory/learning/observations/`).

**No system changes.** Speed 1 is pure observation. The synopsis accumulates.

### Speed 2a: Adjust (Weekly, Automatic)

**Trigger:** Saturday weekly analysis.

The Learner reviews 4 weeks of observations (~20 synopses), groups by context (ICP x format x pillar x change type), and detects patterns.

- **3+ instances** → emerging pattern, noted
- **5+ instances** → `learned_adjustment` created (automatic, no human gate)

Learned adjustments are soft guidance with confidence scores. Pipeline agents treat them as weighted signals. Example: "For [Persona A] — Champion on [Persona E]y Compliance, soften contrarian openings" with confidence 0.68.

**Decay:** Adjustments not reinforced by new observations in 60 days lose confidence. Below 0.3 confidence with no reinforcement in 90 days → archived.

### Speed 2b: Evolve (Weekly, Human-Gated)

**Trigger:** 10+ instances of the same pattern over 4+ weeks.

The Learner writes a skill evolution proposal to `learning_staging_queue` with provenance traces from pipeline records. This sits pending until human review. If approved, the relevant `claude_skill_modules` entry gets a new version (previous preserved for rollback), and the corresponding learned_adjustment can be archived.

### Speed 3: Approved Suggestions

**Trigger:** Human approves a suggestion from the Advisor.

The Learner routes the approved change to the appropriate table (ICP refinements → `icp_profiles`, belief refinements → `core_beliefs`, etc.).

### Category A Self-Analysis

Every Category A agent (Evidence Curator, Contact Matcher, Distributor, Learner, Researcher, Content Suggester, Call Analyst, Response Analyst) runs a weekly self-analysis on Saturday:

1. Pull scratchpad entries from past 4 weeks with outcomes
2. Correlate decisions with results
3. Detect patterns (min 5 instances)
4. Write distilled learnings to shared brain (`content_insights` or `content_performance`)

These are automatic runtime learnings — they update shared intelligence without human gating. They never change frameworks or skills.

---

## 10. Intelligence & Data Architecture

### Central Data Store: PostgreSQL (24 tables + 6 to create)

The central record is `content_items`. Every piece of content has one record that accumulates data through the pipeline:

```
content_items
├── id (UUID)
├── origin: brief | source_generate | received | newsletter_curation
├── input_type: standard | hero | hero_child
├── raw_input (text)
├── status (pipeline status, managed by Orchestrator)
├── calendar_status (human-facing, managed by Approval Queue)
├── channel
├── format
├── scoring_result (JSONB) ← Scorer
├── format_recommendation (JSONB) ← Format Selector
├── evidence_package (JSONB) ← Evidence Curator
├── echo_warnings (JSONB) ← Repetition Monitor
├── assembled_prompt (text) ← Prompt Assembler
├── assembled_prompt_metadata (JSONB) ← Prompt Assembler
├── draft (text) ← Ghost Writer
├── validation_result (JSONB) ← Validator
├── visual_assets (JSONB) ← Visual Producer
├── curation_plan (JSONB) ← Newsletter Curator
└── ... (dates, relationships, revision tracking)
```

### Key Data Tables and Ownership

| Table | Written By | Read By |
|---|---|---|
| `content_items` | Router, Orchestrator, Scorer, Format Selector, Evidence Curator, Repetition Monitor, Prompt Assembler, Ghost Writers, Validator, Visual Producer, Approval Queue | All agents |
| `content_performance` | Engagement Collector, Response Analyst | Learner, Reporter, Advisor, Content Suggester, Autonomy Manager |
| `content_insights` | Repetition Monitor, Evidence Curator, Response Analyst, Learner | Scorer, Format Selector, Content Suggester, Reporter, Advisor, Contact Matcher |
| `market_signals` | Researcher, Call Analyst | Scorer, Evidence Curator, Content Suggester, Market Analyst, Newsletter Curator |
| `learned_adjustments` | Learner | Scorer, Format Selector, Prompt Assembler |
| `trust_tracker_state` | Autonomy Manager | Validator |
| `daily_content_suggestions` | Content Suggester | Approval Queue, Advisor |
| `content_campaigns` | Content Suggester | Router, Orchestrator |
| `distribution_engagement_briefs` | Distributor | Response Analyst, Reporter |
| `learning_staging_queue` | Learner, Advisor | Human review, Learner (routing) |
| `scratchpad_*` (per agent) | Each Category A agent | Self, Reporter, Advisor |

### Intelligence Flow Patterns

**Performance feedback loop:**
```
Ghost Writer → draft → Human edits → Engagement Collector captures delta
→ Learner observes → weekly: learned_adjustments
→ Scorer/Format Selector/Prompt Assembler read adjustments → better briefs
→ Ghost Writer produces closer drafts → fewer edits
```

**Market intelligence loop:**
```
Researcher (external signals) + Call Analyst (internal signals)
→ market_signals → Market Analyst (convergence)
→ market_intelligence → Scorer (context) + Content Suggester (opportunities)
→ content_items → Evidence Curator selects as evidence
```

**Engagement feedback loop:**
```
Published content → Engagement Collector captures metrics → content_performance
→ Response Analyst profiles segments → content_insights
→ Content Suggester reads trends → better suggestions
→ Contact Matcher reads velocity → better targeting
```

---

## 11. Ghost Writer Routing

The Orchestrator routes to the correct Ghost Writer based on the `channel` field:

| Channel | Ghost Writer | Voice |
|---|---|---|
| `[your-channel]` | 07 | Ghost Writer (single, adapts to format) |











All Ghost Writers load `references/ghost-writer-foundation.md` (shared rules) before writing. The Prompt Assembler writes briefs knowing which Ghost Writer will read them — permanent voice rules live in the Ghost Writer, per-piece adjustments come through the brief.

---

## 12. Trust & Autonomy System

The **Autonomy Manager (15)** manages three autonomy levels that govern how the Validator routes content:

### Level 1 — Full Review (Default)

All content goes to human review. Starting level and safety default.

### Level 2 — Standard

High-scoring content auto-approves. Upgrade requires: 50+ pieces at Level 1, edit rate below 30%, no trust incidents in 4 weeks, human approval.

### Level 3 — High Trust

Lower auto-approve threshold with 20% spot-checking. Upgrade requires: 100+ pieces at Level 2, auto-approved quality within 10% of human-reviewed, edit rate below 20%, no trust incidents in 8 weeks, 95%+ spot-check pass rate, human approval.

### Trust Incidents (Immediate Downgrade)

- Published content with factual error
- Published content with wrong voice/brand
- [YOUR COMPANY] claim that doesn't match Context Library
- Content that needed to be publicly retracted
- Any Layer 1 hard fail on auto-approved content

### Key Principles

- Upgrades always require human approval
- Downgrades can happen immediately (safety)
- Autonomy Manager unavailable → default to Level 1 (never default upward)
- Recovery requires the same criteria as the original upgrade

---

## 13. Scheduling & Cron

### Daily Sequence

| Time | Agent | Purpose |
|---|---|---|
| 6:00 AM | SYS-02 Signal Hygiene | Expire, decay, clean all intelligence tables |
| 7:00 AM | 04 Repetition Monitor (daily sweep) | Update anti-repetition state across all dimensions |
| 8:00 AM | 10 Researcher | Scan external world for new signals |
| 9:00 AM | 11 Content Suggester | Generate daily suggestions, update calendar |

### Weekly Sequence

| Day | Time | Agent |
|---|---|---|
| Wednesday | — | 18 Response Analyst (weekly segment analysis) |
| Saturday | — | 08 Learner (weekly pattern detection, adjustments, proposals) |
| Saturday | — | 16 Call Analyst (weekly pattern review) |
| Saturday | — | All Category A agents (weekly self-analysis) |
| Sunday | 8:00 AM | 15 Autonomy Manager (autonomy assessment) |
| Sunday | 10:00 AM | 13 Reporter (weekly intelligence report) |
| Monday | 11:00 AM | 05 Contact Matcher (weekly selection effectiveness review) |
| Day before send | — | SYS-04 Newsletter Curator (per newsletter) |

### Event-Driven Triggers

| Event | Agent Triggered |
|---|---|
| New content submission | SYS-01 Router |
| Human approves tentative item | SYS-06 Approval Queue → SYS-03 Orchestrator |
| Status change on `content_items` | SYS-03 Orchestrator |
| Path A review completed | 09 Engagement Collector (delta capture) → 08 Learner (observation) |
| [YOUR TRANSCRIPT PROVIDER] transcript available | 16 Call Analyst |
| Engagement data arrives | 09 Engagement Collector → 18 Response Analyst |
| Content approved | 05 Contact Matcher → 06 Distributor |
| Human asks question | 14 Advisor |
| Human override on trust level | 15 Autonomy Manager |

---

## 14. External Integrations

| System | Purpose | Used By |
|---|---|---|
| **PostgreSQL** | Primary data store (24 tables + 6 to create), shared state | All agents |
| **[YOUR SCHEDULING TOOL]** | Content scheduling and analytics | Engagement Collector (engagement capture) |
| **LinkedIn** | Publishing platform, manual analytics upload | Engagement Collector (weekly upload), Ghost Writers (channel) |
| **[YOUR TRANSCRIPT PROVIDER]** | Sales call transcripts and metadata | Call Analyst, Contact Matcher (fallback) |
| **[YOUR CRM]** | CRM — contacts, deals, companies | Contact Matcher (contact sourcing, pipeline checks) |
| **LLMs (Claude/ChatGPT)** | Ghost Writers, Validator Layer 2, Learner analysis | Pipeline agents, independent agents |

### Future Integrations (Documented)

- Direct LinkedIn API (replacing manual uploads)
- Newsletter platform webhooks (per-section click tracking)
- Blog/[YOUR PUBLICATION] analytics integration
- Real-time engagement webhooks

---

## 15. Data Flow Diagrams

### Content Lifecycle States

```
tentative → queued → scoring_complete → [parallel: shaping/curating/echo]
→ brief_assembled → prompt_assembled → draft_complete
→ approved | review_required | revision_required
→ approved → [Contact Matcher + Distributor] → published
```

Alternative terminal states: `dismissed`, `killed`, `expired`, `stale`

### Information Flow Between Agent Tiers

```
┌─────────────────────────────────────────────────────────────────────┐
│                     EXTERNAL WORLD                                  │
│  [Persona E]y bodies · Competitors · Industry press · Conferences     │
└──────────────┬──────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────┐     ┌──────────────────────┐
│  10 RESEARCHER       │     │  16 CALL ANALYST     │
│  (external signals)  │     │  (internal signals)  │
└──────────┬───────────┘     └──────────┬───────────┘
           │                            │
           ▼                            ▼
┌──────────────────────────────────────────────────────────────────┐
│                     market_signals                                │
│            (Researcher: external  |  Call Analyst: internal)      │
└──────────────────────────┬───────────────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
     ┌──────────────┐ ┌────────┐ ┌──────────────┐
     │ 03 CURATOR   │ │ 11     │ │ Market       │
     │ (evidence)   │ │SUGGEST.│ │ Analyst*     │
     └──────────────┘ └────────┘ └──────┬───────┘
                                        │
                                        ▼
                              ┌──────────────────┐
                              │ market_           │
                              │ intelligence      │
                              └────────┬─────────┘
                                       │
                              ┌────────┼────────┐
                              ▼        ▼        ▼
                         ┌────────┐ ┌──────┐ ┌──────────┐
                         │ 01     │ │ 11   │ │ 13       │
                         │SCORER  │ │SUGG. │ │REPORTER  │
                         └────────┘ └──────┘ └──────────┘

* Market Analyst not included in this skill suite
```

### Learning & Feedback Flow

```
┌─────────────┐    edit delta    ┌────────────┐
│ HUMAN       │ ───────────────► │ 09 LISTENER│
│ REVIEW      │                  └──────┬─────┘
└─────────────┘                         │
                                        ▼
                              ┌──────────────────┐
                              │ content_          │
                              │ performance       │
                              └────────┬─────────┘
                                       │
                    ┌──────────────────┼────────────────────┐
                    ▼                  ▼                    ▼
          ┌──────────────┐   ┌──────────────┐    ┌──────────────────┐
          │ 08 LEARNER   │   │ 18 RESPONSE  │    │ 13 REPORTER      │
          │              │   │ ANALYST      │    │                  │
          └──────┬───────┘   └──────┬───────┘    └──────────────────┘
                 │                  │
                 ▼                  ▼
     ┌────────────────────┐  ┌──────────────┐
     │ learned_adjustments│  │ content_     │
     │ (automatic)        │  │ insights     │
     │                    │  │ (segments)   │
     │ learning_staging_  │  └──────┬───────┘
     │ queue (human-gated)│         │
     └────────┬───────────┘         │
              │                     │
              ▼                     ▼
     ┌──────────────┐      ┌──────────────┐
     │ 01 SCORER    │      │ 11 SUGGESTER │
     │ 02 SHAPER    │      │ 05 PEOPLE    │
     │ SYS-05 PA    │      │    ENGINE    │
     └──────────────┘      └──────────────┘
```

---

## 16. Failure Handling Philosophy

Every agent follows consistent failure handling principles:

1. **Graceful degradation over hard failure.** If a non-critical data source fails, proceed with reduced confidence and note the gap. Critical failures (no content record, no scoring) block the pipeline.

2. **Never silently use bad data.** Stale data is flagged. Missing sources are noted. Gap flags travel downstream.

3. **Never default upward on safety.** Autonomy Manager unavailable → Level 1. Validator crash → human review. Never auto-approve on error.

4. **Buffer and retry for critical writes.** Database write failures for `content_items`, `content_performance`, and approval actions are treated as critical — alert, retry, never proceed without confirmed write.

5. **Timeouts are flags, not kills.** A timed-out agent gets flagged for investigation. The pipeline does not proceed incomplete.

6. **Publication is never blocked for non-critical failures.** Visual Producer failure, delta capture failure, and distribution write failure all allow the content to proceed. The gap is logged.

### Standard Timeouts

| Agent | Timeout |
|---|---|
| Parallel agents (Format Selector, Evidence Curator, Repetition Monitor) | 10 minutes |
| Prompt Assembler | 5 minutes |
| Ghost Writer | 15 minutes |
| Visual Producer (carousel) | 10 minutes |
| Visual Producer (graphic brief) | 5 minutes |

---

## 17. Database & Framework Reference

The system's shared state lives in a PostgreSQL database. This section documents every table with its structure, the framework reference files that feed into them, and the tables that still need creating.

### Existing Tables (24 tables, excluding temporary Cowork staging)

#### ICP & Positioning

**`icp_profiles`** — 10 rows. Populated. Source: `icp-framework.md`

| Column | Type | Notes |
|---|---|---|
| category | text | [Persona A], [Persona B], [Persona C], [Persona D], [Persona E] |
| role | text | Economic Buyer, Champion |
| pain_points | jsonb | Per-role pain points |
| use_cases | jsonb | Platform use cases per persona |
| language_resonance | jsonb | Vocabulary and framing that lands |

Read by: Scorer (01), Researcher (10), Call Analyst (16), Contact Matcher (05), Advisor (14) — 5 skills.

**`core_beliefs`** — 24 rows. Populated. Source: `F2-core-beliefs.md`

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| belief_name | text | Named conviction |
| belief_statement | text | The belief expressed |
| relevant_icps | text[] | Which ICPs connect |
| relevant_pillars | text[] | Which pillars connect |

Read by: Scorer (01), Researcher (10), Advisor (14) — 3 skills.

**`content_pillars`** — 0 rows. **Empty — needs populating.** Source: `content-pillars.md`

| Column | Type | Notes |
|---|---|---|
| pillar_slug | text | e.g. `industry-expertise`, `thought-leadership` |
| definition | text | What the pillar covers |
| sub_topics | text | Topics within scope |
| adjacent_topics | text | Related but borderline topics |
| out_of_scope | text | What doesn't belong |
| target_percentage | numeric | Target distribution share |
| current_percentage | numeric | Actual distribution share |

A single table. All agents query the same table.

Read by: Scorer (01), Format Selector (02), Repetition Monitor (04), Researcher (10), Content Suggester (11), Reporter (13), Advisor (14), Newsletter Curator (SYS-04) — **8 skills, the most-referenced framework.**

#### Channel & Format Intelligence

**`channel_intelligence`** — 3 rows. Partially populated. Source: `F6-distribution-channels.md`
Read by: Format Selector (02), Distributor (06).

**`algorithm_knowledge` (see frameworks/platform-algorithm.md)** — 0 rows. **Empty — needs populating.** Sources: `F4a-platform-algorithm.md`, `F4b-platform-algorithm.md`
Read by: Format Selector (02), Prompt Assembler (SYS-05).

**`format_constraints`** — 5 rows. Partially populated. Source: `F5a-content-types-2.md`
Read by: Format Selector (02), Repetition Monitor (04), Validator (12) — 3 skills.

**`format_affinity`** — 16 rows. Populated. Source: `F5b-pillar-subcategories.md`
Read by: Format Selector (02).

#### Content Pipeline

**`content_items`** — 43 rows. Active. Central pipeline record.

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| origin | text | `brief`, `source_generate`, `received`, `newsletter_curation` |
| input_type | text | `standard`, `hero`, `hero_child` |
| raw_input | text | Submitted content/brief/source |
| source_material | text, nullable | If origin = source_generate |
| parent_id | uuid, nullable | Link to hero parent |
| campaign_id | uuid, nullable | Link to campaign |
| suggestion_id | uuid, nullable | Link to Content Suggester recommendation |
| channel | text, nullable | Target channel |
| format | text, nullable | Target format |
| status | text | Pipeline status: tentative → queued → scoring_complete → brief_assembled → prompt_assembled → draft_complete → approved/review_required/revision_required → published |
| calendar_status | text | Human-facing: tentative, approved_topic, drafted, approved, completed, dismissed, killed |
| origin_hint | text | Source context |
| suggested_date | date, nullable | Target date at intake |
| scheduled_date | date, nullable | Confirmed date at approval |
| published_date | date, nullable | Actual publication date |
| published_url | text, nullable | Live URL — set by Engagement Collector on [YOUR SCHEDULING TOOL] publish webhook |
| external_post_id | text, nullable | [YOUR SCHEDULING TOOL]/platform post ID — set by Engagement Collector on publish |
| submitted_by | text | |
| submitted_at | timestamp | |
| approval_by | text, nullable | |
| approval_at | timestamp, nullable | |
| created_at | timestamp | |
| scoring_result | jsonb | Scorer: ICP, beliefs, pillar, framing direction |
| format_recommendation | jsonb | Format Selector: format, channel, structure |
| delivery_strategy | jsonb, nullable | Format Selector: multi-channel plan (hero items only) |
| evidence_package | jsonb | Evidence Curator: evidence across 6 layers |
| echo_warnings | jsonb | Repetition Monitor: 5-dimension repetition flags |
| assembled_prompt | text | Prompt Assembler: prose brief for Ghost Writer |
| assembled_prompt_metadata | jsonb | Prompt Assembler: audit trail |
| curation_plan | jsonb, nullable | Newsletter Curator: edition plan |
| draft | text | Ghost Writer output |
| visual_assets | jsonb, nullable | Visual Producer output |
| validation_result | jsonb | Validator: Layer 1 + Layer 2 scores and routing |
| revision_count | integer | Total revision loops |
| revision_notes_human | text, nullable | Path B human notes |
| dismissal_reason | text, nullable | |
| kill_reason | text, nullable | |
| queue_notes | text, nullable | |

Written by: Router, Orchestrator, Scorer, Format Selector, Evidence Curator, Repetition Monitor, Prompt Assembler, Ghost Writers, Validator, Visual Producer, Approval Queue. Read by: All agents.

**`content_performance`** — 1 row. Near-empty.

| Column | Type | Notes |
|---|---|---|
| content_id | uuid | Links to content_items.id |
| channel | text | |
| period | text | `24h`, `48h`, `7d` |
| source | text | `[your-scheduling-tool]`, `linkedin_upload` |
| captured_at | timestamp | |
| impressions | integer | |
| clicks | integer | |
| reactions | jsonb | `{like, celebrate, insightful, love}` |
| comments_count | integer | |
| shares | integer | |
| saves | integer | |
| newsletter_metrics | jsonb, nullable | `{recipients, opens, open_rate, clicks, unsubscribes}` |
| edit_delta | jsonb, nullable | Original draft vs published (Path A reviews) |
| distribution_effectiveness | jsonb, nullable | Per-stage engagement |
| engagement_quality_breakdown | jsonb, nullable | Deep/standard/passive |

Written by: Engagement Collector (09), Response Analyst (18). Read by: Learner (08), Reporter (13), Advisor (14), Content Suggester (11), Autonomy Manager (15).

**`content_insights`** — 0 rows. Shared intelligence brain. Agent-populated over time.
Written by: Repetition Monitor (04), Evidence Curator (03), Response Analyst (18), Learner (08). Read by: Scorer, Format Selector, Content Suggester, Reporter, Advisor, Contact Matcher, Approval Queue.

**`content_campaigns`** — 0 rows. Written by: Content Suggester (11). Read by: Router, Orchestrator, Signal Hygiene.

**`content_calendar`** — 0 rows. Written by: Content Suggester (11). Read by: Content Suggester, Signal Hygiene.

**`daily_content_suggestions`** — 0 rows. Written by: Content Suggester (11). Read by: Approval Queue, Advisor, Signal Hygiene.

**`distribution_engagement_briefs`** — 0 rows. Written by: Distributor (06). Read by: Response Analyst (18), Reporter (13).

#### Market & Sales Intelligence

**`market_signals`** — 76 rows. **Most active table.**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| collected_at | timestamp | |
| source_type | text | `external` (Researcher) or `internal` (Call Analyst) |
| signal_type | text | `regulatory`, `competitive`, `industry_trend`, `technology`, `event`, `partnership`, `market_shift`, `pain_point`, `objection` |
| summary | text | 1-2 sentence interpretation |
| detail | text | Fuller context |
| source_url | text, nullable | |
| source_name | text | |
| relevant_icps | text[] | |
| relevant_pillars | text[] | |
| relevant_beliefs | text[] | |
| urgency | text | `time_sensitive`, `current`, `developing`, `evergreen`, `background` |
| confidence | text | `high`, `medium`, `low` |
| nda_flag | boolean | |
| nda_review_status | text, nullable | `pending_team_review`, `approved`, `rejected` |
| status | text | `active`, `aging`, `archived`, `expired` |

Written by: Researcher (10), Call Analyst (16). Read by: Scorer, Evidence Curator, Content Suggester, Newsletter Curator. Lifecycle: Signal Hygiene (SYS-02).

**`market_intelligence`** — 0 rows. Convergence analysis. Written by Market Analyst (not in this suite). Read by: Scorer, Content Suggester, Newsletter Curator, Reporter, Advisor.

**`sales_call_signals`** — 56 rows. Active. Schema defined in `[your-company]-signal-spotting-framework.md`.

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| created_at | timestamp | |
| meeting_uuid | text | [YOUR TRANSCRIPT PROVIDER] meeting UUID |
| meeting_date | timestamp | |
| meeting_type | text | Discovery, Demo, Technical Deep-Dive, etc. |
| signal_type | text | Extended taxonomy (includes `buying_signal`, `risk_signal`, `champion_signal`, `stakeholder_signal`, `urgency_signal`, `requirement`, `sentiment`, `expansion`, `churn_risk`) |
| signal_summary | text | Agent's interpretation |
| signal_detail | text | Context |
| attributed_to | text | Email |
| attributed_to_name | text | |
| attributed_to_role | text | Champion, Economic_Buyer, etc. |
| deal_id | text, nullable | [YOUR CRM] deal ID |
| account_id | text, nullable | [YOUR CRM] company ID |
| contact_ids | text[], nullable | |
| relevant_icps | text[] | |
| meddicc_element | text, nullable | |
| pipeline_stage_evidence | text, nullable | |
| urgency | text | |
| confidence | text | |
| pain_domain | text, nullable | |
| severity | text, nullable | |
| [your-company]_team_present | text[] | |
| status | text | active, archived, promoted |
| promoted_to | text, nullable | market_signals ID |

Written by: Call Analyst (16).

#### Context & Evidence

**`context_library`** — 4 rows. Near-empty. Evidence Curator's primary evidence pool. Needs significant population.

**`[your-company]_operational_data`** — 0 rows. Company metrics. Evidence Curator flags >30 day staleness.

#### Learning & Trust

**`learned_adjustments`** — 1 row. Runtime intelligence from Learner.

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| skill_id | text | Target agent |
| module | text | Target module |
| condition_json | jsonb | When applies (ICP x pillar x format) |
| instruction_delta | text | The adjustment |
| confidence | numeric | 0.0–1.0, decays over time |
| observation_count | integer | |
| last_reinforced_at | timestamp | |
| decay_status | text | active, archived, superseded |

Written by: Learner (08). Read by: Scorer, Format Selector, Prompt Assembler, Content Suggester. Lifecycle: Signal Hygiene (SYS-02).

**`learning_staging_queue`** — 0 rows. Human-gated proposals. Written by: Learner (08), Advisor (14).

**`trust_tracker_state`** — 0 rows. Validator defaults to Level 1 when empty. Written by: Autonomy Manager (15). Read by: Validator (12).

#### Agent & Pipeline Ops

**`agent_scratchpads`** — 0 rows. Single table for all agent decision logs.

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| agent_id | text | Discriminator — which agent |
| run_id | uuid | |
| run_type | text | `per_item`, `daily`, `weekly_analysis` |
| timestamp | timestamp | |
| input_context | jsonb | |
| decisions_made | jsonb | |
| outcomes_observed | jsonb, nullable | |
| patterns_detected | jsonb, nullable | |
| learnings_shared | jsonb, nullable | |
| confidence_metadata | jsonb, nullable | |

Skills reference per-agent shorthand names (`scratchpad_researcher`, `scratchpad_learner`, etc.). The database uses this single `agent_scratchpads` table — skills query with `WHERE agent_id = '[agent]'`.

Written by: All Category A agents. Read by: Reporter (13), Advisor (14), each agent's own history.

**`lifecycle_logs`** — 0 rows. Daily system run summaries. Written by: Signal Hygiene (SYS-02).

**`pipeline_metrics`** — 0 rows. Throughput, bottlenecks, revision rates. Available to Reporter.

### Framework Reference Files

The repo root contains 20 framework documents. These are the authoritative source content for database tables and agent reference material.

| File | Maps To | Status |
|---|---|---|
| `icp-framework.md` | `icp_profiles` | Needs populating |
| `core-beliefs.md` | `core_beliefs` | Needs populating |
| `content-pillars.md` | `content_pillars` | Needs populating |
| `platform-algorithm.md` | `algorithm_knowledge` | Needs populating |
| `content-types.md` | `format_constraints` | Needs populating |
| `pillar-subcategories.md` | `format_affinity` | Needs populating |
| `distribution-channels.md` | `channel_intelligence` | Needs populating |
| `industry-landscape.md` | `industry_map` | Needs populating |
| `thought-leader-authority-model.md` | Needs table: `authority_credentials` | **Table doesn't exist** |
| `[your-company]_comms_profiles.md` | Needs table: `icp_engagement_approaches` | **Table doesn't exist** |
| `[YOUR COMPANY]_Pipeline_Framework.md` | Needs table: `ecosystem_pipelines` | **Table doesn't exist** |
| `[your-company]-[your-crm]-metadata-framework.md` | Needs table: `[your-crm]_metadata` | **Table doesn't exist** |
| `[your-company]-signal-spotting-framework.md` | `sales_call_signals` (schema doc) | Yes (56 rows) |
| `[your-company]-[your-transcript-provider]-metadata-framework.md` | [YOUR TRANSCRIPT PROVIDER] API reference (not a table) | N/A |
| `[your-company]-brand-framework.md` | Brand voice reference (not a table) | N/A |
| `[YOUR COMPANY]_Terminology_Framework-2.md` | Terminology reference (not a table) | N/A |

### Ghost Writer Reference Files (in `references/`)

| File | Loaded By |
|---|---|
| `references/ghost-writer-foundation.md` | Ghost Writer (07) |
| `references/formats/linkedin.md` | Ghost Writer (07) |
| `references/voice-profile.template.md` | Ghost Writer (07) |

---

## 18. Missing Tables

These tables are referenced by skills but do not exist in the database. Framework files with source content exist in the repo for all except one.

| Missing Table | Skills That Need It | Source Framework File | Priority |
|---|---|---|---|
| `context_library` | Evidence Curator (03), Validator (12), Contact Matcher (05), Newsletter Curator (SYS-04) | **No file exists — needs authoring from scratch** | Critical — 4 skills blocked |
| `ecosystem_pipelines` | Contact Matcher (05), Call Analyst (16) | Pipeline framework — ready | High — 2 skills blocked |
| `icp_engagement_approaches` | Contact Matcher (05) | `comms-profiles.md` — ready | High |
| `industry_map` | Researcher (10) | `industry-landscape.md` — ready | High |
| `authority_credentials` | Validator (12) | Authority model — ready | Medium |
| `[your-crm]_metadata` | Contact Matcher (05) | CRM metadata framework — ready | Medium |

`context_library` is the only table that needs both schema creation and content authoring. It should contain: [YOUR COMPANY] platform capabilities, product information, team members (for internal exclusion filtering), and company stats. The Evidence Curator selects from it for evidence, the Validator verifies draft claims against it, the Contact Matcher filters internal team members using it, and the Newsletter Curator uses it for [YOUR COMPANY]-relevant framing.

---

## 19. What's Not in This Suite

The following are referenced by the skill files but not included in this package:

- **Market Analyst** — Friday convergence analysis synthesising internal + external signals into `market_intelligence`. Referenced by Scorer, Content Suggester, Newsletter Curator, Reporter.
- **Prompt evolution / skill module system** — `claude_skill_modules`, `prompt_evolution_log`. Referenced by Learner Speed 2.
- **File-based memory** — `memory/learning/observations/` and `memory/pipeline-traces/`. Referenced by Learner and Evidence Curator.
- **Frontend specifications** — Calendar/Queue UI, human review space, Advisor chat interface.
- **`references/design-system.md`** — carousel brand CSS and slide patterns. Referenced by Visual Producer (SYS-07) but not in the `references/` directory.

---

*This document describes the architecture as specified in the GTM Content Engine skill suite (24 skills + 3 references). The system is designed as a set of AI agent instructions, not as executable code. Implementation details (database DDL, frontend code, cron runners, API integrations) exist outside this package.*
