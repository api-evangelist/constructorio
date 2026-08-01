---
name: constructorio-search-and-browse
description: Retrieve AI-optimized search, autocomplete and browse results from a Constructor index, with filtering, faceting, sorting and pagination, and attribute the result set for KPI measurement.
api: openapi/constructorio-search-openapi.yml
apis:
  - openapi/constructorio-search-openapi.yml
  - openapi/constructorio-autocomplete-openapi.yml
  - openapi/constructorio-browse-openapi.yml
operations:
  - v1-autocomplete-get-autocomplete-results
  - v1-search-get-search-results
  - v1-search-get-natural-language-search-results
  - v1-browse-get-browse-results
  - v1-browse-get-browse-groups
  - v1-browse-get-browse-facets
  - v1-browse-get-browse-facet-options
---

# Search and browse a Constructor index

Constructor's discovery endpoints are **public** — they take no API token. They take a
`key` query parameter naming the index. Do not send an API token to these operations;
the token is for catalog and configuration work only.

Host: `https://ac.cnstrc.com`

## Steps

1. **Autocomplete as the shopper types.** Call `v1-autocomplete-get-autocomplete-results`
   with the query prefix (max 200 characters) and `key`. The response may carry several
   sections — commonly `Products` and `Search Suggestions`. The `Products` section uses
   the same result shape as search.

2. **Run the search.** Call `v1-search-get-search-results` with the term in the path and
   `key` in the query. Add `page` and `num_results_per_page` for pagination, `filters` +
   `filter_match_types` for facet filtering, `sort_by` + `sort_order` for sorting, and
   `pre_filter_expression` for hard business rules (availability, region, entitlement)
   that must apply before ranking.

3. **Use natural-language search for spoken or conversational input.** Call
   `v1-search-get-natural-language-search-results` instead. Convert speech to text on the
   client first; the API de-naturalizes the query and extracts filters from it.

4. **Browse a category.** Call `v1-browse-get-browse-results` with the facet name and
   value in the path. For navigation chrome, call `v1-browse-get-browse-groups` for the
   group hierarchy and `v1-browse-get-browse-facets` /
   `v1-browse-get-browse-facet-options` for the facet rail.

5. **Read the envelope, not the echo.** Every response is
   `{ request, response, result_id }`. Render from `response.results`. The `request`
   object is documented as debugging-only and subject to change — never depend on it in
   production code.

6. **Keep `result_id` and send it back with behavior.** `result_id` is how Constructor
   attributes clicks, add-to-carts and purchases to the result set that produced them.
   Dropping it breaks the KPI optimization the ranking depends on. Offline events go to
   `v1-offline-behavioral-actions-create-actions`.

## Rules

- `key` is a public index identifier, not a credential. It is safe in browser code.
- Handle `400` (invalid parameters), `404` (unknown facet/collection/group) and `429`
  (rate limit breached) — see `errors/constructorio-problem-types.yml`.
- There are no `RateLimit-*` or `Retry-After` response headers. On `429`, back off
  exponentially; there is no in-band budget signal to read.
- **Do not load test.** Constructor prohibits it; violations can trigger throttling that
  degrades real shopper traffic, usage fines, or termination.
- Section defaults to `Products`. Pass `section` explicitly when querying another one.
- Conventions reference: `conventions/constructorio-conventions.yml`.
