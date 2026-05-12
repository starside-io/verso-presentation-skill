# Verso CLI Commands

Every CLI command, its flags, and what it produces. Read this when you need to remember a flag name or pick between commands.

## verso init

Scaffolds a new Verso project in a fresh directory.

```
verso init [name] [options]
```

| Flag | Description |
|------|-------------|
| `-t, --template <name>` | Template to use. Default: `minimal`. |
| `--with-config` | Adds a starter `verso.config.ts` (custom layouts, components, themes). |

### Templates

| Template | Use when |
|----------|----------|
| `minimal` | Two slides, default styling. Good starting point. |
| `branded` | Custom color palette + cover + closing slide. Polished default. |
| `inline-theme` | Theme config embedded in `deck.json` rather than in `themes/`. |
| `multi-path` | Two content branches for different audiences. Use for path branching. |
| `layouts-gallery` | Demonstrates every built-in layout. Useful as a reference. |
| `extended` | Paths, themes, custom layouts, every block type. Comprehensive example. |

### Examples

```bash
verso init my-deck
verso init q4-readout -t branded --with-config
verso init sales-pitch -t multi-path
```

If no name is provided, the directory is named `verso-deck`.

---

## verso new slide

Creates one or more slides. Three modes: stub, JSON import, Markdown import.

```
verso new slide [id] [options]
```

| Flag | Description |
|------|-------------|
| `-d, --dir <path>` | Project directory. Default: cwd. |
| `-l, --layout <name>` | Layout for the new slide. Default: `title-only` for stubs, preserved when importing. |
| `-p, --paths <ids>` | Comma-separated `path_include` applied to every created slide. |
| `-f, --from <path-or-url>` | Source: a Verso slide JSON, or a Markdown outline (`.md`, `.markdown`, `.mdx`). |

### Stub mode

```bash
verso new slide intro            # slides/intro.json with layout: title-only
verso new slide hero -l hero     # slides/hero.json with layout: hero
verso new slide pricing -p sales # path_include: ["sales"]
```

CLI appends the new id to `deck.json`'s `slide_order`.

### JSON import

```bash
verso new slide --from ../other-deck/slides/cover.json
verso new slide cover2 --from https://example.com/cover.json
```

`id` defaults to the source's `id` field; pass an explicit id to override.

### Markdown outline import

```bash
verso new slide --from outline.md
verso new slide intro --from outline.md         # prefix all ids with "intro-"
verso new slide --from outline.md -l hero -p sales,internal
```

Conversion rules:

- `# Heading` becomes a new slide; heading text becomes `title`.
- `##` and deeper become `heading` blocks inside the current slide.
- Bulleted lists become `bullets` blocks.
- Fenced code blocks become `code` blocks; language tag preserved.
- Plain paragraphs become `text` blocks.
- Inline markdown (bold, italic, links) kept verbatim.
- Slide ids are slugified; collisions get `-2`, `-3` suffixes.
- Content before the first `#` is ignored with a warning.

---

## verso new layout

Scaffolds a custom layout TS file and registers it in `verso.config.ts`.

```
verso new layout <name> [options]
```

| Flag | Description |
|------|-------------|
| `-d, --dir <path>` | Project directory. |

Most decks never need this. Only use if a built-in layout doesn't fit.

---

## verso dev

Live-reload viewer for the current project.

```
verso dev [options]
```

| Flag | Description |
|------|-------------|
| `-d, --dir <path>` | Project directory. |
| `-p, --port <port>` | Server port. Default: `5173`. |
| `-H, --host <host>` | Bind address. Default: `localhost`. |
| `--open` | Open browser automatically. |

URL query params:

- `?path=<id>`: show a specific path; otherwise shows a path selector if multiple exist.
- `?slide=<id>`: jump to a slide.
- `?mode=present`: presentation view.
- `?mode=speaker`: speaker mode (notes + timer + upcoming slide).
- `?mode=debug` or `?debug=1`: diagnostic overlay.

Navigation: right arrow, space, PageDown advance; left, PageUp back; Home, End jump to start/end. In present and speaker modes, mouse drag draws a temporary red annotation.

---

## verso edit

Visual editor with toolbars, inspector, and JSON view. This is the primary authoring surface.

```
verso edit [options]
```

| Flag | Description |
|------|-------------|
| `-d, --dir <path>` | Project directory. |
| `-p, --port <port>` | Editor port. Default: `5180`. |
| `--viewer-port <port>` | Viewer port. Default: `5173`. |
| `-H, --host <host>` | Bind address. |
| `--no-open` | Skip auto-launching the browser. |

Three-pane UI: slide list on the left, live preview center, inspector, transitions, and theme on the right, JSON editor at the bottom.

---

## verso theme

Manage themes.

### theme add

```bash
verso theme add verso-warm     # built-in
verso theme add ./brand.json   # custom; copied into themes/
```

Local themes win over built-ins on name collision.

### theme list

```bash
verso theme list
```

Lists built-ins and project-local themes. Active one is marked `(active)`.

---

## verso build

Export the deck to PDF or HTML.

```
verso build [options]
```

| Flag | Description |
|------|-------------|
| `-d, --dir <path>` | Project directory. |
| `-f, --format <fmt>` | `pdf` or `html`. Default: `pdf`. |
| `-p, --path <id>` | Build one path only. |
| `-o, --out <dir>` | Output directory. Default: `dist/`. |
| `-s, --size <size>` | `16:9`, `4:3`, `letter`, `a4`. Default: `16:9`. |
| `--no-inline-images` | HTML mode only: use image URLs instead of inlining as base64. |

### Format notes

- **PDF**: rendered via headless Chromium, print CSS rules apply, iframes substituted with fallback, transitions ignored.
- **HTML**: single self-contained file. Navigation via arrow keys, space, and hash routing (`#1`, `#2`). Base64 images by default, shareable via email or cloud storage with no extra files.

### Examples

```bash
verso build                              # all paths, PDF 16:9
verso build -p sales                     # sales path only
verso build --format html                # HTML with embedded images
verso build -f html --no-inline-images   # smaller HTML, external images
verso build -f pdf -s a4                 # A4 print
```
