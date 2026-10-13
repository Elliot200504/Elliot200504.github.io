# Pre-rendered set: style guide

**Parked.** The set is left out of `assets/asset_sets.json` until it has proper models, so players only see Classic.
To switch it back on, add this entry to the registry's `sets`:

    "prerendered": {
      "label": "Pre-rendered",
      "path": "assets/units_prerendered",
      "renderer": "atlas",
      "ground": "assets/units_prerendered/ground",
      "backdrop": "assets/units_prerendered/backdrop",
      "gridColor": "rgba(150, 210, 60, 0.75)",
      "scale": 1,
      "offset": [0, 0]
    }

The "Pre-rendered" graphics set: 3D models rendered offline and flattened into 2D sprites, the way Heroes of Might
and Magic III's battle art was made. It sits beside the classic pixel sprites and never replaces them; the player
switches between the two in Settings ("Unit graphics") or with F9. F8 shows what each stack is drawn with.

Everything in this folder is written by the scripts in `scripts/prerender/`. Don't edit the PNGs by hand: change the
model or the script and render again.

## Rendering

    blender --background --factory-startup --python scripts/prerender/render_units.py -- --units pikeman --samples 32
    blender --background --factory-startup --python scripts/prerender/render_ground.py -- --factions castle
    blender --background --factory-startup --python scripts/prerender/render_backdrop.py -- --factions castle

- Blender 4.2 LTS or later, Cycles. A GPU is used when one is found; `--cpu` forces the CPU.
- `render_units.py` renders units and the field's props (trees, rocks and the like). Without `--units`, every
  reference model is rendered.
- With `--blend units.blend`, units come from that file: one collection per unit, named with the game's unit id,
  with actions named after the animations. Give a two-hex unit's collection a custom property `wide = 1`.
- `--smooth` keeps the full-resolution renders instead of flattening them.
- Both scripts refuse to write anywhere outside this folder.

## The look

| | |
|---|---|
| Camera | Orthographic, nearly side-on: 10° above the horizontal, as HoMM3 looks at its troops. Fixed for every unit and prop. |
| Facing | Every unit faces right (+X). Troops on foot turn 40° toward the viewer so their front shows; riders stand nearly in profile, 12°. |
| Scale | One hex is one Blender unit wide and a soldier stands about 1.05 units tall. Each model has a `size` it is drawn at in game: foot soldiers 2 (about two hexes tall, as in HoMM3), the cavalier 1.5, rocks 2 and logs 2.4. |
| Flattening | HoMM3's own sprite density: a model of `size` is rendered at `size` × 132 px a hex and averaged down to `size` × 44 px a hex, so a soldier is about 90 px tall, as HoMM3's are. Hard edges, one 96-colour palette per unit shared by all its frames, the silhouette's edge pixels darkened, no outline. Drawn with hard pixels, never smoothed. |
| Key light | Warm sun (1.0, 0.95, 0.86), strength 6.5, high and from the viewer's left, sharp (3°). It casts the only shadow. |
| Rim light | Cool sun from behind on the right, strength 2.4, so dark armour keeps its edge. |
| Fill | A sky dome, blue-grey above and earth-brown below, only 0.45: shadow sides go nearly black, for HoMM3's high contrast. |
| Shadow | The sun's shadow alone, on a shadow catcher, flat `#0c0a16` at 45%, in its own layer. |
| Materials | Steel is dark and shiny, so it shows black with bright glints as HoMM3's armour does; cloth and wood are matte. Steel `#a8b0bc`, dark steel `#58606c`, gold `#e8b048`, skin `#d8a07c`, white cloth `#e8e0cc`, wood `#7a4a22`, white horse `#e4e2dc`, stone `#82868c`, bark `#5a4028`. |

Bodies are one smooth surface each: Blender's Skin modifier wrapped round landmarks on the rig (hips, chest,
shoulders, knees, hooves), so limbs and torsos join the way HoMM3's do. Proportions follow HoMM3's Castle troops:
a broad, heroic build with thick limbs, big hands and boots. The team's colour is the cloth
(doublet, tabard, caparison bands, pennant, plume streak), set against white cloth and a lot of steel, never the
whole figure.

## Team colour

Parts that carry the owner's colour use a material whose name starts with `TeamColor`. They come out near-black in
the sprite, and in `teamcolor_mask/` in four flat shades of grey as they are lit, lifted toward white so the team's
colour reads vivid. In game the team's colour is laid
over the sprite with the mask as its opacity.

## Animations

| Name | Frames | FPS | Loops | Notes |
|---|---|---|---|---|
| `idle` | 12 | 8 | yes | A slight stir. As in HoMM3, a stack holds its first frame and runs through the idle once every five to ten seconds, each on its own rhythm. |
| `walk` | 8 | 12 | yes | Used for walking and flying. |
| `attack` | 8 | 12 | no | Four parts of two frames each: `ready` 0–1, `windUp` 2–3, `strike` 4–5, `follow` 6–7. |
| `hit` | 4 | 12 | no | Recoil from a blow. Optional. |
| `turn` | 6 | 16 | no | Swings round through facing the viewer to face left. Played backwards to turn right again. |
| `death` | 8 | 14 | no | Falls and lies dead. The last frame stays on the field as the body, in place of the classic remains. Optional. |

The battle decides how long each part of an attack lasts, exactly as with the classic sprites: a part plays at the
set's pace and holds its last frame until the battle moves on. A stack turns to face where it walks or strikes, and
back toward the enemy once its move is over. None of the classic sprites' CSS motion (breathing, leaning, staggering,
toppling) is laid over pre-rendered units; they move only as rendered.

A unit missing `idle`, `walk` or `attack` is drawn classic for that animation, and a warning is logged once. Without
`hit` it stands; without `turn` it flips at once; without `death` it fades into its classic remains.

## Props and ground

The field's one-hex obstacles are rocks and logs, whatever the homeland. Each is a prop: a model rendered in one
still `idle` frame, in its own folder like a unit. Rock piles and cliffs are drawn as a rock on each of their
hexes and fallen trees as logs, each a little bigger or smaller and some turned round; ponds are still drawn classic.

A hero watching beside the field is `hero`: the cavalier's horse and knight holding the army's banner, one still frame.

`backdrop/<homeland>.png` is the land beyond the field behind each army: rolling hills in that homeland's ground with
pines on them and hazy mountains behind, looking level so the horizon sits low, where the battle shows it.

`gridColor` in `asset_sets.json` sets the hex lines while the set is active: HoMM3's thin lime.

`ground/<homeland>.png` is a seamless 256 px tile of each homeland's ground, about six hexes across, rendered top-down
under the same sun and flattened the same way. It tiles without a seam because its noise is sampled on a torus.
Each half of the field is tiled with its army's homeland, blending in the middle.

## Files per unit or prop

    <id>/
      manifest.json
      atlas.png                          every frame, one row per animation
      <animation>/frame_####.png         the same frames, one file each
      shadow/atlas.png, shadow/<animation>/frame_####.png
      teamcolor_mask/atlas.png, teamcolor_mask/<animation>/frame_####.png
    index.json                           the ids this set has; others are drawn classic without a request
    ground/<homeland>.png

All three atlases share one layout. `manifest.json`:

- `frameSize`: every cell's size in atlas pixels.
- `anchor`: the feet-centre inside a cell. It sits on the same point as the classic sprite's bottom centre.
- `pixelsPerHex`, `scale`: the drawn size. `pixelated`: drawn with hard pixel edges.
- `animations`: each animation's `fps`, `loop`, the top-left corner of each frame's cell, and the attack's `phases`.

## Moving units over

Units can be added one at a time. Render a unit and it goes into `index.json`; the next time the set is active it is
drawn pre-rendered, and every other unit stays classic. Sizes are lined up through `scale` and `offset` in
`assets/asset_sets.json` (the whole set) or `scale` in a manifest, never by changing the classic sprites.
