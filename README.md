# Telemetric-Data

Shared data for the [Telemetric](https://github.com/Pryxo/Telemetric) mod (Telos Realms). The mod caches these files
on start or downloads them when it needs them, and they are the same for every player.
It reads them from `https://raw.githubusercontent.com/Pryxo/Telemetric-Data/main/<path>`.

## Rules
- Mostly JSON. Tables, lists and lookup data are `.json` files (UTF-8, 2-space indent, one topic per file).
  Binary files only where JSON can't work, which so far means the Voxy LOD zips. Keep every file under 100 MB and
  don't use Git LFS.
- Only put things here that other clients can use: location names, dungeon layouts, piece sizes, item and mount
  tables, prebuilt maps.
- Nothing sensitive, ever. That means no account names, UUIDs, emails, IPs, tokens, API keys, session ids, chat
  logs, and nothing from a player's own stash or inventory. If you're not sure, leave it out.
- Don't change a file's format once it's out, because old jars download the new files too. There is no version
  field; the mod notices updates through the GitHub ETag.

## Layout
| Path | What |
|---|---|
| `LODs/index.json` | List of prebuilt Voxy maps |
| `LODs/<dungeon_name>.zip` | Voxy LOD store of one fixed-layout dungeon |
| `tables/*.json` | Telos tables the mod reads. All of them are written by hand; the fields are listed below |

Telemetric ships a copy of `tables/*.json` inside its jar. Once per game start it asks this repo whether a file
changed (by ETag, so nothing is downloaded if it didn't). An edit here reaches every player the next time they start
the game. Players can override a file locally in `config/telemetric/data/overrides/tables/`.
The mod only reads the fields listed here, so leave everything else out: no descriptions, sources or notes in the
files.

## Tables
### `tables/Boss.json`
The root has `portalSeconds` (default portal time of tracked bosses), `aliases` (in-game place name → dungeon name,
e.g. `"The Aviary": "Aviary"`) and `bosses` (a list). Each boss has:

| Field | Needed for | What |
|---|---|---|
| `name` | always | Boss name as the game shows it. Use `"???"` for a dungeon boss we don't know yet |
| `dungeon` | dungeon bosses | The dungeon it's fought in (the name from the HUD location box). Multi-part dungeons can name the part (`"Rustborn Kingdom: Part II"`); the Pity Counter then shows only that part's bosses, and the last boss of each part counts as a run. No `dungeon` means it's a realm boss |
| `zone` | realm bosses in a zone | `Shadowlands` or `Radiant Isles`. The Boss Tracker and Pity Counter only list the boss there. No `zone` means the open realm |
| `bossBarHash` | Boss Bar Scale, "which boss is up" | Hash of its boss bar name (one number or a list) |
| `positions` | Pity Counter, Boss Tracker | Spawn spots `{ "x", "y", "z" }`, optionally with a `name` |
| `areas` | Pity Counter | Realm bosses only: HUD location names (the bottom-right box, a string or a list) where its pity shows when no boss is near and no boss bar is up |
| `tracker` | Boss Tracker | `call` / `spawn` / `defeat` (regex of a chat line), `idle`, `progress` (true/false), `portalSeconds`, `waypoints` (spots) |
| `icon` | Boss Tracker | Item model of its icon (`telos:material/boss/anubis`) |
| `pityOnly` | multi-boss dungeons | Names of the only drops that a kill of this boss adds pity to (leave it out and all its drops count) |
| `drops` | Pity Counter, drop lines | List of drops (see below). In a multi-boss dungeon, list the earlier bosses first and the final boss last |

Each drop goes on its own line and has:
- `name`
- `chance`: % per kill
- `id`: the item id, like `telos:weapon/bow/ut-blunder`. Without an id the drop is a material, matched by name
- `rarity`: `irradiated` … `unholy`, or the companion rarity
- `shiny: true` if it has a shiny version. That version's id is the id + `-shiny`, and its chance is `chance`
  divided by the divisor in `shinyChance.json`
- `mob`: the named mob (or a list of them) that drops it instead of the boss. Their kills add the pity
- `kind`: only needed when the id doesn't already tell. An id with `pet/` is a pet, `mount/` is a mount, any other id
  is an item and no id is a material. Use it for things like `mountSkin`

Only drops with pity belong here. Leave out fragments and drops from random mobs or puzzles.

### `tables/rarities.json`
`rarities` is keyed by the last part of the item's tooltip style (`telos:untiered/bloodshot` → `bloodshot`), plus
`companion` and `shiny` as loot groups. Each entry has `label`, `color` (`#RRGGBB`), `glyph` (chat icon), `icon`
(pouch glyph used in Lifetime Stats), `bag` (the loot bag totem, item model `telos:mob/pouch/<bag>`), `sound` (drop
sound suffix `noise:player.bags.create_<sound>`, defaults to the key) and `group` (Lifetime Stats row, defaults to
the key).

`eventBags` holds bags that aren't a rarity. They use the same fields, with `group` defaulting to `event`. An event
bag that has a `color` can be used as a Boss.json drop `rarity` (e.g. `"rarity": "halloween"`), and the Pity Counter
then uses its colour and label on that drop. Pets, mounts and skins without a `rarity` show as Companion.

### `tables/traits.json`
`tiers` lists the tier letters, lowest first (`["D", "C", "B", "A", "S"]`). `weapon`, `ability` and `armour` hold the
unique traits. Each has a `name`, a `template` (the description with `{0}`, `{1}`, … placeholders) and `values`
(for each placeholder, tier → text).

### `tables/shinyChance.json`
Maps a rarity to the divisor of its shiny drop chance. `default` covers every other rarity.

### `tables/chat.json`, `callouts.json`, `realms.json`, `discord.json`
- `chat.json`: chat tab categories, tabs and utility lines.
- `callouts.json`: call-out cooldown, partial boss bars, duos, phases and portal labels.
- `realms.json`: realms and hubs per region. `prefix` is matched against the server name and the first match wins,
  so put the empty prefix last.
- `discord.json`: the Discord application id, art keys, texts, buttons and area lines.

## Voxy maps
These are used by Telemetric's Visual → Voxy Modifier. All instances of a whitelisted dungeon share one Voxy store
with the key `telos:<dungeon_name>`. The name is the HUD name in lower case with every non-letter turned into `_`, so
`Rustborn Kingdom: Part I` becomes `telos:rustborn_kingdom_part_i`. The store is saved under
`.voxy/saves/telosrealms/<world-id>/` for every Telos address.

`LODs/index.json`:
```json
{
  "version": 1,
  "maps": {
    "telos:goblin_lair": { "file": "LODs/goblin_lair.zip", "version": 1, "sha256": "<hex of the zip>", "size": 12345 }
  }
}
```
To add a map, play the dungeon with the module on and quit the game. Then zip the contents of its `<world-id>`
folder (with `storage/` at the zip root, leaving out `LOCK` and `LOG*`), and add the zip plus an index entry.
To update a map, bump its `version`. Clients only replace their copy if it came from here, so a player's own LODs
are never overwritten.
