# SellSell / PickPic Multi-Agent Development Prompts

## Overview

This document contains the structured execution prompts for building the SellSell / PickPic AI multi-agent marketplace safely and incrementally.

---

# Phase 0 — Stabilization & Security

## Goal

Stabilize the platform before major AI architecture changes.

## Tasks

* SSRF protection
* Auth refresh hardening
* Typecheck stabilization
* Prisma validation
* Branch cleanup
* Safe git recovery
* DB drift cleanup

## Status

✅ Completed

---

# Phase 1 — Multi-Agent Schema Foundation

## Goal

Prepare Prisma schema and TypeScript foundation for multi-agent moderation.

## Prompt

```text
Approved. Implement Phase 1 only: Schema Updates.

Change Classification: MEDIUM — Prisma schema and migration only.

Strict rules:
- Do NOT touch product creation flow yet.
- Do NOT edit API routes.
- Do NOT add workers.
- Do NOT add BullMQ queue code.
- Do NOT run db push.
- Do NOT change AI provider.
- Do NOT modify Qdrant logic.
- Do NOT implement moderation behavior yet.

Goal:
Add database foundation for Soft Moderation MVP.

Schema changes:
1. Add enum ModerationStatus:
   - pending
   - approved
   - rejected
   - failed

2. Add enum EmbeddingStatus:
   - pending
   - indexed
   - failed

3. Add fields to Product:
   - moderationStatus ModerationStatus @default(pending) @map("moderation_status")
   - embeddingStatus EmbeddingStatus @default(pending) @map("embedding_status")
   - moderationReason String? @map("moderation_reason") @db.Text

4. Add relation:
   - agentLogs AgentLog[]

5. Add model AgentLog mapped to agent_logs:
   - id String @id @default(uuid())
   - productId String @map("product_id")
   - product Product @relation(fields: [productId], references: [id], onDelete: Cascade)
   - agentType String @map("agent_type") @db.VarChar(50)
   - status String @db.VarChar(50)
   - metadata Json?
   - createdAt DateTime @default(now()) @map("created_at")

6. Add indexes:
   - @@index([productId, agentType])
   - Product indexes for moderationStatus and embeddingStatus.
```

---

# Phase 1.5 — Migration Repair & Safe DB Synchronization

## Goal

Repair Prisma migration history safely without losing local or production data.

## Prompt

```text
Task: Migration Repair Plan Only

Do not modify files.
Do not modify database.
Do not run migrate resolve.
Do not run db push.
Do not run db execute.
Do not run migrate dev.
Do not start Phase 2 workers.

Current status:
- main is synced with origin/main
- latest commit is 613c7ef
- Prisma schema includes multi-agent fields
- no migration file exists for those fields
- database has not been altered for this latest schema commit
- previous local migration drift was detected

Goal:
Create a safe migration repair plan that allows us to generate/apply the missing SQL migration without losing local or production data.
```

## Important Warnings

Never allow:

* DROP TABLE
* DROP COLUMN
* Unrelated ALTER TYPE
* Unrelated index removals
* Automatic DB reset

---

# Phase 2 — Queue Infrastructure

## Goal

Introduce BullMQ safely.

## Components

* Moderation Queue
* Embedding Queue
* Indexing Queue
* Worker bootstrap
* Retry logic
* Queue registration

## Prompt

```text
Task: Queue Infrastructure Only

Change Classification: MEDIUM

Rules:
- Do NOT modify product flow yet.
- Do NOT implement moderation logic.
- Do NOT touch production DB.
- Do NOT add AI moderation yet.

Tasks:
1. Create BullMQ queue definitions.
2. Register moderationQueue.
3. Register embeddingQueue.
4. Register indexingQueue.
5. Update worker bootstrap.
6. Add empty worker placeholders.
7. Add retry/backoff config.

Run:
- npm run typecheck
- npx prisma validate

Return:
- files changed
- queue names
- retry strategy
- worker registration summary
```

---

# Phase 3 — Soft Moderation Product Flow

## Goal

Introduce moderation states without blocking sellers.

## Recommended Mode

Soft Moderation MVP.

## Lifecycle

### Product Creation

```text
status = active
moderationStatus = pending
embeddingStatus = pending
```

### Approved

```text
moderationStatus = approved
```

### Rejected

```text
status = rejected
moderationStatus = rejected
```

### Worker Failure

```text
moderationStatus = failed
```

## Prompt

```text
Implement Soft Moderation Product Flow.

Rules:
- Listings must appear instantly.
- AI moderation must run in background.
- Worker failures must not hide listings.
- Rejected content must automatically hide.
- Keep Ollama + RunPod only.
```

---

# Phase 4 — Moderation Agent

## Goal

Move moderation into BullMQ workers.

## Responsibilities

* Text moderation
* Image moderation
* Prohibited keyword detection
* Safety analysis

## Prompt

```text
Implement Moderation Agent.

Requirements:
- BullMQ worker
- Retry handling
- AgentLog integration
- moderationStatus updates
- SSRF-safe image handling
- RunPod/Ollama only

AgentLog statuses:
- started
- completed
- failed
```

---

# Phase 5 — Image Intelligence Agent

## Goal

Improve listing image quality.

## Features

* Blur detection
* Wrong-category detection
* NSFW checks
* Duplicate image detection
* Future CLIP support

---

# Phase 6 — Category Intelligence Agent

## Goal

Improve marketplace categorization.

## Features

* AI category correction
* Smart category hierarchy
* Arabic-aware categorization

---

# Phase 7 — Data Structuring Agent

## Goal

Extract structured product attributes.

## Examples

### Cars

* make
* model
* year
* mileage

### Phones

* storage
* RAM
* color

### Property

* bedrooms
* bathrooms
* furnished

---

# Phase 8 — AI Description Agent

## Goal

Generate enriched marketplace descriptions.

## Features

* English + Arabic
* SEO-friendly
* Seller tone preservation
* AI enhancement only

---

# Phase 9 — Embedding Agent

## Goal

Generate semantic vectors.

## Inputs

* title
* description
* AI description
* structured data
* image analysis

## Output

Dense embeddings.

---

# Phase 10 — Qdrant Sync Agent

## Goal

Push vectors into Qdrant.

## Responsibilities

* Vector upsert
* Payload sync
* Metadata sync
* Retry indexing

## Rules

Only run after moderation approval.

---

# Phase 11 — Semantic Search Upgrade

## Goal

Deliver natural AI-powered search.

## Features

* Natural language search
* Arabic semantic search
* Visual search
* Hybrid ranking

---

# Phase 12 — Admin AI Dashboard

## Goal

Observe AI health.

## Sections

* Failed moderation
* Failed embeddings
* Queue health
* AI latency
* Agent logs
* Manual review

---

# Phase 13 — Human-in-the-Loop Moderation

## Goal

Add human review workflows.

## Features

* Restore false positives
* Manual approvals
* Queue replay
* Moderation override

---

# Phase 14 — Production Hardening

## Goal

Production-safe AI infrastructure.

## Features

* Queue monitoring
* Dead-letter queues
* Retry policies
* Worker autoscaling
* Metrics/logging
* Observability
* Redis resilience

---

# Phase 15 — Advanced AI Marketplace

## Future Features

* AI pricing agent
* Fraud detection
* Negotiation agent
* Recommendation engine
* Personalized feeds
* Multilingual embeddings
* Voice listing flow

---

# Recommended Delivery Timeline

| Stage                     | Timeline    |
| ------------------------- | ----------- |
| Moderation MVP            | 2–3 weeks   |
| AI Search Intelligence    | 3–6 weeks   |
| Marketplace Intelligence  | 4–8 weeks   |
| Production Hardening      | 2–4 months  |
| Enterprise AI Marketplace | 6–12 months |

---

# Critical Risk Areas

## Highest Risks

1. Migration consistency
2. AI reliability
3. Semantic search quality
4. Queue orchestration

## Advice

Do not rush:

* migrations
* queue systems
* semantic retrieval
* background workers

---

# Current Project Status

## Completed

✅ Stabilization ✅ Security hardening ✅ Schema foundation ✅ Git recovery ✅ Prisma stabilization

## Pending

⚠️ Migration repair ⚠️ Safe DB synchronization

## Not Started

❌ Queue infrastructure ❌ Moderation runtime ❌ Embedding runtime ❌ Qdrant sync runtime ❌ Admin AI dashboard
