---
name: constructorio-merchandising-rules
description: Apply merchandising control in Constructor — refined queries, refined filters, refined collections, refined tags and campaigns — on top of the AI-ranked result set.
api: openapi/constructorio-searchandising-openapi.yml
operations:
  - v1-searchandising-retrieve-refined-queries
  - v1-searchandising-create-refined-query
  - v1-searchandising-update-refined-query
  - v1-searchandising-create-or-replace-refined-filter
  - v1-searchandising-create-or-replace-refined-collection
  - v1-searchandising-create-or-replace-refined-tag
  - v1-searchandising-retrieve-campaigns
  - v1-searchandising-create-campaign
  - v1-searchandising-create-facet-rule-campaign
---

# Merchandise a Constructor result set

Private surface on `https://ac.cnstrc.com`. Bearer tokens need the matching
`searchandising.*` scopes — `searchandising.refined_queries(r|w)`,
`searchandising.refined_filters(r|w)`, `searchandising.refined_collections(r|w)`,
`searchandising.refined_tags(r|w)`, `searchandising.campaigns(r|w)` — plus
`facets.refined_queries` / `facets.refined_filters` where the rule attaches to a facet.

## Steps

1. **Pick the surface the rule attaches to.** Constructor models merchandising per
   surface, not globally:
   - a specific **search term** → refined query (`v1-searchandising-create-refined-query`)
   - a **browse facet value** → refined filter (`v1-searchandising-create-or-replace-refined-filter`)
   - a **collection** → refined collection (`v1-searchandising-create-or-replace-refined-collection`)
   - **user request data** (segment, tag) → refined tag (`v1-searchandising-create-or-replace-refined-tag`)

2. **List before you write.** `v1-searchandising-retrieve-refined-queries` (and the
   `retrieve-refined-filters` / `retrieve-refined-collections` /
   `v1-searchandising-retreive-refined-tags` siblings) tell you what already applies to
   that surface. Overlapping rules compound.

3. **Group rules into a campaign.** `v1-searchandising-create-campaign` for a scheduled
   or themed set; `v1-searchandising-create-facet-rule-campaign` for facet-rule
   campaigns. Retrieve with `v1-searchandising-retrieve-campaigns`.

4. **Verify against live results.** Re-run `v1-search-get-search-results` or
   `v1-browse-get-browse-results` for the affected term/facet and confirm the boost,
   bury, pin or slot landed. Rule performance metrics are surfaced in the Searchandizing
   dashboard.

## Rules

- Every rule overrides the KPI-optimized ranking Constructor computed. Prefer the
  narrowest surface that solves the problem; a refined tag applied broadly is the easiest
  way to lose conversion.
- `v1-searchandising-retreive-refined-tags` is spelled with that typo in the contract.
  Use it verbatim — it is the real `operationId`.
- Use create-or-replace variants for anything retried; plain creates return `409 Conflict`
  on a duplicate. There is no idempotency key in this API.
- Filter expression syntax:
  https://docs.constructor.com/reference/constructor-concepts-filter-expressions
