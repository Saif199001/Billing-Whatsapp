# Cafe Billing with WhatsApp

A production-oriented cafe billing platform with WhatsApp customer communication.

## Project goals

- Fast cafe POS and billing workflow
- Product/menu and inventory foundations
- Orders, payments, taxes and receipts
- Customer records and purchase history
- WhatsApp-based receipt and customer communication
- Multi-tenant SaaS-ready architecture
- Secure, testable and API-first backend

## Initial architecture

The project starts as a modular Django backend with Django REST Framework and PostgreSQL as the primary database. WhatsApp integration is isolated behind a provider/service boundary so billing remains independent from external messaging providers.

### Planned modules

- `core` — shared primitives, configuration and health checks
- `accounts` — authentication and users
- `businesses` — cafe/business and workspace tenancy
- `catalog` — products, categories and pricing
- `orders` — bills, line items and order lifecycle
- `payments` — payment methods and transaction records
- `customers` — customer profiles and history
- `inventory` — stock tracking and adjustments
- `whatsapp` — messaging provider integration and delivery records
- `reports` — operational and sales reporting

## Development principles

1. Foundation before feature expansion.
2. Keep domain logic out of views/controllers.
3. Preserve tenant isolation from the first business model.
4. External integrations must be replaceable behind service/provider interfaces.
5. Every important business rule gets contract-level tests.
6. Never commit secrets; use environment variables.

## Status

**Phase 1 — Project Foundation: started**

The repository is intentionally being built incrementally. Product features will be added only after the foundation, configuration, testing and deployment structure are in place.
