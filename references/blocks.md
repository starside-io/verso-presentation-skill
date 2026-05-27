# Blocks

Every entry in a slide's `content` array is a block. Each block has a `type` field plus type-specific props. Unknown types render as empty placeholders, so spell them correctly.

## Common optional fields (all blocks)

| Field | Type | Notes |
|-------|------|-------|
| `id` | `string` | Stable id. Optional. |
| `path_include` | `string[]` | Only render in these paths. |
| `path_exclude` | `string[]` | Skip in these paths. |
| `style_overrides` | object | Block-level color overrides. See themes.md. |
| `align` | `{ horizontal, vertical }` | Per-block alignment. Most layouts ignore it; cards use it. |

### `align` values

The `align` object accepts `horizontal` and `vertical` keys:

| Key | Valid values | Default |
|-----|-------------|---------|
| `horizontal` | `left`, `center`, `right` | `left` |
| `vertical` | `top`, `middle`, `bottom` | `top` |

Example:
```json
{ "align": { "horizontal": "center", "vertical": "middle" } }
```

**Do not use `center` for vertical alignment.** The correct vertical value is `middle`.

### Per-zone slide alignment

Slides can optionally split alignment between the title zone (header + title) and the content zone (the blocks). Useful when you want the title pinned to the top but the body centered vertically (the PowerPoint-style behavior).

```json
{
  "id": "intro",
  "layout": "content",
  "align": {
    "title":   { "horizontal": "left",   "vertical": "top"    },
    "content": { "horizontal": "center", "vertical": "middle" }
  }
}
```

The flat form (`{ horizontal, vertical }`) still applies to both zones; nested form wins per zone. The editor's Align dropdown shows Title and Content sections so users can set each independently. Collapses back to the flat form automatically when both zones match.

## Text blocks

### heading

```json
{ "type": "heading", "level": 2, "text": "Section title" }
```

| Prop | Default | Notes |
|------|---------|-------|
| `level` | `2` | `1`, `2`, or `3`. Renders as `<h1>`, `<h2>`, `<h3>`. |
| `text` | required | The heading text. |

### text

```json
{ "type": "text", "text": "Plain paragraph." }
```

### bullets

```json
{ "type": "bullets", "items": ["First", "Second", "Third"] }
```

Each item is either a plain string OR an object with optional per-item Phosphor icon:

```json
{
  "type": "bullets",
  "items": [
    { "text": "Free-form content",       "icon": "stack",         "iconTone": "primary"   },
    { "text": "Mix bullets, text, code", "icon": "squares-four",  "iconTone": "secondary" },
    { "text": "No image required",       "icon": "image",         "iconTone": "accent"    },
    "Plain string items still work too"
  ]
}
```

| Item field | Notes |
|------------|-------|
| `text` | Required when using the object form. The bullet text. |
| `icon` | Phosphor icon id (kebab-case). Browse at <https://phosphoricons.com>. |
| `iconWeight` | `thin` / `light` / `regular` / `bold` / `fill` / `duotone`. Default `regular`. |
| `iconTone` | Color tone: `primary` / `secondary` / `accent` / `muted` / `surface`. Default `primary`. |

In the editor, every bullet row has a small square icon-picker button next to its text input.

### quote

```json
{
  "type": "quote",
  "text": "A short, punchy quote.",
  "attribution": "Author Name"
}
```

`attribution` is optional.

## Media blocks

### image

```json
{
  "type": "image",
  "src": "assets/hero.png",
  "alt": "Hero illustration",
  "caption": "Optional caption"
}
```

| Prop | Default | Notes |
|------|---------|-------|
| `src` | required | URL or project-relative path. Editor uploads land in `assets/`. |
| `alt` | `""` | Accessibility text. |
| `caption` | undefined | Rendered as `<figcaption>` below the image. |

### code

```json
{
  "type": "code",
  "language": "ts",
  "source": "const verso = 'hello'"
}
```

Built-in components don't ship a syntax highlighter; drop in Shiki or Prism via a custom component in `verso.config.ts` if needed.

### embed

Iframe (YouTube, CodeSandbox, Figma) with a PDF-safe fallback.

```json
{
  "type": "embed",
  "src": "https://www.youtube.com/embed/dQw4w9WgXcQ",
  "title": "Demo video",
  "aspect": "16:9",
  "fallback_text": "Open the deck online to play this video."
}
```

### icon

A standalone Phosphor icon block. Composes inside any layout or container.

```json
{
  "type": "icon",
  "name": "lightning",
  "weight": "fill",
  "size": 48,
  "tone": "accent",
  "label": "High priority"
}
```

| Prop | Default | Notes |
|------|---------|-------|
| `name` | required | Phosphor icon id, kebab-case (e.g. `lightning`, `check-circle`, `github-logo`). Browse the catalog at <https://phosphoricons.com>. |
| `weight` | `regular` | One of `thin`, `light`, `regular`, `bold`, `fill`, `duotone`. |
| `size` | `32` | Render size in px. |
| `tone` | `primary` | Color tone: `primary`, `secondary`, `accent`, `muted`, `surface`. Picks up from the theme cascade. |
| `label` | undefined | Optional accessibility label. Omitted icons are marked decorative. |

SVGs are lazy-loaded and inlined; PDF and HTML exports get the real SVG (no missing-icon placeholders in print). The editor's Decoration block menu has an "Icon" entry that opens a searchable picker over the full 1,512-icon catalog with weight selection.

## Containers (recursive)

Containers wrap other blocks. Their `content` array follows the same shape as `slide.content`.

### card

```json
{
  "type": "card",
  "tone": "primary",
  "variant": "soft",
  "padding": "md",
  "header": "Key insight",
  "icon": "lightbulb",
  "iconWeight": "fill",
  "iconTone": "accent",
  "content": [
    { "type": "text", "text": "Card body. Header + icon strip above is optional." }
  ]
}
```

| Prop | Values | Default |
|------|--------|---------|
| `tone` | `primary` / `secondary` / `accent` / `muted` / `surface` | `surface` |
| `variant` | `soft` / `solid` / `outline` | `soft` |
| `padding` | `none` / `sm` / `md` / `lg` | `md` |
| `header` | string | undefined. When set, renders as a bold strip above `content`. |
| `icon` | Phosphor icon id | undefined. Renders next to `header` (or above content alone if no header). |
| `iconWeight` | `thin` / `light` / `regular` / `bold` / `fill` / `duotone` | `regular` |
| `iconTone` | Same tones as the card | falls back to the card's `tone` |

Side-by-side cards in `two-col` / `three-col` layouts auto-stretch to equal height ONLY when every column contains exactly card blocks. A card paired with non-card content (bullets, text) keeps its natural height so it doesn't stretch into empty space.

### panel

Wider than card. Can `bleed` past the slide's content area.

```json
{
  "type": "panel",
  "tone": "primary",
  "bleed": "all",
  "content": [
    { "type": "text", "text": "Panel content extends edge-to-edge." }
  ]
}
```

| Prop | Values | Default |
|------|--------|---------|
| `tone`, `variant` | Same as `card` | |
| `bleed` | `left` / `right` / `top` / `bottom` / `all` / `none` | `none` |

## Decoration blocks

Small visual elements. Have a `tone` for color but no `content`.

### callout

```json
{ "type": "callout", "tone": "info", "text": "Heads up." }
```

`tone`: `info` / `warn` / `success` / `danger`. Default: `info`.

### badge

```json
{ "type": "badge", "tone": "accent", "variant": "soft", "text": "New" }
```

Same `tone` and `variant` as cards. Renders inline-ish.

### accent-bar

```json
{ "type": "accent-bar", "tone": "accent", "size": "thick", "orientation": "horizontal" }
```

| Prop | Values |
|------|--------|
| `tone` | Same as cards. |
| `size` | `thin` / `thick`. |
| `orientation` | `horizontal` / `vertical`. |

### divider

```json
{ "type": "divider", "tone": "muted" }
```

A thin horizontal rule.

## Variable interpolation

Any text field can reference deck-level variables as `{{key}}`:

```json
{ "type": "text", "text": "Prepared by {{author}} on {{date}}." }
```

See [features.md](features.md) for the full list of built-in keys and how to declare custom ones.

## Quick reference: when to reach for each block

| Need | Block |
|------|-------|
| Section title | `heading` |
| Body text | `text` |
| List of points | `bullets` (add per-item `icon` for visual rhythm) |
| Pull quote, customer testimonial | `quote` |
| Image, optionally with caption | `image` |
| Code snippet | `code` |
| Video, iframe, Figma | `embed` |
| Symbol / glyph as decoration | `icon` (Phosphor catalog) |
| Grouped info with a colored background | `card` (set `header` + `icon` for a labeled tile) |
| Full-width attention-grabbing band | `panel` with `bleed: all` |
| Warning, tip, success message | `callout` |
| Small inline tag | `badge` |
| Visual divider above a heading | `accent-bar` |
| Horizontal rule between sections | `divider` |
