# Cafe Billing with WhatsApp

A production-oriented, offline-first cafe transaction platform with built-in customer intelligence and WhatsApp customer growth workflows.

## Product thesis

**Every bill is a customer signal. Turn transaction history into intelligent actions that increase repeat revenue.**

Core loop:

`CAPTURE → UNDERSTAND → ACT → RETURN`

## Locked strategy

- Cafe-focused, independent/growing-cafe ICP
- Offline-first POS and transaction architecture
- Multi-tenant SaaS-ready backend
- First-party customer graph built from transactions
- Consent/privacy as first-class concerns
- Rule-based customer intelligence before ML
- Explainable next-best-action recommendations
- WhatsApp as an engagement channel behind a provider boundary
- Revenue attribution as a growth metric
- Future support for external POS integrations
- No generic feature-bloat strategy

See [`docs/MARKET_TRUTH.md`](docs/MARKET_TRUTH.md) for the locked market conclusion and [`docs/BLUEPRINT.md`](docs/BLUEPRINT.md) for the implementation blueprint.

## Initial architecture

The project starts as a modular Django backend with Django REST Framework and PostgreSQL as the primary database. WhatsApp integration is isolated behind a provider/service boundary so billing remains independent from external messaging providers.

### Planned modules

- `core` — shared primitives, configuration and health checks
- `accounts` — authentication and users
- `businesses` — cafe/business and workspace tenancy
- `catalog` — products, categories and pricing
- `orders` — bills, line items and order lifecycle
- `payments` — payment methods and transaction records
- `customers` — customer profiles, consent and purchase history
- `inventory` — stock tracking and adjustments
- `whatsapp` — messaging provider integration and delivery records
- `intelligence` — lifecycle, segmentation and next-best-action rules
- `growth` — campaigns and revenue attribution
- `reports` — operational, sales and customer-growth reporting
- `integrations` — future external POS/import/integration boundaries

## Development principles

1. Foundation before feature expansion.
2. Keep domain logic out of views/controllers.
3. Preserve tenant isolation from the first business model.
4. External integrations must be replaceable behind service/provider interfaces.
5. Offline transactions must be idempotent and safely synchronizable.
6. Every important business rule gets contract-level tests.
7. Never commit secrets; use environment variables.
8. Do not add features outside the locked blueprint without explicit strategy review.

## Status

**Market Truth v1.0 — LOCKED**  
**Product Blueprint v1.0 — LOCKED**  
**Next: Phase 1 — Foundation implementation**

The repository is intentionally being built incrementally. Implementation follows:

`Architecture → Models → Services/Providers → API → Tests → Real Integration → Green`
