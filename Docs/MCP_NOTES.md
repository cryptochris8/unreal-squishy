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

## ⚠️ HARD RULE — player input must use LEGACY key-event nodes, NOT Enhanced Input events

**Do NOT bind gameplay input via `EnhancedInputAction` event nodes when building through MCP.** They do not fire. Confirmed by UE 5.8 engine-source analysis + the proven Gnarly project (2026-06-21, after a long debugging session):

- An `EnhancedInputAction IA_X` event node's runtime binding is generated **only during a full Blueprint compile**, by `K2Node_EnhancedInputAction::ExpandNode()`, from the node's internal `InputAction` property + a connected trigger pin. A node made via `create_node`/MCP doesn't reliably populate that binding, so the event **silently never executes** — on the pawn OR the controller. (`IMC`/subsystem/config were all correct; mouse-look worked; the action event just never fired.)
- The DSL can't author Enhanced Input event **bodies** anyway (`write_graph_dsl` "cannot recreate Enhanced Input event nodes"), and IMC mapping arrays don't round-trip through `ObjectTools` (`get_properties` reads `IMC_Default.Mappings` as `[]`; `set_properties` **replaces** the whole array and rejects ambiguous size+content changes).

**✅ The working recipe (playtested):** legacy key-event nodes wired straight to a function — no `IA_`, no `IMC`, no subsystem:
1. Author the logic as a Blueprint **function** via `write_graph_dsl` (e.g. `OnSquishPressed`).
2. `create_node` a legacy key event: `Input|MouseEvents|LeftMouseButton` or `Input|KeyboardEvents|<Key>` (e.g. `…|F`). Outputs: `Pressed`/`Released`/`Key`.
3. `create_node` `CallFunction|<YourFn>`.
4. `connect_pins` the key event's **`Pressed`** (output index 0) → the function call's **`execute`** (input index 0). One exec input accepts multiple key events.
5. Compile + save. These fire through the `EnhancedInputComponent` (backward-compatible) regardless of the project's Enhanced Input defaults.

*(Camera/movement from the ThirdPerson template still work via the template's own Enhanced Input setup — only OUR custom inputs use legacy key events.)*

## Verified MCP/DSL gotchas (this project)
- **`call_tool`**: pass the **bare** `tool_name` + separate `toolset_name` (fully-qualified name → "Unknown tool").
- **Components on a Blueprint**: `PrimitiveTools.add_sphere/cube/...` on the **CDO** (`Default__BP_*_C`) persists to the SCS (reads `None` on the CDO but a spawned instance has the real component).
- **Graph DSL**: custom events need `add_event` first, then `(event Custom|Name …)`. Bool var `bX` → accessors `Get/SetX` (the `b` is stripped). A multi-exec node (Cast, latent) **terminates** the enclosing exec flow — put must-run logic before it. Multi-output bind works: `(bind (a b … hitActor) (Collision|BreakHitResult hit))`. `write_graph_dsl` replaces the WHOLE graph.
- **`create`** makes Blueprints only — for `InputAction`/other asset types use `AssetTools.duplicate` of an existing asset (creating an `InputAction` via `create` pops a modal that **hangs the MCP server** until dismissed).
- **Class-ref properties** (`DefaultPawnClass`, `GameStateClass`, `DefaultGameMode`, `PlayerControllerClass`) set via `ObjectTools.set_properties` with the `…_C` path string.
- **Bulk ops**: `ProgrammaticToolset.execute_tool_script` (call `get_execution_environment` first) — used to import all 51 FBX in one call.
- **`StartPIE`** may report "PIE ended before warmup" while PIE is actually running (false negative — check `IsPIERunning`).

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
