# Telemetric-Data

Shared data for the [Telemetric](https://github.com/Pryxo/Telemetric) mod (Telos Realms). The mod caches these files
on start or downloads them when it needs them, and they are the same for every player.
It reads them from `https://raw.githubusercontent.com/Pryxo/Telemetric-Data/main/<path>`.
## Telos Map (`tables/map.json`, `maps/*.png`)
One entry per map region: `id` (stable key), `label` (tab name), `zone` (the `Boss.json` zone, `""` = open realm) and
`tiles`. A tile is one PNG (`file`) stretched over the world blocks `x1..x2`, `z1..z2` (inclusive); `sha256` is the
lower-case hex of the PNG (the mod re-downloads when it changes). To update a map: replace the PNG, put its new
`sha256` in the table, push.
