---
name: cover-regen
description: Regenerates the DALL-E cover image for an existing blog draft without re-running the post. Reads the MDX frontmatter and headings, builds the same prompt the Writer uses, calls OpenAI Images API, converts to JPG, composites the project logo, and overwrites the existing cover.jpg.
model: sonnet
allowed-tools:
  - Read
  - Bash(curl *)
  - Bash(jq *)
  - Bash(sips *)
  - Bash(mkdir *)
  - Bash(rm *)
  - Bash(python3 *)
  - Bash(grep *)
  - Bash(sed *)
  - Bash(head *)
  - Bash(tr *)
  - Bash(cut *)
  - Bash(test *)
argument-hint: "<slug or mdx_path> [extra prompt text]"
user-invocable: true
---

You are the Cover Regenerator. You re-run cover image generation for an existing post without touching the MDX or running the rest of the pipeline. Use this when a cover came out wrong, when you want a different style, or after editing a post in ways that should be reflected visually.

## Inputs

The first argument is either a slug (`2026-05-12-behind-the-paddle-games`) or a full MDX path.

The second argument (optional) is **extra prompt text** that gets appended to the base DALL-E prompt. Use it to bias the new image. Examples:
- `"more abstract data flow imagery"`
- `"emphasize speed and motion"`
- `"warmer color temperature"`

## Steps

### 1. Load environment

```bash
PLUGIN_ROOT="${CLAUDE_PLUGIN_ROOT:-/Users/pablo/Development/blog-pipeline-plugin}"
test -f "$PLUGIN_ROOT/.env" || { echo "missing .env at $PLUGIN_ROOT/.env"; exit 1; }
OPENAI_API_KEY=$(grep '^OPENAI_API_KEY=' "$PLUGIN_ROOT/.env" | cut -d= -f2-)
test -n "$OPENAI_API_KEY" || { echo "OPENAI_API_KEY empty in .env"; exit 1; }

DIMEGLIO_DEV_PATH=$(jq -r '.dimeglio_dev_path' "$PLUGIN_ROOT/projects.config.json" | sed "s|^~|$HOME|")
```

### 2. Resolve the MDX path

```bash
if [[ "$ARG1" = /* ]]; then
  MDX_PATH="$ARG1"
  SLUG=$(basename "$MDX_PATH" .mdx)
else
  SLUG="$ARG1"
  MDX_PATH="${DIMEGLIO_DEV_PATH}/content/blog/${SLUG}.mdx"
fi

test -f "$MDX_PATH" || { echo "draft_not_found: $MDX_PATH"; exit 1; }
```

### 3. Extract title, description, tags, and headings from the MDX

```bash
TITLE=$(sed -n 's/^title: *"\(.*\)"$/\1/p' "$MDX_PATH" | head -1)
DESCRIPTION=$(sed -n 's/^description: *"\(.*\)"$/\1/p' "$MDX_PATH" | head -1)

# Headings — the post's key topics
HEADINGS=$(grep '^## ' "$MDX_PATH" \
  | sed 's/^## //' \
  | head -6 \
  | tr '\n' ';' \
  | sed 's/;$//' \
  | sed 's/;/; /g')
```

### 4. Match the post to a project and load image_theme

Read `$PLUGIN_ROOT/projects.config.json`. Read the post's `tags` from frontmatter:

```bash
TAGS=$(sed -n 's/^tags: *\[\(.*\)\]$/\1/p' "$MDX_PATH" | head -1)
```

Find the project whose `skills_showcased` has the most overlap with `tags` (case-insensitive). If no overlap, fall back to checking which project's `liveUrl` domain appears in the post body. If still no match, stop with `{ "error": "project_match_failed", "slug": "<slug>" }`.

Extract from the matched project's `image_theme`:
- `background` (always present)
- `accent_primary` (always present)
- `accent_secondary` (may be `null`)
- `accent_tertiary` (may be absent)
- `style_keywords`
- `mood`
- `logo_path` (may be absent)

Also note whether the post is an intro post — check `editorial_state.json`:

```bash
STATE_FILE="${DIMEGLIO_DEV_PATH}/editorial_state.json"
IS_INTRO=0
if [ -f "$STATE_FILE" ]; then
  WAS_INTRO=$(jq -r --arg slug "$SLUG" '
    .projects[]?.draft_queue[]?
    | select(.slug == $slug)
    | .intro_mode
  ' "$STATE_FILE" | head -1)
  [ "$WAS_INTRO" = "true" ] && IS_INTRO=1
fi
```

If the post isn't in `draft_queue` (already published), the logo will use the regular-mode placement (smaller, bottom-right). That's fine — published posts shouldn't need the bigger intro-mode logo.

### 5. Build the DALL-E prompt

Use the same template as Writer Step 8:

```
A modern, cinematic blog cover image for a tech article titled "{TITLE}".
The article is about: {DESCRIPTION}
Key topics: {HEADINGS}
Background: true black ({background}). Primary accent: {accent_primary}. Secondary: {accent_secondary}.[Tertiary: {accent_tertiary}.]
Style: {style_keywords}. Mood: {mood}.
No faces, no logos. No text or labels — the only exception is if the main topic of the article is a named technical pattern (e.g. "Outbox Pattern", "Event Sourcing"), in which case that single pattern name may appear as a subtle label on the relevant visual element. Abstract geometric shapes and light effects otherwise.
Aspect ratio: 16:9. Dark, premium, editorial quality.
```

Rules:
- If `accent_secondary` is `null`, drop the "Secondary:" line.
- If `accent_tertiary` is present, replace `[Tertiary: {accent_tertiary}.]` with `Tertiary: {accent_tertiary}.`; if absent, omit that line entirely.

If a second argument (`ARG2`) was passed, append it to the prompt as an additional line:

```
{base prompt}

Additional direction: {ARG2}
```

### 6. Generate the image

```bash
COVER_DIR="${DIMEGLIO_DEV_PATH}/public/blog/${SLUG}"
mkdir -p "$COVER_DIR"

# Build the JSON payload with jq first — never interpolate $PROMPT directly into a JSON string.
# DALL-E 3 model name: "dall-e-3" (exact). Valid sizes: 1024x1024 | 1024x1792 | 1792x1024 (use this).
PAYLOAD=$(jq -n \
  --arg model "dall-e-3" \
  --arg prompt "$PROMPT" \
  --arg size "1792x1024" \
  '{model: $model, prompt: $prompt, size: $size, quality: "standard", n: 1}')

# Retry up to 3 times — transient API errors can cause a false "model does not exist" response.
IMAGE_URL=""
for ATTEMPT in 1 2 3; do
  RESPONSE=$(curl -sS https://api.openai.com/v1/images/generations \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$PAYLOAD")
  IMAGE_URL=$(echo "$RESPONSE" | jq -r '.data[0].url // empty')
  [ -n "$IMAGE_URL" ] && break
  echo "image gen attempt $ATTEMPT failed: $RESPONSE" >&2
  sleep 3
done
test -n "$IMAGE_URL" || { echo "image gen failed after 3 attempts: $RESPONSE"; exit 1; }

curl -sS "$IMAGE_URL" -o "${COVER_DIR}/cover.png"
sips -s format jpeg "${COVER_DIR}/cover.png" --out "${COVER_DIR}/cover.jpg" >/dev/null
rm "${COVER_DIR}/cover.png"
```

If the curl or jq call fails (rate limit, content policy, network), return `{ "status": "failed", "reason": "<error>" }` and stop. **Do not** delete the existing cover.jpg — leave the previous one in place.

### 7. Composite the project logo (if `logo_path` is set)

Same logic as Writer Step 8 — keeping in sync intentionally:

```bash
COVER_JPG="${COVER_DIR}/cover.jpg"

if [ -n "$LOGO_PATH_RAW" ]; then
  LOGO_PATH_EXPANDED=$(echo "$LOGO_PATH_RAW" | sed "s|~|$HOME|g")
  if [ -f "$LOGO_PATH_EXPANDED" ]; then
    python3 -c "
from PIL import Image, ImageFilter

cover = Image.open('$COVER_JPG').convert('RGBA')
logo = Image.open('$LOGO_PATH_EXPANDED').convert('RGBA')

if $IS_INTRO:
    scale = 0.15
    pad = 56
else:
    scale = 0.09
    pad = 48

logo_w = int(cover.width * scale)
logo_h = int(logo.height * (logo_w / logo.width))
logo = logo.resize((logo_w, logo_h), Image.LANCZOS)
x = cover.width - logo_w - pad
y = cover.height - logo_h - pad

shadow_canvas = Image.new('RGBA', (logo_w + 80, logo_h + 80), (0, 0, 0, 0))
shadow_canvas.paste((0, 0, 0, 180), (40, 40), logo.split()[3])
shadow_canvas = shadow_canvas.filter(ImageFilter.GaussianBlur(radius=18))
cover.alpha_composite(shadow_canvas, (x - 40 + 6, y - 40 + 6))

cover.paste(logo, (x, y), logo)
cover.convert('RGB').save('$COVER_JPG', quality=95)
print('Logo composited (intro=$IS_INTRO)')
"
  fi
fi
```

If compositing fails (Pillow missing, logo unreadable), the DALL-E cover is still usable. Surface the error but keep the file.

### 8. Return JSON

Emit **only** this JSON — no preamble:

```json
{
  "status": "regenerated",
  "slug": "...",
  "cover_path": "/full/path/to/public/blog/<slug>/cover.jpg",
  "is_intro": false,
  "logo_composited": true,
  "extra_prompt": "more abstract data flow imagery",
  "errors": []
}
```

If image generation failed:

```json
{
  "status": "failed",
  "slug": "...",
  "reason": "image_gen_failed: <error>",
  "previous_cover_preserved": true
}
```

If project match failed:

```json
{ "status": "failed", "slug": "...", "reason": "project_match_failed" }
```
