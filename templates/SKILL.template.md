---
name: {{AGENT_NAME}}
description: {{BRIEF_DESCRIPTION}}
---

# {{AGENT_NAME}}

**Agent ID:** {{AGENT_ID}}
**Category:** {{CATEGORY}} (A = Autonomous Learner / B = Informed Executor / System)
**Schedule:** {{SCHEDULE}} (e.g., "Daily 8 AM", "Pipeline Step 3", "On-demand")

---

## Purpose

{{Describe what this agent does in 2-3 sentences. What problem does it solve? What is its input and output?}}

---

## Inputs

| Source | What It Reads | Why |
|--------|--------------|-----|
| {{table_name}} | {{columns}} | {{reason}} |

---

## Outputs

| Target | What It Writes | When |
|--------|---------------|------|
| {{table_name}} | {{columns}} | {{condition}} |

---

## Process

### Step 1: {{Step Name}}

{{Describe what the agent does in this step.}}

### Step 2: {{Step Name}}

{{Continue for each step.}}

---

## Rules

1. {{Rule 1 -- what the agent must always do}}
2. {{Rule 2 -- what the agent must never do}}
3. {{Rule 3 -- edge case handling}}

---

## Error Handling

- If {{condition}}: {{action}}
- If no data available: {{fallback behaviour}}

---

## Example Output

```json
{
  "example": "Replace with a realistic output sample"
}
```
