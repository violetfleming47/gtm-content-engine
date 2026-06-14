# Learning System

How GTM Content Engine learns from human edits and gets better over time. This is the core value proposition — the system doesn't just automate content production, it compounds from every piece of feedback.

---

## The Core Signal: Edit Deltas

Every time a human edits a draft before publishing, the system captures the **edit delta** — the difference between what the AI wrote and what the human published. This is the richest learning signal available:

- What was cut reveals what the AI over-produces
- What was added reveals what the AI misses
- What was reworded reveals voice misalignment
- What was restructured reveals format preferences

The Engagement Collector captures edit deltas into `content_performance.edit_delta`. The Learner reads them.

---

## Three Learning Speeds

### Speed 1: Observe (Per Review, Immediate)

**Trigger:** Human edits a draft and publishes (Path A completion).

The **Learner** reads both the original draft and the published version. It writes a free-form synopsis — its interpretation of what changed and why. This is stored in `agent_scratchpads`.

**No system changes.** Speed 1 is pure observation. Synopses accumulate.

**Example:** "For [Persona A] ICP on regulatory compliance content, the human softened the contrarian opening — changed from a provocative challenge to a measured observation. The evidence section was kept intact but the framing shifted from 'institutions are failing' to 'institutions face a structural challenge'. This is the third time in two weeks a contrarian opener was softened for [Persona A] content."

---

### Speed 2a: Adjust (Weekly, Automatic)

**Trigger:** Saturday weekly analysis.

The Learner reviews 4 weeks of observations (~20 synopses), groups by context (ICP x format x pillar x change type), and detects patterns.

- **3+ instances** → emerging pattern, noted in scratchpad
- **5+ instances** → `learned_adjustment` created automatically

Learned adjustments are soft guidance with confidence scores. Pipeline agents treat them as weighted signals, not hard rules.

```
Example learned_adjustment:
{
  "skill_id": "ghost-writer",
  "condition_json": {"icp": "[Persona_A]", "pillar": "regulatory"},
  "instruction_delta": "Soften contrarian openings. Lead with measured observation rather than provocative challenge.",
  "confidence": 0.68,
  "observation_count": 7
}
```

The Scorer, Format Selector, and Prompt Assembler read active adjustments and incorporate them into their outputs. The Ghost Writer receives them through the assembled_prompt as per-piece voice notes.

**Decay:** Adjustments not reinforced by new observations in 60 days lose confidence. Below 0.3 confidence with no reinforcement in 90 days → archived. The system forgets what stops being true.

---

### Speed 2b: Evolve (Weekly, Human-Gated)

**Trigger:** 10+ instances of the same pattern over 4+ weeks.

The Learner writes a skill evolution proposal to `learning_staging_queue` with full provenance traces from pipeline records. This sits pending until human review.

If approved, the relevant skill gets updated (previous version preserved for rollback), and the corresponding learned_adjustment can be archived — the lesson is now permanent.

**Upgrades always require human approval.** The system proposes, it never changes its own skills autonomously.

---

### Speed 3: Approved Suggestions

**Trigger:** Human approves a suggestion from the Advisor.

When conversations with the Advisor lead to concrete proposals (ICP refinements, belief updates, pillar adjustments), the Learner routes approved changes to the appropriate database table.

---

## Category A Self-Analysis

Every Category A agent (Evidence Curator, Contact Matcher, Distributor, Learner, Researcher, Content Suggester, Call Analyst, Response Analyst) runs a **weekly self-analysis** on Saturday:

1. Pull scratchpad entries from past 4 weeks with outcomes
2. Correlate decisions with results
3. Detect patterns (minimum 5 instances)
4. Write distilled learnings to shared intelligence (`content_insights`)

These are automatic runtime learnings — they update shared intelligence without human gating. They never change frameworks or skills. They inform downstream agents with richer context.

**Example:** The Evidence Curator might detect that evidence from `market_signals` (source_type: internal) consistently performs better than external signals for [Persona B] ICP content. This insight gets written to `content_insights` where the Content Suggester reads it and weights internal signals higher for [Persona B]-targeted suggestions.

---

## How Learning Flows Through the Pipeline

```
Human edits draft ──→ Engagement Collector captures edit_delta
                           │
                           ▼
                    content_performance
                           │
                           ▼
                    Learner observes (Speed 1)
                           │
                           ▼
                    agent_scratchpads (synopses accumulate)
                           │
                    Saturday weekly analysis
                           │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
        5+ instances   10+ instances   Category A
              │             │           self-analysis
              ▼             ▼             │
     learned_adjustments  learning_       ▼
     (automatic)          staging_queue  content_insights
              │           (human-gated)  (shared brain)
              ▼                           │
     Scorer reads ────────────────────────┘
     Format Selector reads
     Prompt Assembler reads
              │
              ▼
     Better briefs ──→ Ghost Writer ──→ Closer drafts
              │
              ▼
     Fewer human edits ──→ System compounds
```

---

## Trust & Autonomy

The **Autonomy Manager** manages three autonomy levels that govern how much human oversight the pipeline requires:

### Level 1 — Full Review (Default)
All content goes to human review. Starting level and safety default.

### Level 2 — Standard
High-scoring content auto-approves. Upgrade requires: 50+ pieces at Level 1, edit rate below 30%, no trust incidents in 4 weeks, **human approval**.

### Level 3 — High Trust
Lower auto-approve threshold with 20% spot-checking. Upgrade requires: 100+ pieces at Level 2, auto-approved quality within 10% of human-reviewed, edit rate below 20%, no trust incidents in 8 weeks, 95%+ spot-check pass rate, **human approval**.

**Key principles:**
- Upgrades always require human approval
- Downgrades can happen immediately (safety)
- Autonomy Manager unavailable → default to Level 1 (never default upward)
- Recovery requires the same criteria as the original upgrade

---

## What Makes This Work

**Edit deltas are the signal.** Not engagement metrics (lagging), not self-assessment (circular), not user ratings (sparse). The gap between AI draft and human-published version is immediate, specific, and rich.

**Context matters.** The Learner doesn't just track "openings are too aggressive." It tracks "openings are too aggressive for [Persona A] ICP on regulatory content in LinkedIn format." The adjustment is scoped to the specific context where the pattern appears.

**Confidence decays.** An adjustment that was true 6 months ago might not be true today. Unreinforced patterns fade automatically. The system stays current.

**Humans stay in the loop.** Speed 1 and 2a are automatic — they adjust within existing boundaries. Speed 2b (permanent skill changes) always requires human approval. The system can tune itself but cannot redesign itself.

---

## Database Tables for Learning

| Table | Role in Learning |
|-------|-----------------|
| `content_performance` | Stores edit deltas and engagement metrics |
| `agent_scratchpads` | Stores observations, self-analyses, decision logs |
| `learned_adjustments` | Stores active runtime adjustments with confidence scores |
| `learning_staging_queue` | Stores human-gated proposals for permanent changes |
| `content_insights` | Stores shared intelligence from Category A self-analyses |
| `trust_tracker_state` | Stores current autonomy level |

See `architecture/schema.md` for full table definitions.
