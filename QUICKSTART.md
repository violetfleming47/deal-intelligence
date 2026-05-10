# Quickstart: First Signal in 15 Minutes

Clone to classified signal in 15 minutes. You need a Supabase account (free tier works) and access to an AI platform that supports system prompts.

---

## Steps

### 1. Clone the repo

```bash
git clone https://github.com/violetfleming47/deal-intelligence.git
cd deal-intelligence
```

### 2. Create a Supabase project

Go to [supabase.com](https://supabase.com) and create a new project. The free tier is sufficient for evaluation.

### 3. Get your credentials

In your Supabase dashboard: **Settings > API**. Copy:
- Project URL (e.g., `https://xxxx.supabase.co`)
- Service role key (the `service_role` key, not `anon`)

### 4. Install the schema

Open the **SQL Editor** in your Supabase dashboard. Copy the full schema from `architecture/schema.md` and execute it.

The schema creates all tables with the `di_` prefix, appropriate indexes, and JSONB columns for flexible signal storage.

### 5. Verify schema installation

Run this in the SQL Editor:

```sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public'
  AND table_name LIKE 'di_%'
ORDER BY table_name;
```

You should see 8 tables:

```
di_deal_briefs
di_deal_state
di_frameworks
di_lifecycle_state
di_raw_signals
di_scratchpad
di_signal_classifications
di_traceability_log
```

### 6. Seed your data

You have two options:

**Option A: Start with your own deal data.** Export a deal from your CRM — stage history, contacts, and any associated notes or transcripts. Insert it as raw signals into `di_raw_signals` following the schema in `architecture/schema.md`. This gives you a real baseline from day one.

**Option B: Insert a minimal test signal.** Run this in the SQL Editor to create a single test signal you can classify:

```sql
INSERT INTO di_raw_signals (
    id, source_system, source_record_id, signal_type,
    deal_id, contact_id, company_id,
    raw_content, observed_at, captured_at, captured_by,
    confidence_tier, metadata
) VALUES (
    gen_random_uuid(),
    'crm',
    'test_stage_change_001',
    'stage_change',
    'deal_test_001',
    NULL,
    'company_test_001',
    '{"from_stage": "Lead", "to_stage": "Qualified", "deal_name": "Test Deal", "deal_value": 100000}'::jsonb,
    NOW(),
    NOW(),
    'manual',
    'high',
    '{"source": "quickstart_seed"}'::jsonb
);
```

### 7. Verify seed data

```sql
SELECT count(*) FROM di_raw_signals;
```

Expected result: `1` (or more if you seeded multiple signals).

### 8. Load the deal-tracker skill into your AI platform

The deal-tracker skill is located at `skills/deal-tracker/SKILL.md`. Load it according to your platform:

**Cursor / Windsurf / Cline:**
Copy `skills/deal-tracker/SKILL.md` into your IDE's skills directory (e.g., `.cursor/skills/deal-tracker/SKILL.md`).

**Claude Projects:**
Create a new project. Paste the full contents of `SKILL.md` into the project instructions.

**ChatGPT:**
Paste the full contents of `SKILL.md` as Custom Instructions or as the system prompt in a new GPT configuration.

**Any other platform:**
Use `SKILL.md` as the system prompt for a new agent/conversation.

### 9. Connect the agent to your database

The agent needs read/write access to your Supabase Postgres database. Connection method depends on your platform:

**Cursor (via MCP):**
Add the Supabase MCP server to your configuration with your project URL and service role key.

**Claude / ChatGPT / other:**
Provide the agent with your Supabase connection string or configure a database tool that the agent can call. The connection string is available in Supabase under **Settings > Database > Connection string**.

### 10. Run the first classification

Ask the agent:

> "Read the raw signals for deal deal_test_001 and classify them."

The agent will read the raw signals from `di_raw_signals`, load the relevant framework(s), and write classifications to `di_signal_classifications`.

### 11. Verify classification output

```sql
SELECT count(*) FROM di_signal_classifications;
```

You should see classified signals (one or more per raw signal, depending on how many classification dimensions apply).

### 12. Assemble deal state

Load `skills/state-builder/SKILL.md` into your platform (same method as step 8).

Ask the agent:

> "Assemble the deal state for deal deal_test_001."

The state-builder reads classified signals and projects them into a versioned deal state row in `di_deal_state`.

### 13. Generate a forensic brief

Load `skills/deal-briefer/SKILL.md` into your platform.

Ask the agent:

> "Generate a forensic brief on deal deal_test_001."

The deal-briefer reads the deal state, raw signals, and classifications to produce an evidence-backed analytical brief in `di_deal_briefs`.

### 14. Done

You now have the full pipeline running:

```
Raw signals -> Classified signals -> Deal state (versioned) -> Analytical brief
```

---

## What just happened

You ran the three core phases of the deal intelligence pipeline manually:

1. **Signal collection** was seeded (in production, the reader agents produce these daily from your CRM and transcripts).

2. **Classification** applied analytical frameworks to raw signals. The deal-tracker skill read each signal, determined which framework dimensions apply, and wrote structured classifications with confidence tiers and evidence citations.

3. **State assembly** projected the classified signals into a single versioned deal state row. This is the "current truth" about the deal — who the stakeholders are, what progression signals exist, what friction is active, all with evidence quality ratings.

4. **Forensic briefing** synthesised everything into a human-readable analytical brief. The briefer examines evidence quality, identifies gaps, flags contradictions, and produces a narrative that cites specific signals.

In production, steps 1-3 run daily on a schedule (readers in parallel, then lifecycle, then assembler). The briefer runs on demand — before a call, before a forecast review, or when you need to understand a deal deeply.

---

## Next: configure for YOUR deals

The seed data demonstrates the pipeline shape. To point this at your real pipeline:

1. Open [`QUESTIONS.md`](QUESTIONS.md) and answer the questions about your pipeline stages, deal signals, stakeholder roles, and friction patterns.

2. Your answers populate the `[YOUR ...]` markers in the frameworks, customising the classification logic to your business.

3. Follow [`guides/data-mapping-guide.md`](guides/data-mapping-guide.md) to connect your CRM and transcript sources.

4. Replace the `{{PLACEHOLDER}}` values in each skill with your actual field mappings.

See `QUESTIONS.md` for the full configuration walkthrough.
