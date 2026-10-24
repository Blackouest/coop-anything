# Obscure (2004) online co-op: triage notes and plan

Intended location in your fork: `games/obscure/NOTES.md`
Status: Phase 0 (triage) done from public sources. Nothing installed, nothing run.

## Game identity (AGENTS.md rule 5)
- Title: Obscure (Hydravision Entertainment), original 2004 game. Obscure II / The Aftermath is a separate target.
- Platform target: PC (Steam re-release, 2014). Steam app page referenced in community links: app 254460.
- Store: Steam (owner confirmed 2026-10-19).
- Build id, SHA-256 of main exe, exact OS: **unknown, record before any work**.
- Obscure II / The Aftermath: wanted later, as a separate game entry. Not part of this scope.

## Observations (sourced)
- Local 2-player drop-in co-op exists; the second character is an AI buddy a human can take over. (Co-Optimus, Giant Bomb)
- No online or LAN co-op listed for the PC version. (Co-Optimus, co-op.gg)
- Steam re-release added 16:9 and controller support. (Bloody Disgusting, Co-Optimus)
- Not in coop-anything DIRECTORY.md or games/ as of 2026-10-19.
- Wikipedia infobox lists engine as RenderWare.

## Unknowns (do not guess)
- Exact engine and version in the Steam build (check DLL names, exe version info, file extensions).
- Whether any netcode or debug symbols remain in the exe.
- How the game stores state: character roster, deaths, inventory, puzzle and door flags, saves.
- Whether the 2014 Steam build differs from the 2004 PC build in ways that matter for hooking.
- Whether existing community tools (modding, trainers, texture/model extractors) exist for the format.
- Whether anyone has already started an online co-op attempt (check issues/PRs on coop-anything, forums, Discord).

## Difficulty rating
Closed binary, no engine multiplayer, no scripting: "very hard" per playbook/triage.md. Scope is small though: 2 players, one human-controlled buddy slot already exists in the game.

## Plan
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

## Rules to keep in mind (AGENTS.md)
- Ask the owner before installing software, large downloads, using accounts (Steam, Discord, GitHub) or publishing.
- Keep private paths and state in `local/`. No game files, cracks or decompiled source in any PR.
- Stop at the first contradiction between the game and these notes.
- Open a "claim a game" issue on coop-anything before starting real work.

## Sources
- https://github.com/cranebeaker/coop-anything (README, AGENTS.md, DIRECTORY.md, playbook/triage.md, playbook/engine-families.md)
- https://en.wikipedia.org/wiki/Obscure_(video_game)
- https://www.co-optimus.com/game/304/PC/Obscure__The_Aftermath.html
- https://www.co-optimus.com/article/11518/can-t-see-the-obscure-collection-for-the-steam.html
- https://bloody-disgusting.com/news/3279943/obscure-series-coming-to-steam-with-new-features/
- https://giantbomb.com/wiki/Franchises/Obscure

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
