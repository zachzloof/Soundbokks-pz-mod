# HANDOFF → next session on the gaming PC

> Delete this file once the next session has read it and started Stage 0.
> (`git rm HANDOFF.md` — or just delete it; nothing references it.)

## Where things stand (written 2026-11-09 on the laptop)

- Repo is a fresh git repo, **no commits yet**, on `main`.
- Nothing in `Soundbokks/` exists yet. No mod code has been written. The user explicitly
  wants investigation + plan approval before any code.
- The laptop had neither Project Zomboid nor New Music installed, so the investigation
  could not be done. The user's `~/Zomboid/console.txt` (copied from the PC) confirmed:
  B42.19.0 stable, game at `C:\Program Files (x86)\Steam\steamapps\common\ProjectZomboid`,
  translations are JSON (`media/lua/shared/translate/en/itemname.json`, `sandbox.json`).
  That log predates New Music being installed (it lists TrueMusic mods instead).
- Web research done; the useful parts are baked into `.claude/skills/pz-b42-modding/`,
  `.claude/skills/pz-model-pipeline/`, and `docs/reference/pz-modding-guide/`. Anything
  there marked "verify" is unverified against real game files — treat as a hint only.
- Found no installable Zomboid-specific skill anywhere. Vendored two MIT Blender/bpy
  skills from `pjt222/agent-almanac` (reviewed, no network/shell calls in them).

## Files in this repo

| File | Purpose |
|---|---|
| `CLAUDE.md` | Rules, layout, env paths, conventions — read first |
| `PLAN.md` | Staged plan with a status table; Stage 0 is the investigation checklist |
| `.claude/skills/pz-b42-modding/SKILL.md` | B42 modding reference + what to verify where |
| `.claude/skills/pz-model-pipeline/SKILL.md` | Blender→PZ FBX/atlas/icon pipeline |
| `.claude/skills/blender-bpy-automation/`, `blender-render-output/` | Generic bpy skills (vendored, MIT) |
| `docs/reference/pz-modding-guide/` | gotmayonase/pz-modding-guide pages (B42.15+), read-only |

## Set up on the PC before starting

1. Steam: PZ installed on **Build 42 stable**, subscribed to *New Music [42+]*
   (Workshop 3739256725). Confirm `C:\Program Files (x86)\Steam\steamapps\workshop\content\108600\3739256725\` exists.
2. Blender 4.x installed (`blender --version` works, or note the full exe path).
3. Python 3 with Pillow: `python -m pip install pillow` (for logo/texture generation).
4. Optional but useful: `claude plugin install lua-lsp@claude-plugins-official` (needs
   `lua-language-server` on PATH) for Lua code intelligence.
5. Clone/pull this repo, open it in Claude Code, paste the prompt below.

## Initial prompt for the next session

```
Read HANDOFF.md, CLAUDE.md and PLAN.md, then delete HANDOFF.md (its contents are now in
your context). Do NOT write any mod code yet.

Do Stage 0 of PLAN.md: investigate New Music [42+] and the vanilla B42 files.
- New Music: C:\Program Files (x86)\Steam\steamapps\workshop\content\108600\3739256725\
  (find the real mod folder + mod.info inside it)
- Game: C:\Program Files (x86)\Steam\steamapps\common\ProjectZomboid\media\
- My console log: %USERPROFILE%\Zomboid\console.txt

Report back, concisely:
1. How NM's boombox is defined (item script keys, model script, device data, how media
   type/cassette checks work).
2. Whether a new device can be registered through an NM data table/API, or which exact
   functions would need wrapping (name each one + file:line). Recommend one approach and
   say how fragile it is to NM updates.
3. How NM sets playback volume / audible radius, and whether/how it attracts zombies
   (addSound / WorldSoundManager, radius + volume values, which tick/event).
4. How NM syncs playback in multiplayer (commands, ModData, what the server owns).
5. The B42 folder structure NM actually uses (42/, common/, models_X vs models,
   translate format, where mod.info lives), plus the vanilla syntax you verified for
   sandbox-options.txt, itemname.json/sandbox.json and ProceduralDistributions list names.

Write the full findings to docs/investigation.md, then give me a short plan for stages
a–d with risks, update PLAN.md's open questions, and WAIT for my OK before Stage a.
```
