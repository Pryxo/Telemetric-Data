# Telemetric-Data

Shared data for the [Telemetric](https://github.com/Pryxo/Telemetric) mod (Telos Realms). Everything here is
meant to be **cached by the mod on start or downloaded on demand**, and is the same for every player.
The mod reads files from `https://raw.githubusercontent.com/Pryxo/Telemetric-Data/main/<path>`.

## Rules
- **Mostly JSON.** Tables, lists and lookup data are `.json` (UTF-8, 2-space indent, one topic per file).
  Binary data only where JSON is impossible (Voxy LOD zips). Every file under 100 MB, no Git LFS.
- **Shareable info only** — data that is useful to other clients: location names, dungeon layouts, piece sizes,
  item/mount tables, prebuilt maps.
- **No sensitive info. Ever.** No account names, UUIDs, emails, IPs, tokens, API keys, session ids, chat logs,
  or anything from a player's own stash/inventory. If unsure, leave it out.
- Keep a file's format stable; when it must change, add a `"version"` field and bump it.

## Layout
| Path | What |
|---|---|
| `LODs/index.json` | List of prebuilt Voxy maps |
| `LODs/<dungeon_name>.zip` | Voxy LOD store of one fixed-layout dungeon |
| `telos/bosses.json` | Bosses (label, type, boss-bar name hash) |
| `telos/rarities.json` | Item rarity colours, keyed by the last part of the tooltip style |
| `telos/bags.json` | Loot bag types (drop totem model, drop sound) |
| `telos/*.json` (others) | Minibosses, dungeons, portals, items, companions, classes, season pass, traits |

The Telos tables in `telos/` (except `rarities.json` / `bags.json`) come from
[Melinoe](https://github.com/House-Hades/Melinoe) (House-Hades, LGPL-3.0), used with the authors' permission.
Telemetric bundles a copy of `telos/` in its jar and downloads the newest one on start, so an edit here reaches
every player on their next game start. Players can override a file locally in `config/telemetric/data/telos/`.

## Voxy maps
Used by Telemetric's **Visual → Voxy Modifier**. Every instance of a whitelisted dungeon shares one Voxy
store, key `telos:<dungeon_name>` (HUD name lower case, non-letters → `_`, e.g. `Rustborn Kingdom: Part I` →
`telos:rustborn_kingdom_part_i`), saved under `.voxy/saves/telosrealms/<world-id>/` for every Telos address.

`LODs/index.json`:
```json
{
  "version": 1,
  "maps": {
    "telos:goblin_lair": { "file": "LODs/goblin_lair.zip", "version": 1, "sha256": "<hex of the zip>", "size": 12345 }
  }
}
```
To add a map: play the dungeon with the module on, quit the game, zip the **contents** of its `<world-id>` folder
(`storage/` at the zip root, without `LOCK` and `LOG*`), add the zip and an index entry. Bump `version` to update
a map — clients replace their copy only if it came from here (a player's own LODs are never overwritten).
