---
name: Verify payee then pay (VOP)
description: Initiate a Holvi payment with Verification of Payee, then confirm it when the payee name is a fuzzy or no match.
api: openapi/holvi-psd2-openapi.yml
operations: [initiatePayment, confirmVop, getPayment]
---

# Verify payee then pay (VOP)

Run Verification of Payee (VOP) so the payee name is checked against the account holder before the payment proceeds — reduces fraud and misdirected payments.

## Prerequisites
- Same headers as payment initiation (JWT, Signature, Digest). VOP requires both the payee `name` and IBAN.

## Steps
1. `initiatePayment` — `POST /api/v2/payment-initiation/` with `execute_verification_of_payee: true` plus the standard payment fields.
   - `201` = perfect match, payment created normally (proceed to step 3).
   - `202` = fuzzy/no match: the body has `payment_initialization_id` and `vop_result` (`match_result` = `match`|`close-match`|`no-match`, `match_name`, `confidence_score`, `requires_confirmation: true`). Show the expected vs. actual name to the user.
2. `confirmVop` — if the user accepts, `POST /api/v2/payment-initiation/{payment_initialization_id}/confirm-vop` to create the payment (`201`).
3. `getPayment` — poll until `state` = `verified`.

## Errors
- `400` `{"error":"VOP verification failed","details":"..."}` — VOP service unavailable or missing payee name/IBAN.
- `404` payment initialization not found or not in a confirmable state.
- `503` VOP service unavailable — retry. See `errors/holvi-problem-types.yml`.
