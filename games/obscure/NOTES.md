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

### Runs 2 to 7 (2026-10-24): bot input in the test copy
Every run was started and stopped with `local/runs/run.sh`, which compares a before and after snapshot of the owner's files (real install, `Documents\MRS`, Steam `userdata\<id>\254460`). All 6 runs: **owner files unchanged**. The game wrote nothing in the test copy either: no saves, and nothing in `coop_profile/remote` except the file a test put there.

#### DLL stage 1 (own code, in `local/coopdll` for now)
- **Command pipe** `\\.\pipe\obscoop_<pid>`, one line per command:
  - `pad <user> <buttons hex> [lx ly rx ry lt rt]` / `unpad <user>`: virtual XInput pads 0 to 3.
  - `key <DIK hex> <0|1>`: virtual DirectInput keyboard key.
  - `stats`: poll counts and whether the game has the foreground.
  - `ping`.
- **Virtual pads:** `XInputGetState` returns the virtual state for that user index. `OBSCOOP_VPADS=1` reports pad 1 as connected from the first poll.
- **Virtual keyboard:** `DINPUT8!DirectInput8Create` is hooked (import table), then `IDirectInput8A::CreateDevice` (vtable slot 3). For `GUID_SysKeyboard`, `IDirectInputDevice8A::GetDeviceState` (slot 9) gets the virtual keys OR-ed in.
- **Background running** (`OBSCOOP_BACKGROUND=1`): `USER32!RegisterClassA` is hooked to wrap the game's window procedure `FUN_00546660`, so `WM_ACTIVATE` always reports active. That procedure stores the active state in `DAT_0069b870`, and the main loop (`FUN_00547290`) only updates input while it's set.
- **Bot:** `local/runs/bot.py`: pad presses, key taps, `PrintWindow` screenshots. `local/runs/peek.py`: reads the game object, player slots and story flags with ReadProcessMemory.

#### Observed
- The game polls all 4 XInput users every frame, about 3,000 polls/s with the window active.
- **Devices** (`FUN_00586ee0`): `Keyboard` (controller type 1), `Mouse` (2), and XInput users 0 to 3 as `Joystick 0`..`Joystick 3` (types 3 to 6).
- **Input maps:** `default.ipt` maps the keyboard (RETURN, ESCAPE, SPACE, arrows, numpad, LSHIFT, RCONTROL). `default_pad.ipt` maps `Joystick 0`, with buttons `Button 01`..`Button 16` and axes `AXE X/Y`, `ROT X/Y`.
- **The menus don't react to a virtual pad** (A, START, DOWN on user 0, with or without "connected at start"), even though the game reads it and has the foreground. They react to the virtual keyboard: Enter on NEW GAME opens DIFFICULTY.
- **Background:** without the window-procedure wrap, the game stops polling entirely when its window loses the foreground (poll counter frozen).
  - With the wrap, it runs, but the real DirectInput keyboard read fails while unfocused. The first fix only replaced the read while a virtual key was down, so a key's release was never seen and the next press didn't register (stuck on DIFFICULTY twice).
  - Fix: in background mode, always turn a failed read into an empty successful one.
- **Reached gameplay with no human input:** New Game → Normal → opening video (skip prompt "Press ESC to stop watching the video") → Kenny in the gym. Screenshot in `local/runs/shot_play.png`.
- **In-game memory:**
  - Real player 0 exists, with character `+0x54 == 100.0` (health, as derived) and controller type 1 (keyboard).
  - Real player 1 exists but has character NULL and controller `+0x84` NULL. This fits the second character arriving later.
  - `*(game+0xa24)` is now a 161-byte array, all 0 except index 160 (values 72, then 6, in two runs). So the story-flag pointer is right in-game; at the menu it pointed at unrelated text.
  - `game+8` stayed 0 in-game, so 5 is a transient "change level" value.
- **Not confirmed:**
  - The up arrow (DIK 0xC8, 1.2 s) turned Kenny, but nothing in the first 0x200 bytes of the character object changed as a float. Position lives elsewhere (an entity or physics object): **unknown**.
  - START on virtual pad user 1 with only one character in play: no change to real player 1.

#### Saves
- Slots are `game0.sav`..`game9.sav` plus `recover.sav` (save writer `FUN_00416640`); options in `options.sav`; input map in `tm_input.ipt`.
- All are Steam remote-storage files. The LOAD screen asked for `game0..9.sav`.
- **Checksum:** standard CRC-32 (table at `0x69c560` equals the standard table; `FUN_00576640`) over bytes `[4:]`, stored little-endian in bytes `[0:4]`. Every `chapters/chaptr0..9.sav`, `default.sav` and `default2.sav` in datapack matches. The byte at offset 4 is a version, and must be 6 for `options.sav`.
- A chapter template copied as `game0.sav` shows **"Damaged game"** on the LOAD screen. The game-save loader (around `0x452400`) runs `FUN_0044f1e0` on the buffer first, then compares the CRC with a value from a slot index (`FUN_00415c30` entry `+0x10`, version `+0x14`, slot `+0x18`), not with the file header. The templates are not drop-in saves: **unknown** why.

#### Proposed playbook additions (for the PR)
- methods.md, isolation: a Steam game can find its own folder through `ISteamApps::GetAppInstallDir`, so a copied test install reads and writes the store install. Redirect that path inside the process.
- methods.md, background instances: a DirectInput keyboard read fails while the window is in the background. Injected keys must come with successful empty reads, or key releases are lost.

### Runs 8 and 9 (2026-10-24): the owner's save in the test copy, player 2 assignment
- **The owner's save:** the owner played their real install themselves to the point where both characters are in the team, and saved.
  - Files: `game0.sav` (22,528 bytes, CRC ok, version byte 6), `options.sav` (1,024) and `tm_input.ipt` (65,004), in `Steam\userdata\<id>\254460\remote`, plus `remotecache.vdf`. That is the place the code predicted.
  - `Documents\MRS` is still not used.
  - Backed up to `local/backup/` and hash-checked. Only `game0.sav` was copied into the test copy's `coop_profile/remote`.
- **Why the chapter templates showed "Damaged game":** real game saves are 22,528 bytes, the `chapters/chaptr*.sav` templates 30,720. The loader rejects a file whose size differs from the expected size (`+0x5c`).
- **Loading in the test copy:**
  - The bot took LOAD → slot 01 ("Central courtyard (ground floor) / 01 / 00:28:46 / NORMAL") → "Load Game? YES".
  - The level loaded with two team members at the fountain.
  - Real player 0 character health 49.711, controller type 1. Real player 1 slot present, character NULL, controller NULL.
  - Story flags at `*(game+0xa24)`: 24 non-zero entries (0..5, 10..13, 113, 121, 124..132, 148, 153, 154, 160). This confirms the array holds story progress.
- **Player 2 needs a controller assigned first.** Pause → OPTIONS → BUTTON SETTINGS (1/3) "MOVES / TEAMMATE" shows Player 1 = `Keyboard` (W/S/D/A, RIGHT SHIFT walk, NUM 8/2/6/4 team orders) and Player 2 = "No controller assigned". START on virtual pad user 1 does nothing while P2 has no device.
  - Player 1 moves with **W/A/S/D**, not the arrows. That is why the up arrow in run 7 didn't walk.
- **Per-player device:** a table at `*(game+0xa20) + 0x2c0`, entry `+4 + player*0x38` = device type (0 none, 1 Keyboard, 3..6 Joystick 0..3).
  - Setter `FUN_00450590(player, type)`, getter `FUN_004505b0(player)`; both thiscall with ECX = `*(game+0xa20) + 0x2c0`. Seen at `0x5387d2..0x538848`.
  - Read live: P1 = 1, P2 = 0.
  - The menu's apply path (`FUN_005381c0`, panel state `+0x98 == 4`): swaps devices if the other player already has the chosen one, calls the setter, then rebuilds devices (`FUN_00586080`, `FUN_00586530(0)`, `FUN_005854f0(1)`, `FUN_005854d0(1)`, `FUN_005854e0(0)`, `FUN_00586ee0()`) and reloads the maps (`FUN_00451f60`).
- **`CMenuNodeSelectControllerBox`** (vtable `0x654f7c`; found from RTTI: type descriptor → complete object locator → vtable):
  - Event handler slot 3 (`0x540d00`): event 1 with item id `0x3b6`..`0x3ba` selects device type 1, 3, 4, 5, 6 (box `+0x90`), with `+0x84/+0x88` = done.
  - The box is not operable with the virtual keyboard: RIGHT/RETURN/DOWN/ESC had no visible effect on that screen and ESC didn't leave it. Pad buttons A, B, X, Y, START, BACK, LB, RB and dpad on user 1 didn't change P2's device either.
  - The game creates no DirectInput mouse and imports no `GetCursorPos`, so it isn't a mouse screen. **How a human assigns P2 there: unknown.**
- **Writing P2's device = 4 into memory directly** (WriteProcessMemory, test process only) took (read back 4), but START on pad 1 still didn't fill P2's slot. The device rebuild and map reload that the menu does afterwards were not run. **Unverified** whether those are what binds a real player's controller (`+0x84`).
- **DLL stage 1b:** a virtual DirectInput mouse hook (GUID_SysMouse, deltas and buttons) was added. Unused: the game never creates a mouse device.
- **Owner-file guard:** `steam.exe` rewrote `userdata\<id>\254460\remotecache.vdf` at each test launch and exit (mtime 17:43, 17:45) with **byte-identical content** (SHA-256 `5875b10a…`). The owner's three save files kept their hashes. The snapshot now compares that folder by content hash.
  - Playbook note proposed: even with the game's own remote-storage writes blocked, Steam touches the app's `remotecache.vdf` on every launch, so compare by hash, not mtime.

### Runs 10 to 12 (2026-10-24): owner's input settings, buffered keys
- Owner (2026-10-24): "I have selected gamepad 1 with keyboard and it was working fine on my controller." So the BUTTON SETTINGS controller box *is* keyboard-operable for a human, and "gamepad 1" is the UI name. Which column (Player 1 or 2) it was set for: unknown.
- The owner has an XInput controller connected. XInput is read whatever window has focus, so the DLL now has `OBSCOOP_HIDE_REAL_PADS=1` (on in all test runs): real pads report "not connected" and rumble goes nowhere, so the owner's controller can't reach a test copy.
- With the owner's `tm_input.ipt` (65,004 bytes, read by the game at start) and then also `options.sav` (1,024) in the test copy's fake cloud, the per-player device table still reads P1 = 1 (keyboard), P2 = 0, both at the main menu and after loading `game0.sav`. Real player 1 still has no controller. So either the owner's choice isn't saved in these files, or it was for Player 1: unknown.
- Added buffered keyboard events (`IDirectInputDevice8A::GetDeviceData`, slot 10) to the virtual keyboard. The game never calls it (`kbd_data_reads` stays 0), so that's not why the controller box ignores injected keys. The game uses no `GetCursorPos`, `GetAsyncKeyState` or `GetKeyState` either, and its window procedure passes key messages to `DefWindowProc`. How that screen reads its keys: unknown.

### Run 13 (2026-10-24): player 2 joins on a virtual pad (first co-op milestone, local)
- The owner set Player 2 = "game controller 1" in their real game (2026-10-24 17:56). Only `tm_input.ipt` changed (new SHA-256 `f0ca1734…`; backed up). So **the controller assignment is stored in `tm_input.ipt`**, a raw image of the binding tables with `0xcd`/`0xcc` filler, little-endian.
  - Its layout beyond the device-name strings: unknown.
  - With that file in the test copy's fake cloud, the device table reads P1 = 1, P2 = 3 (`Joystick 0` = XInput user 0), already at the main menu.
- **Run:** real pads hidden, virtual pad 0 connected at start, the owner's `game0.sav` loaded, then the virtual pad pressed buttons. The game showed:
  > "Player 2 joins the game. Do you want player 2 to take control of your teammate? ENTER Confirm / ESC Cancel"
  - Virtual Enter (keyboard, player 1) confirmed it.
  - Real player 1 slot (`game+0x500`) then had character `+0xc` set (health `+0x54` = 100.0) and controller `+0x84` set with type 3. Real player 0 kept the keyboard (type 1, health 49.7).
  - Holding the virtual left stick down for 1.5 s walked player 2's character off the bottom of the screen; the game then draws an off-screen marker for it. Screenshots: `local/runs/p2_before.png`, `p2_after.png`.
- Which pad press raised the join prompt: unknown. Buttons were tried in the order A, B, X, Y, BACK, LB, RB, dpad, thumb clicks, START, then stick and triggers, and the prompt was only seen after all of them.
- **What this proves (observed):** a program-driven XInput pad, invisible to the real devices, is accepted by the game as local player 2, and it can join and move a character. That is the input half of the co-op design (remote input → virtual pad on the host).
- **Not done:** networking, mirroring the host's world on a second PC, and finding which exact button raises the join prompt.

### Run 14 (2026-10-24): networked input relay, two copies on one PC (UDP on localhost)
- **Steam API check:** the game's own `steam_api.dll` (2014) exports `SteamNetworking` (`SteamNetworking005`), `SteamMatchmaking` (`SteamMatchMaking009`), `SteamFriends` (013), `SteamUser` (017) and `SteamUtils`. It doesn't export the flat API, so calls go through C++ vtables.
  - Plan (owner chose Steam, 2026-10-24): use these in-process under app 254460, so players show as playing Obscure. Both players own the game anyway, since the guest runs their own copy.
  - Not built yet. Stage 2 below uses UDP on localhost behind the same send/receive calls.
- **Second test copy** `local/obscure-test2`: hardlinks to the four `.hvp` files and `mrs.bik` (link count 2; the game never opened them for writing in any run), plus real copies of the exe, DLLs and small files.
- **DLL stage 2** (`net.c`):
  - `OBSCOOP_ROLE=host|guest`, `OBSCOOP_NET=udp`, port `OBSCOOP_UDP` (default 27960), `OBSCOOP_PEER`.
  - The guest sends its pad 0 (`XINPUT_STATE` layout) at 125 Hz, sends HELLO until WELCOME, and pings every 500 ms.
  - The host accepts one guest, applies only newer sequence numbers to virtual pad 0, and drops the guest (pad 0 "disconnected") after 3 s of silence or a BYE.
  - `OBSCOOP_MULTI=1` renames the `obscure_lock` mutex per process. Two copies then ran side by side with no further single-instance check. That's observed: both reached the main menu and later in-game.
- **Test** `local/runs/net_join_test.py` (checks read from host memory):
  - PASS: save loaded, player 1 has a character.
  - PASS: player 2 empty before the join.
  - PASS: player 2 joined through the network (character set, controller type 3) after the guest's bot pressed A/START on the *guest's* pad, and the host's bot confirmed with Enter.
  - Packets: host received 3389 inputs, 0 bad, round trip about 0 ms on localhost.
  - `RESULT: PASS (3/3 checks)`.
- **Character position (observed):** walking player 2 with the guest's stick changed floats at character `+0x20` (26.0 → 87.5) and `+0x24` (-290.6 → -650.6). These are probably the ground-plane X and Z (derived). Height and facing: not found yet.
- Owner files unchanged across the run.
- **Not done:** the Steam P2P transport; and a guest view of the host's world (the guest's own copy sat at its main menu in this test, so the guest player still can't see the game).

### Runs 15 to 18 (2026-10-24): world sync stage 3, the guest mirrors both player characters
- **DLL stage 3** (`sync.c`):
  - Per-frame hook on `steam_api!SteamAPI_RunCallbacks`, which the main loop calls once per pass (about 14,000 calls in a 90 s test).
  - Host, every 2nd frame: walks the human-character list (`*(game+0x510)+0x1c`, nodes `{item, next}`, read without touching the game's own cursor at `+8`). For each character it sends id `+0x78`, position `+0x20..+0x28`, facing `+0x2c..+0x34`, health `+0x54`, and which real player controls it.
  - Guest: keeps the newest snapshot (network thread) and applies it on the main thread through the game's own setters: entity vtable slot 2 (position) and slot 3 (facing), plus the physics body `+0x68` (slot 2) and its `+0x94` update, the same calls `TM_SetPlayerPos`/`TM_SetPlayerView` make.
  - Guest: its real player 0 switches to the character the guest plays on the host via `FUN_0044b7e0(character)` (thiscall on the real player; it sets `+0xc` and the human flag `+0x84`), retried every 60 frames.
    - `FUN_0044b3b0` is not a switch: it only moves the `+0x84` flag and the teammate link. It returns early while story byte `0x1c` is 1..9 (derived: a scripted-scene lock).
  - Guest: the local game sees **no** XInput pads (the guest's pad is read only to send it), so the friend's controller can't drive their own copy's menus. In run 15, START on the guest pad had opened the notebook on both copies.
- **The join button is START.** Run 16 probe, one button at a time, measuring how much the screen centre changed: A 4.3, B 6.3, X 7.1, Y 6.6, BACK 6.9, START 69.6. "Player 2 joins the game" appeared after START. Pressing START again opens the notebook (Map / Characters / Clues).
- **Run 15 was invalid.** Both copies sat in the notebook, so positions "matched" at 0.0 without anyone moving. The test now also checks that both players moved on the host.
- **Run 18, `local/runs/mirror_test.py`: RESULT: PASS (7/7 checks):**
  - Both copies loaded the owner's save with the same human characters (ids 2, 4, 5).
  - Player 2 joined on the host (guest START, host Enter).
  - The guest's player 1 controls the guest-played character, id 4 (switched on try 1).
  - Host player 1 (id 2) moved 586 units with W/D/S/A; host player 2 (id 4) moved 288 units with the guest's stick.
  - Guest-to-host position error: worst during walking 4.4 (id 2) and 3.2 (id 4) at 4 Hz sampling, final 0.0 for all three.
  - 3617 input and 5214 state packets, 0 bad.
  - Screenshots `local/runs/run18_host.png` and `run18_guest.png` show the same scene, with the guest's camera on its own character.
- **Not synced yet:** animation state (the guest's puppets were only seen standing in screenshots; walk animation unknown), enemies (`CPlayerPNJ*`), the 161 story flags, doors/items/pickups, level changes, health effects, and the initial state (both copies loaded the same save by hand).
- Owner files unchanged in every run.

### Runs 19 to 22 (2026-10-24): animation sync for human characters
- **Movement state** (run 19, host alone, player 1): character `+0x60` = 10 standing, 30 running (default when moving), 20 walking (RIGHT SHIFT held); `+0x230` changes with it. Also changing while moving:
  - `+0xd8`: 0 → 1.
  - `+0xf0`: 1 → 6, probably the locomotion animation set (derived).
  - `+0x238/+0x23c` and `+0x244/+0x248`: the move direction (0.005, 1.0 for W).
  - `+0x250`: 0 → 1000.0.
  - `+0xb8`/`+0xbc`: probably speed (396).
  - `+0x88` holds pointers (action-state objects), not flags.
- **Setting the state directly doesn't stick** (run 20): `FUN_00436bd0(state)` (thiscall on the character, as `TM_SetPlayerState` calls it) ran 347 times on the guest. Its state stayed 10, with host/guest agreement 1/48.
- **Where it's decided** (static):
  - `CPlayerHuman` vtable `0x6449e4`, slot 1 = `Update(dt)` (`FUN_0043f6e0`).
  - Update scales the move direction `+0x238..+0x240` by `+0x250`, runs `FUN_0043a650` (turning and facing), then `FUN_0043bf60`, the state machine. All the 30/20 and most of the 10 transitions are there.
  - In state 10 it switches on **`+0x230`, the requested state** that the brain (player controller or AI) writes earlier in the frame.
- **Stage 3b:**
  - The host sends `+0xd8`, `+0x230` and the block `+0x238..+0x254` per character.
  - The guest patches `CPlayerHuman::Update` (only if the slot still holds `0x43f6e0`). For each mirrored character it writes those fields before the original Update and the host position/facing after it. The game's own state machine and animation then follow the host.
- **Run 22: RESULT: PASS (8/8 checks):**
  - State/animation agreement (`+0x60`, `+0xf0`) for both player characters: 47/48 samples.
  - Positions: worst during walking 4.1, final 0.0.
  - Screenshots `local/runs/guest_running.png` and `host_running.png` taken mid-run: the guest shows both characters in a running stride.
- Run 21, with intent but without `+0x230`: agreement still 1/48. So the requested state is the field that matters.

### Runs 23 to 25 (2026-10-24): levels, story flags, enemies
- **Level ids:** the current level is the byte at `*(game+0x510)+8`. The owner's courtyard save is 193 = `M000`.
  - The id → name table has 111 entries (`FUN_0042c150`: ids at `0x695df8`, name pointers at `0x695e68`). Ids 225..234 are developer test levels (`WLOCK`, `WTEST`, `WCHAIR`, `WHIKARU`, `WDOORS`, `WMORD`, `WCRAB`, `WGRIM`, `WREIN`, `WHOMM`). They are **not** in the shipped archives (no `_levels/w*`). Requesting `WMORD` sent both copies back to mode 1, the loader's failure path in `FUN_00416a70`.
- **Level change** (`FUN_00416a70`): runs when `game+8 == 5` and `game+4 == 2` (in play). It loads `game+0x590` (name) with entry `game+0x5d0`, and keeps the previous level id in `game+0xad0`.
- **Stage 3c (DLL):**
  - **Host:** notices `game+8 == 5` each frame and sends the name and entry (repeated for 2 s). Every state packet carries the level id. The 161 story flags go out every 15 frames.
  - **Guest:** applies the host's request on its main thread (same three writes as `TM_LoadLevel`). It copies the host's flags when in the same level, and mirrors nothing while the levels differ or a change is pending.
  - **Guest:** its `TM_LoadLevel` and `TM_EventChangeLevel` bodies (vtable slot 3 at `0x649660`/`0x64a960`, checked against `0x4ddb80`/`0x4d92a0` first) are replaced by a no-op that succeeds. Only the host changes level.
  - **Test command:** `level <id> <entry>` on the pipe requests a level change the same way.
- **Run 24, `local/runs/level_test.py` 194 (`M001`): RESULT: PASS (7/7 checks).** Both copies were in 193 with identical flags. After the host's request both were in 194 within 5 s; the guest's log shows it followed the host (seq 1). Humans were mirrored in the new level with error 0.0, and the flags stayed identical (574 flag packets applied).
- **Enemies:**
  - The level manager's list `+0x58` holds them. In `C006` (id 52): six `CPlayerPNJMord`, each with `+0x78 = 1` (a type id, not unique), in the same order and at the same positions on both copies after loading.
  - `CPlayerPNJMord` vtable `0x64cafc`: slot 1 Update `0x4d0be0` (it calls the base `CPlayerPNJ::Update` `0x4b7c30`). Slots 2/3 are the same SetPosition/SetDirection as `CPlayerHuman` (`0x430780`/`0x430820`).
  - Base PNJ Update: the entity's position and facing come from the physics body (`+0x68` body `+0x2c`/`+0x38`) every frame. While alive (`+0x58 == 0 && +0x5c == 0`) it runs slots 19, 20 and 22, dead slot 21, and always slot 18.
- **Health and death** (natives):
  - `TM_SetLife` uses vtable slot 6, `AddLife(delta)`.
  - `TM_KillPlayer` sets `+0x54 = 0`, `+0x58 = 1`, `+0x5c = 1`.
  - `TM_IsDead` tests `+0x58 || +0x5c`.
- **Stage 3d (DLL):**
  - The host sends enemies in list order with their vtable (for validation), position, facing, health and dead flags.
  - The guest's own enemy AI keeps running, but every frame each enemy gets the host's position and facing (entity and body) and the host's health through `AddLife`. A death the guest didn't reach is completed with the `TM_KillPlayer` flags.
  - Human health is also the host's, through `AddLife`.
- **Run 25, `local/runs/enemy_test.py` (`C006`): RESULT: PASS (6/6 checks).**
  - Host player 1 steered into the Mordeurs. W/D world directions were probed first: W = (44, -32), D = (-70, 0) per 0.5 s.
  - All six enemies moved 168..311 units on the host. On the guest, final position error 0.0..1.1, worst sample during the fight 85.9.
  - Enemy health matched in 60/60 samples. Player health fell (P1 49.7 → 14.7, P2 100 → 78.9) and the guest showed the host's values (220 AddLife corrections, 0 list mismatches).
  - Both screens showed the game's low-health red tint.
- **Not tested:** an enemy death (the players didn't fight back in this test); enemy animation on the guest (comes from the guest's own AI; its `+0x60` stayed 0 on both copies, so the enemy state field is elsewhere); doors, items, pickups and inventory; cutscenes; giving a joining guest the host's save.

### Runs 26 to 30 (2026-10-24): items, doors, enemy deaths
- **Items:** in the level manager list `+0x30` (same node layout), keyed by type `+4` and id `+8` (`TM_GetIt` 0x4df840); world position at `+0xc..+0x14`.
  - In `M000`: 16 `CItPlayerPos` (type 2, spawn/entry markers), 11 `CItDoor` (type 1; `+0x88` 0, `+0x2c` 1), 1 `CItCDR` (type 162, the save CD), 1 `CItPadlock` (type 915).
  - `+0xfc` is the collected flag only on collectables. Spawn markers and the padlock keep other data there (the padlock a heap pointer).
- **Collecting** (`FUN_00448a00`, the inventory's AddItem): sets item `+0xfc = 1`, calls item vtable slot 35 (0) and slot 18 `SetVisible(0)`, then adds the item to the character's inventory (count `+0x3c`). `TM_ItIsCollected` treats vtable slot 2 as "is collectable".
- **Doors:** `TM_EventOpenDoor` sets `+0x88 = 1` and calls `FUN_004f34d0(ms)` on the door, which accumulates opening time in `+0xfc` up to 2000 and turns the door. `TM_ActivateDoor` writes `+0x2c`.
- **Controls found:**
  - RETURN is the action key (picks up the CD when standing next to it and facing it).
  - SPACE, E, RCONTROL and LSHIFT didn't pick it up.
  - Attacking a Mordeur with RCONTROL, LSHIFT+RCONTROL, LSHIFT+SPACE, SPACE or RCONTROL+SPACE did no damage after teleporting next to it. The attack controls are **unknown**, possibly because no weapon was ready.
  - Keyboard binding order from `tm_input.ipt` (entries every 0x20c bytes): move ×4, …, RSHIFT walk, NUMPAD 8/2/4/6 team orders, RETURN ×2, ESCAPE, LSHIFT, RCONTROL, UP, DOWN, SPACE, BACKSPACE, ESCAPE, RIGHT ×2, LEFT ×2.
- **Stage 3e (DLL):**
  - The host sends every `CItDoor` (`+0x88`, `+0xfc`; the class is recognised by its RTTI name) and every collected collectable, keyed by type, id and vtable, every 6 frames, at most 48 per packet.
  - The guest copies door fields. For a collectable the host has collected, the guest sets `+0xfc = 1` and makes the same two calls as AddItem (slots 35 and 18 with 0), without adding it to any inventory.
- **Test commands:**
  - `teleport x y z [fx fy]`: moves player 1 through the game's own setters, with optional facing.
  - `hurt <enemy index> <delta>`: the enemy's own AddLife (vtable slot 6) on the main thread.
- **Run 30, `local/runs/item_test.py`: RESULT: PASS (5/5 checks).** The host's player 1 teleported next to the CD facing it and pressed RETURN. The CD was collected on the host; the guest marked the same CD collected and removed it from its world. No other collectable differs.
  - Run 28 failed because the character faced away; run 29 because the check also compared the padlock's pointer.
- **Run 30, `local/runs/combat_test.py`: RESULT: PASS (6/6 checks).** In `C006`, three `hurt 0 -7` on the host took Mordeur 0 from 20 → 13 → 6 → 0, and the host's own death logic set it dead. The guest's Mordeur 0 followed through three AddLife calls and died through its own logic (the guest's TM_KillPlayer fallback stayed unused, kills 0). Dead/alive matched for all six.
- **Not synced:** inventories, so far from the host's world only (the CD went into the host character's inventory; the guest's copy of that character didn't get it). Door open/close wasn't exercised by a real door in a test (the `M000` doors are level exits). Breakables and `CItGameState*` objects (screens, elevators) aren't synced.

### Run 31 (2026-10-24): joining, the host's save goes to the guest
- **Saves found:** no remote-storage `FileWrite` happened in any test run (the game only saves at the save menu). `FUN_00416640` (save writer) has one direct caller, `FUN_0053fa60` (the save menu, `param 0` = `game%d.sav`). The `recover.sav` branch (`param != 0`) has no direct caller (unknown when it runs).
- **Stage 3f (DLL):**
  - The cloud-save proxy keeps the bytes of the last `game*.sav`/`recover.sav` the game read.
  - When the host has read a save the guest doesn't have yet, it sends it in 1000-byte chunks, go-back-N: up to 4 chunks per frame from the guest's acknowledged offset.
  - The guest writes the complete file as `game0.sav` in its own co-op save folder (`coop_profile/remote`), never the player's real cloud saves.
  - Limitation: the LOAD screen reads every slot to list it, so with several saves the host sends the last one listed (it resends when the loaded one is read). The test had one save.
- **Run 31, `local/runs/join_test.py`: RESULT: PASS (5/5 checks).**
  - The guest started with no save. After the host loaded its save, the guest had a byte-identical copy (SHA-256 `2f90eee1…`, 22,528 bytes; sent twice: once at the LOAD listing, once at the load).
  - The guest's bot loaded it; player 2 joined; both copies in level 193 with characters mirrored at 0.0 error.
- **Still to do for joining:** load the received save on the guest without a human pressing LOAD, and join while the host is already further than its last save (live sync then corrects level, flags and items, but not inventories).

### Runs 32 to 37 (2026-10-24): Steam transport (in the game's own steam_api.dll), test package
- **Owner (2026-10-24):** the friend owns Obscure on Steam; build the Steam transport.
- **Interfaces** (the game's 2014 `steam_api.dll`, no flat API), called through vtables from C:
  - `SteamUser017` slot 2 `GetSteamID` (CSteamID returned through a hidden pointer, MSVC member-function rule).
  - `SteamFriends013` slot 0 `GetPersonaName`, 3 `GetFriendCount`, 4 `GetFriendByIndex` (hidden pointer), 5 `GetFriendRelationship`, 8 `GetFriendGamePlayed` (FriendGameInfo_t: u64 game id, u32 ip, u16 ports, u64 lobby).
  - `SteamNetworking005` slot 0 `SendP2PPacket`, 1 `IsP2PPacketAvailable`, 2 `ReadP2PPacket`, 3 `AcceptP2PSessionWithUser`, 7 `AllowP2PPacketRelay`.
  - Callbacks: `SteamAPI_RegisterCallback` with a hand-built `CCallbackBase` `{vtable, u8 flags, int id}`, vtable `{Run(param, bool, SteamAPICall_t), Run(param), GetCallbackSizeBytes}` (MSVC reverses the two `Run` overloads). P2PSessionRequest_t = 1202.
  - Slot numbers come from the SDK headers of that interface version (derived). Which are confirmed is listed below.
- **DLL stage 4** (`steam.c`, `net.c` rewritten around a peer key, Steam id or UDP ip:port):
  - All Steam calls run on the main thread from the frame hook (`net_frame`); UDP keeps its thread.
  - The host accepts P2P sessions from Steam friends only, one guest.
  - The guest says hello to `HOST_STEAMID` if set, else to every friend whose game is app 254460, and keeps the one that answers WELCOME.
  - All packets are `k_EP2PSendUnreliable` (the protocol handles loss; every packet is under 1200 bytes).
  - The Steam timeout is 10 s.
  - Settings can come from `obscoop.ini` next to the exe (`[obscoop]`, keys without `OBSCOOP_`); environment variables win.
- **Observed on the owner's PC (one Steam account):**
  - Run 32: the UDP regression after the rewrite passes, `mirror_test.py` 8/8.
  - Run 33: on Steam, host and guest both initialised ("own id valid", persona name readable, a friend count of 55, callback registered without error). The guest was told to call its own account (`HOST_STEAMID=self`, test only). Its HELLO went through `SendP2PPacket` and was read by `ReadP2PPacket` with the right sender id: the host accepted it, 0 bad packets. With one account, whichever process polls first reads all packets (host rx 2171, guest rx 0). So **send/read and sender decoding are confirmed; two-way traffic needs two accounts.**
  - Runs 34 and 35: friends scan in scan-only mode (`OBSCOOP_SCAN_ONLY=1`, contacts nobody): "4 of 55 friends in a game, 0 in Obscure". `GetFriendByIndex`/`GetFriendGamePlayed` return plausible, selective results.
  - Run 36: a guest configured only by `obscoop.ini` linked to a UDP host.
  - Run 37: `start_host.bat` (sets `SteamAppId`, `OBSCOOP_ROLE=host`, `OBSCOOP_NET=steam`) starts the test copy as a Steam host ("steam: ready"). Owner files unchanged.
- **Not confirmed (needs the friend):** the session-request callback firing and `AcceptP2PSessionWithUser`; real two-way traffic, latency and loss over Steam's relay; the guest's real controller via `LOCAL_PAD`.
- **Test package** (`local/package/obscoop-test-0.1.zip`, SHA-256 `a54969e5…`; DLL `2619afbb…`): `xinput1_3.dll`, `obscoop-guest.ini`, `obscoop-host.ini`, `README.txt`. In guest mode the friend's own game sees no pads, so they use the keyboard for menus. While the DLL is installed their own cloud saves are neither used nor touched (co-op saves go to `coop_profile`).

### First session with the friend (2026-10-24, build 0.1): no connection
- **Owner report:** "The save game does not show up on my friends machine"; then "obscoop log file is missing in friends directory"; then "Save data would not show up we did exactly as described".
- **Host log** (owner's test copy, 3 host sessions 19:38..19:52): Steam ready every time, and **0 session requests, 0 packets received**. The guest never reached the host. The save is only sent after a guest connects, so it never went.
- **Friend side:** no `obscoop.log` where expected. Cause **unknown**: the DLL not loading (wrong folder, antivirus), or Windows UAC file virtualization sending an old 32-bit game's writes under `Program Files` to `%LOCALAPPDATA%\VirtualStore\...` (derived possibility; not confirmed).
- **Build 0.2:**
  - The status shows in the window title (host waiting/connected; guest looking for the host with a candidate count / connected / host save received - choose LOAD > 01). So a player can see at a glance whether the mod runs.
  - When the game dir is under Program Files, the log and co-op saves go to `%LOCALAPPDATA%\obscoop\`.
  - Settings are also read from `obscoop.ini.txt` (a hidden `.txt` extension) and the log says which.
  - The host also calls `AcceptP2PSessionWithUser` every 2 s for each friend currently in Obscure, in case callback 1202 never fires.
  - Local check (run 38, UDP, test copies 3 and 2; copy 1 held the owner's live host): `join_test.py` 5/5, and the status lines appeared as designed.
  - The Program Files path logic is **untested** (no copy under Program Files used).
  - Package `local/package/obscoop-test-0.2.zip` SHA-256 `8f184acc…`, DLL `639d938d…`.
- Playbook note proposed: an old 32-bit game under `Program Files` gets UAC file virtualization, so a mod's log or files written "next to the exe" can end up in VirtualStore. Write mod files to a known per-user folder and show status in the game window.

### Second session with the friend (2026-10-24, build 0.2): connected and playable
- **Owner report:** "it works and controllable". The friend's guest found the host through the friends scan ("4 of 22 friends in a game, 1 in Obscure"). It received the save, switched to the guest-played character (id 4, later id 5 when the host's team changed), and mirrored a CD pickup. Round trip 62-78 ms, with patches of about 500 ms.
- **Open bugs reported by the owner:**
  1. On the host the camera follows the remote player; on the guest it stays on the host's character.
  2. The inventory is shared, controllable only by the remote player, and has no per-player inventory or give-item.
- **Observed on the host during the session:** P1 = char 2 (keyboard) with leader flag `+0x84` = 0; P2 = char 5 (pad) with `+0x84` = 1. The join gave the leader flag to player 2.
- **Derived (not verified):** `+0x84` marks the single "leader" character, which drives the room camera and the inventory menu. The guest's switch (`FUN_0044b7e0`) sets real player `+0xc` but does not give the camera to that character.
  - Camera code is around `0x40b000-0x40e000`: CCameraManager vtable `0x642680`; zone owner `+0x184` on cameras; `FUN_0040d130` changes zone owners; recenter is `FUN_0040b5b0`. Which field selects the followed character: **unknown**.
- **Next:** find the camera target field, keep the leader flag on P1's character on the host and on the guest's own character on the guest, and re-test with `mirror_test.py`. Separate inventories plus a give-item feature would be new game design, beyond mirroring.
- Minor: "guest has the whole save" is logged on every duplicate acknowledgement (log spam).

### Runs 39 and 40 (2026-10-24): camera and inventory investigation
- **Camera wiring (static + live):**
  - Each camera follows a real player: `camera+0x180` → real player, whose `+0xc` is the character (`FUN_00408670`).
  - The game object holds one camera list per real-player slot: `game+0x30` and `game+0x44` (`+0x14` apart). A per-slot active camera at `game+0x60`/`+0x64` was 0 here. A per-slot view pointer sits at `game+0x1e8 + 0x238*i`.
  - `FUN_0044c2b0` (a real-player method, called every frame for both slots) **swaps the slots** when the role-100 player's character is missing or dead (`+0x58`) and the other's is alive. Roles: real player `+4` = 100 / 200. It then retargets every camera to the new slot pointers.
- **Leader flag** `+0x84`: after the join it is on player 2's character on the host (both in tests and in the owner's session). On the guest it sits on the host-P1 character.
  - Moving it on the host (WriteProcessMemory, test copy) did **not** change which character the host camera follows.
- **Run 40, host camera checks:** the camera follows player 1 in every case:
  - P2 teleported to another zone (`tp 1 …`, new pipe command);
  - P1 teleported to another zone;
  - P2 walking about 1000 units with the guest's stick, an off-screen marker showing P2.
  - The guest's camera followed the guest's own character (Ashley) in her zone.
  - So the owner's camera reports are **not reproduced**. What triggered them in the live session: unknown.
- **Inventory:** each character has its own inventory menu object (`character+0x88 → +0x10`). `TM_IsInventoryMenuVisible` returns `+0x34` when `+0x1c` equals its argument.
  - On the host both P1's and P2's objects had `+0x1c = 3` (the value of the `Joystick 0` device type). Whether `+0x1c` is an owner device: unknown.
  - SPACE didn't make `+0x34` non-zero, with `+0x1c` 3 or 1. The inventory key is unknown.
- Fixed: "guest has the whole save" is now logged once per transfer.

### Runs 41 to 43 (2026-10-24): inventory key, camera after a team change
- **Owner:** the camera problem was there "from the start"; the inventory opens by **holding Q** and scrolls with the arrow keys. (One-letter key names were missing from the `tm_input.ipt` list because the search needed 2+ characters.)
- **Inventory:** holding Q on the host turns P1's inventory object `+0x1c` from 3 to 1 for as long as Q is held, whatever the leader flag. So `+0x1c` is the inventory mode, not an owner. The item icons show at the screen corners.
  - Arrow keys (host) and dpad-right (guest pad) didn't change anything detectable in the first 0x200 bytes of either inventory object, or in the corner icons. Where the selection lives: unknown.
- **Leader flag on the guest:** moving `+0x84` between the guest's own character and the host's P1 character did not change the guest's camera; it stayed on the guest's own character (Ashley).
- **Team change** (new pipe command `switch <player> <char id>`: `FUN_0044b7e0` on that real player; run 43):
  - Host P2 switched from character 4 to 5; the guest followed ("switch to character id 5 (try 2) -> now 5", the same line as in the friend's log).
  - The guest's camera then showed character 5 (screenshot `local/runs/sw_after.png`).
  - **Not reproduced:** the owner's "camera stays on my player" on the guest, and "remote player controls the camera" on the host.
- **Differences from the live session still untested:** the owner's real controller was connected and visible to the host copy (`start_host.bat` does not hide real pads, and the owner's pad is XInput 0 = `Joystick 0`, player 2's device); real players walking through zone triggers; the friend's own setup.
- **Test setup fixes:** the LOAD list wraps around (UP from slot 01 goes to slot 10); `local/runs/pair.py` now waits for each step. A teleport next to door 1 (`-2159, 108`) is a level exit.
- **Owner file check:** the real `video_cfg.dat` was rewritten at 21:07:16 with **identical content** (SHA-256 `210880db…`, same as the backup). The game writes this file only on a normal exit. The test copies are always force-killed and their logs show only the redirected startup reads. Writer most likely the owner's own launch (to check the inventory key): **unverified**.
