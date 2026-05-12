# Panchayat AI (Gram Vikas AI OS) — Foundation

## 1) Complete Monorepo Folder Structure

```text
panchayat-ai/
├── apps/
│   ├── backend/
│   │   ├── app/
│   │   │   ├── api/
│   │   │   │   └── v1/
│   │   │   │       ├── routes_auth.py
│   │   │   │       ├── routes_dashboard.py
│   │   │   │       ├── routes_due_diligence.py
│   │   │   │       ├── routes_ai_copilot.py
│   │   │   │       └── routes_projects.py
│   │   │   ├── core/
│   │   │   │   ├── config.py
│   │   │   │   ├── security.py
│   │   │   │   ├── rbac.py
│   │   │   │   └── logging.py
│   │   │   ├── db/
│   │   │   │   ├── postgres.py
│   │   │   │   ├── redis.py
│   │   │   │   ├── neo4j.py
│   │   │   │   └── vector.py
│   │   │   ├── schemas/
│   │   │   ├── services/
│   │   │   │   ├── dd_engine/
│   │   │   │   ├── ai_copilot/
│   │   │   │   ├── rag/
│   │   │   │   └── workflow/
│   │   │   ├── workers/
│   │   │   │   ├── celery_app.py
│   │   │   │   └── jobs_due_diligence.py
│   │   │   ├── main.py
│   │   │   └── dependencies.py
│   │   ├── tests/
│   │   ├── pyproject.toml
│   │   └── Dockerfile
│   └── frontend/
│       ├── src/
│       │   ├── app/
│       │   │   ├── (auth)/
│       │   │   ├── (dashboard)/
│       │   │   │   ├── collector/
│       │   │   │   ├── bdo/
│       │   │   │   └── panchayat/
│       │   │   ├── api/auth/[...nextauth]/route.ts
│       │   │   ├── due-diligence/
│       │   │   ├── copilot/
│       │   │   └── layout.tsx
│       │   ├── components/
│       │   │   ├── ui/
│       │   │   ├── dashboard/
│       │   │   ├── due-diligence/
│       │   │   └── copilot/
│       │   ├── lib/
│       │   │   ├── api.ts
│       │   │   ├── auth.ts
│       │   │   ├── i18n.ts
│       │   │   └── rbac.ts
│       │   └── styles/
│       ├── public/
│       ├── package.json
│       └── Dockerfile
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
├── infrastructure/
│   ├── docker-compose.yml
│   ├── nginx/
│   └── terraform/
├── packages/
│   ├── shared/
│   │   ├── types/
│   │   └── constants/
│   └── config/
├── docs/
│   ├── architecture.md
│   ├── security.md
│   └── phase1-foundation.md
├── .env.example
└── README.md
```

## 2) Prisma Schema (Main Models)

> See `prisma/schema.prisma` for the full first-cut production model set.

### Coverage included
- Rajasthan admin hierarchy: State → District → Block → Panchayat
- User/Auth/RBAC: User, Role, Permission, UserRole, Session, AuditLog
- Project lifecycle: Project, ProjectMilestone, ProjectTask, Tender, Payment, AuditReport
- Vendors/DD: Vendor, VendorDocument, DueDiligenceReport, DDCheck, RiskSignal
- Intelligence: Scheme, FundingAllocation, Stakeholder, VendorRelationship
- AI: CopilotConversation, CopilotMessage, AIInsight, EmbeddingDocument

## 3) High-Level Architecture Diagram (Text)

```text
[Next.js 15 Frontend (PWA, i18n hi/en, RBAC UI)]
                |
                | HTTPS + JWT (NextAuth)
                v
[FastAPI Gateway/API Layer]
  |- Auth + RBAC middleware
  |- Validation + rate limiting + audit logging
  |- Module APIs: Dashboard, DD Engine, Copilot, Projects
                |
  +-------------+-------------------+-------------------+
  |             |                   |                   |
  v             v                   v                   v
[PostgreSQL]  [Redis]            [Neo4j]           [Vector DB]
(core tx)     (cache/queues)     (relationships)   (RAG embeddings)
  |                                  ^                  ^
  |                                  |                  |
  v                                  |                  |
[Prisma ORM]                         |                  |
                                     |                  |
                            [AI Orchestration Service (LangChain/LlamaIndex)]
                            |- Tool calling (search vendors, summarize risk, etc.)
                            |- Multi-lingual prompts (Hindi/English)
                            |- Confidence scoring + evidence citations
                            |- LLM routing: GPT-4o / Claude 3.5 / fallback models

Background Jobs (Celery/RQ workers)
  |- OCR/document parsing
  |- Due diligence batch analysis
  |- Alert generation
  |- Periodic project health scoring
```

## 4) First-Phase Implementation Plan (MVP Core: Modules 1, 3, 6)

### Sprint 1 — Platform Foundation (Week 1)
1. Monorepo bootstrap, linting, formatting, pre-commit.
2. PostgreSQL + Prisma schema migration + seed data (Rajasthan districts/blocks/panchayats).
3. FastAPI base app with structured logging, error model, health endpoints.
4. Next.js app shell with role-based route groups and language switcher (Hindi/English).

### Sprint 2 — Auth, RBAC, and Dashboard Core (Week 2)
1. NextAuth + JWT + refresh flow.
2. RBAC policy engine (Collector, BDO, Panchayat Secretary/Sarpanch).
3. Dashboard APIs + frontend role-specific cards:
   - Fund utilization snapshot
   - Project status distribution
   - Risk alerts
4. Audit log middleware for all write operations.

### Sprint 3 — Due Diligence Engine (Critical) (Week 3)
1. Document upload pipeline (PDF/XLSX/images) + OCR extraction.
2. DD checks: financial, technical, legal, social impact.
3. AI-generated DD report:
   - Red/Green flags
   - Risk score 0-100
   - Go/No-Go recommendation
   - Evidence citations
4. Async jobs + status tracking + retry policies.

### Sprint 4 — Hindi/English AI Copilot (Week 4)
1. Copilot chat endpoint + streaming responses.
2. Tool calling for projects, vendors, DD summaries, and alerts.
3. Bilingual prompt templates and transliteration-safe UX.
4. Confidence scoring + disclaimer and traceable evidence.

### Definition of Done for Phase 1
- Secure login with RBAC and audited actions.
- Three role dashboards functional.
- DD reports generated from uploaded files with citations.
- AI copilot works in Hindi + English for governance queries.
- Docker Compose one-command local run.
