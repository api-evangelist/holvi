---
name: Initiate a Holvi SEPA payment (PISP)
description: Initiate a SEPA / SEPA Instant / SWIFT payment from a Holvi customer account and poll for Strong Customer Authentication completion.
api: openapi/holvi-psd2-openapi.yml
operations: [exchangeConsentToken, listPaymentAccounts, initiatePayment, getPayment]
---

# Initiate a Holvi SEPA payment (PISP)

Initiate a payment from a Holvi customer's payment account as a licensed TPP.

## Prerequisites
- Approved Holvi TPP onboarding and a PSU consent JWT (see `holvi-read-account-information.md`).
- Write requests additionally require a `Digest` header (RFC 3230, `SHA-256=<hash>`) and the `Signature` must cover `(request-target) host date content-type digest`.

## Steps
1. `exchangeConsentToken` — obtain the PSU JWT (if not already held).
2. `listPaymentAccounts` — pick the source account `uuid` (and confirm its currency).
3. `initiatePayment` — `POST /api/v2/payment-initiation/` with `payment_account`, `amount` (decimal string, e.g. `"100.00"`), `counterparty` (`name`, `account_identifier`, `account_identifier_type` = `iban`|`national_account_number`, plus `bic`/`street`/`country` for `international`), and either `structured_reference` or `unstructured_reference`. Set `method` = `sepa` (default) or `international`; set `instant: true` for SEPA Instant. Response `201` returns the payment in state `unverified`.
4. The customer completes SCA on their phone.
5. `getPayment` — poll until `state` becomes `verified` (then `paid` once processed). Terminal negatives: `notenoughbalance`, `cancelled`.

## Errors
- `403` the user lacks payment-initiation permission or is not verified.
- `503` a downstream service is unavailable — retry.
- See `errors/holvi-problem-types.yml` and payment states in the API glossary.
