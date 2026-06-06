# SaveInstance (Modified) — Full Union & Terrain Support

A modified build of [UniversalSynSaveInstance (USSI)](https://github.com/luau/UniversalSynSaveInstance) with fixes for correctly saving **UnionOperations** (CSG `MeshData2` geometry) and **Terrain** (`SmoothGrid` / `PhysicsGrid`).

## Usage

```lua
local Params = {
    RepoURL = "https://raw.githubusercontent.com/RealSlimShady2000/SaveInstanceMODIFIEDFullUnionSupport/main/",
    SSI = "saveinstance",
}

local synsaveinstance = loadstring(game:HttpGet(Params.RepoURL .. Params.SSI .. ".luau", true), Params.SSI)()

local Options = { safemode = false } -- Docs: https://luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance
synsaveinstance(Options)
```

## What's different from the normal save instance

- **Unions render correctly.** Re-enabled `gethiddenproperty` during the save (it was being disabled unconditionally) so the union's `MeshData2` is actually read, and stopped `IgnoreSharedStrings` from dropping the `MeshData2` / `ChildData2` SharedStrings.
- **Terrain** `SmoothGrid` / `PhysicsGrid` are serialized.
- A progress bar accompanies the on-screen save status.

> Note: `ChildData` is `NotReplicated`, so client-side saves render unions but can't make them editable/separable in Studio.

## Credits

All credit for the original SaveInstance goes to the **luau** project — please support them:

- Project home & docs: **[luau.github.io](https://luau.github.io/)**
- Source: [luau/UniversalSynSaveInstance](https://github.com/luau/UniversalSynSaveInstance)
- API reference: [luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance](https://luau.github.io/UniversalSynSaveInstance/api/SynSaveInstance)

Modified by **Robloxscripts.com** — Discord: discord.robloxscripts.com
