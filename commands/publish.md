# /blog-pipeline:publish

Publishes an approved draft to dimeglio.dev. Replaces the 4-step manual publish flow with one command.

## What it does

1. Flips `published: false` → `published: true` (with confirmation)
2. Validates the dimeglio.dev working tree is clean of unrelated changes
2b. Switches to a `publish/<slug>` branch (never main), warns if the post is stacked on another unmerged branch, and checks that every component the post uses exists on `origin/main`
3. Runs `npm run ingest` to regenerate the slugs index and embeddings (this updates the Pinecone index right away, before the PR merges)
4. Stages and commits the MDX + cover dir + ingest output as `publish: <title>`
5. Updates `editorial_state.json` (last_posted, posts_this_month, removes from draft_queue, marks intro complete if applicable)
6. Generates LinkedIn, X and Instagram social copy via the Social Manager (interactive review loop), commits as `social: <title>`
7. Asks before pushing the branch and opening a PR. Never pushes to main, never merges

## Usage

```
/blog-pipeline:publish <slug>
```

Slug example: `2026-05-11-strava-webhooks-were-eating-activities`

Or pass a full MDX path:

```
/blog-pipeline:publish /Users/pablo/Development/dimeglio.dev/content/blog/<slug>.mdx
```

## Safety

- Bails out cleanly if dimeglio.dev has unrelated uncommitted changes
- Bails out cleanly if `npm run ingest` fails
- Asks before flipping `published: true`
- Asks before pushing; publishes through a PR, never directly to main
- Editorial state is updated but **not** auto-committed (you commit state manually when you want it persisted)

## Invoke the Deployer skill

@deployer
