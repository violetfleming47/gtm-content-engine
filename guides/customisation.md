# Customisation Guide

How to adapt GTM Content Engine for your company, industry, and content strategy.

---

## What to customise

GTM Content Engine has three layers:

| Layer | What it is | How to customise |
|-------|-----------|-----------------|
| **Engine** | The 24 agent skills + pipeline logic | Rarely needs changes — the skills are domain-agnostic |
| **Frameworks** | Your ICP, beliefs, pillars, voice, brand | Always customise — this is your business logic |
| **References** | Voice profiles, format specs | Always customise — this is how your content sounds |

Most customisation happens in frameworks and references. The skills themselves should work for any B2B content operation without modification.

---

## Framework customisation

### Start with QUESTIONS.md

[`QUESTIONS.md`](../QUESTIONS.md) walks you through every decision the system needs. Your answers populate the `[YOUR ...]` markers in the frameworks.

### Customisation order

1. **ICP Framework** — who you're targeting (required first, everything depends on this)
2. **Core Beliefs** — what you believe (Scorer uses these immediately)
3. **Content Pillars** — your content themes (most-referenced framework)
4. **Voice Profile** — how you sound (Ghost Writer needs this to draft)
5. **Brand Framework** — visual identity (Visual Producer needs this)
6. **Distribution Channels** — where you publish
7. **Industry Landscape** — your market (Researcher needs this for scanning)
8. **Comms Profiles** — per-persona communication styles
9. **Content Types** — mostly generic, adjust if your formats differ
10. **Customer Signals** — only if you have transcript/CRM data

### Minimum viable configuration

To get the pipeline running, you need at minimum:
- 2–3 ICP profiles in `icp_profiles`
- 3–5 core beliefs in `core_beliefs`
- 3–4 content pillars in `content_pillars`
- A populated voice profile in `references/voice-profile.template.md`

Everything else can be added incrementally. Agents degrade gracefully when data is sparse — they flag gaps rather than failing.

---

## Voice customisation

The voice profile (`references/voice-profile.template.md`) is the single most important customisation after ICP. It defines:

- **Positioning** — what the brand stands for
- **Registers** — tonal range (authoritative, educational, playful, etc.)
- **Voice rules** — non-negotiable dos and don'ts
- **Word bans** — terms to never use, with replacements
- **Signature devices** — recurring rhetorical moves
- **Core arguments** — intellectual positions you're known for

The Ghost Writer loads this before every draft. Format specs in `references/formats/` handle structural rules (word counts, section patterns) separately.

---

## Adding new channels

1. Add a channel entry to `frameworks/distribution-channels.md`
2. Insert it into the `channel_intelligence` table
3. Create a format spec in `references/formats/` if the channel has unique structural rules
4. Update the Ghost Writer's format adaptation section if needed

---

## Adding new ICP personas

1. Add the persona to `frameworks/icp-framework.md`
2. Insert into the `icp_profiles` table
3. Add communication dimensions in `frameworks/comms-profiles.md`
4. Insert into `icp_engagement_approaches`

The Scorer, Researcher, and Contact Matcher will pick up new personas automatically on their next run.

---

## Industry adaptation

GTM Content Engine was designed for B2B content operations. The frameworks are industry-agnostic — the specific industry context comes from:

- `frameworks/industry-landscape.md` — your sectors, competitors, market forces
- `frameworks/icp-framework.md` — your buyer personas
- `frameworks/core-beliefs.md` — your intellectual positions

To adapt for a different industry, fill in these frameworks with your domain context. The agent skills don't need to change.

---

## Extending the system

### Adding a new agent

Use `templates/SKILL.template.md` as your starting point. Every skill needs:
- Identity (what it does, what it doesn't do)
- Context loading (what data it reads)
- Process steps (what it does with the data)
- Output specification (what it writes and where)
- Failure handling (how it degrades)

### Adding a new framework

Use `templates/framework.template.md`. Define:
- What the framework teaches
- Which agents consume it
- The data structure (for the database table)
- Template entries with `[YOUR ...]` markers

Add corresponding questions to QUESTIONS.md.
