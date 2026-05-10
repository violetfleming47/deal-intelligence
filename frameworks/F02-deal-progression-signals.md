# F02 - Deal Progression Signals

**Cluster:** deal-analysis
**Purpose:** Define what deal-level progression looks like - what moves a deal forward, what stalls it, what restarts it. This is about the deal, not the people in it.
**Boundary:** This framework answers "what does forward motion look like and what causes it?" It never defines stakeholder roles (F03) or describes how to detect roles from data (F04). When it references people, it references their role as defined in F03, not their title or identity.
**Consumed by:** conversation-scanner, state-builder, deal-briefer (full system: pattern-conversation)
**Loading depth:**
- Vocabulary: signal type names and one-line definitions only
- Reasoning: full signal descriptions with stage context
- Full: everything including signal interactions and compound patterns
**Populated from:** QUESTIONS.md § "Your Deals" → "What does forward motion look like?" and "What are the 3-5 signals that tell you a deal is real?"

---

## Why This Framework Exists

The ICP agent needs to understand what "this persona moved the deal forward" means in concrete terms. This framework defines the deal-level signals that indicate progression, stalling, and restart - so that when a new persona pattern is detected, the agent can assess whether their presence correlates with deals actually moving.

---

## Five Progression Signal Types

[NOTE: The five signal types below are common in B2B sales. Your business may define progression differently. Add, remove, or rename signal types to match how your team thinks about deal movement. The structure (what it looks like → what it means → stage relevance) should stay the same.]

These are deal-level events, not people-level behaviours. Each signal type describes something that happens to the deal. F04 covers how to detect these signals in data.

---

### 1. Activation

The deal goes from passive to active. Something shifts it from "exists in pipeline" to "someone is working on this."

**What activation looks like at the deal level:**
- First high-intent action on a deal that was previously low-activity (demo request, pricing enquiry, evaluation request)
- A deal that existed as an MQL or SAL suddenly generates real engagement
- The transition from "{{COMPANY_NAME}} is reaching out" to "the prospect is reaching in"

**What activation means for the ICP agent:**
When a new persona type appears and deals activate shortly after, that persona may be an activation trigger. The question: are deals that include this persona activating faster or more reliably than deals without them?

**Stage relevance:** Primarily Stage 1-2. Activation is the earliest signal.

---

### 2. Multiplication

The deal expands from single-threaded to multi-threaded. More people from the buying organisation get involved.

**What multiplication looks like at the deal level:**
- New contacts added to the deal within a short window (14-30 days)
- New contacts come from different functions or departments, not just the same team
- Meeting attendee lists grow - more people in the room
- Email threads widen - more CCs, more participants

**What multiplication means for the ICP agent:**
Deals that multiply are healthier than deals that stay single-threaded. If a new persona type consistently appears before multiplication events, their presence may correlate with deals expanding. The correlation matters, not the assumption of causation.

**Stage relevance:** Stage 2-3 primarily. Multiplication should happen before deep evaluation - if the deal is still single-threaded at Stage 3+, that's a risk.

---

### 3. Stage Transition

The deal moves from one pipeline stage to the next. The most concrete progression signal.

**What stage transition looks like at the deal level:**
- CRM deal stage changes (SAL to SQL, SQL to evaluation, evaluation to proposal, proposal to contract)
- Each transition implies something was satisfied - a qualification threshold, a technical gate, a commercial alignment
- The velocity of transitions matters: fast transitions signal momentum, slow transitions signal friction

**What stage transition means for the ICP agent:**
The critical question: who was active in the 7-14 days before a stage transition? If a new persona type consistently appears in that pre-transition window, their presence may be a prerequisite for deals moving forward.

**Stage relevance:** All stages. But the later the stage, the more significant the transition.

---

### 4. Commitment

The deal generates evidence that the buying organisation is committing resources, attention, or reputation to the evaluation.

**What commitment looks like at the deal level:**
- The prospect allocates internal resources (people, time, environments) to the evaluation
- Internal stakeholders are briefed or brought in by the prospect, not by {{COMPANY_NAME}}
- The prospect references {{COMPANY_NAME}} in internal planning ("we're evaluating {{COMPANY_NAME}} for Q3 rollout")
- Commercial conversations begin - pricing discussions, contract terms, procurement process initiated
- The prospect shares information they wouldn't share with a vendor they're not serious about (internal architecture, budget parameters, competitive landscape)

**What commitment means for the ICP agent:**
Commitment signals separate real deals from exploratory ones. If a new persona type's presence correlates with commitment signals appearing, that persona may be the one who converts interest into organisational commitment.

**Stage relevance:** Stage 3-5. Commitment is a mid-to-late signal. Early commitment is rare and very strong.

---

### 5. Acceleration

The deal speeds up. Time-in-stage decreases, cadence increases, things that were slow become fast.

**What acceleration looks like at the deal level:**
- Shorter gaps between meetings and emails
- Faster response times from the prospect
- Multiple actions happening in parallel (technical evaluation + commercial discussion + stakeholder alignment)
- Deadlines set by the prospect, not by {{COMPANY_NAME}}
- External drivers creating urgency (regulatory deadlines, board commitments, competitive pressure, budget cycles)

**What acceleration means for the ICP agent:**
When a deal that was slow suddenly accelerates, something changed - a new contact, an external event, or a shift in internal priorities. If a new persona type consistently correlates with acceleration events across multiple deals, their presence may be a factor in deal velocity.

**Stage relevance:** Can happen at any stage but most impactful at Stage 3-4 where deals often stall.

---

## Negative Signals (Regression and Stalling)

Equally important to progression signals are the signals that a deal is going backwards or stuck.

### Stalling

The deal stops moving. No stage change, declining engagement, increasing time-in-stage.

**What stalling looks like:**
- No stage change beyond the benchmark for that stage
- Declining email and meeting frequency
- Prospect response times lengthening
- Meetings being rescheduled or cancelled
- Actions agreed in meetings not completed between meetings

### Regression

The deal moves backwards. A stage change in the wrong direction, or conditions that were met become unmet.

**What regression looks like:**
- Deal stage moved back in CRM
- A stakeholder who was engaged withdraws
- Requirements that were agreed are reopened
- A competitor enters the evaluation that wasn't previously present
- The champion's language shifts from future-tense ("when we implement") to conditional ("if we decide to")

### Restart

A stalled deal comes back to life. Something changed.

**What restart looks like:**
- Engagement resumes after a quiet period (30+ days of silence followed by activity)
- A new contact appears on a previously stalled deal
- An external event creates new urgency (regulatory change, leadership change, competitive move)
- The deal stage moves forward after being static

**What restart means for the ICP agent:**
Restarts are high-value signals. The person or event that preceded the restart tells you what unsticks deals. If a new persona type is consistently the person who appears before restarts, they may be the missing piece in stalled deals.

---

## Signal Compound Patterns

Individual signals are useful. Combinations are more telling.

**Activation + Multiplication:** A deal that activates and immediately multi-threads is a strong deal. Something triggered both urgency and organisational buy-in simultaneously.

**Multiplication + Commitment:** People are being brought in AND the organisation is allocating resources. The deal has internal sponsorship, not just interest.

**Stalling + Regression:** The deal isn't just slow, it's going backwards. Multiple negative signals compound into "this deal is dying."

**Restart + Acceleration:** A stalled deal doesn't just restart, it accelerates. This often means an external event created urgency that didn't exist before.

[YOUR COMPOUND PATTERNS: Add additional compound patterns you've observed in your deal history. What signal combinations predict wins or losses in your pipeline? Fill from deal data after seeding 20+ deals.]

---

## Signal Strength by Stage

[YOUR SIGNAL WEIGHTS: Calibrate signal strength by stage from your deal data. Each signal type has different weight at different stages. Activation matters most at Stage 1-2. Commitment matters most at Stage 3-4. Acceleration matters most when the deal is already qualified. Replace with weights derived from your closed-won and closed-lost deals.]
