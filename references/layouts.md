# Layouts

Verso ships 36 built-in layouts. Set the layout on a slide with `"layout": "<name>"`. If the layout expects specific block shapes (e.g. an `image` for `image-left`) and they're missing, the schema rejects it at parse time (see "Layout-aware schema validation" at the bottom).

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
| `roadmap` | Multiple blocks; each becomes a quarter / milestone column | Quarterly product roadmap with Q1 / Q2 / Q3 / Q4 columns. |
| `q-and-a` | Two blocks (Q text + A text), more optional | FAQ slide, panel Q&A, or anticipating audience questions. |

## Asymmetric splits

PowerPoint-style 1/3 + 2/3 splits. Two top-level blocks; the first goes in the left column, the second in the right.

| Layout | Split | Use for |
|--------|-------|---------|
| `one-third-left` | 1/3 left + 2/3 right | Label / sidebar on the left, main content on the right. |
| `one-third-right` | 2/3 left + 1/3 right | Main content on the left, sidebar / callout on the right. |
| `two-thirds-left` | 2/3 left + 1/3 right (alt name) | Same as `one-third-right`; alt name for clarity. |
| `two-thirds-right` | 1/3 left + 2/3 right (alt name) | Same as `one-third-left`; alt name for clarity. |
| `split-vertical` | Top half + bottom half (full-width) | Two parallel concepts stacked vertically. |

Requires a multiple of 2 top-level blocks (one per column / half).

## Grids

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `quad` | Exactly 4 blocks (one per cell) | Generic 2x2 grid: four categories, four features, four pillars. |
| `swot` | Exactly 4 blocks | Strengths / Weaknesses / Opportunities / Threats analysis. Labels are auto-generated. |
| `icon-grid` | 3+ blocks (auto-flows 2x2, 3x2, 4x2) | Feature list with icons; product highlights; capability tiles. |
| `kpi-band` | 1+ `card` blocks (KPIs) + optional supporting content below | Dashboard view: small KPI cards across the top, narrative below. |
| `bento` | 3+ blocks (first cell spans larger) | Magazine / dashboard look with one hero cell + smaller cells. |

## Image-driven

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `picture-fill` | One `image` block + slide title (+ optional supporting blocks) | Background image with title overlay; pitch-deck or marketing-style cover. |
| `picture-with-caption` | One `image` block + 1+ caption blocks | Product shot + long-form caption side by side. Different from `image-left` because the caption is type-prominent, not block-stack. |

## Flow

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `process` | 2+ blocks, each becomes a step | Horizontal chevrons / numbered process steps left-to-right. |

## Special

| Layout | What it expects | Use for |
|--------|-----------------|---------|
| `title-band` | Slide title + body blocks | Full-width colored title bar at top, body below. Heavier title than `content`. |
| `title-only` | Just the slide title | Empty canvas with one centered title. Stand-in / blank slide. |
| `callout-banner` | Slide title + first block as banner body | Single attention-grabbing message. Banner-style alert or announcement. |
| `chapter` | Slide title (+ optional intro paragraph) | Book-style chapter divider. Heavier than `section`. |

## Slide-level fields layouts can read

- `slide.title` and `slide.header` (rendered by helper functions)
- `slide.align` for per-slide alignment of content within the slide
- `slide.transition` (runtime stamps the class)
- `slide.notes`, `slide.annotation` (speaker mode only)
- `slide.omit_from_agenda` (read by the agenda layout's fallback)

### `slide.align` values

The flat form applies to BOTH the title zone and the content zone:

```json
"align": { "horizontal": "center", "vertical": "middle" }
```

| Key | Valid values | Default |
|-----|-------------|---------|
| `horizontal` | `left`, `center`, `right` | `left` |
| `vertical` | `top`, `middle`, `bottom` | `top` |

**Do not use `center` for vertical alignment.** The correct value is `middle`. Verso will reject unknown values.

### Per-zone alignment (PowerPoint-style)

Multi-zone layouts (`content`, `two-col`, `three-col`, `image-left`, `image-right`, `hero`, `agenda`, `compare`, `stats`, `big-number`, `timeline`, `author`) support separate alignment for the title strip and the content body:

```json
"align": {
  "title":   { "horizontal": "left",   "vertical": "top"    },
  "content": { "horizontal": "center", "vertical": "middle" }
}
```

Use this when you want the title pinned to the top but the body centered vertically (the common "hero" feel without the hero layout). The flat form still works as a back-compat shortcut applied to both zones. Single-zone layouts (`cover`, `section`, `closing`, `quote`, `full-image`) ignore the nested form and use the flat values.

## Layout-aware schema validation

`Slide.parse` enforces minimum block counts per layout. The schema rejects unbalanced multi-col slides at load time with a readable error naming the slide id, the layout, and what's missing.

| Layout | Requirement |
|--------|-------------|
| `two-col` | `content.length` must be a multiple of 2. |
| `three-col` | `content.length` must be a multiple of 3. |
| `image-left`, `image-right` | At least one block of type `image`. |
| `compare` | At least two top-level `heading` groups (one per side). |
| `one-third-left`, `one-third-right`, `two-thirds-left`, `two-thirds-right` | Multiple of 2 blocks (one per column). |
| `split-vertical` | Multiple of 2 blocks (one per half). |
| `quad`, `swot` | Exactly 4 blocks. |
| `icon-grid` | At least 3 blocks. |
| `bento` | At least 3 blocks. |
| `picture-fill` | At least one block of type `image`. |
| `picture-with-caption` | At least one `image` block + at least one caption block. |
| `q-and-a` | At least 2 blocks (the Q and the A). |
| `process` | At least 2 step blocks. |
| `roadmap` | At least 2 blocks (one per quarter / milestone). |
| `callout-banner` | At least 1 block. |
| `kpi-band` | At least 1 block. |
| Others | Unconstrained. |

This catches a common AI-authoring failure: generating a `two-col` slide with only 1 block (empty right column) or a `three-col` with 2 blocks (one column empty). If a slide refuses to parse, restructure the content or pick a different layout (`content` doesn't enforce anything).

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
| A heavier book-style divider | `chapter` |
| A blank canvas with just a title | `title-only` |
| A list of upcoming topics | `agenda` |
| One key metric you want to land hard | `big-number` |
| A customer quote | `quote` |
| Two approaches to compare | `compare` |
| Multiple metrics to show | `stats` |
| KPIs across the top + narrative below | `kpi-band` |
| A roadmap or chronological list | `timeline` |
| Quarterly roadmap (Q1 / Q2 / Q3 / Q4 columns) | `roadmap` |
| Question + answer pattern | `q-and-a` |
| A process with sequential steps | `process` |
| A screenshot + explanation | `image-left` or `image-right` |
| A diagram that should fill the slide | `full-image` |
| Image as background, title on top | `picture-fill` |
| Large image + long caption side by side | `picture-with-caption` |
| Two parallel topics, equal weight | `two-col` |
| Three buckets or categories | `three-col` |
| 1/3 + 2/3 asymmetric split | `one-third-left` or `one-third-right` |
| Two halves stacked top + bottom | `split-vertical` |
| Four categories in a 2x2 grid | `quad` |
| SWOT analysis | `swot` |
| Feature tiles with icons | `icon-grid` |
| Magazine-style asymmetric grid (one hero + smaller) | `bento` |
| Full-width title bar with body below | `title-band` |
| A single attention-grabbing message | `callout-banner` |
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
