# /blog-pipeline:run

Runs the full blog pipeline: selects projects, generates drafts, scores them, and updates editorial state.

## What it does

1. Scores all configured projects using priority, recency, and monthly cap
2. Selects the top 1–2 projects for this run
3. For each: runs Reporter → Writer → Rater in sequence
4. Drops rated-pass drafts into `dimeglio.dev/content/blog/` with `published: false`
5. Updates `dimeglio.dev/editorial_state.json` with run history and draft queue
6. Prints a summary of what was produced, rejected, or skipped

First post for any project is automatically an architectural overview — the pipeline detects whether a project has had its intro post yet.

## After running

1. Open the draft MDX in `dimeglio.dev/content/blog/<slug>.mdx`
2. Review and edit as needed
3. Flip `published: false` → `published: true`
4. Run `npm run ingest` in the dimeglio.dev directory
5. `git commit && git push` to trigger the Vercel deploy

## Invoke the Publisher skill

@publisher
