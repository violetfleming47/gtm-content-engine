# Database Schema

The system stores all shared state in a PostgreSQL database (Supabase free tier works). Every table is documented below with its columns, which agents write to it, and which agents read from it.

The schema has four layers: **Framework Tables** (your configuration), **Pipeline Tables** (content flowing through the system), **Intelligence Tables** (market signals and customer data), and **Learning Tables** (the feedback loop that makes the system improve).

---

## Framework Tables

These tables hold your populated frameworks. Agents read them at runtime.

### `icp_profiles`

Your ICP personas. Source: `frameworks/icp-framework.md`

```sql
CREATE TABLE icp_profiles (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    category text NOT NULL,
    role text NOT NULL,
    pain_points jsonb,
    use_cases jsonb,
    language_resonance jsonb,
    titles jsonb,
    created_at timestamp DEFAULT now()
);
```

Read by: Scorer, Researcher, Call Analyst, Contact Matcher, Advisor

### `core_beliefs`

Your company's intellectual positions. Source: `frameworks/core-beliefs.md`

```sql
CREATE TABLE core_beliefs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    belief_name text NOT NULL,
    belief_slug text UNIQUE NOT NULL,
    belief_statement text NOT NULL,
    counter_positions text,
    relevant_icps text[],
    relevant_pillars text[],
    created_at timestamp DEFAULT now()
);
```

Read by: Scorer, Researcher, Advisor

### `content_pillars`

Your content themes. Source: `frameworks/content-pillars.md`

```sql
CREATE TABLE content_pillars (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    pillar_slug text UNIQUE NOT NULL,
    definition text NOT NULL,
    sub_topics text,
    adjacent_topics text,
    out_of_scope text,
    target_percentage numeric,
    current_percentage numeric DEFAULT 0,
    created_at timestamp DEFAULT now()
);
```

Read by: Scorer, Format Selector, Repetition Monitor, Researcher, Content Suggester, Reporter, Advisor, Newsletter Curator (8 agents — the most-referenced table)

### `channel_intelligence`

Your distribution channels. Source: `frameworks/distribution-channels.md`

```sql
CREATE TABLE channel_intelligence (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    channel_slug text UNIQUE NOT NULL,
    channel_name text NOT NULL,
    description text,
    content_types text[],
    channel_notes text,
    created_at timestamp DEFAULT now()
);
```

Read by: Format Selector, Distributor

### `algorithm_knowledge` (see frameworks/platform-algorithm.md)

Platform algorithm rules (e.g. LinkedIn). Source: `frameworks/platform-algorithm.md`, `frameworks/platform-algorithm.md`

```sql
CREATE TABLE algorithm_knowledge (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    platform text NOT NULL,
    rule_type text NOT NULL,
    rule_content text NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Read by: Format Selector, Prompt Assembler

### `format_constraints`

Content format definitions. Source: `frameworks/content-types.md`

```sql
CREATE TABLE format_constraints (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    format_name text NOT NULL,
    description text,
    word_count_min integer,
    word_count_max integer,
    structure_rules jsonb,
    platform_notes jsonb,
    created_at timestamp DEFAULT now()
);
```

Read by: Format Selector, Repetition Monitor, Validator

### `format_affinity`

How pillars map to formats. Source: `frameworks/pillar-subcategories.md`

```sql
CREATE TABLE format_affinity (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    subcategory text NOT NULL,
    description text,
    typical_pillars text[],
    typical_formats text[],
    created_at timestamp DEFAULT now()
);
```

Read by: Format Selector

### `industry_map`

Your market landscape. Source: `frameworks/industry-landscape.md`

```sql
CREATE TABLE industry_map (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    sector text NOT NULL,
    description text,
    key_players text[],
    signal_categories text[],
    created_at timestamp DEFAULT now()
);
```

Read by: Researcher

### `icp_engagement_approaches`

Per-persona communication profiles. Source: `frameworks/comms-profiles.md`

```sql
CREATE TABLE icp_engagement_approaches (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    persona_category text NOT NULL,
    persona_role text NOT NULL,
    language_patterns text,
    content_preferences text,
    what_resonates text,
    what_repels text,
    authority_signals text,
    typical_messaging text,
    preferred_tone text,
    brand_relevance_framing text,
    created_at timestamp DEFAULT now()
);
```

Read by: Contact Matcher

---

## Pipeline Tables

These tables track content as it flows through the system.

### `content_items`

The central record. Every piece of content has one row that accumulates data through the pipeline.

```sql
CREATE TABLE content_items (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    origin text NOT NULL,
    input_type text DEFAULT 'standard',
    raw_input text NOT NULL,
    source_material text,
    parent_id uuid REFERENCES content_items(id),
    campaign_id uuid,
    suggestion_id uuid,
    channel text,
    format text,
    status text DEFAULT 'tentative',
    calendar_status text DEFAULT 'tentative',
    origin_hint text,
    suggested_date date,
    scheduled_date date,
    published_date date,
    published_url text,
    external_post_id text,
    submitted_by text NOT NULL,
    submitted_at timestamp DEFAULT now(),
    approval_by text,
    approval_at timestamp,
    created_at timestamp DEFAULT now(),

    -- Pipeline agent outputs (JSONB)
    scoring_result jsonb,
    format_recommendation jsonb,
    delivery_strategy jsonb,
    evidence_package jsonb,
    echo_warnings jsonb,
    assembled_prompt text,
    assembled_prompt_metadata jsonb,
    curation_plan jsonb,
    draft text,
    visual_assets jsonb,
    validation_result jsonb,

    -- Revision tracking
    revision_count integer DEFAULT 0,
    revision_notes_human text,
    dismissal_reason text,
    kill_reason text,
    queue_notes text
);
```

Written by: Router, Orchestrator, Scorer, Format Selector, Evidence Curator, Repetition Monitor, Prompt Assembler, Ghost Writer, Validator, Visual Producer, Approval Queue. Read by: All agents.

### `content_performance`

Engagement metrics and edit deltas. The learning system's primary input.

```sql
CREATE TABLE content_performance (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id uuid REFERENCES content_items(id),
    channel text,
    period text,
    source text,
    captured_at timestamp DEFAULT now(),
    impressions integer,
    clicks integer,
    reactions jsonb,
    comments_count integer,
    shares integer,
    saves integer,
    newsletter_metrics jsonb,
    edit_delta jsonb,
    distribution_effectiveness jsonb,
    engagement_quality_breakdown jsonb
);
```

Written by: Engagement Collector, Response Analyst. Read by: Learner, Reporter, Advisor, Content Suggester, Autonomy Manager.

### `content_insights`

Shared intelligence brain. Agent-populated over time.

```sql
CREATE TABLE content_insights (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    insight_type text NOT NULL,
    insight_content jsonb NOT NULL,
    source_agent text NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Written by: Repetition Monitor, Evidence Curator, Response Analyst, Learner. Read by: Scorer, Format Selector, Content Suggester, Reporter, Advisor, Contact Matcher, Approval Queue.

### `content_campaigns`

Multi-piece campaign plans.

```sql
CREATE TABLE content_campaigns (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    campaign_name text NOT NULL,
    campaign_plan jsonb NOT NULL,
    status text DEFAULT 'planned',
    created_at timestamp DEFAULT now()
);
```

Written by: Content Suggester. Read by: Router, Orchestrator, Signal Hygiene.

### `content_calendar`

Rolling content calendar view.

```sql
CREATE TABLE content_calendar (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    date date NOT NULL,
    channel text,
    planned_content jsonb,
    created_at timestamp DEFAULT now()
);
```

Written by: Content Suggester. Read by: Content Suggester, Signal Hygiene.

### `daily_content_suggestions`

Daily content ideas from the Content Suggester.

```sql
CREATE TABLE daily_content_suggestions (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    suggestion_date date DEFAULT CURRENT_DATE,
    suggestions jsonb NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Written by: Content Suggester. Read by: Approval Queue, Advisor, Signal Hygiene.

### `distribution_engagement_briefs`

Pre-pub, at-launch, and post-pub distribution plans.

```sql
CREATE TABLE distribution_engagement_briefs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    content_id uuid REFERENCES content_items(id),
    stage text NOT NULL,
    brief jsonb NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Written by: Distributor. Read by: Response Analyst, Reporter.

---

## Intelligence Tables

Market signals and customer data.

### `market_signals`

External and internal signals. The system's awareness of the outside world.

```sql
CREATE TABLE market_signals (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    collected_at timestamp DEFAULT now(),
    source_type text NOT NULL,
    signal_type text NOT NULL,
    summary text NOT NULL,
    detail text,
    source_url text,
    source_name text NOT NULL,
    relevant_icps text[],
    relevant_pillars text[],
    relevant_beliefs text[],
    urgency text NOT NULL,
    confidence text NOT NULL,
    nda_flag boolean DEFAULT false,
    nda_review_status text,
    status text DEFAULT 'active'
);
```

Written by: Researcher (external), Call Analyst (internal). Read by: Scorer, Evidence Curator, Content Suggester, Newsletter Curator. Lifecycle: Signal Hygiene.

### `market_intelligence`

Convergence analysis — patterns across multiple signals.

```sql
CREATE TABLE market_intelligence (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    theme text NOT NULL,
    analysis jsonb NOT NULL,
    source_signals uuid[],
    created_at timestamp DEFAULT now()
);
```

Read by: Scorer, Content Suggester, Newsletter Curator, Reporter, Advisor.

### `sales_call_signals`

Signals extracted from customer call transcripts. Optional — only if you have transcript data.

```sql
CREATE TABLE sales_call_signals (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at timestamp DEFAULT now(),
    meeting_uuid text,
    meeting_date timestamp,
    meeting_type text,
    signal_type text NOT NULL,
    signal_summary text NOT NULL,
    signal_detail text,
    attributed_to text,
    attributed_to_name text,
    attributed_to_role text,
    deal_id text,
    account_id text,
    contact_ids text[],
    relevant_icps text[],
    urgency text,
    confidence text,
    pain_domain text,
    severity text,
    team_present text[],
    status text DEFAULT 'active',
    promoted_to text
);
```

Written by: Call Analyst.

### `context_library`

Your evidence pool — product capabilities, case studies, proof points, company stats.

```sql
CREATE TABLE context_library (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    category text NOT NULL,
    title text NOT NULL,
    content text NOT NULL,
    metadata jsonb,
    usage_count integer DEFAULT 0,
    last_used_at timestamp,
    created_at timestamp DEFAULT now()
);
```

Written by: Manual population. Read by: Evidence Curator, Validator, Contact Matcher, Newsletter Curator.

### `operational_data`

Company metrics and stats. Evidence Curator flags anything older than 30 days as stale.

```sql
CREATE TABLE operational_data (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    metric_name text NOT NULL,
    metric_value text NOT NULL,
    as_of_date date NOT NULL,
    metadata jsonb,
    created_at timestamp DEFAULT now()
);
```

Read by: Evidence Curator.

---

## Learning Tables

The feedback loop. These tables make the system improve over time.

### `learned_adjustments`

Runtime intelligence from the Learner. Soft guidance with confidence scores that decay over time.

```sql
CREATE TABLE learned_adjustments (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    skill_id text NOT NULL,
    module text,
    condition_json jsonb NOT NULL,
    instruction_delta text NOT NULL,
    confidence numeric DEFAULT 0.5,
    observation_count integer DEFAULT 0,
    last_reinforced_at timestamp DEFAULT now(),
    decay_status text DEFAULT 'active',
    created_at timestamp DEFAULT now()
);
```

Written by: Learner. Read by: Scorer, Format Selector, Prompt Assembler, Content Suggester. Lifecycle: Signal Hygiene.

### `learning_staging_queue`

Human-gated proposals for permanent skill changes.

```sql
CREATE TABLE learning_staging_queue (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    proposal_type text NOT NULL,
    proposal_content jsonb NOT NULL,
    source_agent text NOT NULL,
    status text DEFAULT 'pending',
    reviewed_by text,
    reviewed_at timestamp,
    created_at timestamp DEFAULT now()
);
```

Written by: Learner, Advisor. Read by: Human review.

### `trust_tracker_state`

System autonomy level. Validator defaults to Level 1 when empty.

```sql
CREATE TABLE trust_tracker_state (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    current_level integer DEFAULT 1,
    assessment_date date,
    assessment_details jsonb,
    created_at timestamp DEFAULT now()
);
```

Written by: Autonomy Manager. Read by: Validator.

### `agent_scratchpads`

Decision logs for all Category A agents. Single table with agent_id discriminator.

```sql
CREATE TABLE agent_scratchpads (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id text NOT NULL,
    run_id uuid DEFAULT gen_random_uuid(),
    run_type text NOT NULL,
    timestamp timestamp DEFAULT now(),
    input_context jsonb,
    decisions_made jsonb,
    outcomes_observed jsonb,
    patterns_detected jsonb,
    learnings_shared jsonb,
    confidence_metadata jsonb
);

CREATE INDEX idx_scratchpads_agent ON agent_scratchpads(agent_id);
CREATE INDEX idx_scratchpads_timestamp ON agent_scratchpads(timestamp);
```

Written by: All Category A agents. Read by: Reporter, Advisor, each agent's own history.

### `lifecycle_logs`

Daily system run summaries.

```sql
CREATE TABLE lifecycle_logs (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    run_date date DEFAULT CURRENT_DATE,
    summary jsonb NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Written by: Signal Hygiene.

### `pipeline_metrics`

Throughput, bottlenecks, revision rates.

```sql
CREATE TABLE pipeline_metrics (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    period text NOT NULL,
    metrics jsonb NOT NULL,
    created_at timestamp DEFAULT now()
);
```

Read by: Reporter.

---

## VOC Scanner Table (Optional)

If using the Voice of Customer Scanner skill (`skills/standalone/voc-scanner/`):

```sql
CREATE TABLE voc_signals (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    person_name text,
    person_title text,
    person_level text,
    persona_type text,
    institution text,
    institution_type text,
    signal_type text,
    pain_domain text,
    summary text,
    quote_snippet text,
    sentiment text,
    voice_context text,
    source_url text UNIQUE,
    source_name text,
    source_authority text,
    published_date date,
    relevance text,
    categories text[],
    score_recency numeric,
    score_specificity numeric,
    score_relevance numeric,
    institutions_mentioned text[],
    competitors_mentioned text[],
    technologies_mentioned text[],
    geography text,
    routed_to text DEFAULT 'filed',
    routed_at timestamp,
    notes text,
    created_at timestamp DEFAULT now()
);
```

Written by: VOC Scanner.

---

## Installation

Run all CREATE TABLE statements in your database's SQL editor (Supabase SQL Editor, psql, or your preferred client). The schema uses PostgreSQL features: `uuid`, `jsonb`, `text[]`, `gen_random_uuid()`.

After creating tables, populate the framework tables with your answers from QUESTIONS.md. See `guides/setup.md` for the full walkthrough.
