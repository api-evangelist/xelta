# Xelta AI Skills

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/version-0.1.0-green.svg)](./VERSION)
[![Skills](https://img.shields.io/badge/skills-5-blueviolet.svg)](#skills)

AI agent skills for image, prompt, and website generation via [Xelta AI](https://xelta.ai). Works with Claude Code, Cursor, Codex, and other AI coding agents that load Markdown-based skills.

## Install

Pick one. Xelta's tools are MCP tools connected directly in your agent — no CLI to install or separate login step.

### `npx skills` — recommended, cross-agent

```bash
npx skills add matchbest-group/xelta_skills
```

### `gh skill install`

GitHub CLI v2.90+:

```bash
gh skill install matchbest-group/xelta_skills
```

### Claude Code marketplace

Inside Claude Code:

```
/plugin marketplace add matchbest-group/xelta_skills
/plugin install xelta@xelta
```

### Setup script

Universal fallback:

```bash
git clone --depth 1 https://github.com/matchbest-group/xelta_skills.git
cd xelta_skills
./setup
```

More options in [INSTALL.md](./INSTALL.md). Agent-driven install (paste into your agent): [INSTALL_FOR_AGENTS.md](./INSTALL_FOR_AGENTS.md).

## Skills

| Skill | Invoke | Description |
|---|---|---|
| [`xelta-generate`](./xelta-generate) | `/xelta:generate` | Generate images (model + resolution wizard), mixboards (grids of AI images from one prompt), short video reels, and street-art / graffiti-style brand ads via Xelta's MCP tools. |
| [`xelta-websites`](./xelta-websites) | `/xelta:websites` | Generate and optionally deploy an AI website with Xelta's Pomeli Website Builder, and look up previously generated sites. |
| [`xelta-prompt-enhance`](./xelta-prompt-enhance) | `/xelta:prompt-enhance` | Turn a rough prompt (English or Hinglish) into three distinct enhanced options to pick from, then hand the chosen one to the right skill above. |
| [`xelta-real-estate`](./xelta-real-estate) | `/xelta:real-estate` | Real-estate-specialized prompt builder: shot-type templates, multi-scene reel shot lists, a session-persisted house style, caption pairing, and batch prompt sets for a listing. |
| [`xelta-image-to-prompt`](./xelta-image-to-prompt) | `/xelta:image-to-prompt` | Reverse-prompting for property photos: scene/lighting/style/material breakdown, three prompt variants, match-style mode, batch upload, and an optional prompt-to-video upgrade. |

## Quick Reference

| What you want | Skill | Note |
|---|---|---|
| Generate an image from a prompt | `xelta-generate` | Walks a 3-step wizard: pick model, then resolution (`1080p` / `2K` / `4K`) |
| A grid of AI images from one creative prompt | `xelta-generate` | Mixboard, defaults to 8 images |
| A short AI video reel | `xelta-generate` | Optional seed image via `image_url` |
| A street-art / graffiti-style brand ad | `xelta-generate` | Style/surface default to `auto` / `brick` |
| Check remaining Xelta credits | `xelta-generate` | `check_balance` |
| Build / generate a website or landing page | `xelta-websites` | Requires a prompt, industry, and page list; `deploy: true` publishes live to Vercel |
| See recently generated websites | `xelta-websites` | Returns live/preview URLs |
| Turn a rough/short idea into a few polished prompt options | `xelta-prompt-enhance` | Gives exactly 3 options; nothing generates until you pick one |
| A real-estate shot (curb appeal, walkthrough, kitchen/bath, aerial, twilight, before/after) | `xelta-real-estate` | Offers basic/detailed/cinematic levels; templates in `references/shot-templates.md` |
| A property reel shot list, or a full prompt set for a listing | `xelta-real-estate` | Reel mode outputs 6–8 scenes with durations; batch mode covers multiple rooms in one request |
| Turn an uploaded property photo into a prompt | `xelta-image-to-prompt` | Vision-based, no separate Xelta tool call for the analysis step; gives 3 variants |
| Apply one photo's style/lighting to a different property photo | `xelta-image-to-prompt` | "Match this style" mode; needs both images |

## License

MIT — see [LICENSE](./LICENSE).
