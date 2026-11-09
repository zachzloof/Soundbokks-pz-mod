# Soundbokks — build plan

Status legend: ⬜ not started · 🔍 investigating · 🟨 in progress · ✅ done (tested in-game)

| Stage | What | Status |
|---|---|---|
| 0 | Investigate New Music + vanilla, write findings, get user OK | ⬜ |
| a | Skeleton + one Black Soundbokks using NM boombox model; plays cassettes | ⬜ |
| b | Loudness, zombie attraction range, sandbox options | ⬜ |
| c | Six colour variants, loot distribution, translations | ⬜ |
| d | Logo, Blender script, custom model + textures, icons, poster, README | ⬜ |

Update this table as stages land. Each stage ends with: files changed, exact in-game
test steps, what to grep in `console.txt`, and a stop for the user's OK.

---

## Stage 0 — Investigation (no code until this is approved)

Read, in this order, and write findings to `docs/investigation.md`:

1. `<NM>/.../mod.info` (id, require, versionMin, where it lives: root vs `42/`),
   folder tree (`42/`, `common/`, `models_X/` vs `models/`, `textures/`, translate format).
2. `<NM>/42/media/scripts/*.txt`: the boombox item block (ItemType, device/radio keys,
   `WorldStaticModel`, `ReplaceInPrimaryHand`, battery keys, tags), its `model {}` block
   (mesh/texture/scale/attachment), cassette item + media definitions.
3. NM Lua — map the architecture:
   - how devices are recognised (item type? tag? hard-coded list? `ModData`? a table like
     `NewMusic.Devices = {...}` we can register into?)
   - media/cassette validation (what makes an item "a cassette"; how to make our device
     accept cassettes but not CDs/vinyl)
   - playback: where the FMOD/sound emitter is created, how volume and audible distance are
     set, whether it calls `getWorldSoundManager():addSound(...)` (zombie attraction) and
     with what radius/volume, on what tick
   - carry/place/pickup transitions ("ground-to-hand"), UI (the boombox window)
   - multiplayer: `sendClientCommand`/`OnClientCommand`, `sendServerCommand`, ModData
     sync, "MP track seeking" — what the server owns vs client
   - sandbox options it exposes (file + how it reads them)
   - its loot distribution file (lists + weights) — mirror at lower weight
4. Vanilla: `sandbox-options.txt` syntax; `translate/en/itemname.json` + `sandbox.json`
   shape; `ProceduralDistributions.lua` list names (ElectronicStore*, Garage*, LivingRoom*,
   Party/house); vanilla radio/boombox script keys.
5. Decide: **register via NM data/API** (preferred) vs **wrap N functions** (fallback), and
   list exactly which functions need wrapping. Estimate NM-update fragility.
6. Report back with a short plan + risks; **wait for OK**.

## Stage a — Skeleton + placeholder Black Soundbokks

- `Soundbokks/mod.info` (+ `42/mod.info` if NM does), `poster.png` placeholder, `icon.png`.
- `42/media/scripts/Soundbokks_items.txt`: `Soundbokks.SoundbokksBlack` cloned from the NM
  boombox block, `WorldStaticModel`/hand models pointing at NM's boombox model name,
  NM boombox icon as placeholder.
- `42/media/lua/client/Soundbokks_NewMusicHooks.lua` (+ `shared/Soundbokks_Defs.lua` with the
  item list and `isSoundbokks(item)`): register the device with NM or wrap the minimum.
- Cassette-only: hook NM's media-accept check.
- Test: spawn item, insert cassette, play; pick up/put down while playing; batteries drain.
  Console: `[Soundbokks] hooked ...` lines, no ERROR.

## Stage b — Loudness + zombie range + sandbox

- `42/media/sandbox-options.txt` + `translate/en/sandbox.json`: `AttractionRange` (default 3.0×
  boombox), `VolumeRange` (audible distance ×), `LootMultiplier` (used in c).
- Hook the place where NM emits the world sound / sets emitter distance; multiply for
  Soundbokks items. If NM never calls `addSound`, add our own `EveryOneMinute`/tick
  emitter (server-side in MP) with the sandbox radius.
- Test: sandbox option visible under page "Soundbokks"; zombies spawned at e.g. 60 tiles
  react at Soundbokks, not at the boombox; music audible further away. Grep radius log line.

## Stage c — Colours, loot, translations

- Five more items sharing the same block (texture swap only at this stage via NM model),
  `translate/en/itemname.json` ("Soundbokks (Red)" …), `tooltip` if NM uses one.
- `lua/server/Soundbokks_Distributions.lua` on `OnPreDistributionMerge`, weights ×
  `LootMultiplier`; lists: electronics stores, garages, living-room/party loot (names
  verified against vanilla + NM's own file).
- Test: all six in Item List with correct names; loot: debug "Spawn loot" or visit an
  electronics store with `LootMultiplier` = 10.

## Stage d — Logo, Blender script, model, icons

- `tools/gen_logo.py` → `tools/logo.svg` + `logo.png` (original blocky wordmark).
- `tools/inspect_fbx.py` → measure NM boombox FBX (scale, axes, bbox, tris, UV).
- `tools/build_soundbokks.py` (Blender 4.x, headless): mesh → UV atlas → textures (6
  colours, logo decal region) → FBX export matching the reference → icons (6) → poster.
- Script `model {}` blocks per colour; swap items from the placeholder to `Soundbokks`
  mesh; tune `attachment Bip01_Prop1` with Attachment Editor; final `poster.png`, README.
- Test: world model on floor, in-hand model, icons in inventory, no texture/UV seams at
  isometric zoom levels; tri count printed ≤ 3k.

## Open questions (answer in Stage 0)

- Does NM expose a registration table/API for new devices? (decides hook vs copy)
- Does NM's boombox attract zombies at all today, and via which call?
- Does NM store playback state in item ModData, world-object ModData, or a Lua table?
- `models_X/` or `models/`? `mod.info` at root or `42/`? Texture size of NM boombox?
- Which cassette "media category" string does NM use?
