---
name: efs-integrations
description: Implement J.P. Morgan Embedded Finance & Solutions (EF&S / Embedded Payments) API integrations in a platform project. Use when the user wants to call Embedded Payments REST APIs — onboard a client, accounts, recipients / linked accounts, transactions / payouts, or notification subscriptions (webhooks). Assumes API authentication is already wired in the project. Recommends implementing one API capability at a time.
metadata:
  version: 1.0.0
---

# EF&S API Integrations

Guide a platform through integrating **one** Embedded Finance & Solutions (Embedded Payments) API capability at a time.

> **Product names:** The Payments Developer Portal says **Embedded Payments** / **Embedded Finance Solutions**. This skill uses **EF&S** as shorthand for the same API domain.

## Disclaimer (show once per session)

> **Disclaimer:** We do not assume any responsibility for any issues, damages, or losses that may arise from using the code generated or modified by this skill. These skills rely on AI agents to produce code, and all generated code should be carefully reviewed and verified by a qualified engineer before being deployed to a production environment.

## Step 1 — Confirm API access is ready

The platform should already be able to call EF&S sandbox/production hosts with whatever auth their project uses (configured outside this skill).

Ask:
- Question: "Can this project already call Embedded Payments APIs (sandbox or otherwise)?"
- Header: "API access?"
- Options:
  - "Yes — auth and base URL are wired"
  - "No — not yet"

If **No**: point them at the Payments Developer Portal auth docs ([API Authentication](https://developer.payments.jpmorgan.com/api/authentication)) and the [Embedded Payments](https://developer.payments.jpmorgan.com/docs/embedded-finance-solutions/embedded-payments) overview, then exit. Do **not** generate auth/mTLS modules in this skill.

If **Yes**, ask where their existing HTTP/API client lives (path) so new code can import it.

## Step 2 — Pick one API capability

Implement **one capability per change set**. Bundling onboarding + payouts + webhooks in one PR makes failures harder to isolate.

Ask:
- Question: "Which Embedded Payments API capability do you want to integrate first?"
- Header: "API"
- Options:
  - "Onboard a client" — create/update clients & parties, due diligence, documents, submit for review
  - "Accounts" — create and manage limited / transaction accounts
  - "Recipients / linked accounts" — external bank accounts, microdeposit verification
  - "Transactions / payouts" — list, get, and initiate money movement
  - "Notifications (webhooks)" — subscribe, verify, and handle event payloads

Then ask free text:

> "Briefly describe what you're building — e.g. 'marketplace seller KYC', 'payout dashboard with ACH'. A sentence or two is enough."

Read the matching reference:

| Choice | Reference |
| --- | --- |
| Onboard a client | `references/onboard-a-client.md` |
| Accounts | `references/accounts.md` |
| Recipients / linked accounts | `references/recipients.md` |
| Transactions / payouts | `references/transactions.md` |
| Notifications (webhooks) | `references/notifications.md` |

For a fuller sample of recipient linking, also see `examples/linked-account-flow.md`.

If the user asks for **merchant Checkout / Online Payments** (card drop-in, IDAnywhere OAuth merchant APIs), hand them to `jpm-merchant-integrations` — that is a different PDP product family.

If they ask for hosted iframe UI or the React component library, tell them this skill covers the **REST API domain only**; point at the OSS guides under https://github.com/jpmorgan-payments/embedded-finance without expanding into UI work here.

## Step 3 — Implement the chosen API

Follow the chosen reference. Cross-cutting principles:

- **Reuse the project's existing API client.** Do not re-implement authentication inside feature modules.
- **Environment-aware base URLs.** Read hosts/paths from env (e.g. sandbox `https://api-sandbox.payments.jpmorgan.com`). Never hardcode production hosts in source.
- **Confirm output paths before writing.** Default to something like `src/efs/onboarding/`, `src/efs/transactions/`, etc.
- **Honor current PDP OpenAPI / docs** for field names — do not invent payloads from memory.
- **Smoke test in sandbox** using the [Testing Catalog](https://github.com/jpmorgan-payments/embedded-finance/blob/main/embedded-components/docs/TESTING_CATALOG.md) where magic values apply.
- Prefer official docs:
  - https://developer.payments.jpmorgan.com/docs/embedded-finance-solutions/embedded-payments
  - https://developer.payments.jpmorgan.com/api/embedded-finance-solutions/embedded-payments/overview

## Step 4 — Offer to add another

Once the capability works in sandbox:

Ask:
- Question: "That API integration looks good. Want to add another Embedded Payments capability now?"
- Header: "More?"
- Options:
  - "Yes — pick another"
  - "No — I'm done for now"

If **Yes**, return to Step 2. If **No**, exit with a one-line summary of which APIs were integrated.

## Rules

- Scope is **EF&S / Embedded Payments REST APIs only** — not merchant Checkout, not auth module generation, not React component packaging.
- Confirm every output file path with the user before writing.
- Do not invent request/response fields; follow PDP docs and OAS.
- Keep secrets and cert material out of feature code and out of the browser.
