# Path branching

The Verso pitch: one deck, multiple audiences. You author every slide once and tag which audiences see what. At present-time, the viewer renders only the slides (and blocks within slides) that survive the path filter.

A deck with no `paths` is single-audience: everything renders for everyone. Don't bring up paths if there's only one audience. It's overhead.

## Declaring paths

In `deck.json`:

```json
{
  "title": "Q4 readout",
  "paths": {
    "sales":       { "label": "Sales team", "color": "#5eead4" },
    "engineering": { "label": "Engineering", "color": "#a78bfa" },
    "exec":        { "label": "Executive summary", "color": "#fb923c" }
  },
  "slide_order": [...]
}
```

Each path has a `label` (display name in the picker) and an optional `color` (used in the editor's Paths view and the path picker swatch).

## Slide-level filtering

```json
{
  "id": "deep-dive",
  "title": "Architecture deep dive",
  "layout": "two-col",
  "path_include": ["engineering"],
  "content": [...]
}
```

Rules:

- `path_include`: a whitelist. Slide renders only in these paths. Default: all paths.
- `path_exclude`: a blacklist. Slide is dropped from these paths. Wins over `path_include`.
- A slide is included if `pathId` is not in `path_exclude` AND (`path_include` is empty OR `pathId` is in `path_include`).

## Block-level filtering

Same fields on any block, including ones nested inside `card` or `panel`:

```json
{
  "type": "bullets",
  "items": ["Common point", "Sales-only point", "Engineering-only point"],
  "path_include": ["sales", "engineering"]
}
```

For different items per path, either split into two `bullets` blocks (each with its own `path_include`) or use variables for the simple case.

## URL parameters at present-time

| Param | Effect |
|-------|--------|
| `?path=<id>` | Render the named path. |
| (none) + multiple paths in manifest | Show the path picker. |
| (none) + single path in manifest | Render that path directly. |

## In the editor: the Paths view

Click **Paths** in the toolbar to switch from the slide editor to a timeline view.

- **Legend**: one pill per path. Click to filter. `+ Path` button adds a new path. `Path colors` and `Theme colors` toggle swaps SVG line coloring.
- **Timeline**: each slide is a card; cards stack into lanes by path membership.
- **Path lines**: one continuous line per path, threading through every slide it includes.
- **Path checkboxes per card**: toggle `path_include` membership without leaving the view.

Drag cards horizontally to reorder slides. Double-click a card to open the slide editor.

## Programmatic access in custom layouts

```ts
defineLayout({
  name: 'audience-aware',
  render: (slide, ctx) => {
    if (ctx.pathId === 'exec') {
      return html`<div class="exec">...</div>`
    }
    return html`<div>...</div>`
  },
})
```

Most projects use schema-level filtering rather than code-level checks.

## Tips

- **Use sections**. Add `section` markers in `slide_order` (via the Paths view) to group slides. The agenda layout's auto-build prefers sections over individual slide titles when both exist.
- **Keep slide ids stable**. The editor tracks history per slide id; renaming loses its undo stack.
- **Don't over-branch**. If two paths share 80% of the deck, branching at the block level is cleaner than maintaining two near-identical slides.
