# Lean API – KSA Postman Collection

This collection lets you explore Lean's Open Banking APIs for Saudi Arabia (KSA) without writing code. It mirrors the journey a real integration follows: authenticate, register a customer, connect their bank as an entity, then pull data or run verifications.

## Overview

Lean is a financial data connectivity platform that lets your application access a customer's bank data — accounts, transactions, balances, identity — after the customer grants consent through a SAMA‑compliant flow.

The typical journey is:

1. **Authenticate** — get an API access token from Lean's OAuth server.
2. **Create a Customer** — register the end user in Lean.
3. **Connect an Entity** — the customer links a bank through the LinkSDK; Lean creates an *entity* and notifies you by webhook (`entity.created`).
4. **Consent** — the customer authorizes specific permissions and date ranges on Lean's (or your own) consent screen.
5. **Fetch Data** — call the Raw Data v2 endpoints with the `entity_id` (and `account_id` where required).
6. **Refresh / Verify** — optionally trigger data refreshes or run verification APIs (IBAN, business, freelancer, beneficiary name).

See the full docs: [Overview](https://docs.leantech.me/v2.0-KSA/docs/overview) · [Creating a Customer](https://docs.leantech.me/v2.0-KSA/docs/creating-a-customer) · [Creating an Entity](https://docs.leantech.me/v2.0-KSA/docs/creating-an-entity) · [The Consent](https://docs.leantech.me/v2.0-KSA/docs/explaining-the-consent) · [Getting Started with Data](https://docs.leantech.me/v2.0-KSA/docs/getting-started-with-data).

## Before you start

Set the collection variables (`client_id`, `client_secret`, `baseUrl`, `auth_baseUrl`) from your [KSA Lean Dashboard](https://dev.sa.leantech.me/). See the [top‑level README](../README.md) for setup details.

---

## Requests

### Authentication

Token endpoints used by every other call. Run one of these first; the returned `access_token` is plugged into the `Authorization: Bearer` header of subsequent requests.

- **POST Generate API Access Token** — Server‑to‑server token used for almost all backend calls (customers, entities, data, verifications).
- **POST Generate Customer Access Token** — Customer‑scoped token used when the LinkSDK or another client acts on behalf of a single customer.

### Customers

A *Customer* is the foundational object representing one of your end users inside Lean. It must exist before any entity, consent, or data call.

- **POST Create Customer** — Registers a new customer with your `app_user_id`. Lean returns a `customer_id` to store on your side.
- **GET List Customers** — Paginated list of all customers under your application. Useful for sanity checks during development.
- **GET Get Customer by app_user_id** — Looks up a customer using the identifier you assigned.
- **GET Get Customer by ID** — Looks up a customer using Lean's `customer_id`.

### Entities

An *Entity* is the connection between a customer and a specific bank. Entities aren't created via API directly — they're produced when the customer completes the LinkSDK flow and consents. These endpoints let you inspect what's been connected.

- **GET Get Entities for Customer** — Lists all bank connections for a given customer (a customer can have many entities, one per bank).
- **GET Get Entity by ID** — Fetches a single entity, including its consent status and linked accounts.
- **GET Retrieve all entities for a given date range** — Bulk listing of entities created in a date window; handy for reconciliation and reporting.

### Data — Raw Data v2

Once an entity exists and consent is granted, these endpoints return the actual financial data. Always start with **Get Accounts** to discover the `account_id`s the user shared, then drill into per‑account endpoints.

**Accounts**

- **GET Get Accounts** — Returns only the accounts the customer chose to share. The `account_id`s here feed every endpoint below.

**Per‑account (`/data/v2/accounts/{account_id}/…`)**

- **GET Get Transactions** — Transaction history for an account.
- **GET Get Balances** — Current and available balances.
- **GET Get Beneficiaries** — Saved beneficiaries on the account.
- **GET Get Identities** — Identity details tied to the account holder.
- **GET Get Direct Debits** — Active direct debit mandates.
- **GET Get Scheduled Payments** — One‑off future‑dated payments.
- **GET Get Standing Orders** — Recurring outgoing payments.

**Identity (entity‑level)**

- **GET Get Identity** — Identity information for the entity as a whole (the person who granted consent), independent of any single account.

**Data Refreshes**

Lean caches the last data pulled for an entity. Refresh endpoints let you pull fresh data on demand, subject to the consent window.

- **POST Trigger a Manual Data Refresh** — Asks Lean to re‑fetch data from the bank for an entity. Returns a `refresh_id`.
- **GET Get Data Refreshes** — Lists previous refreshes for an entity.
- **GET Get Data Refresh Status** — Polls the status/result of a specific `refresh_id`.

**Results (legacy)**

- **GET Get Results** — Legacy endpoint for retrieving the outcome of a results‑based call. Kept for backwards compatibility; new integrations should use the refresh endpoints above.

### Banks

- **GET List Banks** — Returns banks supported in KSA, optionally filtered by `account_types` (e.g. `PERSONAL`, `BUSINESS`). Use this when building a custom bank picker instead of the LinkSDK default UI.

### Verifications

Verification APIs work alongside or independently of the data flow. Some require a connected entity, others just take details in the request body.

- **POST Verify Entity Ownership** — Confirms that a connected entity actually belongs to the person you expect. Used in KYC/onboarding.
- **POST Account Verification (per‑country)** — IBAN / account‑number verification across 25+ countries (SA, AE, GB, US, …). One request per country to match the local account format.
- **POST Get Beneficiary Name** — Returns the registered name behind a given IBAN, used to prevent misdirected payouts (confirmation‑of‑payee style check).
- **POST Business Verification** — Verifies a Saudi commercial registration / corporate entity.
- **POST Managers Verification** — Verifies the authorized managers/signatories of a corporate entity.
- **POST Verify Freelancer Certificate** — Validates a Saudi freelancer certificate (e.g. Monsha'at / freelance license).

### Payouts

Placeholder folder reserved for upcoming payout endpoints. No requests are shipped yet.

---

## Where to go next

- **LinkSDK demos** — see `KSA/LinkSDK/` for runnable HTML pages that drive the connection flow end‑to‑end.
- **Webhooks** — example payloads (`entity.created`, `bank.availability.updated`, `entity.data.refresh.updated`) live in `KSA/Webhooks/`. Wire these into your backend to react to connection and refresh events.
