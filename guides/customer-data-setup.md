# Customer Data Setup

How to configure GTM Content Engine when you have access to customer call transcripts, CRM data, or internal customer intelligence.

If you don't have this data, use the [VOC Scanner](voc-setup.md) instead.

---

## What this enables

The **Call Analyst** skill (`skills/independent/call-analyst/`) extracts structured signals from customer conversations:

- Buying signals, objections, and pain points
- Champion and stakeholder identification
- Competitive mentions
- Urgency indicators
- Deal progression evidence

These signals are written to `sales_call_signals` and — when patterns emerge across 3+ calls — promoted to `market_signals` where they feed the content pipeline alongside the Researcher's external signals.

**NDA guardrails are enforced by default.** Customer names, deal specifics, and individual identifiers are never allowed into public-facing content. The Call Analyst flags sensitive signals for team lead review before they enter the content pipeline.

---

## Prerequisites

| Requirement | Purpose |
|------------|---------|
| Transcript provider (Gong, Chorus, Fireflies, or similar) | Source of call transcripts |
| API access to your transcript provider | Call Analyst reads transcripts via API |
| CRM access (optional but recommended) | Cross-reference contacts and deals |

---

## Setup steps

### 1. Configure the customer signals framework

Open `frameworks/customer-signals.md` and populate:

- **Signal taxonomy** — what signal types matter in your sales conversations
- **Pain domains** — the categories of customer pain you want to track
- **NDA rules** — what triggers an NDA flag, who approves flagged signals

### 2. Configure the Call Analyst skill

Open `skills/independent/call-analyst/SKILL.md` and update:

- `[YOUR TRANSCRIPT PROVIDER]` — your provider name
- `[YOUR API BASE URL]` — your transcript API endpoint
- `[YOUR COMPANY]` team email addresses — so the agent knows who is internal
- `[YOUR TEAM LEAD]` — who approves NDA-flagged signals

### 3. Set up transcript API access

The Call Analyst needs to read transcripts. How you connect depends on your platform:

**MCP server approach:**
Configure an MCP server for your transcript provider with API credentials.

**Direct API approach:**
Provide the Call Analyst with a tool that can make authenticated API calls to your provider.

### 4. Create the database table

If you haven't already, run the `sales_call_signals` CREATE TABLE from `architecture/schema.md`.

### 5. Test with one call

Run the Call Analyst against a single recent call transcript. Verify:
- Signals are written to `sales_call_signals`
- NDA flags are applied correctly
- Contact attribution works
- Signal types match your taxonomy

### 6. Set up the schedule

The Call Analyst can run:
- **On webhook** — triggered when a new transcript is available
- **On schedule** — daily batch processing of new transcripts
- **On demand** — manual runs for specific calls

---

## CRM integration (optional)

If you have CRM access, the Call Analyst can cross-reference contacts and deals:

1. Configure your CRM connection (MCP server or API tool)
2. Update the Call Analyst skill with your CRM field mappings
3. The agent will link signals to specific deals, contacts, and accounts

Without CRM access, the Call Analyst still extracts signals — it just can't link them to CRM records.

---

## How customer signals flow into content

```
Customer calls ──→ Call Analyst ──→ sales_call_signals
                                         │
                              (3+ calls, same pattern)
                                         │
                                         ▼
                                   market_signals (source_type: internal)
                                         │
                              ┌──────────┼──────────┐
                              ▼          ▼          ▼
                          Evidence Curator    Content Suggester   Newsletter Curator
                       (evidence)  (ideas)      (editions)
```

Individual call signals stay in `sales_call_signals`. When the Call Analyst detects patterns across 3+ calls over 2+ weeks, it promotes the pattern to `market_signals` with `source_type: internal`. This is where the content pipeline picks it up.

The promotion process strips all identifying information. The content pipeline only ever sees anonymised patterns like "3 prospects in the last month raised concerns about evaluation timelines" — never "Acme Corp's CTO said they can't get through procurement."
