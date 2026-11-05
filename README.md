# LR-Dynamic-Dungeon-Fixes

A standalone Forge 1.20.1 add-on that fixes bugs in [LR's Dynamic Dungeon](https://www.curseforge.com/minecraft/mc-mods/lrs-dynamic-dungeon) using Mixins. It does not modify or contain the original mod.

## What it fixes
- Boss portal not opening after you die in the fight and then kill the boss
- Reconnecting inside the dungeon resets the run and kicks you out
- Red-skull doors staying shut when a mob despawns or is removed
- Dead-end rooms and a failing first path from the start room
- Crash while spawning a room's mobs (`Not a JSON Array: null`) caused by a malformed enemy pool
- Boss rooms with several lock blocks skipping stages
- Unlimited-height Dungeon Pass portals
- Stage Refill crash after the last boss

## What each part fixes
| Class | What it does |
|---|---|
| `FixSavedData` | Saves runId, stage, biome, start room, boss counter, difficulty, pending room objectives and the placed-room list in `<world>/data/lrddfix_state.dat`. |
| `DungeonControllerMixin` | Loads that state on `initialize()`, skips the "reset to stage 1" tail when a saved run exists, clears pools so compat entries stop duplicating, saves after every mutation, wipes stale rooms right after a reset. |
| `ObjectiveControllerMixin` | Tracks room mobs by UUID with a room tag, routes the original defeat callback to it, 5-second reconcile safety net (only after the room has locks), persists locks/enemies, unregisters finished controllers. |
| `FixEvents` | Counts kills and despawns by tag, rebuilds controllers on server start, clears static state on server stop. |
| `BossLockBlockMixin` | Advances the stage at most once per tick. |
| `RoomConnectionBlockMixin` | Adds the missing `break`, 6 attempts, boss-counter refund on failure, door retried about 8 times before becoming a dead end, full logging. |
| `StructureLoaderMixin` + `PlacementFix` | Validates the room's true footprint (the original checked the wrong corner for every rotation except NONE), creates the controller only after a valid placement, clears a band around a forced start room. |
| `RoomRegistry` + `StoneGeneratorBlockMixin` | Records every placed room, wipes stale rooms at reset, and stops the lazy wipe from touching current-run rooms. |
| `EnemySpawnBaseBlockMixin` | Removes malformed entries from enemy pool groups before spawning. |
| `SkyPortalBuilder` + `DungeonPassItemMixin` | Dungeon Pass portal keeps the original design (platform of chiseled stone bricks + a ceiling of any block) but the ceiling must be within `dungeon-portal-max-height` blocks (default 8). The original had no limit and could build a portal hundreds of blocks tall. Also: iterative platform scan, clear error messages, nothing consumed on failure. |

Everything logs with the prefix `[lrdd_fix]` in `latest.log`.

## New features
- Step-by-step refill chat messages
- `terminate-on-exit-last-stage` option for the ground portal after the last boss
- Keep inventory on death in the dungeon (no lost graves)
- Optional full dungeon reset on death / exit / quit / world close

## Configuration
`config/lrdd_fix-common.toml`

| Option | Default | Description |
|---|---|---|
| `dungeon-portal-max-height` | `8` | Max air blocks between the chiseled-brick platform and its ceiling (1 to 64) |
| `terminate-on-exit-last-stage` | `true` | `true`: last-stage ground portal exits the dungeon. `false`: returns to the last stage's start room |
| `keep-inventory-in-dungeon` | `true` | Keep inventory and XP when dying in the dungeon |
| `reset-on-death` | `false` | Reset the dungeon after a death in it |
| `reset-on-exit` | `false` | Reset after a player leaves the dimension |
| `reset-on-quit` | `false` | Reset after a player disconnects inside it |
| `reset-on-world-close` | `false` | Reset on every world / server shutdown |

## Compatibility
- Works with datapack/KubeJS changes to the dungeon's enemy pools, loot and structures.
- Strict Mixin targets (`defaultRequire = 1`): if a future version of the original renames a patched method, the game refuses to start instead of silently misbehaving. Look for an lrdd_fix update first.
- Keep-inventory covers the vanilla inventory and XP, not items in extra slot systems (for example Curios).

## Reporting bugs
Use the issue templates and attach `latest.log`. Every message from this mod starts with `[lrdd_fix]`.

## Credits

**[LinkedR](https://www.curseforge.com/members/linkedr/projects)**, creator of [LR's Dynamic Dungeon](https://www.curseforge.com/minecraft/mc-mods/lrs-dynamic-dungeon). This mod exists only because of their work; some patches replace methods with reworked versions of the original logic, so credit goes to them.

*This is an unofficial add-on and is not affiliated with or endorsed by the original author. Do not report any issue to them with this mod installed*