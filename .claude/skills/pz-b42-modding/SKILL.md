---
name: pz-b42-modding
description: >
  Project Zomboid Build 42 (42.15+) modding reference: mod folder layout, mod.info,
  item/model scripts, Lua client/server/shared split, multiplayer command pattern,
  sandbox options, JSON translations, loot distributions, zombie-attracting sounds,
  and the debug/test loop. Use for any file under Soundbokks/ or when reading
  New Music's Lua/scripts. Also says what MUST be verified against real game files.
---

# Project Zomboid Build 42 modding

Target: **B42.19+ stable**. Most online tutorials are Build 41 and will silently break
you. Three big B41→B42 breaks: versioned folders (`42/media/` not `media/`),
`ItemType = base:normal` (not `Type = Normal`), and `craftRecipe {}` (not `recipe {}`).
Longer reference pages are vendored in `docs/reference/pz-modding-guide/` (read
`gotchas.md`, `visual-assets.md`, `multiplayer.md` when relevant).

## Ground truth beats this file

Vanilla game files are the authority. When syntax matters, open the real file:

| Need | Where (game install = `<PZ>` = `.../steamapps/common/ProjectZomboid`) |
|---|---|
| Item script keys | `<PZ>/media/scripts/` (B42 uses `media/scripts/generated/` for many) — grep `Radio`, `WorldStaticModel`, `IsPortable`, `TwoWay` |
| Model script keys | grep `^model ` / `mesh =` in `<PZ>/media/scripts/` |
| Sandbox option syntax | `<PZ>/media/sandbox-options.txt` and any mod's `42/media/sandbox-options.txt` |
| Translations (JSON) | `<PZ>/media/lua/shared/translate/en/*.json` (`itemname.json`, `sandbox.json`, `ui.json`, `tooltip.json`) |
| Loot tables | `<PZ>/media/lua/server/Items/ProceduralDistributions.lua`, `Distributions.lua`, `ItemPickerJava` |
| Sound/zombie attraction | `<PZ>/media/lua/` grep `addSound`, `getWorldSoundManager`; Java: `zombie.WorldSoundManager` |
| Events list | `<PZ>/media/lua/` grep `Events\.On`; stubs: github.com/demiurgeQuantified/PZEventStubs |
| Java API | Unofficial B42 JavaDocs: https://pzwijava.bountiful-games.com/b42/ |

New Music (`<Steam>/steamapps/workshop/content/108600/3739256725/`) is the second
authority for everything device/music related. Read it before writing a line.

## Folder layout (B42, local dev)

```
~/Zomboid/mods/Soundbokks/            ← folder name = mod id
├── mod.info                           ← some mods put it here, some in 42/; check NewMusic
├── poster.png                         ← 256×256
└── 42/
    ├── mod.info                       ← REQUIRED location per 42.x guides
    ├── poster.png
    └── media/
        ├── scripts/                   ← *.txt item/model scripts
        ├── sandbox-options.txt
        ├── lua/{client,server,shared}/
        ├── lua/shared/translate/en/   ← itemname.json, sandbox.json, ui.json (42.15+)
        ├── models_X/  or models/      ← .fbx (check which folder NewMusic uses)
        └── textures/                  ← Item_*.png icons, model textures
```

`common/` is loaded by both builds; for a B42-only mod use `42/` only. Since 42.15
only one of `common/` or `42/` needs to exist. `mod.info` location differs between
guides (root vs `42/`) — **copy whatever New Music does**.

`mod.info`:
```
name=Soundbokks
id=Soundbokks
author=...
description=...
poster=poster.png
require=NewMusic
versionMin=42.0
```
Workshop packaging: `Workshop/Soundbokks/Contents/mods/Soundbokks/42/...` + `workshop.txt`.

## Item scripts (`42/media/scripts/*.txt`)

```
module Soundbokks
{
    imports { Base }

    item SoundbokksBlack
    {
        DisplayName      = Soundbokks (Black),   // overridden by itemname.json
        DisplayCategory  = Electronics,
        ItemType         = base:normal,          // CHECK: NewMusic boombox may use base:radio or a device type
        Weight           = 8.0,
        Icon             = SoundbokksBlack,      // file: textures/Item_SoundbokksBlack.png (Item_ prefix on file only)
        WorldStaticModel = SoundbokksBlack,      // model script name
        StaticModel      = SoundbokksBlack,
        ReplaceInPrimaryHand   = SoundbokksBlack holdingbagright,
        ReplaceInSecondHand    = SoundbokksBlack holdingbagleft,
        RequiresEquippedBothHands = true,
        Tags             = ...,
    }
}
```
A boombox in vanilla/NewMusic is a **radio-type device** with `IsPortable`, `TwoWay`,
`TransmitRange`, `BaseVolumeRange`, `UsesBattery`, `IsTelevision=false` and device data
(`DeviceData`). Copy the New Music boombox block verbatim and change only what's needed.
Item and model scripts are parsed at load: **changes need a full game restart**.

Model script (same `.txt` files):
```
model SoundbokksBlack
{
    mesh    = Soundbokks,                 // 42/media/models_X/Soundbokks.fbx
    texture = Soundbokks_Black,           // 42/media/textures/Soundbokks_Black.png
    scale   = 1.0,                        // verify key exists; NewMusic may set it
    attachment Bip01_Prop1 { offset = 0 0 0, rotate = 0 0 0, scale = 1.0, }
}
```
Six colour items can share one `mesh` with six `model {}` blocks differing only in
`texture`. Use the in-game Attachment Editor (debug) to tune `offset`/`rotate`.

## Lua

- `lua/client/` runs on each client, `lua/server/` on server/host, `lua/shared/` both.
  Host of a listen server runs both. All `.lua` auto-load; order is alphabetical per
  folder, mods in load order. `require("path/File")` for explicit dependencies.
- Hook, don't copy: keep every override of a New Music function in **one** file
  (`lua/client/Soundbokks_NewMusicHooks.lua`) using the wrap pattern:
  ```lua
  local orig = SomeNMTable.someFunc
  function SomeNMTable.someFunc(self, ...)
      if not isSoundbokks(self.item) then return orig(self, ...) end
      -- custom behaviour, or call orig then adjust
  end
  ```
  Wrap after NM has defined the table: do it inside `Events.OnGameBoot.Add` or at file
  top if `require` guarantees order. Check nil before wrapping and log once.
- Prefer a data-table registration if New Music exposes one (device list, media type
  table, model table). Investigate first; the whole plan depends on this.
- Sandbox values: `SandboxVars.Soundbokks.AttractionRange` (section = option prefix).
  Read at use time, not file load time (SandboxVars may not exist at boot).
- Multiplayer: client detects → `sendClientCommand(player, "Soundbokks", "cmd", args)`
  → server `Events.OnClientCommand` validates and acts → engine syncs. Server→client:
  `sendServerCommand(...)` / `Events.OnServerCommand`. Never mutate shared state on a
  client only. Use `isClient()`, `isServer()`, `isMultiplayer()`. New Music already
  handles playback sync — find how (ModData? commands? `syncItemFields`?) and reuse it.
- Zombie attraction: `getWorldSoundManager():addSound(source, x, y, z, radius, volume)`
  (Java `zombie.WorldSoundManager`; overloads add `stressHumans`, `zombieIgnoreDist`,
  `stressMod`). Find where New Music calls it (or vanilla radio code does) and scale the
  radius/volume by the sandbox multiplier for Soundbokks items. Audible-distance for the
  player is separate: FMOD emitter / `BaseVolumeRange` / NM's own distance attenuation.
- Debug log: `print()` → `~/Zomboid/console.txt`. Errors show as `ERROR: ... Callframe`.
  Prefix all log lines with `[Soundbokks]` so `grep Soundbokks console.txt` works.

## Sandbox options

File: `42/media/sandbox-options.txt`. Pattern (verify against vanilla/NM file):
```
VERSION = 1,

option Soundbokks.AttractionRange
{
    type = double, min = 0.5, max = 10.0, default = 3.0,
    page = Soundbokks, translation = Soundbokks_AttractionRange,
}
option Soundbokks.LootMultiplier
{
    type = double, min = 0.0, max = 10.0, default = 1.0,
    page = Soundbokks, translation = Soundbokks_LootMultiplier,
}
```
Translations go in `translate/en/sandbox.json` as `Sandbox_Soundbokks_AttractionRange`
and `..._tooltip`. Adding a mod mid-save: new options take defaults.

## Translations (42.15+ JSON — confirmed on 42.19 log)

`42/media/lua/shared/translate/en/itemname.json` (UTF-8, lowercase paths):
```json
{
  "ItemName_Soundbokks.SoundbokksBlack": "Soundbokks (Black)",
  "ItemName_Soundbokks.SoundbokksRed": "Soundbokks (Red)"
}
```
Copy the exact top-level shape from a vanilla `itemname.json` first (it may be a flat
object or wrapped) — do not guess. Same for `sandbox.json` (`Sandbox_<page>` page title,
option labels, tooltips) and `ui.json`/`tooltip.json` for context-menu text.

## Loot distributions

`42/media/lua/server/Soundbokks_Distributions.lua`:
```lua
local function addLoot()
    local m = SandboxVars.Soundbokks and SandboxVars.Soundbokks.LootMultiplier or 1
    local function add(listName, item, weight)
        local list = ProceduralDistributions.list[listName]
        if not list then print("[Soundbokks] missing dist list " .. listName) return end
        table.insert(list.items, item); table.insert(list.items, weight * m)
    end
    for _, c in ipairs({"Black","White","Red","Blue","Green","Orange"}) do
        add("ElectronicStoreMusic", "Soundbokks.Soundbokks"..c, 0.3)   -- verify list names exist in B42
        add("GarageTools",          "Soundbokks.Soundbokks"..c, 0.05)
        add("LivingRoomShelf",      "Soundbokks.Soundbokks"..c, 0.05)
    end
end
Events.OnPreDistributionMerge.Add(addLoot)
```
Verify list names by grepping `ProceduralDistributions.lua`; see how New Music
distributes its own boombox/cassettes and mirror those lists at a lower weight.

## Debug / test loop

1. Steam → PZ → Properties → Launch options: `-debug`.
2. Mods menu → enable NewMusic + Soundbokks → new Sandbox game.
3. Debug menu (left-side bug icon / F11): **Item List / Spawn** → search `Soundbokks`.
   Right-click world → Debug → Attachment Editor for model tweaks.
4. Lua changes: `/reloadlua` in console or the debug "Reload Lua" button (not for
   scripts/models — restart).
5. Check `~/Zomboid/console.txt`: `grep -n "Soundbokks\|ERROR" console.txt | tail -50`.
   Script parse errors appear early in the log at mod load.
6. Zombie range test: spawn a zombie horde at N tiles (debug → zombie spawn) and play.

## Known gotchas

- `Icon = Foo` ↔ file `Item_Foo.png` (8-bit PNG, 32×32 or 64×64). No `Item_` in script.
- `mod.info` `id` must have no spaces and match the folder name.
- `TwoHandWeapon` is weapons-only; non-weapons use `RequiresEquippedBothHands = true`.
- Don't put `mod.info` in both root and `42/` unless NM does.
- Mods using the old `ItemName_EN.txt` translation format are broken on 42.15+.
- 42.16.0 had a mod-loading regression; test on ≥42.16.1 (user is on 42.19).
