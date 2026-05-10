# Deal Intelligence Starter — Data Schema

**Status:** Starter schema — 8 tables for deal-level intelligence
**Database:** Any Postgres-compatible with JSONB support
**Table prefix:** `di_` (deal intelligence)

---

## Design Principles

This is the starter schema — the foundation for deal-level intelligence. Eight tables support the full Phase 1-2 pipeline (signal collection, classification, lifecycle management, and versioned deal state assembly) plus on-demand forensic analysis.

**Event sourcing:** `di_raw_signals` is the event store — the source of truth. Every observation is an immutable event. The deal state projection is a materialised view built from the signal stream. If a projection looks wrong, it can be rebuilt from events.

**SCD Type 2 versioning:** `di_deal_state` uses Slowly Changing Dimension Type 2 — each version of a deal's state is a complete row with `valid_from` and `valid_to` timestamps. New versions are only created when something actually changed. The full deal journey is readable from the version history.

**Framework loading:** Frameworks (F02, F03, F07) live in the `di_frameworks` table in the database, loaded via SQL queries. The schema records which framework version produced each classification via `framework_id` and `framework_version` fields.

**Projection updates:** `di_deal_state` updates daily (readers and assembler run on a daily schedule) plus on significant events (stage transitions, classified signals). A new version is only written when something changed in any of the five dimensions. If nothing changed, `last_confirmed_at` is updated on the current version to show the deal was checked.

---

## Schema Overview

| Table | Phase | Purpose |
|-------|-------|---------|
| `di_raw_signals` | Phase 1 (Reading) | Event store — every observation from every source |
| `di_signal_classifications` | Phase 1 (Reading) | Framework-based classifications applied to signals |
| `di_deal_state` | Phase 2 (Assembly) | SCD Type 2 versioned deal state projection |
| `di_lifecycle_state` | Phase 2 (Assembly) | Freshness and decay management |
| `di_scratchpad` | Cross-cutting | Working memory for reader observations |
| `di_traceability_log` | Cross-cutting | Append-only reasoning record (all agents write here) |
| `di_frameworks` | Cross-cutting | Framework content storage (loaded via SQL queries) |
| `di_deal_briefs` | On-demand | Generated deal intelligence briefs |

---

## Phase 1: Signal Collection

### `di_raw_signals`

Every observation from every source, stored exactly once. No interpretation at this layer — structured capture only.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| source_system | text | crm / {{transcript_tool}} / manual_ae |
| source_record_id | text | ID in the originating system |
| signal_type | text | e.g., "email_sent", "stage_change", "transcript_pain_signal" |
| deal_id | text | CRM deal ID |
| contact_id | text | CRM contact ID (nullable) |
| company_id | text | CRM company ID (nullable) |
| raw_content | jsonb | The actual signal data — structure varies by source |
| observed_at | timestamptz | When the signal occurred in the real world |
| captured_at | timestamptz | When the system captured it |
| captured_by | text | Skill or human who captured it |
| confidence_tier | text | high / medium / low |
| metadata | jsonb | Additional context |

**Provenance rule:** Every signal record includes source system, source record ID, both timestamps, confidence, and who captured it. No orphan data.

---

### `di_signal_classifications`

Framework-based classifications applied to raw signals. One row per framework dimension assessed per signal.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| signal_id | uuid | FK to di_raw_signals |
| framework_id | text | F02, F03, F07 identifier |
| framework_version | text | Version from di_frameworks table |
| dimension | text | e.g., "deal_progression", "stakeholder_role", "friction_boost" |
| classification | text | e.g., "Activation", "Champion", "Political friction" |
| confidence | text | strong / moderate / weak |
| evidence_summary | text | Why this classification was assigned |
| classified_at | timestamptz | When classification occurred |
| classified_by | text | Skill that performed it |

**Classification is not exhaustive per signal.** A stage transition gets classified on F02. A transcript pain signal gets classified on F03 and F07. Framework loading rules determine which dimensions apply to which signal types. Each agent loads only the frameworks necessary to its judgment.

---

## Phase 2: Assembly

### `di_deal_state`

SCD Type 2 versioned projection. Complete five-dimension state per version. Each row is a self-contained picture — no need to replay from version 1 to understand version 5.

The signal stream in `di_raw_signals` is the source of truth. This table is the materialised view. If a projection looks suspect, replay the signals and rebuild.

#### Top-level columns

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| deal_id | text | CRM deal ID |
| version | integer | Incrementing per deal |
| valid_from | timestamptz | When this version became active |
| valid_to | timestamptz | When superseded (null = current) |
| last_confirmed_at | timestamptz | When last checked and confirmed still current |
| trigger_type | text | stage_transition / significant_signal / scheduled / manual |
| trigger_signal_id | uuid | FK to di_raw_signals (nullable for scheduled/manual) |
| deal_stage | text | Pipeline stage at this version |
| deal_value | numeric | Deal value at this version |
| close_date | date | Expected close date at this version |
| org_type | text | Organisation type |
| industry_vertical | text | More specific vertical if relevant (nullable) |
| use_case | text | Primary use case (nullable) |
| use_case_secondary | text | Secondary use case (nullable) |
| deal_synopsis | text | What this deal is about — plain language, updated each version |
| titles_state | jsonb | Dimension 1: who is involved and what they do |
| conversation_state | jsonb | Dimension 2: what signals are present |
| cadence_state | jsonb | Dimension 3: engagement frequency |
| velocity_state | jsonb | Dimension 4: time-in-stage and progression |
| boost_friction_state | jsonb | Dimension 5: what's accelerating or stalling |
| classification_ids | uuid[] | All classifications informing this version |
| is_current | boolean | True for latest version |

#### Dimension 1: `titles_state`

Who is involved, what stakeholder roles are filled, and what each person is about.

<!-- Role taxonomy is configurable. Default names are generic. Define your own roles in QUESTIONS.md Section 3. -->

```json
{
  "contact_count": 4,
  "multi_thread": true,
  "stakeholder_roles": {
    "primary_sponsor": {
      "contact_id": "contact_123",
      "synopsis": "Driving the evaluation internally, introducing new stakeholders.",
      "strength": 7
    },
    "budget_authority": {
      "contact_id": "contact_456",
      "synopsis": "Budget authority confirmed, asking about pricing and timelines.",
      "strength": 5
    },
    "technical_assessor": null,
    "process_owner": null
  },
  "contacts": [
    {
      "contact_id": "contact_123",
      "role_key": "primary_sponsor",
      "synopsis": "Driving the evaluation internally, introducing new stakeholders.",
      "first_appeared_version": 3,
      "engagement_status": "active"
    }
  ],
  "roles_present": ["primary_sponsor", "budget_authority"],
  "roles_missing": ["technical_assessor", "process_owner"]
}
```

#### Dimension 2: `conversation_state`

What signal types are present at this point and what's unresolved. Counts, not detail — the detail lives in `di_signal_classifications`.

```json
{
  "signals_present": {
    "pain": { "count": 3, "most_recent": "2026-04-15" },
    "commitment": { "count": 1, "most_recent": "2026-04-10" },
    "authority": { "count": 0, "most_recent": null },
    "objection": { "count": 2, "most_recent": "2026-04-12" },
    "competitor_mention": { "count": 1, "most_recent": "2026-04-08" }
  },
  "language_posture": "evaluating"
}
```

#### Dimension 3: `cadence_state`

Engagement frequency tracking.

```json
{
  "last_engagement": "2026-04-15",
  "days_since_engagement": 5,
  "stage_benchmark_days": 21,
  "risk_flag": null
}
```

Risk flags fire at thresholds: `gone_quiet`, `at_risk`, `critical`.

#### Dimension 4: `velocity_state`

Where the deal is, how long it's been there, and how it's progressing.

```json
{
  "days_in_stage": 14,
  "stage_benchmark_days": null,
  "close_date": "2026-06-30",
  "deal_value": 250000,
  "regressions_total": 0
}
```

#### Dimension 5: `boost_friction_state`

What's actively accelerating or stalling the deal. Friction persistence is tracked across versions.

```json
{
  "friction": [
    {
      "type": "process_friction_pending_review",
      "classification_id": "uuid",
      "observed_at": "2026-04-08",
      "contact_ids": []
    }
  ],
  "boosts": []
}
```

#### Example version history

> **Example stages** -- replace with your pipeline. See pipeline-stage-guide.md.

| deal_id | version | deal_stage | valid_from | valid_to | is_current | last_confirmed_at |
|---------|---------|------------|------------|----------|------------|-------------------|
| deal_123 | 1 | SAL | Jan 5 | Jan 19 | false | Jan 19 |
| deal_123 | 2 | SAL | Jan 19 | Feb 3 | false | Feb 3 |
| deal_123 | 3 | SQL | Feb 3 | Mar 10 | false | Mar 10 |
| deal_123 | 4 | SQL | Mar 10 | Mar 28 | false | Mar 28 |
| deal_123 | 5 | Evaluating | Mar 28 | null | true | Apr 18 |

Five versions. Three stage changes, two within-stage changes. Version 5 is current, last confirmed two days ago.

---

### `di_lifecycle_state`

Tracks freshness and decay across the system. The lifecycle agent reads and writes here. Runs before any analytical agent.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| entity_type | text | signal / classification |
| entity_id | uuid | FK to relevant table |
| freshness_status | text | active / aging / stale / archived |
| freshness_window_days | integer | How long this entity type stays active |
| status_changed_at | timestamptz | When freshness status last changed |
| extension_reason | text | Why extended (nullable) |
| last_lifecycle_run | timestamptz | When lifecycle last assessed this entity |

---

## Cross-cutting Tables

### `di_scratchpad`

Working memory for reader observations. In the starter system, readers use this for language observations and friction notes. In the full system, pattern detection agents also write cross-deal observations here.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| run_id | uuid | Which agent run |
| run_type | text | daily_read / triggered |
| observed_at | timestamptz | When observed |
| observation_type | text | language_shift / friction_note / anomaly |
| observation | text | Free-form description |
| deal_ids | text[] | Deals involved |
| signal_classification_ids | uuid[] | Classifications forming this |
| dimensions_involved | text[] | Which of the five dimensions |
| status | text | new / accumulating / stale |

---

### `di_traceability_log`

Append-only. Every reasoning step recorded. Never deleted.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| entity_type | text | signal / classification / deal_state |
| entity_id | uuid | FK to relevant table |
| action | text | classified / state_versioned / capability_analysis_completed / etc. |
| reasoning | text | Why this action was taken |
| frameworks_consulted | text[] | Which frameworks were loaded |
| input_data | jsonb | What data was available |
| output_data | jsonb | What the decision produced |
| logged_at | timestamptz | When logged |
| logged_by | text | Skill or human |

---

### `di_frameworks`

Framework content storage. Agents load frameworks via SQL at the start of each run.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| framework_id | text | F02, F03, F07 (starter), F01-F10 (full system) |
| version | text | Semantic version |
| content | text | Full framework markdown |
| status | text | active / deprecated / draft |
| created_at | timestamptz | When stored |
| updated_at | timestamptz | When last updated |

---

### `di_deal_briefs`

Generated deal intelligence briefs from the Deal Analyst. Each brief is a complete forensic deep dive stored as structured markdown sections.

| Column | Type | Purpose |
|--------|------|---------|
| id | uuid | Primary key |
| deal_id | text | CRM deal ID |
| version | integer | Brief version (incrementing per deal) |
| deal_state_version | integer | Which di_deal_state version this brief was built from |
| deal_stage | text | Stage at time of analysis |
| deal_value | numeric | Value at time of analysis |
| company_name | text | Extracted from deal_synopsis for display |
| deal_synopsis | text | Deal synopsis at time of analysis |
| narrative | text | 4-6 paragraph Deal Narrative (markdown) |
| timeline | text | Section 1: Deal Timeline Reconstruction (markdown) |
| pattern_analysis | text | Section 2: Cross-Deal Pattern Context (markdown) |
| stakeholder_deep_dive | text | Section 3: Stakeholder Deep Dive (markdown) |
| friction_forensics | text | Section 4: Friction Forensics (markdown) |
| velocity_context | text | Section 5: Velocity Context (markdown) |
| evidence_quality | text | Section 6: Evidence Quality Assessment (markdown) |
| evidence_grade | text | strong / moderate / weak |
| key_risk | text | One sentence: most dangerous thing about this deal |
| key_opportunity | text | One sentence: strongest positive signal |
| stage_movement | text | advancing / stalling / regressing / new / closed |
| stakeholder_count | integer | Number of classified stakeholders |
| champion_names | text[] | Names of identified champions |
| signal_count | integer | Raw signals analysed |
| classification_count | integer | Classifications analysed |
| pattern_match_count | integer | Pattern matches (0 in starter) |
| version_count | integer | Deal state versions analysed |
| days_in_current_stage | integer | Days in stage at time of analysis |
| matching_patterns | jsonb | Pattern match details (empty array in starter) |
| generated_at | timestamptz | When brief was generated |
| generated_by | text | 'deal-briefer' |
| is_current | boolean | True for latest brief version |
| superseded_at | timestamptz | When this brief was replaced (nullable) |
| stale | boolean | Whether the brief needs regeneration |

---

## SQL Setup

Run this SQL against your Postgres database to create all 8 tables. See [`guides/database-setup.md`](../guides/database-setup.md) for connection and setup instructions.

```sql
-- ============================================================
-- Deal Intelligence Starter — Schema Setup
-- 8 tables for deal-level intelligence
-- Requires: Postgres 14+ with JSONB support
-- ============================================================

-- Enable uuid generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- ============================================================
-- Phase 1: Signal Collection
-- ============================================================

CREATE TABLE IF NOT EXISTS di_raw_signals (
    id              uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    source_system   text NOT NULL,
    source_record_id text,
    signal_type     text NOT NULL,
    deal_id         text,
    contact_id      text,
    company_id      text,
    raw_content     jsonb NOT NULL DEFAULT '{}'::jsonb,
    observed_at     timestamptz NOT NULL,
    captured_at     timestamptz NOT NULL DEFAULT NOW(),
    captured_by     text NOT NULL,
    confidence_tier text NOT NULL CHECK (confidence_tier IN ('high', 'medium', 'low')),
    metadata        jsonb DEFAULT '{}'::jsonb
);

CREATE INDEX IF NOT EXISTS idx_raw_signals_deal_id ON di_raw_signals (deal_id);
CREATE INDEX IF NOT EXISTS idx_raw_signals_observed_at ON di_raw_signals (observed_at);
CREATE INDEX IF NOT EXISTS idx_raw_signals_signal_type ON di_raw_signals (signal_type);

CREATE TABLE IF NOT EXISTS di_signal_classifications (
    id                uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    signal_id         uuid NOT NULL REFERENCES di_raw_signals(id),
    framework_id      text NOT NULL,
    framework_version text,
    dimension         text NOT NULL,
    classification    text NOT NULL,
    confidence        text NOT NULL CHECK (confidence IN ('strong', 'moderate', 'weak')),
    evidence_summary  text,
    classified_at     timestamptz NOT NULL DEFAULT NOW(),
    classified_by     text NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_classifications_signal_id ON di_signal_classifications (signal_id);
CREATE INDEX IF NOT EXISTS idx_classifications_framework ON di_signal_classifications (framework_id);

-- ============================================================
-- Phase 2: Assembly
-- ============================================================

CREATE TABLE IF NOT EXISTS di_deal_state (
    id                  uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    deal_id             text NOT NULL,
    version             integer NOT NULL,
    valid_from          timestamptz NOT NULL DEFAULT NOW(),
    valid_to            timestamptz,
    last_confirmed_at   timestamptz DEFAULT NOW(),
    trigger_type        text CHECK (trigger_type IN ('stage_transition', 'significant_signal', 'scheduled', 'manual')),
    trigger_signal_id   uuid REFERENCES di_raw_signals(id),
    deal_stage          text,
    deal_value          numeric,
    close_date          date,
    org_type            text,
    industry_vertical   text,
    use_case            text,
    use_case_secondary  text,
    deal_synopsis       text,
    titles_state        jsonb DEFAULT '{}'::jsonb,
    conversation_state  jsonb DEFAULT '{}'::jsonb,
    cadence_state       jsonb DEFAULT '{}'::jsonb,
    velocity_state      jsonb DEFAULT '{}'::jsonb,
    boost_friction_state jsonb DEFAULT '{}'::jsonb,
    classification_ids  uuid[],
    is_current          boolean NOT NULL DEFAULT true,
    UNIQUE (deal_id, version)
);

CREATE INDEX IF NOT EXISTS idx_deal_state_deal_id ON di_deal_state (deal_id);
CREATE INDEX IF NOT EXISTS idx_deal_state_is_current ON di_deal_state (is_current) WHERE is_current = true;
CREATE INDEX IF NOT EXISTS idx_deal_state_deal_current ON di_deal_state (deal_id, is_current) WHERE is_current = true;

CREATE TABLE IF NOT EXISTS di_lifecycle_state (
    id                    uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type           text NOT NULL,
    entity_id             uuid NOT NULL,
    freshness_status      text NOT NULL CHECK (freshness_status IN ('active', 'aging', 'stale', 'archived')),
    freshness_window_days integer,
    status_changed_at     timestamptz NOT NULL DEFAULT NOW(),
    extension_reason      text,
    last_lifecycle_run    timestamptz DEFAULT NOW()
);

CREATE INDEX IF NOT EXISTS idx_lifecycle_entity ON di_lifecycle_state (entity_type, entity_id);
CREATE INDEX IF NOT EXISTS idx_lifecycle_status ON di_lifecycle_state (freshness_status);

-- ============================================================
-- Cross-cutting Tables
-- ============================================================

CREATE TABLE IF NOT EXISTS di_scratchpad (
    id                        uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    run_id                    uuid,
    run_type                  text,
    observed_at               timestamptz NOT NULL DEFAULT NOW(),
    observation_type          text,
    observation               text NOT NULL,
    deal_ids                  text[],
    signal_classification_ids uuid[],
    dimensions_involved       text[],
    status                    text NOT NULL DEFAULT 'new' CHECK (status IN ('new', 'accumulating', 'stale'))
);

CREATE INDEX IF NOT EXISTS idx_scratchpad_status ON di_scratchpad (status);

CREATE TABLE IF NOT EXISTS di_traceability_log (
    id                   uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type          text NOT NULL,
    entity_id            uuid,
    action               text NOT NULL,
    reasoning            text,
    frameworks_consulted text[],
    input_data           jsonb,
    output_data          jsonb,
    logged_at            timestamptz NOT NULL DEFAULT NOW(),
    logged_by            text NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_traceability_logged_at ON di_traceability_log (logged_at);
CREATE INDEX IF NOT EXISTS idx_traceability_entity ON di_traceability_log (entity_type, entity_id);

CREATE TABLE IF NOT EXISTS di_frameworks (
    id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    framework_id text NOT NULL,
    version      text NOT NULL,
    content      text NOT NULL,
    status       text NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'deprecated', 'draft')),
    created_at   timestamptz NOT NULL DEFAULT NOW(),
    updated_at   timestamptz NOT NULL DEFAULT NOW()
);

CREATE UNIQUE INDEX IF NOT EXISTS idx_frameworks_active ON di_frameworks (framework_id) WHERE status = 'active';

CREATE TABLE IF NOT EXISTS di_deal_briefs (
    id                    uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    deal_id               text NOT NULL,
    version               integer NOT NULL,
    deal_state_version    integer,
    deal_stage            text,
    deal_value            numeric,
    company_name          text,
    deal_synopsis         text,
    narrative             text,
    timeline              text,
    pattern_analysis      text,
    stakeholder_deep_dive text,
    friction_forensics    text,
    velocity_context      text,
    evidence_quality      text,
    evidence_grade        text CHECK (evidence_grade IN ('strong', 'moderate', 'weak')),
    key_risk              text,
    key_opportunity       text,
    stage_movement        text CHECK (stage_movement IN ('advancing', 'stalling', 'regressing', 'new', 'closed')),
    stakeholder_count     integer,
    champion_names        text[],
    signal_count          integer,
    classification_count  integer,
    pattern_match_count   integer DEFAULT 0,
    version_count         integer,
    days_in_current_stage integer,
    matching_patterns     jsonb DEFAULT '[]'::jsonb,
    generated_at          timestamptz NOT NULL DEFAULT NOW(),
    generated_by          text NOT NULL DEFAULT 'deal-briefer',
    is_current            boolean NOT NULL DEFAULT true,
    superseded_at         timestamptz,
    stale                 boolean NOT NULL DEFAULT false,
    UNIQUE (deal_id, version)
);

CREATE INDEX IF NOT EXISTS idx_deal_briefs_deal_current ON di_deal_briefs (deal_id, is_current) WHERE is_current = true;
```

---

## Lifecycle and Decay Rules

The lifecycle agent runs before all other analytical agents — by the time they start work, the data layer has been cleaned.

**Raw signals:** Decay based on signal type. Engagement signals age faster than structural signals (stage changes, contact associations).

**Scratchpad observations:** Stale after 90 days with no reinforcement. Archived by lifecycle.

---

## Full System Schema

The full Deal Intelligence system adds 5 tables to support cross-deal pattern detection (Phase 3), hypothesis formation and confirmation (Phase 4), and intelligence agents (Phase 5):

| Table | Phase | What it adds |
|-------|-------|-------------|
| `di_hypotheses` | Phase 4 | Pattern hypotheses with provisional state, confidence decay, and human review workflow |
| `di_confirmed_patterns` | Phase 4 | Immutable confirmed patterns with structured conditions for matching against deal state |
| `di_icp_detections` | Phase 4 | ICP pattern detections — new persona types emerging from deal data |
| `di_pattern_matches` | Phase 5 | Active deals matched against confirmed and provisional patterns with match strength scores |
| `di_portfolio_briefs` | Phase 5 | Portfolio-level risk and opportunity analysis across the entire active pipeline |

These tables power the learning loop: observations accumulate in `di_scratchpad` → get promoted to `di_hypotheses` → confirmed by humans into `di_confirmed_patterns` → matched against active deals in `di_pattern_matches`. The starter schema's 8 tables are the foundation that the full system builds on — no schema changes required, only additions.
