# xelta-create-images-videos-amp-more-with-generative-ai

Xelta AI Studio is a credit-based generative-AI content platform from Xelta Pvt Ltd for creating
images, videos, audio, micro-dramas, ads, websites and social content across 90+ third-party
models (FAL, BytePlus, OpenAI, Google, HuggingFace, Replicate).

Xelta ships **three developer surfaces that do not reference one another**:

  with a Swagger UI at `/api-docs/`. Public, parseable, and linked from no Xelta page.
- **A hosted MCP server** — `https://mcp.xelta.ai/mcp`, OAuth 2.1 with PKCE and a device-code
  fallback, eight tools, plus five Agent Skills published to npm as `@xelta999/skills-cli`.
- **An llms.txt** — `https://xelta.ai/llms.txt`, a site guide that names neither of the above.

Not one MCP tool maps to a REST operation. See `mcp/xelta-tool-crosswalk.yml`.

> **Correction, 2026-09-01.** The first profile of this company recorded "does not publish a
> conventional public REST/GraphQL API" and set `has_public_api: false`. That was wrong. The
> OpenAPI was live the whole time at the API host root, not the docs host — it was simply never
> linked. Round-2 contract discovery found it, ownership was confirmed from the document's own
> `info.title`, `contact` and `servers[]`, and it is saved verbatim under `openapi/_original/`.
