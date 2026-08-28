# Visual styles and detail

`style` controls the medium and shape language. `detail` controls how much internal information appears. Never use detail as a synonym for realism.

Every style board uses the same five references, left to right: Blancanieves, Enano Gruñón, Cazador de Blancanieves, Espejo Encantado, and Reina Madrastra. Use the boards to learn treatment, never as subject templates.

## Styles

### `cuento-infantil`

The established Grimm style. Read [the reference board](../assets/cuento-infantil-style-board.png) before generating.

- Rounded, schematic folklore character or object with a friendly editorial finish.
- Heavy, smooth black exterior and important interior contours.
- Large clean color masses with restrained soft gradients; no painted texture or scenery.
- Expressive face built from very few marks and a dominant story object occupying roughly 30-45 percent of the composition.
- Team color on clothing or the main support mass: green for Pueblerino, olive for Ayudante, ochre for Errante, red for Siervo, dark brown for Oscuro. Natural story colors still win.

### `icono-representativo`

An original tabletop emblem: portrait and story object interlock into a compact near-circular silhouette. Use flat colors, strong negative space, a uniform heavy outline, and no texture, scene, or decorative frame. It may share the instant readability of social-deduction tokens but must not reproduce another game's compositions or characters.

Reference: [icono-representativo-style-board.png](../assets/icono-representativo-style-board.png).

### `polaroid-realista`

A plausible photographed character or object inside a worn instant-photo frame. Use natural adult anatomy, materials, window light, muted analog color, and a simple background. No fantasy cosplay polish. This preset is raster-first and defaults to `detallado`.

Reference: [polaroid-realista-style-board.png](../assets/polaroid-realista-style-board.png).

### `grabado-sencillo`

A clean linocut or woodcut made from bold black shapes, cream negative space, and one muted team-color ink. Use broad carved curves and sparse cut marks, never dense crosshatching or tiny noise.

Reference: [grabado-sencillo-style-board.png](../assets/grabado-sencillo-style-board.png).

### `clasico-sobrio`

A restrained nineteenth-century editorial portrait or object still life in a circular medallion. Use muted oil color, controlled soft brushwork, a dark neutral field, and one thin antique-gold rim. Avoid baroque decoration and fantasy glow.

Reference: [clasico-sobrio-style-board.png](../assets/clasico-sobrio-style-board.png).

### `vidriera-narrativa`

Build the character from jewel-toned translucent glass cells separated by dark lead lines. Keep the silhouette compact and the cell count controlled; use glow only to reveal the material, never as a background effect.

Reference: [vidriera-narrativa-style-board.png](../assets/vidriera-narrativa-style-board.png).

### `papel-recortado`

Construct the icon from separate layers of hand-cut colored paper with irregular cut edges, visible fibers, overlapping planes, and shallow shadows between local pieces. Do not use a black ink contour, white die-cut border, sticker silhouette, glossy vector finish, or the geometry of an existing icon as a base. Avoid a full diorama or background scene.

Reference: [papel-recortado-style-board.png](../assets/papel-recortado-style-board.png).

### `esmalte-cloisonne`

Design the subject from scratch as a physical cloisonne plaque or brooch. Antique-brass wires must define every major color compartment, with opaque or translucent jewel-toned enamel pooled inside them and controlled highlights that reveal both glass and metal. Do not add a gold outline around an existing icon, use a thick black cartoon contour, or produce a sticker-like vector rendering.

Reference: [esmalte-cloisonne-style-board.png](../assets/esmalte-cloisonne-style-board.png).

### `talla-madera-pintada`

Render a shallow hand-carved wooden relief with visible carved planes and grain, restrained folk paint, slight edge wear, and enough dark separation to remain readable as a token.

Reference: [talla-madera-pintada-style-board.png](../assets/talla-madera-pintada-style-board.png).

### `manuscrito-iluminado`

Use medieval illuminated-manuscript language: flat perspective, ink contours, parchment colors, compact poses, and restrained gold-leaf accents. Do not add page borders, letters, or decorative marginalia.

Reference: [manuscrito-iluminado-style-board.png](../assets/manuscrito-iluminado-style-board.png).

### `teatro-sombras`

Use dominant cut-paper black silhouettes, one restrained team-color layer, and a few meaningful interior cutouts. Depend on negative space rather than facial rendering; do not add a stage or scenery.

Reference: [teatro-sombras-style-board.png](../assets/teatro-sombras-style-board.png).

## Detail levels

When visual calibration is useful, inspect [detail-levels-style-board.png](../assets/detail-levels-style-board.png): it shows the same character at `bajo`, `medio`, and `detallado` from left to right.

### `bajo`

- About 5-7 large masses.
- Only the exterior contour and essential face or object cuts.
- No material texture.
- Best for 48-64 px use and clean SVG conversion.

### `medio`

- About 10-14 masses.
- A few garment folds, grouped hair or beard shapes, and the important object fittings.
- Default for every style except `polaroid-realista`.

### `detallado`

- About 18-24 controlled masses or equivalent photographic information.
- More separated hair, stitching, material cues, or restrained value changes.
- No tiny noise; the same silhouette must still read at 64 px.

## Recommended defaults

| Style | Detail |
|---|---|
| `cuento-infantil` | `medio` |
| `icono-representativo` | `medio` |
| `polaroid-realista` | `detallado` |
| `grabado-sencillo` | `medio` |
| `clasico-sobrio` | `detallado` |
| `vidriera-narrativa` | `medio` |
| `papel-recortado` | `medio` |
| `esmalte-cloisonne` | `medio` |
| `talla-madera-pintada` | `medio` |
| `manuscrito-iluminado` | `medio` |
| `teatro-sombras` | `bajo` |
