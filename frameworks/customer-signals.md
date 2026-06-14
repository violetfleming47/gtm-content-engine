# [YOUR COMPANY] Signal Spotting Framework

**Purpose:** Teaches the Call Analyst agent (and any agent reading `sales_call_signals`) how to identify, classify, attribute, tag, and reason about signals found in [YOUR TRANSCRIPT PROVIDER] call content. This is not a lookup table of phrases — it's a reasoning framework for interpreting any conversation.

**Runtime context:** This doc is metadata for the GTM Content Engine system. The Call Analyst agent reads this to know how to process [YOUR TRANSCRIPT PROVIDER] webhook payloads and write structured signal records to the `sales_call_signals` table. Other agents (Market Analyst, Content Suggester, Evidence Curator, Contact Matcher, Profiler) read those records via tagged queries.

**How this doc works:**
1. Section 1 defines the **signal taxonomy** — the `signal_type` enum values and what each means
2. Section 2 defines the **signal record structure** — what gets written to the database
3. Section 3 defines the **attribution model** — how to attach signals to people, deals, accounts
4. Section 4 defines the **tagging rules** — `relevant_icps`, `urgency`, and cross-agent discoverability
5. Sections 5–10 define the **recognition reasoning** — how to spot each signal class in call content. These are not exhaustive phrase lists. They teach *reasoning patterns* so the agent can identify signals regardless of exact wording
6. Section 11 defines **pattern detection and promotion** — when `sales_call_signals` patterns should be promoted to `market_signals`
7. Section 12 defines **cross-call reasoning** — how to build intelligence across multiple calls for a deal or account

**Companion assets:**
- **[YOUR TRANSCRIPT PROVIDER] Sales Call Metadata Framework** — Schema, CRM linkage, data quality. Read that first to understand the input data
- **[YOUR CRM] Metadata Framework** — CRM fields where signals should be validated/cross-referenced
- **[YOUR COMPANY] Pipeline Framework** — Authoritative stage definitions. Signals map to stage progression evidence
- **Data Trust Hierarchy** — When call signals contradict CRM data, call data generally wins
- **GTM Content Engine System Architecture** — Runtime, database tables, agent communication, tagging system

---

## 1. Signal Taxonomy

Every signal extracted from a call gets exactly one `signal_type`. This is the primary classification and must use these exact enum values (shared across `sales_call_signals` and `market_signals`):

### 1.1 Signal Type Definitions

| `signal_type` | Definition | Example recognition |
|---|---|---|
| `pain_point` | Prospect describes a problem, frustration, or inefficiency they're experiencing. The foundation signal — without pain, there's no deal | Describing team bottlenecks, quantifying cost of status quo, comparing current state to desired state |
| `buying_signal` | Prospect demonstrates active movement toward a purchase decision. Distinct from pain — pain is "we have a problem", buying is "we want to solve it now" | Asking about pricing, requesting a pilot, introducing procurement, scheduling follow-ups proactively |
| `risk_signal` | Anything indicating a deal is stalling, a customer is disengaging, or a relationship is deteriorating | Priority shifts, budget freezes, champion going quiet, reorgs, repeated delays |
| `competitive` | Information about alternatives the prospect is considering — named competitors, build-vs-buy, consultant recommendations, status quo defence | Mentioning another vendor, describing an internal build effort, consultant recommending a different approach |
| `champion_signal` | Evidence about the strength, identity, or behaviour of an internal advocate | Who's driving meetings, who shares internal context, who asks for ammunition to sell internally |
| `stakeholder_signal` | Information about the prospect's decision-making structure — who's involved, who has authority, how decisions get made | Naming decision-makers, describing approval processes, introducing new attendees, describing committees |
| `urgency_signal` | Compelling events, deadlines, or time-bound pressures driving the deal forward (or their absence) | [Persona E]y deadlines, budget cycles, board mandates, programme launch dates |
| `objection` | Explicit pushback, concern, or resistance from the prospect | Price concerns, feature gaps, security worries, implementation concerns, timing pushback |
| `requirement` | Specific technical, commercial, or operational requirement the prospect states | Integration needs, compliance standards, data sovereignty, deployment models, security certifications |
| `sentiment` | General tone or relationship health signal that doesn't fit the above categories but matters | Enthusiasm, frustration, trust-building moments, rapport shifts, disengagement |
| `expansion` | Customer expressing interest in doing more — new use cases, new teams, new business units (Phase 3 only) | Other teams showing interest, new use case discussions, requests for capabilities beyond current scope |
| `churn_risk` | Customer signals indicating renewal or retention risk (Phase 3 only) | Declining usage mentions, value questioning, champion departure, competitive alternatives being explored |

### 1.2 Signal Type Selection Rules

A single moment on a call may contain multiple signal types. The agent should create **separate signal records** for each distinct signal type, even if they come from the same statement.

**Example:** "We need this in place before the regulator reviews us in March, but I'm not sure we have budget approved yet"
- Signal 1: `urgency_signal` — regulatory deadline in March
- Signal 2: `risk_signal` — budget not approved
- Signal 3: `stakeholder_signal` — implies an Economic Buyer who hasn't engaged yet

When in doubt about classification, prefer the more specific type. `pain_point` is more useful than `sentiment`. `urgency_signal` is more useful than `buying_signal`.

---

## 2. Signal Record Structure

Every signal extracted from a call gets written to `sales_call_signals` with this structure:

```
sales_call_signals:
  - id                      (auto-generated)
  - created_at              (timestamptz — when the signal was written by the agent)
  - meeting_uuid            (text — [YOUR TRANSCRIPT PROVIDER] meeting UUID, links back to the source call)
  - meeting_date            (timestamptz — when the call happened)
  - meeting_type            (text — [YOUR TRANSCRIPT PROVIDER] meeting type label: "Discovery", "Demo", etc.)

  # Signal classification
  - signal_type             (text — exact enum from Section 1. One per record)
  - signal_summary          (text — 1-2 sentence interpretation in the agent's own words)
  - signal_detail           (text — fuller context: what was said, by whom, what preceded it)

  # Attribution (Section 3)
  - attributed_to           (text — email of the person who produced the signal)
  - attributed_to_name      (text — display name)
  - attributed_to_role      (text — agent's role assessment: "Champion", "Economic_Buyer",
                              "[Persona_C]", "[Persona_A]", "[Persona_D]",
                              "Influencer", "Unknown")

  # CRM linkage (from [YOUR TRANSCRIPT PROVIDER] crm_associations)
  - deal_id                 (text, nullable — [YOUR CRM] deal ID)
  - account_id              (text, nullable — [YOUR CRM] company ID)
  - contact_ids             (text[], nullable — [YOUR CRM] contact IDs)

  # Tagging (Section 4)
  - relevant_icps           (text[] — which ICP profiles this signal matters to)
  - meddicc_element         (text, nullable — "metrics", "economic_buyer",
                              "decision_criteria", "decision_process",
                              "identify_pain", "champion", "competition")
  - pipeline_stage_evidence (text, nullable — "MQL", "SAL", "SQL", "Evaluating",
                              "Technical_DD", "Contract_Negotiation", or Phase 3 stage)
  - urgency                 (text — "time_sensitive", "current", "developing",
                              "evergreen", "background")
  - confidence              (text — "high", "medium", "low")

  # Pain-specific (Section 5, only when signal_type = "pain_point")
  - pain_domain             (text, nullable)
  - severity                (text, nullable — "critical", "high", "medium", "low", "polite")

  # Metadata
  - [your-company]_team_present    (text[] — @[YOUR DOMAIN] attendees on this call)
  - status                  (text — "active", "archived", "promoted")
  - promoted_to             (text, nullable — market_signals ID if promoted)
```

### 2.1 Writing Rules

**One signal per record.** Don't bundle "they mentioned pain AND asked about pricing" into one record. That's a `pain_point` and a `buying_signal` — two records.

**Signal summary is the agent's interpretation, not a transcript quote.** Write what it *means*, not what was *said*. The `signal_detail` field carries the context.

**Good signal_summary:** "[Example Company A]'s PoC setup process currently takes 6+ months, creating a bottleneck in their vendor evaluation programme. The innovation team lead described this as the primary frustration."

**Bad signal_summary:** "Tom said 'it takes us ages to get PoCs up and running, probably six months or more.'"

The summary is what other agents read when they query `sales_call_signals`. It must be self-contained, interpretive, and useful without needing to go back to the source call.

**Always attribute.** Every signal came from someone's mouth. If you can identify who, record it. If you can't (e.g., group discussion), set `attributed_to` to the most senior external attendee and note the ambiguity in `signal_detail`.

---

## 3. Attribution Model

Signals don't float in the air. Someone said something, on a call about a specific deal, at a specific company. The attribution model ties signals to people, deals, and accounts — this is how other agents find and cross-reference signal intelligence.

### 3.1 Person Attribution

For every signal, identify **who produced it** (not who was on the call — who *said the thing*):

**From [YOUR TRANSCRIPT PROVIDER] notes:** AI notes often attribute statements to speakers. When the notes say "Tom discussed...", attribute to Tom.

**From attendee list:** When notes don't attribute clearly, use the attendee list and reasoning:
- If one external attendee → they said it
- If multiple external attendees from the same company → attribute to the most senior person unless the notes clearly indicate otherwise
- If multiple companies on the call → use context to determine which company's representative spoke

**From [YOUR CRM] cross-reference:** Once you have the person's email (from [YOUR TRANSCRIPT PROVIDER] attendees), check CRM associations to get their [YOUR CRM] contact ID. Populate `contact_ids`. This is how the Contact Matcher and Profiler later find signals about specific people.

### 3.2 Role Attribution

Every attributed person gets a role assessment. This is the agent's best judgement based on all available evidence — their title, their behaviour on the call, their position in the hierarchy, and what they're saying:

| `attributed_to_role` | How to identify from behaviour | Pipeline Framework persona |
|---|---|---|
| `Champion` | Drives meetings, shares internal context, asks for ammunition, takes action items, pushes things forward | [Persona B] — [Example Title], Platform Lead, Project Lead |
| `Economic_Buyer` | Controls budget, has authority to approve spend, discusses budget and terms, is introduced as the decision-maker | Sponsor — VP/SVP Innovation, CTO, CPO, Business Line Head |
| `[Persona_C]` | Assesses technical fit, asks about architecture/integration/security, evaluates against technical requirements | [Persona C] — [Example Title], Architect, InfoSec |
| `[Persona_A]` | Controls process gates — procurement, [YOUR COMPLIANCE PROCESS], legal, compliance. Can block but rarely initiates | [Persona A] — [Example Title] Analyst, Procurement Ops, Legal |
| `Operator` | Will use the platform day-to-day. Focuses on usability, workflow fit, practical concerns | Operator — Analyst, Associate, Programme Coordinator |
| `Influencer` | Shapes the decision without owning it — consultant, advisor, peer | External advisors, consulting firm contacts, industry peers |
| `Unknown` | Can't determine role from available evidence | Default when insufficient data. Flag for investigation |

**Role is assessed from behaviour, not just title.** Don't just look at their job title. Look at:
- Who asks the most questions? → Likely `Champion` or `[Persona_C]`
- Who describes pain from personal experience? → Likely `Champion` or `Operator`
- Who takes action items? → Likely `Champion`
- Who mentions internal politics or decision processes? → Likely `Champion` sharing intel
- Who asks about compliance/security? → Likely one of your defined personas
- Who asks about pricing/terms? → Likely `Economic_Buyer` or their proxy
- Who was introduced by the Champion? → Determine from their focus area

**Role can change across calls.** Someone who was `Unknown` on Call 1 may become clearly a `Champion` by Call 3. The agent should update its assessment based on accumulated evidence — the latest signal record for a person represents the current role assessment.

### 3.3 Deal and Account Attribution

Use [YOUR TRANSCRIPT PROVIDER]'s `crm_associations` to link signals to [YOUR CRM] records:

**Single deal associated:** Attribute all signals to that deal.

**Multiple deals associated:** [YOUR TRANSCRIPT PROVIDER] over-associates (see [YOUR TRANSCRIPT PROVIDER] Metadata Framework Section 4.2). The agent must reason about which deal the signal actually relates to:
- Check the meeting subject and type for context
- Check which deal's stage makes sense for the conversation (a Discovery call probably relates to an early-stage deal, not a renewal)
- If genuinely ambiguous, attribute to all associated deals but note the ambiguity in `signal_detail`

**No deal associated:** The signal still has value. Attribute to the account (`account_id`). It may relate to a deal that doesn't exist in [YOUR CRM] yet (pre-pipeline) or to the account relationship generally.

**No CRM association at all:** Still write the signal. Use attendee email domains to identify the company. Note in `signal_detail` that there's no CRM linkage. The Contact Matcher may be able to match later.

---

## 4. Tagging Rules

Tags are the connective tissue of the entire GTM Content Engine system. They're how the Market Analyst finds patterns, the Content Suggester identifies content opportunities, and the Evidence Curator selects evidence. Getting tags wrong means the signal is invisible to other agents.

### 4.1 ICP Tagging (`relevant_icps`)

Every signal gets tagged with which ICP profiles it's relevant to. This is an array — a single signal can be relevant to multiple ICPs. These values must match `icp_profiles` names exactly:

**Exact allowed values:**
- `Impact Prover`
- `[Friction Persona]`
- `Governance Implementer`
- `Data Foundation Fixer`
- `Cost-Conscious Operator`
- `ROI Defender`
- `[Persona E]y Risk Holder`
- `Decision Protector`
- `Portfolio Optimiser`

**Tagging reasoning — tag the ICPs who would *care about* this signal if they saw it:**

| Signal content | ICP tag reasoning |
|---|---|
| Pain about evaluation timelines or vendor management | `Impact Prover` (needs to show throughput), `[Friction Persona]` (responsible for evaluation quality) |
| Pain about innovation ROI or budget justification | `ROI Defender` (defending the budget), `Impact Prover` (proving impact) |
| Pain about regulatory compliance or governance | `Governance Implementer` (responsible for compliance), `[Persona E]y Risk Holder` (owns the risk) |
| Pain about data provisioning or infrastructure | `Data Foundation Fixer` (owns data infrastructure), `[Friction Persona]` (needs test data) |
| Pain about cost of evaluation or operational overhead | `Cost-Conscious Operator` (focused on efficiency), `ROI Defender` (justifying spend) |
| Buying signals (pricing, procurement, pilot requests) | Tag the ICP of the person producing the signal based on their role and pain context |
| Competitive signals | Tag the ICPs that care about the differentiator mentioned. Security → `Governance Implementer`, `[Persona E]y Risk Holder`. Cost → `Cost-Conscious Operator`, `ROI Defender` |
| Champion signals | Tag the ICP the champion maps to based on their role and the pain they're championing |
| Stakeholder signals | Tag the ICP each identified stakeholder maps to |
| Objection or requirement | Tag the ICPs whose concerns the objection/requirement reflects |

**Why this matters:** The Market Analyst queries `SELECT * FROM sales_call_signals WHERE 'Governance Implementer' = ANY(relevant_icps) AND status = 'active'`. If you tagged a regulatory compliance pain as only `[Persona E]y Risk Holder` and missed `Governance Implementer`, the Market Analyst can't find the pattern. Over-tagging (2-3 ICPs) is better than under-tagging.

### 4.2 Urgency Tagging

Every signal gets an `urgency` value that determines lifecycle and consumption priority:

| `urgency` | Definition | Lifecycle | Typical signals |
|---|---|---|---|
| `time_sensitive` | Has a specific deadline or will become irrelevant soon | Active 14 days max | [Persona E]y deadlines, budget cycle endings, event-driven urgency |
| `current` | Reflects the present state. True now but may change | Active 21 days | Current pain, active buying signals, ongoing champion behaviour |
| `developing` | Emerging pattern. Not fully formed but worth watching | Active 30 days | Early pain recognition, first mention of a topic, new stakeholder |
| `evergreen` | Structural truth. Won't change quickly | Active 60 days | Organisational structure, decision processes, established competitive positions |
| `background` | Context that enriches understanding but doesn't drive action | Active 90 days | Relationship history, company culture, personal preferences |

**These lifecycle values are enforced by the Signal Hygiene system skill (daily 3 AM cron).** Signals past their lifecycle window get archived automatically. Setting the right urgency determines how long the signal stays discoverable.

### 4.3 Confidence Tagging

| `confidence` | When to use |
|---|---|
| `high` | Direct, unambiguous statement. Clear speaker attribution. Corroborated by other evidence |
| `medium` | Reasonable interpretation but requires inference. Speaker attribution probable but not certain. Single data point |
| `low` | Significant interpretation required. Could mean multiple things. Speaker uncertain. Contradicts other evidence |

### 4.4 MEDDICC Element Tagging

Every signal gets tagged with the MEDDICC element it provides evidence for (if applicable). This is how the Contact Matcher and deal-level reporting assess qualification from call evidence.

| Signal type | Primary MEDDICC element | Secondary elements |
|---|---|---|
| `pain_point` | `identify_pain` | — |
| `buying_signal` (budget/pricing related) | `economic_buyer` | `metrics` if ROI discussed |
| `buying_signal` (process/timeline related) | `decision_process` | — |
| `buying_signal` (evaluation/comparison) | `decision_criteria` | `competition` |
| `risk_signal` | Depends on what's at risk — champion fading → `champion`, budget cut → `economic_buyer`, priority shift → `identify_pain` weakening | — |
| `competitive` | `competition` | `decision_criteria` (what they're comparing on) |
| `champion_signal` | `champion` | — |
| `stakeholder_signal` | `decision_process` | `economic_buyer` if the EB is identified |
| `urgency_signal` | `decision_process` | `identify_pain` (compelling event validates urgency of pain) |
| `objection` | Depends on target — price → `economic_buyer`, feature → `decision_criteria`, timing → `decision_process` | — |
| `requirement` | `decision_criteria` | `decision_process` if procurement/compliance requirement |
| `expansion` | New MEDDICC cycle for expansion deal | `metrics`, `champion` (may be different for expansion) |
| `churn_risk` | `champion` (health), `metrics` (value delivery), `competition` (alternatives) | — |

### 4.5 Pipeline Stage Evidence Tagging

Map each signal to the pipeline stage it provides evidence for:

| Evidence pattern | `pipeline_stage_evidence` | What it proves |
|---|---|---|
| Pain described, ICP fit confirmed, willingness to explore | `SAL` | Prospect can articulate a problem [YOUR COMPANY] might solve |
| Budget holder identified, success metrics discussed, top-3 priority confirmed | `SQL` | Deal is qualified. Clear path to cash |
| Pilot commitment, procurement engaged, defined success criteria | `Evaluating` | Organisational resources committed to testing |
| InfoSec/[YOUR COMPLIANCE PROCESS] underway, technical requirements being collected, verbal yes | `Technical_DD` | Supporting their internal approval process |
| Contract terms under discussion, legal engaged, pricing agreed | `Contract_Negotiation` | Administrative closure underway |
| Usage feedback, value realisation, expansion interest, renewal signals | Phase 3 stage | Customer lifecycle signals |

---

## 5. Pain Signal Recognition

Pain signals are `signal_type = "pain_point"`. They also get `pain_domain` and `severity`.

### 5.1 Pain Domains

Every pain signal gets classified into a domain. This is how the Market Analyst spots thematic patterns across calls:

| `pain_domain` | What it covers | [YOUR COMPANY] relevance |
|---|---|---|
| `vendor_evaluation` | Evaluation cycle length, vendor triage, PoC setup, assessment standardisation, comparison methodology | Core [evaluation environment] + marketplace value prop |
| `innovation_programme` | ROI on innovation spend, throughput measurement, [evaluation event]/programme operations, innovation-to-production gap | [Evaluation Environment] operations + metrics layer |
| `regulatory_compliance` | [Persona E]y mandates, AI governance, [evaluation environment] requirements for regulators, compliance testing | [Persona E]y [evaluation environment] capability |
| `infrastructure` | Test environments, data provisioning, synthetic data, API integration, multi-cloud complexity | Technical [evaluation environment] capability |
| `procurement_onboarding` | [YOUR COMPLIANCE PROCESS] bottleneck, InfoSec review timelines, vendor onboarding speed, procurement complexity | [Persona A] pain — [YOUR COMPANY] pre-clearance value |
| `resource_constraint` | Team capacity, headcount limits, manual process overhead, skilled staff shortage | Automation and efficiency argument |
| `stakeholder_alignment` | Internal disagreement, cross-team coordination, competing priorities, organisational silos | Platform as common ground / shared visibility |
| `budget_justification` | Can't prove ROI, innovation budget under threat, cost of status quo not quantifiable | Metrics and reporting capability |
| `competitive_pressure` | Falling behind peers, market window closing, competitors moving faster | Urgency driver — speed to evaluate and deploy |
| `technology_sprawl` | Too many tools, disjointed evaluation processes, fragmented vendor landscape | Platform consolidation argument |

### 5.2 Pain Severity Assessment

| `severity` | Recognition pattern | Deal implication |
|---|---|---|
| `critical` | External forcing function (regulation, board mandate) + prospect quantifies cost/impact + it's a named priority | Deal should be active. If it's not → something else is blocking |
| `high` | Prospect describes the problem specifically, from personal experience, with impact on their work | Real deal territory. MEDDICC should be advancing |
| `medium` | Prospect acknowledges the problem but hasn't quantified it or prioritised it | Potential deal but needs a compelling event to create urgency |
| `low` | Prospect mentions the topic in general terms. No personal stake, no quantification | Not yet a deal driver. Ecosystem-level insight |
| `polite` | Prospect says things like "interesting" or "we should explore" without specificity or commitment | Not pain. This is politeness. Do not create a pain record with severity "polite" — instead, skip or record as `sentiment` |

### 5.3 Pain Recognition Reasoning

The agent should NOT match phrases. It should reason:

**Is it pain?** Is the prospect describing something that is *costing them* — time, money, resources, reputation, opportunity? If yes → pain. If they're describing something they find *intellectually interesting* → not pain, probably `sentiment`.

**Is it their pain?** Is the person speaking affected personally or is their team affected? First-person, present-tense pain ("We struggle with...", "My team spends 60% of their time on...") is stronger than third-person, hypothetical pain ("I hear some teams have challenges with..."). First-person pain has a natural `attributed_to` — the speaker. Third-person pain may indicate the speaker is relaying someone else's problem, which is still useful but lower confidence.

**Is it acute?** Is this a top-3 priority or background noise? Intensity markers: urgency in language, quantification of impact, connection to business outcomes, mention of consequences if unsolved, escalation to leadership.

**Is it [YOUR COMPANY]-relevant?** Does this pain map to something [YOUR COMPANY] can solve? Tag the `pain_domain`. If the pain doesn't fit any domain, it may not be a signal worth recording — unless it reveals something about the account's broader situation.

**Has it deepened since last call?** Compare with prior signals for this deal/account. Pain that gets more specific, more quantified, or more urgent across calls → update severity upward, note the progression. Pain that stays at the same level of generality → not being prioritised internally.

---

## 6. Buying Signal Recognition

Buying signals are `signal_type = "buying_signal"`. They indicate movement toward a purchase, distinct from pain (the problem) and urgency (why now).

### 6.1 Buying Signal Strength

The agent should assess where a buying signal falls on the commitment spectrum and capture this in `signal_detail`:

| Strength | Recognition pattern | Interpretation |
|---|---|---|
| **Active commitment** | Prospect takes an action — schedules procurement meeting, sends security questionnaire, requests pilot proposal, introduces their VP | They are investing effort to advance the deal. Strongest signal. `confidence = high` |
| **Future intent** | Prospect describes an action they plan to take — "I'll bring this to my team meeting", "Let me talk to our CTO" | Intent exists but hasn't converted to action yet. `confidence = medium`. Follow up to check if they did it |
| **Information seeking** | Prospect asks for specifics — pricing, implementation timelines, reference customers, technical docs | Evaluation mode. Genuine interest but not yet commitment. `confidence = medium` |
| **Passive interest** | Prospect says positive things without committing — "This is really interesting", "I can see how this would help" | Intellectual engagement only. Not a buying signal in isolation. Record as `sentiment`, not `buying_signal` |

### 6.2 Buying Signal Escalation (Cross-Call Pattern)

Across calls, buying signals should escalate. The agent should track whether the pattern is:

**Escalating (healthy):** Discovery questions → Evaluation questions → Process questions → Commitment actions. Each call shows a higher level of commitment than the last.

**Flat (stalling):** Same type of questions call after call. No deepening. No new stakeholders. No process language. Buying signals exist but don't progress.

**Declining (dying):** Questions becoming more general. Fewer people on calls. Longer gaps. Previous commitments not followed through. Active buying signals reverting to passive interest.

This assessment goes into the cross-call deal health summary (Section 12).

---

## 7. Risk & Stall Signal Recognition

Risk signals are `signal_type = "risk_signal"`. Churn risk signals for existing customers use `signal_type = "churn_risk"`.

### 7.1 Risk Recognition Reasoning

Risk signals are often implicit. The agent should reason about:

**What changed?** Any shift from previous calls — fewer attendees, different people, changed tone, postponed timelines, new caveats on previous commitments. Change is the signal. Record what was different from expectations.

**What's missing?** Expected progression that didn't happen — no next steps agreed, no new stakeholders introduced when the stage demands them, no procurement discussion at a supposedly SQL+ deal. Absence is a signal (see Section 12.2).

**What's repeated?** The same conversation happening again — same pain described without deepening, same questions asked, same vague commitments without follow-through. Repetition without progression is a stall. This is particularly dangerous because it *looks* like engagement (meetings are happening!) but nothing is advancing.

**The "everything's fine" trap (Phase 3):** "Everything's great" with declining usage is worse than "we're frustrated" with active engagement. The frustrated customer is telling you what to fix. The "everything's fine" customer is disengaging quietly. When recording Phase 3 sentiment, cross-reference against any available usage data or activity patterns.

### 7.2 Risk Severity

| Severity | Pattern | `urgency` tag |
|---|---|---|
| **Critical** | Champion departing, budget cancelled, explicit "we're looking at alternatives", renewal declined | `time_sensitive` |
| **High** | Priority shift, reorg affecting the team, Economic Buyer disengaging, repeated delays without explanation | `current` |
| **Medium** | Meeting cadence slowing, fewer attendees, timeline pushing vaguely | `developing` |
| **Low** | Single instance of rescheduling, passing mention of competing priorities | `background` — don't over-react to single data points |

---

## 8. Competitive Signal Recognition

Competitive signals are `signal_type = "competitive"`.

### 8.1 What to Capture

When any competitive signal appears, the agent should extract and record in `signal_detail`:

- **Who** is the alternative — named competitor, internal build, consultant recommendation, status quo
- **What** the prospect values about it — the specific capability or attribute (this is their Decision Criteria showing)
- **How far** they are — evaluating, piloting, contracted, considering
- **Who** introduced it — prospect's own research, consultant, another team, management directive

### 8.2 Competitive Threat Types

| Threat type | Recognition pattern | Tagging notes |
|---|---|---|
| Named competitor | Prospect mentions a specific company or product | `confidence = high`. Name the competitor in `signal_detail` |
| Build vs buy | Describing an internal development effort or considering one | Often the most dangerous. The "competitor" is the prospect's own engineering team |
| Consultant influence | Advisory firm recommending an approach or platform | Tag as `competitive` AND create a `stakeholder_signal` (the consultant is an `Influencer`) |
| Status quo / inertia | Resistance to changing current manual processes or existing tools | The incumbent "competitor" is doing nothing. Hard to beat because it costs nothing and requires no change management |
| Hyperscaler bundling | Cloud provider offering something similar as part of existing contract | Note the commercial dynamic — bundled vs best-of-breed argument |
| [Persona E]y / public sector alternative | Government body or regulator building centralised capability | Note timeline and scope — these usually move slowly but carry authority |

---

## 9. Champion & Stakeholder Signal Recognition

Champion signals are `signal_type = "champion_signal"`. Stakeholder signals are `signal_type = "stakeholder_signal"`.

### 9.1 Champion Recognition Reasoning

Don't look for a job title. Look for **behaviour**:

**Drives.** They schedule meetings, set agendas, create urgency, take action items. Test: if [YOUR COMPANY] stops pushing and the deal stops moving → no champion. If the deal moves even when [YOUR COMPANY] isn't pushing → strong champion.

**Shares.** They give [YOUR COMPANY] information about internal dynamics — who to talk to, what to avoid saying, how the org decides. Trust-based behaviour indicating genuine advocacy.

**Arms.** They ask for materials, case studies, data — ammunition to sell internally. This means they're actively building the case with other stakeholders.

**Fights.** They describe internal obstacles and ask for help overcoming them. They're encountering resistance and working to resolve it rather than surrendering to it.

**Champion strength is not binary.** Track it as a spectrum across calls via multiple `champion_signal` records. A champion strong on Call 1 but quiet on Call 3 is fading. A champion growing in activity across calls is strengthening. The cross-call assessment (Section 12) synthesises this.

### 9.2 Champion Health Indicators

| Indicator | Strong champion | Fading champion |
|---|---|---|
| Meeting behaviour | Schedules follow-ups themselves | Waits for [YOUR COMPANY] to initiate. Cancels/reschedules |
| Multi-threading | Brings new colleagues unprompted | Attends alone, doesn't introduce others |
| Information sharing | Shares internal context freely | Stops sharing, becomes guarded |
| Between-call engagement | Emails, responds quickly, sends materials proactively | Goes quiet between meetings |
| Delegation | Personally engaged on calls | Sends a junior instead |
| Internal advocacy | Asks for ammunition, describes internal selling efforts | Defers all decisions, says "I'll need to check with..." |

### 9.3 Stakeholder Mapping from Calls

Every person mentioned by name on a call — whether they're in the room or not — is a stakeholder signal:

Create a `stakeholder_signal` with:
- `attributed_to`: the email of the person who *mentioned* the stakeholder
- `signal_detail`: who was mentioned, what role they seem to play, how they relate to the decision
- `attributed_to_role`: the *speaker's* role. In `signal_detail`, describe the *mentioned person's* likely role

**People mentioned but not present are often the most important stakeholders.** "My VP will need to approve this" identifies the Economic Buyer. "Our InfoSec team lead is [Name]" identifies a process gatekeeper. These mentions go into `signal_detail` to build the stakeholder map over time.

---

## 10. Urgency & Compelling Event Recognition

Urgency signals are `signal_type = "urgency_signal"`.

### 10.1 Real Compelling Events

| Event type | Recognition pattern | `urgency` tag |
|---|---|---|
| [Persona E]y deadline | Specific regulation, specific date, specific requirement | `time_sensitive` |
| Board / executive mandate | Named directive from leadership with expectation or timeline | `time_sensitive` if dated, `current` if undated |
| Business initiative | Programme launch, product launch, event date requiring capability | `time_sensitive` |
| Contract cycle | Existing vendor contract ending, renewal window, switching opportunity | `time_sensitive` |
| Budget cycle | Fiscal year ending, budget allocation window, use-it-or-lose-it | `time_sensitive` |
| Competitive response | Peer organisation doing something that creates pressure to respond | `current` |
| Operational mandate | Directive to reduce timelines, cut costs, improve throughput by X% | `current` |

### 10.2 False Urgency Recognition

Not everything that sounds urgent is real:

| Pattern | Why it's false | How to tag |
|---|---|---|
| "We want to move quickly" with no specific date or consequence | Aspiration, not urgency. What happens if they don't? | Don't create an `urgency_signal`. May be `sentiment` at best |
| "This is a priority" but they cancel the next meeting | Words say urgent, actions say it's not | Create a `risk_signal` noting the contradiction |
| "We'd like something by year-end" (said in January) | Far enough to feel urgent without being urgent. Will slip | `urgency = developing` at best. Watch for reinforcement |
| [YOUR COMPANY]-created urgency (pricing deadline, limited offer) | Sales tactic, not buyer urgency. Doesn't indicate genuine compelling event | Don't record as an urgency signal. It's not prospect intelligence |

---

## 11. Pattern Detection and Promotion

The Call Analyst doesn't just extract individual signals — it detects patterns across calls and accounts and **promotes** them to `market_signals` when they cross a threshold. This is how call intelligence feeds the GTM Content Engine Intelligence Loop.

### 11.1 When to Promote

A pattern in `sales_call_signals` should be promoted to `market_signals` (with `source_type = "internal"`) when:

| Threshold | Example | What gets promoted |
|---|---|---|
| **Same pain domain from 3+ different accounts** | Three [target organisations] independently describe vendor evaluation pain in the same quarter | Market signal: recurring pain theme across tier-1 pipeline |
| **Same objection from 2+ deals** | Two prospects raise the same concern about [YOUR COMPANY] | Market signal: recurring objection — may indicate positioning gap or product gap |
| **Competitive pattern across accounts** | Multiple prospects mention the same competitor or build-vs-buy reasoning | Market signal: competitive threat pattern with common comparison points |
| **Stakeholder pattern across deals** | A specific role or department keeps appearing as a blocker | Market signal: structural sales barrier (e.g., [YOUR COMPLIANCE PROCESS] gatekeeping across enterprise deals) |
| **Urgency pattern** | Multiple accounts reference the same external driver | Market signal: external forcing function creating pipeline-wide urgency |

### 11.2 Promotion Record Structure

When promoting, the agent writes to `market_signals`:

```
market_signals (promoted from calls):
  - source_type = "internal"
  - signal_type = best-fit type for the pattern
  - relevant_icps = union of ICPs from the underlying call signals
  - urgency = highest urgency from the underlying signals
  - evidence_refs = array of sales_call_signals IDs forming the pattern
  - content (text) = the synthesised pattern description
```

Set the underlying `sales_call_signals` records to `status = "promoted"` and populate `promoted_to` with the new `market_signals` ID.

### 11.3 What NOT to Promote

- Single-account signals (one prospect's pain is not a market pattern)
- Internal [YOUR COMPANY] team dynamics
- Personal/relationship signals that are account-specific
- Anything with `confidence = "low"` unless corroborated by other signals
- Signals already promoted (don't double-promote)

---

## 12. Cross-Call Reasoning

The Call Analyst processes individual calls via [YOUR TRANSCRIPT PROVIDER] webhooks. But the real intelligence emerges across calls. When processing a new call for a deal/account that already has signals, the agent should build and update the signal picture.

### 12.1 Signal Accumulation Workflow

When processing a new call:

```
1. Query existing signals:
   SELECT * FROM sales_call_signals
   WHERE (deal_id = [this deal] OR account_id = [this account])
   AND status = 'active'
   ORDER BY meeting_date DESC

2. Compare new signals against existing:
   - Same pain_domain deepening?
     → Note progression in signal_detail. Consider severity upgrade
   - New pain_domain appearing?
     → New signal. Flag scope expansion in signal_detail
   - Buying signals escalating?
     → Note the progression pattern (information → intent → commitment)
   - Risk signals emerging where there were none?
     → Flag the change explicitly
   - Champion signals weakening?
     → Flag explicitly. This is always important
   - New stakeholder identified?
     → Cross-reference with existing stakeholder signals. Build the map

3. Detect contradictions with [YOUR CRM]:
   - Call shows Discovery conversation, [YOUR CRM] says SQL → stage is optimistic
   - Call shows pricing discussion, [YOUR CRM] MEDDIC has no Economic Buyer → CRM gap
   - Call shows champion fading, [YOUR CRM] deal stage hasn't changed → stale CRM

4. If 3+ calls with content exist for this deal, write deal health assessment
   (see 12.3)
```

### 12.2 Absence as a Signal

What's NOT discussed can be as important as what is. The agent should flag:

| What's missing | When to flag | What it means |
|---|---|---|
| No budget discussion across 3+ calls | Any stage past SAL | Economic Buyer isn't engaged. Budget hasn't been discussed internally |
| No new stakeholders after 2+ calls | SQL+ where multi-threading should be happening | Champion operating alone. Organisational buy-in may not exist |
| No timeline mentioned | Any stage past SAL | No compelling event. Deal will drift |
| No competitive mentions by SQL | By SQL, competition should be understood | Either no competition (unlikely) or prospect isn't sharing |
| No next steps at end of a call | After any call | No momentum generated. Flag immediately |
| No requirements or success criteria | Evaluating stage where criteria should be defined | Evaluation is vague, not structured |
| No procurement/security discussion | Technical DD or later | Process gates haven't been engaged. Timeline risk |

Record absence signals as `signal_type = "risk_signal"` with `confidence = "medium"` (absence could mean it happened off-call) and detail noting what's missing and why it's expected by this stage.

### 12.3 Deal Health Assessment

When sufficient signals exist (3+ calls with extractable content), the agent should write or update a deal health assessment. This goes into `signal_detail` of a summary `sentiment` signal or into a dedicated note format:

```
Deal health assessment:
  deal_id: [[YOUR CRM] deal ID]
  account: [Company name]
  assessment_date: [now]
  signal_count: [total active signals for this deal]
  call_count: [calls with content]
  
  Pain status: [deepening / stable / fading / absent]
    Domain(s): [which pain_domains are active]
    Severity trend: [improving / stable / declining]
  
  Buying momentum: [escalating / flat / declining]
    Latest strength: [commitment / intent / information / passive]
  
  Champion health: [strong / stable / fading / absent / unknown]
    Identified as: [name, email, role]
    Key behaviour: [what they're doing or not doing]
  
  MEDDICC coverage from calls:
    identify_pain: [covered / gap]
    champion: [covered / gap]  
    metrics: [covered / gap]
    economic_buyer: [covered / gap]
    decision_criteria: [covered / gap]
    decision_process: [covered / gap]
    competition: [covered / gap]
  
  Stage alignment:
    [YOUR CRM] says: [current stage]
    Call evidence supports: [stage that evidence actually supports]
    Alignment: [aligned / CRM lagging / CRM optimistic]
  
  Key risks: [top risk signals]
  Key gaps: [what intelligence is missing]
  
  Recommendation: [what should happen next based on the signal picture]
```

### 12.4 The Signal Story

For any deal, the accumulated signals should tell a coherent story. The agent should be able to answer:

1. **What is the pain?** (pain_domain + severity + deepening or not?)
2. **Who is the champion?** (attributed_to + role + strengthening or fading?)
3. **Who else is involved?** (stakeholder signals — right people for this stage?)
4. **What's the competition?** (named alternatives, build-vs-buy, status quo)
5. **What's the urgency?** (compelling event, deadline, budget cycle)
6. **What's blocking?** (risk signals, objections, missing progression)
7. **Does the CRM agree?** ([YOUR CRM] stage vs call evidence — contradictions?)

If the agent can't answer these questions, **the gaps themselves are the insight**: "4 calls with [Account] but Economic Buyer unidentified. No budget discussed. Despite SQL in [YOUR CRM], call evidence supports SAL at best."

---

## 13. Recording Gaps for Unrecorded Calls

Per the [YOUR TRANSCRIPT PROVIDER] Metadata Framework, enterprise calls frequently fail to record due to security policies (CAPTCHA, bot denial, bot kicked). The Call Analyst should still create a minimal signal record:

```
For meetings where notes_ready = false AND state = completed:
  signal_type = "sentiment"
  signal_summary = "Meeting occurred but was not recorded. [reason from processing_status]."
  signal_detail = "Attendees: [list]. Duration: [if available]. Meeting type: [type].
                   Recording failure reason: [disable_reason or processing_status]."
  attributed_to = [primary external attendee]
  confidence = "low"
  urgency = "background"
```

This ensures the engagement timeline is complete even when content is missing. The Contact Matcher and cross-call reasoning can use the timestamp, attendees, and meeting type data even without transcript content.

**Critical note from [YOUR TRANSCRIPT PROVIDER] framework:** Enterprise [target organisations] (tier-1 ICP accounts) have the *lowest* recording success rates. The most valuable call intelligence is systematically the hardest to capture. When [YOUR TRANSCRIPT PROVIDER] is sparse for an enterprise account, the agent should note this gap explicitly and signal that [YOUR CRM] activity data and manual notes become the primary intelligence sources for that account.

---

## 14. Tag Consistency Reference

Quick reference for all enum values used in this framework, aligned to the GTM Content Engine Tag Taxonomy:

**`signal_type`:** `pain_point`, `buying_signal`, `risk_signal`, `competitive`, `champion_signal`, `stakeholder_signal`, `urgency_signal`, `objection`, `requirement`, `sentiment`, `expansion`, `churn_risk`

**`attributed_to_role`:** `Champion`, `Economic_Buyer`, `[Persona_C]`, `[Persona_A]`, `Operator`, `Influencer`, `Unknown`

**`relevant_icps`:** `Impact Prover`, `[Friction Persona]`, `Governance Implementer`, `Data Foundation Fixer`, `Cost-Conscious Operator`, `ROI Defender`, `[Persona E]y Risk Holder`, `Decision Protector`, `Portfolio Optimiser`

**`meddicc_element`:** `metrics`, `economic_buyer`, `decision_criteria`, `decision_process`, `identify_pain`, `champion`, `competition`

**`urgency`:** `time_sensitive`, `current`, `developing`, `evergreen`, `background`

**`confidence`:** `high`, `medium`, `low`

**`severity`** (pain only): `critical`, `high`, `medium`, `low`, `polite`

**`pain_domain`:** `vendor_evaluation`, `innovation_programme`, `regulatory_compliance`, `infrastructure`, `procurement_onboarding`, `resource_constraint`, `stakeholder_alignment`, `budget_justification`, `competitive_pressure`, `technology_sprawl`

**`status`:** `active`, `archived`, `promoted`

**`source_type`** (on `market_signals` when promoted): `internal`
