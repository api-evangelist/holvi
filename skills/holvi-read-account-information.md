---
name: Read Holvi account information (AISP)
description: Obtain a PSU consent token and read a Holvi customer's payment accounts and recent payments over the PSD2 API.
api: openapi/holvi-psd2-openapi.yml
operations: [exchangeConsentToken, listPaymentAccounts, getPaymentAccount, listPayments, getPayment]
---

# Read Holvi account information (AISP)

Use this to retrieve a Holvi customer's (PSU's) payment accounts and payment history as a licensed Third Party Provider.

## Prerequisites
- An approved Holvi TPP onboarding: eIDAS QSEAL client certificate, `X-Holvi-Client-Id`, `X-Holvi-Client-Secret`.
- Every request (except onboarding) must carry `Host`, `Date` (RFC 7231), and a `Signature` header (Draft Cavage HTTP Signatures v10, RSA-SHA256, key >= 2048-bit) over `(request-target) host date` for GET. See `conventions/holvi-conventions.yml`.
- Base URL: `https://api.psd2.holvi.com`.

## Steps
1. Send the PSU to `https://psd2.holvi.com/login/?client_id=...&redirect_uri=...&state=...`; on return you receive a `code`.
2. `exchangeConsentToken` — `GET /api/v2/consent/token/{code}/exchange/` to get the JWT (`id_token`). Put it in `Authorization: Bearer <id_token>` on all following calls.
3. `listPaymentAccounts` — `GET /api/v2/payment-accounts/` to list the PSU's accounts (each has `uuid`, `iban`, `currency`, `balance`).
4. `getPaymentAccount` — `GET /api/v2/payment-accounts/{uuid}/` for one account's detail.
5. `listPayments` — `GET /api/v2/payment-accounts/{uuid}/payments/` for its payments. Filter with `state`, `direction`, `from_date`, `to_date`. Paginate with `page` (response has `count`/`next`/`previous`/`results`). Only the last 365 days are returned.
6. `getPayment` — `GET /api/v2/payment-accounts/{payment_account_uuid}/payments/{payment_uuid}/` for one payment.

## Errors
- `401` invalid signature/certificate/token. Re-check the signature headers and token freshness. See `errors/holvi-problem-types.yml`.
