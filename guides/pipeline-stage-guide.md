# Pipeline Stage Guide — Signal Collection Shared Reference

**Cluster:** deal-intelligence
**Type:** Shared data reading guide (not a framework)
**Purpose:** What each pipeline stage means for deal signal collection. What the collection system records at each stage and each transition.
**Authoritative source:** Your pipeline framework — full stage definitions, qualification framework gates, lifecycle phases. This guide does not duplicate those definitions. It adds what the collection layer needs to know on top of them.
**Consumed by:** F02 (progression signals), F04 (detection patterns), F08 (velocity benchmarks), collection layer

> **Note on stage IDs:** All stage IDs below use `{{STAGE_ID_*}}` placeholders. Replace every placeholder with your CRM's actual stage IDs. See `guides/data-mapping-guide.md` for how to pull your pipeline's stage values.

---

## Pipeline ID and Stage Map

> **Example pipeline:** The stage names below are from an enterprise SaaS sales process. Your pipeline stages may differ.

Pipeline: `default` (Sales Pipeline)

| Stage | CRM Stage ID | Pipeline Framework Phase | Collection Relevance |
|-------|-----------|-------------------------|----------------|
| Farming | `{{STAGE_ID_FARMING}}` | Phase 1 (legacy) | **Exclude from collection.** Not active pipeline. |
| Prospect | `{{STAGE_ID_PROSPECT}}` | Phase 1 (legacy) | **Exclude from collection.** Not active pipeline. |
| MQL | `{{STAGE_ID_MQL}}` | Phase 2 entry | **Collection starts here.** Record first engagement signals. |
| SAL | `{{STAGE_ID_SAL}}` | Phase 2 — SAL | Active collection. Qualification begins. Primary sponsor identification starts. |
| SQL | `{{STAGE_ID_SQL}}` | Phase 2 — SQL | Active collection. Full qualification present. Budget authority identified. |
| Evaluating | `{{STAGE_ID_EVALUATING}}` | Phase 2 — Evaluating | Active collection. Highest signal density. PoC underway. |
| Technical + DD | `{{STAGE_ID_TECH_DD}}` | Phase 2 — Tech DD | Active collection. Process owner signals dominant. |
| Contract Negotiation | `{{STAGE_ID_CONTRACT}}` | Phase 2 — Contract | Active collection. Commercial signals. Decision signals. |
| Ready to Invoice | `{{STAGE_ID_READY_TO_INVOICE}}` | Financial close | **Record transition timestamp only.** Deal is operationally won. |
| Invoiced | `{{STAGE_ID_INVOICED}}` | Financial close | Record transition timestamp only. |
| Paid | `{{STAGE_ID_PAID}}` | Financial close | Record transition timestamp only. Terminal for collection purposes. |
| Closed Lost | `{{STAGE_ID_CLOSED_LOST}}` | Terminal | **Record terminal state fully.** What was present at loss. Why. |
| Disqualified | `{{STAGE_ID_DISQUALIFIED}}` | Terminal | Record as disqualified. Minimal collection. |
| Stale | `{{STAGE_ID_STALE}}` | Terminal | **Record stale transition.** When, what was happening before. |

---

## What the Collection System Records at Each Stage

### MQL (Collection Starts)

- Who: first contact(s) associated. Title, function, seniority.
- What: how they entered (lead source, referral, event, inbound). First engagement type.
- When: timestamp of MQL entry. Time from first touch to MQL.
- Signal: activation signal type (F02). Intent tier of first action (F04).

**Do not record at MQL:** Qualification framework fields (too early), primary sponsor assignment (too early), stakeholder map (too early). These are empty and should be treated as "not yet applicable," not as gaps.

### SAL (Discovery and Qualification Begins)

- Who: primary sponsor candidate(s) emerging. First multi-threading signals. Contact additions.
- What they say: pain signals from discovery calls. Language indicating problem severity. Competitor mentions. Use case description in their words.
- Cadence: engagement frequency established. Response latency baseline.
- Velocity: time from MQL to SAL.
- Signals: activation (confirmed), multiplication (starting?), commitment (early signs?).

**Qualification framework recording starts here:** Pain, primary sponsor fields should begin populating. Record what's present AND what's absent — but absence at SAL is normal, not a risk signal.

### SQL (Qualified — Full Qualification Present)

- Who: full buying committee emerging. Budget authority identified. Technical assessor engaged. Stakeholder expansion pattern taking shape.
- What they say: metrics language (what success looks like to them), decision criteria (what they'll evaluate against), decision process (how they buy). Verbatim quotes on pain, budget, timeline.
- Cadence: should be accelerating from SAL cadence. Record direction change.
- Velocity: time from SAL to SQL. Compare against F08 benchmarks.
- Signals: multiplication (should be present), commitment (starting), stage transition evidence.

**Highest qualification recording density:** All elements should have data. Record completeness score per F04's qualification completeness scoring. Flag any element still empty at SQL as a genuine gap.

### Evaluating (PoC / Pilot — Highest Signal Density)

- Who: broadest stakeholder engagement. Technical assessors active. Process owner personas starting to appear. Primary sponsor's behaviour most visible here.
- What they say: technical questions, security concerns, integration requirements, evaluation criteria in their language, feedback on the PoC. This is where the richest conversation data lives.
- Cadence: highest expected frequency. Multiple touchpoints per week. Record actual vs expected.
- Velocity: time at this stage. The pipeline framework embeds "6m" as the stage label — that's the outer boundary.
- Boost/friction: most boost and friction moments happen here. PoC success = boost. Security review = potential friction. Stakeholder expansion at this stage = strong positive signal.

**Record everything.** This stage produces more forensic data per week than any other. Every meeting, every email thread, every new contact, every objection raised.

### Technical + DD (Verbal Yes — Process Stage)

- Who: process owner personas dominant — procurement, legal, InfoSec, TPRM. Primary sponsor should still be engaged but the new contacts are process people.
- What they say: requirements, compliance needs, timeline constraints from their side. Non-negotiable process language.
- Cadence: may slow down — this is process, not evaluation. Distinguish process-normal slowdown from deal-risk slowdown.
- Velocity: time at this stage. "3m" embedded in stage label.
- Friction: most friction at this stage comes from process — procurement timelines, security questionnaire turnaround, legal redlines. Record each friction event with what preceded it and how long it lasted.

### Contract Negotiation (Business Decision Made)

- Who: legal and commercial contacts. Budget authority should be engaged or recently engaged.
- What they say: terms, pricing discussions, SLA requirements.
- Cadence: transaction-pace — request/response pattern.
- Velocity: time from DD to contract. Time from contract to signature.
- Signal: commitment confirmed. Decision made. Record the moment and what was present.

### Terminal States

**Closed Lost:** Record fully. What stage did it die at? Who was the last active contact? What was the last signal before death? Why (from the AE's notes)? What was present in terms of qualification framework coverage, stakeholders, cadence? This is the data F06 (win-loss patterns) needs.

**Stale:** Record the transition. When did it go stale? What was the cadence leading up to staleness? Who went quiet first? Was there a specific friction event or did it just fade?

**Closed Won (Ready to Invoice → Paid):** Record the win. What was present at close? Full stakeholder map at time of win. Final deal value. Total cycle time. Stage-by-stage velocity. This is the other half of F06's data.

---

## Stage Transition Recording

Every stage transition gets a record containing:

1. **From stage → To stage** (with timestamps)
2. **Who was active in the 14 days before transition** (contact activity)
3. **What happened in the 14 days before transition** (meetings, emails, calls, notes)
4. **Velocity:** days at the previous stage
5. **What changed:** new contacts added? New signals detected? Qualification framework fields updated?

This transition record is the core unit of data for cross-deal pattern analysis. It's what allows the system to find "deals that transition from SQL to Evaluating typically had [X] happen in the two weeks before."

---

## Stage ID Trap — Critical Rule

CRM stage IDs are often misleading — slug-style IDs rarely match the display label your team uses. For example, a stage called "Sales Accepted Lead" might have an ID like `appointmentscheduled`, and "Evaluating" might be stored as `decisionmakerboughtin`.

**Never display raw stage IDs in any output.** Always resolve to the human-readable label using the mapping above or via your CRM's properties API. This is a non-negotiable anti-hallucination rule.

---

## What This Guide Does NOT Do

- Define what qualification framework elements mean → your qualification framework
- Define the full lifecycle beyond pipeline stages → your pipeline framework
- Define how to query the CRM → your CRM query framework
- Interpret why deals move fast or slow → F07 (friction), F02 (signals)
- Set velocity benchmarks → F08

This guide tells the collection layer **what to record, at which stage, and how to record stage transitions.** Everything else lives in the frameworks or the authoritative references.
