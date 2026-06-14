# Questions Your Organisation Needs to Answer

Before this system works for you, you need to teach it how your business thinks about content, customers, and market positioning. That means answering specific questions — and your answers populate the frameworks automatically.

The quality of the system's output is directly proportional to how well you encode your team's judgment here.

---

## How this works

1. **You answer the questions below** — each section maps to specific framework sections
2. **Your answers populate the frameworks** — each framework has `[YOUR ...]` markers showing exactly where your answers go
3. **The agents use the populated frameworks** — they load frameworks from your database and reason against your content

You can populate the frameworks manually (copy answers into the marked sections) or use an agent: give it this document with your answers and the framework templates, and it will fill in every `[YOUR ...]` section automatically.

Each question below shows:
- **Why it matters** — what the system does with the answer
- **Feeds into** — which files and sections to update

Work through the sections in order. Earlier answers inform later ones.

---

## 1. Your ICP (Ideal Customer Profile)

These answers define who you're creating content for — the personas your content system targets.

### Who are you trying to reach?

Define 3–10 persona types. For each: what role do they play, what do they care about, what problems do they face?

**Why it matters:** The Scorer uses ICP profiles to determine who every piece of content is for. The Researcher scans for signals relevant to these personas. The Contact Matcher finds real contacts matching them. Without defined ICPs, the system has no targeting.

**Feeds into:** `frameworks/icp-framework.md` (all persona definitions), `frameworks/comms-profiles.md` (per-persona communication dimensions)

### For each persona: who is the economic buyer and who is the champion?

**Why it matters:** The same persona type (e.g. "Decision Maker" or "End User") often has two roles — the senior person who controls the budget (economic buyer) and the operational person who drives the day-to-day evaluation (champion). They read different content, respond to different framing, and use different language.

**Feeds into:** `frameworks/icp-framework.md` (buyer/champion split per persona)

### What titles do your personas have?

**Why it matters:** Title lists help the Contact Matcher match real contacts to personas, and help the VOC Scanner identify who is speaking in public forums.

**Feeds into:** `frameworks/icp-framework.md` (title lists per persona)

### What are each persona's pain points?

**Why it matters:** Pain points drive content relevance. The Scorer matches content to the personas whose pain it addresses. The Researcher scans for signals about these specific pains.

**Feeds into:** `frameworks/icp-framework.md` (pain points per persona per role)

---

## 2. Your Core Beliefs

These answers define what your company believes — the intellectual positions your content is built on.

### What does your company believe that your market doesn't yet accept?

List 5–20 beliefs. Each should be a clear position that some of your audience would push back on. The best beliefs are specific, defensible, and connected to what you sell.

**Why it matters:** The Scorer selects 1–3 beliefs per piece of content. The Validator checks that content expresses beliefs authentically. Beliefs give your content a consistent intellectual spine — without them, content becomes generic.

**Feeds into:** `frameworks/core-beliefs.md` (belief_slug, belief_statement, counter_positions for each)

**Example:** "Evaluation speed is a competitive advantage, not a risk factor. The institution that validates in weeks will outrun the institution that validates in quarters." Counter-position: "Thorough evaluation protects the institution."

### For each belief: what is the counter-position?

**Why it matters:** Counter-positions make beliefs argumentative rather than declarative. The Ghost Writer can acknowledge the opposing view and argue against it, which creates more compelling content than stating opinions without context.

**Feeds into:** `frameworks/core-beliefs.md` (counter_positions field)

---

## 3. Your Content Pillars

These answers define the thematic territories your content occupies.

### What 4–8 topics do you want to be known for?

Each pillar is a content territory. Name it, define what it covers, and what's out of scope.

**Why it matters:** Pillars are the most-referenced framework in the system (8 agents use them). The Scorer classifies every piece into a pillar. The Repetition Monitor tracks distribution across pillars to prevent over-concentration. The Content Suggester identifies gaps.

**Feeds into:** `frameworks/content-pillars.md` (pillar_slug, definition, sub_topics, adjacent_topics, out_of_scope, target_percentage)

### What percentage of your content should each pillar receive?

**Why it matters:** Target percentages let the Repetition Monitor flag when one pillar dominates and others are neglected.

**Feeds into:** `frameworks/content-pillars.md` (target_percentage per pillar)

---

## 4. Your Voice

These answers define how your brand sounds across all content.

### How would you describe your brand's voice in 3–5 adjectives?

**Why it matters:** The Ghost Writer loads your voice profile before writing every piece. Without a defined voice, content will sound generic.

**Feeds into:** `references/voice-profile.template.md`

### What registers does your voice operate in?

Most brands have 2–3 registers — e.g. authoritative (when presenting evidence), educational (when explaining), playful-confident (when the content has personality). Define yours.

**Why it matters:** Different content types call for different registers. The Ghost Writer selects the register based on the brief.

**Feeds into:** `references/voice-profile.template.md` (registers section)

### What words and phrases are banned?

Every brand has words it never uses. "Excited to announce", "cutting-edge", "synergy", "leverage" — list yours.

**Why it matters:** The Validator's Layer 1 checks for banned phrases. The Ghost Writer avoids them.

**Feeds into:** `references/voice-profile.template.md` (word bans), `references/ghost-writer-foundation.md`

### What are your signature rhetorical devices?

How does your brand make arguments? Platform data references? Reframes at the end? Direct challenges to conventional wisdom?

**Why it matters:** Signature devices make your content recognisable. The Ghost Writer uses them consistently.

**Feeds into:** `references/voice-profile.template.md` (signature devices)

### What are your core arguments?

The recurring intellectual positions your audience associates with you. Different from beliefs — these are the arguments you make repeatedly, not the underlying convictions.

**Feeds into:** `references/voice-profile.template.md` (core arguments)

---

## 5. Your Brand

These answers define your visual and institutional identity.

### What are your brand colours, fonts, and logo usage rules?

**Why it matters:** The Visual Producer uses brand guidelines when creating carousels and graphic briefs.

**Feeds into:** `frameworks/brand-framework.md`

### What is your brand positioning statement?

One paragraph. What you do, who you do it for, and why you're different.

**Feeds into:** `frameworks/brand-framework.md` (positioning section)

---

## 6. Your Industry

These answers define the market landscape you operate in.

### What industry sectors do you serve?

**Why it matters:** The Researcher scans these sectors daily for signals. The Content Suggester turns those signals into content opportunities.

**Feeds into:** `frameworks/industry-landscape.md` (sectors section)

### Who are your competitors?

**Why it matters:** The Researcher monitors competitor activity. The Evidence Curator uses competitive context as evidence in content.

**Feeds into:** `frameworks/industry-landscape.md` (competitors section)

### What market forces are shaping your industry right now?

Regulatory changes, technology shifts, consolidation, emerging trends.

**Feeds into:** `frameworks/industry-landscape.md` (market forces section)

---

## 7. Your Distribution Channels

These answers define where your content is published.

### What channels do you publish on?

List every channel: LinkedIn (personal, company), newsletters, blog, podcast, etc.

**Why it matters:** The Format Selector selects channels per piece. The Distributor creates channel-specific distribution briefs. The Ghost Writer adapts format to the channel.

**Feeds into:** `frameworks/distribution-channels.md` (one entry per channel)

### What content types work on each channel?

For each channel: what formats are allowed? (Text posts, carousels, articles, etc.)

**Feeds into:** `frameworks/distribution-channels.md` (content_types per channel), `frameworks/content-types.md`

---

## 8. Your Communication Profiles

These answers define how each persona prefers to be communicated with.

### For each ICP persona: how do they talk about their problems?

**Why it matters:** The Contact Matcher and Distributor use comms profiles to tailor distribution approaches. The Ghost Writer uses them to match the audience's language.

**Feeds into:** `frameworks/comms-profiles.md` (8 dimensions per persona: language patterns, content preferences, what resonates, what repels, authority signals, typical messaging, preferred tone, brand relevance framing)

---

## 9. Your Customer Data (Optional)

These answers are only needed if you have access to customer call transcripts, CRM data, or internal customer intelligence. If you don't, skip this section and use the VOC Scanner skill instead (see `guides/voc-setup.md`).

### Where do your customer transcripts come from?

**Why it matters:** The Call Analyst extracts signals from transcripts. It needs to know your provider's API, authentication, and data format.

**Feeds into:** `frameworks/customer-signals.md`, `guides/customer-data-setup.md`

### What signal types matter in your sales conversations?

Buying signals, objections, pain points, competitive mentions, urgency indicators — define your taxonomy.

**Feeds into:** `frameworks/customer-signals.md` (signal taxonomy)

### What NDA rules govern your customer data?

**Why it matters:** The Call Analyst applies NDA flags. Customer names and deal details must never leak into public-facing content. The system needs clear rules for what requires review before it enters the content pipeline.

**Feeds into:** `frameworks/customer-signals.md` (NDA guardrails)

---

## What to do with your answers

### Option A: Auto-populate with an agent

Give your answers and the framework templates to an agent in your harness:

```
"Here are my answers to QUESTIONS.md: [paste answers].
Populate every [YOUR ...] section in the frameworks/ directory with the corresponding answers.
Preserve the framework structure. Only fill in the marked sections."
```

The agent will fill in all frameworks based on your answers. Review the output — the frameworks are the reasoning engine, so they need to be accurate.

### Option B: Populate manually

Each framework has `[YOUR ...]` markers that correspond to your answers. Work through each framework file and replace the markers with your content.

### After populating frameworks

1. **Set up the database** — follow `guides/setup.md` to create your database and install the schema from `architecture/schema.md`
2. **Load frameworks into the database** — insert your populated frameworks into the corresponding tables
3. **Configure your voice** — fill in `references/voice-profile.template.md` with your voice rules
4. **Deploy the skills** — load each skill as a system prompt in your AI platform of choice (see `guides/deployment.md`)
5. **Run the pipeline** — start with the Researcher (daily), then submit content through the Router

The frameworks will be incomplete at first. That's expected. The `[YOUR ...]` markers that you can't fill yet stay as prompts — they're templates for the reasoning the system will apply to your data.
