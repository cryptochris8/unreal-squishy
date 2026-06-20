# MCP_NOTES — Unreal MCP for Squishy Smash UE

Engine-level facts verified live in the sibling Gnarly project (UE 5.8, 2026-06-18). **Re-verify here after bootstrap** and append results to the log at the bottom.

## How it works

- **Plugin:** Epic's official **`ModelContextProtocol`** (Experimental), at `…/UE_5.8/Engine/Plugins/Experimental/ModelContextProtocol`. Shown in Plugins as **"Unreal MCP."** (The other hit, **"MCP Client Toolset," is unrelated — leave it OFF.**)
- **Transport / endpoint:** streamable **HTTP** via Unreal's built-in server, bound to localhost: **`http://127.0.0.1:8000/mcp`** (port `8000`, path `/mcp`).
- **Server start:** `bAutoStartServer` defaults to **false** — the HTTP server does **not** start on its own. Start it either:
  - **Per session:** editor console (backtick `` ` ``) → **`ModelContextProtocol.StartServer`** (optionally `… 8000`).
  - **Persistently:** Editor Preferences → *Model Context Protocol* → *Server* → tick **Auto Start Server**, restart editor.
  - CLI override: `-ModelContextProtocolPort=N`.
- **Confirm listening:** editor log `LogModelContextProtocol: Starting MCP server on port 8000`.
- **Claude Code side:** repo-root **`.mcp.json`** registers `unreal` → that endpoint (`type: http`). **Restart Claude Code** to pick up a new `.mcp.json`, then approve the trust prompt.
- A **`Roblox_Studio`** MCP is also present (global, in `~/.claude.json`) but is **unrelated** — do not use it here.

> Experimental: the plugin/menu/flow may change between UE builds. Settings persist per-project in `EditorPerProjectUserSettings`.

## ⭐ Capability reality (the important part)

- Tools are exposed via **"toolsets."** Out of the box only `AgentSkillToolset` is registered (4 useless-for-us skill tools). **The real capability toolsets ship as separate plugins** under `Engine/Plugins/Experimental/Toolsets/`, **disabled by default**. **`AllToolsets`** is an aggregator that enables them all — **you MUST enable it** (then restart the editor) or you'll only see ~4 tools.
- `bEnableToolSearch=true` → `tools/list` shows 3 meta-tools (`list_toolsets`, `describe_toolset`, `call_tool`); the model discovers/dispatches the rest on demand.
- **`EditorToolset` (C++):** `GetSelectedActors`/`SelectActors`/`GetVisibleActors`/`FocusOnActors`, camera get/set, **`CaptureViewport`/`CaptureEditorImage`** (screenshots), `ScreenCoordsToWorld`, content-browser nav, open/inspect assets, `SearchCVars`, **`StartPIE`/`StopPIE`**.
- **`editor_toolset.*` (Python) — world-building (with `AllToolsets` on):**
  - **`SceneTools`** — `add_to_scene_from_class`, `add_to_scene_from_asset` (**spawn**), `remove_from_scene` (**delete**), `load_level`, `get_current_level`, `find_actors`, `create_level_instance`, `merge_actors`, `trace_world`, outliner ops.
  - `AssetTools`, `BlueprintTools`, `MaterialTools`/`MaterialInstanceTools`, `StaticMeshTools`/`SkeletalMeshTools`, `TextureTools`, `DataTableTools`/`CurveTableTools`/`DataAssetTools`, `PrimitiveTools` (add primitive geo to actors), `ObjectTools` (get/set any UObject property).
  - Domain suites: Niagara, UMG, GAS, PCG, Sequencer, Physics, GameplayTags, ConfigSettings.

**Net:** MCP can build the world here — spawn/delete actors, load levels, create/edit Blueprints/materials/meshes/data-tables/Niagara/UMG, run PIE, capture clips.

## Workflow = MCP-first

Build via `editor_toolset.*` tools where one exists; fall back to Unreal **Python** (`unreal` API) only for gaps; manual Blueprint **graph** work + Claude guidance for logic & feel. Use MCP to inspect, run **PIE**, and capture screenshots/clips.

**Before any mutating op (MCP or Python):** project under Git + current level **saved** + change small & reversible.

> Gotcha: `call_tool` wants the **bare** `tool_name` (e.g. `get_current_level`) with `toolset_name` passed **separately**. Passing the fully-qualified `editor_toolset.toolsets.scene.SceneTools.get_current_level` fails with "Unknown tool."

## Good MCP tasks
Place/spawn friend pads & landmarks · list/inspect Blueprints in `Content/SquishySmash` · set actor transforms · create a material instance for a friend tint · build a greybox land layout · run PIE + capture a clip.

## Bad MCP tasks (never)
"Build the entire game" / "make it AAA" · mass-deleting assets · whole-project C++ conversion · huge Marketplace imports · packaging builds.

## First smoke test (run after bootstrap; mirrors the proven Gnarly round-trip)
1. `list_toolsets` → expect ~50 (incl. `EditorToolset` + `editor_toolset.*`). 
2. `SceneTools.get_current_level` → reads live level path.
3. `SceneTools.add_to_scene_from_asset` `/Engine/BasicShapes/Cube` named `MCP_RoundTripTest_Cube`.
4. `SceneTools.find_actors` "MCP_RoundTripTest" → returns 1.
5. `SceneTools.remove_from_scene` → `true`; `find_actors` again → `[]`. Don't save → zero net change.
6. `EditorToolset.CaptureViewport` + `StartPIE`/`StopPIE`.

## Results log (append here)
| Date | Action | Tool | Result | Notes |
|---|---|---|---|---|
| _pending bootstrap_ | | | | |
