---
name: verso-presentation-skill
description: "Create, edit, and export presentations using Verso, a JSON-driven slide deck system (CLI: @starside-io/verso-cli). Use this skill whenever the user wants to build, author, scaffold, edit, theme, preview, or export a presentation deck, including converting a Markdown outline into slides, generating PDF or HTML exports, applying a theme, adding path branching for multi-audience decks, or working with deck.json / slides/*.json files. Trigger when the user mentions Verso, says they want to build a slide deck or presentation and Verso is available, references a deck.json or verso.config.ts file, asks to import a Markdown outline into slides, or asks about layouts/themes/blocks in a slide-authoring context. Also use when the user has a markdown file and wants slides out of it without specifying a tool, and Verso is installed or installable."
---

# Verso Presentation

Verso is a JSON-driven presentation system. Decks are folders of JSON: a `deck.json` manifest plus one JSON file per slide in `slides/`. Themes are JSON too. The CLI (`verso`) scaffolds projects, runs a dev server, opens a visual editor, and exports to PDF or HTML.

**The goal of this skill is to produce a presentation that looks designed**, not auto-generated. A working deck where every slide uses `content` layout, every block is a card, and every bullet has an icon is a failure mode, even if the schema accepts it. The user is trusting you with their visual reputation. Treat it like that.

Your job is to help the user go from "I want a deck about X" to a polished Verso project, optionally previewing and exporting it. The skill works in two flavors: starting from scratch (interview the user, hand-author slides) and starting from a Markdown outline (use Verso's built-in importer). The user should never have to learn the schema themselves: you author the files, they review.

## When you start

Before you do anything else, run this small triage. Don't skip it. The first two questions decide the whole shape of the session.

1. **Is the CLI installed?** Run `verso --version`. If it errors, ask the user before installing globally: `npm install -g @starside-io/verso-cli`. Don't install silently. Global npm installs are sticky. `verso init` detects whether `verso` is on PATH and only adds the CLI as a project dep when it isn't, so a global install means a clean scaffold with no redundant `node_modules`.

2. **Where does the content come from?** Ask the user this directly, even if they didn't bring it up:

   > "Do you already have a Markdown outline I should import, or are we writing the deck from scratch? Verso has a built-in `--from outline.md` importer that turns `#` headings into slides, lists into bullets, and code fences into code blocks. If you've got a .md draft, that's the fastest path."

   If they have an MD file, jump to **Path A: Markdown import** below. If not, go to **Path B: From scratch**.

3. **Capture the basics either way.** Title, audience(s), rough length (5 slides? 20?), tone (formal vs casual), and whether they want a particular theme or color palette. If you don't ask, you'll guess wrong.

4. **Read the reference files before writing any slide content.** Before you create or edit any slide JSON, read [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md) in full. These contain every valid block type, layout name, prop, and enum value. Do not guess field values from memory: look them up. Incorrect enum values (e.g. `"center"` instead of `"middle"` for vertical alignment) are silently accepted or rejected at render time, and debugging them is painful. The references are the source of truth.

## Design principles

These apply to both paths below. Read them before authoring slides; they're the difference between a deck that looks designed and one that looks AI-generated.

### Vary the layout from slide to slide

A deck where every slide uses `content` is the AI tell. The deck should feel like it has rhythm: openers (`cover`, `section`), structured slides (`compare`, `stats`, `big-number`, `quote`, `timeline`), text-and-image slides (`image-left`, `image-right`, `hero`, `full-image`), and a closer (`closing`). The `content` layout is the fallback when nothing more specific fits, not the default.

Rule of thumb: across a 15-slide deck, you should be using **at least 4-5 different layouts**, not the same one 15 times. Open the deck after authoring and ask "does any single layout dominate?" If yes, fix it.

### Match content density to layout

| Content density | Pick from |
|-----------------|-----------|
| 1-3 short blocks (a quote, a metric, a section title) | `cover`, `section`, `closing`, `hero`, `big-number`, `quote`, `full-image` |
| 4-8 blocks, mixed text and image | `content`, `image-left`, `image-right`, `two-col` |
| 4-12 short parallel items (features, steps) | `three-col`, `stats`, `timeline`, `agenda` |
| Two parallel things to compare | `compare` |

Sparse content on a `content` layout looks like a half-empty page. A `hero` or `big-number` makes the same content land.

### Bump font sizes when slides are sparse

Verso renders at a fixed 1920x1080. A slide with two sentences in default `text` looks lost. Three options:

1. **Move to a layout with bigger built-in type**: `hero` (oversized title + subtitle), `big-number` (huge number + label), `cover` (centered, large), `quote` (pulled quote, large). These don't need any font tweaking. **Prefer this option.**
2. **Use `heading` level 1 instead of level 2 or 3** for any text that should be visually heavy. `level: 1` is the biggest.
3. **Add a project `styles.css`** (or `verso.css`) at the project root with custom rules:
   ```css
   /* slides/highlights.json gets bigger body text */
   .verso-slide[data-slide-id="highlights"] .verso-text {
     font-size: 1.5rem;
     line-height: 1.5;
   }
   ```
   The file is auto-loaded by the viewer and bundled into PDF / HTML exports. Use this when a particular slide needs a one-off size beyond what the layout offers.

Don't pad a sparse slide with filler text to "fill the space". Filler is more obvious than empty space.

### Don't trade overflow for empty space, or empty space for overflow

After authoring, run `verso build` and check the stdout for `⚠ Slide overflows by Npx` warnings. Each overflow means content past the page boundary is silently clipped. Two real fixes:

- **Rework the layout.** A `content`-layout slide with 12 blocks usually fits as `two-col` or `three-col`. A wall of bullets can become two paired `card` blocks. A long list of tips can become a `stats` grid.
- **Split into two slides.** If the content is genuinely too much for one slide, split it. Two slides at 80% full beat one slide at 130% with the bottom cut off. Title the split slides with a Part 1 / Part 2 pattern or just continue the topic naturally.

The fix is NOT "delete content until it fits", and it's NOT "leave the overflow because it's almost there". Pick rework or split.

The reverse failure: sparse slides with huge empty areas. Apply the density-to-layout table above. Empty space is a design choice in the right layout (`big-number`, `quote`, `cover`) and a bug in the wrong one (`content` with two bullets).

### Verify visually before declaring done

You don't have to guess at how a slide looks. `verso build -f png` exports each slide as a 1920x1080 PNG to `dist/`. Pick a couple of slides that worry you (the ones with the most content, the ones with the fewest, anything where you reached for a card or icon decision) and check them. The PNG files are small enough to read directly and tell you immediately whether the layout is working.

When the user opens `verso edit`, they get live preview + the per-slide overflow badge in the slide list. Either path beats shipping a deck blind.

### Consistency, then dynamic

The deck should feel like one document. Same theme, same set of card tones used the same way, same kind of decoration. **Within that consistent shell**, vary the layout per slide so it has rhythm. The opposite (different themes per slide, ad-hoc card colors everywhere) is chaos.

If you're tempted to switch theme or restyle for "variety", don't. The variety comes from layout choice, not theme.

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

4. **Pick layouts deliberately, don't default everything to `content`.** Cover slide goes to `cover`. Section dividers go to `section`. A vs B goes to `compare`. Big metric goes to `big-number`. Pull quote goes to `quote`. Closing slide goes to `closing`. See [Design principles → Vary the layout](#vary-the-layout-from-slide-to-slide) above and [references/layouts.md](references/layouts.md) for the full list and what each one expects in `content`.

5. **Use the right block for the content. Don't default to bullets, but don't default to cards either.** Two opposite failure modes:
   - Slide after slide of `bullets` reads like a Word doc. Reach for `callout` (warnings/tips), `quote` (testimonials), `image`, `code`, `accent-bar`, `divider` to break the rhythm.
   - Wrapping every block in a `card` reads like a Trello board. Cards have weight: borders, padding, internal margins. Stacking 3-4 cards on the same slide visually flattens hierarchy and looks busy. **Reach for `card` only when the content is genuinely a discrete unit** (a definition, an example, a comparison side, a metric tile). For prose, sub-points, or simple structure, plain `heading` + `text` + `bullets` is almost always better.
   - Cards can carry their own `header` text and a Phosphor `icon` strip. Use those when a card represents a named thing (a feature, a step, a card you'd actually call "the Goal card"). Skip them for cards that just hold a heading + body.
   - Bullet items can carry per-item leading icons via `{ text, icon, iconWeight, iconTone }`. Useful for short feature lists or checklists. Skip on long prose-style bullets where the icon column adds noise.
   - The standalone `icon` block is for hero positions: a single big glyph above a section title, or a tile inside a stats layout. Don't sprinkle inline `icon` blocks through body text.

   See [references/blocks.md](references/blocks.md) for the full catalog.

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
- `verso build -f png` for per-slide PNG screenshots at 1920x1080.
- `verso build -f pdf -s a4` for print-friendly A4 PDF.
- `verso build -p sales` for only the `sales` path.
- `verso build -f html --no-inline-images` for smaller HTML using image URLs instead of base64.

Output goes to `dist/` unless `-o` says otherwise. Ask the user which format they want before building. Defaults are fine for most cases but check.

### Overflow warnings

Every build target measures each rendered slide's natural height against the page height (1080px at 16:9). Slides that overshoot get a yellow warning line on stdout:

```
⚠ Slide "iterating" overflows by 541px (1621 of 1080).
⚠ Slide "capstone" overflows by 608px (1688 of 1080).
```

The build still completes (overflow doesn't fail it) but content past the page boundary is silently clipped in the PDF/PNG. The same detection runs in the live editor preview: overflowing slides get a red badge in the slide list, the toolbar shows a `⚠ active slide overflows by Npx` pill, and the Export success message suffixes "N slide(s) overflow" if relevant.

See [Design principles → Don't trade overflow for empty space](#dont-trade-overflow-for-empty-space-or-empty-space-for-overflow). Briefly: rework the layout or split the slide. Don't ignore the warning and don't just delete content.

### Visual check before sign-off

When in doubt about how a slide actually looks, render it:

```bash
verso build -f png -p full
```

Per-slide PNGs land in `dist/`. Open the ones for your highest-density and lowest-density slides. Reading the PNG tells you in 5 seconds whether the layout works, whether anything overflows, and whether sparse slides have too much empty space. Cheaper than re-running `verso edit`.

## What you should and shouldn't do

- **Do** read [Design principles](#design-principles) before writing any slide. Layout variety, density matching, and the don't-trade-overflow-for-empty-space rule are the difference between a designed deck and AI slop.
- **Do** read [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md) before writing any slide JSON. Never guess block types, layout names, or enum values from memory. Look them up.
- **Do** open `verso edit` for the user once the deck is in decent shape. The visual editor is the primary intended authoring surface; you're scaffolding to save them time, not replacing it.
- **Do** read 2-3 generated slides after `--from outline.md` and improve layouts and visual rhythm. The importer always produces all-`content`-layout slides with no decorations. Change at least half of them to something more specific.
- **Do** `verso build -f png` a couple of slides when you're unsure how something will look. Quicker than re-launching the editor.
- **Don't wrap every block in a `card`.** It looks structured to the LLM but visually it's a sea of bordered boxes. Cards are for discrete named units (Goal, Context, Rules, Step 1, Step 2). For prose, plain heading + text + bullets reads better. If you find yourself writing 3+ cards on a `content`-layout slide, ask whether the content is really 3 separate things or just one thing you've over-fragmented.
- **Don't use the same layout on every slide.** Across a deck, you should reach for at least 4-5 different layouts. See [Design principles → Vary the layout](#vary-the-layout-from-slide-to-slide).
- **Don't add icons everywhere.** Per-item bullet icons are great for short feature lists; they add noise on prose bullets. Standalone `icon` blocks are for hero positions, not inline decoration. Card `icon` strips are for cards that represent a named thing.
- **Don't pad sparse slides with filler.** A sparse slide on the wrong layout looks empty. Move it to `hero`, `big-number`, `quote`, `cover`, or `section` and let the type sing. See [Design principles → Bump font sizes when slides are sparse](#bump-font-sizes-when-slides-are-sparse).
- **Don't ignore overflow warnings.** Rework the layout or split the slide. See [Design principles → Don't trade overflow for empty space](#dont-trade-overflow-for-empty-space-or-empty-space-for-overflow).
- **Don't** put long code lines without thinking. Wrap happens at word boundaries, but a single 200-character string still looks ugly. Break long examples across multiple lines or trim them.
- **Don't** install `@starside-io/verso-cli` globally without asking first. Global npm installs are sticky.
- **Don't** invent block types or layout names. The full list is in [references/blocks.md](references/blocks.md) and [references/layouts.md](references/layouts.md). Anything not there will silently render as an empty placeholder.
- **Don't** write a `verso.config.ts` unless the user actually needs custom layouts or components. The vast majority of decks need only `deck.json` and `slides/*.json`.
- **Don't switch themes mid-deck for variety.** Consistency at the theme level, variety at the layout level.

## Reference files

When you need exact prop tables, full block lists, or the path-filter semantics, read these. They're organized so you only load what you need.

- [references/commands.md](references/commands.md): every CLI command with all flags and examples.
- [references/blocks.md](references/blocks.md): every block type, its props, defaults, and JSON examples.
- [references/layouts.md](references/layouts.md): all 17 built-in layouts and what content they expect.
- [references/themes.md](references/themes.md): built-in themes, the color cascade, and per-level overrides.
- [references/path-branching.md](references/path-branching.md): `path_include` and `path_exclude` rules for slides and blocks.
- [references/features.md](references/features.md): transitions, variables, watermark, embed, speaker mode, overflow detection, Phosphor icons, per-zone alignment.

## Templates

- [assets/example-deck.json](assets/example-deck.json): a minimal multi-slide `deck.json` you can copy and modify.
- [assets/example-slide.json](assets/example-slide.json): a slide JSON showing common block types in `content`.
- [assets/example-outline.md](assets/example-outline.md): a markdown outline showing the import format so users can see what the importer expects.
