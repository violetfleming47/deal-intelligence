# F07 - Objection & Friction Patterns

**Cluster:** deal-analysis
**Purpose:** Define what stalls and kills deals mid-process. What resistance looks like, where it comes from, what it means. This is about the forces that slow or stop deals while they're alive.
**Boundary:** This framework answers "what gets in the way and why?" It never describes terminal outcomes or what wins vs losses had in common (F06). It never defines timing thresholds or what normal speed looks like (F08). It never defines stakeholder roles (F03) or how to detect them (F04). When it references roles, it uses F03 definitions. When it references deal signals, it uses F02 definitions.
**Consumed by:** blocker-scanner, deal-briefer, state-builder (full system: pattern-friction)
**Loading depth:**
- Vocabulary: friction category names and one-line definitions only
- Reasoning: full friction descriptions with stage context and escalation paths
- Full: everything including compound friction, persona correlation, and resolution patterns
**Populated from:** QUESTIONS.md § "Your Friction" → "What objections do you hear repeatedly?" and "What stalls deals in your market?"

---

## Why This Framework Exists

The ICP agent needs to understand why deals slow down and what resistance looks like — because a new persona type might be the one who introduces friction, or the one who resolves it. This framework defines the categories of friction so the agent can assess whether a persona's presence correlates with deals hitting resistance or breaking through it.

---

## How to Read This in Your Data

Friction and objections live in unstructured text — transcripts, emails, and meeting notes. Detection is pattern-matching against language, speaker context, and timing.

**From transcript and email content — friction keyword patterns:**
- **Process friction:** "vendor assessment process," "security questionnaire," "procurement needs to review," "we need to go through our TPRM," "third-party risk," "our compliance team requires," "waiting on legal"
- **Technical friction:** "we also need it to handle," "what about integration with," "our architecture requires," "can it work with [existing system]," "we'd need a POC for that," "scope is expanding"
- **Political friction:** harder to detect directly — look for signals like "we need to align internally," "there are different views on this," "the other team is looking at," "I need to socialise this," "not everyone is on board yet"
- **Commercial friction:** "what's the pricing model," "that's above our budget," "we need to build a business case," "ROI justification," "can you do better on price," "our budget cycle is"
- **Security friction:** "our InfoSec team needs to review," "we have a vendor assessment process," "TPRM requirements," "data residency," "where is data hosted," "penetration testing," "SOC 2," "ISO 27001"

**From speaker context:**
- Who said it matters as much as what was said. The same phrase from a champion vs a gatekeeper means different things
- Map the speaker to their F03 role. Friction raised by a gatekeeper is structural. The same concern raised by a champion may be them pre-empting a gate they know is coming
- If a person who hasn't spoken before raises a concern in a late-stage meeting, that's a new stakeholder introducing friction — check if they're a blocker (F03)

**From timing relative to stage:**
- Process and security friction at Stage 2-3 = prospect using process to slow down (possible soft no). Same friction at Stage 5-6 = normal procurement gate
- Technical friction at Stage 4+ = something was missed in evaluation scoping
- Commercial friction before Stage 4 = either strong buying signal (serious enough to discuss price) or early disqualification attempt

**From `di_deal_state` — friction indicators:**
- Stage regression (deal moves backward) = unresolved friction forced a reset
- Extended time at a single stage beyond F08 benchmarks = friction is present but may not be named in any text. Cross-reference with transcript content from that period
- Declining cadence + no stage movement = friction or inertia — check last 3 interactions for objection language

---

## Friction vs Objection

These are different things. Both slow deals. They need different responses and they mean different things for the ICP agent.

**Friction** is structural resistance — the deal encounters a process, a requirement, a dependency, or an absence that slows it down. Friction isn't anyone being against the deal. It's the deal hitting something hard.

**Objection** is human resistance — a person raises a concern, expresses doubt, or pushes back on the value, risk, fit, or timing of the deal. Objections come from people. Friction comes from systems and structures.

The distinction matters because friction and objections have different resolution paths, different persona correlations, and different implications for the ICP framework.

---

## Friction Categories

[NOTE: The friction categories below are common in B2B sales. Your market may have additional categories or different names. Add your specific objection types and friction patterns from QUESTIONS.md answers. The keyword detection patterns in "How to Read This in Your Data" should be updated with your specific language.]

Structural resistance that slows deals. Not about anyone being opposed — about the deal hitting something.

---

### 1. Process Friction

The buying organisation's internal process creates delay.

**What this looks like:**
- Procurement requirements that take weeks to complete (security questionnaires, vendor assessments, compliance checklists)
- Sequential gates — one must complete before the next starts, no parallelism
- Requirements that appear mid-deal that nobody mentioned upfront
- Internal approval chains where each step waits for the previous
- Budget cycle misalignment — the deal is ready but the budget window isn't

**Where it typically appears:** Stage 3-5. Process friction is a late-deal phenomenon. If it appears earlier, the prospect may be using process to slow things without saying no.

**What process friction means for the ICP agent:**
A persona type that navigates process friction effectively — because they know the internal process, they anticipate requirements, they pre-clear gates — is a high-value ICP signal. The gatekeeper role (F03) is defined by process. The question is whether a new persona type either smooths the process or introduces additional process.

---

### 2. Technical Friction

The evaluation or integration surface creates delay.

**What this looks like:**
- Technical requirements that weren't scoped upfront — "we also need it to do X"
- Integration complexity with the prospect's existing stack
- Security or data handling requirements that need custom work
- Evaluation scope expanding — sandbox becomes POC becomes pilot
- Technical stakeholders who need convincing individually, not as a group

**Where it typically appears:** Stage 2-3. Technical friction belongs in evaluation. If it appears at Stage 4+, something was missed earlier.

**What technical friction means for the ICP agent:**
Technical friction is normal and expected. It becomes a problem when it's unbounded — no clear evaluation criteria, no defined scope, no timeline. A persona type that bounds technical evaluation (defines criteria, sets timelines, makes recommendations) reduces friction. A persona type that expands evaluation scope without bounding it increases friction.

---

### 3. Political Friction

Internal organisational dynamics create delay.

**What this looks like:**
- Competing internal priorities — {{COMPANY_NAME}} is one of several initiatives fighting for attention
- Stakeholder misalignment — different people want different things from the evaluation
- Territory concerns — the innovation team chose {{COMPANY_NAME}} but the technology team wasn't consulted
- Leadership change mid-deal — new person questions existing commitments
- A prior failed vendor relationship creating institutional caution about new vendors

**Where it typically appears:** Any stage, but most damaging at Stage 3-4 where organisational commitment is required and internal politics surface.

**What political friction means for the ICP agent:**
Political friction is the hardest to detect from data — it shows up as stalling (F02) without obvious cause. A persona type that spans organisational boundaries (cross-functional role, senior enough to align competing interests) may be a political friction reducer. This is one of the champion's key functions (F03) — bridging lines of business.

---

### 4. Commercial Friction

Pricing, terms, or commercial structure creates delay.

**What this looks like:**
- Price sensitivity — the prospect pushes back on pricing or asks for discounts early
- Commercial model misalignment — the prospect expects one pricing structure, {{COMPANY_NAME}} offers another
- Contract terms negotiation — legal redlines that go back and forth
- Competing budget allocation — the money exists but it's allocated elsewhere
- ROI justification required — the prospect needs a business case before they can commit budget

**Where it typically appears:** Stage 4-5. Commercial friction is a late-stage phenomenon. If pricing comes up at Stage 2, it's either a strong buying signal (they're serious enough to discuss money) or a qualifying question (checking if {{COMPANY_NAME}} is in range before investing time).

**What commercial friction means for the ICP agent:**
The persona who controls or influences budget allocation is the one who resolves commercial friction. If a new persona type correlates with commercial conversations advancing (not just starting), they may have economic buyer proximity.

---

### 5. Timing Friction

External timing creates delay — nothing about the deal itself is wrong, but the moment isn't right.

**What this looks like:**
- Budget cycle misalignment — "we can look at this in Q3 when new budget opens"
- Strategic planning cycle — "we're in planning right now, can't commit to new vendors"
- Regulatory waiting — "we need to see what the regulator says before we act"
- Organisational change — "we're restructuring, nobody knows who owns this yet"
- Seasonal patterns — certain months or quarters when buying organisations go quiet

**Where it typically appears:** Any stage. Timing friction can pause a deal at any point. The distinguishing feature: the deal doesn't regress, it freezes.

**What timing friction means for the ICP agent:**
Timing friction is context-dependent, not persona-dependent. The ICP agent should not draw persona conclusions from timing friction unless a specific persona type consistently creates urgency that overrides timing constraints (mobiliser behaviour per F03).

---

## Objection Categories

Human resistance — a person raises a concern or pushes back. Objections have a source (who raised it) and a nature (what it's about).

---

### 1. Value Objection

"I don't see why we need this."

**What this sounds like:**
- "What problem does this solve that we can't solve ourselves?"
- "We already have a process for vendor evaluation"
- "I'm not sure the ROI is there"
- "What's the cost of doing nothing?"

**Who typically raises it:** Economic buyers and senior stakeholders who haven't been part of the evaluation. They're seeing the ask (budget, time, change) without the context the evaluation team has.

**What it means for the ICP agent:**
Value objections from economic buyers are normal — they should ask "why." The question is whether the champion (F03) can translate evaluation outcomes into business value for the objector. A new persona type that bridges technical evaluation to business case is a high-value ICP signal.

---

### 2. Risk Objection

"What if this goes wrong?"

**What this sounds like:**
- "What happens to our data?"
- "How do we know this will scale?"
- "We tried something like this before and it didn't work"
- "What's the rollback plan?"
- "Who's responsible if this fails?"

**Who typically raises it:** Technical evaluators, security teams, compliance teams, and blocker personas (F03). Risk objections are legitimate and often the most important ones to resolve well.

**What it means for the ICP agent:**
Risk objections that get resolved cleanly are a progression signal (the deal passed a gate). Risk objections that repeat across meetings without resolution are a stalling signal. The persona who resolves risk objections — by providing evidence, by owning the risk, by bounding the exposure — is valuable.

---

### 3. Competitive Objection

"Why not [competitor]?"

**What this sounds like:**
- "We're also looking at X"
- "X offers this and you don't"
- "X is cheaper / bigger / more established"
- "Our consultant recommended X"

**Who typically raises it:** Anyone, but most impactful from technical evaluators (who compare features) and coaches (who relay competitive intel).

**What it means for the ICP agent:**
Competitive objections are information, not threats. They tell you who else is in the deal and what the prospect values. A persona type that introduces competitive framing may be testing {{COMPANY_NAME}}'s positioning or may have a genuine preference for the competitor. The response reveals the deal dynamics.

---

### 4. Authority Objection

"I can't make this decision."

**What this sounds like:**
- "I need to take this upstairs"
- "This isn't my budget"
- "I'd need to get X's approval"
- "The board would need to see this"

**Who typically raises it:** Champions and technical evaluators who've reached the limit of their authority. This isn't resistance — it's a boundary signal.

**What it means for the ICP agent:**
Authority objections are actually deal progression signals (commitment per F02) disguised as friction. "I need to take this to the board" means they want to proceed but can't alone. The objection reveals the economic buyer path. A persona type that raises authority objections and then follows through (actually takes it upstairs, actually gets the meeting) is demonstrating mobiliser behaviour (F03).

---

### 5. Inertia Objection

"Not now."

**What this sounds like:**
- "Let's revisit next quarter"
- "We're not ready for this yet"
- "We have other priorities right now"
- "Can you check back in six months?"

**Who typically raises it:** Anyone, but most commonly from contacts who were never fully bought in. Different from timing friction — timing friction is about external constraints. Inertia objection is about internal willingness.

**What it means for the ICP agent:**
Inertia is the most common deal-killer and the hardest to counter because it's not about {{COMPANY_NAME}} — it's about the prospect's readiness to change. A persona type that creates urgency against inertia (mobiliser per F03) or that reframes the cost of waiting is a high-value ICP signal.

---

## Friction Escalation

Friction becomes terminal when it compounds or when it's left unresolved.

**Single friction → manageable:** One procurement requirement, one technical question, one stakeholder concern. Normal deal process.

**Compounding friction → dangerous:** Process friction + political friction simultaneously. The procurement team is slow AND internal stakeholders are misaligned. Each friction source makes the other harder to resolve.

**Unresolved friction → terminal:** Any single friction that sits unresolved beyond the stage benchmark (F08) eventually kills the deal. Not dramatically — the deal just goes quiet. The prospect stops responding because the friction was never cleared and they've moved on.

**Escalation indicator:** When the same friction is referenced in three or more consecutive interactions without progress toward resolution, it has escalated from friction to blocker. The deal will not move until it's resolved or the friction source is removed.

[YOUR ESCALATION THRESHOLDS: Calibrate from your deal data — how many days of unresolved friction correlates with eventual loss in your pipeline? Does this vary by friction category? Fill from QUESTIONS.md § "Your Friction" answers and refine after seeding 20+ closed deals.]

---

## Friction Resolution Patterns

When friction is resolved, how does it happen?

[YOUR RESOLUTION PATTERNS: Fill from your deal history. Structure:
- Which roles (F03) most commonly resolve which friction categories in your deals
- Whether resolution correlates with specific deal signals (F02) — e.g., does resolving technical friction produce an acceleration signal?
- Whether certain persona types consistently appear before friction resolution events
- Average time to resolution by friction category

Fill from QUESTIONS.md § "Your Friction" → "What typically unblocks a stalled deal?" and calibrate from deal data after seeding.]
