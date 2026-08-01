---
name: constructorio-ai-shopping-agent
description: Drive Constructor's AI Shopping Agent — conversational intent-based product discovery over Server-Sent Events — and the Product Insights Agent question/answer surface.
api: openapi/constructorio-ai-shopping-agent-openapi.yml
operations:
  - v1-asa-retrieve-intent
  - v1-asa-retrieve-item-questions
  - v1-asa-retrieve-item-questions-answer
---

# Use the Constructor AI Shopping Agent

Host: `https://agent.cnstrc.com`. This spec declares no security schemes — the agent
endpoints are part of the public shopper-facing surface, addressed by `key` like the
other discovery endpoints.

## Steps

1. **Send free-form shopper intent.** Call `v1-asa-retrieve-intent` with the
   conversational text in the path and `key` in the query. The response is a
   `text/event-stream`: results arrive in small batches so the UI can render
   progressively. Consume it as a stream — do not wait for a complete body.

2. **Show questions on a product detail page.** Call `v1-asa-retrieve-item-questions` for
   an item to get the AI-generated list of frequently asked questions.

3. **Answer one.** Call `v1-asa-retrieve-item-questions-answer` with the question. If the
   client sends `Accept: text/event-stream` the answer streams; otherwise the complete
   JSON is returned in one response. Choose deliberately — streaming is the better
   shopper experience, buffered is simpler for a server-side agent.

## Rules

- Handle both response modes on the answer operation. The content negotiation is real and
  the two shapes differ.
- Streamed responses carry correlation identifiers (`thread_id`, `intent_result_id`,
  `qna_result_id`) — retain them to tie follow-up turns and behavioral events to the
  conversation.
- These are beta-adjacent, fast-moving surfaces. Check
  `changelog/constructorio-changelog.yml` and the release notes at
  https://releases.constructor.io/ before depending on a field.
- The prebuilt UI for this flow is `@constructor-io/constructorio-ui-pia` and
  `@constructor-io/constructorio-ui-agent-overview` — see
  `components/constructorio-components.yml`.
- Constructor's own MCP server (`https://docs.constructor.com/mcp`) is a **documentation**
  server. It cannot call these operations. An agent that needs to shop must call the REST
  API directly — see `mcp/constructorio-tool-crosswalk.yml`.
