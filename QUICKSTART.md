# Quickstart: First Scored Content in 15 Minutes

Clone to scored content in 15 minutes. You need a Supabase account (free tier works) and access to an AI platform that supports system prompts.

---

## Steps

### 1. Clone the repo

```bash
git clone https://github.com/violetfleming47/gtm-content-engine.git
cd gtm-content-engine
```

### 2. Create a Supabase project

Go to [supabase.com](https://supabase.com) and create a new project. The free tier is sufficient.

### 3. Install the schema

Open the **SQL Editor** in your Supabase dashboard. Copy the CREATE TABLE statements from `architecture/schema.md` and execute them.

Start with these essential tables:
- `icp_profiles`
- `core_beliefs`
- `content_pillars`
- `content_items`
- `market_signals`
- `learned_adjustments`
- `agent_scratchpads`

### 4. Verify schema installation

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
ORDER BY table_name;
```

You should see the tables you created.

### 5. Seed minimal framework data

You need at least one ICP profile and one content pillar for the Scorer to work. Run this in the SQL Editor:

```sql
-- One ICP profile
INSERT INTO icp_profiles (category, role, pain_points, use_cases)
VALUES (
    '[Your Persona]',
    'Champion',
    '["Evaluation takes too long", "Cannot compare vendors side-by-side", "Every project starts from scratch"]'::jsonb,
    '["Vendor evaluation", "AI experimentation", "Platform comparison"]'::jsonb
);

-- One content pillar
INSERT INTO content_pillars (pillar_slug, definition, sub_topics, target_percentage)
VALUES (
    'industry-expertise',
    'Your primary authority pillar. Education on the core problem your company solves.',
    'Market trends, best practices, common mistakes, emerging patterns',
    40
);

-- One core belief
INSERT INTO core_beliefs (belief_name, belief_slug, belief_statement, relevant_icps, relevant_pillars)
VALUES (
    'Speed is a competitive advantage',
    'speed-advantage',
    'The organisation that evaluates and decides fastest will outperform the one that evaluates most thoroughly.',
    ARRAY['[Your Persona]'],
    ARRAY['industry-expertise']
);
```

### 6. Load the Scorer skill into your AI platform

The Scorer skill is at `skills/pipeline/scorer/SKILL.md`. Load it according to your platform:

**Cursor / Windsurf / Cline:**
Copy `skills/pipeline/scorer/SKILL.md` into your IDE's skills directory.

**Claude Projects:**
Create a new project. Paste the full contents of `SKILL.md` into the project instructions.

**ChatGPT:**
Paste the full contents of `SKILL.md` as Custom Instructions or as the system prompt in a new GPT configuration.

**Any other platform:**
Use `SKILL.md` as the system prompt for a new agent/conversation.

### 7. Connect the agent to your database

The agent needs read/write access to your Supabase Postgres database. Connection method depends on your platform:

**Cursor (via MCP):**
Add the Supabase MCP server to your configuration with your project URL and service role key.

**Claude / ChatGPT / other:**
Provide the agent with your Supabase connection string or configure a database tool. Available in Supabase under **Settings > Database > Connection string**.

### 8. Create a test content item

Insert a raw content brief for the Scorer to process:

```sql
INSERT INTO content_items (origin, raw_input, submitted_by, status)
VALUES (
    'brief',
    'Write about why enterprises need to evaluate AI vendors faster. The procurement bottleneck is killing innovation programmes. Most organisations take 6-18 months to evaluate a single vendor.',
    'quickstart',
    'queued'
);
```

### 9. Run the Scorer

Ask the agent:

> "Score the queued content item. Read the content_items table for items with status 'queued', then score against ICP profiles, core beliefs, and content pillars."

The Scorer will read the content item, query the framework tables, and write a `scoring_result` to the record.

### 10. Verify scoring output

```sql
SELECT id, status, scoring_result
FROM content_items
WHERE submitted_by = 'quickstart';
```

You should see `status = 'scoring_complete'` and a JSONB `scoring_result` containing ICP match, belief alignment, and pillar classification.

---

## What just happened

You ran the first step of the content pipeline. The Scorer read your brief, queried your ICP profiles, core beliefs, and content pillars, and produced a structured scoring result that tells downstream agents:

- **Who** this content is for (ICP match with confidence percentage)
- **What** it believes (selected core beliefs with expression strength)
- **Where** it fits (content pillar alignment)
- **How** to frame it (framing direction incorporating any learned adjustments)

In the full pipeline, this scoring result flows to the Format Selector (format selection), Evidence Curator (evidence curation), and Repetition Monitor (repetition monitoring) — all running in parallel. Then the Prompt Assembler translates everything into a prose brief, and the Ghost Writer produces a draft in your voice.

---

## Next: configure for YOUR content

The seed data demonstrates the pipeline shape. To point this at your real operation:

1. Open [`QUESTIONS.md`](QUESTIONS.md) and answer the questions about your ICP, beliefs, pillars, voice, and channels.
2. Your answers populate the `[YOUR ...]` markers in the frameworks.
3. Load your populated frameworks into the database tables.
4. Fill in `references/voice-profile.template.md` with your voice rules.
5. Deploy all 24 skills to your platform of choice (see `guides/deployment.md`).

See [`guides/setup.md`](guides/setup.md) for the complete walkthrough.
