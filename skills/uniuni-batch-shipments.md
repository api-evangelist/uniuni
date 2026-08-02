---
name: Batch UniUni shipments for drop-off or pickup
description: Group purchased shipments into a batch, manage its contents, and print the batch label with the UniUni Platform Client API.
api: openapi/uniuni-platform-client-api-openapi.yml
operations: [createBatch, addShipmentsToBatch, removeShipmentsFromBatch, retrieveBatch, listBatches, getLabel, deleteBatch]
generated: '2026-07-21'
method: generated
---

# Batch shipments for drop-off or pickup

Batching groups purchased shipments for bulk label printing and organized dispatch. Batch
tracking IDs are `URB` + 16 digits.

1. **Create a batch** — `createBatch` (`POST /client/batch/create`), optionally seeding it with
   purchased shipment order numbers, or create it empty.
2. **Add shipments** — `addShipmentsToBatch` (`POST /client/batch/{batchNumber}/addShipment`).
   Only **purchased** shipments can be batched; a shipment already in another batch is *moved*
   into this one.
3. **Adjust** — `removeShipmentsFromBatch` (`POST /client/batch/{batchNumber}/removeShipment`)
   to pull shipments back out.
4. **Inspect** — `retrieveBatch` (`GET /client/batch/{batchNumber}`) for the shipment order
   numbers; `listBatches` (`GET /client/batch`) paginates with `page`/`pageSize` (max 500).
5. **Print the batch label** — `getLabel` (`GET /client/label/{id}`, `labelType: batch`);
   decode the Base64 `data.body` PDF.
6. **Cleanup** — `deleteBatch` (`POST /client/batch/{batchNumber}/delete`) works only for
   batches that have been received.

## Rules

- Check `code === 0` in the `{ message, code, data }` envelope; HTTP status stays 200 on
  business errors. Code 1014 means the batch/label was not found.
- One shipment belongs to at most one batch at a time.
