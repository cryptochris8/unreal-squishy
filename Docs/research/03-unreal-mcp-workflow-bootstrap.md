# Unreal MCP Workflow + Bootstrap Plan for `Unreal-squishy` (Squishy Smash)

Research date: 2026-06-19. Source studied: `C:\Users\chris\Unreal-Gnarly` (the working "Gnarly Nutmeg" UE 5.8 + MCP project).
Author: research/tech-lead pass. Goal: (A) document how the Unreal MCP build workflow works, and (B) give an actionable bootstrap checklist to stand up the brand-new EMPTY project at `C:\Users\chris\Unreal-squishy` with the same MCP wiring.

> **The single most important finding:** The "Unreal MCP" is **Epic's built-in engine plugin** `ModelContextProtocol` (Experimental, ships inside UE 5.8 — *not* a third-party npm/python server). It runs an **HTTP MCP server INSIDE the running Unreal Editor** at `http://127.0.0.1:8000/mcp`. Claude Code connects to it purely by a tiny project-level `.mcp.json` that points at that URL. **Nothing is installed into the project** — no `Plugins/` folder, no `Source/`, no C++ module. The plugin is enabled by 2 lines in the `.uproject`. This makes the bootstrap simple: scaffold files on disk, but **the Unreal Editor must be open with the plugin's server started** for any MCP tool to work.

---

## 1. Unreal MCP architecture — how the connection works end to end

```
┌─────────────────┐     reads      ┌──────────────────────────┐
│  Claude Code    │ ─────────────► │  <projectroot>/.mcp.json │   (declares server "unreal", type http,
│  (in project    │                │   url 127.0.0.1:8000/mcp │    url = the editor's HTTP endpoint)
│   root folder)  │                └──────────────────────────┘
│                 │
│                 │   HTTP (streamable) POST /mcp  JSON-RPC: initialize / tools/list / call_tool
│                 │ ───────────────────────────────────────────────────────────►  ┌───────────────────────────┐
│                 │ ◄───────────────────────────────────────────────────────────  │  Unreal Editor (RUNNING)  │
└─────────────────┘                                                                │  ModelContextProtocol     │
                                                                                   │  plugin = HTTP MCP server │
                                                                                   │  on 127.0.0.1:8000/mcp    │
                                                                                   │  + AllToolsets (toolsets) │
                                                                                   └───────────────────────────┘
```

End-to-end chain:

1. **Claude Code** is launched in the **project root** (`C:\Users\chris\Unreal-squishy`). On start it reads the project-scoped `.mcp.json` and sees a server named `unreal` of `type: http`.
2. `.mcp.json` contains only a URL — there is **no command to spawn**. Claude Code just opens an HTTP connection to `http://127.0.0.1:8000/mcp`.
3. That endpoint is served by the **Unreal Editor itself**, via Epic's `ModelContextProtocol` plugin (built into UE 5.8). The plugin uses Unreal's built-in HTTP server, bound to localhost, port `8000`, path `/mcp`.
4. With the `AllToolsets` plugin enabled, the server registers ~50 "toolsets" (the actual editor capabilities) that Claude can discover and call.

### What MUST be running for MCP tools to work

This is the crux. Unlike a stdio MCP server that Claude Code launches on demand, **this server lives inside the editor and does not auto-start**:

| Requirement | Why | How |
|---|---|---|
| **UE 5.8 Editor open** with the Squishy project loaded | The MCP server IS the editor process. No editor = nothing listening on :8000 = `unreal` tools fail/absent. | Open the `.uproject` in UE 5.8. |
| **`ModelContextProtocol` plugin enabled** | Provides the HTTP MCP server. | 2 lines in `.uproject` `Plugins` (see below). Restart editor after first enabling. |
| **`AllToolsets` plugin enabled** | Without it, only `AgentSkillToolset` registers (4 skill-CRUD tools, no editor ops). With it, you get Scene/Blueprint/Material/Editor/etc. toolsets. | 2 lines in `.uproject` `Plugins`. Restart editor after enabling. |
| **The MCP server STARTED** (`bAutoStartServer` defaults to **false**) | The HTTP listener is off until told to start. | Per session: editor console (`` ` `` key) → `ModelContextProtocol.StartServer` (optionally `... 8000`). Persistent: Editor Preferences → *Model Context Protocol* → *Auto Start Server* = ON, then restart editor. CLI override: `-ModelContextProtocolPort=N`. |
| **Claude Code restarted** after `.mcp.json` exists | Claude Code only loads MCP servers at startup. | Quit + relaunch Claude Code in the project root; approve the `unreal` server at the trust prompt. |

**Confirm the server is listening:** editor Output Log shows `LogModelContextProtocol: Starting MCP server on port 8000`.

### Transport / ports (verbatim from the plugin defaults, confirmed in Gnarly's `Docs/MCP_NOTES.md`)
- Transport: **streamable HTTP** via Unreal's built-in HTTP server, bound to localhost.
- Endpoint: **`http://127.0.0.1:8000/mcp`** — host `127.0.0.1`, port `8000`, path `/mcp`.
- Defaults confirmed in plugin source `ModelContextProtocol.h`. Settings stored per-project in `EditorPerProjectUserSettings` (per-user — NOT in tracked `Config/`).
- Tool-search mode `bEnableToolSearch=true` (default): `tools/list` returns only **3 meta-tools** — `list_toolsets`, `describe_toolset`, `call_tool` — and Claude discovers/dispatches everything else on demand through those.

### Where the plugin lives (engine-side, not project-side)
- `…/UE_5.8/Engine/Plugins/Experimental/ModelContextProtocol` (shown in the Plugins browser as **"Unreal MCP"**).
- Capability toolsets ship as separate engine plugins under `…/UE_5.8/Engine/Plugins/Experimental/Toolsets/` (disabled by default). `AllToolsets` is the aggregator that enables all of them (EditorOnly, Experimental).
- **Do NOT enable "MCP Client Toolset"** — that makes UE a *client* of other MCP servers; it's unrelated. Leave it OFF.

---

## 2. Available MCP tools / capabilities

With `AllToolsets` enabled, `list_toolsets` returns ~50 toolsets (verified live in Gnarly 2026-06-18). Two families matter most:

### A. C++ `EditorToolset` — inspection, camera, PIE, screenshots (read/drive)
- **Actor inspect/select/focus:** `GetSelectedActors`, `SelectActors`, `GetVisibleActors`, `FocusOnActors`
- **Camera:** `GetCameraTransform`, `SetCameraTransform`; `ScreenCoordsToWorld`, `WorldPosToScreenCoords`
- **Play-in-Editor:** **`StartPIE`**, **`StopPIE`**
- **Screenshots (for clips/verification):** `CaptureViewport`, `CaptureEditorImage`, `CaptureAssetImage`
- **Content browser / assets:** `GetContentBrowserPath`/`SetContentBrowserPath`, `OpenEditorForAsset`, `GetOpenAssets`, `GetSelectedAssets`, `SelectAssets`
- **CVars:** `SearchCVars`

### B. Python `editor_toolset.*` family — world-building & asset creation (mutate)
This is the family that actually **spawns actors and creates assets** (the early "MCP can't build the world" claim was retracted after seeing this family). Key toolsets:
- **`SceneTools`** — `add_to_scene_from_class`, `add_to_scene_from_asset` (spawn), `remove_from_scene` (delete), `load_level`, `get_current_level`, `find_actors`, `create_level_instance`, `merge_actors`, `trace_world`, outliner folder ops.
- **`BlueprintTools`** — create/edit Blueprints.
- **`MaterialTools` / `MaterialInstanceTools`** — create/edit materials + instances.
- **`StaticMeshTools` / `SkeletalMeshTools`** — mesh ops.
- **`TextureTools`**, **`DataTableTools` / `CurveTableTools` / `DataAssetTools` / `StringTableTools`**.
- **`PrimitiveTools`** — add primitive geometry to actors.
- **`ObjectTools`** — list/get/set any UObject property.
- **`ActorTools`** — e.g. `set_actor_transform` (⚠ see gotcha below).
- **Domain suites:** Physics, Niagara (full system editing), UMG (widget trees), GAS, PCG, Sequencer/animation, Physics assets, GameplayTags, ConfigSettings, plugins, Slate UI automation.

### Meta-tools (always present, the dispatch layer)
- `list_toolsets` — enumerate registered toolsets.
- `describe_toolset` — list the tools inside a toolset + their params.
- `call_tool` — invoke a specific tool. **Gotcha:** pass the **bare** `tool_name` (e.g. `get_current_level`) with `toolset_name` separately; passing the fully-qualified `editor_toolset.toolsets.scene.SceneTools.get_current_level` as `tool_name` fails with "Unknown tool".

### What MCP can do, in one line
Inspect/select/focus actors · drive the camera · **Start/Stop PIE** · **capture screenshots** · **spawn & delete actors** · **load levels** · **create & edit Blueprints / materials / meshes / data tables / primitives** · read/set any object property · edit Niagara/UMG/GAS/Sequencer/Physics · query CVars and domain config.

### What MCP should NOT do (guardrails)
Mass-delete assets · convert the whole project to C++ · import huge asset packs · edit engine files/plugins · package builds · "build the entire game" / "make it AAA" / "full multiplayer." Blueprint **graph** logic that needs feel/judgment is best done by the human in-editor with Claude's guidance.

---

## 3. Build workflow

### The core loop (one pass per smallest task — from `Docs/METHOD_unreal_mcp_game_dev.md`)
1. **Pick the next smallest task** from `Docs/NEXT_STEPS.md`.
2. **Confirm preconditions:** `git status` clean (or intentionally staged) **and** current level **saved**. Never start on a dirty/unsaved state.
3. **Claude plans the smallest reversible change** — which asset/actor, which MCP action vs editor action, and the rollback path. No implementation yet.
4. **Chris approves** that single task. (If not approved, re-plan smaller.)
5. **Implement:** MCP `editor_toolset.*` action(s) where a tool exists; Unreal Python (`unreal` API) only for gaps; manual editor + Claude guidance for Blueprint graph logic & feel-tuning.
6. **Test in PIE:** `StartPIE` / press Play, verify the one expected behavior; `CaptureViewport` if useful.
7. **Save** the level + changed assets in the editor (editor is the source of truth for binary `.uasset`/`.umap`).
8. **Commit** (when asked): `git add . && git commit -m "<task>"`.
9. **Update `Docs/DEVLOG.md`:** date, task, files/assets changed, what worked, what broke, next step. Log breakages in `BUGS.md`.

### How to compile C++
- **Not applicable for the MVP** — Squishy (like Gnarly) is **Blueprint-only**: no `Source/`, no `Modules` in the `.uproject`, no compile step. Blueprints "compile" in-editor (the Compile button / on save).
- C++ is a **deferred, ask-before** item. Only when a system is stable and needs perf/architecture: add a `Source/` C++ module (requires **Visual Studio 2022** with *Desktop development with C++* + *Game development with C++*), then build via the editor's "Compile" / Live Coding or the VS solution. Converting the Blueprint project to C++ requires Chris's explicit approval.

### How to run / play-in-editor and verify
- Run: MCP `StartPIE` (and `StopPIE`), or press Play in the editor.
- Verify: watch the one expected behavior in PIE; use MCP `CaptureViewport` / `CaptureEditorImage` for screenshots; read PIE logs in the Output Log. Definition of done = open project → press Play → within ~30s exercise the core loop and record a 15–30s clip.

### One-shotting larger changes vs incremental
- **Incremental is the rule.** Big-bang building fails in Unreal: the editor is heavy, MCP is experimental, and binary assets desync/corrupt easily. Each step changes ONE thing and is verified in PIE before the next.
- **Smallest-fun-thing-first** + **placeholders before art** + **capture a clip early**.
- Gnarly's later `NEXT_STEPS.md` does say "work in bigger batches, check in at milestone boundaries" — i.e. once the pipeline is proven you can batch related work — but still verify per milestone and never stack an unverified change on another.
- **Parallel agents:** safe to fan out **text-only** work (docs, planning, research, naming specs) across agents, but **all editor/MCP mutations must be serialized through ONE session** — two sessions touching the same level/Blueprint will clobber binary state.

### Common pitfalls / failure modes called out in the docs
- **MCP plugin not visible:** confirm UE is truly 5.8; restart Epic Launcher + editor; search Plugins for `MCP`/`Model Context Protocol`/`AI`/`Experimental`; enable Show Plugin/Engine Content.
- **Claude can't see MCP tools:** launched from the wrong folder; `.mcp.json` not loaded (didn't restart Claude Code); **MCP server not started** in the editor; editor not restarted after enabling `AllToolsets`; wrong project. (These 4 are the usual smoke-test failures.)
- **Only 3–4 tools show up:** `AllToolsets` not enabled / editor not restarted after enabling it (only `AgentSkillToolset` registered).
- **`call_tool` "Unknown tool":** passed a fully-qualified name instead of the bare `tool_name` + separate `toolset_name`.
- **`ActorTools.set_actor_transform` resets unspecified fields to identity** (despite docs saying it won't) — **always pass full location + rotation + scale**, or it silently resets e.g. scale to 1.0. (Real bug hit in Gnarly's penalty work — reset ball scale 0.36→1.0.)
- **Gameplay broken on open:** wrong level open; GameMode override / default pawn / input mapping context / player start; read PIE logs.
- **Ball/object won't move:** Simulate Physics + collision enabled; mass/damping not extreme; impulse on the correct component; not permanently asleep.
- **Goal/trigger won't fire:** trigger collision + Generate Overlap Events; overlapping collision profiles; correct class check; volume actually inside the goal.
- **Performance poor:** raytracing off, simple lighting, placeholder meshes, lower scalability.
- **Claude overbuilds:** paste the "Pause. Return to the MVP…" redirect.
- **Install error `II-E1003` (UE 5.8):** caused by an outdated VC++ runtime (needs `14.50.35719+`). Fix: run `Engine\Extras\Redist\en-us\vc_redist.x64.exe /install /quiet /norestart` elevated, restart. The engine files themselves are usually fine. ("Launch did nothing" was just a slow ~60–90s first launch / shader compile, not a failure.)

---

## 4. Project conventions (from Gnarly — carry over to Squishy)

| Aspect | Convention |
|---|---|
| **Engine** | Unreal Engine **5.8.0** (Gnarly's `EngineAssociation` GUID `{E59F97F2-450C-5C57-0F4B-1C9C08E94088}` — Squishy will get its own GUID when created). |
| **Template** | Games → **Third Person**, **Blueprint** type. (Immediate controllable pawn; easy to evolve.) |
| **Starter Content** | Gnarly notes differ (kit said "on"; CLAUDE.md current state says "NOT installed"). Your call for Squishy; either works. Raytracing **off** for first tests recommended. |
| **C++ vs Blueprint** | **Blueprint-first.** No `Source/`, no C++ module until a system is stable and needs it (ask-before). |
| **Plugins (the MCP wiring)** | `.uproject` enables: `ModelContextProtocol`, `AllToolsets` (+ template defaults `ModelingToolsEditorMode`, `GameplayStateTree`). |
| **Content layout** | All game content under `Content/<GameFolder>/` with subfolders `Blueprints/{Characters,Ball,...,UI}`, `Maps`, `Materials`, `Meshes`, `Audio`, `VFX`, `Data`, `Input`, `Prototypes`. (Created in-editor / via MCP, not on disk by hand — UE manages `.uasset`.) |
| **Naming prefixes** | `BP_` blueprints/characters · `WBP_` widgets · `M_` materials · `MI_` material instances · `IA_` input actions · `IMC_` input mapping contexts · `LVL_` levels · `GM_` game modes · `PC_` player controllers · `DA_` data assets. Avoid `NewBlueprint3`, `test2_final`, etc. |
| **Docs** | Markdown at repo root in `Docs/`: `DEVLOG.md`, `BUILD_NOTES.md`, `MCP_NOTES.md`, `BUGS.md`, `NEXT_STEPS.md`, `GAME_BLUEPRINT.md`, `METHOD_*.md`, `BOOTSTRAP_HANDOFF_PROMPT.md`, etc. |
| **Source control** | Git at repo root. `.gitignore` ignores `Binaries/ Build/ DerivedDataCache/ Intermediate/ Saved/` + IDE (`.vs/ *.sln *.suo …`); **keeps** `Content/`, `Config/`, and the `.uproject`. |
| **MCP scoping** | `unreal` MCP is **project-scoped** in `<root>/.mcp.json`. The **Roblox_Studio** MCP is **global** (in `~/.claude.json` top-level `mcpServers`) — that's why it appears in every project incl. empty Squishy. It is unrelated to Unreal; ignore it here. |

### Verbatim: Gnarly's `.uproject` plugin block (`GnarlyNutmegUE_MCP1/GnarlyNutmegUE_MCP1.uproject`)
```json
{
	"FileVersion": 3,
	"EngineAssociation": "{E59F97F2-450C-5C57-0F4B-1C9C08E94088}",
	"Category": "",
	"Description": "",
	"Plugins": [
		{ "Name": "ModelingToolsEditorMode", "Enabled": true, "TargetAllowList": [ "Editor" ] },
		{ "Name": "GameplayStateTree", "Enabled": true },
		{ "Name": "ModelContextProtocol", "Enabled": true },
		{ "Name": "AllToolsets", "Enabled": true }
	]
}
```
> Note: **no `Modules` array** → Blueprint-only, no C++. The two MCP-relevant entries are `ModelContextProtocol` and `AllToolsets`.

### Verbatim: Gnarly's `.mcp.json` (`C:\Users\chris\Unreal-Gnarly\.mcp.json`)
```json
{
  "mcpServers": {
    "unreal": {
      "type": "http",
      "url": "http://127.0.0.1:8000/mcp"
    }
  }
}
```

### Verbatim: Gnarly's `.gitignore` (`C:\Users\chris\Unreal-Gnarly\.gitignore`)
```gitignore
# --- Unreal generated directories ---
Binaries/
Build/
DerivedDataCache/
Intermediate/
Saved/

# --- Visual Studio / IDE ---
.vs/
*.sln
*.suo
*.opensdf
*.sdf
*.VC.db
*.VC.opendb
```

### Note: no MCP/port config in tracked `Config/`
`DefaultEngine.ini` / `DefaultGame.ini` contain **no MCP/HTTP/port settings**. The MCP server's port/auto-start live in per-user `EditorPerProjectUserSettings` (under `Saved/`, gitignored). So the only project-side MCP wiring you check into git is the `.uproject` plugin enablement + the `.mcp.json`.

---

## 5. ⭐ BOOTSTRAP CHECKLIST for `C:\Users\chris\Unreal-squishy`

Goal: stand up a new empty UE 5.8 + MCP "Squishy Smash" project with the same wiring as Gnarly. Steps are ordered. Each is tagged:
- **[AUTO]** = an agent can do it now by writing files to disk (no editor needed).
- **[USER]** = requires Chris / the Unreal Editor / a Claude Code restart (cannot be done by file-writing alone).

> Convention used below: new project name **`SquishySmashUE_MCP1`**, content folder **`Content/SquishySmash/`**. (Mirrors Gnarly's `GnarlyNutmegUE_MCP1` / `Content/GnarlyNutmeg/`. UE has a project-name length limit — keep it short, avoid a long `_Prototype` suffix.)

### Phase 0 — Scaffold on disk (can run BEFORE the editor exists) — **[AUTO]**
These create the repo skeleton so Claude Code has context the moment it reconnects. They do **not** create the `.uproject` or any `.uasset` (those require the editor).
1. **[AUTO]** Create folders: `Docs/`, `Docs/research/` (this file is already here).
2. **[AUTO]** Write **`.gitignore`** at root — copy Gnarly's verbatim (Section 4).
3. **[AUTO]** Write **`.mcp.json`** at root — exact content in Section 5.1 below (identical to Gnarly; same localhost port). This is the file that wires Claude Code → the editor's MCP server.
4. **[AUTO]** Write **`CLAUDE.md`** at root — adapt Gnarly's: keep the guardrails, naming table, MCP rules, ask-before list, build loop; swap project name → `SquishySmashUE_MCP1`, content folder → `Content/SquishySmash/`, and replace the soccer-specific design/scope with Squishy Smash's MVP.
5. **[AUTO]** Seed **`Docs/`**: `DEVLOG.md`, `BUILD_NOTES.md`, `MCP_NOTES.md` (copy Gnarly's MCP_NOTES nearly verbatim — it's the verified capability map), `BUGS.md`, `NEXT_STEPS.md`, and a `BOOTSTRAP_HANDOFF_PROMPT.md` (adapt Gnarly's; it's the post-connection verify+smoke-test prompt). Optionally copy `METHOD_unreal_mcp_game_dev.md` (it's genre-generic and explicitly designed to be reused).
6. **[AUTO]** (Optional) Copy the reusable kit folder. The Gnarly kit `gnarly-nutmeg-unreal-mcp-claude-kit/` is soccer-specific in places, but docs **02 (MCP setup)**, **03 (guardrails)**, **07 (structure)**, **13 (troubleshooting)** are reusable references. Either copy those into a `squishy-smash-claude-kit/` and rewrite the game-specific ones, or skip the kit and rely on `CLAUDE.md` + `Docs/`.
7. **[AUTO]** `git init` at the root (commit after the editor creates the project so the `.uproject`/`Config` are included).

> **What CANNOT be done by file-writing:** the `.uproject`, `Config/Default*.ini`, the Content Browser folders, and any `.uasset`/`.umap`. Those are generated by the Unreal Editor when the project is created. **Do not hand-author a `.uproject`** — let the editor create it, then edit only its `Plugins` array (Phase 2).

### Phase 1 — Create the UE project (editor) — **[USER]**
8. **[USER]** Open **Epic Games Launcher → Unreal Engine 5.8** (install if missing; if `II-E1003`, fix VC++ redist per Section 3).
9. **[USER]** **New Project → Games → Third Person**, type **Blueprint**, Raytracing **off**, Starter Content your choice. **Set the project location so the `.uproject` lands at `C:\Users\chris\Unreal-squishy\SquishySmashUE_MCP1\SquishySmashUE_MCP1.uproject`** (UE creates a subfolder named after the project — point the "Location" field at `C:\Users\chris\Unreal-squishy`). This matches Gnarly's nested layout (repo root = parent of the `.uproject` subfolder).
   - This generates `SquishySmashUE_MCP1.uproject`, `Config/Default*.ini`, `Content/` (template assets), and `Saved/`/`Intermediate/`/`DerivedDataCache/`.

### Phase 2 — Enable the MCP plugins (editor) — **[USER]**, then **[AUTO]** verify
10. **[USER]** In the editor: **Edit → Plugins**, enable **"Unreal MCP" (`ModelContextProtocol`, Experimental)** and **`AllToolsets`** (search `MCP`, `Toolsets`, `Model Context Protocol`). Do **NOT** enable "MCP Client Toolset." Restart the editor when prompted.
    - Equivalent / verify: the `.uproject` `Plugins` array should contain `ModelContextProtocol` and `AllToolsets` both `"Enabled": true`. An agent **[AUTO]** can edit the `.uproject` to add these entries (it's plain JSON), but the **editor must still be restarted** to load them — so it's cleanest to enable in the Plugins UI, or edit the JSON then have the user reopen the editor.

### Phase 3 — Start the MCP server (editor) — **[USER]**
11. **[USER]** Start the in-editor MCP server (it does NOT auto-start):
    - Per-session: open the console (`` ` ``) and run **`ModelContextProtocol.StartServer`** (or `ModelContextProtocol.StartServer 8000`).
    - OR persistent: **Editor Preferences → Model Context Protocol → Auto Start Server = ON**, then restart the editor.
12. **[USER]** Confirm the Output Log shows **`LogModelContextProtocol: Starting MCP server on port 8000`**.

### Phase 4 — Connect Claude Code — **[USER]**
13. **[USER]** Ensure `C:\Users\chris\Unreal-squishy\.mcp.json` exists (from Phase 0). **Restart Claude Code** in that folder so it loads the new server.
    - At startup Claude Code will show a trust prompt for the project `.mcp.json` server `unreal` — **approve it** ("Use this MCP server"). (Gnarly's project entry in `~/.claude.json` shows this is approved via the interactive trust dialog, not pre-baked.)
    - Alternative to editing `.mcp.json`: `claude mcp add --transport http unreal http://127.0.0.1:8000/mcp`.
14. **[USER]** Keep the **Roblox_Studio** MCP in mind: it's global and will still be connected; it is **unrelated** to Unreal — do not use it for Squishy.

### Phase 5 — Verify the connection is live — **[AUTO]** (Claude runs these via MCP)
15. **[AUTO/Claude]** Call meta-tools: **`list_toolsets`** (expect ~50, incl. `EditorToolset` + `editor_toolset.*` Scene/Blueprint/Material/etc.) and **`describe_toolset` for `EditorToolset`**. If you see only 3–4 tools, `AllToolsets` isn't loaded → restart editor.
16. **[AUTO/Claude]** Read live state: **`SceneTools.get_current_level`** → should return the open level path. Report project name / level / UE version.
17. **[AUTO/Claude]** **Reversible spawn smoke test** (do NOT save the level):
    - `SceneTools.add_to_scene_from_asset` (`/Engine/BasicShapes/Cube`, name e.g. `MCP_RoundTripTest_Cube`) → `find_actors "MCP_RoundTripTest"` (confirm present) → `remove_from_scene` → `find_actors` again (confirm `[]`).
    - Remember: `call_tool` wants the **bare** `tool_name` + separate `toolset_name`.
18. **[AUTO/Claude]** Prove PIE + capture: `EditorToolset CaptureViewport`, then `StartPIE` → `StopPIE`.
19. **[AUTO/Claude]** Log pass/fail (and the actual observed tool names) in `Docs/MCP_NOTES.md`. **Only then** begin building, one approved task at a time.

### Phase 6 — First commit + content scaffold — mixed
20. **[USER/AUTO]** `git add . && git commit -m "Initial SquishySmashUE_MCP1 UE 5.8 + MCP prototype setup"` (now includes the `.uproject` + `Config`).
21. **[AUTO/Claude via MCP]** Create the `Content/SquishySmash/` folder structure + first placeholder level `LVL_*` in-editor/via MCP (NOT on disk). Then proceed through `NEXT_STEPS.md` with the Section 3 loop.

### 5.1 — Exact `.mcp.json` to place at `C:\Users\chris\Unreal-squishy\.mcp.json`
**Identical to Gnarly** — the endpoint is a fixed localhost address and the same machine runs one editor at a time, so no path/port change is needed:
```json
{
  "mcpServers": {
    "unreal": {
      "type": "http",
      "url": "http://127.0.0.1:8000/mcp"
    }
  }
}
```
**Port-collision caveat:** the port `8000` is the editor's, not the project's. If you ever run **both** the Gnarly editor and the Squishy editor at the same time, they'll fight over `8000`. For normal single-project work (only one editor open) this is fine and the same `.mcp.json` works for both. If you need them concurrent, start one editor's server on a different port (`ModelContextProtocol.StartServer 8001` or `-ModelContextProtocolPort=8001`) and set that project's `.mcp.json` `url` to `http://127.0.0.1:8001/mcp`.

### 5.2 — Manual (USER) vs automatable (AUTO) summary
| Can be AUTOMATED now (write files) | REQUIRES the USER / editor / Claude restart |
|---|---|
| `.gitignore`, `.mcp.json`, `CLAUDE.md` | Install/open UE 5.8; create the Third-Person Blueprint project (makes `.uproject` + `Config` + Content) |
| `Docs/*` (DEVLOG, BUILD_NOTES, MCP_NOTES, BUGS, NEXT_STEPS, handoff prompt, METHOD) | Enable `ModelContextProtocol` + `AllToolsets` in Plugins; **restart editor** |
| Optional `squishy-smash-claude-kit/` reference docs | Start the MCP server (`ModelContextProtocol.StartServer` or Auto Start) |
| `git init`; editing the `.uproject` `Plugins` JSON (but editor must still reload) | **Restart Claude Code** + approve the `unreal` trust prompt |
| (After connection) all asset/level creation **via MCP** | The very first project creation + plugin load + server start are hard gates with no file-only path |

---

## 6. Quick reference — verify MCP is live (paste-ready intent)
1. `list_toolsets` → ~50 toolsets incl. `EditorToolset` and `editor_toolset.*` (Scene/Blueprint/Material/StaticMesh/Object…).
2. `SceneTools.get_current_level` → returns the open level path (proves read).
3. Spawn `/Engine/BasicShapes/Cube` via `SceneTools.add_to_scene_from_asset` → `find_actors` → `remove_from_scene` → `find_actors` empty (proves mutate + cleanup; don't save).
4. `EditorToolset CaptureViewport` + `StartPIE`/`StopPIE` (proves screenshot + play).
If any fail: editor not open / server not started / `AllToolsets` off (only 3–4 tools) / editor not restarted after enabling / Claude Code not restarted / wrong folder.

## 7. Source files studied (all under `C:\Users\chris\Unreal-Gnarly\`)
- `.mcp.json`, `.gitignore`, `CLAUDE.md`
- `GnarlyNutmegUE_MCP1/GnarlyNutmegUE_MCP1.uproject` (no `Plugins/` or `Source/` dir confirmed — Blueprint-only)
- `GnarlyNutmegUE_MCP1/Config/DefaultEngine.ini`, `DefaultGame.ini`, `DefaultEditorPerProjectUserSettings.ini` (no MCP/port settings present)
- `Docs/MCP_NOTES.md` (verified capability map + smoke-test log), `Docs/METHOD_unreal_mcp_game_dev.md`, `Docs/BUILD_NOTES.md`, `Docs/BOOTSTRAP_HANDOFF_PROMPT.md`, `Docs/NEXT_STEPS.md`, `Docs/PENALTY_PIVOT_NOTES.md`
- `gnarly-nutmeg-unreal-mcp-claude-kit/` (README, project_manifest.json, 02 setup, 07 structure, 13 troubleshooting, claude_bootstrap_prompt.md) — reusable starter kit
- `gnarly-nutmeg-penalty-pivot-claude-kit/` + `.zip` — a later *design pivot* kit (penalty-mode game design, not MCP setup); the `.zip` is just a zipped copy of that kit. Not needed for MCP bootstrap.
- `C:\Users\chris\.claude.json` — confirms `Roblox_Studio` is a **global** MCP (top-level `mcpServers`); the Gnarly project entry has `enabledMcpjsonServers: []` + interactive trust approval for `unreal`.
