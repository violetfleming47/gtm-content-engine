# GTM Content Engine

A 24-agent system that automates the full Go-To-Market content pipeline — from market signal scanning and ICP scoring, through AI-assisted drafting across multiple channels, to validation, human review, and distribution. The system learns from every human edit and gets better over time.

The agents are defined as portable markdown skill files that work with any AI platform — Claude, ChatGPT, Cursor, Windsurf, or any environment that can run agent instructions. Backed by PostgreSQL (Supabase free tier works), GTM Content Engine turns your GTM content operation into a structured, learning pipeline.

---

## What it does

24 agents work across a structured pipeline:

```
Market Signals ──→ Router ──→ Scorer ──→ Format Selector + Evidence Curator + Repetition Monitor
                                              │
                                    Orchestrator ──→ Prompt Assembler
                                              │
                                      Ghost Writer ──→ Validator
                                              │
                                      Human Review ──→ Contact Matcher ──→ Distributor
                                              │
                              Performance Data ──→ Learner ──→ System Improves
```

**Pipeline agents** process content sequentially through scoring, shaping, evidence curation, drafting, and validation.

**Independent agents** run on schedules — scanning external signals daily, analysing engagement patterns, profiling audience segments, and generating a weekly intelligence report.

**System skills** handle infrastructure: routing, orchestration, signal hygiene, newsletter curation, and visual production.

**The learning loop** is the core value. Every human edit to a draft is captured, observed by the Learner, and — over time — converted into runtime adjustments that make future drafts closer to what you'd write yourself. At 50+ observations, the system starts compounding.

---

## What's in the repo

```
gtm-content-engine/
├── README.md
├── QUICKSTART.md                      ← First scored content in 15 minutes
├── QUESTIONS.md                       ← Start here. Answer these for your org.
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
│
├── skills/                            ← 24 AI agent skill definitions
│   ├── pipeline/                      Content processing agents
│   │   ├── scorer/                 ICP scoring, belief alignment, pillar matching
│   │   ├── format-selector/        Format, channel, structure selection
│   │   ├── evidence-curator/       Evidence curation across 6 layers
│   │   ├── repetition-monitor/     5-dimension repetition monitoring
│   │   ├── ghost-writer/           One writer, all formats, your voice
│   │   └── validator/              2-layer quality gate
│   │
│   ├── system/                        Infrastructure coordination
│   │   ├── router/                 Intake and classification
│   │   ├── signal-hygiene/         Daily data hygiene
│   │   ├── orchestrator/           Pipeline coordination
│   │   ├── newsletter-curator/     Newsletter edition curation
│   │   ├── prompt-assembler/       JSONB → prose brief translation
│   │   ├── approval-queue/         Approval gate + calendar
│   │   └── visual-producer/        Carousel + graphic briefs
│   │
│   ├── independent/                   Cron + event-driven agents
│   │   ├── contact-matcher/        Contact matching for distribution
│   │   ├── distributor/            3-stage distribution briefs
│   │   ├── learner/                Edit observation + pattern detection
│   │   ├── engagement-collector/   Engagement data capture
│   │   ├── researcher/             External signal scanning
│   │   ├── content-suggester/      Content suggestions + campaigns
│   │   ├── reporter/               Weekly intelligence report
│   │   ├── advisor/                Human chat interface to system
│   │   ├── autonomy-manager/       Autonomy level management
│   │   ├── response-analyst/       Engagement analysis + segments
│   │   └── call-analyst/           Customer signal extraction (optional)
│   │
│   └── standalone/
│       └── voc-scanner/              Voice of Customer mining (optional)
│
├── frameworks/                        ← 11 configurable reasoning templates
│   ├── icp-framework.md              Who your buyers are
│   ├── core-beliefs.md               What your company believes
│   ├── content-pillars.md            Your content themes
│   ├── content-types.md              Format definitions
│   ├── pillar-subcategories.md       How pillars express in practice
│   ├── distribution-channels.md      Where content is published
│   ├── platform-algorithm.md          Platform knowledge (expandable)
│   ├── industry-landscape.md         Your sectors and market
│   ├── brand-framework.md            Your visual identity
│   ├── comms-profiles.md             Per-persona communication
│   └── customer-signals.md           Customer data signals (optional)
│
├── references/                        ← Voice profiles and format specs
│   ├── ghost-writer-foundation.md    Shared writing rules
│   ├── voice-profile.template.md     Your brand voice
│   └── formats/                      Per-format structural rules
│
├── architecture/
│   └── schema.md                     ← Database schema (20+ tables)
│
├── guides/
│   ├── setup.md                      Step-by-step installation
│   ├── customisation.md              Adapting for your company
│   ├── customer-data-setup.md        If you have transcripts/CRM data
│   ├── voc-setup.md                  If you don't (mine public language)
│   └── deployment.md                 Platform-specific deployment
│
├── templates/                         ← For extending the system
│   ├── SKILL.template.md
│   ├── framework.template.md
│   └── voice-profile.template.md
│
└── docs/
    ├── ARCHITECTURE.md               Full system design
    ├── AGENTS.md                     Agent reference guide
    └── LEARNING-SYSTEM.md            How the learning cycle works
```

---

## How the pieces fit together

**Skills** are the agents — each one is a complete system prompt that defines the agent's boundary, what it reads, what it writes, how it loads frameworks, and how it degrades gracefully when data is sparse.

**Frameworks** are the reasoning engine. Each framework teaches agents *what* something is and *how to read it from your data*. The "what" comes from your answers in QUESTIONS.md — your ICP personas, your beliefs, your content pillars. Every framework has `[YOUR ...]` markers showing exactly where your answers go.

**The schema** defines the database tables — content pipeline, signal collection, learning loop, traceability. Every write is traced. Every version is preserved. The learning tables (`learned_adjustments`, `agent_scratchpads`, `content_performance`, `content_insights`) are what make the system compound over time.

**References** define how the Ghost Writer sounds. One voice profile, applied consistently across all formats. Format specs in `references/formats/` define structural rules per channel.

---

## Deployment

**The skills are harness-agnostic.** Each skill is a system prompt — structured markdown that defines what the agent does, what it refuses to do, what context to load, and how to verify its work. You can deploy them in:

- **Cursor / Windsurf / Cline** — as agent skills in any IDE harness
- **Claude Code** — as project context
- **Claude Projects / ChatGPT** — as project instructions or custom GPTs
- **Anthropic Managed Agents** — as managed agents with MCP connections
- **LangChain / LangGraph / CrewAI** — as system prompts in your orchestrator
- **Custom orchestration** — any system that can send a system prompt to an LLM and connect to your data sources

The `[YOUR ...]` values in each skill tell you what to wire up.

---

## Architecture

The system is composed of three agent categories:

| Category | Agents | Behaviour |
|----------|--------|-----------|
| **Category A — Autonomous Learner** | Evidence Curator, Contact Matcher, Distributor, Learner, Researcher, Content Suggester, Call Analyst, Response Analyst | Has a scratchpad, runs weekly self-analysis, shares learnings |
| **Category B — Informed Executor** | Scorer, Format Selector, Repetition Monitor, Ghost Writer, Validator, Engagement Collector, Reporter, Advisor, Autonomy Manager | Reads shared intelligence, improves through richer data |
| **System** | Router, Signal Hygiene, Orchestrator, Newsletter Curator, Prompt Assembler, Approval Queue, Visual Producer | Infrastructure coordination, no learning loop |

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full system design, data flows, and agent interactions.

---

## The learning loop

The learning system operates at three speeds:

**Speed 1 — Observe (immediate):** Every human edit to a draft is captured. The Learner reads both versions and writes an observation.

**Speed 2a — Adjust (weekly, automatic):** After 5+ similar observations, the Learner creates a `learned_adjustment` — soft guidance with a confidence score. Pipeline agents treat these as weighted signals.

**Speed 2b — Evolve (weekly, human-gated):** After 10+ instances of the same pattern over 4+ weeks, the Learner proposes a permanent skill change. Sits pending until human approval.

Adjustments that aren't reinforced decay over time. The system forgets what stops being true.

See [docs/LEARNING-SYSTEM.md](docs/LEARNING-SYSTEM.md) for the full learning architecture.

---

## Customer data: two paths

**If you have customer call transcripts or CRM data:**
Use the **Call Analyst** skill (`skills/independent/call-analyst/`). It extracts signals from transcripts — objections, buying signals, pain points — and feeds them into `market_signals` with NDA guardrails. Setup: [guides/customer-data-setup.md](guides/customer-data-setup.md).

**If you don't:**
Use the **VOC Scanner** skill (`skills/standalone/voc-scanner/`). It mines the public web for how your ICP talks about their problems — the exact words, phrases, and framing practitioners use. This becomes your messaging fuel. Setup: [guides/voc-setup.md](guides/voc-setup.md).

Both paths feed the same downstream pipeline. The Researcher handles external market signals regardless of which path you choose.

---

## Quick start

Get from clone to first scored content in 15 minutes. See [QUICKSTART.md](QUICKSTART.md).

---

## Getting started

### 1. Answer the questions

Start with [`QUESTIONS.md`](QUESTIONS.md). It walks through what the system needs to know about your business — your ICP, beliefs, content pillars, voice, brand, industry, and channels.

### 2. Populate the frameworks

Your answers fill the `[YOUR ...]` markers in the 11 frameworks. You can do this manually or with an agent — give it your answers and the framework templates.

### 3. Set up the database

Follow [`guides/setup.md`](guides/setup.md) to create your database and install the schema from [`architecture/schema.md`](architecture/schema.md). Load your populated frameworks into the corresponding tables.

### 4. Configure your voice

Fill in `references/voice-profile.template.md` with your voice rules, word bans, signature devices, and core arguments. This is what the Ghost Writer loads before writing every piece.

### 5. Deploy and run

Load each skill's `SKILL.md` as the system prompt for an agent in your harness. See [`guides/deployment.md`](guides/deployment.md) for platform-specific instructions.

Run the Researcher daily, submit content through the Router, and let the pipeline process it. The learning loop starts working from your first human edit.

---

## Documentation

- [Architecture](docs/ARCHITECTURE.md) — Full system design, data flows, agent interactions
- [Agent Reference](docs/AGENTS.md) — Detailed guide to each of the 24 agents
- [Learning System](docs/LEARNING-SYSTEM.md) — How the feedback loop works
- [Setup Guide](guides/setup.md) — Step-by-step installation
- [Customisation](guides/customisation.md) — Adapting for your company
- [Deployment](guides/deployment.md) — Platform-specific deployment guides

---

## Patterns you can reuse in any domain

| Pattern | What it solves |
|---------|---------------|
| **Frameworks as reasoning guides** | Agents that access data but can't interpret it the way your people do |
| **Three-layer scope control** | Agent drift — frameworks fill space, classifications contain, prohibitions close gaps |
| **SCD-style versioning on deal/content state** | Knowing what changed, when, and why — not just current state |
| **Freshness decay** | Stale data silently informing live decisions |
| **Collect/classify boundary** | One agent both capturing and interpreting, compounding errors |
| **Edit delta as learning signal** | The gap between AI draft and human-published version is the richest feedback |
| **Confidence decay on learned adjustments** | Unreinforced patterns automatically fade instead of accumulating forever |
| **Trust levels for autonomy** | Graduated human oversight that loosens as the system proves itself |

---

## Contributing

PRs welcome — especially framework implementations for domains beyond B2B, alternative CRM integrations, and harness-specific deployment guides.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.
