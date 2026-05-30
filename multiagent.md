Task: Multi-Agent Foundation Plan Only

Change Classification: SAFE — planning only.

Do not modify code.
Do not edit Prisma schema.
Do not create migration.
Do not run db push.
Do not run migrate.
Do not create files.
Do not commit.

Goal:
Prepare a proper implementation plan for multi-agent listing processing in SellSell/PickPic.

Current state:
- main is clean and synced
- typecheck passes
- Prisma validate passes
- local DB drift cleaned
- Ollama/RunPod remains the only AI provider
- PostgreSQL, Redis, Qdrant, BullMQ are the stack

Plan these changes only:

1. Product creation flow
- Product should be created as pending first
- Explain how it becomes active later
- Explain what happens if moderation fails

2. Database planning
Plan fields/enums only:
- moderationStatus
- embeddingStatus
- AgentLog table
- any indexes needed
- no schema changes yet

3. BullMQ moderation job
Plan:
- queue name
- job payload
- retry behavior
- failure handling
- worker responsibility

4. Agent pipeline
Plan these agents:
- Publishing Agent
- Moderation Agent
- Image Validation Agent
- Category Agent
- Data Structuring Agent
- AI Description Agent
- Embedding Agent
- Qdrant Sync Agent

For each agent include:
- purpose
- input
- output
- DB fields touched
- sync or background
- risk level

5. Files likely needing changes
List exact likely files:
- Prisma schema
- product creation API
- queue files
- worker files
- AI service files
- Qdrant indexing files
- admin dashboard files if needed

6. Migration strategy
Plan a proper Prisma migration:
- create migration file
- validate locally
- avoid db push
- rollback plan

7. Safe implementation order
Give step-by-step order:
Phase 1: schema only
Phase 2: queue only
Phase 3: product pending flow
Phase 4: moderation worker
Phase 5: Qdrant/embedding worker
Phase 6: admin visibility
Phase 7: tests

8. Risk report
List:
- low risk changes
- medium risk changes
- high risk changes
- things to avoid

Return only a detailed implementation plan.
Do not make any changes.

SellSell / PickPic Multi-Agent System — Full Phase Roadmap

Phase 0 — Stabilization & Security ✅

Goal: Stabilize the platform before major AI architecture changes.

Completed

* SSRF protection for image URLs
* Auth refresh hardening
* Security tests
* Prisma/typecheck stabilization
* Git history cleanup
* Branch recovery
* DB drift cleanup
* Schema foundation commit (613c7ef)

Outcome

* Stable main
* Clean repo
* Synced remote
* Type-safe codebase

⸻

Phase 1 — Multi-Agent Schema Foundation ✅ (Code Only)

Goal: Prepare Prisma models/types without touching runtime flow yet.

Added in Prisma schema

Product

* moderationStatus
* embeddingStatus
* moderationReason

New Enums

* ModerationStatus
* EmbeddingStatus

New Model

* AgentLog

Indexes

* moderation indexes
* embedding indexes
* agent tracking indexes

Status

✅ Schema committed
❌ Migration not generated/applied yet

Important

Runtime DB still does NOT contain:

* agent_logs
* moderation_status
* embedding_status

⸻

Phase 1.5 — Migration Repair & DB Synchronization ⚠️

Goal: Repair Prisma migration history safely.

This is the current blocker.

⸻

Problems

DB Drift

Database contains:

* contact_info
* bio
* google_map_url

But migration history does not fully track them.

Prisma Issue

migrate dev wants to reset DB.

⸻

Planned Safe Fix

Step 1

Mark dangling migration as applied:

npx prisma migrate resolve --applied 20260512202125_add_contact_info_to_product

Step 2

Generate true DB→schema delta SQL.

Step 3

Inspect generated SQL manually.

Step 4

Apply ONLY safe SQL.

Step 5

Track migration safely.

⸻

Must NOT Exist in SQL

* DROP TABLE
* DROP COLUMN
* unrelated ALTER TYPE
* unrelated index removals

⸻

Phase 2 — Queue Infrastructure

Goal: Introduce BullMQ architecture safely.

⸻

Components

Redis Queue

* moderation queue
* embedding queue
* indexing queue

Worker Registration

* worker bootstrap
* queue registration
* retry logic

Files

* src/lib/queue/queues.ts
* scripts/start-worker.ts
* src/jobs/*

⸻

Important

Still no moderation logic yet.

Only queue skeletons.

⸻

Phase 3 — Soft Moderation Product Flow

Goal: Introduce moderation statuses without breaking seller UX.

⸻

Recommended Mode: Soft Moderation MVP

Product Lifecycle

On Create

status = active
moderationStatus = pending
embeddingStatus = pending

Listing appears instantly.

⸻

If AI Approves

moderationStatus = approved

⸻

If AI Rejects

status = rejected
moderationStatus = rejected

Listing hidden.

⸻

If AI Fails

moderationStatus = failed

Listing stays active.

Admin reviews later.

⸻

Phase 4 — Moderation Agent

Goal: Move moderation into BullMQ workers.

⸻

Responsibilities

* text moderation
* image moderation
* prohibited keyword detection
* AI policy checks

⸻

Worker Flow

AgentLog

started
completed
failed

Retries

* exponential backoff
* timeout handling

AI

Only:

* Ollama
* RunPod

No OpenAI/Claude APIs.

⸻

Phase 5 — Image Intelligence Agent

Goal: Improve listing quality.

⸻

Responsibilities

* blur detection
* wrong-category detection
* stock image detection
* NSFW detection
* duplicate detection

⸻

Future

Potential CLIP integration.

⸻

Phase 6 — Category Intelligence Agent

Goal: Auto-correct category structure.

⸻

Examples

User selects:

Electronics

AI upgrades:

iPhone > Mobile Phones > Apple

⸻

Inputs

* title
* images
* description

⸻

Phase 7 — Data Structuring Agent

Goal: Extract structured attributes.

⸻

Examples

Car

* make
* model
* year
* mileage

Phone

* storage
* RAM
* color

Property

* bedrooms
* bathrooms
* furnished

⸻

Phase 8 — AI Description Agent

Goal: Generate marketplace-quality descriptions.

⸻

Features

* English
* Arabic
* SEO-friendly
* seller tone preservation

⸻

Rules

User data always has priority.

AI enriches only.

⸻

Phase 9 — Embedding Agent

Goal: Generate semantic embeddings.

⸻

Inputs

Combined:

* title
* description
* AI description
* structured data
* image analysis

⸻

Output

Dense vectors.

⸻

Phase 10 — Qdrant Sync Agent

Goal: Sync vectors into semantic search.

⸻

Responsibilities

* upsert vectors
* hybrid search payloads
* metadata sync
* retry indexing

⸻

Conditions

Runs ONLY if:

moderationStatus = approved

⸻

Phase 11 — Semantic Search Upgrade

Goal: Improve buyer experience.

⸻

Features

Natural Search

Toyota under 2000 KD

Visual Search

upload image → similar products

Arabic Semantic Search

Arabic-first retrieval.

⸻

Phase 12 — Admin AI Dashboard

Goal: Observe AI system health.

⸻

Dashboard Sections

Failed Moderation

Failed Embeddings

Queue Health

AI Latency

Agent Logs

Manual Overrides

⸻

Phase 13 — Human-in-the-Loop Moderation

Goal: Add human correction layer.

⸻

Features

* restore false positives
* approve manually
* retrain prompts
* queue replay

⸻

Phase 14 — Production Hardening

Goal: Make AI infrastructure production-safe.

⸻

Features

* queue monitoring
* dead-letter queues
* worker autoscaling
* Redis resilience
* retry policies
* AI timeout handling
* metrics/logging
* observability

⸻

Phase 15 — Advanced AI Marketplace

Future expansion phase.

⸻

Possible Features

* autonomous pricing agent
* fraud detection agent
* negotiation agent
* seller reputation agent
* AI recommendations
* personalized feeds
* multilingual embeddings
* realtime voice listing flow

⸻

Current Status

Completed

✅ Stabilization
✅ Security hardening
✅ Schema foundation
✅ Git recovery
✅ Prisma stabilization

Blocked Pending

⚠️ Safe migration repair

Not Started

❌ Queue infrastructure
❌ Runtime agents
❌ Worker orchestration
❌ AI moderation runtime
❌ Embedding runtime
❌ Qdrant sync runtime