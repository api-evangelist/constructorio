# Constructor.io

Constructor (Constructor.io) is an AI-powered ecommerce search and product discovery
platform for online retailers — autocomplete, keyword and natural-language search,
image search, browse, recommendations, quizzes, collections, offsite/email discovery,
retail media, an AI Shopping Agent and Product Insights Agent, plus a full
catalog-management, configuration and searchandising surface.

- Website — https://constructor.com/
- Documentation — https://docs.constructor.com/
- API Reference — https://docs.constructor.com/reference/main-readme
- Status — https://constructor.status.io/
- Releases — https://releases.constructor.io/
- GitHub — https://github.com/Constructor-io

## API surface

**17 OpenAPI 3.1.0 documents, 180 operations**, harvested verbatim from the specs
Constructor publishes through its ReadMe-hosted documentation (`/branches/1/apis/*.json`).
They are stored as YAML in `openapi/` with the untouched JSON in `openapi/_original/`.

| Spec | Ops | Host |
|---|---|---|
| search | 2 | ac.cnstrc.com |
| autocomplete | 1 | ac.cnstrc.com |
| browse | 9 | ac.cnstrc.com |
| recommendations | 1 | ac.cnstrc.com |
| image-search | 2 | image-search.cnstrc.com |
| ai-shopping-agent | 3 | agent.cnstrc.com |
| quizzes | 3 | quizzes.cnstrc.com |
| offsite-discovery-recommendations | 6 | offsite-discovery.cnstrc.com |
| product-details | 2 | product-details.cnstrc.com |
| catalog-management | 31 | ac.cnstrc.com |
| catalog-batching | 4 | batching.catalog.cnstrc.com |
| configuration | 77 | ac.cnstrc.com |
| searchandising | 33 | ac.cnstrc.com |
| retail-media | 2 | retail.media-cnstrc.com |
| retail-media-display-ads | 2 | display.media-cnstrc.com |
| behavioral-actions | 1 | ac.cnstrc.com |
| user-profile | 1 | user-profile.cnstrc.com |

Auth is two-track: shopper-facing discovery endpoints are public (addressed by a public
`key` index identifier), while catalog, configuration, searchandising, retail-media and
profile endpoints take an API token over HTTP Basic (token as username) or HTTP Bearer
with 37 named scopes.

## What is here

`openapi/` · `overlays/` · `authentication/` · `scopes/` · `conventions/` · `errors/` ·
`lifecycle/` · `changelog/` · `conformance/` · `security/` · `well-known/` · `packages/` ·
`cli/` · `components/` · `data-model/` · `mcp/` · `llms/` · `skills/`

## What is deliberately absent

- **No `a2a/`** — `/.well-known/agent-card.json` and `/.well-known/agent.json` return 404
  on every Constructor host. Agent cards are never authored on a provider's behalf.
- **No `asyncapi/` and no webhooks** — Constructor publishes no event, streaming or
  webhook surface. Behavioral data flows *into* Constructor via the beacon and the
  offline behavioral-actions endpoint; nothing flows out as events. Not applicable
  rather than missing.
- **No `sandbox/`** — Constructor documents no test/live key separation, magic test
  identifiers or sandbox environment. Environments are separated by provisioning
  additional API keys (dev / staging / prod), which is not a published test-data surface.
- **No `/.well-known/` documents at all** — 28 probes across four hosts, zero hits.
- **No vulnerability disclosure policy** — no security.txt, no responsible-disclosure
  page, no bug-bounty program. SOC 2 Type 2 and ISO 27001 are published at
  https://constructor.com/security-and-compliance (reports under NDA).
- **No idempotency contract** — no `Idempotency-Key` header appears anywhere in the 180
  operations. Retry safety is structural (PUT create-or-replace, PATCH merge, 409 on
  duplicate create).
- **No rate-limit headers** — 429 is declared on 146 of 180 operations, but no
  `RateLimit-*` or `Retry-After` response header is defined.

Constructor's published MCP server (`https://docs.constructor.com/mcp`, 6 tools,
anonymous) is a **documentation** server — it reads the specs and docs, it cannot call
the product API. See `mcp/constructorio-tool-crosswalk.yml`.
