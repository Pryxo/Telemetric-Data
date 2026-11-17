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
| `tables/*.json` | Telos tables the mod reads — **all hand-written**, field reference below |

Telemetric bundles a copy of `tables/*.json` in its jar and asks this repo once per game start whether a file changed
(ETag, nothing is downloaded when it did not), so an edit here reaches every player on their next game start.
Players can override a file locally in `config/telemetric/data/overrides/tables/`. Only fields listed here are read —
leave out anything else (no descriptions, sources or notes in the files).

The chat, call-out, realm and Discord tables, Boss Tracker lines, spawn spots and the bag list come from
[Melinoe](https://github.com/House-Hades/Melinoe) (House-Hades, LGPL-3.0), used with the authors' permission.

## Tables
### `tables/Boss.json`
Root: `version`, `portalSeconds` (default portal time of tracked bosses), `aliases` (in-game place name → dungeon name,
e.g. `"The Aviary": "Aviary"`), `bosses` (list). One boss:

| Field | Needed for | What |
|---|---|---|
| `name` | always | Boss name as the game shows it. Dungeon boss not known yet: `"???"` |
| `dungeon` | dungeon bosses | Dungeon it is fought in (dungeon name from the HUD location box). No `dungeon` = realm boss |
| `zone` | realm bosses in a zone | `Shadowlands` / `Radiant Isles` — the Boss Tracker / Pity Counter only list it there. No `zone` = open realm |
| `bossBarHash` | Boss Bar Scale, "which boss is up" | Hash of its boss bar name (one number, or a list) |
| `positions` | Pity Counter, Boss Tracker | Spawn spots `{ "x", "y", "z" }` (optional `name`) |
| `areas` | Pity Counter | Realm bosses: HUD location names (bottom-right box, string or list) where its pity is shown when no boss is near and no boss bar is up |
| `tracker` | Boss Tracker | `call` / `spawn` / `defeat` (regex of a chat line), `idle`, `progress` (true/false), `portalSeconds`, `waypoints` (spots) |
| `icon` | Boss Tracker | Item model of its icon (`telos:material/boss/anubis`) |
| `pityOnly` | multi-boss dungeons | Names of the only drops a kill of this boss adds pity to (none = all its drops) |
| `drops` | Pity Counter, drop lines | List of drops (below). Multi-boss dungeon: earlier bosses first, final boss last |

One drop (one per line): `name`, `chance` (% per kill), `id` (item id `telos:weapon/bow/ut-blunder`; no id = material,
matched by name), `rarity` (`irradiated` … `unholy`, or the companion rarity), `shiny: true` (has a shiny version:
id + `-shiny`, chance = `chance` / `shinyChance.json` divisor), `mob` (the named mob(s) it drops from instead of the
boss — one name or a list; their kills add the pity), `kind` (only when the id does not tell it: id `pet/` = pet,
`mount/` = mount, other id = item, no id = material; e.g. `mountSkin`).
Only drops with pity belong here — no fragments, no drops from random mobs / puzzles.

### `tables/rarities.json`
`rarities`: key = last part of the item's tooltip style (`telos:untiered/bloodshot` → `bloodshot`; plus `companion`
and `shiny` as loot groups) → `label`, `color` (`#RRGGBB`), `glyph` (chat icon), `icon` (pouch glyph, Lifetime
Stats), `bag` (loot bag totem: item model `telos:mob/pouch/<bag>`), `sound` (drop sound suffix
`noise:player.bags.create_<sound>`, default = key), `group` (Lifetime Stats row, default = key).
`eventBags`: bags that are no rarity (same fields, `group` default `event`). An event bag with a `color` can be a Boss.json drop `rarity` (e.g. `"rarity": "halloween"`): the Pity Counter then uses its colour / label, on any drop; without a `rarity` pets, mounts and skins show as Companion.

### `tables/traits.json`
`tiers`: tier letters, lowest first (`["D", "C", "B", "A", "S"]`). `weapon` / `ability` / `armour`: unique traits,
each `name`, `template` (description with `{0}`, `{1}`, … placeholders) and `values` (per placeholder: tier → text).

### `tables/shinyChance.json`
Rarity → divisor of the shiny drop chance (`default` = every other rarity).

### `tables/chat.json`, `callouts.json`, `realms.json`, `discord.json`
Chat tab categories / tabs / utility lines; call-out cooldown, partial boss bars, duos, phases, portal labels; realms +
hubs per region (`prefix` matched against the server name, first match wins, empty prefix last); Discord application
id, art keys, texts, buttons and area lines.

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
