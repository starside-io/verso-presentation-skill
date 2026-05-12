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

## Containers (recursive)

Containers wrap other blocks. Their `content` array follows the same shape as `slide.content`.

### card

```json
{
  "type": "card",
  "tone": "surface",
  "variant": "soft",
  "padding": "md",
  "content": [
    { "type": "heading", "level": 3, "text": "Card title" },
    { "type": "text", "text": "Card body." }
  ]
}
```

| Prop | Values | Default |
|------|--------|---------|
| `tone` | `primary` / `secondary` / `accent` / `muted` / `surface` | `surface` |
| `variant` | `soft` / `solid` / `outline` | `soft` |
| `padding` | `none` / `sm` / `md` / `lg` | `md` |

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
| List of points | `bullets` |
| Pull quote, customer testimonial | `quote` |
| Image, optionally with caption | `image` |
| Code snippet | `code` |
| Video, iframe, Figma | `embed` |
| Grouped info with a colored background | `card` |
| Full-width attention-grabbing band | `panel` with `bleed: all` |
| Warning, tip, success message | `callout` |
| Small inline tag | `badge` |
| Visual divider above a heading | `accent-bar` |
| Horizontal rule between sections | `divider` |
