# SaveInstance (Modified) — Full Union & Terrain Support

A modified build of [UniversalSynSaveInstance (USSI)](https://github.com/luau/UniversalSynSaveInstance) that correctly saves **UnionOperations** (CSG `MeshData2` geometry) and **Terrain**, with optional **full-map streaming** and a **parallel decompile prepass** for fast, complete saves of big games.

## Usage

```lua
local Params = {
    RepoURL = "https://raw.githubusercontent.com/RealSlimShady2000/SaveInstanceMODIFIEDFullUnionSupport/main/",
    SSI = "saveinstance",
}

local synsaveinstance = loadstring(game:HttpGet(Params.RepoURL .. Params.SSI .. ".luau", true), Params.SSI)()

local Options = {
    safemode = false,
    SetStreaming = true,     -- force-load the entire StreamingEnabled map first (whole map, not just nearby chunks)
    DecompilePrepass = true, -- decompile all scripts in parallel before saving (big speedup on script-heavy games)
    -- Full option list: https://luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance
}

synsaveinstance(Options)
```

`SetStreaming` and `DecompilePrepass` are **off by default** — turn on the ones you need. It all runs from this one call; no separate scripts required.

## What's different from the normal save instance

- **Unions render correctly.** Re-enabled `gethiddenproperty` during the save (it was being disabled unconditionally) so the union's `MeshData2` is actually read, and stopped `IgnoreSharedStrings` from dropping the `MeshData2` / `ChildData2` SharedStrings. On executors that can't read union mesh data, unions fall back to a visible bounding-box Part instead of being invisible.
- **Terrain** `SmoothGrid` / `PhysicsGrid` are serialized.
- **`SetStreaming`** — force-loads an entire `StreamingEnabled` map before saving so the whole map is captured (see below).
- **`DecompilePrepass`** — decompiles every client script in parallel before saving so the save's decompile step is near-instant (see below).
- **Faster, lighter saves** — the file is assembled with a table buffer instead of repeated string concatenation, fixing the `O(n²)` growth that caused "not enough memory" on large games.
- **Executor capability check** — on each save it prints (console) which required functions your executor is missing (`gethiddenproperty`, `decompile`, `request`, `sethiddenproperty`, …) and the impact, so you know up front why a save might be incomplete.
- **`UseUGCValidationService`** (default `true`) — set `false` to never use `UGCValidationService` as a hidden-property fallback (some executors flag or lack it); `gethiddenproperty` is still used.
- A progress bar accompanies the on-screen save status.

> Note: `ChildData` is `NotReplicated`, so client-side saves render unions but can't make them editable/separable in Studio.

## Map streaming — `SetStreaming`

Many big maps use **StreamingEnabled**, which only loads the chunks near your character — so a normal save captures just a fraction of the map. With `SetStreaming = true`, the tool pins all loaded content (`ModelStreamingMode = Persistent`) so nothing streams back out, then sweeps the whole map firing **concurrent** `RequestStreamAroundAsync` requests (each yields when its region loads — faster than teleport-and-poll). Runs headlessly with progress in the status bar, then saving begins automatically. Expect lag on large maps.

Tunables (defaults): `StreamingAreaSize` (10000), `StreamingRadius` (1024, auto-detected when possible), `StreamingConcurrency` (8), `StreamingSlices` (2), `StreamingTimeout` (20), `StreamingMaxTime` (120 — overall seconds cap; the sweep gives up on stuck/empty chunks and proceeds, so it never hangs on ocean-heavy maps). Based on / speeds up [centerepic/Streamer7](https://github.com/centerepic/Streamer7).

## Decompiling — lua.expert fallback + `DecompilePrepass`

If your executor has **no decompiler** (e.g. Volt), scripts are automatically decompiled via the **[lua.expert](https://lua.expert)** free API instead of being saved as "no decompiler" stubs. Decompiled scripts are credited to lua.expert in their header.

`DecompilePrepass = true` makes this fast on script-heavy games: every client script is decompiled **in parallel** via the API and cached *before* saving, so the save's per-script decompile becomes an instant cache hit. Without the prepass, scripts are decompiled on demand during the save (slower, but still works).

> Heads-up: this sends script bytecode to a **third-party API** (`api.lua.expert`) and needs `getscriptbytecode` plus an HTTP function (`request` / `http_request`). `DecompilePrepass` is **off by default**. Tunables: `PrepassConcurrency` (24), `PrepassTimeout` (20), `PrepassRateGap` (0.12 — min seconds between API requests; lower = faster if the API allows, raise if you get failures), `PrepassApiUrl`. Based on [centerepic/ussiprepass](https://gitlab.com/centerepic/ussiprepass); decompiler API by [lua.expert](https://discord.com/invite/y63m4zUYa4).

## Credits

All credit for the original SaveInstance goes to the **luau** project — please support them:

- Project home & docs: **[luau.github.io](https://luau.github.io/)**
- Source: [luau/UniversalSynSaveInstance](https://github.com/luau/UniversalSynSaveInstance)
- API reference: [luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance](https://luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance)

Streaming & prepass approaches by [centerepic](https://github.com/centerepic) ([Streamer7](https://github.com/centerepic/Streamer7), [ussiprepass](https://gitlab.com/centerepic/ussiprepass)).

Decompiler API: **[lua.expert](https://lua.expert)** — free decompiler ([Discord](https://discord.com/invite/y63m4zUYa4)).

Modified by **Robloxscripts.com** — Discord: discord.robloxscripts.com
