# Xelta - Create Images, Videos & More with Generative AI

Xelta AI Studio is a credit-based generative-AI creative platform from Xelta Pvt Ltd, unifying
90+ AI models for image generation, video and micro-drama production, voice synthesis and dubbing,
website building, and marketing/ad automation.

- **Provider:** https://xelta.ai
- **MCP endpoint:** https://mcp.xelta.ai/mcp
- **Tags:** generative-ai, ai-image, ai-video, ai-audio, text-to-video, creative-tools,
  marketing-automation, social-media-content, video-editing, website-builder, mcp, agent-native
- **Public API:** yes — OpenAPI 3.0.0, 74 operations
- **Agent-native (MCP / llms.txt / skills):** yes — all three

## APIs

  Site Scan / Brand DNA, Community, Upload, Asset History and Contact. Bearer JWT, page/limit
  pagination, credit-metered generation with HTTP 402 on exhaustion, submit-then-poll async.
- **Xelta MCP Server** (`https://mcp.xelta.ai/mcp`) — hosted remote MCP over streamable HTTP.
  OAuth 2.1 authorization code + PKCE S256, RFC 8628 device code, RFC 7591 dynamic client
  registration, RFC 7009 revocation, published through a complete RFC 8414 + RFC 9728 discovery
  pair. Eight tools: `generate_image`, `create_mixboard`, `create_reel`, `generate_street_ad`,
  `build_website`, `get_website_history`, `check_balance`, `xelta_connection_status`.

## What round 2 found (2026-09-01)

The first profile concluded Xelta had no public API. STEP 0b contract discovery probed the API
**host root** rather than the docs host and found a real, first-party OpenAPI 3.0.0 embedded in a
was confirmed from the document itself — `info.title` "Xelta AI Platform API", `contact.url`

Also captured this round:

- Five **provider-authored Agent Skills**, saved verbatim from the npm package
  `@xelta999/skills-cli` v0.1.6 (2026-07-24). These are how the MCP tool inventory was
  established without an authenticated `tools/list`.
- A **machine-readable plan catalogue** at `GET /api/subscription/plans` (HTTP 200, anonymous) —
  three active tiers priced monthly in INR, and two internal test fixtures left public.
- The **MCP OAuth discovery pair**, saved verbatim under `well-known/`.

## Notable gaps

- **No operationIds** on any of the 74 operations. `overlays/` assigns deterministic ones.
- **No idempotency** anywhere. Every credit-spending call is unsafe to retry.
- **No rate-limit headers** and no documented 429, so an agent cannot back off intelligently.
- **No versioning** — one unversioned 1.0.0 surface, no deprecation policy, no status page.
- **No reversal** for 9 of 11 write surfaces, including a cascading Brand DNA delete and a live
  website publish. None carries a stated window.
- **`https://docs.xelta.ai`**, the documentation host named inside Xelta's own OpenAPI, does not
  resolve (NXDOMAIN).
- **No security.txt** and no vulnerability-disclosure channel on any host.
- **The declared `CommunityPost` schema does not match the live payload** — a client generated
  from the spec will not read the community feed correctly.
- Xelta's `robots.txt` disallows ClaudeBot, GPTBot, Google-Extended, CCBot and others while the
  company ships an MCP server, an llms.txt and five Agent Skills.

_Profiled by API Evangelist. All findings probed live 2026-09-01; every HTTP status is recorded in
the artifact it supports._
