---
name: verso-presentation-skill
description: "Create, edit, and export presentations using Verso, a JSON-driven slide deck system (CLI: @starside-io/verso-cli). Use this skill whenever the user wants to build, author, scaffold, edit, theme, preview, or export a presentation deck, including converting a Markdown outline into slides, generating PDF or HTML exports, applying a theme, adding path branching for multi-audience decks, or working with deck.json / slides/*.json files. Trigger when the user mentions Verso, says they want to build a slide deck or presentation and Verso is available, references a deck.json or verso.config.ts file, asks to import a Markdown outline into slides, or asks about layouts/themes/blocks in a slide-authoring context. Also use when the user has a markdown file and wants slides out of it without specifying a tool, and Verso is installed or installable."
---

# Verso Presentation

Verso is a JSON-driven presentation system. Decks are folders of JSON: a `deck.json` manifest plus one JSON file per slide in `slides/`. Themes are JSON too. The CLI (`verso`) scaffolds projects, runs a dev server, opens a visual editor, and exports to PDF or HTML.

Your job with this skill is to help the user go from "I want a deck about X" to a working Verso project, optionally previewing and exporting it. The skill works in two flavors: starting from scratch (interview the user, hand-author slides) and starting from a Markdown outline (use Verso's built-in importer). The user should never have to learn the schema themselves: you author the files, they review.

## When you start

Before you do anything else, run this small triage. Don't skip it. The first two questions decide the whole shape of the session.

1. **Is the CLI installed?** Run `verso --version`. If it errors, ask the user before installing globally: `npm install -g @starside-io/verso-cli`. Don't install silently. Global npm installs are sticky.

2. **Where does the content come from?** Ask the user this directly, even if they didn't bring it up:

   > "Do you already have a Markdown outline I should import, or are we writing the deck from scratch? Verso has a built-in `--from outline.md` importer that turns `#` headings into slides, lists into bullets, and code fences into code blocks. If you've got a .md draft, that's the fastest path."

   If they have an MD file, jump to **Path A: Markdown import** below. If not, go to **Path B: From scratch**.

3. **Capture the basics either way.** Title, audience(s), rough length (5 slides? 20?), tone (formal vs casual), and whether they want a particular theme or color palette. If you don't ask, you'll guess wrong.

4. **Read the reference files before writing any slide content.** Before you create or edit any slide JSON, read [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md) in full. These contain every valid block type, layout name, prop, and enum value. Do not guess field values from memory: look them up. Incorrect enum values (e.g. `"center"` instead of `"middle"` for vertical alignment) are silently accepted or rejected at render time, and debugging them is painful. The references are the source of truth.

## Path A: Markdown import

The fastest happy path. Verso parses Markdown into slides using these rules:

- `# Heading` becomes a new slide; the heading text becomes the slide title.
- `##` and deeper headings become `heading` blocks inside the current slide.
- Bulleted lists become `bullets` blocks.
- Fenced code blocks (` ``` `) become `code` blocks; the language tag is preserved.
- Plain paragraphs become `text` blocks.
- Inline markdown (bold, italic, links) is kept verbatim.
- Slide IDs are slugified from headings (`# Welcome` becomes `welcome`). Duplicates get `-2`, `-3` suffixes.
- Anything before the first `#` is ignored with a warning.

### Steps

1. Init the project. Pick a template based on what they need (see [references/commands.md](references/commands.md) for the full list; `minimal` is the default).
   ```bash
   verso init <name>
   cd <name>
   ```

2. Import the outline.
   ```bash
   verso new slide --from path/to/outline.md
   ```
   This creates one slide JSON per `#` heading and appends them all to `deck.json`'s `slide_order`. If they want all generated slides tagged with paths (multi-audience decks), pass `-p sales,internal`. If they want a specific layout applied to all, pass `-l hero`.

3. **Read the result.** Open `deck.json` and at least 2-3 generated slides. The importer is lossy by design: it doesn't know what should be a `cover` vs a `content` slide, doesn't pick image-heavy layouts, and doesn't add decorations. Pass over the slides and improve them (set `layout: cover` on the first slide, switch a list-heavy slide to `timeline` if it's a roadmap, add `callout`s for important asides). See [references/layouts.md](references/layouts.md) for what each layout expects.

4. Show the user a preview. `verso edit` opens the visual editor in their browser (port 5180 by default); `verso dev` is the lighter live-reload viewer. Ask which they want. If they're going to keep editing in the editor, hand off there. The editor is fully WYSIWYG.

5. When they're happy, build. See [references/commands.md](references/commands.md#verso-build).

## Path B: From scratch

When there's no Markdown to import, you're authoring the JSON yourself based on a short interview.

### Interview

Ask in one batch (don't drip-feed questions one at a time):

- What's the deck title?
- Who's the audience? Is there more than one? (If yes, plan paths; see [references/path-branching.md](references/path-branching.md).)
- Roughly how many slides? What's the rough outline (sections, key points)?
- Any branding constraints (colors, font, logo)? Or pick a built-in theme?
- Do they need speaker notes, watermarks, embedded videos, transitions? (Mention these as options; most decks don't need them.)

If the user seems hesitant or doesn't know, suggest a template and ask them to react to it. Concrete proposals are easier to redirect than open questions.

### Author

1. `verso init <name> -t <template>`. Templates: `minimal`, `branded`, `inline-theme`, `multi-path`, `layouts-gallery`, `extended`. Pick `branded` if they want a polished default; `multi-path` if there are multiple audiences; `extended` if you want a reference for every block type while authoring.

2. Edit `deck.json`. Set `title`, `theme`, `paths`, and `slide_order`. The manifest is the only required file. Minimal example:
   ```json
   {
     "title": "Q4 Readout",
     "theme": "verso-slate",
     "paths": { "full": { "label": "Full Deck" } },
     "slide_order": ["cover", "agenda", "results", "next-steps", "closing"]
   }
   ```

3. Write one slide JSON per id in `slide_order`, in `slides/<id>.json`. The shape:
   ```json
   {
     "id": "results",
     "header": "Q4",
     "title": "What landed",
     "layout": "content",
     "content": [
       { "type": "heading", "level": 2, "text": "Highlights" },
       { "type": "bullets", "items": ["Shipped X", "Grew Y by 12%"] }
     ],
     "notes": "Speaker notes go here. Visible only in speaker mode."
   }
   ```

   Use the `verso new slide <id> -l <layout>` CLI to scaffold each stub, then fill in `content`. Or just write the JSON files directly. Verso doesn't care which way you got there.

4. **Pick layouts deliberately, don't default everything to `content`.** Cover slide goes to `cover`. Section dividers go to `section`. A vs B goes to `compare`. Big metric goes to `big-number`. Pull quote goes to `quote`. Closing slide goes to `closing`. See [references/layouts.md](references/layouts.md) for the full list and what each one expects in `content`.

5. **Reach for blocks beyond `bullets`.** Slide after slide of bullets is the most common Verso failure mode. Use `callout` for warnings and tips, `card` and `panel` for grouped content, `image` (drop files in `assets/`), `code` for snippets, `accent-bar` and `divider` for visual rhythm. See [references/blocks.md](references/blocks.md) for the full catalog.

6. Preview and iterate. Open `verso edit` and walk through the deck. Fix what looks off. The visual editor lets the user click around and tweak; if they want to keep iterating in there, that's fine, hand off.

## Themes

Built-in themes (no install needed, just set `theme` in `deck.json`):

- `verso-slate` (default): slate gray + steel blue, professional.
- `verso-warm`: warm beige + terracotta.
- `verso-mono`: monochrome charcoal.
- `verso-neon`: cyberpunk dark with neon accents.
- `verso-mars`: dark with deep red.
- `verso-forest`: cream + forest green + sage.

To preview: `verso theme list`. To swap: edit the `theme` field in `deck.json` or run `verso theme add <name>`. For brand colors that aren't covered by a built-in, use `style_overrides` on the deck, slide, or block level (the cascade resolves later levels winning). See [references/themes.md](references/themes.md).

## Multi-audience decks (path branching)

If the user said the deck has more than one audience (sales vs eng, exec vs IC), use paths. Each path is a named filter; slides and even individual blocks can be tagged `path_include` and `path_exclude`. The same source authors many versions. See [references/path-branching.md](references/path-branching.md). Don't bring this up if there's only one audience. It's overhead.

## Exporting

`verso build` exports the deck. Defaults to PDF, 16:9.

- `verso build` for all paths, PDF, 16:9.
- `verso build -f html` for a single self-contained HTML file with embedded base64 images. Best for email or cloud sharing.
- `verso build -f pdf -s a4` for print-friendly A4 PDF.
- `verso build -p sales` for only the `sales` path.
- `verso build -f html --no-inline-images` for smaller HTML using image URLs instead of base64.

Output goes to `dist/` unless `-o` says otherwise. Ask the user which format they want before building. Defaults are fine for most cases but check.

## What you should and shouldn't do

- **Do** read [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md) before writing any slide JSON. Never guess block types, layout names, or enum values from memory. Look them up.
- **Do** open `verso edit` for the user once the deck is in decent shape. The visual editor is the primary intended authoring surface; you're scaffolding to save them time, not replacing it.
- **Do** read 2-3 generated slides after `--from outline.md` and improve layouts and visual rhythm. The importer always produces all-`content`-layout slides with no decorations.
- **Don't** install `@starside-io/verso-cli` globally without asking first. Global npm installs are sticky.
- **Don't** invent block types or layout names. The full list is in [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md). Anything not there will silently render as an empty placeholder.
- **Don't** write a `verso.config.ts` unless the user actually needs custom layouts or components. The vast majority of decks need only `deck.json` and `slides/*.json`.

## Reference files

When you need exact prop tables, full block lists, or the path-filter semantics, read these. They're organized so you only load what you need.

- [references/commands.md](references/commands.md): every CLI command with all flags and examples.
- [references/blocks.md](references/blocks.md): every block type, its props, defaults, and JSON examples.
- [references/layouts.md](references/layouts.md): all 17 built-in layouts and what content they expect.
- [references/themes.md](references/themes.md): built-in themes, the color cascade, and per-level overrides.
- [references/path-branching.md](references/path-branching.md): `path_include` and `path_exclude` rules for slides and blocks.
- [references/features.md](references/features.md): transitions, variables, watermark, embed, speaker mode.

## Templates

- [assets/example-deck.json](assets/example-deck.json): a minimal multi-slide `deck.json` you can copy and modify.
- [assets/example-slide.json](assets/example-slide.json): a slide JSON showing common block types in `content`.
- [assets/example-outline.md](assets/example-outline.md): a markdown outline showing the import format so users can see what the importer expects.
