---
name: constructorio-catalog-ingestion
description: Load and maintain a product catalog in Constructor — items, variations and item groups — using authenticated create-or-replace and merge-update operations, and track the asynchronous tasks they return.
api: openapi/constructorio-catalog-management-openapi.yml
apis:
  - openapi/constructorio-catalog-management-openapi.yml
  - openapi/constructorio-catalog-batching-openapi.yml
operations:
  - v1-catalog-create-or-replace-catalog
  - v1-catalog-update-catalog
  - v2-items-create-or-replace-items
  - v2-items-update-items
  - v2-items-delete-items
  - v2-items-retrieve-items
  - v2-variations-create-or-replace-variations
  - v2-variations-update-variations
  - v2-item-groups-create-or-replace-item-groups
  - v2-item-groups-update-item-groups
  - v1-tasks-retrieve-task
  - v2-batching-items-update-items
---

# Ingest and maintain a Constructor catalog

These are **private** operations. Authenticate with either HTTP Basic (API token as the
username, empty password) or `Authorization: Bearer <token>`. A Bearer token must carry
`catalog(w)` — most write operations also require `search_suggestions(w)`; reads require
`catalog(r)`. A token that authenticates but lacks the scope is rejected with `403`.

Host: `https://ac.cnstrc.com` (batched operations: `https://batching.catalog.cnstrc.com`)

## Steps

1. **Bulk load.** For a full catalog, call `v1-catalog-create-or-replace-catalog` with
   `mode=full` and up to three multipart files keyed `items`, `variations`,
   `item_groups`. Omit a key to leave that resource type untouched. With `mode=ids`,
   upload files containing only an `id` column — every record whose ID is absent is
   deleted. For deltas, call `v1-catalog-update-catalog`.

2. **Record-level writes.** Use `v2-items-create-or-replace-items` (PUT — replaces the
   record) or `v2-items-update-items` (PATCH — merges into the existing record). Same
   pair exists for variations (`v2-variations-*`) and item groups (`v2-item-groups-*`).
   `v2-items-delete-items` removes records.

3. **High-volume batches.** For large update or delete waves use the batching service:
   `v2-batching-items-update-items`, `v2-batching-items-delete-items`,
   `v2-batching-variations-update-variations` and
   `v2-batching-variations-delete-variations` on `batching.catalog.cnstrc.com`.

4. **Follow the task.** Every catalog write returns `202 Accepted` with a task
   identifier — the write has been queued, not applied. Poll `v1-tasks-retrieve-task`
   until it completes. Do not treat the 202 as success.

5. **Verify.** Read back with `v2-items-retrieve-items` (optionally filtered by id) and
   check field coverage with `v1-items-fields-stats-retrieve-items-fields-stats`.

## Rules

- **There is no idempotency key.** Constructor defines no `Idempotency-Key` header
  anywhere in its contract. Retry-safety comes from shape: PUT create-or-replace and
  PATCH merge converge on replay; POST list-creates return `409 Conflict` on a repeat.
  Prefer the PUT/PATCH variants for anything an agent might retry.
- Model the hierarchy through `ItemGroup.parent_ids` (self-referential) and attach items
  via `group_ids`. Variations point at their parent through `item_id`.
- Identifiers are **caller-supplied natural keys** (your SKU / product ID), not
  server-issued opaque IDs. Keep them stable — they are the join key for everything.
- The `v1-item-groups-*` operations are deprecated in the spec; use the `v2-item-groups-*`
  equivalents. See `lifecycle/constructorio-lifecycle.yml`.
- Errors: `errors/constructorio-problem-types.yml`. Scopes: `scopes/constructorio-scopes.yml`.
