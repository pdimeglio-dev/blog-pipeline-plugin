# /blog-pipeline:rerate

Re-runs the Rater on an existing draft. Useful after hand-editing a draft or after the Rater rules have been updated.

## Usage

```
/blog-pipeline:rerate <slug>
```

Or pass a full MDX path:

```
/blog-pipeline:rerate /Users/pablo/Development/dimeglio.dev/content/blog/<slug>.mdx
```

## What it does

Returns the same JSON the Rater produces during a normal pipeline run: composite score, per-axis scores, voice_flags, notes. Pass threshold is 6.5 composite; any non-empty voice_flags array means automatic fail.

No side effects — does not modify the MDX, does not touch editorial_state.json. Use `/blog-pipeline:rewrite <slug>` to act on the feedback.

## Invoke the Rater skill

@rater
