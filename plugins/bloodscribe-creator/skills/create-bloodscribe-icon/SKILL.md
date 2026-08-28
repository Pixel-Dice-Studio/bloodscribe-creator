---
name: create-bloodscribe-icon
description: Create or refine original character icons for BloodScribe packs using named visual styles and independent detail levels. Use when the user explicitly asks for character artwork; not for cover art or character mechanics.
---

# Create a BloodScribe icon

Use the user's available image-generation tool. Reply in the user's language.

Icon generation is optional and separate from character mechanics. Do not create artwork unless the user requests it or the target pack requires it; an image failure must not block a valid character proposal.

## Workflow

1. Before generating, ask how the user wants the result delivered unless they already specified it: **separate pack-ready SVGs**, **one PNG comparison sheet for review**, or **both**. Ask in the user's language; for example: “¿Cómo quieres recibir el resultado: SVG separados listos para el pack, una lámina comparativa para revisar, o ambos?”. Do not generate until this choice is known.
2. Identify the character, one dominant narrative symbol, the pack's team color, and any explicit constraints. Keep story objects in their natural colors; reserve the team color for clothing, trim, frame, or one secondary mass.
3. Choose a `style` and `detail` level from [references/styles.md](references/styles.md). Treat them as independent axes. If the user names neither, use `icono-representativo` with `medio`; for Grimm or requests matching the existing Grimm collection, use `cuento-infantil` with `medio`.
4. Load only the selected style board linked from `references/styles.md`. Use it to calibrate medium, shape language, outline, density, and finish; never copy one of its characters into a different character. Inspect `assets/detail-levels-style-board.png` only when the requested detail level is ambiguous.
5. Generate one centered character or object per 1254 x 1254 square PNG proposal with a genuinely transparent exterior and generous padding. For a comparison sheet, arrange the requested characters or variants in a clear review grid; the sheet is a preview, never the icon stored in a pack. For `both`, derive the sheet from the separate proposals. Do not add text, scenery, frames, logos, watermarks, or extra characters to individual icons unless the selected style explicitly requires a physical frame such as a Polaroid or medallion.
6. Review every icon at full size and as a 64 px thumbnail. The subject and narrative symbol must remain recognizable; increasing detail must not change the silhouette, pose, identity, or realism level.
7. Verify each separate icon is RGBA, alpha 0 in all four corners, with no baked checkerboard, opaque exterior rectangle, or pale halo on white or near-black backgrounds. Retry once if transparency fails; do not fake transparency by adding an unused alpha channel.
8. Save proposals without overwriting existing art. Do not vectorize or replace pack icons until the user approves the PNG.
9. When a separate or `both` result is requested and the PNG is approved, use BloodScribe Creator's existing conversion flow, compare the SVG visually with the PNG, and return each standalone `.svg`. When the icon belongs to a character or tale workflow, also place the exact complete SVG markup in `character.icon` and revalidate the changed proposal or pack. Never replace it with an external URL.

## Boundaries

- Use original folklore and genre language; do not imitate a franchise, artist, or another game's individual designs.
- Do not infer a character's rules, alignment, or team from its appearance.
- `polaroid-realista` remains raster-first. If a pack-ready SVG is requested, retain the approved PNG as the visual reference and use BloodScribe Creator's real vector conversion; never embed raster data inside the SVG because pack icon security rejects `<image>` elements.
- An object character remains an object unless the user asks for anthropomorphism.
