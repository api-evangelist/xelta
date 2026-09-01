---
version: 0.1.0
name: xelta-prompt-enhance
description: |
  Take a user's rough or short creative prompt (English or Hinglish) and turn
  it into three distinct enhanced prompt options before generating anything,
  then hand the one the user picks to the right Xelta tool. Use when: "make
  this prompt better", "give me a few options for this", "enhance my prompt",
  or as a default step before generate_image/create_mixboard/create_reel/
  generate_street_ad/build_website when the user's raw prompt is short or
  vague. NOT for prompts the user has already written in full, specific
  detail (skip straight to xelta-generate/xelta-websites) or for anything
  that isn't a creative/generation prompt.
argument-hint: "[raw prompt] [--for image|mixboard|reel|street-ad|website]"
allowed-tools: mcp__claude_ai_xelta__generate_image, mcp__claude_ai_xelta__create_mixboard, mcp__claude_ai_xelta__create_reel, mcp__claude_ai_xelta__generate_street_ad, mcp__claude_ai_xelta__build_website, mcp__claude_ai_xelta__check_balance, mcp__claude_ai_xelta__xelta_connection_status
---

# Xelta Prompt Enhance

Sits in front of the other two skills. Doesn't call any tool by itself
until the user has picked one of three enhanced prompt options — then hands
that chosen prompt into the normal `xelta-generate` / `xelta-websites` flow
for whichever action it's for.

## When to enhance vs. skip

- **Enhance** when the user's raw prompt is short, vague, or just an idea
  (e.g. "a dragon", "coffee shop website", "kuch cool sa banado ek shehar
  ka"). This is the common case — default to enhancing.
- **Skip straight to generation** when the user has already written a
  full, specific prompt (subject + setting + style, or a clear site brief)
  — don't make them sit through options for a prompt they already
  finished writing. If unsure, ask once: "want me to give you a few
  enhanced variants, or generate this as-is?"
- **Always enhance** if the user explicitly asks for options/enhancement,
  regardless of how detailed their prompt already is.

## Workflow

1. **Identify the target action** — image, mixboard, reel, street ad, or
   website. Infer from context (mentions of "video", "site", "grid of
   images", "graffiti ad") or ask once if genuinely ambiguous.
2. **Detect input language.** The user may write in English, Hindi, or
   Hinglish (romanized Hindi/English mix). Understand the intent
   regardless of which.
3. **Generate exactly three enhanced prompt options.** Each one:
   - Is a distinct creative direction, not three phrasings of the same
     thing (vary angle/mood/style/composition/setting — see "What makes
     an option distinct" below).
   - Is normalized into clear English, even if the user typed Hinglish —
     generation models perform best on English prompts. Only the prompt
     *text* is translated/normalized; your chat messages to the user stay
     in whichever language they used.
   - Follows the target tool's prompt shape (see "Per-target guidance").
4. **Present the three options numbered 1/2/3**, short and scannable, in
   the user's own language for any surrounding explanation. Don't call
   any generation tool yet.
5. **Wait for the user's pick.** They may pick a number, ask to tweak one,
   or ask for a fresh set of three — accommodate any of these before
   moving on.
6. **Hand off.** Once a prompt is chosen, feed it into the matching tool's
   normal flow:
   - Image → `generate_image`'s 3-step wizard (model, then size) using the
     chosen prompt as `prompt`.
   - Mixboard → `create_mixboard` with the chosen prompt as `command`.
   - Reel → `create_reel` with the chosen prompt as `prompt`.
   - Street ad → `generate_street_ad`; the chosen prompt informs `style`/
     description, `brand` is still asked for separately if not given.
   - Website → `build_website`'s chosen prompt as `prompt`; still ask for
     `industry` and `pages` if not already known.

## What makes an option distinct

Vary one or two real axes per option, not just word choice:

- **Style/medium** — e.g. cinematic photo vs. oil painting vs. 3D render.
- **Mood/lighting** — e.g. golden hour vs. moody neon-lit night vs. bright
  and airy.
- **Composition/angle** — e.g. close-up vs. wide establishing shot vs.
  dynamic low angle.
- **Tone (for websites)** — e.g. minimal/corporate vs. bold/energetic vs.
  warm/personal.

## Per-target guidance

- **Image/mixboard/street-ad/reel** — subject + setting + style in one
  sentence beats a list of adjectives; name camera/lighting/medium when it
  matters; for reels describe motion, not just the static scene; phrase
  exclusions positively (no negative-prompt field); keep each option well
  under ~200 words.
- **Website** — one or two sentences describing the business/purpose and
  tone, e.g. "A warm, minimal portfolio site for a freelance UX designer,
  soft neutral palette" — this becomes `build_website`'s `prompt`.

## Tool mechanics

This skill calls the generation tools directly, so it carries its own copy
of the parts of their contract that aren't obvious from the tool name alone:

- **generate_image** is a 3-step wizard — call with only `prompt` first (the
  tool returns a model list), call again with `model_id` (returns size
  options `1080p`/`2K`/`4K`), then call a third time with `prompt` +
  `model_id` + `size` to actually generate. Skip a step only if the user
  already named that value explicitly.
- **create_mixboard** returns `num_images` (default 8) separate image URLs,
  not one composited image. Present them as a numbered list captioned with
  the count (e.g. "Here are your 8 options:"), not bare links with no
  framing.
- Avoid real public figures, sexual content, and trademarked
  characters/logos in any enhanced option — generations of those are likely
  to be rejected.
- If a call fails with an auth-looking error, call `xelta_connection_status`
  and tell the user to check their Xelta connector if it reports
  disconnected. If `check_balance` comes back at 0 or errors, tell the user
  their account may need credits/re-auth rather than retrying in a loop.

## UX rules

1. Three options, always — not two, not five.
2. No raw IDs or JSON in chat; show only the prompt text for each option.
3. Don't call a generation tool until the user has picked.
4. Detect the user's language and reply in it; only the enhanced prompt
   text itself gets normalized to English.
5. If the user rejects all three, offer to generate three new ones with a
   different angle rather than repeating the same set.

## Errors

- User gives an empty or non-creative prompt ("hi", "test") → ask what
  they'd like to make instead of fabricating three options for nothing.
- Ambiguous target action (could be image or website) → ask once, don't
  guess and generate the wrong thing.
