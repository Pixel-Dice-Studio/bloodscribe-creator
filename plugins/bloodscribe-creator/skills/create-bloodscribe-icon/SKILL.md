---
name: create-bloodscribe-icon
description: Create or refine original character icons for BloodScribe packs using named visual styles and independent detail levels. Use when the user explicitly asks for character artwork; not for cover art or character mechanics.
---

# Create a BloodScribe icon

Use the user's available image-generation tool. Reply in the user's language.

Icon generation is optional and separate from character mechanics. Do not create artwork unless the user requests it or the target pack requires it; an image failure must not block a valid character proposal.

## Workflow

1. Always deliver one separate pack-ready SVG per character. Do not ask the user to choose an output format and do not offer PNG comparison sheets; PNG proposals are review intermediates only.
2. Identify the character, one dominant narrative symbol, the pack's character-type color, and any explicit constraints. Keep story objects in their natural colors; reserve that color for clothing, trim, frame, or one secondary mass.
3. Require three explicit, independent choices before generating: `style`, `detail`, and `composition`. Choose `style` and `detail` from [references/styles.md](references/styles.md). The exact composition values are `personaje-con-objeto` and `objeto-solo`. If `style` is missing, inspect and show `assets/style-selector-board.png`, whose labels are the exact accepted names. If `detail` is missing, inspect and show `assets/detail-selector-board.png`. If `composition` is missing, ask: “¿Quieres personaje con objeto u objeto solo?”. Ask for every missing choice in one message and do not generate until all three are explicit. You may recommend an option when asked, but never infer or apply a silent default.
4. Load only the full reference board for the selected style and `assets/detail-levels-style-board.png`. Use them to calibrate medium, shape language, outline, density, and finish; never copy one of their characters into a different character. A style board's white or gray inspection canvas is never part of the style or the final icon.
5. Generate one centered subject per square PNG proposal, targeting 1254 x 1254 with generous padding. For `personaje-con-objeto`, crop the character to head and shoulders and interlock the dominant narrative object beside it. For `objeto-solo`, omit the character and let one recognizable object fill the composition; do not add a face, body, or hands unless the user explicitly requests anthropomorphism. Each PNG is only a review intermediate for its corresponding SVG: prefer real transparency, but accept a clean removable background or a different raster size when the existing converter can crop and normalize it without cutting the artwork. Compose the visible subject as a compact, almost square mass: its occupied bounding box should normally be at least 75% as wide as it is tall. Bend or place long objects diagonally; do not use tall full-body poses, narrow portrait compositions, or stack the symbol underneath. Do not add text, scenery, frames, logos, watermarks, or extra characters unless the selected style explicitly requires a physical frame such as a Polaroid or medallion.
6. Review every icon at full size and as a 64 px thumbnail. The subject and narrative symbol must remain recognizable, and the composition must still feel balanced when clipped into a circle. Increasing detail must not change the silhouette, pose, identity, or realism level.
7. Inspect whether transparency is real, but do not spend another image-generation attempt solely to obtain alpha or exact pixel dimensions. An RGB image, baked checkerboard, or opaque exterior is acceptable when the subject remains cleanly separable by color. Reject or regenerate only when the background merges with the subject or leaves a halo that the converter cannot remove safely.
8. Save proposals without overwriting existing art. Do not vectorize or replace pack icons until the user approves the PNG.
9. After each PNG is approved, use BloodScribe Creator's existing conversion flow. If the PNG lacks usable alpha, enable background removal: start with edge detection, then add samples for every remaining area or color and adjust tolerance as needed. The converter supports up to 16 samples, multiple disconnected regions, square cropping, and size normalization. Compare the background-free SVG with the PNG on light and near-black surfaces, then return each standalone `.svg`. When the icon belongs to a character or tale workflow, also place the exact complete SVG markup in `character.icon` and revalidate the changed proposal or pack. Never replace it with an external URL.

## Boundaries

- Use original folklore and genre language; do not imitate a franchise, artist, or another game's individual designs.
- Do not infer a character's rules, alignment, or character type from its appearance.
- `polaroid-realista` remains raster-first. If a pack-ready SVG is requested, retain the approved PNG as the visual reference and use BloodScribe Creator's real vector conversion; never embed raster data inside the SVG because pack icon security rejects `<image>` elements.
- An object character remains an object unless the user asks for anthropomorphism.
