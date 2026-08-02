---
name: Track UniUni shipments and receive webhook updates
description: Poll tracking by tracking ID and consume HMAC-signed webhook events for shipment and insurance status changes on the UniUni Platform.
api: openapi/uniuni-platform-client-api-openapi.yml
operations: [trackShipment, testWebhook]
generated: '2026-07-21'
method: generated
---

# Track shipments and receive webhook updates

1. **Poll on demand** — `trackShipment` (`GET /client/tracking?trackingId=...`). Tracking IDs
   match `^(UR\d{17}|URB\d{16})$`. The response carries status, recipient, destination,
   weight/dimensions, and a scan `events[]` array (Unix-ms timestamps, lat/lng locations).
2. **Prefer push** — configure a webhook in the portal (Settings → Integrations → REST API →
   Webhook Configuration): HTTPS URL, auth mode `HMAC` (recommended), header name, and the
   portal-generated secret (shown once).
3. **Verify signatures** — recompute HMAC-SHA256 over the **raw JSON body** with the secret and
   compare timing-safe against the configured header (e.g. `X-Webhook-Signature`).
4. **Test the wiring** — `testWebhook` (`POST /webhook/test`) fires a real signed POST at the
   configured endpoint; return a 2xx.
5. **Handle events** — `shipment.status_updated` (status + statusCode per the shipment status
   reference; `proofOfDelivery` appears only on `DELIVERED`, statusCode 203),
   `shipment.custom_event`, and `insurance.status_updated` (PURCHASED, CLAIM_* states).

## Rules

- Status values are uppercase API constants; map them with the numeric codes (180 DRAFT … 203
  DELIVERED, 209 SHIPMENT_EXCEPTION, 230 RETURNED, 231-233 failed-delivery attempts).
- Tracking lookup failures surface as `code: 1031` inside an HTTP 200 envelope.
- Webhook payloads are versioned (`version: "1"`); tolerate unknown fields.
