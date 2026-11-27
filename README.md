# Telemetric-Data

Shared data for the [Telemetric](https://github.com/Pryxo/Telemetric) mod (Telos Realms). The mod caches these files
on start or downloads them when it needs them, and they are the same for every player.
It reads them from `https://raw.githubusercontent.com/Pryxo/Telemetric-Data/main/<path>`.
## Telos Map (`tables/map.json`, `maps/*.png`)
One entry per map region: `id` (stable key), `label` (tab name), `zone` (the `Boss.json` zone, `""` = open realm) and
`tiles`. A tile is one PNG (`file`) stretched over the world blocks `x1..x2`, `z1..z2` (inclusive); `sha256` is the
lower-case hex of the PNG (the mod re-downloads when it changes). To update a map: replace the PNG, put its new
`sha256` in the table, push.

## Map markers (`tables/mapMarkers.json`)
Everything the Telos Map and minimap mark — the mod has no marker data of its own.
- `states`: tooltip text per Boss Tracker state (`alive`, `fighting` with `{player}`, `portal` with `{seconds}`,
  `idle`, `defeated`, `unseen` = the tracker hasn't seen it).
- `categories` (legend rows, drawing order): `id`, `label` (legend + settings name), `source` (`markers` = the list
  below, `players` = loaded players), `live` (state + state colour from the Boss Tracker, matched by marker name),
  `shape` (`icon` — a dot if the icon isn't in the pack —, `dot`, `diamond`, `head`), optional `icon` (item model for
  the legend and markers without one), `color` (`#RRGGBB` or a Boss Tracker state: `alive`, `fighting`, `portal`,
  `idle`, `defeated`), `size` (factor), `labelZoom` (names from this zoom in GUI px/block; leave out = hover only),
  `fade` (opacity of inactive markers), `map` / `minimap` (shown by default).
- `markers`: `category`, `name`, `region` (`map.json` id), optional `icon`, `color`, `note` (tooltip line), and
  `spots` (`x`, `y`, `z` block coordinates; a non-live marker is drawn at every spot, a live one at the spot the
  Boss Tracker reports, else the first). A legend row only appears when the region has markers of it (players always).
