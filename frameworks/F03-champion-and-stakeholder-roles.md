# F03 - Champion & Stakeholder Roles

> This framework uses generic role names. Define your own roles in QUESTIONS.md Section 3, using your methodology's terminology as aliases.

**Cluster:** deal-analysis
**Purpose:** Define the roles people play in {{COMPANY_NAME}} deals and what each role means. Pure definitions only — detection patterns live in F04, deal-level progression patterns live in F02.
**Boundary:** This framework answers "what roles exist and what does each one mean?" It never describes how to detect a role from data. It never describes deal-level patterns.
**Consumed by:** state-builder, deal-briefer (full system: pattern-icp, pattern-titles)
**Loading depth:**
- Vocabulary: role names and one-line definitions only
- Reasoning: full descriptions, anti-patterns, and role interactions
- Full: everything including role combinations and stage relevance

**Populated from:** QUESTIONS.md § "Your People" → "What roles exist in your buying process?"

**Note:** The seven roles below are common in complex multi-stakeholder purchases. Your market may have more, fewer, or different roles. Add, remove, or rename roles to match your buying process. The structure (definition → strong signals → weak/absent signals → placeholder for calibration) should stay the same for each role.

---

## Why This Framework Exists

Every person in a deal plays a role. Understanding what that role is determines how to engage them, what their absence means, and whether a new persona type is genuinely adding something the existing framework doesn't cover.

---

## The Seven Roles

Not mutually exclusive - a person can play more than one role. Each role is defined by what the person does, not by their title or seniority.

---

### 1. Primary Sponsor

The coordinator. The connective tissue in the deal. They bring people in, bridge lines of business, keep things moving, and increase connectivity across the organisation.

**What they do:**
- Coordinate across functions — connecting the buying team to the evaluating team to procurement
- Bring new people into the conversation at the right time — not randomly, but because the deal needs them next
- Present in all (or nearly all) email threads — the consistent thread across communications
- Champion with individuals — tailor the case to each stakeholder they bring in
- Maintain high cadence — frequent engagement, quick responses
- Share internal context that helps {{COMPANY_NAME}} navigate (org politics, competing priorities, who to bring in, who to avoid)

[YOUR PRIMARY SPONSOR SPECIFICS: What does a primary sponsor look like in your deals specifically? What functions do they typically bridge? What internal context do they share?]

**What a strong primary sponsor looks like:**
- Connectivity expands over time - they keep introducing new people, new functions
- They take ownership of internal process - they're driving it, not waiting for {{COMPANY_NAME}} to push
- They communicate outside formal channels - personal investment in the outcome

**What a weak or absent primary sponsor looks like:**
- The deal stays single-threaded - one contact, no expansion
- Someone is engaged but never introduces anyone new or bridges to other functions
- Someone is responsive but not proactive - replies to everything but never initiates

[YOUR PRIMARY SPONSOR THRESHOLDS: What quantitative signals separate a strong primary sponsor from a weak one? e.g., "present in 80%+ of email threads," "introduces 2+ new contacts." Calibrate from your deal data. Detection patterns for identifying primary sponsors live in F04.]

---

### 2. Budget Authority

The overall approver. The deal doesn't close without their sign-off. Often difficult to identify because they don't appear frequently - their presence is felt through others referencing them and through deal stage cadence rather than direct engagement.

**What they do:**
- Give final approval on budget and vendor selection
- Delegate the evaluation and relationship to others but retain the ultimate yes/no
- May only engage at decision points - or may never engage directly at all
- Their priorities and concerns shape the deal even when they're not in the room

**What identified budget authority access looks like:**
- The primary sponsor knows who they are and what they need to hear
- The deal has a clear path from evaluation to approval
- Their concerns (even second-hand) are being addressed in the deal strategy

**What missing budget authority access looks like:**
- Nobody can name who approves the budget
- The primary sponsor is enthusiastic but vague about "next steps" after evaluation
- The deal progresses technically but has no commercial path

Detection patterns for identifying the budget authority from data (including stage-based inference) live in F04.

---

### 3. Technical Assessor

The person who assesses whether {{COMPANY_NAME}} works — technically, architecturally, from a security and compliance standpoint.

[YOUR EVALUATOR SPECIFICS: What does a technical assessor assess in your product specifically? What's their typical title? What does a "positive evaluation" look like for your deals?]

**What they do:**
- Run or oversee the hands-on evaluation (sandbox, POC, technical deep-dive)
- Assess integration feasibility, security posture, data handling, API quality
- Produce a technical recommendation that the budget authority relies on
- Can veto on technical grounds even if the primary sponsor and budget authority are aligned

**What a positive technical assessor looks like:**
- Thorough but constructive - asks hard questions because they want it to work
- Shares their evaluation criteria openly
- Gives clear feedback on what passes and what doesn't

**What a negative technical assessor looks like:**
- Asks questions with no path to "yes" - evaluation as an end in itself
- Keeps moving the goalposts or introducing new requirements
- Engaged with the product but has no influence on the decision - a good review nobody reads

---

### 4. Process Owner

Controls access to the budget authority, to budget, to procurement, or to the next stage. They don't decide - they permit or prevent.

**What they do:**
- Run procurement, legal review, vendor assessment, security clearance, compliance sign-off
- Control the process and timeline, not the outcome
- Can slow or stop a deal without being the decision-maker
- Impose requirements (security questionnaires, compliance docs, reference checks) that consume time

**What a smooth process owner engagement looks like:**
- Requirements are known early and are standard
- The process owner engages predictably - request, response, move forward
- The primary sponsor has navigated this process before and knows the steps

**What a problematic process owner engagement looks like:**
- Requirements appear late and reset the timeline
- The process owner is used as a stalling tactic by the organisation
- Multiple gates in sequence with no clear end point

---

### 5. Internal Ally

A person (internal or external to the buying org) who gives {{COMPANY_NAME}} guidance on how to navigate the deal. Not an active advocate like a primary sponsor - they inform, they don't orchestrate.

**What they do:**
- Provide context about the organisation, decision process, politics, timing
- May be a former colleague, a consultant advising the buying organisation, or a contact from a previous deal
- Help {{COMPANY_NAME}} understand the landscape but don't actively sell internally
- Lower risk than a primary sponsor - they're not staking their reputation

**What a valuable internal ally looks like:**
- Current, accurate information about the organisation and its priorities
- Access to context {{COMPANY_NAME}} couldn't get through formal channels
- Honest about risks and obstacles, not just encouraging

**What a misleading internal ally looks like:**
- Information is outdated - they left the org years ago and the landscape shifted
- Overly encouraging without substance - "you'll be fine" without specifics
- Unknowingly steering {{COMPANY_NAME}} based on incomplete or wrong assumptions

---

### 6. Opposing Stakeholder

A person whose apprehension, risk aversion, or uncertainty slows or stops deal progression. Opposing stakeholders are rarely hostile - they're hesitant. They worry about what could go wrong, about disruption, about being responsible for a decision that doesn't work out.

**What they do:**
- Delay decisions by requesting more information, more proof, more time
- Raise concerns rooted in risk and uncertainty rather than specific objections
- Defer to others rather than taking a position
- Go quiet when a decision point approaches - their silence is the block
- May introduce new requirements or stakeholders late - not to derail, but because they genuinely feel it hasn't been properly vetted

**What apprehension looks like:**
- Same concern raised more than once across meetings - the answer didn't land, the worry is deeper than the question
- Deferral language - "I'd need to speak to...," "let's not rush this"
- Concerns raised without a path to resolving them - the concern sits there

**The difference between an opposing stakeholder and a cautious person:**
- A cautious person asks, listens, and moves forward when satisfied
- An opposing stakeholder asks, listens, and still hesitates
- The response to an opposing stakeholder is reassurance and evidence, not escalation

---

### 7. Cross-Functional Advocate

The person who creates urgency and organisational momentum. They may or may not be the primary sponsor, but they're the reason the deal moves instead of sitting in pipeline.

**What they do:**
- Set deadlines tied to business drivers ("we need this live by Q3")
- Connect the evaluation to something the organisation already committed to
- Bring the right stakeholders in when the deal needs them - not just adding contacts but advancing the conversation
- Push past internal resistance by tying {{COMPANY_NAME}} to an external driver

**What genuine mobilisation looks like:**
- Urgency tied to a real business driver (regulatory deadline, board commitment, competitive pressure)
- Deadlines that hold - the timeline is real, not aspirational
- New stakeholders brought in are relevant to the next stage, not just cc'd

**What false mobilisation looks like:**
- Deadlines that slip repeatedly - signals internal misalignment, not urgency
- Urgency without authority - pushes hard but can't actually make things happen
- Energy without follow-through - enthusiastic in meetings, nothing moves between them

---

## Role Combinations

People commonly play multiple roles. These are the patterns that matter:

- **Primary Sponsor + Cross-Functional Advocate:** The ideal - they coordinate AND create urgency. Deals with this combination move fastest.
- **Primary Sponsor + Technical Assessor:** Common in smaller orgs where the CTO both evaluates and coordinates. Risk: they're stretched thin.
- **Budget Authority + Opposing Stakeholder:** The worst scenario - the person with budget authority is apprehensive. The deal can't close until their concerns are resolved.
- **Process Owner + Opposing Stakeholder:** Procurement used to express organisational hesitation, not just process compliance.
- **Technical Assessor + Cross-Functional Advocate:** "We need this to solve an urgent problem" - they're evaluating because there's a live need, not just exploring.

[YOUR COMBINATIONS: What role combinations do you see in your deals? Which ones predict success? Which ones predict trouble? Populate from your deal history once you have 10+ seeded deals.]

---

## Role Relevance by Deal Stage

Not every role matters equally at every stage.

[YOUR STAGE MAP: For each pipeline stage, which roles should be present? Which are primary (must have), secondary (should have), and not expected? Build this matrix from your deal history. It becomes the assembler's expectation model for flagging missing stakeholders.]

---

## Role Transitions

People change roles as deals progress. Common transitions:

- Technical Assessor who becomes a Primary Sponsor after the product proves itself in evaluation
- Primary Sponsor who becomes an Opposing Stakeholder when the deal gets real and the risk of advocating becomes personal
- Process Owner who becomes a Cross-Functional Advocate when a regulatory deadline creates urgency they own

A role assignment is a snapshot, not permanent.
