# Gedonia (2022)

| | |
|---|---|
| Engine | Unity 2019.4.20, Mono backend (Assembly-CSharp.dll), NGUI, Enviro sky, A* Pathfinding, DunGen |
| Versions tested | Steam, app 1114220, build 16727998 (Gedonia.exe SHA-256 B375205374C5355991FE1C91577CEB729AD9FBC2FC6534FE4EF897288A5DF91B), Windows 11 x64 |
| Projects | [Gedonia Co-op](https://github.com/Zerbith/gedonia-coop) (directory entry "Gedonia / Gedonia Co-op"), claim [#214](https://github.com/cranebeaker/coop-anything/issues/214) |
| QA level | automated |
| Last updated | 2026-10-26 |

This is the original Gedonia, not Gedonia 2 (which has official co-op).

## Where things stand

Two or more players can play the open world and the story together over Steam (friends, in-game invites,
relay) or in the home network. Players, enemies, story, conversations, loot spots, world events,
expeditions with their generated dungeons, time and weather are shared. Every feature has an automated
two-client scenario; the owner's first real two-PC sessions over Steam went fine. Status: alpha (0.6.1).

## Existing work

None found before this project (web search, Nexus, this repo's issues and pull requests, 2026-10-24).

## How the game works (for co-op)

- **No netcode.** UNet only appears in the bundled Enviro sky samples. The game ships Steamworks.NET 12
  (SDK 1.42) in Assembly-CSharp-firstpass: classic SteamNetworking P2P with relay fallback, lobbies,
  friends, rich presence; no SteamNetworkingMessages. It runs under its own app id.
- **Singletons everywhere.** `PlayerScript.instance` is referenced about 11,400 times. So each machine
  keeps one local player and other players are puppets: a clone of the player root (`PlayerFinal`) made
  under an inactive holder (so no Awake runs), stripped to Transform/Animator/renderers/
  PlayerModelCustomization, then activated with root motion and animation events off.
- **Appearance** is applied with `PlayerModelCustomization.RefreshPlayerModel(...)`, but it also reads
  the LOCAL player (helmet/cloak visibility from PlayerPrefs, quiver by the local weapon, torch light by
  local stealth). These must be sent and set on the puppet before calling it.
- **Enemies:** `FighterNPCModule` AI runs from `NPC.Update` via `npc.updateFunction`; all damage goes
  through `NPC.Damage(7 args)`. Target search wants the "Player" tag, and an attacking enemy drops a
  target that has neither that tag nor an NPC component, so puppets carry a disabled NPC component.
  Enemies in streamed cells (World Streamer, scenes `MediumStr__x..._z...`) load and unload by distance.
- **Story:** about 980 public methods on `GameManager`, dispatched by name with
  `GameManager.instance.SendMessage(name)` from dialogue answers, mystery confirmations, switch triggers,
  timelines and world events. Quests: `QuestsData.AddQuest/ObjectiveCompleted/ReturnQuest`.
  Dialogue: `DialogueNPCModule.Talk` -> `InterfaceManager.RefreshDialogue` (buttons named by answer
  index, requirements checked on the local player) -> `OnDialogueButton`.
- **World state:** `SaveLoadManager.WorldDatabase` (Dictionary<int,string>) with `SaveableItem`
  ids; containers, ore and herbs are `Container` with loot taken in `InterfaceManager.TakeLootItem`.
- **Saves:** Easy Save 3 files in `%USERPROFILE%\AppData\LocalLow\Oleg Kazakov\Gedonia`, PlayerPrefs in
  `HKCU\Software\Oleg Kazakov\Gedonia`, both named after the Unity product name (see the playbook lesson).
- **Time and weather:** Enviro (`EnviroSky.GameTime`, per-zone random weather rolls).
- **Expeditions:** rolled with UnityEngine.Random per machine; dungeon phases use DunGen with
  `ShouldRandomizeSeed = true` (the stored seed was ignored).
- **Travellers:** `TravellersManager` keeps a virtual position per traveller (46 of them: merchants,
  bards, sages, citizens, rabbits) and spawns the NPC when the local player comes near; each save had
  them in different places.
- Tools: ilspycmd 9.1 for decompiling, BepInEx 5.4.23.5 + HarmonyX, the plugin is compiled with the
  .NET Framework csc that ships with Windows (C# 5).

## What we changed

All in the [Gedonia Co-op](https://github.com/Zerbith/gedonia-coop) plugin; each item has a test scenario
(`scripts/test-local.ps1 -Scenario <name>`).

- **Transport:** `ITransport` with Steam P2P (friends-only lobby, invites from an in-game friend list,
  join requests) and UDP with its own reliable channel (numbered, acked, resent). A mock of the Steam API
  over loopback UDP follows Steam's rules (sessions held until accepted, size limits, loss), so the Steam
  code path is tested with two clients on one PC.
- **Players** (walk, looks, bow): puppet model, animator mirror (dominant layer-0 clip match about 98%,
  Speed parameter error 0), look signature hash checked on both sides, gear/helmet/cloak/mod weapons,
  Steam name, world map/minimap/compass markers.
- **Enemies** (fight, apart, lossy, death): presence reports, host picks one owner per enemy with epochs
  and a hold time; the owner publishes, others mirror and send damage requests; deaths everywhere with
  XP and loot per player. `scripts/check-ownership.ps1` fails if an enemy is owned twice in a row.
  Bug found: an exception on an enemy whose cell had unloaded aborted the rest of an ownership message.
- **Story** (story, latejoin): a Harmony transpiler swaps the `SendMessage` in the answer/trigger sites for
  a dispatcher; the host numbers story calls and every machine runs the same function in that order;
  quest digests repair anyone who fell behind; the world database changes are sent to late joiners.
- **Shared conversations** (dialogue): conversations are announced; players near the NPC join with F;
  clicks are votes; agreement decides at once, otherwise the starter's vote after a configurable time
  (or only agreement); an answer's stat requirement is met if any member meets it, money/items by all;
  only the leader dispatches the story function.
- **Loot** (loot, farloot, chestgold, chestlid): a lock per container from the host, takes broadcast and
  applied by item; containers that are not loaded apply them when they load.
- **World events and expeditions** (event, eventhandover, expedition, expfull): the event owner spawns,
  others build the same enemies with the same ids; expedition lists come from the host, generation is
  seeded with the stored id (also DunGen), every expedition phase type was played through.
- **Companions and summons** (companions, summons): each player owns their followers; others see
  mirrors built from the same Resources prefab.
- **Clock/weather** (clock), **worlds and dungeons** (worlds, dungeon), **travellers** (travellers),
  **sounds** (Harmony postfix on `AudioSource.PlayOneShot/Play`: clip name and path sent, played on the
  puppet or mirror), **pause** (the pause menu no longer stops time in co-op), **respawn crystals**
  (respawn, respawnnorth: the game only has a reload on death, which would turn back the world; in
  co-op the fallen player gets up at a crystal bound with F).

## QA

- `scripts/test-local.ps1 -Transport udp|steam-mock -Scenario <name>` starts two copies of the game
  (the second is hard links; each has its own Unity product name, so saves and settings are separate)
  with a fixed test save, runs timed test steps and bots in both, and logs what each side observed
  (positions, animation, look signatures, ownership, story events, votes) plus screenshots.
- 34 scenarios. Last runs 2026-10-24 to 2026-10-26 on the 0.6.x code: dialogue (5 runs: timer,
  agreement, charisma check covered by the friend, refused gold answer, story function once, leader
  leaving), respawn, pause, travellers, fight with sounds; each with 0 errors in both BepInEx logs.
  Earlier milestones (enemies with 20 % packet loss, expedition phases, late join) are listed in the
  mod repository's history.
- After every run the real saves and registry values were compared with a backup (unchanged).
- Real mouse clicks in the test windows were not automated (desktop control was not used). The owner
  used the menus by hand in real two-PC sessions up to 0.5.x; the shared-conversation buttons (0.6) have
  not been clicked by hand yet.

## How to play

Download the ZIP from the [releases](https://github.com/Zerbith/gedonia-coop/releases), run `Install.cmd`
(finds Gedonia via Steam, backs up saves, downloads BepInEx 5.4.23.5 with SHA-256 check). Main menu:
"Host co-op" (Steam or home network) and "Join co-op"; in game the co-op button in the top bar or F8.
Everyone needs the same mod version.

## Known issues

- Story cutscenes and story teleports move every player.
- Steam's overlay invite may not open when the game is started outside Steam; the in-game friend list
  works.
- Arena, Survival and Dungeon mode (separate main-menu modes) are not tuned for co-op.
- Effects that only exist as visuals on one machine can be silent for the others.
- The "Uncharted Islands" cannot be reached in this game version (no travel route).

## Next steps

- Real-network sessions with more than two players.
- In-game chat (the co-op menu has a placeholder).
- Voiced quest texts (TTS) are planned.
