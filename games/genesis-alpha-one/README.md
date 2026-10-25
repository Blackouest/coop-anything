# Genesis Alpha One (2019)

| | |
|---|---|
| Engine | Unreal Engine 4.23 (`++UE4+Release-4.23-CL-0`) |
| Versions tested | Epic Games Store, build files dated 2022-07-26, `Genesis-Win64-Shipping.exe` SHA-256 `e009335b33199dd5dddc37213823b12180076aa71b9b7e0da5f144af319d953c` |
| Projects | [genesis-alpha-one-coop](https://github.com/Blackouest/genesis-alpha-one-coop) (UE4SS Lua prototype, this write-up) |
| QA level | none (manual two-instance runs on one PC only, no automated suite yet) |
| Last updated | 2026-10-25 |

## Where things stand

Not playable yet. On one PC, a host and a guest can be in the same ship: the guest joins over UE's own
netcode, gets the host's ship layout copied over by the mod, both screens render, the guest walks around
and players collide. The guest cannot use weapons, the map/builder, lockers or terminals yet: that code is
single-player Blueprint that runs on the guest without authority.

## Existing work

None found. The developers said there are no multiplayer plans
([Steam thread](https://steamcommunity.com/app/712190/discussions/0/3809532340214341715),
[playco-opgame](https://playco-opgame.com/genesis-alpha-one/)). Searched: web search, this repo's
directory, issues and PRs (2026-10-25). Claim issue: #199.

## How the game works (for co-op)

- **Netcode is compiled in.** IpNetDriver, IpConnection, NetDriver, DemoNetDriver and OnlineBeaconHost
  classes exist. `GameEngine.NetDriverDefinitions` still points `GameNetDriver` at
  `OnlineSubsystemSteam.SteamNetDriver`, which is **not** shipped in the Epic build. Rewriting that
  definition in memory to `/Script/OnlineSubsystemUtils.IpNetDriver` and running
  `open <current map>?listen` gives a working listen server; `open 127.0.0.1` joins it.
- **Console commands from Lua:** `PlayerController:ConsoleCommand` fails under UE4SS 3.0.1 here;
  `KismetSystemLibrary.ExecuteConsoleCommand` works.
- **The ship is ~31 runtime level instances** (`/Game/Maps/Structure/*`, `LevelStreamingDynamic`,
  created from the save by `LoadLevelInstance`). UE does not replicate those to clients. Names are
  `<Short>_LevelInstance_<N>` from a per-process counter, so a guest that loaded the same save first
  and then loads the host's list in the same order gets **identical names** (verified 31/31). 4.23 has
  no `OptionalLevelNameOverride` (checked in the exe strings).
- **Per-player helpers:** each player gets `BP_PlayerRemoteCamera_C`, `BP_PlayerInventoryAvatar_C`
  (parked at z=600000) and `BuilderCharacter_C` (z=-199900). Duplicates are not what breaks the view.
- **The HUD owns the fade.** `PlayerHudWidget_C` contains `Border FadeOutBorder` (opaque black) and a
  nested `LoadingScreen_Widget_C`. Several HUD copies stay in memory after map reloads.
- **Player init is single-player:** `Eden_PlayerCharacter_C` has `IsPlayerInitialized`,
  `BlockActionFPSPlayerInput`, `PlayerMovementSpeed` and BP functions such as `InitializePlayer`,
  `OnLevelLoadedInitializePlayer`, `InitWeaponsFromSave`, `SpawnBuilderCharacter`.
- **Saves and config** live in `%LOCALAPPDATA%\Genesis_EGS\Saved`, shared by every copy of the game.
  A test copy writes into the owner's save folder; back it up first.
- DLC content only loads when started from the Epic launcher; a copy started directly shows
  "connection failed" and runs without DLC.

## What we changed

All in the UE4SS Lua mod `CoopProbe` of the project repo; each item was seen working in a two-window run.

| Problem | Root cause | Fix |
|---|---|---|
| Listen server cannot start | `GameNetDriver` -> SteamNetDriver, missing in the Epic build | rewrite NetDriverDefinitions to IpNetDriver in memory before `open ?listen` |
| Guest is in an empty world | runtime level instances are not replicated | host writes PackageNameToLoad + transform of each streaming level; guest calls `LoadLevelInstance` in the same order (same-PC file hand-off for now) |
| Guest stuck on "CHARGEMENT" | single-player loading flow never finishes on the guest | remove `LoadingScreen_Widget_C` |
| Both screens fully black after a join (scene still rendered: engine HighResShot showed the normal view) | `PlayerHudWidget_C.FadeOutBorder` left opaque by the join, on host and guest | hide it when any copy stays opaque black for 6 s |
| Guest cannot move | guest character never initialized: `IsPlayerInitialized=false`, `BlockActionFPSPlayerInput=true`, speed 0, CharacterMovement inactive, on the host's copy and the guest's own pawn | set those to the host's values and activate CharacterMovement (Walking) on both sides |

Ruled out on the way: PlayerCameraManager fade (also note `StopCameraFade` in 4.23 leaves the fade at
its *target*), `SetEnableWorldRendering`, post-processing, view targets, duplicate helper actors.

## QA

None automated yet. All runs: two instances of the test copy on one PC, `127.0.0.1`, keys F9 (host)
and F8 (guest), logs per instance in `Mods/CoopProbe/probe_HHMMSS.log`. Lab notes with every run:
[NOTES.md](NOTES.md).

## How to play

Not playable. Developer instructions in the project README.

## Known issues

- Guest cannot take weapons, open the map/builder, or use lockers and terminals.
- Guest sees the host's weapon floating: the player mesh is first-person only, no third-person body.
- Door states are not replicated (closed on the guest, walkable).
- Parts of the ship are missing on the guest (not identified yet).
- Ship layout is passed through a file, so host and guest must be on the same PC for now.
- Intermittent crash at startup (3 of roughly 20 launches on 2026-10-25), cause unknown, no UE4SS frames in the stack.

## Next steps

1. Run the game's own init for the guest (`OnLevelLoadedInitializePlayer` / `InitializePlayer`,
   `InitWeaponsFromSave`) on the host instead of copying values.
2. Guest-to-host messages without new RPCs (an existing reliable string RPC hooked on the host), then
   send the ship layout over it instead of a file.
3. Forward the guest's input actions (`InpActEvt_*` Blueprint functions) to the host and run them on
   the guest's pawn; deal with `GetPlayerController(0)` assumptions (impersonation window, see
   [the-doors-of-trithius](../the-doors-of-trithius/README.md)).
4. Third-person body, door sync, Steam relay (the Epic build has no steam_api64.dll), automated
   two-client tests with the repo's tools.
