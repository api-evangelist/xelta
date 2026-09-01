---
version: 0.1.0
name: xelta-generate
description: |
  Generate images, mixboards, video reels, and street-art brand ads via
  Xelta AI. Use when: "generate an image", "make a video", "create a reel",
  "make a mixboard", "grid of images", "street ad", "graffiti ad for my
  brand", "check my credit balance", or "is Xelta connected". NOT for
  building/deploying full websites (use xelta-websites).
argument-hint: "[prompt] [--mixboard|--reel|--street-ad] [--images N]"
allowed-tools: mcp__claude_ai_xelta__generate_image, mcp__claude_ai_xelta__create_mixboard, mcp__claude_ai_xelta__create_reel, mcp__claude_ai_xelta__generate_street_ad, mcp__claude_ai_xelta__check_balance, mcp__claude_ai_xelta__xelta_connection_status
---

# Xelta Generate

Calls Xelta's built-in MCP tools directly. There is no CLI to install and no
separate login step — the connection is a pre-authenticated MCP connector.
If a call fails with an auth-looking error, call `xelta_connection_status`
and tell the user to check their Xelta connector configuration if it
reports disconnected.

## Choosing a tool

- **A single image** → `generate_image`.
- **A grid/board of many images from one idea** → `create_mixboard`.
- **A short video** → `create_reel`.
- **A street-art/graffiti-style ad for a brand** → `generate_street_ad`.
- **"How many credits do I have"** → `check_balance`.

## generate_image — 3-step wizard

`generate_image` must be called up to 3 times per generation; do not try to
guess `model_id` or `size` yourself.

1. Call with only `prompt` (leave `model_id` and `size` empty). The tool
   returns a numbered list of available models — present it and ask the
   user to pick one.
2. Call again with `prompt` + `model_id` (leave `size` empty). The tool
   returns size options (`1080p` / `2K` / `4K`) — present them and ask.
3. Call a third time with `prompt` + `model_id` + `size` to actually
   generate. The tool returns the image URL.

Skip step 1 and/or 2 only if the user already explicitly named a model or
size in their request. `optimize_prompt` defaults to `true`; leave it
unless the user asks for their exact wording untouched.

## create_mixboard

One call: `command` (the creative prompt) and optional `num_images`
(default 8). Good for "give me a grid/board of options" requests.

A mixboard is a *set* of `num_images` separate generated images, not one
composited collage image — the tool returns that many individual URLs.
Present them as a numbered list captioned with the count (e.g. "Here are
your 8 mixboard options:"), not a bare list of links with no framing.

## create_reel

One call: `prompt`, and optional `image_url` if the user has a seed image
to animate/feature. There is no separate model/size wizard for reels.

## generate_street_ad

One call: `brand` (required), plus optional `style` (default `auto`,
e.g. `graffiti`, `stencil`, `wildstyle`), `surface` (default `brick`, e.g.
`wall`, `concrete`), and `logo_url` if the user has a logo to incorporate.

## UX rules

1. Be concise. No raw IDs or JSON dumps in chat — print the resulting
   media URL, or the model/size list when the wizard needs a choice. For
   results with more than one URL (mixboards), caption the count and
   number each one rather than pasting links with no framing.
2. No internal jargon. Don't narrate "calling generate_image" or "polling".
3. Detect the user's language from their message and reply in it.
4. Ask one thing at a time — don't batch multiple questions (model choice,
   size choice, style choice) into a single message.
5. Prefer a sane default and only ask when a required field is genuinely
   missing (e.g. `brand` for a street ad, `prompt` for everything else).

## Writing a good prompt

- **Subject + setting + style** in one sentence beats a long list of
  disconnected adjectives: "a red fox curled in a snowy pine forest,
  golden hour, cinematic".
- Name camera/lighting/medium when it matters: lens angle, rim light,
  neon glow, oil painting vs. photograph vs. 3D render.
- For a reel, describe motion, not just the static scene: "the camera
  dollies in as the dancer spins" rather than only describing what's in
  frame.
- Phrase exclusions positively — there's no negative-prompt field:
  "tack sharp" instead of "no blur", "uninhabited landscape" instead of
  "no people".
- Keep prompts focused (well under ~200 words); very long prompts tend to
  produce muddier results than a tight, concrete one.
- Avoid real public figures, sexual content, and trademarked
  characters/logos — generations of those are likely to be rejected.

## Troubleshooting

- Generation fails or times out → retry once with a slightly reworded
  prompt before escalating; server-side hiccups and content-policy
  rejections both often present as a generic failure.
- Result looks nothing like the request → check whether the prompt
  redescribed a reference image instead of describing the *change*
  wanted (`generate_street_ad`/`create_reel` with `image_url`/seed input
  behave like image-to-image: describe the transformation, not the input).
- `check_balance` returns 0 or an error → tell the user their Xelta
  account may be out of credits or the connector needs re-authenticating;
  don't retry generation calls in a loop when this happens.
