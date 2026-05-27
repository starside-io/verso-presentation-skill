# Features

Mid-tier features: variables, transitions, watermark, embed, speaker mode. Most decks don't need these. Pull this file in when the user asks about one of them.

## Variables

Any text field in a slide can reference deck-level variables as `{{key}}`. They resolve at render time.

### Built-in keys

| Key | Value |
|-----|-------|
| `{{date}}` | Today's date, `YYYY-MM-DD`. |
| `{{time}}` | Current time, `HH:MM`. |
| `{{deckTitle}}` | `manifest.title`. |
| `{{pathId}}` | Active path. |

### Custom keys

Declared in `manifest.variables`:

```json
{
  "title": "Q4 readout",
  "variables": {
    "author": "Mederic Burlet",
    "client": "Acme Corp",
    "version": "v1.2.0"
  }
}
```

In a slide:

```json
{ "type": "text", "text": "Prepared by {{author}} for {{client}} ({{version}})" }
```

Custom keys override built-ins on collision. Unknown keys pass through verbatim (no error). Substitution covers every text field, including slide title, header, notes, and HTML attribute values like `alt`. It also covers text inside `code` blocks, so escape with a zero-width space if you need literal `{{` in code samples.

## Transitions

Each slide can declare a `transition` string. When that slide becomes active in present or speaker mode, a keyframe animation plays.

```json
{ "id": "intro", "layout": "cover", "transition": "fade", "content": [...] }
```

### Built-ins (16)

| Group | Names |
|-------|-------|
| Off | `none` (default) |
| Move | `fade`, `slide-left`, `slide-right`, `slide-up`, `slide-down` |
| Scale | `zoom`, `zoom-out`, `pop` |
| 3D | `flip-x`, `flip-y`, `tilt` |
| Reveal | `iris`, `wipe-right`, `wipe-down` |
| FX | `blur` |

Timings: 260 to 440ms. Respects `prefers-reduced-motion`. Transitions are runtime-only; PDF exports rasterize one static frame per slide.

## Watermark

Small, semi-transparent label stamped on every slide. Deck-wide.

```json
{
  "watermark": {
    "text": "DRAFT",
    "position": "bottom-center",
    "opacity": 0.18
  }
}
```

`position`: `bottom-left`, `bottom-center`, `bottom-right`. `opacity`: 0.05 to 0.80, defaults to 0.18. All three fields are required when `watermark` is set; omit the entire field to clear. Uses `var(--color-foreground)` so it stays readable across themes. No per-slide override and no image watermarks (text only).

## Embed block

Iframe with PDF-safe fallback.

```json
{
  "type": "embed",
  "src": "https://www.youtube.com/embed/dQw4w9WgXcQ",
  "title": "Demo: live editing",
  "aspect": "16:9",
  "fallback_src": "https://img.youtube.com/vi/dQw4w9WgXcQ/maxresdefault.jpg",
  "fallback_text": "Watch the demo at https://example.com/demo"
}
```

| Prop | Notes |
|------|-------|
| `src` | Required. Use the embed URL, not the share URL (`youtube.com/embed/...`, not `youtube.com/watch?v=...`). |
| `title` | A11y label and PDF fallback heading. |
| `aspect` | `16:9` (default), `4:3`, `1:1`, `21:9`, `auto`. |
| `allow` | iframe `allow` attribute. Defaults cover fullscreen + autoplay + clipboard. |
| `fallback_src` | Image shown in PDF when iframe can't render. |
| `fallback_text` | Plain-text fallback card. |

For YouTube fallbacks: `https://img.youtube.com/vi/<videoId>/maxresdefault.jpg`. The iframe has `referrerpolicy="strict-origin-when-cross-origin"` and a curated allow list.

## Speaker mode

Append `?mode=speaker` to the viewer URL, or pick **Speaker view** from the editor's **Present** dropdown.

- **Main panel**: current slide, full-size.
- **Side panel** (top-right): `slide.notes` and the next slide's title.
- **Stopwatch** (bottom-left): starts when speaker view mounts. Click to reset.

Keyboard nav matches present mode. Single-monitor screen sharing: present mode on the shared screen, speaker mode on a phone via local network.

## Laser pointer

Active in present and speaker modes. Click-and-drag anywhere; a red trail follows the cursor and fades over about 900ms. No toggle, no settings. Disabled in edit mode (the editor passes `?edit=1` to the viewer, which the runtime reads to skip the laser canvas).

## Phosphor icons

Verso bundles the full Phosphor icon catalog (1,512 icons across six weights: `thin`, `light`, `regular`, `bold`, `fill`, `duotone`). Icons appear in three places:

1. **Standalone `icon` block** (see [blocks.md](blocks.md#icon)): drops into any layout or container.
2. **Per-item bullet icons** (see [blocks.md](blocks.md#bullets)): each bullet item can carry `{ text, icon, iconWeight, iconTone }` for a leading glyph.
3. **Card header strip** (see [blocks.md](blocks.md#card)): cards have optional `header` text and `icon` rendered as a label strip above content.

Icons inherit color from the chosen `tone` via the theme cascade. SVGs are lazy-loaded by the viewer/editor, inlined for PDF and HTML exports (no missing icons in print), and served via Vite's filesystem mount so the editor works regardless of how npm/pnpm hoisted `@phosphor-icons/core`.

In the editor, the `+ Block` menu has an **Icon** entry under Decoration that opens a searchable picker over the full catalog, with a weight selector. Card forms have a Browse-icon button. Each bullet row has a small square icon-picker button.

Browse the catalog at <https://phosphoricons.com> to find names.

## Slide overflow detection

Verso slides render at a fixed 1920x1080 (16:9). Content beyond that gets silently clipped in PDF/PNG exports, so the runtime measures every slide's natural height and reports overshoots.

### CLI

`verso build` (any format) prints one yellow warning per overflowing slide on stdout:

```
⚠ Slide "iterating" overflows by 541px (1621 of 1080).
⚠ Slide "capstone" overflows by 608px (1688 of 1080).
```

The build still succeeds; warnings are informational. The fix is almost always to split the slide or drop a block.

### Editor

The viewer iframe in the editor measures the active slide and `postMessage`s the result to the editor app. The editor shows:

- A red `!` badge next to each overflowing slide in the SlideList.
- A `⚠ active slide overflows by Npx` pill in the toolbar when an overflowing slide is selected.
- A `N slide(s) overflow` suffix on Export success messages.

Authors can fix overflow live: trim a block, watch the badge clear.
