# /blog-pipeline:status

Shows the current state of the pipeline: last run, draft queue, and per-project posting history.

## What it shows

- Last pipeline run time
- Per-project: last posted date, posts this month, whether the intro post has been produced
- Draft queue: all drafts with status (pending_review, rejected), Rater score, and creation date
- Any projects hitting their monthly cap or cooldown period

## How to use

Run this before `/blog-pipeline:run` to see what's queued and which projects are eligible today.

## Implementation

Read `$PLUGIN_ROOT/editorial_state.json` (same root as `projects.config.json`). If the file does not exist, print: "No pipeline runs yet. Run `/blog-pipeline:run` to start."

Format the output as a readable summary:

```
Blog Pipeline Status — <date>

Last run: <timestamp or "never">

Projects:
  paddle-games     last posted: never  |  this month: 0/4  |  intro: pending
  dimeglio-dev     last posted: never  |  this month: 0/3  |  intro: pending

Draft queue (N drafts):
  ✓ 2026-05-12-strava-webhooks-were-eating-activities  (8.4)  pending_review
  ✗ 2026-05-12-some-other-post                          (5.1)  rejected — voice flags: robust

Next eligible run: <project> in <N days> / available now
```

@publisher
