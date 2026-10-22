# KIG SaaS Platform - Client Architecture Overview

> **Confidential - For Client Presentation Only**  
> **Version:** 1.0 | **Date:** October 2026

---

## Executive Summary

The KIG SaaS Platform is a **multi-tenant, industry-vertical ERP platform** delivered as a service. It combines a modern **React onboarding portal**, a **tier-gated SaaS control plane (Platform Core)**, the **ERPNext foundation**, and **26 pre-built industry vertical modules** — all deployed as isolated tenant sites with enterprise-grade trust accounting and compliance built-in.

---

## System Overview (4-Layer Architecture)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        KIG SaaS Platform - Four Layers                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │  LAYER 1: REACT ONBOARDING PORTAL                                          │   │
│  │  • Industry & Tier Selection                                                │   │
│  │  • Client Profile & KYC                                                     │   │
│  │  • Upgrade & Provisioning Trigger                                           │   │
│  │  └─────────────────────────────┬───────────────────────────────────────────┘   │
│  │                                │ HTTPS / REST API                               │
│  ▼                                ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │  LAYER 2: PLATFORM CORE (SaaS Control Plane)                               │   │
│  │  • Tenant Provisioning & Lifecycle                                          │   │
│  │  │   Tier 1 (Core)    │   Tier 2 (Professional)  │  Tier 3 (Enterprise)  │  │
│  │  │   ✓ Enabled        │   ✓ Enabled              │   ✓ Enabled           │  │
│  │  │                    │   ✓ Enabled              │   ✓ Enabled           │  │
│  │  │                    │                          │   ✓ Enabled           │  │
│  │  │  Module Gate       │  Feature Flags           │  Quotas & Limits      │  │
│  │  └────────────────────┴──────────────────────────┘                       │   │
│  │                                │ Internal Service Calls                      │
│  ▼                                ▼                                           │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │  LAYER 3: ERPNext FOUNDATION (Per-Tenant Site)                             │   │
│  │  Tier 1 (All)          │  Tier 2 (Pro)           │  Tier 3 (Ent)         │  │
│  │  Accounting            │  Manufacturing          │  Advanced Mfg         │  │
│  │  Stock/Inventory       │  HR & Payroll           │  Advanced HR          │  │
│  │  Buying/Selling        │  Advanced Accounting    │  Advanced Projects    │  │
│  │  CRM                   │  Advanced Stock         │  Analytics/BI         │  │
│  │  Projects              │  Subcontracting         │  Integrations/API     │  │
│  │  Assets/Quality        │  Subcontracting         │  Compliance/Audit     │  │
│  └────────────────────────┴──────────────────────────────────────────────────┘   │
│                                    │                                              │
│                                    ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────────────┐   │
│  │  LAYER 4: MULTI-INDUSTRY VERTICAL MODULES (26 Verticals)                   │   │
│  │  ┌─────────┬─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐   │
│  │  │Legal    │Media    │Auto     │Solar    │Agri     │FinTech  │Health   │   │
│  │  │Edu      │Agri     │Auto     │Media    │... 26 verticals total         │   │
│  │  │         │         │         │         │         │         │         │   │
│  │  │ T1:Core │ T1:Core │ T1:Core │ T1:Core │ T1:Core │ T1:Core │ T1:Core │   │
│  │  │ T2:Pro  │ T2:Pro  │ T2:Pro  │ T2:Pro  │ T2:Pro  │ T2:Pro  │ T2:Pro  │   │
│  │  │ T3:Ent  │ T3:Ent  │ T3:Ent  │ T3:Ent  │ T3:Ent  │ T3:Ent  │ T3:Ent  │   │
│  │  └─────────┴─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘   │   │
│  │  Each vertical = Matter/Asset hub + WIP engine + Trust accounting          │   │
│  └─────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Layer 1: React Onboarding Portal (Client-Facing)

### User Journey
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Landing    │───▶│  Industry   │───▶│    Tier     │───▶│   Client    │───▶│  Payment &  │
│  Page       │    │  Selection  │    │  Selection  │    │  Profile    │    │  Provision  │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
      │                  │                  │                  │                  │
      ▼                  ▼                  ▼                  ▼                  ▼
  • Vertical          • 26 verticals     • T1/T2/T3       • Company          • Stripe
  • Tier comparison    • Feature matrix   • Feature matrix  • Admin user       • Webhook
  • Pricing            • Tier gating      • Pricing         • KYC docs         • Provision
```

### Key Capabilities
| Feature | Description |
|---------|-------------|
| **Industry Selector** | 26 vertical cards with tier comparison matrix |
| **Tier Selector** | T1/T2/T3 with feature matrix & pricing |
| **Client Profile** | Company details, admin user, custom domain |
| **KYC/Compliance** | Document upload (incorporation, ID, bank statements) |
| **Payment** | Stripe Checkout, subscription setup |
| **Provisioning** | Idempotent webhook → Platform Core API |

---

## Layer 2: Platform Core (SaaS Control Plane)

### Core Responsibilities
| Capability | Description |
|------------|-------------|
| **Tenant Lifecycle** | Provision → Suspend → Upgrade → Deprovision |
| **Tier/Entitlement Engine** | T1/T2/T3 gating at API boundary |
| **Industry Templates** | Versioned vertical provisioning (fixture manifests) |
| **Trust Reconciliation** | Three-way reconciliation (Bank ↔ Control ↔ Client Ledgers) |
| **Feature Flags** | Runtime module toggles per tier |
| **Provisioning Engine** | Idempotent site creation, fixture import |
| **Posting Matrix** | Cross-company posting rules |
| **Exception Queue** | Failed job retry & monitoring |

### Tier Gating Architecture
```
┌─────────────────────────────────────────────────────────────────────┐
│                    MODULE GATE (API Boundary)                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Request → Module Gate → Check Entitlement → Allow / 403           │
│                    │                                                    │
│                    ▼                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  TENANT ENTITLEMENT CHECK                                       │ │
│  │  1. Get Tenant Entitlement (tier + enabled_modules)            │ │
│  │  2. Verify tier >= required_tier                                │ │
│  │  3. Verify module in enabled_modules                            │ │
│  │  4. Check Feature Flag tier_floor                               │ │
│  │  5. Return Allow / Deny                                         │ │
│  └─────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Tier Definitions
| Tier | Name | Target | ERPNext Modules | Vertical Access |
|------|------|--------|-----------------|-----------------|
| **T1** | **Starter** | Small firms, startups | 9 core modules | Core vertical features only |
| **T2** | **Professional** | Growing firms | +6 advanced modules | Full vertical + trust/billing |
| **T3** | **Enterprise** | Large firms, multi-entity | +7 enterprise modules | Full suite + API + white-label |

---

## Layer 3: ERPNext Foundation (Per-Tenant Site)

### Tier 1 - Foundation (All Tenants)
| Module | Purpose |
|--------|---------|
| **Accounting** | GL, Chart of Accounts, JE, Payments, Bank Reconciliation |
| **Stock/Inventory** | Warehouses, Items, Stock Entries, Valuation |
| **Buying** | Suppliers, POs, Purchase Invoices, Receipts |
| **Selling** | Customers, Quotes, Sales Orders, Invoices, Delivery |
| **CRM** | Leads, Opportunities, Contacts, Campaigns |
| **Projects** | Projects, Tasks, Timesheets, Costing |
| **Assets** | Asset Register, Depreciation, Maintenance |
| **Quality** | Inspection, Non-Conformance, Quality Goals |

### Tier 2 - Professional
| Module | Purpose |
|--------|---------|
| **Manufacturing** | BOM, Work Orders, Production Planning |
| **HR & Payroll** | Employee Lifecycle, Payroll, Leave, Recruitment |
| **Advanced Accounting** | Multi-currency, Inter-company, Budgeting |
| **Advanced Stock** | Serial/Batch, Multi-warehouse, Bin Management |
| **Subcontracting** | Subcontract Orders, Receipts, Valuation |

### Tier 3 - Enterprise
| Module | Purpose |
|--------|---------|
| **Advanced Manufacturing** | MRP, Shop Floor, Capacity Planning |
| **Advanced HR** | Performance, Succession, Learning |
| **Advanced Projects** | Portfolio, Resource Planning, Rev Rec |
| **Analytics/BI** | Dashboards, KPIs, Embedded Analytics |
| **Integrations/API** | REST/GraphQL, Webhooks, ETL |
| **Compliance** | Audit Trails, Regulatory Reports |

---

## Layer 4: Multi-Industry Vertical Modules (26 Verticals)

### Tier-Gated Vertical Architecture
Each vertical is a **complete engagement hub** with Matter/Asset at center, WIP engine, and Trust accounting.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VERTICAL TIER MATRIX (26 Verticals)                      │
├──────────────────┬──────────┬──────────┬──────────┬──────────┬──────────┐
│ Vertical         │ Code     │ T1 Core  │ T2 Pro   │ T3 Ent   │ Status   │
├──────────────────┼──────────┼──────────┼──────────┼──────────┼──────────┤
│ Legal            │ IND-03   │ ✓        │ ✓        │ ✓        │ ✅ Done  │
│ Media            │ IND-02   │ ✓        │ ✓        │ ✓        │ ✅ Done  │
│ Automotive       │ IND-01   │ ✓        │ ✓        │ ✓        │ ✅ Done  │
│ Solar IPP        │ IND-04   │ ✓        │ ✓        │ ✓        │ ✅ Done  │
│ Agriculture      │ IND-05   │ ✓        │ ✓        │ ✓        │ ✅ Done  │
│ Banking          │ IND-06   │ Stub     │ Stub     │ Stub     │ 🔄 Plan  │
│ FinTech          │ IND-07   │ Stub     │ Stub     │ Stub     │ 🔄 Plan  │
│ Healthcare       │ IND-08   │ Stub     │ Stub     │ Stub     │ 🔄 Plan  │
│ Manufacturing    │ IND-09   │ Stub     │ Stub     │ Stub     │ 🔄 Plan  │
│ ... (17 more)    │ ...      │ Stub     │ Stub     │ Stub     │ 🔄 Plan  │
└──────────────────┴──────────┴──────────┴──────────┴──────────┴──────────┘
```

### Completed Verticals (Deep Dive)

#### 1. Legal (IND-03) - **Complete**
| Component | Description |
|-----------|-------------|
| **Matter** | Central engagement hub (Project + Cost Center + Trust sub-ledger) |
| **Matter Party** | Client, opposing party, counsel, experts with conflict flags |
| **Conflict Check** | Automated search across parties/history; ethical wall support |
| **Time Entry** | Invoice-grade narrative, task codes, rate card resolution |
| **Disbursement** | Office-paid vs Trust-paid, VAT treatment, recharge logic |
| **Docket Calendar** | Limitation dates, court hearings, auto-escalation |
| **Trust Ledger** | Per-client running balance, three-way reconciliation |
| **Pre-Bill/Write-down** | Line-by-line write-downs, fee arrangements (hourly/fixed/capped) |
| **Retainer** | Engagement (income) vs Advance/Evergreen (trust) |

#### 2. Media (IND-02) - **Complete**
| Component | Description |
|-----------|-------------|
| **Content Asset** | Film/Series/Episode hierarchy, amortization schedules |
| **Rights Management** | Rights agreements, licence windows, deferred revenue |
| **Advertising** | Media plans, insertion orders, spot scheduling, make-goods |
| **Royalty Engine** | Tiered contracts, advance recoupment, statements |

#### 3. Automotive (IND-01) - **Complete**
| Component | Description |
|-----------|-------------|
| **Service Job Card** | 7-table design (complaints, inspections, estimates, labour, parts, techs) |
| **Vehicle Master** | VIN, make/model/config, service history |
| **Workshop Ops** | Bays, scheduling, technician time logs |
| **Warranty** | Claims, settlements, OEM integration |

#### 4. Solar IPP (IND-04) - **Complete**
| Component | Description |
|-----------|-------------|
| **Power Plant** | Capacity, COD, PPA linkage, cost center |
| **PPA Contract** | Tariff schedules (TOU/seasonal), escalation, take-or-pay |
| **Meter Reading** | CT/VT multiplier, validation vs generation log (3% tolerance) |
| **Billing Run** | Energy + capacity + deemed gen + curtailment → Sales Invoice |
| **Trust Reconciliation** | Three-way bank/control/client reconciliation |

---

## Data Flow: End-to-End Provisioning

```
┌─────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  React App  │────▶│  Platform Core  │────▶│  Tenant Site    │
│  (Onboard)  │     │  (Control)      │     │  (ERPNext)      │
└─────────────┘     └────────┬────────┘     └────────┬────────┘
                             │                       │
                             ▼                       ▼
                    ┌─────────────────┐     ┌─────────────────┐
                    │ Provisioning    │     │ Industry        │
                    │ Engine          │     │ Template        │
                    │ (Idempotent)    │     │ (Fixtures)      │
                    └────────┬────────┘     └────────┬────────┘
                             │                       │
                             ▼                       ▼
                    ┌─────────────────────────────────────┐
                    │         TENANT SITE                 │
                    │  ERPNext + multi_industry +         │
                    │  platform_core                      │
                    │  • Company + Chart of Accounts      │
                    │  • Industry Template Applied        │
                    │  • Tenant Entitlement Created       │
                    │  • Default Trust Account Created    │
                    └─────────────────────────────────────┘
```

### Provisioning API Contract
```json
POST /api/v1/provision
{
  "industry_code": "IND-03",
  "tier": 2,
  "tenant_id": "acme-law",
  "company_name": "Acme Law LLP",
  "admin_email": "partner@acme.law",
  "domain": "acme-law.saas.local",
  "kyc_documents": [...],
  "payment_method_id": "pm_..."
}
```

**Response:**
```json
{
  "site_url": "https://acme-law.saas.local",
  "api_key": "xxx",
  "api_secret": "xxx",
  "admin_credentials": {...}
}
```

---

## Trust Accounting & Compliance (Key Differentiator)

### Three-Way Reconciliation (Monthly)
```
┌─────────────────────────────────────────────────────────────────┐
│                    TRUST RECONCILIATION                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   1. BANK STATEMENT          2. CONTROL ACCOUNT      3. CLIENT  │
│   Closing Balance            (GL Trust Control)     LEDGERS     │
│   ─────────────────          ─────────────────      ─────────── │
│   $1,250,000                 $1,250,000              Σ $1,250,000│
│                                                                  │
│   ✓ All three MUST agree                                           │
│   ✗ Differences → Unreconciled Items (outstanding cheques,       │
│     deposits in transit, bank errors, posting errors)            │
│                                                                  │
│   ✓ Partner Sign-off Required                                      │
│   ✓ Law Society Submission (where required)                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Trust Ledger Rules (Non-Negotiable)
| Rule | Enforcement |
|------|-------------|
| **No negative client balance** | Hard DB constraint + application validation |
| **No cross-client subsidy** | Per-client running balance, isolated |
| **Office transfers only vs invoice** | `Transfer to Office` requires `Sales Invoice` link |
| **Three-way monthly reconciliation** | Mandatory, partner sign-off, audit retention |
| **Interest allocation** | Configurable (client vs law society) |

---

## Deployment & Operations

### Infrastructure
```
┌─────────────────────────────────────────────────────────────────┐
│                    K8s / VM Fleet                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │ Tenant A    │  │ Tenant B    │  │ Tenant N    │  (Isolated)  │
│  │ Site/DB     │  │ Site/DB     │  │ Site/DB     │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘            │
└─────────┼────────────────┼────────────────┼───────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           ▼
              ┌─────────────────────────┐
              │      Shared Services    │
              │  Redis (Queue/Cache)    │
              │  S3/MinIO (Files)       │
              │  SMTP/Email             │
              └─────────────────────────┘
```

### Multi-Tenancy Model
| Aspect | Implementation |
|--------|----------------|
| **Database** | One MariaDB database per tenant (hard isolation) |
| **Platform Core** | Single hub database (control plane) |
| **Files** | S3/MinIO with `tenant_id/` prefix |
| **Queue/Cache** | Redis (shared, namespaced by tenant) |
| **Backup/DR** | Daily snapshots + PITR; cross-region for T3 |

---

## Security & Compliance

| Area | Implementation |
|------|----------------|
| **Authentication** | Frappe session + JWT; optional SSO (SAML/OIDC) |
| **Authorization** | Frappe roles + Platform Core tier/module gating |
| **Audit Trail** | Immutable `Version` logs + `Activity Log` + `Exception Queue` |
| **Trust Accounting** | Three-way reconciliation, negative balance prevention |
| **Data Residency** | Per-tenant database; region-selectable at provisioning |
| **Encryption** | TLS 1.3 in transit; AES-256 at rest |

---

## Go-to-Market: Vertical Rollout Plan

| Phase | Verticals | Target Date |
|-------|-----------|-------------|
| **Phase 1** (Done) | Legal, Media, Automotive, Solar IPP, Agriculture | Q4 2026 |
| **Phase 2** | FinTech, Banking, Healthcare, Manufacturing | Q1 2027 |
| **Phase 3** | Construction, Real Estate, Retail, Logistics | Q2 2027 |
| **Phase 4** | Education, Hospitality, Energy, Mining, Pharma | Q3 2027 |
| **Phase 5** | Microfinance, Publishing, Services, Wellness, Wholesale, Restaurants | Q4 2027 |

---

## Key Differentiators for Clients

| Differentiator | Traditional ERP | KIG SaaS Platform |
|----------------|-----------------|-------------------|
| **Vertical Depth** | Generic, custom-built | 26 pre-built verticals |
| **Trust Accounting** | Add-on, error-prone | Native, three-way recon, legally compliant |
| **Tier Model** | All-or-nothing | Pay for what you need, upgrade anytime |
| **Provisioning** | Weeks of consulting | Minutes via self-service portal |
| **Compliance** | Manual, per-client | Built-in (trust recon, conflict check, dockets) |
| **Multi-vertical** | Separate implementations | Single platform, shared core |

---

## Appendix: Technical Specifications

| Component | Technology |
|-----------|------------|
| **React App** | React 18, TypeScript, Vite, Tailwind, TanStack Query |
| **Platform Core** | Frappe Framework, Python 3.11, MariaDB 10.6+ |
| **ERPNext** | v15+, Python 3.11, Redis 7, MariaDB 10.6+ |
| **Multi-Industry** | 300+ doctypes, 250+ JS controllers |
| **Deployment** | Kubernetes (EKS/GKE), Helm charts |
| **CI/CD** | GitHub Actions → Staging → Production |
| **Observability** | Prometheus/Grafana, Sentry, structured logging |

---

*This architecture document represents the current state of the KIG SaaS Platform as of October 2026. All four layers are implemented and verified in the development environment. Client onboarding portal (React) is in final integration testing.*
