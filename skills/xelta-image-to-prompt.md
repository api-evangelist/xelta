---
version: 0.1.0
name: xelta-image-to-prompt
description: |
  Analyze an uploaded property photo directly (Claude's own vision — no
  separate Xelta tool call) and produce a prompt accurate enough to
  regenerate or animate it: scene breakdown, lighting & mood, style/
  aesthetic, and material/texture detail, shown as labeled fields plus
  three variants (literal / enhanced / stylized). Supports batch upload
  (multiple photos → one prompt set), "match this style" (apply a
  reference image's look to a different property photo), confidence flags
  for ambiguous details, exclusion suggestions (clutter, watermarks,
  people), and an optional prompt-to-video upgrade with suggested camera
  movement. Use when: "turn this photo into a prompt", "describe this
  property photo", "reverse prompt this image", "match this photo's style
  to that one", "what's the lighting/style in this photo". NOT for
  generating new images from a prompt you already have (use
  xelta-generate) or building real-estate shots/reels from scratch with no
  source photo (use xelta-real-estate).
argument-hint: "[image] [--match-style reference-image] [--video]"
allowed-tools: mcp__claude_ai_xelta__generate_image, mcp__claude_ai_xelta__create_mixboard, mcp__claude_ai_xelta__create_reel, mcp__claude_ai_xelta__check_balance, mcp__claude_ai_xelta__xelta_connection_status
---

# Xelta Image to Prompt

Reverse-prompting for property photos. The analysis step is Claude's own
vision reading the image directly — there's no Xelta MCP tool for this, so
nothing gets called until a prompt is produced and the user is ready to
generate. Once a variant is picked, hand off to `xelta-generate`'s tools the
same way `xelta-prompt-enhance` does.

## Workflow

1. **Confirm there's an image to analyze.** If none was provided or the
   link/file is broken, ask for one — don't fabricate an analysis.
2. **Sanity-check the subject.** If the image clearly isn't a property/
   interior/exterior shot, say so and ask the user to confirm they still
   want a breakdown, rather than forcing real-estate framing onto an
   unrelated photo.
3. **Break the image down into four analysis layers**, shown as labeled
   fields so the user can tweak just one without rewriting everything:
   - **Scene** — subject, setting, composition (wide/close/aerial), camera
     angle, framing.
   - **Lighting & mood** — time of day, light source/direction, warm vs.
     cool tone, contrast level. This is usually the hardest thing for a
     user to describe themselves — spend real effort here.
   - **Style/aesthetic** — architectural style, interior design style
     (modern, Scandinavian, industrial, traditional, etc.), color palette.
   - **Material & texture** — flooring, surfaces, finishes. Matters more
     for real estate than most subjects, since buyers/agents care about
     these specifics.
4. **Flag anything ambiguous** instead of guessing silently — e.g. "can't
   tell if this is dusk or dawn from the light alone; treating it as dusk,
   say if that's wrong." Attach flags inline next to the relevant field.
5. **Suggest exclusions as positive phrasing**, not a separate negative-
   prompt field — Xelta's generation tools don't have one (see
   `xelta-generate`'s prompt guidance). If the photo has clutter, a
   watermark, or people in frame that shouldn't carry into the generated
   version, fold that into the prompt as a positive instruction ("clean,
   staged, uninhabited interior") rather than listing what to avoid.
6. **Produce three variants** from the same analysis:
   - **Literal/accurate** — describes exactly what's in the photo.
   - **Enhanced/idealized** — same scene, elevated staging/lighting/finish
     quality.
   - **Stylized** — same scene reinterpreted in a clear alternate style or
     mood.
   Present numbered 1/2/3. Each variant's prompt text should be editable —
   show it as plain text the user can directly propose edits to, not a
   black box straight to generation.
7. **Match-style mode** (two images: a reference + a target property
   photo) — analyze both, then produce a prompt using the *target's*
   subject/scene/composition fields with the *reference's* lighting/mood/
   style fields substituted in. If only one image is given but the user
   asked for match-style, ask for the second (reference) image before
   proceeding.
8. **Batch mode** (multiple photos in one request) — run steps 3–6 per
   image, then present one grouped set, image by image. Don't collapse
   distinct rooms/shots into a single averaged prompt.
9. **Prompt-to-video upgrade**, if requested or if the target is a reel —
   add a suggested camera movement appropriate to the inferred shot type
   (e.g. slow push-in for an establishing shot, lateral dolly for a room
   walkthrough, orbit for an aerial) on top of whichever variant was
   picked, turning it into a `create_reel`-ready prompt.
10. **Hand off**, once a variant is approved:
    - Single image → `generate_image`'s normal 3-step wizard.
    - Batch, as a set → `create_mixboard`.
    - Video-upgraded → `create_reel`.

## Tool mechanics

This skill calls the generation tools directly rather than through
`xelta-generate`, so it carries its own copy of the parts of their contract
that aren't obvious from the tool name alone:

- **generate_image** is a 3-step wizard — call with only `prompt` first (the
  tool returns a model list), call again with `model_id` (returns size
  options `1080p`/`2K`/`4K`), then call a third time with `prompt` +
  `model_id` + `size` to actually generate. Skip a step only if the user
  already named that value explicitly.
- **create_mixboard** (batch mode, as a set) returns `num_images` separate
  image URLs, not one composited image. Present them as a numbered list
  captioned with the count, not bare links with no framing.
- Avoid real public figures, sexual content, and trademarked
  characters/logos in any variant — generations of those are likely to be
  rejected.
- If a call fails with an auth-looking error, call `xelta_connection_status`
  and tell the user to check their Xelta connector if it reports
  disconnected. If `check_balance` comes back at 0 or errors, tell the user
  their account may need credits/re-auth rather than retrying in a loop.

## UX rules

1. Always show the four analysis fields before the three variants — the
   user should see *why* a variant reads the way it does, not just the
   output.
2. Three variants, always — not two, not five.
3. No raw IDs or JSON in chat — labeled fields and prompt text only.
4. Confidence flags are inline and brief, not a separate disclaimer block.
5. Don't call a generation tool until a variant is picked and, if the user
   wants to tweak it, until the edit is settled.
6. Detect the user's language and reply in it; prompt text itself is
   normalized to English.

## Errors

- No image / unreachable image → ask for one, don't proceed on a guess.
- Match-style requested with only one image → ask for the reference image.
- Image content is unclear even after a careful look (e.g. heavily
  cropped, too dark) → say what's uncertain and ask the user to fill the
  gap rather than inventing specifics.
