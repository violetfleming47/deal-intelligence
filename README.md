# Deal Intelligence Starter

A 6-agent system that turns CRM data and meeting transcripts into structured, versioned deal intelligence. It reads your pipeline, classifies what it finds across three analytical dimensions, builds a versioned deal state for every deal, and produces forensic deep-dive briefs on demand.

The system is methodology-neutral -- stakeholder roles, pipeline stages, and qualification frameworks are all configurable. It ships with generic defaults and includes worked examples for MEDDIC teams and B2B SaaS companies.

This is the starter system. It does the hard part most teams never get right — structuring raw CRM and conversation data into something you can actually reason about. Point it at a deal before a call and you'll know who the champion is, what friction is active, what the progression signals say, and what the evidence quality looks like behind each claim.

---

## What it does

Six agents work in a three-phase pipeline:

```
Phase 1 — Reading (3 agents, daily, parallel)
  ├── Deal Properties Reader ─── stage transitions, value changes, close date shifts
  ├── Conversation Reader ─────── pain, commitment, authority, language posture
  └── Friction Reader ─────────── objections, blockers, boosts, competitive mentions

Phase 2 — Assembly (2 agents, daily, sequential)
  ├── Lifecycle ───────────────── freshness decay, entity status transitions (runs first)
  └── Assembler ───────────────── 3-dimension deal state with SCD Type 2 versioning

On-demand
  └── Deal Analyst ────────────── forensic deep dive on any individual deal
```

The readers pull signals from your CRM and transcripts. The assembler aggregates them into versioned deal state. The deal analyst reads everything and produces evidence-backed analytical briefs.

---

## What's in the repo

```
deal-intelligence/
├── README.md
├── QUICKSTART.md                   ← First signal in 15 minutes
├── QUESTIONS.md                    ← Start here. Answer these for your org.
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
│
├── skills/                         ← The 6 agents (system prompts)
│   ├── deal-tracker/               Phase 1: CRM property signals
│   ├── conversation-scanner/       Phase 1: deal progression signals
│   ├── blocker-scanner/            Phase 1: friction & boost signals
│   ├── freshness-guard/            Phase 2: freshness & decay management
│   ├── state-builder/              Phase 2: 3-dimension deal state projection
│   └── deal-briefer/               On-demand: forensic deal deep dive
│
├── frameworks/                     ← Analytical reasoning templates
│   ├── F02-deal-progression-signals.md
│   ├── F03-champion-and-stakeholder-roles.md
│   └── F07-objection-friction-patterns.md
│
├── guides/                         ← Data reading + integration guides
│   ├── database-setup.md           ← Supabase MCP connection + schema install
│   ├── data-mapping-guide.md       ← How to connect your CRM + transcripts
│   ├── crm-data-reading-guide.md   ← Anti-hallucination rules for CRM data
│   └── pipeline-stage-guide.md     ← Stage definitions and collection rules
│
│
└── architecture/
    └── schema.md                   ← Database schema (8 tables)
```

---

## How the pieces fit together

**Skills** are the agents — each one is a complete system prompt that defines the agent's boundary, what it reads, what it writes, how it loads frameworks, and how it degrades gracefully when data is sparse.

**Frameworks** are the reasoning engine. Each framework teaches two things: *what* something is and *how to read it from your data*. The "what" comes from your answers in QUESTIONS.md — your pipeline stages, your friction types, your stakeholder roles. The "how to read" sections are universal patterns for detecting signals in CRM data and transcripts. Every framework has `[YOUR ...]` markers showing exactly where your answers go.

**Guides** teach agents how to read your CRM correctly — when to trust a field, when to cross-reference, how to handle empty values, what "stale" means. These stop agents from confidently reporting on data that's eight months old.

**The schema** defines eight tables — signal collection, assembly, lifecycle, traceability, and framework storage. Every write is traced. Every version is preserved.

---

## How we run it

We run this system on **Anthropic Managed Agents** — each skill becomes a managed agent with MCP server connections to our database and CRM.

**But the skills are harness-agnostic.** Each skill is a system prompt — structured markdown that defines what the agent does, what it refuses to do, what context to load, and how to verify its work. You can deploy them in:

- **Anthropic Managed Agents** — what we use, with MCP connections
- **Cursor / Windsurf / Cline** — as agent skills in any IDE harness
- **Claude Code** — as project context
- **LangChain / LangGraph / CrewAI** — as system prompts in your orchestrator
- **Custom orchestration** — any system that can send a system prompt to an LLM and connect to your data sources

The `{{PLACEHOLDER}}` values in each skill tell you what to wire up.

### Platform deployment

**Cursor / Windsurf / Cline:** Copy each `skills/*/SKILL.md` file into your IDE's skills directory. The agent loads them automatically when relevant tasks match.

**Claude Projects:** Create a project per agent. Paste the skill content as project instructions. Attach the relevant frameworks as project knowledge.

**ChatGPT:** Paste each skill as the system prompt in a custom GPT or as Custom Instructions for a conversation. Attach frameworks as uploaded files.

**OpenClaw / LangChain / CrewAI:** Use each `SKILL.md` as the system prompt for a node in your orchestration graph. Wire MCP or direct database connections as tool definitions.

---

## Quick start

Get from clone to first classified signal in 15 minutes. See [QUICKSTART.md](QUICKSTART.md).

---

## Getting started

### 1. Answer the questions

Start with [`QUESTIONS.md`](QUESTIONS.md). It walks through what the system needs to know about your business — your pipeline stages, deal signals, stakeholder roles, and friction patterns.

### 2. Populate the frameworks

Your answers fill the `[YOUR ...]` markers in the three frameworks. You can do this manually or with an agent — give it your answers and the framework templates.

### 3. Pull your data source schemas and map the fields

Follow [`guides/data-mapping-guide.md`](guides/data-mapping-guide.md). Every `{{PLACEHOLDER}}` in the skills must be replaced with actual values from your CRM, transcript provider, and database.

### 4. Set up the database

Follow [`guides/database-setup.md`](guides/database-setup.md) to connect your database and install the schema. The guide covers Supabase MCP server setup, running the CREATE TABLE statements, loading frameworks, and verifying the connection. The SQL is in [`architecture/schema.md`](architecture/schema.md).

### 5. Deploy and run

Load each skill's `SKILL.md` as the system prompt for an agent in your harness. Run the three readers daily, lifecycle and assembler after them. Trigger the deal analyst on demand.

---

## Data source neutral

The system is designed for any CRM and any transcript provider. The three reader agents are the only ones that touch external data sources. The assembler and deal analyst operate on the internal database tables.

| Layer | Swap in your own |
|-------|-----------------|
| CRM | HubSpot, Salesforce, Pipedrive, Dynamics, custom API |
| Transcripts | Gong, Chorus, Fireflies, Otter, custom |
| Database | Any Postgres — Supabase, Neon, RDS, self-hosted |
| Agent harness | Anthropic Managed Agents, Cursor, LangChain, custom |

---

## The full system — what this grows into

This starter gives you **deal-level intelligence** — structured, classified, versioned state for every deal in your pipeline, with forensic briefs on demand.

The full system adds **cross-deal intelligence** — pattern detection, a learning loop, and portfolio-level analysis. Here's the complete architecture:

```
STARTER (this repo)
═══════════════════════════════════════════════════════════════

  Phase 1 — Reading (3 agents, daily, parallel)
    ├── Deal Properties Reader
    ├── Conversation Reader
    └── Friction Reader

  Phase 2 — Assembly (2 agents, daily, sequential)
    ├── Lifecycle
    └── Assembler

  On-demand — Deal Analyst


FULL SYSTEM
═══════════════════════════════════════════════════════════════

  Phase 1 — Reading (5 agents)
    ├── Deal Properties Reader          ← in starter
    ├── Conversation Reader             ← in starter
    ├── Friction Reader                 ← in starter
    ├── Cadence Reader                  engagement frequency, gaps, who drives it
    └── Use Case Reader                 what the buyer wants, which value prop resonates

  Phase 2 — Assembly (2 agents)
    ├── Lifecycle                       ← in starter
    └── Assembler                       ← in starter (expands to 5 dimensions)

  Phase 3 — Pattern Detection (6 agents, weekly, parallel)
    ├── Titles Pattern                  stakeholder composition across deals
    ├── Conversation Pattern            signal sequences across deals
    ├── Velocity Pattern                timing and duration patterns
    ├── Friction Pattern                friction persistence and resolution
    ├── Cadence Pattern                 engagement frequency patterns
    └── ICP Pattern                     cross-dimension persona patterns

  Phase 4 — Hypothesis & Confirmation (2 agents)
    ├── Hypothesis Formation            observations → testable hypotheses
    └── Confirmation                    human review → confirmed patterns

  Phase 5 — Intelligence (3 agents)
    ├── Matcher                         matches deals against confirmed patterns
    ├── Pipeline Risk                   portfolio-wide risk assessment
    └── ICP Evolution                   persona analysis, baseline updates

  On-demand — Deal Analyst              ← in starter (gains pattern matching)
```

### What the full system adds

**Two additional readers** — cadence and use case — give the assembler five analytical dimensions instead of three. More dimensions means richer deal state and more surface area for pattern detection.

**Six pattern detection agents** run weekly across every deal simultaneously. They read the full version history of `di_deal_state` and find what repeats — which stakeholder compositions correlate with wins, which friction types always stall deals at the same stage, which engagement patterns predict close. Observations accumulate in a scratchpad.

**The learning loop** promotes scratchpad observations into testable hypotheses once they appear across enough deals (3+ observations, 2+ deals). Hypotheses are provisional until a human reviews and confirms them. Confirmed patterns are immutable, versioned records with full evidence chains.

**The matcher** checks every active deal against every confirmed pattern daily. When a deal matches a known win pattern at 0.8 strength, or a known loss pattern at 0.6, that intelligence surfaces in the deal analyst's forensic briefs and in the pipeline risk assessment.

**Pipeline risk** assesses the entire active pipeline simultaneously — not deal by deal, but structurally. Where are the same friction patterns appearing across multiple deals? Where is velocity diverging from benchmarks? What does the portfolio look like as a whole?

**ICP evolution** synthesises persona intelligence from the pattern library to answer: is your ICP baseline still accurate, and what new buyer types are emerging?

### The compounding effect

The starter gives you structured intelligence about individual deals. The full system gives you intelligence that compounds — every deal that flows through makes the pattern library stronger, the matcher more accurate, and the frameworks more calibrated. At 50 confirmed patterns, the system becomes meaningfully predictive. At 150, it becomes a distinct competitive advantage.

The starter is the foundation. The full system is the learning engine built on top of it.

### Seven additional frameworks

The full system adds seven frameworks to the three in this starter:

| Framework | What it adds | In starter? |
|-----------|-------------|-------------|
| F01 — Baseline ICP | Who your current ICPs are — generated from your CRM data | No |
| F04 — Engagement Intent Weighting | How to detect roles and signals from data patterns | No — requires pattern library |
| F05 — Org Structure Patterns | Where functions sit in different org types | No |
| F06 — Win/Loss Patterns | What was present in wins versus losses | No |
| F08 — Velocity Benchmarks | What normal deal speed looks like, calibrated from your data | No — requires 20+ closed deals |
| F09 — Industry Context | What's happening in your target verticals | No — requires external data feeds |
| F10 — Positioning & Product | What your product does and why each persona cares | No |

These frameworks power the pattern detection agents and the intelligence layer. They're calibrated from your deal data over time — the system builds its own playbook.

---

## Patterns you can reuse in any domain

| Pattern | What it solves |
|---------|---------------|
| **Frameworks as reasoning guides** | Agents that access data but can't interpret it the way your people do |
| **Perspective separation** | Cross-pollination between analytical lenses producing rationalised output |
| **Three-layer scope control** | Agent drift — frameworks fill space, noise naming classifies distractions, prohibitions close gaps |
| **SCD Type 2 versioning** | Knowing what changed, when, and why — not just current state |
| **Freshness decay** | Stale data silently informing live decisions |
| **Collect/classify boundary** | One agent both capturing and interpreting, compounding errors |
| **Verbatim enforcement** | Quiet paraphrasing that makes evidence unverifiable |
| **Verification queries** | No way to prove the agent did its job — SQL checks after every stage |

---

## Contributing

PRs welcome — especially framework implementations for domains beyond B2B sales, alternative CRM integrations, and harness-specific deployment guides.

## License

MIT
