# Deployment Guide

How to deploy GTM Content Engine skills on different AI platforms. Each skill is a system prompt — structured markdown that any LLM can follow.

---

## Cursor / Windsurf / Cline

Copy each `SKILL.md` file into your IDE's skills directory. The agent loads them automatically when relevant tasks match.

**Database connection:** Add the Supabase MCP server to your configuration:
```json
{
  "mcpServers": {
    "supabase": {
      "url": "https://your-project.supabase.co",
      "apiKey": "your-service-role-key"
    }
  }
}
```

---

## Claude Projects

Create one project per agent. Paste the skill content as project instructions. Attach the relevant frameworks as project knowledge.

**Database connection:** Use the Supabase MCP server or provide the connection string as project context.

---

## Claude Code

Add skills as project context. Place them in your project directory and reference them in your CLAUDE.md or project instructions.

---

## ChatGPT

Paste each skill as the system prompt in a custom GPT or as Custom Instructions for a conversation. Attach frameworks as uploaded files.

**Database connection:** Use a Code Interpreter action that connects to your database, or provide raw SQL that the agent can suggest for manual execution.

---

## LangChain / LangGraph / CrewAI

Use each `SKILL.md` as the system prompt for a node in your orchestration graph. Wire database connections as tool definitions.

```python
from langchain.agents import AgentExecutor

scorer_prompt = open("skills/pipeline/scorer/SKILL.md").read()
scorer_agent = AgentExecutor(
    agent=create_agent(llm, tools=[db_tool], prompt=scorer_prompt),
    tools=[db_tool]
)
```

---

## Anthropic Managed Agents

Each skill becomes a managed agent with MCP server connections to your database. This is the most integrated deployment option.

---

## Custom Orchestration

Any system that can:
1. Send a system prompt to an LLM
2. Connect to a PostgreSQL database
3. Schedule recurring runs (for cron agents)

...can run GTM Content Engine skills.

---

## Scheduling

Some agents run on schedules:

| Time | Agent | Purpose |
|------|-------|---------|
| Daily 6 AM | Signal Hygiene | Data hygiene |
| Daily 7 AM | Repetition Monitor | Anti-repetition sweep |
| Daily 8 AM | Researcher | External signal scanning |
| Daily 9 AM | Content Suggester | Content suggestions |
| Saturday | Learner + all Category A agents | Weekly analysis |
| Sunday 8 AM | Autonomy Manager | Autonomy assessment |
| Sunday 10 AM | Reporter | Weekly intelligence report |

Configure these as cron jobs, scheduled tasks, or recurring agent runs in your platform.

---

## Minimum viable deployment

You don't need all 24 agents running on day one. Start with:

1. **Router** — intake
2. **Scorer** — ICP scoring
3. **Ghost Writer** — drafting
4. **Validator** — quality gate

Add agents as you need them. The pipeline degrades gracefully — agents check for upstream data and flag gaps rather than failing.
