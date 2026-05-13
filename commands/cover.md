# /blog-pipeline:cover

Regenerates the DALL-E cover image for an existing draft without touching the post itself. Use this when the cover came out wrong, when you want a different style, or after editing a post in ways that should be reflected visually.

## Usage

```
/blog-pipeline:cover <slug>
```

Or with extra prompt direction:

```
/blog-pipeline:cover <slug> "more abstract data flow imagery"
/blog-pipeline:cover <slug> "emphasize speed and motion"
/blog-pipeline:cover <slug> "warmer color temperature"
```

You can also pass a full MDX path instead of a slug.

## What it does

1. Reads the MDX frontmatter (title, description) and `##` section headings
2. Matches the post to a project via tag overlap, loads `image_theme` + `logo_path`
3. Detects whether this is an intro post (from `editorial_state.json`) for logo sizing
4. Builds the same DALL-E prompt the Writer uses (appends your extra direction if provided)
5. Calls OpenAI Images API at 1792x1024
6. Converts PNG → JPG via `sips`, composites the project logo via Pillow
7. Overwrites `public/blog/<slug>/cover.jpg`

## Safety

- Does not touch the MDX
- Does not update `editorial_state.json`
- If image generation fails (rate limit, content policy, network), the previous cover stays in place

## Cost

~$0.04 per regeneration (DALL-E 3 standard).

## Invoke the Cover Regenerator skill

@cover-regen
