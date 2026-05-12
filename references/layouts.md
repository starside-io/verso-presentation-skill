# Layouts

Verso ships 17 built-in layouts. Set the layout on a slide with `"layout": "<name>"`. If the layout expects specific block shapes (e.g. an `image` for `image-left`) and they're missing, it falls back to sensible defaults rather than erroring.

## Core layouts

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `content` | Any blocks | Default. Stacks blocks vertically. |
| `two-col` | Any blocks | Splits content roughly in half across two columns. |
| `three-col` | Any blocks | Three equal columns. |
| `image-left` | One `image` block + others | Image on the left, rest fills the right. |
| `image-right` | Same | Mirror of `image-left`. |
| `hero` | A `heading` + an `image` | Big heading, large image, optional supporting text. |
| `full-image` | One `image` | Image bleeds to slide edges; text overlays on top. |

## Openers and closers

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `cover` | Slide `title` + optional `header` | First slide. Centered, big type. |
| `section` | Slide `title` | Visual divider between sections. |
| `closing` | Slide `title` + optional contact text | Last slide. Thanks, Questions, etc. |
| `author` | A `heading` + an `image` | Author bio: portrait + name + role. |

## Structured layouts

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `agenda` | `bullets` (or `heading` blocks, or nothing) | Auto-builds from the deck outline when empty. |
| `compare` | Two `card` blocks side by side | A vs B comparison. |
| `stats` | Several `card` blocks, often with big numbers | Metric tiles. |
| `big-number` | One `heading` with the number + a `text` for context | One huge number, one short label. |
| `quote` | A `quote` block | Big quote, optional attribution. |
| `timeline` | A `bullets` block | Renders as horizontal timeline. |

## Slide-level fields layouts can read

- `slide.title` and `slide.header` (rendered by helper functions)
- `slide.align` for per-slide vertical alignment of content within the slide
- `slide.transition` (runtime stamps the class)
- `slide.notes`, `slide.annotation` (speaker mode only)
- `slide.omit_from_agenda` (read by the agenda layout's fallback)

### `slide.align` values

| Value | Effect |
|-------|--------|
| `top` | Content pins to the top of the slide. |
| `middle` | Content is vertically centered (default for most layouts). |
| `bottom` | Content pins to the bottom. |

**Do not use `center` for vertical alignment.** The correct value is `middle`. Verso will reject unknown values.

## Agenda auto-build

The `agenda` layout falls back through three tiers:

1. A manual `bullets` block on the slide. Wins if present.
2. `heading` blocks on the slide. Their text becomes the list.
3. The deck outline from `ctx.deckOutline`: section markers if any exist, otherwise every path-filtered slide title.

Slide titles in the auto fallback skip any slide with `omit_from_agenda: true` (use this for cover, the agenda slide itself, closing slides).

Item count drives density: 8 or fewer in one column, 9 to 20 in two columns, more than 20 in three columns with smaller type.

## How to pick a layout

| You have... | Reach for |
|-------------|-----------|
| The first slide of the deck | `cover` |
| The last slide | `closing` |
| A divider between sections | `section` |
| A list of upcoming topics | `agenda` |
| One key metric you want to land hard | `big-number` |
| A customer quote | `quote` |
| Two approaches to compare | `compare` |
| Multiple metrics to show | `stats` |
| A roadmap or chronological list | `timeline` |
| A screenshot + explanation | `image-left` or `image-right` |
| A diagram that should fill the slide | `full-image` |
| Two parallel topics, equal weight | `two-col` |
| Three buckets or categories | `three-col` |
| Anything else | `content` |

## Custom layouts

A layout is an object with a `name` and a `render(slide, ctx)` function. Scaffold with `verso new layout <name>`, which creates a TS file in `layouts/` and registers it in `verso.config.ts`.

```ts
import { defineLayout, html } from '@starside-io/verso-runtime'

export const splitThirds = defineLayout({
  name: 'split-thirds',
  render: (slide, ctx) => {
    const [first, ...rest] = ctx.blocks
    return html`
      <div class="layout-split-thirds">
        <div class="left">${first ? ctx.block(first) : ''}</div>
        <div class="right">${rest.map((b, i) => ctx.block(b, [i + 1]))}</div>
      </div>
    `
  },
})
```

Register in `verso.config.ts`:

```ts
import { splitThirds } from './layouts/split-thirds'

export default { layouts: [splitThirds] }
```

Then any slide can use `"layout": "split-thirds"`.

### Layout context

```ts
interface LayoutContext {
  pathId: string                  // Active path id
  colors: ResolvedColors          // Resolved color roles for this slide
  blocks: ContentBlock[]          // Path-filtered, flattened
  deckOutline: DeckOutlineEntry[] // Whole-deck outline
  block: (b, path?) => HTML       // Render a single block
  component: (c, props) => HTML   // Render a custom component
}
```

`ctx.block(b)` renders a child block. The runtime wraps it in a `.verso-block` div with the right CSS variables.

## CSS conventions

Built-in layouts target classes like `.layout-content`, `.layout-two-col`. Custom CSS should:

- Reference CSS variables (`var(--color-primary)`) rather than hex, so themes propagate.
- Avoid `position: absolute` for primary content; the slide section is a flex column.
- Set `flex: 1` on internal columns so they stretch to fill the slide height.
