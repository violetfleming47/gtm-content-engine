# Setup Guide

Step-by-step installation from clone to running pipeline.

---

## Prerequisites

| Requirement | Purpose | Free tier? |
|------------|---------|-----------|
| PostgreSQL database | Shared state for all agents | Yes (Supabase 500MB) |
| AI platform | Runs the agent skills | Varies |
| Web search capability | Researcher agent needs it | Platform-dependent |

---

## Step 1: Create your database

Go to [supabase.com](https://supabase.com) and create a new project. Any region. The free tier is sufficient for evaluation and moderate usage.

Note your:
- **Project URL** (e.g. `https://xxxxxxxxxxxx.supabase.co`)
- **Service role key** (Settings > API > service_role key)

---

## Step 2: Install the schema

Open the SQL Editor in your Supabase dashboard. Copy and execute the CREATE TABLE statements from [`architecture/schema.md`](../architecture/schema.md).

Install in this order:
1. Framework tables (icp_profiles, core_beliefs, content_pillars, etc.)
2. Pipeline tables (content_items, content_performance, etc.)
3. Intelligence tables (market_signals, sales_call_signals, etc.)
4. Learning tables (learned_adjustments, agent_scratchpads, etc.)

Verify with:
```sql
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public' ORDER BY table_name;
```

---

## Step 3: Answer the questions

Open [`QUESTIONS.md`](../QUESTIONS.md) and work through each section. Your answers populate the frameworks.

---

## Step 4: Populate the frameworks

For each framework in `frameworks/`, replace the `[YOUR ...]` markers with your answers.

You can do this manually or use an agent:
```
"Here are my answers to QUESTIONS.md: [paste answers].
Populate every [YOUR ...] section in the frameworks/ directory.
Preserve the framework structure. Only fill in the marked sections."
```

---

## Step 5: Load frameworks into the database

Insert your populated framework content into the corresponding tables:

| Framework file | Database table |
|---------------|---------------|
| `icp-framework.md` | `icp_profiles` |
| `core-beliefs.md` | `core_beliefs` |
| `content-pillars.md` | `content_pillars` |
| `distribution-channels.md` | `channel_intelligence` |
| `platform-algorithm.md` + `platform-algorithm.md` | `algorithm_knowledge` (see frameworks/platform-algorithm.md) |
| `content-types.md` | `format_constraints` |
| `pillar-subcategories.md` | `format_affinity` |
| `industry-landscape.md` | `industry_map` |
| `comms-profiles.md` | `icp_engagement_approaches` |

---

## Step 6: Configure your voice

Fill in `references/voice-profile.template.md` with your brand voice rules: positioning, registers, dos/don'ts, word bans, signature devices, and core arguments.

This is what the Ghost Writer loads before writing every piece.

---

## Step 7: Populate the context library

Insert evidence into `context_library` — product capabilities, case studies, proof points, company stats. The Evidence Curator selects from this when building evidence packages for content.

Start with 10–20 entries covering your core product claims and strongest proof points. Add more over time.

---

## Step 8: Deploy the skills

See [`deployment.md`](deployment.md) for platform-specific instructions.

---

## Step 9: Run the first pipeline

1. Start the **Researcher** — it scans for market signals and writes to `market_signals`
2. Start the **Content Suggester** — it reads signals and produces content suggestions
3. Submit a brief through the **Router** — or insert directly into `content_items`
4. The **Orchestrator** picks it up and coordinates the pipeline

Monitor progress by checking `content_items.status`.

---

## Verification checklist

- [ ] Database has all tables from schema.md
- [ ] `icp_profiles` has your persona data
- [ ] `core_beliefs` has your beliefs
- [ ] `content_pillars` has your pillars
- [ ] `context_library` has at least 10 evidence entries
- [ ] Voice profile is populated
- [ ] At least one skill is deployed and can read/write to the database
- [ ] A test content item can be scored successfully
