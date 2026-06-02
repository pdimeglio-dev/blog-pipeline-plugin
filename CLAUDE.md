# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin (`blog-pipeline`) that automates a daily content pipeline: local git history → MDX blog post + DALL-E 3 cover image → LinkedIn/X copy-paste text. Primary goal is landing engineering interviews by publishing 3–5 posts/week to [dimeglio.dev](https://dimeglio.dev).

Full design spec: [blog-pipeline-design.md](blog-pipeline-design.md). This is the source of truth for all agent specs, config schemas, and locked decisions.

## Plugin Structure (to build)

```
blog-pipeline-plugin/
├── .claude-plugin/plugin.json   ← manifest
├── .mcp.json                    ← tool connections (bash-only for now)
├── skills/
│   ├── reporter/SKILL.md        ← Phase 1
│   ├── writer/SKILL.md          ← Phase 2
│   ├── rater/SKILL.md           ← Phase 3
│   ├── publisher/SKILL.md       ← Phase 4
│   └── social-manager/SKILL.md  ← Phase 5
├── commands/run-pipeline.md     ← /blog-pipeline:run slash command
└── projects.config.json         ← project list + image themes
```

Skills live at `skills/<name>/SKILL.md` — not `.claude/skills/`.

## Build Order

Build sequentially — each phase depends on the previous output contract.

| Phase | Deliverable |
|---|---|
| 1 — Reporter | Git extraction → structured JSON. Test on 2 real repos before proceeding. |
| 2 — Writer | Reporter JSON → MDX + DALL-E 3 cover image |
| 3 — Rater | Draft MDX → score + pass/fail (threshold: 6.5 composite) |
| 4 — Publisher | `editorial_state.json` + project selection + daily orchestration |
| 5 — Social Manager | Approved draft → LinkedIn + X copy-paste text |
| 6 — Review UI | Cowork artifact: draft queue dashboard (Phase 4 dependent) |

## Agent Models

| Agent | Model | Reason |
|---|---|---|
| Reporter | Haiku | Mechanical data extraction, no creativity |
| Rater | Haiku | Rubric-based scoring, no creativity |
| Publisher | Haiku | Orchestration logic, no creativity |
| Writer | Sonnet | Voice adherence and writing quality matter |
| Social Manager | Sonnet | Platform-native tone matters |

## Locked Design Decisions

- **Repo access:** Local repos only via bash (`git log`, `git diff --stat`, read CHANGELOG + README)
- **Publishing:** Drop `.mdx` into `dimeglio.dev/content/blog/<slug>.mdx` with `published: false`, then `npm run ingest`
- **Draft queue:** `published: false` files in the dimeglio.dev repo — no separate store
- **Images:** DALL-E 3 via existing `OPENAI_API_KEY` (same key as ingest embeddings) → `public/blog/<slug>/cover.png`
- **Social output:** Copy-paste text only — no direct API posting
- **State file:** `editorial_state.json` lives in the plugin root alongside `projects.config.json` and `series.config.json`

## Critical External Dependency — dimeglio.dev AGENTS.md

The Writer skill **must read `dimeglio.dev/AGENTS.md` fresh on every run** before generating any draft. That file contains the canonical voice rules (banned buzzwords, required style). Do not hardcode voice rules into the skill — read the file.

## Blog Post Frontmatter Schema

Every MDX draft must include all of these fields:

```yaml
title:         "Post title"
description:   "Short summary for listing cards and OG meta"
date:          "2026-05-08"
lastModified:  "2026-05-08"
tags:          ["react", "architecture"]
coverImage:    "/blog/my-slug/cover.png"
coverAlt:      "Descriptive alt text"
published:     false
```

## Reporter Output Contract (downstream agents depend on this)

```json
{
  "project_id": "thepaddlegames",
  "period": { "from": "2026-04-24", "to": "2026-05-08" },
  "low_signal": false,
  "summary": "One paragraph overview",
  "recent_features": [{ "title": "...", "description": "...", "commits": ["sha1"] }],
  "bugs_fixed": [{ "title": "...", "impact": "..." }],
  "architecture_changes": [{ "title": "...", "before": "...", "after": "..." }],
  "highlights": ["most impressive item"],
  "tech_signals": ["skills visible in this work period"]
}
```

If fewer than 3 meaningful commits are found, set `low_signal: true`.

## Publisher Scoring Formula

Each morning, Publisher scores all projects to select 2–3:

- Base = project priority (1–5)
- +2 for every 7 days since last post
- −3 if posted within `min_days_between_posts`
- −10 if `max_posts_per_month` reached
- +1 if last Reporter run flagged high-signal activity

## DALL-E 3 Cover Image Prompt Template

```
"A modern, cinematic blog cover image for a tech article titled '{title}'."
"Background: true black (#000000). Primary accent: {accent_primary}. Secondary: {accent_secondary}."
"Style: {style_keywords}. Mood: {mood}."
"No faces, no text, no logos. Abstract geometric shapes and light effects only."
"Aspect ratio: 16:9 (1200x630). Dark, premium, editorial quality."
```

Model: `dall-e-3` · Size: `1792x1024` · Quality: `standard`

## Plugin Installation (once built)

```bash
claude plugin marketplace add pdimeglio/blog-pipeline-plugin
claude plugin install blog-pipeline
```
