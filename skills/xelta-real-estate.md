---
version: 0.1.0
name: xelta-real-estate
description: |
  Turn a short real-estate idea or listing detail into camera-and-lighting-
  aware prompts for Xelta generation — single shots, real-estate shot-type
  templates (curb appeal, room walkthrough, kitchen/bath close-up, aerial,
  twilight, before/after), multi-shot reel shot lists with per-scene
  durations, a session-persisted "house style," paired caption/hook copy,
  and batch prompt sets for a whole listing. Use when: "real estate reel",
  "property walkthrough video", "curb appeal shot", "twilight exterior
  shot", "shot list for this listing", "keep this house style for the rest
  of the shoot", "caption for this listing photo". NOT for turning an
  existing photo into a prompt via vision analysis (use
  xelta-image-to-prompt) or for generic non-real-estate prompts (use
  xelta-prompt-enhance).
argument-hint: "[listing idea or shot type] [--reel|--batch] [--style bright|luxury|moody]"
allowed-tools: mcp__claude_ai_xelta__generate_image, mcp__claude_ai_xelta__create_mixboard, mcp__claude_ai_xelta__create_reel, mcp__claude_ai_xelta__check_balance, mcp__claude_ai_xelta__xelta_connection_status
---

# Xelta Real Estate

Real-estate-specialized prompt builder that sits in front of `xelta-generate`.
Builds camera/lighting/mood-aware prompts from real-estate shot conventions,
then hands the result into `generate_image` / `create_mixboard` / `create_reel`
the same way `xelta-prompt-enhance` hands off. Doesn't call a generation tool
until the user has a prompt — or shot list — they're happy with.

## Photos, not descriptions

This skill builds prompts from what the user *tells* you about a shot. If
they instead want to upload a property photo and get a prompt back from it —
including "match this style" mode and confidence flags on ambiguous details —
that's `xelta-image-to-prompt`. Hand off to it rather than guessing at a
photo's contents from a filename or description alone.

## Workflow

1. **Determine mode:**
   - **Single shot** — one idea, one prompt. Default.
   - **Shot-type template** — user names or implies a specific real-estate
     shot; see `references/shot-templates.md`.
   - **Reel** — user wants a sequence/walkthrough/video, not one image.
   - **Batch** — user lists multiple rooms/spaces in one request and wants a
     full prompt set for the listing.
   If genuinely ambiguous between modes, ask once — don't guess.
2. **Apply the active house style, if one is set** (see "Brand/style
   consistency") to whichever mode is running, before expanding.
3. **Expand the idea into three levels**, the same way `xelta-prompt-enhance`
   offers 3 options — except here the three are levels of the *same* shot,
   not different directions:
   - **Basic** — subject + setting + one style cue.
   - **Detailed** — + lighting (time of day, light source/direction, warm vs.
     cool tone) and lens/framing.
   - **Cinematic** — + camera movement (slow dolly-in, drone reveal, gimbal
     walk) and pacing/mood — ready for `create_reel`.
   Present numbered 1/2/3, short and scannable. Wait for the user's pick;
   don't default to level 1 and generate automatically.
4. **Shot-type templates** — for a named shot type, start from the matching
   entry in `references/shot-templates.md` instead of a blank expansion;
   still offer the 3 levels on top of it.
5. **Reel mode** — output a shot list, not one prompt: 6–8 scenes in
   shooting order, each numbered with a one-line prompt and a suggested
   duration (2–5s per scene is typical for a listing reel; total should land
   near 30–60s). This list is what feeds `create_reel` — ask the user
   whether they want one `create_reel` call per scene, or a single call
   whose `prompt` describes the full sequence.
6. **Batch mode** — for multiple rooms/spaces named in one request, produce
   one prompt (or shot) per room, grouped under the listing, reusing the
   active house style for all of them. This works from room *descriptions*
   the user gives you in chat — if they want to batch-generate from
   uploaded photos of each room instead, hand off to
   `xelta-image-to-prompt`'s batch mode.
7. **Caption pairing** — if the user mentions listing details worth
   surfacing (price, sqft, "Just Listed", beds/baths), offer a short one-line
   caption/hook alongside the visual prompt. Skip it if nothing in the
   request lends itself to a caption — don't force one in.
8. **Hand off**, once a prompt (or shot list) is approved:
   - Single image → `generate_image`'s normal 3-step wizard (model, then
     size), using the chosen prompt.
   - Grid of options → `create_mixboard`.
   - Reel/walkthrough → `create_reel`, per the split decided in step 5.

## Tool mechanics

This skill calls the generation tools directly rather than through
`xelta-generate`, so it carries its own copy of the parts of their contract
that aren't obvious from the tool name alone:

- **generate_image** is a 3-step wizard — call with only `prompt` first (the
  tool returns a model list), call again with `model_id` (returns size
  options `1080p`/`2K`/`4K`), then call a third time with `prompt` +
  `model_id` + `size` to actually generate. Skip a step only if the user
  already named that value explicitly.
- **create_mixboard** returns `num_images` (default 8) separate image URLs,
  not one composited image. Present them as a numbered list captioned with
  the count, not bare links with no framing.
- Avoid real public figures, sexual content, and trademarked
  characters/logos in any prompt/shot built here — generations of those are
  likely to be rejected.
- If a call fails with an auth-looking error, call `xelta_connection_status`
  and tell the user to check their Xelta connector if it reports
  disconnected. If `check_balance` comes back at 0 or errors, tell the user
  their account may need credits/re-auth rather than retrying in a loop.

## Brand/style consistency

- If the user states a house style ("keep everything bright and minimal",
  "warm & luxury for this listing", "moody & editorial"), remember it for
  the rest of the session and fold it into every subsequent prompt/template
  without re-asking.
- Three reference styles the user's own wording will usually map onto:
  **bright & minimal** (soft daylight, neutral palette, clean lines),
  **warm & luxury** (golden hour, rich materials, soft interior fill),
  **moody & editorial** (low-key contrast, deep shadow, dramatic
  single-source light). If their wording doesn't fit one of the three, use
  it verbatim instead of forcing a bucket.
- A style set for one listing doesn't automatically carry over to a clearly
  new, unrelated listing later in the same session — confirm if it's
  ambiguous whether they're still on the same property.

## UX rules

1. Three expansion levels, always — not two, not five (same discipline as
   `xelta-prompt-enhance`).
2. No raw IDs or JSON in chat — only prompt text, and for reels, the
   numbered shot list.
3. Don't call a generation tool until the user has picked a level/template
   or approved a shot list.
4. Detect the user's language and reply in it; prompt text itself is
   normalized to English (generation models perform best on it).
5. Ask one thing at a time. If mode, style, and level are all unresolved,
   resolve mode first, then style (only if never set this session), then
   present the three levels.

## Errors

- User uploads or references a photo and asks for a prompt back → hand off
  to `xelta-image-to-prompt` instead of guessing at the photo from context.
- Batch/reel request with no room or scene details at all → ask for a quick
  list of spaces/shots before generating a set; don't invent listing details.
- Ambiguous mode (could be single shot or reel) → ask once, don't guess and
  generate the wrong shape.
