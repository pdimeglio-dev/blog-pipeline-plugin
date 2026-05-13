# Blog Pipeline Plugin

Agentic blog content pipeline for [dimeglio.dev](https://dimeglio.dev). Turns local git history into MDX blog posts with DALL-E 3 cover images — ready to approve and publish.

## Setup

### 1. Load the plugin

Add to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "pablo-plugins": {
      "source": {
        "source": "directory",
        "path": "/Users/pablo/Development/blog-pipeline-plugin"
      }
    }
  },
  "enabledPlugins": {
    "blog-pipeline@pablo-plugins": true
  }
}
```

The plugin activates in every Claude Code session automatically after that.

### 2. Set the OpenAI API key

Create a `.env` file at the plugin root (gitignored):

```bash
cp .env.example .env
# then add your key:
OPENAI_API_KEY=sk-...
```

This key is used for DALL-E 3 cover image generation. Copy it from `dimeglio.dev/.env.local` if you already have it there.

### 3. Configure projects

Edit `projects.config.json` to:
- Set `dimeglio_dev_path` to your local clone of the dimeglio.dev repo
- Update `path` for each project to its actual local path

`project-batcave` ships with a placeholder path. Update it when that repo exists.

## Usage

### Run the full pipeline

```
/blog-pipeline:run
```

Selects the top 1–2 projects by score, then for each runs Reporter → Writer → Rater and drops rated-pass drafts into `dimeglio.dev/content/blog/` with `published: false`. Cover images go to `dimeglio.dev/public/blog/<slug>/cover.jpg`.

**First post for any project is always an architectural overview.** The pipeline detects `has_intro_post: false` in state and instructs the Writer to produce an intro post before any feature posts.

### Check pipeline status

```
/blog-pipeline:status
```

Shows the draft queue, last run time, and per-project posting history.

### Generate social copy for a draft

After reviewing a draft and deciding to publish it:

```
/blog-pipeline:social-manager /path/to/draft.mdx
```

Presents LinkedIn and X copy for review, revises on feedback, saves `<slug>.social.json` next to the MDX when approved.

### Regenerate the cover image

If the cover came out wrong, regenerate it without re-running the post:

```
/blog-pipeline:cover <slug>
```

Or with extra prompt direction:

```
/blog-pipeline:cover <slug> "more abstract data flow imagery"
```

Overwrites `public/blog/<slug>/cover.jpg`. Does not touch the MDX.

### Revise a draft

To re-rate a draft after editing (no side effects):

```
/blog-pipeline:rerate <slug>
```

To interactively revise a draft (Rewriter prompts you, applies edits, validates, saves on approval):

```
/blog-pipeline:rewrite <slug>
```

The Rewriter applies targeted edits to the section(s) you name, runs the same self-check the Writer uses (zero em-dashes, no banned words, no AI prose patterns), and only saves the file when you approve.

### Approve and publish a draft

After reviewing and editing the MDX:

```
/blog-pipeline:publish <slug>
```

The Deployer skill flips `published: true` (with confirmation), runs `npm run ingest`, commits with `publish: <title>`, updates `editorial_state.json`, and asks before pushing. Bails out cleanly if dimeglio.dev has unrelated uncommitted changes or ingest fails.

Manual flow (if needed):

1. Open the draft in `dimeglio.dev/content/blog/<slug>.mdx`
2. Review and edit as needed
3. Flip `published: false` → `published: true`
4. In the dimeglio.dev repo:
   ```bash
   npm run ingest
   git add content/blog/<slug>.mdx public/blog/<slug>/
   git commit -m "publish: <post title>"
   git push
   ```
5. Vercel auto-deploys within ~30 seconds

### Run skills individually (testing / manual)

```
/blog-pipeline:reporter paddle-games
/blog-pipeline:writer <reporter_json>
/blog-pipeline:rater /path/to/draft.mdx
```

## Pipeline agents

| Agent | Model | Role |
|---|---|---|
| Reporter | Haiku | Extracts git history + Claude Code session context → structured JSON |
| Writer | Sonnet | JSON → MDX post + DALL-E 3 cover image |
| Rater | Haiku | Scores draft, checks voice rules, pass/fail gate (threshold 6.5) |
| Publisher | Sonnet | Orchestrates Reporter → Writer → Rater, manages `editorial_state.json` |
| Social Manager | Sonnet | Interactive LinkedIn + X copy — revises on feedback, saves on approval |
| Rewriter | Sonnet | Interactive revision of an existing draft — applies targeted edits, runs Writer self-check |
| Cover Regenerator | Sonnet | Re-runs DALL-E cover generation + logo compositing for an existing post |
| Deployer | Sonnet | Publishes an approved draft — flip `published`, ingest, commit, update state, push on confirm |

## State

`editorial_state.json` lives in the dimeglio.dev repo root. It tracks run history, draft queue, and per-project intro/posting status. It is **not** auto-committed — commit it manually when you want to persist state across machines:

```bash
git -C ~/Development/dimeglio.dev add editorial_state.json
git -C ~/Development/dimeglio.dev commit -m "chore: update editorial state"
```

## Cost

| Item | Cost | Frequency | Monthly est. |
|---|---|---|---|
| DALL-E 3 cover image | ~$0.04/image | ~2/week | ~$0.35 |
| Anthropic API (Reporter + Rater) | ~$0.01/run | ~2/week | ~$0.10 |
| Anthropic API (Writer + Publisher + Social) | ~$0.05/run | ~2/week | ~$0.45 |
| **Total** | | | **~$0.90/month** |

Costs scale linearly with post volume. Running the pipeline more frequently doesn't change per-post cost.

## Phases shipped

- [x] Phase 1 — Reporter (git history + Claude Code session context → JSON)
- [x] Phase 2 — Writer (JSON → MDX + DALL-E 3 cover image)
- [x] Phase 3 — Rater (voice rules + rubric scoring, pass/fail gate)
- [x] Phase 4 — Publisher (orchestrator + intro mode for first posts)
- [x] Phase 5 — Social Manager (LinkedIn + X copy with review loop)
