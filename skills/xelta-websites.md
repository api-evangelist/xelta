---
version: 0.1.0
name: xelta-websites
description: |
  Generate and deploy AI websites via Xelta's Pomeli Website Builder. Use when:
  "build a website", "make a site", "generate a landing page", "create a
  portfolio site", "show my recent websites", "deploy my site". NOT for
  image/video/reel generation (use xelta-generate).
argument-hint: "[site description] [--industry <name>] [--pages <list>] [--deploy]"
allowed-tools: mcp__claude_ai_xelta__build_website, mcp__claude_ai_xelta__get_website_history
---

# Xelta Websites

Calls Xelta's `build_website` MCP tool, which generates a full website from
a brief in one call — there is no framework or code for the agent to write
or review here, just a good brief.

## build_website

Required: `prompt`, `industry`, `pages`.

1. **prompt** — a one- or two-sentence description of the site, e.g. "A
   modern portfolio for a UX designer" or "A landing page for a boutique
   coffee roastery". Ask for this if the user hasn't described what the
   site is for.
2. **industry** — a category such as technology, healthcare, fashion,
   education, finance, food & beverage, real estate, etc. Infer it from
   the prompt when obvious; ask only if genuinely ambiguous.
3. **pages** — comma-separated page names, e.g. `"Home, About, Services,
   Contact"`. If the user doesn't specify, propose a sensible default set
   for the type of site (a portfolio needs Home/Work/About/Contact; a
   product site needs Home/Features/Pricing/Contact) and confirm before
   generating.

Optional:
- **palette** — a color palette name or hex values, if the user has brand
  colors in mind.
- **logo_url** — a public URL to an existing logo image.
- **deploy** — `false` by default. Only set to `true` after the user
  explicitly confirms they want it published live to Vercel — this is a
  real, visible action, not a preview.

Either way the call returns a URL: with `deploy: false` it's a preview link
for reviewing the result, not a public site; with `deploy: true` it's the
live, published one. Tell the user explicitly which kind they're getting —
don't let them assume a preview link is already public, or that they need to
redeploy to see a live one they already have.

Call `build_website` once all three required fields are known. Print the
returned URL(s) clearly — don't bury them in extra commentary.

## get_website_history

Takes an optional `limit` (default 5). Use it for "show my recent sites" /
"what did I build earlier" requests, and to remind the user of a live URL
they've generated before instead of rebuilding from scratch.

Present each returned entry as a short line — whatever fields actually come
back (e.g. name/prompt summary, URL, deploy status) — rather than a raw
dump. Don't invent fields that aren't in the response; if it comes back
sparser than expected, show what's there cleanly instead of padding it with
guesses.

## UX rules

1. Ask for missing required fields one at a time rather than a long
   intake form, unless the user already volunteered several at once.
2. Don't set `deploy: true` speculatively — confirm first.
3. After generation, tell the user they can ask again with tweaks
   (different palette, added pages) rather than implying the result is
   final and unchangeable.
4. Detect the user's language from their message and reply in it; keep
   technical field names (`industry`, `pages`) as-is when confirming
   values back to the user.
