# Blog Pipeline Plugin

Agentic blog content pipeline for [dimeglio.dev](https://dimeglio.dev). Turns local git history into MDX blog posts with DALL-E 3 cover images and LinkedIn/X copy — ready to approve and publish.

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

**To share publicly via GitHub:** swap the `source` block to `{ "source": "github", "repo": "pdimeglio/blog-pipeline-plugin" }` — the `enabledPlugins` key stays the same.

### 2. Configure env vars

The plugin reads `OPENAI_API_KEY` from `~/Development/dimeglio.dev/.env.local`. Make sure that file exists and contains:

```
OPENAI_API_KEY=sk-...
```

This key is used for:
- DALL-E 3 cover image generation (Writer)
- `npm run ingest` embeddings sync (existing dimeglio.dev script)

### 3. Configure projects

Edit `projects.config.json` at the plugin root to:
- Set `dimeglio_dev_path` to your local clone of the dimeglio.dev repo
- Update `path` for each project to its actual local path
- Add/remove projects as needed

`project-batcave` ships with a placeholder path (`~/Development/project-batcave`). Update it when that repo exists.

## Usage

### Run the full pipeline

```
/blog-pipeline:run
```

The Publisher will:
1. Score all projects and pick the top 1–2 candidates for today
2. Run Reporter → Writer → Rater → Social Manager on each
3. Drop approved drafts into `dimeglio.dev/content/blog/<slug>.mdx` with `published: false`
4. Write cover images to `dimeglio.dev/public/blog/<slug>/cover.jpg`
5. Write social copy to `dimeglio.dev/content/blog/<slug>.social.json`

### Approve and publish a draft

1. Open the draft in `dimeglio.dev/content/blog/`
2. Flip `published: false` → `published: true`
3. In the dimeglio.dev repo:
   ```bash
   npm run ingest
   git add -A && git commit -m "publish: <post title>"
   git push
   ```
4. Vercel auto-deploys within ~30 seconds

### Run the Reporter on a single project (testing)

Invoke the Reporter skill directly:

```
/reporter paddle-games
```

Inspect the JSON output. This is how to validate Phase 1 before building Phase 2.

## Pipeline Agents

| Agent | Model | Role |
|---|---|---|
| Reporter | Haiku | Extracts git history → structured JSON |
| Writer | Sonnet | Converts JSON → MDX post + DALL-E 3 cover image |
| Rater | Haiku | Scores drafts, checks voice rules, pass/fail gate |
| Publisher | Sonnet | Orchestrates the full pipeline, manages `editorial_state.json` |
| Social Manager | Sonnet | Produces LinkedIn + X copy-paste text per approved draft |

## State

`editorial_state.json` lives in the dimeglio.dev repo root. It tracks posting history and the draft queue. It is **not** auto-committed — commit it manually when you want to persist state across machines:

```bash
git -C ~/Development/dimeglio.dev add editorial_state.json
git -C ~/Development/dimeglio.dev commit -m "chore: update editorial state"
```

## Build phases

- [x] Phase 1 — Foundation + Reporter
- [ ] Phase 2 — Writer + Cover Image
- [ ] Phase 3 — Rater
- [ ] Phase 4 — Publisher
- [ ] Phase 5 — Social Manager
- [ ] Phase 6 — Polish + Ops
