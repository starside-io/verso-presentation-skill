# Themes

A theme is a JSON file (or inline object in `deck.json`) declaring color roles. The runtime resolves colors through a four-level cascade and emits CSS variables.

## The cascade

```
theme defaults  >  deck.style_overrides  >  slide.style_overrides  >  block.style_overrides
```

Later levels win. Each level can override any subset of color roles.

## Color roles

| Role | What it's for |
|------|---------------|
| `primary` | Brand color. Headings, accent bars, primary buttons. |
| `secondary` | Supporting brand color. Subheadings, dim text, secondary buttons. |
| `classic` | Canvas, page background. The cream, paper color. |
| `accent` | Highlights, hover states. Defaults to `secondary` if unset. |
| `surface` | Card, panel backgrounds. Defaults to a mix of `primary` + neutral. |
| `muted` | Borders, soft dividers. Defaults to a mix of `secondary` + neutral. |
| `background` | Whole-deck background. Defaults to `classic`. Set explicitly for self-contained dark modes. |
| `foreground` | Body text. Computed from `background` for contrast if unset. |

## Built-in themes

| Name | Vibe |
|------|------|
| `verso-slate` | Slate gray + steel blue. The default. |
| `verso-warm` | Warm beige + terracotta. |
| `verso-mono` | Monochrome charcoal on cream. |
| `verso-neon` | Cyberpunk dark: black background, neon accents. |
| `verso-mars` | Dark with deep red accents. |
| `verso-forest` | Cream + forest green + sage. |

List them with `verso theme list`. Activate by setting `theme` in `deck.json`.

## Authoring a custom theme

### Project-local JSON

```bash
verso theme add ./my-brand.json
```

```json
{
  "name": "my-brand",
  "colors": {
    "primary": "#1A3C6E",
    "secondary": "#4A90D9",
    "classic": "#F0EAD2",
    "accent": "#d1603d"
  }
}
```

The CLI copies the file into `themes/<name>.json` and sets `manifest.theme: "my-brand"`. Local themes override built-ins of the same name.

### Inline in the manifest

```json
{
  "title": "My Deck",
  "theme": {
    "name": "inline-brand",
    "colors": {
      "primary": "#000",
      "secondary": "#666",
      "classic": "#fff"
    }
  },
  "slide_order": [...]
}
```

The editor's Themes tab shows inline themes as read-only; switch to a project-local theme to edit through the UI.

## Per-deck, per-slide, per-block overrides

Override any role at any level without copying the full theme.

```json
// deck.json: override accent deck-wide
{
  "theme": "verso-slate",
  "style_overrides": { "colors": { "accent": "#ff0066" } }
}
```

```json
// slides/cover.json: dark background for one slide
{
  "id": "cover",
  "layout": "cover",
  "style_overrides": {
    "colors": { "background": "#0a0a0a", "foreground": "#ffffff" }
  }
}
```

```json
// inside slide content: a single card recolored
{
  "type": "card",
  "tone": "primary",
  "style_overrides": { "colors": { "primary": "#ff8800" } },
  "content": [...]
}
```

## CSS variables emitted

| Role | CSS var |
|------|---------|
| `primary` | `--color-primary` |
| `secondary` | `--color-secondary` |
| `classic` | `--color-classic` |
| `accent` | `--color-accent` |
| `surface` | `--color-surface` |
| `muted` | `--color-muted` |
| `background` | `--color-background` |
| `foreground` | `--color-foreground` |

Layouts and components reference these via `var(--color-primary)` etc. Never hard-code hex in a layout: themes won't propagate.

## Editor: the Themes tab

`verso edit` ships a Themes tab in the right pane:

- List of every theme available (built-ins + project-local).
- Click to switch.
- For project themes: clickable color swatches open a native color picker; "Edit details" expands every role.
- For built-ins: read-only. "Duplicate to project" creates an editable copy in `themes/`.
