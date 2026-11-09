---
name: pz-model-pipeline
description: >
  Procedural Blender 4.x → Project Zomboid B42 asset pipeline for item models:
  mesh building with bmesh, texture-atlas UV layout, texture/logo generation, FBX export
  settings (scale/axes), in-hand vs world models, and rendering 2D inventory icons.
  Use when writing or running tools/build_soundbokks.py or anything touching
  models_X/, textures/, or icons. Pair with blender-bpy-automation and
  blender-render-output for generic bpy patterns.
---

# PZ model pipeline (Blender 4.x → B42)

## First: measure the reference, don't assume

Before exporting anything, inspect New Music's own boombox FBX and model script:

```
<NM>/42/media/models_X/*.fbx       (or models/; note which)
<NM>/42/media/scripts/*.txt        → model { mesh=, texture=, scale=, attachment ... }
<NM>/42/media/textures/*.png       → texture size, single atlas or per-part
```

Import the boombox FBX into Blender headlessly and print its bounding box, axis
orientation, unit scale, material/UV count and tri count:
```bash
blender --background --python tools/inspect_fbx.py -- "<path to boombox.fbx>"
```
Match **those** numbers. Everything below about scale/axes is the community default and
is only a fallback when the reference can't be read.

## Export defaults (fallback)

- Apply all transforms before export; origin at the base centre (feet on the floor).
- Vanilla items are roughly **real-world metres × 1.0 in-engine**, and the community
  guide exports Blender FBX at **`global_scale=0.01`** (a 1 m object otherwise shows 100×
  too large). Verify against the reference: if the NM boombox FBX imports at ~0.4 m
  tall with scale 1, mirror its settings.
- Axes: `axis_forward='-Z'`, `axis_up='Y'`. `apply_unit_scale=True`,
  `bake_space_transform=True` (so axis conversion is baked into the mesh), `use_mesh_modifiers=True`,
  `add_leaf_bones=False`, `object_types={'MESH'}`, `path_mode='STRIP'`, `use_selection=True`.
- One mesh, one material, one UV map, triangulated (`bmesh.ops.triangulate` or
  triangulate modifier). Keep ≤3k tris. No n-gons. Name the mesh object `Soundbokks`.
- Export the model once; colour variants are **texture swaps** via separate
  `model {}` blocks in the script, not separate FBX files.
- In-hand vs world: B42 uses the same mesh for `WorldStaticModel` and
  `ReplaceInPrimaryHand`; grip is positioned by `attachment Bip01_Prop1 {offset, rotate,
  scale}` in the model script, tuned in-game with the Attachment Editor. Only export a
  separate hand FBX if NM does.

## Atlas layout (single 1024×1024 PNG, or 512 if NM uses 512)

Keep each region axis-aligned so textures can be regenerated or hand-edited in place:

| Region | UV rect (u0,v0,u1,v1) | Content |
|---|---|---|
| body | 0.00–0.50, 0.50–1.00 | matte colour, subtle panel lines, screws |
| grille | 0.50–1.00, 0.50–1.00 | dark hex-perforated pattern, high contrast vs body |
| bumpers/handle | 0.00–0.50, 0.25–0.50 | black rubber, slight noise |
| back panel/battery | 0.50–1.00, 0.25–0.50 | control strip, battery slot, LEDs |
| **logo decal** | 0.00–0.50, 0.00–0.25 | wordmark on transparent/flat bg — swappable |
| spare | 0.50–1.00, 0.00–0.25 | reserved |

The logo decal is applied to two small quads (front frame bottom-centre, top panel)
whose UVs point only at the logo region, so swapping `tools/logo.png` and re-running
the texture step never touches geometry. Light logo on dark bodies, dark on White.

Generate textures with Pillow (`python -m pip install pillow`, or use Blender's bundled
Python: `blender --background --python-expr "import ensurepip; ensurepip.bootstrap()"`
then `<blender>/4.x/python/bin/python -m pip install pillow`). Fallback: build images
with `bpy.data.images.new` + pixel arrays (slow but dependency-free).

Isometric readability rules: strong value contrast body vs grille, bumpers darker than
body, no fine detail below ~4 px at 64 px icon size, slight AO baked into the atlas
(darken edges/recesses) because PZ lighting is flat.

## Inventory icons

- Vanilla icons: PNG, 8-bit, **32×32** (64×64 accepted). File `Item_<Icon>.png` in
  `42/media/textures/`, script `Icon = <Icon>`. Check the NM boombox icon's exact size
  and angle and match it.
- Render with EEVEE, transparent film (`scene.render.film_transparent = True`),
  orthographic camera at the PZ isometric angle (~45° yaw, ~30–35° pitch down; measure
  from NM's icon), 3-point soft lighting, render at 256×256 then downscale to the target
  with LANCZOS + light unsharp for crisp pixels. One render per colour by swapping the
  material's image.
- Also render a 512×512 hero shot per colour for `poster.png` (256×256 final) and the
  README.

## Script structure (`tools/build_soundbokks.py`)

Run: `blender --background --python tools/build_soundbokks.py -- --out Soundbokks/42/media --colors all`
Steps, each idempotent and selectable by flag: `mesh` → `uv` → `textures` → `export` →
`icons`. Print the output paths and the tri count at the end. Keep all dimensions in a
`SPEC` dict at the top (height/width/depth, bumper size, grille inset, logo rect) so the
user can tweak by hand. Blender 4.x API notes: `bpy.ops.export_scene.fbx` keyword names
are stable; `mesh.uv_layers.new()`; use `bmesh` for building, `bm.to_mesh()`; set
`mesh.use_auto_smooth` is **gone in 4.1+** (use `bpy.ops.object.shade_auto_smooth` or a
Smooth by Angle modifier, or just shade flat with bevelled edges — fine for this asset).

## Headless Blender on Windows

```
"C:\Program Files\Blender Foundation\Blender 4.x\blender.exe" --background --python <script> -- <args>
```
Everything after `--` is in `sys.argv[sys.argv.index('--')+1:]`. Exit code non-zero on
exception only if you `raise`; wrap main in try/except and `sys.exit(1)`.
