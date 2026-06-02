# /blog-pipeline:run

Runs the full blog pipeline: selects projects, generates drafts, scores them, and updates editorial state.

## Usage

```
/blog-pipeline:run                                  # automatic — scores all projects
/blog-pipeline:run <project_id>                     # force this project
/blog-pipeline:run <project_id> "<topic>"           # force project AND bias to topic
/blog-pipeline:run "<topic>"                        # automatic project, biased to topic
/blog-pipeline:run series <series-id> [next]        # draft the next planned part of a series
```

## Examples

```
/blog-pipeline:run
/blog-pipeline:run paddle-games
/blog-pipeline:run paddle-games "outbox shadow mode rollout"
/blog-pipeline:run "the new webhook idempotency fix"
/blog-pipeline:run series batcave-build             # draft the next unwritten part
```

## What it does

1. Parses optional arguments — forced project, topic hint, or both
2. Bootstraps `editorial_state.json` if missing
3. Scores all configured projects (unless forced) and selects the top 1–2
4. For each selected project: runs Reporter → Writer → Rater
5. Drops rated-pass drafts into `dimeglio.dev/content/blog/` with `published: false`
6. Updates `dimeglio.dev/editorial_state.json` with run history and draft queue
7. Prints a summary of what was produced, rejected, or skipped

First post for any project is automatically an architectural overview — the pipeline detects whether a project has had its intro post yet. The topic hint still applies in intro mode.

When a project is forced, the cooldown (`min_days_between_posts`) and monthly cap are bypassed. Same-day idempotency still applies — you can't draft the same project twice in one day.

**Series mode** (`series <series-id>`) reads `series.config.json`, picks the next part whose status is `planned`, and forces that part's project. It bypasses the cooldown and monthly cap so a multi-part arc isn't blocked, passes the part's brief and `kind` to the Writer as a `series_info` block, and records progress in both `series.config.json` (the plan) and `editorial_state.json` (the ledger). It only links back to a previous part once that part is actually published, so publish each part before drafting the next.

## After running

Review the draft, then:

```
/blog-pipeline:rerate <slug>     # re-score after editing
/blog-pipeline:rewrite <slug>    # interactive revision
/blog-pipeline:cover <slug>      # regenerate cover image
/blog-pipeline:publish <slug>    # ingest, commit, optionally push
```

## Invoke the Publisher skill

@publisher
