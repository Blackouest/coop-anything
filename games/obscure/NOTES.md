# Obscure (2004) online co-op: notes

Status (2026-10-24): Phases 0 to 2 done. Phase 3 (understand) is under way. The static analysis is done; nothing has been run in the game yet.
Claim issue: https://github.com/cranebeaker/coop-anything/issues/183. Branch: `game/obscure`.

The summary below is current. The dated sections after `## Sources` are the running log, with the evidence and sources.

## Game identity (AGENTS.md rule 5), recorded 2026-10-24
- Title: Obscure (Hydravision Entertainment, 2004). Obscure II / The Aftermath (Steam app 254480) is a separate target, for later.
- Store: Steam, app 254460, build id 215286. The exe says "Mighty Rocket Studio", FileVersion 1.0.1.0, PE timestamp 2014-03-18 (the Steam re-release).
- `Obscure.exe` SHA-256 `7107439eb6ee1b20cad32e87f7a10a3a95b71c0b5ee26b1ec9139b263577a0f2`, PE32 x86, SteamStub-wrapped (`.bind`), `.text` not encrypted.
- Engine: **Hydravision in-house** (Zoe/Noe/Hoe/TM layers, `.hvp` archives, `.zwo` models, compiled `.hoe` scripts). **Not RenderWare**, despite the Wikipedia infobox.
- OS tested: Windows 11 Pro 10.0.26300.

## Where things stand
- **Existing co-op:**
  - The game has 2-player local co-op: player 2 presses Start on a second controller once the second character is on screen.
  - No online co-op mod was found (search list in Phase 1).
  - Steam Remote Play Together works by player reports, but needs a controller for at least one player.
- **Netcode:** none in the exe (no socket, dplay or Steam networking imports).
- **Co-op seams (Phase 3):**
  - Two real-player slots in the global game object, with characters switchable by script natives.
  - XInput pads polled per frame through `xinput1_3.dll`, so a proxy DLL can make a remote friend local player 2.
  - Story state is a 161-byte flag array; level changes go through 3 known fields.
- **Startup checks:**
  - Mutex `obscure_lock` (one instance per PC).
  - `IsDebuggerPresent` (no debugger).
  - Steam must be running.
- **Saves:**
  - Folder: `%USERPROFILE%\Documents\MRS\Obscure\` (from code: `SHGetFolderPathA(CSIDL_PERSONAL)` + `\MRS\` + `\Obscure\`). The test copy shares this folder with the real install.
  - The game also uses `SteamRemoteStorage` (Steam Cloud).
  - As of 2026-10-24 the owner has no Obscure saves: no `Documents\MRS` folder and no `userdata/<id>/254460`.

## Plan (updated)
1. ~~Research~~ done. 2. ~~Workspace~~ done.
3. **Understand (now):** done statically for players, input, story flags, levels and the startup checks. Next steps:
   - Runtime checks in the test copy, Steam Cloud off for Obscure (owner decision 2026-10-24):
     - Log which XInput index player 2 uses.
     - Confirm the real-player fields and the story flags change as predicted.
     - Find the per-frame hook.
   - Decode the `.hoe` bytecode so the story scripts can be read.
4. **Choose:**
   - Host authority. The guest's pad input goes to the host's player 2 through a proxy `xinput1_3.dll`.
   - The host streams world state (both characters, enemies, the 161 flags, doors/items, level changes) to the guest's copy.
   - Transport: Steam relay (playbook/networking-steam-spacewar.md), unless the owner prefers direct IP.
5. **Build:** a separate DLL package; no game files shipped. Milestones:
   - a. Remote input drives player 2.
   - b. The guest sees the correct world.
   - c. Levels and deaths stay in sync.
   - d. Save/load and drop-in/out.
6. **QA:** bots and real clients (playbook/automated-qa.md). The mutex needs handling to run two copies on one PC.
7. **Ship, 8. Give back:** as in AGENTS.md.

## Rules to keep in mind (AGENTS.md)
- Ask the owner before installing software, large downloads, using accounts (Steam, Discord, GitHub) or publishing.
- Keep private paths and state in `local/`.
- No game files or assets in any PR. Decompiled code may be used, but it goes in the game's own repo, not this hub (rule 7).
- Never unpack or remove the SteamStub wrapper; analyse the file as shipped.
- Stop at the first contradiction between the game and these notes.

## Sources
- https://github.com/cranebeaker/coop-anything (README, AGENTS.md, DIRECTORY.md, playbook/triage.md, playbook/engine-families.md)
- https://en.wikipedia.org/wiki/Obscure_(video_game)
- https://www.co-optimus.com/game/304/PC/Obscure__The_Aftermath.html
- https://www.co-optimus.com/article/11518/can-t-see-the-obscure-collection-for-the-steam.html
- https://bloody-disgusting.com/news/3279943/obscure-series-coming-to-steam-with-new-features/
- https://giantbomb.com/wiki/Franchises/Obscure

---

## Phase 0: triage (2026-10-19, from public sources; kept as written, superseded by the summary above)

Status: Phase 0 (triage) done from public sources. Nothing installed, nothing run.

### Game identity (AGENTS.md rule 5)
- Title: Obscure (Hydravision Entertainment), original 2004 game. Obscure II / The Aftermath is a separate target.
- Platform target: PC (Steam re-release, 2014). Steam app page referenced in community links: app 254460.
- Store: Steam (owner confirmed 2026-10-19).
- Build id, SHA-256 of main exe, exact OS: **unknown, record before any work**.
- Obscure II / The Aftermath: wanted later, as a separate game entry. Not part of this scope.

### Observations (sourced)
- Local 2-player drop-in co-op exists; the second character is an AI buddy a human can take over. (Co-Optimus, Giant Bomb)
- No online or LAN co-op listed for the PC version. (Co-Optimus, co-op.gg)
- Steam re-release added 16:9 and controller support. (Bloody Disgusting, Co-Optimus)
- Not in coop-anything DIRECTORY.md or games/ as of 2026-10-19.
- Wikipedia infobox lists engine as RenderWare.

### Unknowns (do not guess)
- Exact engine and version in the Steam build (check DLL names, exe version info, file extensions).
- Whether any netcode or debug symbols remain in the exe.
- How the game stores state: character roster, deaths, inventory, puzzle and door flags, saves.
- Whether the 2014 Steam build differs from the 2004 PC build in ways that matter for hooking.
- Whether existing community tools (modding, trainers, texture/model extractors) exist for the format.
- Whether anyone has already started an online co-op attempt (check issues/PRs on coop-anything, forums, Discord).

### Difficulty rating
Closed binary, no engine multiplayer, no scripting: "very hard" per playbook/triage.md. Scope is small though: 2 players, one human-controlled buddy slot already exists in the game.

### Plan
1. **Research (Phase 1).** Follow playbook/research.md. Search GitHub, forums, ModDB, Steam discussions, Discord for any Obscure online, LAN, trainer or modding tool. Mine bug lists if anything turns up. Check engine-families.md for RenderWare co-op work to reuse.
2. **Workspace (Phase 2).** Copy the install into a test folder (never touch the real install or saves). Set up Ghidra or x64dbg and a notes file. Ask owner before installing anything.
3. **Understand (Phase 3).** Identify the engine, then map: main loop, player/buddy controller (the AI-buddy slot is the natural hook for a remote player), character switching, character death, inventory, doors/puzzle flags, level loading, saves.
4. **Choose (Phase 4).** Likely design: host authority. Host runs the full game; the client is a puppet that sends inputs and receives the buddy character's state. Transport: Steam relay via Spacewar (playbook/networking-steam-spacewar.md), so no port forwarding.
5. **Build (Phase 5).** Separate DLL/mod package, no game files shipped. Milestones in order:
   - a. Remote controller drives the buddy character (inputs only).
   - b. Remote player sees correct world, enemies and doors.
   - c. Level transitions and character death stay in sync.
   - d. Save/load and drop-in/drop-out.
6. **QA (Phase 6).** Automated tests with an in-game bot and real game clients (playbook/automated-qa.md). Do not use the owner as the test plan. Report only checks that actually ran.
7. **Ship (Phase 7).** One-click install, host and join instructions, README with known issues.
8. **Give back (Phase 8).** Add directory entry in data/directory.yaml, write games/obscure/README.md from templates/game.md, open a draft PR (owner reviews the diff first).

### Rules to keep in mind (AGENTS.md)
- Ask the owner before installing software, large downloads, using accounts (Steam, Discord, GitHub) or publishing.
- Keep private paths and state in `local/`. No game files, cracks or decompiled source in any PR.
- Stop at the first contradiction between the game and these notes.
- Open a "claim a game" issue on coop-anything before starting real work.

---

## Phase 1: research (2026-10-24)

### Searched (no Obscure online co-op mod found by any of these)
- This repo: `data/directory.yaml`, `games/`, upstream issues and PRs (`gh issue/pr list -R cranebeaker/coop-anything --search obscure --state all`): nothing.
- Web: "Obscure 2004 online co-op mod", "Obscure Hydravision PC multiplayer mod parsec remote play together", French "Obscure jeu Hydravision mod coop en ligne", Russian "Obscure 2004 мод кооператив по сети онлайн".
- GitHub: repo search "obscure hydravision", "obscure 2004", "obscure game"; code search "Obscure.exe" (hits are mostly Choo-Choo Charles, whose UE4 project is also called "Obscure").
- ModDB: `site:moddb.com Obscure Hydravision`, no ModDB pages returned. Not searched directly on the site.
- Nexus: search found no Obscure (2004) game page. Nexus returns 403 to automated fetches, so this is from search result text only.
- Steam discussions for app 254460: two threads read in full (links below).
- Not searched yet: archive.org, Reddit, the game's Discord (none found), Zenhax/XeNTaX for formats.

### Observations (sourced)
- Steam app 254460 is Obscure (2004), developer Hydravision Entertainment. The store page lists Single-player, Multi-player, Shared/Split Screen and **Remote Play Together**; the description says co-op is "on the same screen". Obscure II is a separate app, 254480. https://store.steampowered.com/app/254460/
- Players report that **Steam Remote Play Together works** for the full campaign:
  - HappyDroplets, 4 Jun 2025: "Steam remote play does work, I beat the game with a friend just last week using it"; the connection sometimes dropped and stream settings had to be tweaked. https://steamcommunity.com/app/254460/discussions/0/540735426727130309
  - One user, 1 May 2021: works, "however it seems you need Controllers", because the game sees both PCs as the same keyboard. Other users say one keyboard plus one controller works. QuotedSias, 15 Sep 2025: "a bit cluncky but it works". https://steamcommunity.com/app/254460/discussions/0/142261027580957991
  - Fruits, 3 May 2025: Parsec "works brilliantly". (same thread as HappyDroplets)
- How local co-op joins: the second player plugs in a controller and presses Start once the second character is on screen, about 10 minutes into the game (Alia Sun and Dusk Golem, 11 Mar 2014, first thread above).
- Wikipedia infobox: engine "RenderWare"; platforms PS2, Windows, Xbox. https://en.wikipedia.org/wiki/Obscure_(video_game)
- Modding tools:
  - studiosleafmore-maker/ObscureZWOimporter, "Obscure Mod Studio": a Blender add-on that imports `.zwo` maps, characters, skeletons and animations for Obscure (2004). Alpha. No license stated. Two commits, both on 2026-09-04; one branch (`main`); the code is shipped as a zip. Its README lists "HVP extraction/repacking workflow" as planned and gives the asset layout `data/_levels/...` and `data/_common/textures`. https://github.com/studiosleafmore-maker/ObscureZWOimporter
  - Nexus (Obscure 2): an HVP unpack/repack tool (obscure.hvptool.exe) and HVFilePacker. Whether either works on Obscure 1 archives is unknown. Pages return 403, so details come from search text only. https://www.nexusmods.com/obscure2/mods/2, https://www.nexusmods.com/obscure2/mods/3

### Derived
- Asset formats `.zwo` and `.hvp` are Hydravision's own. The `.txd` check that `detect_engine` uses for RenderWare may not apply. The engine is still **unknown** until the install is checked: Wikipedia says RenderWare, but that is unconfirmed for the PC/Steam build.
- Remote Play Together streams the host's game. The guest needs no copy of the game and there is no state to sync. For 2 players that may already meet the goal "online co-op with friends". A real netcode mod would only add native rendering on the guest (no stream latency or compression), and it is a "very hard" closed-binary project.
- The game has exactly 2 player slots, so 3 or more friends is not possible either way without deep changes.

### Corrections to the Phase 0 notes above
- The Co-Optimus link `game/304/PC/Obscure__The_Aftermath` is about **Obscure II**, not Obscure (2004). The Steam store page is a better source for local co-op.
- "No ... decompiled source in any PR": AGENTS.md rule 7 says decompiled code is fine to use. It goes in the game's own repo, not in this hub. Game files and assets stay out of every PR.
- "Not in DIRECTORY.md": rechecked 2026-10-24, still true.

### Unknowns (still)
- Build id, SHA-256 of `Obscure.exe`, engine: need the owner's install path (read only).
- Whether the "same keyboard" input quirk is still present in Remote Play Together today.
- Whether the Obscure 2 HVP tools open Obscure 1 archives.

### Decision for the owner (blocking Phase 2)
Try Remote Play Together first (zero install, already supported on the store page), or go straight to a native netcode mod? See summary in chat.

---

## Game identity, recorded 2026-10-24 (owner's Steam install, read only)
- Store: Steam, app 254460, **buildid 215286**. Owner's install is about 3.3 GB.
- `Obscure.exe`: SHA-256 `7107439eb6ee1b20cad32e87f7a10a3a95b71c0b5ee26b1ec9139b263577a0f2`, 3,662,336 bytes, PE32 x86. PE timestamp 1395162021 (2014-03-18). FileVersion 1.0.1.0, CompanyName "Mighty Rocket Studio".
- `steam_api.dll` SHA-256 `6756b377865f55e1c8c8a21526f87559741f204036ce33e5be594d3e13b53bce`; `binkw32.dll` SHA-256 `3627edef5383ad43ffc14ab14f8d81171b6c08fdcb9f52a657f24301d3fd7b00`.
- Files in `<game dir>`: `Obscure.exe`, `steam_api.dll`, `binkw32.dll`, `cachpack.hvp`, `datapack.hvp`, `kinepack.hvp`, `strmpack.hvp`, `mrs.bik`, `logoload.bmp`, `video_cfg.dat`, `Deluxe Content/`.
- OS (owner): Windows 11 Pro 10.0.26300.
- Tools: hub MCP `game_identity` and `detect_engine`, plus a read-only Python PE scan (`python -I`).

## First look at the exe (observed, static, read only)
- **SteamStub-wrapped but `.text` not encrypted.** Sections `.text .rdata .data .rsrc .reloc .bind`. Entry point 0x3862ee is in `.bind` (entropy 7.998). `.text` entropy is 6.572, which is normal code. As with MW2 (games/modern-warfare-2), the exe file can be disassembled directly without removing the wrapper. Never unpack it or publish an unwrapped copy.
- **No network code imported.** Imports are VERSION, WINMM, AVIFIL32, MSVFW32, d3d9 (`Direct3DCreate9`), DINPUT8, DSOUND, SHLWAPI, binkw32, XINPUT1_3, KERNEL32, USER32, GDI32, COMDLG32, ADVAPI32, SHELL32, ole32 and steam_api (UserStats, Utils, Friends, User, Apps, RemoteStorage, callbacks). No wsock32, ws2_32 or dplay; steam_api has no Networking or Matchmaking imports.
- **Engine: Hydravision in-house, not RenderWare** (contradicts the Wikipedia infobox and the Phase 0 note). `detect_engine` found no known engine. There are no RenderWare/Criterion strings and no `.txd`/`.dff` files. 722 MSVC RTTI class names show its layers:
  - `CZoe*`: renderer and scene (DX9).
  - `CNoe*`: physics, collision, entities.
  - `CToe*`, `CHoe*`: behaviour/AI, `CHoeEntityPlayer`.
  - `CHV*`: input, controller, sound, streams.
  - `CZPackage*`: the HVP archives.
  - `CZMemoryCardSteam`: saves through Steam.
  - `Cslh_L*`: a script language ("slh") with functions, expressions and consequences.
  - About 300 `CTM_*` script-callable natives.
- **Co-op seams visible by name (derived, unverified):**
  - Players and control: `CRealPlayer`, `CPlayerHuman`, `CPlayerHumanManager`, `CTM_GetRealPlayer`, `CTM_GetRealPlayerControlerPort`, `CTM_GetMyPlayer`, `CTM_ChangeTeammate`, `CTM_SetTeammate`, `CTeammateMenu`, `CTM_GetNbPlayersAlive`, `CTM_ResetPlayer2`, `CTM_WalkableUntilPlayer2`, `CMenuNodeSelectControllerBox`.
  - Input: `CHVInputDX9`, `CHVControlerDX9`, `CInputConfig`. Hooking DirectInput8/XInput could feed a remote player's pad into port 2.
  - World state: `CItGameStateManager`, `CItGameState*`, `CTM_SetEventState` and `CTM_SetGameEventState`, `CTM_EventChangeLevel`, `CTM_LoadLevel`, `CTM_PNJGetSaveState` and `CTM_PNJSetSaveState`, `CSave`.
  - Save string "Can't read this file because the number of player saved is different of actual player number!" means saves record the player count.
  - Script paths: `data\_common\scripts_fx\*.txt` (FX scripts as text inside the HVP); character models `data\_common\players\<boy1..7|gir1..4>\skin\*.zwo`.
- A GOG release also exists (`setup_obscure.exe` in GOG checksum lists, e.g. github.com/cloudskytian/MD5-for-GOG.com). It isn't needed for static analysis, because the Steam `.text` is already readable.

## Contradiction (AGENTS rule 6), reported to owner 2026-10-24
- Phase 0 notes and Wikipedia say RenderWare; the Steam exe shows a Hydravision engine (Zoe/Noe/Hoe/TM/slh). Proposed fix: correct the engine in the write-up and directory entry, and do **not** add Obscure under RenderWare in playbook/engine-families.md. Wikipedia may be right about the PS2 version only (unknown).

---

## Phase 2: workspace (2026-10-24)
- Claim issue: https://github.com/cranebeaker/coop-anything/issues/183. The claim template's `in-progress` label doesn't exist upstream, so `gh issue create --label in-progress` fails ("could not add label"). The issue was opened without a label; proposed fix: create the label or drop it from `.github/ISSUE_TEMPLATE/claim-a-game.yml`.
- Test copy: `local/obscure-test/` (robocopy, 199 files, 3.2 GB). The copied `Obscure.exe` SHA-256 matches the install. All runs and patches happen only here.
- Tools, all inside `local/tools/` (git-ignored), checked by SHA-256 against the publishers' values:
  - Ghidra 12.2 (`ghidra_12.2_PUBLIC_20261016.zip`, `2c2cd776db3c1f6a097ba418922378ac3649cadd5e844a6f5f088ae628a7be67`, from the GitHub release body).
  - Temurin JDK 25.0.4.1+1 (`OpenJDK25U-jdk_x64_windows_hotspot_25.0.4.1_1.zip`, `00c847d804f4a78e9f04f2683faf14fed898535b177b7fc704486cb0284e9283`, from the Adoptium API).
- Pitfall: **Ghidra 12.2 needs Java 25** (`application.java.min=25` in `Ghidra/application.properties`), not Java 21. `JAVA_HOME_OVERRIDE` in `support/launch.properties` is not enough on Windows, because `analyzeHeadless.bat` first looks for `java` on PATH or `JAVA_HOME` ("The 'java' command could not be found"). Set `JAVA_HOME` and PATH for the call.
- Ghidra headless smoke test: `analyzeHeadless <proj> Obscure -import <test copy>\Obscure.exe` completed with "Analysis succeeded", "Save succeeded", "Import succeeded" (exit 0). The project is in `local/ghidra-proj/`. Workspace ready for Phase 3.

---

## Phase 3: understand (2026-10-24), static analysis of build 215286

Everything below comes from the exe file and the archives in the test copy. Nothing has been run in the game yet. Addresses are virtual addresses in `Obscure.exe` (image base 0x400000), SHA-256 `7107439e...77a0f2` only.

### Tools used
- Ghidra 12.2 headless: auto-analysis, then `DecompileAll.java` (from cranebeaker/deus-ex-coop, read before use). 7,738 functions, 1 failed.
- Own scripts, in `local/` for now:
  - `ExportXrefs.java`: strings with the functions that reference them.
  - `DecompileAt.java`: creates, names and decompiles functions from a TSV.
  - `hvp.py`: HVP archive reader.
  - `pe.py`: reads bytes at a virtual address.
- Pitfall: Ghidra's auto-analysis didn't label the MSVC vtables (`vftable` symbols: 0), so functions reached only through a vtable were never created. Fix: read the vtable address from the constructor (`*this = &PTR_...`), read the slots from the file, and create the functions with `DecompileAt.java`.

### Archives (.hvp): observed, reader verified on all four
- Big-endian. Header 0x28 bytes:
  - magic `HV PackFile\0`, u16 3, u16 1;
  - u32 root entries, u32 total entries, u32 file entries, u32 TOC size;
  - 8 bytes unknown.
- TOC entries, depth first, each starting with u32 entry size (counting itself) and a u8 type:
  - type 0, directory: u32 0, u32 child count, u32 name length, name.
  - type 1, file: u32 compressed, u32 stored size, u32 size, u32 crc, u32 absolute offset, u32 name length, name.
- Compressed data is zlib (`78 da`). Checked on 847 files: all inflate to exactly the listed size.
- The `crc` field is **not** standard CRC-32: 2 of 847 match, probably by chance. It only matters for repacking, and it's **unknown**.
- File counts: cachpack 732, datapack 1708, kinepack 40 (`.bik` cutscenes, stored), strmpack 6447 (`.ogg` voice/music).
- Content types (datapack/cachpack):
  - `.hoe` ×202: compiled behaviour scripts. Named globals such as `gPlayerDetected`, `gIsRespawn`, `gHelpRespawn`, `gPNJs`; 179 of them name `TM_*` natives.
  - `.tm` ×150: per-level binary data, mostly floats (`<level>_n.tm`). Its meaning is unknown.
  - Also: `.noe` ×100, `.dip` ×139, `.coa` ×81.
  - Text configs: `gdatas*.ini` (weapon and difficulty tables, plain text with French comments).
  - `default.ipt` and `default_pad.ipt`: input maps, a binary table with device names such as "Keyboard".
  - `_common/chapters/chaptr0..9.sav` and `default.sav`/`default2.sav`: save templates, 30720 bytes each (22528 for `default.sav`).

### Script natives (observed)
- 255 natives. Each is a class with a constructor calling `FUN_005b8860(name, flag, nargs)` and setting its vtable. Slot 3 is the body (all 255 slot-3 targets are distinct).
- The by-name factory is `FUN_004a3140`. The full table (name, flag, args, constructor, vtable, body) is in `local/decomp/natives.tsv`.
- Two families: `TM_*` (world, events, players, music) and the AI ones (`GetDistanceWithPlayer`, `SelectPlayer`, `LookAt` and others, the `CHoeSlh_*` classes).

### Players and controllers (observed in code, field meanings derived)
- `DAT_006ab8e4` is the global game object.
- `game+0x4fc` and `game+0x500` are the **two real-player slots** (`CRealPlayer*`, index 0 and 1). Natives reject an index of 2 or more, so there are exactly 2 human slots.
- In a real player:
  - `+0xc`: the controlled character. `TM_GetRealPlayer` returns it only if `+0x58 == 0`, `+0x5c == 0` and float `+0x54 >= 1e-5` (alive check; that +0x54 is health is derived).
  - `+0x84`: the controller object; its `+4` is the port/type returned by `TM_GetRealPlayerControlerPort` (`0x4dfc20`).
- `FUN_0044b3b0(character)` switches which character a real player controls. `TM_ChangeTeammate` (`0x4df680`) and `TM_SetTeammate` (`0x4de350`) both call it.
- `TM_GetMyPlayer` (`0x4de890`) matches characters by `+0x68` against `*(game+0x57c)+0x60`. It walks one of two lists depending on `*(...+0x10) == 7`.
- Controller polling is `FUN_005868e0`. Controller type `+4`:
  - 1: keyboard (state buffer at `+0x138`, 0x100 bytes).
  - 2: probably a DirectInput joystick (unconfirmed).
  - 3 to 6: XInput user 0 to 3, read by `XInputGetState(index, this+0x250)` and rumbled by `XInputSetState`. The raw sticks become floats at `+0x260..+0x26c` and the buttons are unpacked to `+0x270..+0x2ac`.
  - Pad names "Pad 1".."Pad 4" (`FUN_005417a0`) and "Joystick 0".."Joystick 3" (`FUN_00586ee0`).
- XInput is imported by ordinal from `XINPUT1_3.dll` (#2 XInputGetState, #3 XInputSetState), and DirectInput through `DirectInput8Create`.

### Story, levels, saves (observed)
- **Story flags:** a byte array of 161 entries at `*(game+0xa24)`, written by `TM_SetGameEventState` (`0x4d8c70`) and read by `TM_GetGameEventState` (`0x4d8c30`).
- **Per-event state:** `TM_SetEventState` (`0x4d8c10`) writes `event+0x18`.
- **Level change:** `TM_LoadLevel` (`0x4ddb80`) and `TM_EventChangeLevel` (`0x4d92a0`) set `game+8 = 5` (the game state machine, value 5 = level change requested: derived), copy the level name to `game+0x590`, and set the entry index at `game+0x5d0`.
- **Saves:** they go through `CZMemoryCardSteam`, and the exe imports `SteamRemoteStorage`, so **saves are probably Steam Cloud files** (derived). Error strings: "Not a valid save file", "Save directory does not exist", "Can't read this file because the number of player saved is different of actual player number!".

### Startup checks (observed)
- `CreateMutexA(NULL, 0, "obscure_lock")`: if the mutex already exists (`GetLastError() == 0xb7`), the game doesn't start. Two copies on one PC need this handled in our test copy, see playbook/multi-instance-testing.md.
- `IsDebuggerPresent()`: the game only runs its main function (`FUN_00547290`) when no debugger is attached. Don't attach a debugger; use an injected DLL and logs instead.
- `SteamAPI_Init` failure shows a localized "Steam must be running" message and exits.
- No network code anywhere: no winsock or dplay imports, no Steam networking or matchmaking interfaces.

### Derived: co-op design implications
- **Remote input:**
  - Plan: a proxy `xinput1_3.dll` in the test folder that forwards to the real system DLL, and reports a virtual pad on one XInput index whose state comes from the remote player. The game then treats them as local player 2. No exe patch needed.
  - The Remote Play "same keyboard" report fits the code: there is one keyboard device, so two remote keyboards merge into it.
- **What to sync:**
  - Host-authoritative world.
  - Guest view: the guest runs its own copy and mirrors the host. Characters (positions and animation of the 2 slots plus the teammates the host's AI drives), enemies (`CPlayerPNJ*`), the 161 story flags, event states, items and doors (`CItGameStateManager`), and level changes (`game+8/+0x590/+0x5d0`).
  - Simpler first milestone: input-only on the host (the guest streams video, like Remote Play but with our own input path). That's only useful as a stepping stone.
- **Risk before any runtime test:**
  - Steam Cloud saves (`SteamRemoteStorage`). Running the test copy under the owner's Steam account could sync test saves over the owner's real cloud saves.
  - Before the first launch, back up the owner's save files and decide how to keep test saves apart (Steam Cloud off for this game, or a separate account). That decision belongs to the owner.

### Unknowns
- Meaning of the `.tm` files; the `.hoe` bytecode format (needed to read story scripts as text); the HVP crc algorithm.
- The character, enemy and item structs, beyond the fields above.
- Where the per-frame main loop is, the best hook point for sync; `PeekMessageA` appears once, which is a candidate.
- Whether controller type 2 is DirectInput, and how a pad gets assigned to real player 2 (the controller-select menu `CMenuNodeSelectControllerBox`).
- Save file location and format; whether the 161 flags plus the chapter `.sav` templates are the whole story state.

### Runtime isolation blockers (observed in code, 2026-10-24); no launch made
- **`video_cfg.dat` lives in the Steam install folder, not next to the exe.** `FUN_00548740` (load) and `FUN_00548890` (save) build the path from `SteamApps()->GetAppInstallDir(254460)` (vtable `+0x48`, app id `0x3e1fc`) plus `\video_cfg.dat`. A copied test install would therefore read and **write the real install's** `video_cfg.dat` (49 bytes, display adapter and mode string). These are the only two `GetAppInstallDir` calls; archives are not loaded through it. Fix: hook that call inside the test copy's process and return the test folder.
- **Saves are Steam remote-storage files.** `FUN_0040da30`..`FUN_0040dd50` call `SteamRemoteStorage()` methods at vtable `+0xc`, `+0x28`, `+0x30`, `+0x34` and others (exists, size, delete, count; the exact slot names depend on the `steam_api.dll` interface version: unknown). Per playbook/methods.md (Shadow of War), `steam.exe` writes `FileWrite` data into `Steam\userdata\<id>\<appid>\remote` **even with Steam Cloud off**. Fix: block the remote-storage write slots in-process, as described there, and hash the userdata folder before and after each run.
- `video_cfg.dat` exists only in the real install (the game wrote it on the owner's own launch, 2026-10-24); a copy is in the test folder.
- So the first runtime test needs a small injected DLL (proxy `xinput1_3.dll`, the same DLL that will later carry remote input) with both hooks. There is no compiler on the owner's PC yet.
- Playbook note proposed: "a copy at another path does not isolate the game" also covers `GetAppInstallDir`: a Steam game can find its install folder through Steam rather than from its own exe path.

### Run 1 (2026-10-24): first launch of the test copy, isolation DLL "stage 0"
What ran (observed):
- **Setup:**
  - The test copy has a proxy `xinput1_3.dll` (own code, built with llvm-mingw 20261020, i686; the code lives in `local/` until it gets its own repo), plus `steam_appid.txt` = 254460.
  - The test copy's `video_cfg.dat` was set to `NVIDIA GeForce RTX 5090 [TnL HAL] - Windowed\0`. That is the windowed form built by `FUN_005ac610` (`<adapter> [TnL HAL] - Windowed`).
  - The game was started with env `SteamAppId`/`SteamGameId` = 254460, with Steam running.
- **Result:**
  - The process ran from the test folder; there was **no relaunch through Steam** and no second `Obscure.exe`.
  - It reached the main menu in about 6 s, with a window titled "Obscure".
- **DLL log:**
  - All 7 import hooks installed: CreateFileA/W, CreateDirectoryA, FindFirstFileA/ExA, SHGetFolderPathA, SteamRemoteStorage.
  - SteamStub did not rewrite the import table: the watcher checked for 20 s and never re-applied.
  - `CreateFileA ...\steamapps\common\Obscure\video_cfg.dat` was redirected to the test folder (2 calls).
  - `SteamRemoteStorage` was proxied. The game asked `FileExists tm_input.ipt` and `FileExists options.sav` (both 0 = first run). So the input config and options are cloud files too.
  - The game polls **XInput users 0, 1, 2 and 3**. All returned 1167 (ERROR_DEVICE_NOT_CONNECTED, no pad plugged in).
  - `SHGetFolderPathA` was not called before the main menu.
- **Memory peek** (ReadProcessMemory only, module base 0x400000, so no ASLR on the exe):
  - At the main menu the game object `*(0x6ab8e4)` is non-null, `game+8 == 0`, both real-player slots `+0x4fc/+0x500` are NULL, and no level name is set.
  - `*(game+0xa24)` pointed at text (French script comments), not a flag array. Either the story-flag array only exists in-game, or the offset is wrong: **unverified**.
- **Owner-file check:** a snapshot was taken before and after (sizes, mtimes and hashes of every real-install file under 2 MB; `Documents\MRS`; `userdata\<id>\254460`) and the two are **identical**.
  - The game was killed with `taskkill /F`, so its on-exit `video_cfg.dat` save was not exercised.

Next (planned):
- Inject virtual pad input through the proxy, which is the same path the remote player will use: drive the menus with a scripted pad on user 0 and start a New Game.
- Then verify the player slots, the controller type values (expected 3 to 6 for pads) and the story-flag pointer in a level.
