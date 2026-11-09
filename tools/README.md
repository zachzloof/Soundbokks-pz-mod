# tools/

Asset tooling (Stage d). Planned contents:

- `inspect_fbx.py` — headless Blender: import New Music's boombox FBX and print bbox,
  axes, scale, tri count, UV/material count. Run first; the build script matches it.
- `gen_logo.py` → `logo.svg`, `logo.png` — original "SOUNDBOKKS" wordmark (light + dark).
- `build_soundbokks.py` — headless Blender 4.x: mesh → UV atlas → textures (6 colours,
  swappable logo decal region) → FBX → inventory icons → poster render.

Run: `blender --background --python tools/build_soundbokks.py -- --out Soundbokks/42/media`
