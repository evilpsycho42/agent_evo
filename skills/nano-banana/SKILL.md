---
name: nano-banana
description: Image generation and editing CLI using Google Gemini image models. Use when the user needs to create images from text prompts, edit or modify existing images, change image styles, or generate visual content. Triggers include requests to "generate an image", "create a picture", "draw something", "edit this image", "modify this photo", "make it look like", "change the style", "add/remove elements from image", or any task requiring AI image generation or manipulation. Supports multiple resolutions (1K/2K/4K), aspect ratios, and image-to-image editing via --input-image.
allowed-tools: Bash(nano-banana:*)
---

# Nano Banana Pro Image Generation & Editing

Generate new images or edit existing ones using Google's Nano Banana Pro API (Gemini 3 Pro Image).

## Usage

Run the script using relative path (from the skill directory):

**Generate new image:**
```bash
scripts/nano-banana --prompt "your image description" --filename "output-name.png" [--resolution 1K|2K|4K] [--aspect RATIO]
```

**Edit existing image:**
```bash
scripts/nano-banana --prompt "editing instructions" --filename "output-name.png" --input-image "path/to/input.png" [--resolution 1K|2K|4K] [--aspect RATIO]
```

**Important:** Always run from the user's current working directory so images are saved where the user is working, not in the skill directory.

## Default Workflow (draft → iterate → final)

Goal: fast iteration without burning time on 4K until the prompt is correct.

- Draft (1K): quick feedback loop
  - `scripts/nano-banana --prompt "<draft prompt>" --filename "yyyy-mm-dd-hh-mm-ss-draft.png" --resolution 1K`
- Iterate: adjust prompt in small diffs; keep filename new per run
  - If editing: keep the same `--input-image` for every iteration until you’re happy.
- Final (4K): only when prompt is locked
  - `scripts/nano-banana --prompt "<final prompt>" --filename "yyyy-mm-dd-hh-mm-ss-final.png" --resolution 4K`

## Resolution Options

The script supports three resolutions (uppercase K required):

- **1K** (default) - ~1024px resolution
- **2K** - ~2048px resolution
- **4K** - ~4096px resolution

Map user requests to API parameters:
- No mention of resolution → `1K`
- "low resolution", "1080", "1080p", "1K" → `1K`
- "2K", "2048", "normal", "medium resolution" → `2K`
- "high resolution", "high-res", "hi-res", "4K", "ultra" → `4K`

**Model note:** The default model (`gemini-2.5-flash-image-preview`) may return ~1K output even if you request `2K`/`4K`. For true `2K`/`4K`, use `--model gemini-3-pro-image-preview`.

## Aspect Ratio Options

Use `--aspect` to control the output image aspect ratio:

- `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`

## Environment Variables

The script reads configuration from environment variables (typically set in `~/.zshrc` or `~/.zshenv`):

- **`GEMINI_API_KEY`** (required) — Your Gemini API key. The script will exit with an error if this is not set.
- **`GEMINI_BASE_URL`** (optional) — Custom API base URL. If not set, the script uses Google's official Gemini endpoint. Set this if you use a proxy or gateway.

## Preflight + Common Failures (fast fixes)

- Preflight:
  - `command -v uv` (required — the script runs via `uv` shebang)
  - `test -n \"$GEMINI_API_KEY\"` (must be set in environment)
  - If editing: `test -f \"path/to/input.png\"`

- Common failures:
  - `Error: GEMINI_API_KEY environment variable is not set.` → add `export GEMINI_API_KEY="..."` to `~/.zshrc`
  - `Error loading input image:` → wrong path / unreadable file; verify `--input-image` points to a real image
  - “quota/permission/403” style API errors → wrong key, no access, or quota exceeded; try a different key/account

## Filename Generation

Generate filenames with the pattern: `yyyy-mm-dd-hh-mm-ss-name.png`

**Format:** `{timestamp}-{descriptive-name}.png`
- Timestamp: Current date/time in format `yyyy-mm-dd-hh-mm-ss` (24-hour format)
- Name: Descriptive lowercase text with hyphens
- Keep the descriptive part concise (1-5 words typically)
- Use context from user's prompt or conversation
- If unclear, use random identifier (e.g., `x9k2`, `a7b3`)

Examples:
- Prompt "A serene Japanese garden" → `2025-11-23-14-23-05-japanese-garden.png`
- Prompt "sunset over mountains" → `2025-11-23-15-30-12-sunset-mountains.png`
- Prompt "create an image of a robot" → `2025-11-23-16-45-33-robot.png`
- Unclear context → `2025-11-23-17-12-48-x9k2.png`

## Image Editing

When the user wants to modify an existing image:
1. Check if they provide an image path or reference an image in the current directory
2. Use `--input-image` parameter with the path to the image
3. The prompt should contain editing instructions (e.g., "make the sky more dramatic", "remove the person", "change to cartoon style")
4. Common editing tasks: add/remove elements, change style, adjust colors, blur background, etc.

## Prompt Handling

**For generation:** Pass user's image description as-is to `--prompt`. Only rework if clearly insufficient.

**For editing:** Pass editing instructions in `--prompt` (e.g., "add a rainbow in the sky", "make it look like a watercolor painting")

Preserve user's creative intent in both cases.

## Prompt Templates (high hit-rate)

Use templates when the user is vague or when edits must be precise.

- Generation template:
  - “Create an image of: <subject>. Style: <style>. Composition: <camera/shot>. Lighting: <lighting>. Background: <background>. Color palette: <palette>. Avoid: <list>.”

- Editing template (preserve everything else):
  - “Change ONLY: <single change>. Keep identical: subject, composition/crop, pose, lighting, color palette, background, text, and overall style. Do not add new objects. If text exists, keep it unchanged.”

## Output

- Saves PNG to current directory (or specified path if filename includes directory)
- Script outputs the full path to the generated image
- **Do not read the image back** - just inform the user of the saved path

## Examples

**Generate new image:**
```bash
scripts/nano-banana --prompt "A serene Japanese garden with cherry blossoms" --filename "2025-11-23-14-23-05-japanese-garden.png" --resolution 4K
```

**Edit existing image:**
```bash
scripts/nano-banana --prompt "make the sky more dramatic with storm clouds" --filename "2025-11-23-14-25-30-dramatic-sky.png" --input-image "original-photo.jpg" --resolution 2K
```
