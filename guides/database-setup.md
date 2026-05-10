# Database Setup Guide

How to set up the database for the Deal Intelligence Starter. This guide uses Supabase, but any Postgres-compatible database with JSONB support works.

---

## 1. Create your database

### Option A: Supabase (recommended for getting started)

1. Go to [supabase.com](https://supabase.com) and create a project
2. Note your project URL and anon/service key — you'll need these for the MCP connection
3. Your database is ready — Supabase provides Postgres with full JSONB support

### Option B: Other Postgres providers

Any of these work:
- **Neon** — serverless Postgres
- **AWS RDS** — managed Postgres
- **Railway** — simple managed Postgres
- **Self-hosted** — any Postgres 14+ instance

You need: a connection string and a way for your agents to execute SQL (MCP server, API, or direct connection).

---

## 2. Connect the Supabase MCP server

Your agents need to execute SQL against your database. The Supabase MCP server gives them that ability.

### Install the Supabase MCP server

1. Go to [supabase.com/dashboard/account/tokens](https://supabase.com/dashboard/account/tokens) and generate an access token
2. Install the Supabase MCP server (`@supabase/mcp-server-supabase`) into your agent harness using that access token
3. The server uses `npx` to run — your harness documentation will show where to add MCP server configurations (Claude Code, Claude Desktop, Cursor, Windsurf, and most IDE harnesses all support MCP servers)

### Verify the connection

Ask your agent to list your Supabase projects or run a simple query. If it returns your project, the connection is working.

---

## 3. Run the schema

Run the SQL below against your database. If using Supabase, you can paste this into the SQL Editor (Dashboard > SQL Editor > New query), or have your agent run it via the MCP connection.

The full CREATE TABLE statements are in [`architecture/schema.md`](../architecture/schema.md) under the **SQL Setup** section. Copy the entire SQL block and run it.

### Quick verification after running the schema

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

---

## 4. Load your frameworks

The agents load frameworks from the `di_frameworks` table at the start of each run. You need to insert your populated frameworks (after filling in the `[YOUR ...]` markers from your QUESTIONS.md answers).

For each framework file, insert it:

```sql
INSERT INTO di_frameworks (id, framework_id, version, content, status, created_at, updated_at)
VALUES (
    gen_random_uuid(),
    'F02',
    '1.0',
    '[paste the full content of frameworks/F02-deal-progression-signals.md here]',
    'active',
    NOW(),
    NOW()
);
```

Repeat for F03 and F07.

**Tip:** You can have your agent do this. Point it at the framework files and ask it to insert each one into di_frameworks.

---

## 5. Update the skill placeholders

Every skill references `{{DATABASE_PROJECT_ID}}` and `{{DATABASE_EXECUTE_SQL}}`. Replace these with your actual values:

| Placeholder | Replace with |
|-------------|-------------|
| `{{DATABASE_PROJECT_ID}}` | Your Supabase project ID (from the project URL) |
| `{{DATABASE_EXECUTE_SQL}}` | The MCP tool name for executing SQL (e.g., `supabase_execute_sql` or your provider's equivalent) |

See [`guides/data-mapping-guide.md`](data-mapping-guide.md) for the full placeholder list across all skills.

---

## 6. Test the pipeline

Run a minimal test to confirm everything is wired up:

### Test 1: Write a test signal

```sql
INSERT INTO di_raw_signals (
    id, source_system, source_record_id, signal_type, deal_id,
    raw_content, observed_at, captured_at, captured_by, confidence_tier, metadata
) VALUES (
    gen_random_uuid(),
    'manual_ae',
    'test_001',
    'stage_change',
    'test_deal_001',
    '{"from_stage": "MQL", "to_stage": "SQL", "note": "Test signal"}'::jsonb,
    NOW(),
    NOW(),
    'setup_test',
    'high',
    '{}'::jsonb
);
```

### Test 2: Verify the write

```sql
SELECT id, signal_type, deal_id, raw_content 
FROM di_raw_signals 
WHERE deal_id = 'test_deal_001';
```

### Test 3: Clean up

```sql
DELETE FROM di_raw_signals WHERE deal_id = 'test_deal_001';
```

If all three work, your database is ready. The agents can now read and write through the MCP connection.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| MCP server won't connect | Check your access token. Regenerate if expired. |
| "relation di_raw_signals does not exist" | Schema hasn't been run. Paste the CREATE TABLE statements into the SQL editor. |
| Agent can't find tables | Check that tables are in the `public` schema. Supabase defaults to `public`. |
| JSONB write errors | Ensure your JSONB values are valid JSON. Use `'{"key": "value"}'::jsonb` syntax. |
| Permission denied on INSERT | Check your Supabase Row Level Security (RLS) policies. For initial setup, you may need to disable RLS on the `di_*` tables or create appropriate policies. |

---

## Row Level Security (Supabase)

Supabase enables Row Level Security (RLS) by default on new tables. For the deal intelligence pipeline, the agents need full read/write access. You have two options:

### Option A: Disable RLS on di_* tables (simpler for internal tools)

```sql
ALTER TABLE di_raw_signals DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_signal_classifications DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_deal_state DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_lifecycle_state DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_scratchpad DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_traceability_log DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_frameworks DISABLE ROW LEVEL SECURITY;
ALTER TABLE di_deal_briefs DISABLE ROW LEVEL SECURITY;
```

### Option B: Create service-role policies (if you want RLS active)

Use your service role key (not the anon key) in the MCP connection, and create policies that grant full access to the service role.

---

## Next steps

Once the database is set up:

1. **Map your CRM fields** — follow `guides/data-mapping-guide.md`
2. **Configure your transcript provider** — map the `{{TRANSCRIPT_*}}` placeholders
3. **Load your populated frameworks** — insert F02, F03, F07 into `di_frameworks`
4. **Run your first reader** — start with `deal-tracker` on a single deal to verify end-to-end
