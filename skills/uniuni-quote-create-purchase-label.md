---
name: Quote, create, purchase, and label a UniUni shipment
description: End-to-end happy path for shipping a parcel with the UniUni Platform Client API — price it, create the DRAFT shipment, purchase it from the wallet, and download the label PDF.
api: openapi/uniuni-platform-client-api-openapi.yml
operations: [getPricingQuote, createShipment, purchaseShipment, retrieveShipment, getLabel]
generated: '2026-07-21'
method: generated
---

# Quote, create, purchase, and label a shipment

Base URLs: sandbox `https://api-sandbox.ship.uniuni.com/client/`, production
`https://api.ship.uniuni.com/prod/client/`. Every request sends
`Authorization: Bearer <ACCESS_TOKEN>` (dashboard-generated token) and
`Content-Type: application/json; charset=utf-8`. All keys are camelCase.

1. **Price the parcel** — `getPricingQuote` (`POST /client/shipments/quote`) with destination
   address, dimensions, weight, and postage type. This does not create or reserve anything.
2. **Create the shipment** — `createShipment` (`POST /client/shipments/create`). The shipment
   lands in `DRAFT` (statusCode 180); no label exists yet.
3. **Purchase it** — `purchaseShipment` (`POST /client/shipments/{orderNumber}/purchase`).
   Cost is deducted from the wallet balance; status moves `DRAFT` → `PENDING` (181). Ensure the
   wallet is funded first (sandbox: top up with the documented test card `4242 4242 4242 4242`).
4. **Confirm** — `retrieveShipment` (`GET /client/shipments/{orderNumber}`) and check
   `data.status`.
5. **Get the label** — `getLabel` (`GET /client/label/{id}`, `labelType: shipping`). The PDF is
   Base64-encoded in `data.body`; decode and save it.

## Rules

- The envelope is `{ message, code, data }`; HTTP is 200 even on business errors — **check
  `code === 0`**, not the HTTP status. Malformed payloads return HTTP 422 with Zod issues.
- There is **no idempotency key** — do not blindly retry `purchaseShipment` on timeout; first
  `retrieveShipment` to see whether the purchase landed.
- Only `DRAFT` shipments can be deleted (`deleteShipment`); purchased shipments are refunded
  (`refundShipment`) instead.
- Error codes: 1002 invalid request, 1006 database error, 1009 general/auth error, 1014 not
  found (labels/batches), 1031 tracking lookup failed.
