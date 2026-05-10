# Data Mapping Guide

How to connect this system to your CRM, transcript provider, and database. Every `{{PLACEHOLDER}}` in the skills needs a concrete value from your stack before the agents will work.

---

## The process

```
1. Pull your API / MCP schemas
2. Identify the data nodes each agent needs
3. Map your field names to the placeholders
4. Hardcode the mapped values into skills, frameworks, and guides
5. Verify with a dry run
```

This is a one-time setup. Once the mappings are hardcoded, the agents run against your live data.

---

## Step 1: Pull your schemas

### CRM schema

You need the schema for your CRM's deal, contact, engagement, and note objects. How you get it depends on your CRM and connection method.

**If using MCP (recommended):**

Your MCP server exposes tool descriptors that tell you exactly what parameters each tool accepts and what it returns. Pull the tool list from your MCP server and read each tool's schema.

```bash
# Example: list available MCP tools
# The exact command depends on your MCP client
curl https://your-mcp-server/tools | jq '.tools[] | {name, description, inputSchema}'
```

What you're looking for:
- **Deal search tool** — what filters does it accept? What fields does it return?
- **Deal retrieval tool** — what properties can you request?
- **Note/email search tool** — how do you filter by deal association?
- **Contact retrieval tool** — what fields are available?
- **Engagement/activity tools** — how do you pull meetings, calls, emails?

**If using direct API:**

Pull the API docs for your CRM. You need the endpoints and response schemas for:
- List/search deals (with filters for pipeline, stage, date range)
- Get deal properties (including custom fields)
- Get deal associations (contacts, companies)
- Search notes/emails by deal
- Get engagement history (meetings, calls)

### Transcript provider schema

Pull the API/MCP schema for your transcript provider. You need:
- **List meetings** — filtered by deal ID, date range, or attendee email
- **Get transcript** — full text content for a given meeting
- **Search meetings** — by attendee, keyword, date

### Database schema

Run the table definitions from `architecture/schema.md` against your Postgres instance. The DI tables (`di_raw_signals`, `di_signal_classifications`, `di_deal_state`, etc.) are the same regardless of CRM or transcript provider.

---

## Step 2: Identify the data nodes

Each Phase 1 reader agent needs specific data from your CRM and transcript provider. Here's what each one pulls:

| Agent | What it reads from CRM | What it reads from transcripts |
|-------|----------------------|-------------------------------|
| **Deal Properties Reader** | Deal stage, value, close date, owner, last modified date | — |
| **Cadence Reader** *(Full System)* | Meetings, emails, notes, calls (timestamps + associations) | Meeting list (for timestamp/attendance) |
| **Conversation Reader** | Emails, notes (body text) | Full transcript text |
| **Friction Reader** | Emails, notes (body text) | Full transcript text |
| **Use Case Reader** *(Full System)* | Deal description, notes | Full transcript text |

Phase 2+ agents only read from the DI database tables — they don't touch your CRM or transcripts directly.

---

## Step 3: Map your fields

### CRM field mapping

Pull the field names from your CRM schema and map them to the placeholders used in the skills.

| Placeholder | What it represents | HubSpot example | Salesforce example |
|-------------|-------------------|-----------------|-------------------|
| `{{CRM_FIELD_DEAL_NAME}}` | Deal/opportunity name | `dealname` | `Name` |
| `{{CRM_FIELD_DEAL_STAGE}}` | Current pipeline stage | `dealstage` | `StageName` |
| `{{CRM_FIELD_CLOSE_DATE}}` | Expected close date | `closedate` | `CloseDate` |
| `{{CRM_FIELD_OWNER_ID}}` | Assigned sales rep | `hubspot_owner_id` | `OwnerId` |
| `{{CRM_FIELD_DEAL_ID}}` | Unique deal identifier | `hs_object_id` | `Id` |
| `{{CRM_FIELD_LAST_MODIFIED}}` | Last update timestamp | `hs_lastmodifieddate` | `SystemModstamp` |
| `{{CRM_FIELD_NOTE_BODY}}` | Note/activity body text | `hs_note_body` | `Body` |
| `{{CRM_FIELD_TIMESTAMP}}` | Activity timestamp | `hs_timestamp` | `CreatedDate` |
| `{{CRM_FIELD_CREATED_BY}}` | Who created the record | `hs_created_by` | `CreatedById` |
| `{{CRM_FIELD_MEETING_TITLE}}` | Meeting subject line | `hs_meeting_title` | `Subject` |
| `{{CRM_FIELD_NOTES_UPDATED}}` | Last note update time | `notes_last_updated` | (custom) |
| `{{CRM_PIPELINE_ID}}` | Your sales pipeline ID | `default` | (your pipeline record ID) |

### Pipeline stage mapping

Map your pipeline stages to the stage placeholders. Every CRM uses different IDs for stages.

| Placeholder | What it means | Your stage ID |
|-------------|--------------|--------------|
| `{{STAGE_FARMING}}` | Pre-pipeline / nurture | _(fill in)_ |
| `{{STAGE_PROSPECT}}` | Identified prospect | _(fill in)_ |
| `{{STAGE_MQL}}` | Marketing qualified | _(fill in)_ |
| `{{STAGE_SAL}}` | Sales accepted | _(fill in)_ |
| `{{STAGE_SQL}}` | Sales qualified | _(fill in)_ |
| `{{STAGE_EVALUATING}}` | Active evaluation | _(fill in)_ |
| `{{STAGE_TECHNICAL_DD}}` | Technical / due diligence | _(fill in)_ |
| `{{STAGE_CONTRACT_NEGOTIATION}}` | Contract stage | _(fill in)_ |
| `{{STAGE_READY_TO_INVOICE}}` | Deal won, pending invoice | _(fill in)_ |
| `{{STAGE_INVOICED}}` | Invoiced | _(fill in)_ |
| `{{STAGE_PAID}}` | Paid / closed won | _(fill in)_ |
| `{{STAGE_CLOSED_LOST}}` | Lost | _(fill in)_ |
| `{{STAGE_DISQUALIFIED}}` | Disqualified | _(fill in)_ |
| `{{STAGE_STALE}}` | Stale / inactive | _(fill in)_ |

**Important:** Your pipeline might have fewer or more stages. Add or remove stage placeholders as needed. The stage ordering (used by the Deal Properties Reader to determine forward/regression/skip) must match your pipeline's actual progression sequence.

### MCP / API tool mapping

Map your MCP tool names or API endpoints to the tool placeholders.

| Placeholder | What it does | Your tool/endpoint |
|-------------|-------------|-------------------|
| `{{DATABASE_EXECUTE_SQL}}` | Run SQL against your database | _(fill in)_ |
| `{{CRM_SEARCH_DEALS}}` | Search deals with filters | _(fill in)_ |
| `{{CRM_GET_DEAL}}` | Get a single deal's properties | _(fill in)_ |
| `{{CRM_GET_DEAL_ASSOCIATIONS}}` | Get contacts associated with a deal | _(fill in)_ |
| `{{CRM_SEARCH_NOTES}}` | Search notes/emails by deal | _(fill in)_ |
| `{{TRANSCRIPT_LIST_MEETINGS}}` | List meetings for a deal | _(fill in)_ |
| `{{TRANSCRIPT_GET_TRANSCRIPT}}` | Get transcript metadata | _(fill in)_ |
| `{{TRANSCRIPT_GET_CONTENT}}` | Get full transcript text | _(fill in)_ |
| `{{TRANSCRIPT_SEARCH_MEETINGS}}` | Search meetings by attendee | _(fill in)_ |

### Other placeholders

| Placeholder | What it is |
|-------------|-----------|
| `{{DATABASE_PROJECT_ID}}` | Your database project/connection identifier |
| `{{DATABASE_NAME}}` | Your database name |
| `{{COMPANY_NAME}}` | Your company name (appears in boundary definitions and noise filters) |
| `{{TRANSCRIPT_PROVIDER_DOMAIN}}` | Your transcript provider's domain (for data quality filters) |
| `{{MEETING_BOT_DOMAIN}}` | Your meeting bot's domain (for data quality filters) |
| `{{QUALIFICATION_FRAMEWORK}}` | Your sales qualification methodology name (e.g., MEDDIC, BANT, Challenger) |

---

## Step 4: Hardcode into the skills

Once you have your mapping, do a find-and-replace across all files in the repo. Every `{{PLACEHOLDER}}` becomes the actual value from your stack.

```bash
# Example: replace all placeholders in skills/ (HubSpot example values shown)
find skills/ -name "*.md" -exec sed -i '' \
  -e 's/{{CRM_FIELD_DEAL_NAME}}/your_deal_name_field/g' \
  -e 's/{{CRM_FIELD_DEAL_STAGE}}/your_stage_field/g' \
  -e 's/{{CRM_FIELD_CLOSE_DATE}}/your_close_date_field/g' \
  -e 's/{{STAGE_FARMING}}/your_farming_stage_id/g' \
  {} +
```

**Don't just replace placeholders — validate the data shape.** Your CRM's API might return data in a different structure. For example:
- The Deal Properties Reader expects `raw_content` to be a flat JSON object. If your CRM nests properties inside `properties.fieldname`, update the SQL INSERT templates.
- The Cadence Reader expects meeting timestamps. If your transcript provider returns ISO 8601 dates but your CRM returns Unix timestamps, note the conversion in the skill.
- The Conversation Reader expects `raw_content.text` to contain verbatim text. Make sure your transcript provider returns full text, not summaries.

### Also update:
- **`guides/crm-data-reading-guide.md`** — replace generic CRM references with your CRM's specific behaviours (pagination, rate limits, field trust rules)
- **`guides/pipeline-stage-guide.md`** — fill in your actual stage IDs and what each stage means in your process
- **`frameworks/`** — some frameworks reference stage names or field names; update to match your terminology

---

## Step 5: Verify with a dry run

Before running the full pipeline, test each Phase 1 reader individually:

1. **Run the Deal Properties Reader** on a single known deal. Check:
   - Did it pull the correct properties from your CRM?
   - Did it write a valid `deal_snapshot` signal to `di_raw_signals`?
   - Are the field names in `raw_content` correct?

2. **Run the Cadence Reader** *(Full System — skip in starter)* on the same deal. Check:
   - Did it find meetings, emails, notes?
   - Are the timestamps correct?
   - Did the data quality filters work (skipping auto-generated notes, cancelled meetings)?

3. **Run the Assembler** after the readers. Check:
   - Did it create a `di_deal_state` row?
   - Are all five JSONB dimensions populated?
   - Does the stage resolve correctly?

Use the verification queries in `guides/database-setup.md` at each checkpoint.

---

## Common issues

| Problem | Cause | Fix |
|---------|-------|-----|
| Agent says "tool not found" | MCP tool name placeholder not replaced | Replace `{{CRM_SEARCH_DEALS}}` with your actual tool name |
| `raw_content` has wrong field names | CRM returns different property names than expected | Update the INSERT template in the reader skill |
| Stage shows as raw ID in output | Stage ID not in the resolution table | Add your stage ID to the Pipeline Stage ID Resolution table in each skill |
| No meetings found | Transcript tool filtering wrong | Check the tool parameters — deal ID field name, date format |
| Data quality filter too aggressive | Filter references a domain you don't use | Update `{{TRANSCRIPT_PROVIDER_DOMAIN}}` and `{{MEETING_BOT_DOMAIN}}` |
| Pagination fails | CRM paginates differently | Update the pagination logic in the reader skill to match your CRM's cursor/offset pattern |
