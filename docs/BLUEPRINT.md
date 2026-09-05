# Cafe Billing + Customer Growth Platform — Blueprint v1.0

**Status:** LOCKED  
**Locked on:** 2026-09-05  
**Market Truth:** `docs/MARKET_TRUTH.md`

## 1. Product definition

This product is an **offline-first cafe transaction platform with a built-in customer intelligence and retention engine**.

It is not a generic restaurant ERP, a WhatsApp-only product, or a clone of an existing POS.

### Product promise

> **Run the cafe simply, understand customers automatically, and turn repeat-customer opportunities into measurable revenue.**

### Core loop

`CAPTURE → UNDERSTAND → ACT → RETURN`

## 2. Product architecture at a glance

```text
POS / Order Capture
        ↓
Transaction Engine
        ↓
Customer Identity + Consent
        ↓
Customer Graph
        ↓
Behavior / Lifecycle Engine
        ↓
Next Best Action
        ↓
Engagement Providers (WhatsApp first)
        ↓
Customer Response / Visit
        ↓
Revenue Attribution
        ↓
Learning / Analytics
```

The architecture must also allow:

```text
External POS → Integration Layer → Customer Intelligence
```

at a later phase.

## 3. Core engineering principles

1. **Offline-first:** billing must continue during internet outages.
2. **Transaction integrity first:** never sacrifice bill/payment correctness for convenience.
3. **Multi-tenancy from day one:** every business-owned resource must be tenant scoped.
4. **Domain logic outside controllers/views:** use domain services/use-cases.
5. **Provider isolation:** WhatsApp and other external services sit behind interfaces/adapters.
6. **API-first:** frontend/POS clients communicate through stable contracts.
7. **Idempotency:** retries must not duplicate bills, payments, messages or sync operations.
8. **Auditability:** critical business and consent changes are traceable.
9. **Privacy-by-design:** collect only what is needed and record consent purposefully.
10. **Exportability:** business data must be portable.
11. **Test contracts, not implementation trivia:** important rules require cohesive contract-level tests.
12. **No scope drift:** new modules/features require explicit strategy approval.

## 4. Domain/module map

### Foundation

- `core`
- configuration/settings
- health/readiness checks
- shared IDs/time utilities
- audit primitives
- error handling

### Identity & tenancy

- `accounts`
- `businesses`
- users
- roles/permissions
- businesses/outlets
- tenant isolation

### Catalog

- `catalog`
- categories
- products
- variants
- modifiers/add-ons
- prices
- tax configuration
- product availability

### Transactions

- `orders`
- bills/orders
- line items
- discounts
- order types
- KOT lifecycle
- refunds/voids
- bill numbering

### Payments

- `payments`
- cash
- UPI
- card
- split payments
- payment transaction records
- reconciliation primitives

### Customers

- `customers`
- identity
- customer profiles
- customer merge/deduplication
- transaction history
- consent
- opt-out
- customer preferences derived from behavior

### Inventory

- `inventory`
- stock items
- stock movements
- adjustments
- low-stock rules
- item/product mapping
- future recipe/BOM support

### WhatsApp / engagement

- `whatsapp`
- provider abstraction
- templates
- outbound messages
- inbound webhooks
- delivery/read/failure statuses
- consent enforcement
- rate/frequency controls

### Intelligence

- `intelligence`
- customer lifecycle
- segmentation
- expected return interval
- overdue ratio
- customer value metrics
- opportunity detection
- recommendation rules
- next-best-action engine

### Growth

- `growth`
- campaigns
- audience snapshots
- campaign approvals
- message execution
- offer/reward references
- attribution
- control groups/experiments later

### Reporting

- `reports`
- sales
- payments
- product performance
- customer metrics
- retention
- campaign revenue
- operational KPIs

### Integration

- `integrations`
- external POS ingestion
- import/export
- webhooks
- future third-party POS adapters

## 5. MVP scope

### 5.1 Business setup

- business registration
- outlet creation
- user/role basics
- business settings
- tax configuration
- timezone/currency configuration

### 5.2 Catalog

- categories
- products
- variants
- modifiers
- prices
- availability
- basic tax mapping

### 5.3 Billing/POS

- fast bill creation
- dine-in
- takeaway
- order status
- line-item discounts
- bill-level discounts
- taxes
- bill numbering
- receipt generation
- hold/resume where required by workflow
- void/refund permissions

### 5.4 Payments

- cash
- UPI
- card
- split payment foundation
- payment status
- duplicate-payment protection

### 5.5 KOT

- create KOT
- kitchen status
- basic print/output abstraction
- bill/KOT linkage

Deep KDS is later.

### 5.6 Customer capture

Customer can be attached to a transaction by:

- phone lookup
- new customer creation
- QR/customer self-entry flow later in MVP expansion

The billing path must remain fast when customer identification is skipped.

### 5.7 Consent

Support separate consent records for relevant purposes, including:

- digital receipts
- service/order communication
- loyalty communication
- marketing communication

Record:

- purpose
- status
- timestamp
- source
- policy/version reference
- revoked timestamp where applicable

### 5.8 Customer profile

Initial derived metrics:

- first visit
- last visit
- visit count
- total spend
- average order value
- favourite categories/items
- expected return interval
- days since last visit
- lifecycle stage

### 5.9 Intelligence v1

Rule-based only.

Initial lifecycle:

`NEW → SECOND-VISIT OPPORTUNITY → ACTIVE → LOYAL → AT-RISK → LAPSED`

Initial opportunity rules must be explainable and inspectable.

### 5.10 WhatsApp v1

- provider abstraction
- webhook endpoint
- outbound message service
- template reference
- receipt message flow
- approved marketing/utility workflow as applicable
- delivery/failure status storage
- opt-out handling
- frequency guard

No bulk-blast-first design.

### 5.11 Basic growth actions

Initial recommended actions:

- thank-you
- feedback request
- second-visit nudge
- win-back
- birthday/occasion support only when reliable data exists

The system should recommend an action; automatic sending must be governed by consent and configured policy.

### 5.12 Dashboard

Owner should see:

- today's sales
- orders
- average order value
- payment breakdown
- top products
- known customers
- new customers
- at-risk customers
- second-visit opportunities
- recent retention actions

## 6. Offline-first architecture

The POS client must maintain a local transaction store.

```text
User action
   ↓
Local transaction
   ↓
Local validation
   ↓
Local receipt/KOT
   ↓
Sync queue
   ↓
Server API
   ↓
Idempotency check
   ↓
Server persistence
   ↓
Acknowledgement
   ↓
Local sync state
```

Requirements:

- unique client-generated transaction IDs
- idempotency keys
- ordered/causal sync where required
- retry with backoff
- conflict detection
- explicit sync states
- no silent data loss
- server reconciliation tools
- audit trail

The exact client technology can be selected during architecture implementation; the contract must remain technology-independent.

## 7. Customer graph design

Customer identity is separate from transaction records.

Core relationship:

```text
Customer
  ├── identities
  ├── consents
  ├── transactions
  ├── visits
  ├── preferences/derived signals
  ├── lifecycle state
  └── engagement history
```

Phone number must not be the only conceptual identity field even if it is the primary initial matching key.

Customer merge/deduplication must be supported before advanced analytics are trusted.

## 8. Intelligence engine v1

No opaque AI dependency in the initial version.

### Expected return interval

Estimate from historical visit gaps when enough history exists.

### Overdue ratio

Conceptually:

`days_since_last_visit / expected_return_interval`

### Example rules

- one visit and no repeat → second-visit opportunity
- customer materially beyond expected interval → at-risk
- prolonged inactivity after prior activity → lapsed
- high frequency + high spend → loyal/high-value signal

Rules must include minimum-data thresholds so a new customer is not falsely classified.

## 9. Next Best Action engine

Input:

- lifecycle
- customer value
- recency
- frequency
- product affinity
- prior campaign response
- consent
- communication frequency
- business configuration

Output:

- no action
- feedback request
- second-visit nudge
- win-back
- product recommendation
- loyalty/reward action

The recommendation must include an explainable reason.

Example:

> Customer normally returns every ~10 days and is now 28 days since last visit.

## 10. WhatsApp provider boundary

Business logic must not directly call Meta APIs.

Conceptually:

```text
EngagementService
      ↓
WhatsAppProvider interface
      ↓
WhatsApp Cloud API adapter
```

Provider records must support:

- provider message ID
- template ID/name/version where applicable
- recipient reference
- status
- timestamps
- failure code/details
- business/tenant reference

Provider credentials must never be stored in normal application records or source control.

## 11. Campaign architecture

A campaign is not merely a message list.

```text
Campaign
 ├── audience definition
 ├── audience snapshot
 ├── eligibility checks
 ├── consent checks
 ├── frequency checks
 ├── message/template
 ├── execution records
 └── attribution records
```

Audience must be snapshotted at execution time so later customer changes do not silently rewrite historical campaign membership.

## 12. Revenue attribution

Initial attribution can use conservative deterministic rules.

Every campaign should be able to connect:

`campaign → customer → subsequent qualifying transaction`

Future versions may add control groups and incremental-lift estimation.

We must clearly distinguish:

- attributed revenue
- estimated incremental revenue
- total customer revenue

They are not interchangeable.

## 13. Inventory v1

Keep inventory focused.

Support:

- stock item
- unit
- current quantity
- stock movement
- manual adjustment
- low-stock threshold
- product-to-stock mapping where practical

Recipe/BOM and automated ingredient deduction are later milestones unless pilot customers prove they are essential for the initial ICP.

## 14. Tax/GST architecture

Tax configuration must be data-driven.

Do not hard-code a single restaurant GST rate.

Support the foundations for:

- tax-inclusive/exclusive pricing
- intra/interstate tax components as applicable
- product tax classification
- effective dates
- invoice tax breakdown
- future e-invoice integration boundary

Compliance rules must be validated against current official requirements before production release.

## 15. Security and tenancy

Required from the foundation:

- authenticated APIs
- role-based authorization
- tenant-scoped query enforcement
- object-level authorization
- secure secrets/configuration
- audit logging for critical operations
- rate limiting on public/webhook endpoints
- webhook signature verification where supported
- idempotency on externally retried operations
- safe file/receipt handling
- secure deletion/retention processes

A user must never be able to access another business's customer, transaction, inventory or campaign data.

## 16. API structure

Use versioned APIs.

Suggested boundaries:

- `/api/v1/auth/`
- `/api/v1/businesses/`
- `/api/v1/catalog/`
- `/api/v1/orders/`
- `/api/v1/payments/`
- `/api/v1/customers/`
- `/api/v1/inventory/`
- `/api/v1/whatsapp/`
- `/api/v1/intelligence/`
- `/api/v1/growth/`
- `/api/v1/reports/`
- `/api/v1/integrations/`

Exact endpoint contracts are implementation deliverables and must be documented before client integration.

## 17. Testing strategy

Follow cohesive contract-level testing.

### Must test

- tenant isolation
- authentication/authorization
- bill calculations
- tax calculations
- discounts
- payment idempotency
- refund/void permissions
- KOT linkage
- customer matching/deduplication
- consent enforcement
- WhatsApp provider contract
- webhook verification/idempotency
- campaign audience eligibility
- frequency controls
- intelligence rules
- attribution logic
- offline sync contracts
- conflict handling

Tests must be green before moving a domain to the next integration stage.

## 18. Development workflow

For every major domain:

`Architecture → Models → Domain Services/Provider → API → Tests → Real Integration → Green`

Do not implement UI-driven shortcuts that bypass domain contracts.

Do not add features because competitors have them; add them only when justified by the locked ICP/problem or validated customer evidence.

## 19. Implementation phases

### Phase 0 — Locked strategy

- Market Truth v1.0
- Blueprint v1.0
- ICP and product thesis locked

### Phase 1 — Foundation

- Django project structure
- environment/configuration
- PostgreSQL
- API foundation
- health checks
- logging/error handling
- test infrastructure
- CI
- deployment baseline

### Phase 2 — Tenancy & identity

- accounts
- businesses/outlets
- roles
- permissions
- tenant isolation

### Phase 3 — Catalog & tax

- categories
- products
- variants/modifiers
- pricing
- tax rules

### Phase 4 — Transaction engine

- orders/bills
- line items
- discounts
- taxes
- numbering
- refunds/voids
- KOT

### Phase 5 — Payments

- payment methods
- payment records
- split payment
- idempotency
- reconciliation foundation

### Phase 6 — Customer graph

- customer profiles
- phone matching
- transaction linkage
- merge/deduplication
- consent model

### Phase 7 — Offline POS/sync

- local store contract
- sync queue
- idempotency
- retry
- conflict handling
- reconciliation

### Phase 8 — WhatsApp foundation

- provider interface
- Cloud API adapter
- webhook handling
- templates
- receipt flow
- delivery status
- consent/frequency enforcement

### Phase 9 — Customer intelligence v1

- lifecycle
- expected return interval
- overdue ratio
- segmentation
- explainable opportunity rules
- next-best-action engine

### Phase 10 — Growth engine

- campaigns
- audience snapshots
- approvals
- execution
- attribution
- basic ROI reporting

### Phase 11 — Pilot hardening

- real cafe workflow validation
- printer/device validation
- offline failure scenarios
- performance
- security review
- backup/recovery
- observability
- support/admin tools

### Phase 12 — Expansion

Only after pilot validation:

- QR customer capture
- QR ordering
- deeper inventory/recipes
- advanced loyalty
- multi-outlet analytics
- external POS integrations
- control groups/experimentation
- predictive ML
- AI next-best-action optimization
- additional messaging channels

## 20. Explicit non-goals for v1

- enterprise restaurant ERP
- payroll
- full accounting replacement
- franchise management
- delivery fleet management
- deep aggregator integrations
- giant KDS suite
- generic chatbot
- generic AI content generator
- autonomous marketing without controls
- complex loyalty mechanics before core retention is proven

## 21. Definition of Done for MVP

MVP is not complete merely because screens work.

It is complete when a pilot cafe can:

1. configure its business and menu;
2. create bills quickly;
3. accept supported payments;
4. operate through an internet interruption without losing transactions;
5. sync safely after reconnection;
6. optionally identify customers during billing;
7. maintain auditable consent;
8. generate customer history and lifecycle signals;
9. receive explainable retention opportunities;
10. send compliant WhatsApp communications through the provider boundary;
11. see basic resulting customer/revenue metrics;
12. operate with correct tenant isolation and permissions;
13. recover from normal operational failures without data corruption.

## 22. Product success metrics

Pilot metrics:

- bill completion time
- billing failure rate
- offline transaction success rate
- sync success/conflict rate
- known-customer capture rate
- WhatsApp opt-in rate
- second-visit conversion
- at-risk customer recovery
- repeat revenue
- revenue per communication cost
- owner weekly active usage
- support incidents per outlet

The product should optimize for **repeat revenue and operational reliability**, not feature count.

## 23. Locked decisions

The following are locked for v1:

- cafe-focused ICP
- offline-first architecture
- multi-tenant SaaS architecture
- transaction engine as source of customer signals
- customer graph as a first-class domain
- consent as a first-class domain
- WhatsApp behind provider abstraction
- rule-based intelligence before ML
- explainable next-best-action
- revenue attribution as a core growth metric
- data exportability/privacy-by-design
- external POS integration as a future capability
- no generic feature-bloat strategy

## 24. Change control

This blueprint is locked.

A proposed change that affects:

- target customer
- product thesis
- core architecture
- offline strategy
- tenancy/security model
- customer-data model
- WhatsApp provider boundary
- MVP scope

must be explicitly reviewed and recorded before implementation.

Normal implementation details may evolve without changing the blueprint when they preserve the locked contracts and principles.
