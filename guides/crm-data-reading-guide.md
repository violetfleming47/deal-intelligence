# CRM Data Reading Guide — Signal Collection Shared Reference

**Cluster:** deal-intelligence
**Type:** Shared data reading guide (not a framework)
**Purpose:** How the collection layer reads, queries, and interprets CRM data. Prevents quiet hallucination — the kind where output looks authoritative but is wrong.
**Authoritative source:** Your CRM query framework — full query patterns, property references, anti-hallucination rules, data quality warnings. This guide inherits those rules and adds what's specific to longitudinal collection.
**Consumed by:** Collection layer, detection patterns, any skill interpreting CRM data for the signal database

> **Note on field names:** This guide uses `{{CRM_FIELD_*}}` placeholders for CRM-specific field names. Map these to your CRM's actual field names (e.g., `{{CRM_FIELD_LAST_MODIFIED}}` might be `hs_lastmodifieddate` in HubSpot or `SystemModstamp` in Salesforce). See `guides/data-mapping-guide.md` for the full mapping process. The anti-hallucination rules and collection patterns are CRM-agnostic.

---

## Anti-Hallucination Rules (Inherited)

These are non-negotiable. They come from the CRM query framework. Every interaction with CRM data must follow them.

### 1. Never infer what you didn't pull
If a property wasn't in your query, you don't have it. Don't guess based on the deal name, the company name, or training data. If you need a field, request it.

### 2. Never display raw stage IDs
CRM stage IDs are often misleading slugs that don't match their display labels. Always resolve stage IDs to their human-readable labels using the stage map in `pipeline-stage-guide.md` or via your CRM's properties API.

### 3. Always scope to pipeline first
Stage labels appear in multiple pipelines with different IDs. Filter by pipeline before filtering by stage. The collection system collects from your primary sales pipeline only — unless explicitly extended.

### 4. Always distinguish empty from missing
A `null` field could mean: nobody filled it in, it was never relevant, or it was relevant but not logged. **Never interpret empty as "doesn't apply."** Interpret as "we don't know." Flag the gap but do not treat it as a signal unless the pipeline-stage-guide says the field should be populated at this stage.

### 5. Always check data freshness
A {{QUALIFICATION_FRAMEWORK}} field filled 8 months ago on a deal that's moved through 3 stages since is probably stale. Check `{{CRM_FIELD_LAST_MODIFIED}}` on the record. If the record was recently modified but specific fields weren't, the AE is updating stages but not notes — that's a different situation from a completely stale record.

### 6. Never trust a single object layer
{{QUALIFICATION_FRAMEWORK}} data exists on Deals (free-text), Contacts (checkboxes), and Companies (strategic context). They contradict each other. Pull from all three and note discrepancies — don't pick whichever looks better.

### 7. Cross-reference before concluding
When notes or activities reveal something that contradicts a deal property, note both. Activity logs and notes are closer to ground truth than property fields. Neither is gospel — present both views.

---

## Collection-Specific Rules (Additions)

These rules apply specifically to the longitudinal collection layer. They don't exist in the CRM query framework because that framework serves read-only intelligence skills, not a system that stores and compares data over time.

### 8. Record raw, interpret later
The collection layer stores what the CRM returns — field values, timestamps, activity content. It does NOT classify, score, or interpret at collection time. Classification is a separate judgment call that happens downstream. This prevents the collection layer from baking in assumptions that can't be audited.

### 9. Always capture the timestamp of collection
Every piece of data stored must include when it was collected from the CRM, not just the timestamp of the original activity. This allows the system to distinguish "this was true when we collected it" from "this was true when it happened." CRM data changes — deals get re-staged, contacts get updated, notes get edited.

### 10. Detect changes, don't overwrite history
When the collection layer pulls a deal and a field has changed since last collection, store the new value AND preserve the old value with its collection timestamp. The history of how fields changed IS signal data. A deal that was $200k and became $500k tells a different story than one that was always $500k.

### 11. Never assume a single pull is complete
`search_crm_objects` returns max 200 records per call. For any collection run covering the full pipeline, paginate using the `offset` value. Always check the `total` count against records returned. If total > records returned, you missed data.

### 12. Respect the data hierarchy for {{QUALIFICATION_FRAMEWORK}}
- **Deal-level fields** are the primary truth (but check freshness)
- **Contact-level checkboxes** say IF something was assessed (Yes/No), not WHAT was found
- **Company-level fields** are background context, set once and rarely updated
- When layers contradict, record all layers. Let the classification step handle reconciliation.

### 13. Email body priority
`{{CRM_FIELD_EMAIL_HTML}}` contains the full formatted email body. `{{CRM_FIELD_EMAIL_TEXT}}` is a plain text fallback that may be truncated. When collecting email content for signal extraction, always prefer the HTML version (strip HTML tags), fall back to the plain text version only if HTML is null/empty.

### 14. Meeting attendee extraction priority
Attendees for a specific meeting come from (in order of reliability):
1. `{{CRM_FIELD_MEETING_NOTES}}` — transcript provider participant lists (highest reliability)
2. `{{CRM_FIELD_MEETING_BODY}}` — calendar invite text
3. Contacts associated with the meeting object
4. **Never** from contacts associated with the deal — that's everyone ever involved, not who was in this meeting

### 15. Activity vs property: which tells the truth?
When a deal property says one thing and the activity log says another, the activity log is closer to ground truth. Properties are updated by humans with varying discipline. Activities are system-recorded events. But activities can be incomplete (not all calls are logged, not all meetings have notes). Record both, flag the discrepancy, let the classification layer reason about it.

---

## Known Data Quality Issues

Inherited from the CRM query framework. The collection layer must handle these explicitly.

### Structural
- CRM fields may have typos or inconsistencies (e.g., `pane_point` instead of `pain_point`) — check for field variants and query using the actual CRM field name, not what you expect it to be
- Related fields may exist under multiple names (singular and plural variants) — check both, prefer the one with more data
- Contact-level owner fields are often free text, not authoritative — use `{{CRM_FIELD_OWNER_ID}}` on the deal (the system owner ID, not free text)
- Some CRM records may contain data entry artifacts (e.g. double ampersands in company names) — always match on exact string
- Stage IDs are often misleading slugs with custom labels — always resolve via the stage map

### Trust levels by object
| Object | Field Type | Trust Level | Why |
|--------|-----------|-------------|-----|
| Deal | Stage, amount, close date | High | System-tracked or frequently updated |
| Deal | {{QUALIFICATION_FRAMEWORK}} free-text fields | Medium | Filled once, often stale |
| Deal | `notes_last_contacted` | High | Auto-calculated |
| Contact | {{QUALIFICATION_FRAMEWORK}} checkboxes | Low | Ticked during qualification, rarely revisited |
| Contact | Engagement timestamps | High | Auto-tracked |
| Contact | `jobtitle` | Medium | May be outdated if person changed roles |
| Company | Type, industry | Medium | Set once, stable for most companies |
| Company | {{QUALIFICATION_FRAMEWORK}} fields | Low | Set once, rarely updated, may differ from deal |
| Note | Content, timestamp | High | Written close to the event |
| Meeting | {{TRANSCRIPT_TOOL}} notes | High | Auto-generated from transcript |
| Meeting | Body (calendar invite) | Medium | May not reflect actual attendance |
| Email | Content, direction, timestamp | High | System-recorded |

### Sparse but valuable fields
These fields are rarely populated but carry high signal when they are. Map your CRM's equivalents:
- `{{CRM_FIELD_QUALIFICATION}}` (Deal) — structured qualification assessment
- `{{CRM_FIELD_SPONSOR_DRIVER}}` (Company) — primary sponsor motivation
- `{{CRM_FIELD_POC_PROCESS}}` (Company) — PoC or evaluation structure
- `{{CRM_FIELD_BUDGET_CYCLE}}` (Company) — timing intelligence
- `{{CRM_FIELD_MARKET_SIZE}}` (Deal) — market size proxy

The collection layer should always request these. Empty is expected. Populated is gold.

---

## Query Patterns for Longitudinal Collection

The CRM query framework defines patterns for single-point-in-time queries. The collection system runs repeatedly over time. These patterns are adapted for that use case.

### Incremental collection (preferred)
Rather than pulling everything every run, query for what changed since last collection:

```
Filter: {{CRM_FIELD_LAST_MODIFIED}} > [last collection timestamp]
```

This reduces API calls and makes change detection automatic — if a record appears in the results, something changed.

### Staged depth (inherited)
Don't pull notes and emails for every deal on every run. Use deal-level properties first to identify which deals had changes, then go deep only on those:

1. Pull all active deals with basic properties + `{{CRM_FIELD_LAST_MODIFIED}}`
2. Identify which deals have been modified since last collection
3. For modified deals only: pull notes, emails, meetings, calls, contacts
4. For unchanged deals: skip — nothing new to record

### New contact detection
When a deal gains a new contact association, that's a signal. To detect this:

1. Pull contacts associated with each active deal
2. Compare against contacts recorded in last collection
3. New contacts = potential multiplication signal, new stakeholder to classify
4. Lost contacts (previously associated, no longer are) = investigate — contact disengaged or data cleanup?

### Stage transition detection
When a deal's stage changes between collection runs, that's a stage transition event:

1. Record: from stage, to stage, transition timestamp, who was active in the 14-day window before
2. Forward transitions are progression signals (F02)
3. Backward transitions are regression signals (F02)
4. Record what else changed at the same time — new contacts, cadence shift, value change

---

## What This Guide Does NOT Do

- Define what each pipeline stage means for collection → `pipeline-stage-guide.md`
- Define role detection patterns → F04
- Define deal signal types → F02
- Define velocity benchmarks → F08
- Provide the full CRM property reference → your CRM query framework
- Define ICP criteria → your ICP framework

This guide tells the collection layer **how to interact with CRM data safely and longitudinally.** It is the anti-hallucination rulebook for every query the collection system makes.
