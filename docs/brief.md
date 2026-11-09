# Original brief (verbatim from the user, 2026-11-06)

# Project: "Soundbokks" — a loud portable speaker add-on for New Music [42+] (Project Zomboid)

## Context
- Game: Project Zomboid, Build 42 (stable). Single player, and it should also work in multiplayer.
- Base mod this depends on: "New Music [42+]" by Talismon
  - Workshop ID: 3739256725, Mod ID: NewMusic
  - Local files: <Steam>/steamapps/workshop/content/108600/3739256725/
- My mod must be a separate add-on that loads after New Music (`require=NewMusic` in mod.info).
  Never edit New Music's own files.

## What I want
A new music device called "Soundbokks": a big, rugged, battery-powered party speaker.
1. It works like New Music's existing boombox (same UI, same cassette playback, batteries,
   carry/place behaviour), but it only takes cassettes.
2. It is much louder than the boombox and attracts zombies from much further away.
   - Put the sound radius and volume in a sandbox option (e.g. "Soundbokks zombie
     attraction range", default about 3x the boombox), so I can tune it.
   - The music should also be audible to the player from further away.
3. Several colour variants: Black, White, Red, Blue, Green, Orange (each one a separate item
   sharing one model, with a different texture per colour).
4. Loot: spawns rarely, in places that make sense (electronics stores, garages, party/house
   loot). Spawn rate should be a sandbox multiplier.
5. Translations: English item names, e.g. "Soundbokks (Red)".

## Look / 3D model
The model is based on the Soundboks Gen 4 portable speaker. Match its overall form as
closely as you can:
- Tall, upright rectangular box with slightly rounded edges
- Full-face front grille (hex/perforated metal look) with a recessed frame around it
- Chunky rubber corner bumpers
- Integrated carry handle (top / shoulder-strap points)
- Removable battery pack slot on the back, control panel strip
- Matte, rugged materials

Logo: add an original "SOUNDBOKKS" logo on the front frame (bottom-centre) and on the top panel.
- Design it from scratch: bold, blocky sans-serif wordmark, all caps, wide letter spacing.
  Do not copy or imitate the real Soundboks logo's typeface, shape or layout.
- Make it a separate decal area on the texture atlas, so I can swap the logo image later
  without touching the model.
- Light logo on dark colours, dark logo on White, so it stays readable from the
  isometric camera.
- Also put the logo on the inventory icons and on the mod's poster.png.

It has to read well from Project Zomboid's isometric camera:
- Strong, chunky silhouette; low/medium poly (roughly 1–3k tris)
- Put the fine detail (grille holes, screws, panel lines) into the texture, not the geometry
- High-contrast grille vs. body so it stays recognisable when small on screen
- Produce: a world model (placed on the floor), an in-hand model (correct scale and
  attachment for carrying), and 2D inventory icons for each colour

Since you can't hand-model, write a **Blender Python script** (Blender 4.x) that:
- Builds the mesh procedurally with proper UVs
- Creates a texture atlas layout and generates base textures (body colour, grille, bumpers,
  logo decal)
- Exports FBX at the scale/orientation Build 42 expects (check how New Music's own
  boombox model is exported and match it)
- Renders the inventory icons (transparent PNG, matching vanilla icon size/angle)
I'll run the script and polish the model by hand if needed.

## How to work
1. **Investigate first. Don't write code until this is done.** Read New Music's Lua and
   scripts and report back:
   - How its boombox is defined (item script, device data, media type checks)
   - Whether a new device can be registered through a data table/API, or whether functions
     need overriding
   - How it sets sound volume/radius and zombie attraction (and whether it uses
     addSound / WorldSoundManager)
   - How it handles multiplayer sync for playback
   - B42 folder structure it uses (42/, common/, media/...)
   Give me a short plan based on what you find, and wait for my OK.
2. Build in stages, with something I can test in-game after each:
   a. Mod skeleton + one black Soundbokks using the existing New Music boombox model
      as a placeholder. It plays cassettes.
   b. Loudness + zombie range + sandbox options.
   c. Colour variants, loot distribution, translations.
   d. Logo, Blender script, custom model, icons.
3. Prefer hooking/extending New Music over copying big chunks of its code, so my mod
   survives New Music updates. Put any override in one clearly named file.
4. After each stage, tell me exactly how to test it (debug mode, item spawner, what to
   check in console.txt).

## Deliverables
- Folder: Soundbokks/ with Build 42 layout (mod.info, poster.png, icon.png,
  42/media/scripts, 42/media/lua/{shared,client,server}, models, textures, translations)
- The Blender script in a tools/ folder
- logo.svg / logo.png source files in a tools/ folder
- A short README: install, dependencies, load order, sandbox options
