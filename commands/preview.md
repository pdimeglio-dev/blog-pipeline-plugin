# /blog-pipeline:preview

Opens a draft in the browser on the local dimeglio.dev dev server, without flipping `published`. Use it after `/blog-pipeline:run` or `/blog-pipeline:rewrite` to read the post the way a reader will see it.

## Usage

```
/blog-pipeline:preview <slug>
```

## What it does

1. Resolves the MDX at `${dimeglio_dev_path}/content/blog/<slug>.mdx` (from `projects.config.json`). Stops if it doesn't exist.
2. Checks for a dev server on port 3000: `lsof -iTCP:3000 -sTCP:LISTEN -n -P`. If none is running, starts `npm run dev` in dimeglio.dev **in the background** and waits until it prints `Ready` (up to about 60 seconds).
3. Requests `http://localhost:3000/blog/<slug>` and checks for a 200. Drafts render under `next dev` with a "Draft preview" banner (dimeglio.dev #64). If it returns 404, the site doesn't have draft preview yet: say so, and do **not** flip `published` as a workaround unless Pablo asks.
4. Opens the URL in the default browser with `open <url>` (YouTube embeds often won't play in VS Code's built-in browser, so use the system one).
5. Leaves the dev server running and prints the URL, plus how to stop it if this command started it.

## Safety

- Never edits the MDX. In particular, never changes `published`.
- Never commits anything.
