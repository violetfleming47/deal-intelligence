# Questions Your Organisation Needs to Answer

Before this system works for you, you need to teach it how your business thinks about deals. That means answering specific questions — and your answers populate the frameworks automatically.

The quality of the system's output is directly proportional to how well you encode your team's judgment here.

---

## How this works

1. **You answer the questions below** — each section maps to specific framework sections
2. **Your answers populate the three frameworks** — each framework has `[YOUR ...]` markers showing exactly where your answers go
3. **The agents use the populated frameworks** — they load frameworks via SQL and reason against your content

You can populate the frameworks manually (copy answers into the marked sections) or use an agent in your harness: give it this document with your answers and the framework templates, and it will fill in every `[YOUR ...]` section automatically.

Each question below shows:
- **Why it matters** — what the system does with the answer
- **Feeds into** — which files and sections to update

Work through the sections in order. Earlier answers inform later ones.

---

## 1. Your Pipeline

These answers define how deals flow through your system — what stages exist, what each means, and what "done" looks like.

### What are your pipeline stages and what does each mean?

**Why it matters:** The readers need to know which stages to pull deals from. The assembler needs to know stage ordering for transition direction.

**Feeds into:** `guides/pipeline-stage-guide.md`, `guides/data-mapping-guide.md` (stage ID mapping), `skills/deal-tracker/` (stage filters), `frameworks/F02-deal-progression-signals.md` (stage transition signals)

### What CRM are you using and how do you access it?

**Why it matters:** The reader agents need to know your CRM's API patterns — how to search deals, pull contacts, fetch engagement history. The field names, query syntax, and pagination rules are all CRM-specific. You need to pull your CRM's API or MCP schema, identify the data nodes, and hardcode the correct field names into the reader skills.

**Feeds into:** `guides/data-mapping-guide.md` (field mapping tables), `skills/deal-tracker/` (deal field names), `skills/conversation-scanner/` (email/note tools), `guides/crm-data-reading-guide.md` (field trust rules)

### What qualifies a deal to enter your pipeline?

**Why it matters:** The readers need to know when to start capturing signals. Collecting too early wastes resources on noise. Collecting too late misses the activation signals that predict success.

**Feeds into:** `guides/pipeline-stage-guide.md` (collection start point), `frameworks/F02-deal-progression-signals.md` (activation signal definitions)

### What does "closed won" look like? What does "closed lost" look like?

**Why it matters:** Terminal states define the boundary of the deal timeline. Clear definitions help the assembler version history and the deal analyst's timeline reconstruction.

**Feeds into:** `guides/pipeline-stage-guide.md` (terminal states), `architecture/schema.md` (deal_state terminal versions)

---

## 2. Your Deals

These answers define what deal progression looks like in your specific business — what signals matter and what tells you a deal is real.

### What does forward motion look like in your deals?

**Why it matters:** This is the single most important question. The answer becomes the core of F02, which teaches the Conversation Reader what "progression" means. Without this, the reader has no basis for judging whether a deal is moving.

**Feeds into:** `frameworks/F02-deal-progression-signals.md` (all signal type definitions), `skills/conversation-scanner/` (classification taxonomy)

### What are the 3-5 signals that tell you a deal is real?

**Why it matters:** These become the high-confidence signal types that the reader agents prioritise. Not every signal is equal — some are noise, some are strong indicators of a real opportunity.

**Feeds into:** `frameworks/F02-deal-progression-signals.md` (confidence tier assignments)

---

## 3. Your People

These answers define the roles in your buying process — who matters, how to detect them, and what their engagement patterns mean.

### What roles exist in your buying process?

**Why it matters:** The system needs a role taxonomy to classify contacts against. Without defined roles, it can't distinguish a primary sponsor from a process owner from an opposing stakeholder. The Assembler uses these roles to build `titles_state`.

**Feeds into:** `frameworks/F03-champion-and-stakeholder-roles.md` (role definitions), `skills/state-builder/` (titles_state structure)

> **Note:** If your team uses a named methodology (e.g., MEDDIC, BANT, etc.), map those role names to whatever labels you define. The system's F03 taxonomy is framework-agnostic — your methodology's role names become aliases.

### How do you detect who's playing which role?

**Why it matters:** CRM titles don't reliably indicate buying roles. The same title might be a primary sponsor in one deal and a process owner in another. The detection framework needs behavioural indicators, not just titles.

**Feeds into:** `frameworks/F03-champion-and-stakeholder-roles.md` (role indicators)

### What engagement patterns distinguish real sponsors from polite attendees?

**Why it matters:** Not everyone in a meeting is a stakeholder. The system needs to distinguish between people who attend because they were invited and people who attend because they're invested.

**Feeds into:** `frameworks/F03-champion-and-stakeholder-roles.md` (evidence thresholds)

---

## 4. Your Friction

These answers define what blocks deals in your market — the objections, the process hurdles, and the patterns that predict stalling.

### What objections do you hear repeatedly?

**Why it matters:** The Friction Reader needs a taxonomy of known objection patterns. When it encounters an objection in a transcript or note, it needs to classify it — is this a pricing objection, a security concern, a "not the right time" deflection?

**Feeds into:** `frameworks/F07-objection-friction-patterns.md` (objection taxonomy and classification rules), `skills/blocker-scanner/` (friction type labels)

### What stalls deals in your market?

**Why it matters:** Stalling is different from objection. An objection is voiced. A stall is silence — meetings stop getting scheduled, emails stop getting answered. The framework needs to know what stalling looks like so the friction reader can detect it early.

**Feeds into:** `frameworks/F07-objection-friction-patterns.md` (stalling patterns)

### What's the difference between a blocker and a buying signal in disguise?

**Why it matters:** Some objections are actually positive signals. "This is too complex for our team" might mean "we need professional services" (expansion opportunity). The framework needs to distinguish genuine blockers from signals that look negative but indicate engagement.

**Feeds into:** `frameworks/F07-objection-friction-patterns.md` (boost patterns — friction that's actually positive)

---

## 5. Your Transcripts

These answers define how you access meeting recordings and transcripts.

### Where do your meeting transcripts live?

**Why it matters:** The Conversation Reader and Friction Reader both pull transcript text. They need to know your provider's endpoint, auth, and response format.

**Feeds into:** `guides/data-mapping-guide.md` (transcript tool mapping), `skills/conversation-scanner/` (transcript retrieval), `skills/blocker-scanner/` (transcript retrieval)

**Example:** "We use Gong. Transcripts are accessible via Gong's API with OAuth. Each transcript has speaker attribution, timestamps, and CRM deal associations."

### Do your transcripts have speaker attribution?

**Why it matters:** Speaker-attributed transcripts are dramatically more valuable than plain text. The system can link statements to specific stakeholders, track language shifts per person, and attribute signals to the right contact. Without attribution, everything is "someone in the meeting said..."

**Feeds into:** `skills/conversation-scanner/` (speaker attribution in raw signals), `skills/blocker-scanner/` (contact attribution on friction signals)

---

## 6. Your Data

These answers define how you interact with your CRM data — what to trust, what's unreliable, and where the quiet hallucinations hide.

### What CRM fields do you trust? Which ones are stale?

**Why it matters:** Not all CRM data is equal. Some fields are reliably updated (deal stage, amount). Others are filled once and never touched again. The data reading guide needs to know which fields to trust at face value and which to cross-reference.

**Feeds into:** `guides/crm-data-reading-guide.md` (field trust rules, freshness checks)

### How do you distinguish empty from missing in your CRM?

**Why it matters:** This is the single biggest source of operational hallucination. A null primary sponsor field could mean: nobody is a primary sponsor, the AE forgot to log it, or the field wasn't relevant at this stage. Without explicit rules, the agent treats every empty field as a "gap" — and produces false risk signals everywhere.

**Feeds into:** `guides/crm-data-reading-guide.md` (empty vs missing rules)

### What's your database setup?

**Why it matters:** The pipeline writes to multiple tables — raw signals, classifications, deal state, scratchpad, lifecycle, traceability. You need a Postgres-compatible database with JSONB support.

**Feeds into:** `architecture/schema.md` (table creation), `guides/data-mapping-guide.md` (database placeholders), all reader and assembler skills (write targets)

---

## What to do with your answers

### Option A: Auto-populate with an agent

Give your answers and the framework templates to an agent in your harness:

```
"Here are my answers to QUESTIONS.md: [paste answers]. 
Populate every [YOUR ...] section in the frameworks/ directory with the corresponding answers. 
Preserve the framework structure. Only fill in the marked sections."
```

The agent will fill in F02, F03, and F07 based on your answers. Review the output — the frameworks are the reasoning engine, so they need to be accurate.

### Option B: Populate manually

Each framework has `[YOUR ...]` markers that correspond to your answers. The mapping:

| Your answer | Goes into | Section marker |
|-------------|----------|----------------|
| Pipeline stages | F02, pipeline-stage-guide | `[YOUR STAGES]` |
| Deal progression signals | F02 | `[YOUR SIGNAL TYPES]`, `[YOUR COMPOUND PATTERNS]` |
| Buying roles | F03 | `[YOUR SPONSOR SPECIFICS]`, `[YOUR ASSESSOR SPECIFICS]`, etc. |
| Friction / objections | F07 | `[YOUR OBJECTION TYPES]`, `[YOUR FRICTION SPECIFICS]` |
| CRM field mapping | data-mapping-guide | `{{PLACEHOLDER}}` values |

### After populating frameworks

1. **Map your data sources** (`guides/data-mapping-guide.md`) — pull your CRM and transcript provider schemas, map every `{{PLACEHOLDER}}` value, hardcode them into the skills.
2. **Fill in the pipeline stage guide** (`guides/pipeline-stage-guide.md`) — your stages are the skeleton everything hangs on.
3. **Configure the data reading guide** (`guides/crm-data-reading-guide.md`) — the anti-hallucination rules are universal, just add your CRM-specific field trust rules.
4. **Set up the database** (`architecture/schema.md`) — run the table definitions against your Postgres instance.
5. **Deploy and run** — load each skill as a system prompt, run readers daily, assembler after them, deal analyst on demand.

The frameworks will be incomplete at first. That's expected. The `[YOUR ...]` markers that you can't fill yet stay as prompts — they're templates for the reasoning the system will apply to your data.

---

## The full system — 7 additional frameworks

The full Deal Intelligence system adds 7 more frameworks to the 3 in this starter:

| Framework | What it adds |
|-----------|-------------|
| F01 — Baseline ICP | Who your current ICPs are — generated from your CRM data |
| F04 — Engagement Intent Weighting | How to detect roles and signals from data patterns |
| F05 — Org Structure Patterns | Where functions sit in different org types |
| F06 — Win/Loss Patterns | What was present in wins versus losses |
| F08 — Velocity Benchmarks | What normal deal speed looks like, calibrated from your data |
| F09 — Industry Context | What's happening in your target verticals |
| F10 — Positioning & Product | What your product does and why each persona cares |

These frameworks power the pattern detection agents and the intelligence layer. They require additional questions about your market, your ICP, and your product — questions that the starter system doesn't need because it operates at the individual deal level. As your deal data accumulates, these frameworks become the reasoning engine for cross-deal intelligence.
