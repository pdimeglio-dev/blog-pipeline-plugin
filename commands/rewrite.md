# /blog-pipeline:rewrite

Interactively revises an existing draft without re-running the full pipeline. Useful after Rater feedback, or when you want to tighten a section, change the tone, or restructure without regenerating from scratch.

## Usage

```
/blog-pipeline:rewrite <slug>
```

Or pass a full MDX path:

```
/blog-pipeline:rewrite /Users/pablo/Development/dimeglio.dev/content/blog/<slug>.mdx
```

## How it works

1. The Rewriter reads the draft and shows you the title, slug, word count, and section list.
2. You describe what to change — be specific: section name, phrasing, tone, length, structure.
3. The Rewriter applies the edit and runs the Writer's self-check (zero em-dashes, no banned words, no AI patterns).
4. You see the before/after for small edits, or the new section in full for larger ones.
5. Loop until you say "save it". The file is only written on approval.

## What it won't do

- Won't flip `published: true` — that's `/blog-pipeline:publish`'s job
- Won't touch the frontmatter unless you explicitly ask
- Won't update the cover image — that's `/blog-pipeline:cover`'s job
- Won't rewrite the whole post from scratch — that's what re-running the pipeline does

## Follow-up

After saving:

```
/blog-pipeline:rerate <slug>   # verify the revision passes
/blog-pipeline:publish <slug>  # publish when ready
```

## Invoke the Rewriter skill

@rewriter
