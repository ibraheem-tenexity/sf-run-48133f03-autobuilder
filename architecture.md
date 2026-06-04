# AutoBuilder — Architecture

**Run:** run-48133f03  
**Date:** 2026-06-04  
**Deploy target:** Railway (single service `sf-run-48133f03`)

---

## Guiding Principle: Demo-Simple (YAGNI)

One Railway service. One database. No microservices. React SPA served by the same Express backend. Supabase for auth + Postgres. This is an MVP to prove parity with Tribute and enable a working quote flow end-to-end.

---

## Components

```
┌─────────────────────────────────────────────────────┐
│                  Browser (React SPA)                │
│  - Dashboard (5 families)                           │
│  - Assembly Configurator (wizard)                   │
│  - Quote Generator + Export                         │
│  - Recipe Library + Editor + History                │
└─────────────────┬───────────────────────────────────┘
                  │ HTTPS REST + JSON
┌─────────────────▼───────────────────────────────────┐
│          Railway Service: sf-run-48133f03           │
│          Node.js + Express + TypeScript             │
│                                                     │
│  /api/auth          ← Supabase Auth (JWT)           │
│  /api/families      ← list 5 assembly families      │
│  /api/configure     ← POST config → BOM expansion   │
│  /api/quote         ← POST BOM → priced quote       │
│  /api/recipes       ← CRUD assembly recipes         │
│  /api/export        ← GET quote → CSV download      │
│  /* (static)        ← serve React build             │
│                                                     │
│  BOM Engine (pure TypeScript)                       │
│  └─ 5 family expanders (MTL-MC, Cryo, SF4500, etc.) │
└─────────────────┬───────────────────────────────────┘
                  │ Postgres client (Supabase SDK)
┌─────────────────▼───────────────────────────────────┐
│            Supabase (managed Postgres)              │
│                                                     │
│  Tables:                                            │
│    users       (managed by Supabase Auth)           │
│    families    (id, name, slug, description)        │
│    assemblies  (id, family_id, name, spec JSONB)    │
│    recipe_versions (id, assembly_id, recipe JSONB,  │
│                     created_at, author_id)          │
│    quotes      (id, assembly_id, config JSONB,      │
│                 bom JSONB, price, created_at)       │
└─────────────────────────────────────────────────────┘
```

---

## Data Model

### families
| col | type | notes |
|-----|------|-------|
| id | uuid | pk |
| name | text | e.g. "MTL-MC with insulation" |
| slug | text | e.g. "mtl-mc-insulated" |
| description | text | |

### assemblies
| col | type | notes |
|-----|------|-------|
| id | uuid | pk |
| family_id | uuid | FK → families |
| name | text | part number or descriptive name |
| current_recipe | jsonb | latest recipe snapshot |
| created_at | timestamptz | |
| updated_at | timestamptz | |

### recipe_versions
| col | type | notes |
|-----|------|-------|
| id | uuid | pk |
| assembly_id | uuid | FK → assemblies |
| recipe | jsonb | full recipe at this point in time |
| change_note | text | optional note from author |
| author_id | uuid | FK → auth.users |
| created_at | timestamptz | |

### quotes
| col | type | notes |
|-----|------|-------|
| id | uuid | pk |
| assembly_id | uuid | nullable (can be ad-hoc config) |
| config | jsonb | input parameters |
| bom | jsonb | expanded BOM |
| unit_price | numeric | computed price |
| margin_pct | numeric | applied margin floor |
| created_by | uuid | FK → auth.users |
| created_at | timestamptz | |

---

## BOM Engine

Pure TypeScript module — no I/O, fully deterministic, easy to unit-test against Tribute reference data.

```
src/engine/
  index.ts        ← dispatch by family slug
  mtl-mc.ts       ← MTL-MC with insulation expander
  cryo-no-armor.ts
  cryo-armor.ts
  sf4500-std.ts
  sf4500-armor.ts
  types.ts        ← ConfigInput, BomLine, PricedQuote
```

Each expander: `expand(config: ConfigInput): BomLine[]`  
Pricing: `price(bom: BomLine[], margin: number): PricedQuote`

---

## Feature → Screen Mapping

| Feature | Screen | Route |
|---------|--------|-------|
| F8 Auth | Login | /login |
| F1 Family Browser | Dashboard | / |
| F2 Assembly Configurator | Configure Wizard | /configure/:familySlug |
| F3 BOM Expansion | Step 3 of wizard | /configure/:familySlug (step 3) |
| F6 Quote Generator | Step 4 of wizard | /configure/:familySlug (step 4) |
| F7 ERP Export | Quote view (download button) | /configure/:familySlug (step 4) |
| F4 Recipe Authoring | Recipe Editor | /recipes/:id/edit |
| F5 Version History | Recipe History | /recipes/:id/history |
| F9 Parity Validator | (Admin panel) | /admin/parity |

---

## Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend | React + Vite + TypeScript + Tailwind CSS | Standard, fast, easy |
| Backend | Node.js + Express + TypeScript | Simple, matches BOM engine lang |
| Database | Supabase (PostgreSQL) | Auth + Postgres in one; managed |
| Hosting | Railway `sf-run-48133f03` | Constraint from factory |
| Build | esbuild via Vite | Fast |
| Testing | Jest | Unit tests for BOM engine |

---

## Deployment

Single Docker-style service on Railway:
- Build: `npm run build` (Vite builds React → dist/, tsc compiles server)
- Start: `node dist/server/index.js`
- Serves static React from `dist/client/`
- Env vars injected by Railway: `DATABASE_URL`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `JWT_SECRET`

---

## Dependencies Between Features

```
F8 Auth
  └── F1 Family Browser (requires auth)
        └── F2 Configurator (requires family)
              └── F3 BOM Engine (requires config inputs)
                    └── F6 Quote (requires BOM)
                          └── F7 Export (requires quote)

F4 Recipe Authoring (requires auth + author role)
  └── F5 Version History (requires recipe)

F9 Parity Validator (requires BOM engine)
```

Wave order:
- Wave 1: Project scaffold + auth + DB schema + seed data
- Wave 2: BOM engine (all 5 families) + API endpoints
- Wave 3: React frontend (dashboard, configurator wizard, quote, export)
- Wave 4: Recipe authoring + version history
- Wave 5: Polish + health check endpoint + Railway deploy config

---

## Required Tokens at Runtime

| Token | Source | Used for |
|-------|--------|---------|
| DATABASE_URL | Supabase | Postgres connection |
| SUPABASE_URL | Supabase | JS client |
| SUPABASE_ANON_KEY | Supabase | Frontend auth |
| SUPABASE_SERVICE_ROLE_KEY | Supabase | Server-side admin ops |
| PORT | Railway auto | HTTP server port |
