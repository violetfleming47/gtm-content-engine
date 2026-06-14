# Platform Algorithm Framework

How each platform's algorithm works, what signals it rewards, and what rules to follow when publishing. This framework is **expandable** — it ships with LinkedIn as a worked example. Add sections for any platform you publish on.

Your answers in QUESTIONS.md (Section 7: Your Distribution Channels) determine which platforms you need to document here.

**Consumed by:** Format Selector (format and timing decisions), Prompt Assembler (platform-aware briefs), Ghost Writer (structural rules), Distributor (posting strategy).

**Stored in:** `algorithm_knowledge` (see frameworks/platform-algorithm.md) table. Each row is a platform + rule.

---

## How to Use This Framework

For each platform you publish on:

1. **Document the ranking pipeline** — how does the platform decide what to show? What stages does content pass through?
2. **Document the signals** — what does the algorithm measure? What's weighted heavily?
3. **Document the dos and don'ts** — practical rules for content, format, timing, and engagement
4. **Keep it current** — platform algorithms change. Review quarterly.

The Format Selector reads this when selecting format and timing. The Prompt Assembler reads it when writing briefs. The Ghost Writer reads format-specific rules.

---

## Adding a New Platform

Copy this template for each platform you add:

```
## [Platform Name]

### Ranking Pipeline
How the platform decides what to distribute.

### Key Signals
What the algorithm measures and how each signal is weighted.

### Content Rules (Do / Don't)
### Format Rules (Do / Don't)
### Timing and Cadence (Do / Don't)
### Engagement Rules (Do / Don't)
### Authority Building (Do / Don't)
### Personal vs Company/Brand Profile
```

---

## LinkedIn

### Ranking Pipeline

Every post passes through five stages. A post must clear each stage to reach the next.

**Stage 1: Quality Filter.** LinkedIn classifies the post on submission — spam, low-quality, or clear. Spam is suppressed. Low-quality gets limited distribution. Clear moves to Stage 2.

**Stage 2: Small Audience Test (Golden Hour).** The post is shown to a small subset of first-degree connections. The first 30–60 minutes are the evaluation window. LinkedIn monitors engagement velocity. Strong early signals trigger Stage 3. Weak signals cap distribution here.

**Stage 3: Expanded Distribution.** Posts that clear the golden hour are distributed to second-degree connections, followers, and topic-interest groups based on relevance matching.

**Stage 4: Ongoing Re-Ranking.** Distribution is continuous. A post that receives a second wave of engagement can be redistributed. Posts that stall taper off. LinkedIn surfaces posts up to 2–3 weeks old if they remain relevant.

**Stage 5: Platform Balancing.** A final layer balances individual relevance with platform-level objectives — content diversity, creator equity, professional community health.

### Key Signals

| Signal | What It Measures | Weight |
|--------|-----------------|--------|
| **Dwell Time** | Seconds spent reading before scrolling | Primary quality signal. 61+ seconds = ~15.6% engagement rate. Under 3 seconds = ~1.2%. |
| **"See More" Expansion** | Whether user clicks to reveal the full post | Binary gate on your hook. No expansion = dwell time stays near zero. |
| **Comments** | Text responses from other users | ~15x more weight than likes. Quality matters — 5+ words with author reply is highest value. Industry expert comments carry 5-7x more weight. Threads with multiple participants generate ~5.2x more amplification. |
| **Saves / Bookmarks** | User saves the post for later | High-signal due to rarity. Strong evidence of content quality and utility. |
| **Shares / Reposts** | User distributes to their own network | Stronger than reactions. Instant reposts extend reach by 5–10% over two days. |
| **Reactions** | Like, Celebrate, Support, Love, Insightful, Funny | Lowest-friction signal. Insightful and Celebrate carry slightly more weight than Like. |
| **Profile Completeness** | How complete the author's profile is | Affects baseline distribution floor. Incomplete profiles start lower. |
| **Posting Consistency** | Regular vs sporadic posting pattern | Regular accounts receive higher baseline distribution. Long gaps erode this signal. |
| **Topic Authority** | Sustained posting on specific topics | Builds over time. Consistent topic focus = algorithm learns what you're about and shows your content to relevant audiences. |
| **Network Quality** | Who engages with you | Engagement from high-authority profiles amplifies more than engagement from low-activity accounts. |
| **Creator Mode** | Whether creator mode is enabled | Changes profile layout, enables follow, surfaces content to non-connections. |

### Content Rules

**Do:**
- Open with a hook in the first two lines — data point, provocative question, or contrarian statement
- Write at a 6–9 year reading age — conversational, short sentences, no jargon
- Use short paragraphs — one to two sentences each, line breaks between
- Keep captions between 700–900 characters for text + image posts
- Deliver on the hook — the body must pay off what the opening promised
- End with a genuine open-ended question that invites informed answers
- Include specific details — named examples, concrete timelines, quantified outcomes
- Include personal experience — anecdotes, lessons, decisions, mistakes

**Don't:**
- Use engagement bait — "Comment YES," "Like for option A," "Tag someone who needs this"
- Write generic content — broad motivational advice underperforms niche insights
- Produce content that reads as AI-generated without original perspective
- Use clickbait hooks that don't deliver
- Use dense jargon-heavy language without inline definitions

### Format Rules

**Do:**
- Use document posts (PDF carousels) for structured content — frameworks, step-by-step, data
- Use native video for perspectives and quick thought leadership — vertical, under 60 seconds, captioned
- Use text + image for event reflections, takeaways — images of real people, portrait orientation
- Use text-only sparingly for high-impact thought leadership
- Upload video directly to LinkedIn — native only

**Don't:**
- Use single standalone images as the primary format
- Share YouTube links or external video
- Rely on external links as the main content — the post must stand alone without the click
- Use more than 1–3 hashtags

### Timing and Cadence

**Do:**
- One structured post per day as baseline
- Add reactive posts when market events warrant
- Post on Tuesday and Thursday for highest engagement
- Post during peak windows: 7–8 AM, 10–11 AM, 12–2 PM, or 4–6 PM (audience local time)
- Be available to engage during the first 60 minutes after posting

**Don't:**
- Post multiple planned posts in one day
- Post inconsistently — irregular patterns are worse than a steady lower cadence
- Post when you can't engage — a post published when the author is asleep misses the golden hour

### Engagement Rules

**Do:**
- Engage with 5–10 posts from others before publishing your own — pre-post warm-up
- Respond to comments within the first 30 minutes — substantive replies, not just "thanks"
- Engage with 5+ other posts after publishing
- Reactivate your post at the 8-hour and 24-hour marks
- Prioritise comments from relevant industry connections

**Don't:**
- Skip pre-post engagement — publishing cold reduces baseline distribution
- Use engagement pods — LinkedIn detects coordinated rings and suppresses all members
- Tag more than 5 people per post
- Self-comment without responding to others — monologue triggers a reach penalty

### Personal vs Company Profile

Personal profiles outperform company pages for organic reach. Employee advocacy generates more reach than the same content from the company page. Company pages benefit from paid amplification and employee reshares.

---

## [YOUR NEXT PLATFORM — e.g. X/Twitter]

*Copy the template above. Document the ranking pipeline, key signals, and rules for this platform. Common platforms to add: X/Twitter, YouTube, TikTok, Newsletter platforms, Reddit, industry-specific forums.*

### Ranking Pipeline

[YOUR DESCRIPTION — how does this platform decide what to show?]

### Key Signals

[YOUR SIGNALS — what does the algorithm measure?]

### Content Rules / Format Rules / Timing / Engagement

[YOUR RULES]
