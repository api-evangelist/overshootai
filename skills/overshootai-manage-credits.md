---
name: Check balance and buy Overshoot credits
description: List model pricing, check the prepaid balance, and create a checkout session to top up credits before running inference.
api: openapi/overshootai-openapi.yaml
operations: [listPricing, getPricing, getBalance, createCheckout, listModels]
---

# Check balance and buy credits

Overshoot bills on prepaid credits. Money fields are integer **microcents**
(`1 cent = 1,000,000 microcents`). Use this flow to keep enough balance so
inference does not fail with `402`.

## Steps
1. **Discover models** — `listModels` (`GET /models`, no auth). Use each `id` as the `model` field in inference.
2. **Check prices** — `listPricing` (`GET /billing/pricing`, no auth) for all public models, or `getPricing` (`GET /billing/pricing/{model}`) for one. Prices are per-token input/output/cached-input in microcents; the `{model}` path may contain slashes.
3. **Read balance** — `getBalance` (`GET /billing/accounts/me/balance`, auth required). `balance_cents` is for display; `balance_microcents` is the exact billing unit.
4. **Top up** — `createCheckout` (`POST /billing/checkout`) with `amount_cents` >= `100`; you receive a checkout session URL to complete payment.

## Notes
- Lower resolutions and fewer frames cost significantly less.
- Cached prompt prefixes (threads) are billed at the reduced `cached_input_microcents_per_token` rate — see `conventions/overshootai-conventions.yml`.
- All amounts are integers; never send fractional cents.
