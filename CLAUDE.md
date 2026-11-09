# Soundbokks — Project Zomboid B42 add-on for New Music [42+]

A loud, rugged party speaker (six colours) that plugs into **New Music [42+]** by
Talismon (Workshop 3739256725, mod id `NewMusic`). It behaves like NM's boombox but is
cassette-only, much louder, and pulls zombies from a sandbox-tunable range.

## Hard rules

- **Never edit New Music's files.** Hook/extend it from our own Lua; all overrides of NM
  functions live in ONE file: `Soundbokks/42/media/lua/client/Soundbokks_NewMusicHooks.lua`
  (server-side ones in the matching `server/` file). Log every successful hook once.
- **Investigate before writing code.** Read NM's Lua + scripts and the vanilla files
  first; the plan in `PLAN.md` is approved stage by stage by the user. Don't start a
  stage without the user's OK, and stop at the end of each stage with test instructions.
- **Verify syntax against real files**, not memory: vanilla `<PZ>/media/scripts`,
  `<PZ>/media/sandbox-options.txt`, `<PZ>/media/lua/shared/translate/en/*.json`,
  `ProceduralDistributions.lua`. B42 ≠ B41 (see `.claude/skills/pz-b42-modding`).
- Keep the mod **multiplayer-safe**: client → `sendClientCommand` → server validates.
  Reuse NM's own sync for playback instead of inventing a second one.
- Don't copy the real Soundboks logo/typeface. Our wordmark is original ("SOUNDBOKKS").
- Don't commit unless asked. Never add attribution lines to commits.

## Layout

```
Soundbokks/                 ← the mod (what gets symlinked/copied to ~/Zomboid/mods/Soundbokks)
  mod.info  poster.png  icon.png
  42/media/{scripts,sandbox-options.txt,lua/{client,server,shared},models_X,textures}
tools/                      ← build_soundbokks.py (Blender), inspect_fbx.py, logo.svg/png, gen_logo.py
docs/reference/             ← vendored third-party reference (pz-modding-guide, MIT-ish; read-only)
.claude/skills/             ← pz-b42-modding, pz-model-pipeline, blender-bpy-automation, blender-render-output
PLAN.md                     ← staged plan + current status (update the status table as stages land)
README.md                   ← user-facing: install, dependencies, load order, sandbox options
```

## Environment (dev PC)

- Game: `C:\Program Files (x86)\Steam\steamapps\common\ProjectZomboid` (B42.19 stable, per console.txt)
- New Music: `C:\Program Files (x86)\Steam\steamapps\workshop\content\108600\3739256725\`
  (inside: `mods/NewMusic/...`; confirm actual subfolder and where its `mod.info` lives)
- User data: `C:\Users\<you>\Zomboid\` — `mods/`, `console.txt`, `Sandbox Presets/`
- Install for testing: junction `~/Zomboid/mods/Soundbokks` → `<repo>/Soundbokks`
  (`cmd /c mklink /J "%USERPROFILE%\Zomboid\mods\Soundbokks" "<repo>\Soundbokks"`)
- Blender 4.x headless: `blender --background --python tools/build_soundbokks.py -- ...`
- Launch option `-debug` for the item spawner, Attachment Editor and `/reloadlua`.

## Conventions

- Module `Soundbokks`; items `Soundbokks.Soundbokks<Colour>` (Black, White, Red, Blue,
  Green, Orange). One mesh `Soundbokks.fbx`, texture `Soundbokks_<Colour>.png`,
  icon `Item_Soundbokks<Colour>.png`.
- Sandbox page `Soundbokks`: `AttractionRange` (multiplier vs boombox, default 3.0),
  `VolumeRange` (audible distance multiplier), `LootMultiplier` (default 1.0).
- Lua files prefixed `Soundbokks_`; all `print` lines prefixed `[Soundbokks]`.
- Translations: JSON in `42/media/lua/shared/translate/en/` (42.15+ format).

## Testing a stage

Enable `-debug`, new sandbox save with NewMusic + Soundbokks, spawn via debug Item List,
then `grep -n "Soundbokks\|ERROR" "%USERPROFILE%\Zomboid\console.txt" | tail -60`.
Each stage ends with exact test steps written for the user.
