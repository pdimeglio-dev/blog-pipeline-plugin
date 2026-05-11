# Blog Pipeline System — Design Document

**dimeglio.dev · May 2026**

> Automate a daily content pipeline that turns active engineering work into polished blog posts and social media content — with the primary goal of landing interviews. One plugin. One config. Fully agentic from raw git history to published MDX file.
>
> **Decisions locked:** local repos via bash · copy-paste social output · DALL-E 3 images · publishing = drop MDX + `npm run ingest` · draft queue = `published: false` files

---

## Table of Contents

1. [Goals & Success Criteria](#1-goals--success-criteria)
2. [dimeglio.dev — Publishing Architecture](#2-dimegliodev--publishing-architecture)
3. [Pipeline Architecture](#3-pipeline-architecture)
4. [Configuration Schema](#4-configuration-schema)
5. [Agent Specifications](#5-agent-specifications)
6. [Plugin Repo & Installation](#6-plugin-repo--installation)
7. [Build Order & Phases](#7-build-order--phases)
8. [Open Questions — Status](#8-open-questions--status)
9. [Future Ideas (Post-MVP)](#9-future-ideas-post-mvp)

---

## 1. Goals & Success Criteria

The pipeline exists to serve one outcome: help Pablo land engineering interviews by building a consistent, visible, high-quality public record of his work.

### Primary Goal

- Publish 3–5 blog posts per week to dimeglio.dev, each showcasing real engineering decisions, problem-solving, and technical depth.
- Distribute every post to LinkedIn and X as copy-paste-ready text, adapted for each platform's format and audience.
- Maintain variety across projects — no single project dominates the feed.

### Success Looks Like

- A queue of 2–3 rated drafts waiting each morning, ready for one-click approval.
- Posts that read like genuine engineering reflection, not automated changelogs.
- Social content that consistently signals: "this person ships things and thinks deeply."
- Zero manual work beyond reading drafts and hitting publish.

> **Key Constraint — Every agent must know why this exists**
>
> The Rater scores low on trivial changes. The Writer frames work through the lens of skills demonstrated, not just features shipped. The Social Manager writes for recruiters and senior engineers, not just followers. The goal is interviews, not impressions.

---

## 2. dimeglio.dev — Publishing Architecture

Before building the pipeline, we audited the dimeglio.dev repo to understand exactly how content flows from files to the live site. Everything the pipeline produces must conform to this system.

### Tech Stack

| Layer | Details |
|---|---|
| Framework | Next.js 16 (App Router) · TypeScript · Tailwind CSS v4 · shadcn/ui |
| Content | MDX files with typed frontmatter — parsed by gray-matter, rendered by next-mdx-remote/rsc |
| Code | rehype-pretty-code + Shiki (github-dark theme) |
| Deployment | Vercel — auto-deploys on git push to main |
| RAG | Pinecone vector store + OpenAI text-embedding-3-small — synced via `npm run ingest` |
| Monitoring | PostHog analytics + Sentry error tracking |

### Publishing a Blog Post — Exact Steps

> **Two steps. That's it.**
>
> 1. Drop an `.mdx` file into `content/blog/<slug>.mdx` with `published: true` in frontmatter
> 2. Run: `npm run ingest`
>
> The ingest script syncs all content to Pinecone (so Guillermo the AI assistant can find the new post) and auto-regenerates `lib/generated-blog-slugs.ts` (so Guillermo can link directly to the post). Then git commit + push triggers Vercel deploy.

### Draft Queue via `published: false`

The pipeline does NOT maintain a separate draft store. It uses the repo itself. Drafts are dropped into `content/blog/<slug>.mdx` with `published: false` — invisible on the site but readable in the editor. When Pablo approves a draft, he flips to `published: true` and runs `npm run ingest`.

### Blog Post Frontmatter Schema

The pipeline must produce all of these fields:

```yaml
---
title:         "Post title"
description:   "Short summary (shown in listing cards and OG meta)"
date:          "2026-05-08"
lastModified:  "2026-05-08"
tags:          ["react", "architecture"]
coverImage:    "/blog/my-slug/cover.png"
coverAlt:      "Descriptive alt text for the cover image"
published:     false
---
```

### Blog Voice Rules (from `AGENTS.md`)

The `AGENTS.md` in the dimeglio.dev repo has a detailed anti-AI-detection ruleset. **The Writer skill must read this file before generating any draft.**

#### Banned — immediate AI tells

- Buzzwords: "force multiplier", "game-changer", "paradigm shift", "revolutionize", "empower", "leverage" (verb), "delve", "tapestry", "landscape", "harness", "cutting-edge", "robust", "seamless", "humbling"
- Dramatic pauses: "That's not a typo", "Here's the beautiful part", "Let that sink in", "Here's the thing nobody tells you about X"
- Defensive framing: "This isn't about AI replacing engineers"
- Mic-drop endings, inspirational wrap-ups, profound conclusions
- Self-referential cleverness — explaining why something is ironic instead of letting the reader notice

#### Required — what real posts sound like

- Open with something personal — a frustration, a confession, a specific moment
- Show friction — what went wrong, what was annoying, what took too long
- Use imperfect transitions — "Anyway,", "So,", "The other thing was..."
- First person, casual — contractions, fragments, parenthetical asides
- Be specific — "40 minutes fighting the typography plugin" not "some challenges"
- End honestly — a practical takeaway or open question, never inspirational

> **Writer self-check before finalising any draft**
>
> ✓ Sounds like a dev blog by a real person → ship it
> ✗ Sounds like a Medium article or marketing case study → rewrite it

---

## 3. Pipeline Architecture

One Cowork plugin, installed at the user level. Five agents. One config file. The Publisher is the only stateful component — everything else is stateless and reads from config.

### Daily Flow

| Step | What happens |
|---|---|
| Trigger | Scheduler wakes Publisher each morning (configurable time) |
| Step 1 | Publisher reads `editorial_state.json` → scores projects → selects 2–3 for today |
| Step 2 | Reporter runs on each selected project path via bash `git log` + CHANGELOG + README |
| Step 3 | Writer takes Reporter JSON + project config → writes MDX draft + image prompt |
| Step 4 | Writer calls DALL-E 3 with the image prompt → saves PNG to `public/blog/<slug>/` |
| Step 5 | Rater scores draft on substance, showcase value, writing quality (threshold: 6.5) |
| Step 6 | Passing drafts dropped into `content/blog/<slug>.mdx` with `published: false` |
| Step 7 | Pablo reviews drafts → flips `published: true` → runs `npm run ingest` → git push |
| Step 8 | Social Manager generates copy-paste LinkedIn + X text alongside each draft |

### Agent Overview

| Agent | Role |
|---|---|
| Reporter | Reads git log, CHANGELOG, README for a local repo path. Outputs structured JSON. |
| Writer | Converts Reporter JSON + project config into a full MDX post + DALL-E 3 cover image. |
| Rater | Scores drafts 1–10 on three axes. Filters noise. Threshold 6.5 composite to pass. |
| Publisher | Owns `editorial_state.json` and the project selection/scheduling logic. |
| Social Mgr | Produces copy-paste LinkedIn narrative + X hook/thread per approved draft. |

---

## 4. Configuration Schema

A single JSON config file inside the plugin repo. The only file Pablo edits manually when adding a project or changing its tone.

### `projects.config.json` — per-project fields

```
id                      Unique slug e.g. "thepaddlegames"
name                    Display name e.g. "The Paddle Games"
path                    Absolute local path to the repo
git_remote              Default "origin"
lookback_days           How far back Reporter scans (default 14)
tone                    Writing personality — freeform text description
skills_showcased        string[] — skills to foreground in posts
min_days_between_posts  Cadence guard (default 5)
max_posts_per_month     Diversity cap (default 6)
priority                1–5, higher = featured more often when tied
image_theme             Object — see image theme schema below
```

### `image_theme` — per-project visual identity

Each project gets its own cover image aesthetic. The Writer reads this block and constructs the DALL-E 3 prompt. All projects share a dark background — only accents differ.

```
background         Hex — always dark e.g. "#000000"
accent_primary     Hex — main accent colour
accent_secondary   Hex — optional secondary accent
style_keywords     Comma-separated visual motifs for the prompt
mood               Emotional/tonal adjectives for the prompt
```

### Project Visual Identities

| Project | Accent Colours | Style & Mood |
|---|---|---|
| dimeglio.dev | `#a855f7` (purple) + `#6366f1` (indigo) | AI neural nets, geometric shapes, flowing data. Intelligent, sophisticated, forward-thinking. |
| The Paddle Games | `#4ade80` (lime) + `#06b6d4` (teal/cyan) | Wave forms, kinetic energy, water dynamics, sports tech. Energetic, competitive, fluid. |
| The Batcave | `#f7dd30` (Batman yellow) | Gothic geometry, dramatic light beams, signal radiating shapes. Dark, dramatic, technical. |
| Default | `#93c5fd` (cool blue-white) | Clean abstract tech. Used for any project until a custom theme is defined. |

### Full Example — The Paddle Games

```json
{
  "id": "thepaddlegames",
  "name": "The Paddle Games",
  "path": "~/projects/thepaddlegames.com",
  "git_remote": "origin",
  "lookback_days": 14,
  "tone": "playful, product-focused, accessible to non-developers",
  "skills_showcased": ["React", "real-time game logic", "UX design", "WebSockets"],
  "min_days_between_posts": 5,
  "max_posts_per_month": 4,
  "priority": 3,
  "image_theme": {
    "background": "#000000",
    "accent_primary": "#4ade80",
    "accent_secondary": "#06b6d4",
    "style_keywords": "wave forms, kinetic energy, water dynamics, sports tech",
    "mood": "energetic, competitive, fluid"
  }
}
```

---

## 5. Agent Specifications

### 5.1 Reporter

Read-only. Gathers raw evidence of work done from local git history. Does not editorialize — that's the Writer's job. Runs via bash on the local repo path.

**Model: Haiku** (no creativity needed — mechanical data extraction)

#### Inputs
- Project config entry (path, git_remote, lookback_days)
- Optional date range override

#### Process — bash commands on local repo
- `git log --oneline --since="14 days ago"` — commit messages, SHAs, timestamps
- `git diff --stat HEAD~N HEAD` — files changed, insertions/deletions for scope
- Read `CHANGELOG.md` or `RELEASES.md` if present
- Read `README.md` — project description and tech stack context
- Group commits into logical themes (not one bullet per commit)
- If < 3 meaningful commits found → set `low_signal: true` to let Publisher skip today

#### Output — structured JSON

```json
{
  "project_id": "thepaddlegames",
  "period": { "from": "2026-04-24", "to": "2026-05-08" },
  "low_signal": false,
  "summary": "One paragraph overview of the work period",
  "recent_features": [{ "title": "...", "description": "...", "commits": ["sha1"] }],
  "bugs_fixed": [{ "title": "...", "impact": "..." }],
  "architecture_changes": [{ "title": "...", "before": "...", "after": "..." }],
  "highlights": ["most impressive or interesting item"],
  "tech_signals": ["skills visible in this work period"]
}
```

---

### 5.2 Writer

Transforms Reporter JSON into a complete MDX file ready to drop into `content/blog/`. Also generates and downloads the cover image via DALL-E 3.

**Model: Sonnet** (writing quality and voice adherence matter)

#### Inputs
- Reporter JSON output
- Project config (tone, skills_showcased, image_theme)
- Blog voice rules from `dimeglio.dev/AGENTS.md` — **read this file fresh every run**

#### Cover Image — DALL-E 3

The `dimeglio.dev` repo has `scripts/generate-cover-prompt.mjs` as a reference. The Writer extends this: builds a prompt using the post's frontmatter + project `image_theme`, calls DALL-E 3 (same OpenAI API key used by the ingest script), saves to `public/blog/<slug>/cover.png`.

```
Prompt structure:
"A modern, cinematic blog cover image for a tech article titled '{title}'."
"Background: true black (#000000). Primary accent: {accent_primary}. Secondary: {accent_secondary}."
"Style: {style_keywords}. Mood: {mood}."
"No faces, no text, no logos. Abstract geometric shapes and light effects only."
"Aspect ratio: 16:9 (1200x630). Dark, premium, editorial quality."

Model: dall-e-3 · Size: 1792x1024 · Quality: standard
Saved to: public/blog/{slug}/cover.png
```

#### MDX Output

Complete `.mdx` file written to `content/blog/<slug>.mdx`:

```mdx
---
title:        "Post title"
description:  "2-sentence summary for listing + OG"
date:         "2026-05-08"
lastModified: "2026-05-08"
tags:         ["react", "architecture"]
coverImage:   "/blog/my-slug/cover.png"
coverAlt:     "Descriptive alt text for the cover image"
published:    false
---

Full MDX body — 400–800 words, first person, casual, shows friction and specific detail.
Opens with a personal frustration or specific moment. Ends with a practical takeaway.
No buzzwords. No mic-drop endings. Sounds like a real dev blog.
```

---

### 5.3 Rater

Quality gate. Scores each draft and decides whether it enters the queue. Aware of the interview-landing goal.

**Model: Haiku** (structured scoring with a rubric — no creativity needed)

#### Scoring Dimensions

| Dimension | Weight | Description |
|---|---|---|
| Substance (1–10) | 40% | Real engineering work? Decision-making visible? Not just feature delivery? |
| Showcase Value (1–10) | 40% | Would a hiring manager be impressed? Signals the skills Pablo wants to be known for? |
| Writing Quality (1–10) | 20% | Worth reading? Opens with something interesting? Voice sounds human? |
| Composite | — | Must be ≥ 6.5 to pass. Also flags any banned phrases from AGENTS.md voice rules. |

#### Output

```json
{
  "scores": { "substance": 8, "showcase": 7, "writing": 8 },
  "composite": 7.6,
  "passes_threshold": true,
  "voice_flags": [],
  "reasoning": "Strong post — the architecture decision section shows real systems thinking.",
  "suggested_improvements": "The intro buries the lede. Lead with the perf problem, not the solution."
}
```

---

### 5.4 Publisher

The only stateful agent. Owns the editorial calendar, the project selection logic, and the daily orchestration.

**Model: Haiku** (orchestration logic — mechanical, no creativity needed)

#### `editorial_state.json`

Lives in the `dimeglio.dev` repo at `/editorial_state.json`. Committed to git — publishing history survives machine switches.

```json
{
  "projects": {
    "thepaddlegames": {
      "last_posted": "2026-05-01",
      "posts_this_month": 2,
      "last_reporter_run": "2026-05-07",
      "last_signal_level": "high"
    }
  },
  "draft_queue": [
    {
      "id": "draft-20260508-001",
      "project_id": "thepaddlegames",
      "slug": "paddle-games-realtime-sync",
      "title": "...",
      "composite_score": 7.6,
      "created_at": "2026-05-08T07:00:00Z",
      "status": "pending"
    }
  ]
}
```

#### Project Selection Logic

Each morning the Publisher scores every project:

- Base score = project priority (1–5)
- Bonus: +2 for every 7 days since last post
- Penalty: −3 if posted within `min_days_between_posts`
- Penalty: −10 if `max_posts_per_month` reached
- Bonus: +1 if Reporter flagged high-signal activity last run

Top 2–3 scoring projects are run through Reporter → Writer → Rater.

---

### 5.5 Social Media Manager

Runs after a draft passes the Rater. Produces copy-paste-ready text for LinkedIn and X. Both versions are interview-aware.

**Model: Sonnet** (platform-native tone matters)

#### LinkedIn Post

- 150–250 words, short paragraphs
- Structure: hook → what I solved / built → what was interesting or hard → CTA with link
- Tone: professional but human
- Ends with a question or thought to drive comments
- Explicitly surfaces one of the project's `skills_showcased`

#### X Post

- **Option A** — single tweet (~240 chars): punchy hook + link
- **Option B** — thread (3–6 tweets): if the topic has 3+ distinct ideas; last tweet has the link
- Tone: direct, technical, confident
- Max 2 hashtags. No "excited to share".

#### Output

```json
{
  "linkedin": {
    "body": "Full LinkedIn post text — ready to paste",
    "char_count": 198
  },
  "x": {
    "format": "thread",
    "tweets": [
      "Tweet 1 — hook",
      "Tweet 2 — key insight",
      "dimeglio.dev/blog/my-slug — full post"
    ]
  }
}
```

---

## 6. Plugin Repo & Installation

This section is the implementation brief. The `blog-pipeline-plugin` GitHub repo IS the plugin — no build step, no packaging infrastructure. Everything is markdown and JSON.

**Hand this document to Claude Code running inside the `blog-pipeline-plugin` folder to begin implementation.**

### Repo Structure

```
blog-pipeline-plugin/
├── .claude-plugin/
│   └── plugin.json          ← manifest (name, description, version)
├── .mcp.json                ← tool connections (empty for now — local bash only)
├── skills/
│   ├── reporter/
│   │   └── SKILL.md         ← Phase 1
│   ├── writer/
│   │   └── SKILL.md         ← Phase 2
│   ├── rater/
│   │   └── SKILL.md         ← Phase 3
│   ├── publisher/
│   │   └── SKILL.md         ← Phase 4
│   └── social-manager/
│       └── SKILL.md         ← Phase 5
├── commands/
│   └── run-pipeline.md      ← /blog-pipeline:run slash command
├── projects.config.json     ← project list + per-project config
└── README.md
```

### `plugin.json` Manifest

```json
{
  "name": "blog-pipeline",
  "displayName": "Blog Pipeline",
  "description": "Agentic blog content pipeline for dimeglio.dev — reporter, writer, rater, publisher, and social manager.",
  "version": "0.1.0",
  "author": "Pablo Di Meglio"
}
```

### Execution Model — Subagents

Each skill runs as a subagent spawned by the Publisher. This keeps context clean — the Reporter's raw git history never pollutes the Writer's context, and the Writer's MDX never pollutes the Rater's context.

| Agent | Role | Model |
|---|---|---|
| Publisher | Orchestrator. Reads state, selects projects, spawns all subagents sequentially. | Haiku |
| Reporter | Subagent. Spawned per project. Runs bash git commands. Returns JSON. | Haiku |
| Writer | Subagent. Takes Reporter JSON. Writes MDX + calls DALL-E 3. | Sonnet |
| Rater | Subagent. Takes draft MDX. Returns score + pass/fail. | Haiku |
| Social Manager | Subagent. Takes published post. Returns LinkedIn + X text. | Sonnet |

### Install & Reinstall

```bash
# Add your plugin repo as a marketplace source
claude plugin marketplace add pdimeglio/blog-pipeline-plugin

# Install the plugin
claude plugin install blog-pipeline

# Done — skills activate automatically in every Cowork session
```

**Reference:** [github.com/anthropics/knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) — 11 production plugins in the same format. Use the `engineering` or `productivity` plugin as a structural reference.

### State & Config Files Outside the Plugin

| File | Location | Notes |
|---|---|---|
| `projects.config.json` | Plugin repo root | Edit here to add/update projects. Committed to git. |
| `editorial_state.json` | `dimeglio.dev/` root | Committed — publishing history survives machine switches. |
| `public/blog/<slug>/` | `dimeglio.dev/public/` | Cover images written here by the Writer. |
| `content/blog/<slug>.mdx` | `dimeglio.dev/content/blog/` | Drafts written with `published: false`. Flipped on approval. |

---

## 7. Build Order & Phases

Build sequentially. Each phase depends on the previous. The Reporter output contract must be stable before the Writer is written.

| Phase | Deliverable | Depends On |
|---|---|---|
| Phase 1 — Reporter | Skill reads local git history + CHANGELOG + README. Outputs validated JSON on 2 real repos. | — |
| Phase 2 — Writer | Skill writes MDX draft + calls DALL-E 3 for cover image. Follows voice rules from AGENTS.md. | Phase 1 |
| Phase 3 — Rater | Skill scores drafts, applies voice-flag check, outputs pass/fail with reasoning. | Phase 2 |
| Phase 4 — Publisher | `editorial_state.json` + project selection scoring + daily scheduler integration. | Phase 3 |
| Phase 5 — Social Manager | LinkedIn + X copy-paste output. Tuned per platform. | Phase 4 |
| Phase 6 — Review UI | Cowork artifact: draft queue dashboard — title, score, project, approve/reject. | Phase 4 |

> **Start with Phase 1.** Run the Reporter on `dimeglio.dev` and The Paddle Games repos before writing any other agent. Inspect the output JSON. Everything downstream depends on it producing clean, structured, high-signal data.

---

## 8. Open Questions — Status

| Question | Decision | Status |
|---|---|---|
| Publishing mechanism | Drop MDX into `content/blog/` + `npm run ingest` + git push. Vercel autodeploys. | ✓ Resolved |
| Draft queue location | `published: false` files in `content/blog/` — the repo is the queue. | ✓ Resolved |
| Repo access | Local repos only via bash. All project repos are on Pablo's machine. | ✓ Resolved |
| Image generation | DALL-E 3 via existing OpenAI key (same key as ingest embeddings). No new setup. | ✓ Resolved |
| Social posting | Copy-paste text output only. No direct API posting for now. | ✓ Resolved |
| Project image themes | Per-project `image_theme` block in config. Three initial themes defined. | ✓ Resolved |
| Plugin structure | GitHub repo with `.claude-plugin/`, `skills/`, `commands/`. Install via CLI. | ✓ Resolved |
| Review UI | Cowork artifact for the draft queue. Design TBD in Phase 6. | Pending — Phase 6 |
| State storage | `editorial_state.json` committed to `dimeglio.dev` repo. | ✓ Resolved |
| OpenAI key scope | Confirm `OPENAI_API_KEY` in `.env.local` covers both embeddings and DALL-E 3 images. | Open — verify |

---

## 9. Future Ideas (Post-MVP)

- **Narrative arc tracking** — Publisher notices if last 5 posts were all frontend, boosts backend/architecture projects.
- **Engagement feedback** — import LinkedIn/X metrics and adjust Rater scoring model based on what actually performs.
- **Draft editing in Cowork** — let Pablo edit a draft inline before approving, not just approve/reject.
- **Cross-project posts** — a single post connecting work done across two projects (e.g. a shared library).
- **Interview prep mode** — on demand, generate a Greatest Hits summary of the last 3 months as STAR-method interview stories.
- **Recruiter one-pager** — auto-generate a project showcase PDF from the last N published posts.
- **Direct social posting** — LinkedIn + X API integration once the copy-paste flow is validated.
- **GitHub MCP** — extend Reporter to handle remote-only repos without requiring a local clone.

---

> **How to use this document**
>
> 1. Create a new GitHub repo: `pdimeglio/blog-pipeline-plugin`
> 2. Open it in Claude Code
> 3. Hand it this design doc as the build brief
> 4. Start with Phase 1: Reporter skill
> 5. Each phase is a self-contained skill file — build, test, commit, repeat
>
> Reference: [github.com/anthropics/knowledge-work-plugins](https://github.com/anthropics/knowledge-work-plugins) for plugin structure examples
