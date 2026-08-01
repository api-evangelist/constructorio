---
name: constructorio-search-configuration
description: Configure how a Constructor index behaves — facets, facet options, searchabilities, synonyms, sort options, redirect rules and collections — using the v2 configuration surface.
api: openapi/constructorio-configuration-openapi.yml
operations:
  - v2-facets-retrieve-facets
  - v2-facets-create-facet
  - v2-facets-update-facet
  - v2-facets-delete-facet
  - v1-facet-options-create-or-update-facet-options
  - v2-searchabilities-create-or-update-searchabilities
  - v2-one-way-synonyms-create-one-way-synonym
  - v1-synonyms-create-synonym
  - v1-sort-options-create-or-replace-sort-options
  - v1-redirects-create-redirect-rule
  - v1-collections-create-collection
  - v1-collections-create-or-ignore-collection-items
---

# Configure a Constructor index

Private surface on `https://ac.cnstrc.com`. Authenticate with Basic or Bearer. Each
resource has its own scope pair — `facets(r|w)`, `searchabilities(r|w)`, `synonyms(r|w)`,
`sort_options(r|w)`, `redirects(r|w)`, `collections(r|w)` — listed per operation in
`scopes/constructorio-scopes.yml`.

## Steps

1. **Declare searchability first.** `v2-searchabilities-create-or-update-searchabilities`
   controls which catalog fields are searchable, displayable and fuzzy-matched. Nothing
   else behaves correctly until this is right.

2. **Define facets, then their options.** Create the facet with
   `v2-facets-create-facet` (list with `v2-facets-retrieve-facets`, amend with
   `v2-facets-update-facet`). Curate individual values with
   `v1-facet-options-create-or-update-facet-options` — display names, ordering, hiding.

3. **Add vocabulary.** `v1-synonyms-create-synonym` creates a bidirectional synonym
   group; `v2-one-way-synonyms-create-one-way-synonym` creates a directional mapping
   (child phrases resolve to a parent phrase, but not the reverse).

4. **Add sort options.** `v1-sort-options-create-or-replace-sort-options` — a sort option
   is the composite of `sort_by` and `sort_order`.

5. **Add redirects.** `v1-redirects-create-redirect-rule` sends matching queries to a
   destination URL instead of a result set. Search responses can return a
   `RedirectResponse` in place of a `SearchResponse`, so client code must handle both
   branches of that `anyOf`.

6. **Curate collections.** `v1-collections-create-collection`, then
   `v1-collections-create-or-ignore-collection-items` to populate. Collections are
   addressable from browse (`v1-browse-get-browse-results-by-collection-id`) and from
   Offsite Discovery.

## Rules

- Use the **v2** facets and searchabilities operations. The `v1-facets-*` and
  `v1-searchabilities-*` equivalents are marked `deprecated: true`; Constructor publishes
  a migration guide at
  https://docs.constructor.com/reference/configuration-facets-and-searchabilities-v2-migration-guide
- Configuration changes affect live shopper traffic on the index named by `key`. Apply to
  a development or staging key first — accounts commonly hold one key per environment.
- `409 Conflict` on a create means the resource already exists; switch to the
  create-or-replace or update variant rather than retrying the create.
