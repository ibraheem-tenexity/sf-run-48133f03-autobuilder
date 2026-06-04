# AutoBuilder — Product Requirements Document

**Customer:** Singer Industrial / Unisource (Dallas, TX)  
**Contractor:** Tenexity Inc. (Glen Allen, VA)  
**Date:** 2026-06-04  
**Version:** 1.0

---

## 1. Problem

Unisource, a division of Singer Industrial, configures and sells made-to-order hose and flexible-connector assemblies. Configuration recipes today live inside two legacy character-mode programs — OHSM and OLFM — running inside the Tribute ERP. The logic is correct; the operability is not:

- Configuration knowledge is locked with 1–2 experts (key-person risk)
- No preview or validation before committing a recipe
- No change history — silent drops cause build holds, mispriced quotes
- Zero accessibility outside the ERP terminal
- Cannot scale to other Singer business units

**Cost:** Incorrect quotes, production holds, margin erosion, institutional knowledge flight risk.

---

## 2. Users

| User | Job-to-be-done |
|------|----------------|
| Assembly Author (OHSM/OLFM expert) | Author and maintain assembly recipes without a green screen |
| Sales Rep | Configure an assembly, get a price, generate a customer-ready quote fast |
| Sales Manager | Enforce margin floors; review quotes before they go out |
| Operations / Engineering | Track recipe changes, audit history, hand off to production |

---

## 3. Value Proposition

AutoBuilder delivers:
1. **Trusted quotes** — reproduces Tribute's BOM expansion logic for all 5 Unisource assembly families
2. **Guided authoring** — any team member can build or change an assembly; validation catches mistakes before they propagate
3. **Durable knowledge base** — every assembly versioned; every change tracked; config logic becomes a Singer asset, not a person
4. **Fast quoting** — sales configures and prices in minutes with built-in margin protection
5. **ERP-ready output** — exports a structured data file Tribute/P21 can consume

---

## 4. Competitor Landscape

### 4.1 Danfoss Hose Assembly Pro
**URL:** https://www.mdm.com/news/operations/manufacturing/danfoss-launches-self-service-hose-configuration-quoting-tool/  
**What it does:** Powered by Intelli.Build configuration engine. Rules-based hose/fitting compatibility checker, embedded into distributor websites. Generates quotes online.  
**Features:** Compatibility rules engine, distributor branded subdomain, catalog/inventory visibility, analytics.  
**Gaps:** Designed for distributors of Danfoss products only. Not a general made-to-order assembly authoring platform. No recipe versioning. No internal authoring workflow. No margin-protected quoting.

### 4.2 GoldLeaf App
**URL:** https://goldleaf.app/  
**What it does:** Web-based hose assembly configurator for distributors and OEMs. Field technicians enter part numbers or descriptions; GoldLeaf converts to standard parameters from manufacturer catalogs.  
**Features:** Multi-manufacturer catalog, mobile-friendly, preloaded manufacturer data, quoting with labor/scrap rates, pricing models.  
**Gaps:** Generic catalog-lookup tool, not tailored to Unisource's specific 5-family recipe logic. No internal authoring/versioning of custom recipes. No ERP integration for export.

### 4.3 Tacton CPQ
**URL:** https://global.tacton.com/  
**What it does:** Enterprise CPQ platform for complex manufacturing (Gartner Magic Quadrant "Visionary"). Guided selling, real-time pricing, ERP/CRM integrations, 3D visualization.  
**Features:** Needs-based configuration, instant pricing, CAD generation, SAP/Oracle ERP integration, constraint-based product modeling.  
**Gaps:** Enterprise-scale, expensive, not pre-configured for hose/connector assembly logic. Requires extensive modeling effort. Overkill for Unisource's 5-family scope.

### 4.4 Revalize Configure One Cloud (CPQ)
**URL:** https://revalizesoftware.com/configure-one-cloud/  
**What it does:** 20+ years in complex manufacturing CPQ. Has fluid-handling vertical (Intelliquip Select for pumps/valves). ERP/CRM integration, 3D views, automated BOM generation.  
**Features:** Selection-first SCPQ, dynamic pricing, BOM explosion, ERP integration, compliance validation.  
**Gaps:** Pump/valve focused for fluid handling vertical; not pre-built for hose assembly recipe families. Requires significant configuration. Not sized for a mid-market Unisource deployment.

### 4.5 Cincom CPQ (Pumps & Valves)
**URL:** https://www.cincom.com/cpq/pumps-and-valves-manufacturing/  
**What it does:** CPQ for fluid handling manufacturers. Rules-based product configuration, BOM automation, ERP integration.  
**Features:** Technical configuration rules, BOM generation, quoting, ERP sync.  
**Gaps:** Again, general CPQ—not specific to hose/connector assembly families. No direct Tribute integration. Enterprise license cost.

**AutoBuilder's differentiation:** Purpose-built for Unisource's exact 5-family recipe logic. Parity-proven against Tribute output. Includes internal recipe authoring (not just customer-facing quoting). Priced and sized for a mid-market distribution division.

---

## 5. Assembly Families in Scope

1. **MTL-MC with insulation** — metal-cored, insulated flexible connectors
2. **Cryo no-armor** — cryogenic assemblies without armor
3. **Cryo with armor** — cryogenic assemblies with armor layer
4. **SF4500 standard** — standard SF4500 hose assembly
5. **SF4500 with armor** — SF4500 with armor layer

Each family has its own BOM expansion logic (parts recipe) that AutoBuilder must reproduce exactly to match Tribute output.

---

## 6. Features

### Must-Have (MVP Scope)

| # | Feature | Description |
|---|---------|-------------|
| F1 | Assembly Family Browser | List all 5 families; navigate into a family |
| F2 | Assembly Configurator | Step-by-step guided form to configure an assembly (select hose length, fittings, options) |
| F3 | BOM Expansion Engine | Apply recipe logic to produce a full bill of materials from configuration inputs |
| F4 | Recipe Authoring | Create/edit assembly recipes with field validation; diff preview before save |
| F5 | Version History | Every recipe change logged; revert to prior version |
| F6 | Quote Generator | Price a configured assembly; apply margin floor; produce PDF-ready quote |
| F7 | ERP Export | Export quote line items as structured CSV/JSON for Tribute/P21 ingestion |
| F8 | User Auth | Email/password login; role-based access (author vs sales rep vs manager) |
| F9 | Parity Validator | Run a part number against both AutoBuilder and the Tribute reference; diff output |

### Out of Scope (Later Phases)

- Live two-way Tribute/P21 API integration (read M1 only for discovery)
- Automated order submission to ERP
- Customer-facing portal
- Other Singer business units
- Mobile app

---

## 7. Screens / Primary User Journeys

### Screen Map

```
Login
  └─ Dashboard (family list)
       ├─ Configure Assembly (guided wizard)
       │    ├─ Step 1: Select family
       │    ├─ Step 2: Enter specs (length, fittings, options)
       │    ├─ Step 3: BOM Preview (expanded parts list)
       │    └─ Step 4: Quote (price + margin + export)
       └─ Recipe Library
            ├─ Browse assemblies
            ├─ Edit recipe (author role)
            │    ├─ Edit form
            │    ├─ Diff preview
            │    └─ Save → version logged
            └─ History (audit trail per recipe)
```

### Happy Flow (Playwright Gate)
1. Navigate to deployed URL → see login page
2. Log in with test credentials
3. See dashboard with 5 assembly families
4. Click "Configure Assembly" → select MTL-MC with insulation family
5. Enter specs (length, fitting type, insulation)
6. View generated BOM (parts list with quantities)
7. Click "Generate Quote" → see priced line items with margin
8. Click "Export" → download CSV
9. Navigate to Recipe Library → see list of assemblies
10. Click an assembly → view its version history

---

## 8. MVP Scope

Deliver a working web application that:
- Authenticates users (email/password, 2 roles: author + sales)
- Shows all 5 assembly families on a dashboard
- Lets a sales rep configure any family through a guided wizard and see a BOM + priced quote
- Lets an author create/edit recipes with validation and versioning
- Exports quotes as CSV
- All primary happy-flow steps above pass in Playwright

**Tech stack:** React frontend, Node.js/Express or FastAPI backend, PostgreSQL (Supabase), hosted on Railway.

---

## 9. Risks

| Risk | Mitigation |
|------|-----------|
| BOM expansion logic complexity | Seed with realistic but simplified recipe rules; parity test against reference data |
| Tribute integration discovery takes time | P21/Tribute integration is M2; M1 is export-only |
| Auth complexity | Use Supabase Auth (email/password) out of the box |
| Timeline pressure | 5-family scope is fixed; no new families in MVP |

---

## 10. Recommended Approach

**Backend:** Node.js + Express + TypeScript, deployed to Railway  
**Database + Auth:** Supabase (PostgreSQL + row-level security + email auth)  
**Frontend:** React + Vite + Tailwind CSS, served by the same backend (SSR or SPA)  
**Deployment:** Single Railway service `sf-run-48133f03`; Supabase managed DB  
**Parity testing:** Jest test suite comparing AutoBuilder BOM output against reference fixtures

---

*End of PRD*
