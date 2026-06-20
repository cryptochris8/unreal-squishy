# DEVLOG — Squishy Smash UE

Newest first. Format: **date — task — files/assets — what worked — what broke — next.**

## 2026-06-19 — Imported all 51 squishy 3D meshes + mapped roster  (Claude)
- **Found existing 3D models** (Chris's earlier work): 51 squishy meshes as both `.fbx` (`Roblox-squishy/tools/mesh_pipeline/output/`, the cleaner game export + `manifest.json`) and `.glb` (`Squishy-smash/_tmp_3d_renders/`). AI-generated (~30 credits each), geometry-only (no embedded materials).
- **Decision (Chris):** import all 51 now + map the 16 Pudding Hills friends, then resume the loop.
- **Verified import path:** `StaticMeshTools.import_file` (FBX→StaticMesh). Probe (`soft_dumpling`): bounds ≈ **191×191×145 cm, centered on origin** (pivot = center, not base), **12,148 tris**, one empty material slot. Geometry-only is on-brand — design wants our tinted `M_JellyPlush` + per-friend `SuggestedTintHex`, not raw AI textures.
- **Bulk import via `ProgrammaticToolset.execute_tool_script`** (one sandboxed call, per-file try/except): imported the remaining **50 → 51/51 total, 0 failures**, in `/Game/SquishySmash/Meshes` as `SM_<name>`. (Far cheaper than 51 round-trips.)
- **Mapping rule (clean):** friend CSV `Name` == mesh filename, so **FriendId → `/Game/SquishySmash/Meshes/SM_<Name>`**. All 16 Pudding Hills friends (001 soft_dumpling … 016 celestial_dumpling_core) have meshes. No separate mapping table needed; DataTable can derive the path.
- **Scale/pivot pass (applied on the Body component, since import can't bake asset scale):** friend `Body` set to `SM_peach_mochi`, **RelativeScale3D 0.5** (≈72 cm tall), **RelativeLocation Z +36** (base at actor origin for ground placement). Verified on the SCS template + compiled + saved.
- **Collision:** deferred — line traces hit complex (per-poly) collision by default, so squish-trace should work without generated simple collision; will `generate_convex_collisions` only if PIE trace misses.
- **Next:** resume the squish loop — author `BP_SquishyFriend` `Squish`→`HappyPop`→`Respawn` + breathing (mesh-agnostic, unaffected by the mesh swap), then character + input, then place + PIE.

## 2026-06-19 — M1 batch 2 start: verified component + graph-DSL pipelines  (Claude)
- **Pushed** batch 1 to GitHub `cryptochris8/unreal-squishy` (renamed branch `master`→`main`; commit `53297d1`).
- **Verified `PrimitiveTools` adds SCS components to a Blueprint** by calling `add_sphere` on the **CDO** (`Default__BP_SquishyFriend_C`): added `Body` sphere (r=50). CDO reads `Body=None` (expected for SCS templates), but a **spawned instance** has a real `Body` component under `DefaultSceneRoot` — confirmed it persists. (Spawn landed in the still-loaded `Lvl_ThirdPerson`, not `LVL_PuddingHills` — must `load_level` to work in our map; test actor removed.)
- **Verified graph-DSL authoring** (`write_graph_dsl`/`read_graph_dsl`): implemented `BP_SquishyGameState.AddSparkleCoins(Amount:int)` → `SetSparkleCoins(GetSparkleCoins + Amount)`. Compiles clean with **warnings-as-errors**, reads back exact. DSL patterns that worked w/o discovery: `Variables|Default|Get<Var>` / `Set<Var>`, `(+ ...)`, `(fn Name (Params) ...)`.
- **BP_SquishyFriend** now has `Body` sphere component + state vars `LastSquishTime`(float), `bPopped`(bool) added (alongside the §3 tuning vars). Compiled + saved.
- **Next (uncommitted):** author friend `Squish`→`HappyPop`→`Respawn` graph (cooldown 0.12 via game-time delta, Joy += 0.34, squash-stretch on Body, at Joy≥1 hide+disable-collision+timer 1.2s→respawn, award coins via GameState). Then `BP_SquishyCharacter` movement + Enhanced Input (`IMC_Squishy`/`IA_*`) + look-trace that calls the friend's Squish. Then `load_level` LVL_PuddingHills, strip template blocks, place friends, PIE test.

## 2026-06-19 — MCP live + M1 foundations batch  (Claude)
- **MCP verified live:** Unreal `ModelContextProtocol` plugin responding inside the editor; `AllToolsets` active (~50 toolsets). Confirmed the `call_tool` contract: **bare** `tool_name` + separate `toolset_name` (fully-qualified names error). `CaptureViewport` needs explicit `null`s for optional params and returns a ~2 MB base64 PNG (too big to inline — use sparingly).
- **Folder tree:** created `Content/SquishySmash/{Blueprints/{Character,Friends,Gameplay,UI},Maps,Materials,Meshes,Niagara,Audio,Data,Input,Cards}` via `AssetTools.create_folder`.
- **Greybox level:** `LVL_PuddingHills` created by **duplicating** `/Game/ThirdPerson/Lvl_ThirdPerson` → `/Game/SquishySmash/Maps/` (instant lit/sky/floor/PlayerStart base; template playground blocks still in it — strip/replace next).
- **Blueprint skeletons** (correct parents, compiled clean): `BP_SquishyFriend`:Actor, `BP_SquishyCharacter`:Character, `BP_FriendPad`:Actor, `BP_SquishyGameState`:GameStateBase.
- **Data vars w/ §3 tuning defaults:** `BP_SquishyFriend` → Joy 0, JoyPerSquish **0.34**, SquishCooldown **0.12**, RespawnDelay **1.2**, CoinReward **8**, FriendId(Name). `BP_SquishyGameState` → SparkleCoins, FriendsWoken (int). Defaults set on CDO via `ObjectTools.set_properties` and verified.
- **Worked:** big-batch asset/skeleton creation via MCP is fast and reliable. `add_variable` type_name vocab confirmed: `float`, `int`, `name`.
- **What broke / gotchas:** `BlueprintTools` has **no add-component tool** → SCS components (mesh, collision, spring-arm/camera) + all graph logic (squish line-trace, Happy Pop, coin award) are the **next batch**, partly manual per CLAUDE.md §11. `describe_toolset` for big toolsets overflows the token limit (saved to file; parse with python).
- **Not yet:** components, graph logic, set `LVL_PuddingHills` as default map, strip template blocks. Nothing committed yet (awaiting Chris).
- **Next:** components + squish→Joy→Happy Pop graph on `BP_SquishyFriend`; movement/input on `BP_SquishyCharacter` (`IMC_Squishy`/`IA_*`); then place friends + test in PIE.

## 2026-06-19 — Phase 0: project scaffold + research  (Claude)
- **Research (3 parallel agents):** studied the Roblox game, the story/art IP, and the Unreal-MCP workflow. Wrote `Docs/research/01-roblox-gameplay-systems.md`, `02-story-character-art-bible.md`, `03-unreal-mcp-workflow-bootstrap.md` (~114 KB).
- **Key finding:** "Squishy Smash" is a **wholesome no-combat collector/explorer** (not a fighter). "The Lost Sparkle" = **Book #2** of the real storybook (3 shards across 3 lands). Mechanics are simple → buildable in big batches.
- **Scaffolded repo:** `.mcp.json`, `.gitignore`, `README.md`, `CLAUDE.md`, and `Docs/{GAME_BLUEPRINT, NEXT_STEPS, MCP_NOTES, BOOTSTRAP_HANDOFF_PROMPT, BUILD_NOTES, BUGS, CONTENT_STRUCTURE_PLAN}.md`. `git init`.
- **Worked:** confirmed canonical IP at `C:\Users\chris\Squishy-smash\` (incl. finished Lost Sparkle book spreads) and proven tuning from the Roblox source.
- **Pre-staged content (editor-independent prep):** extracted **all three land rosters — 48 launch friends** — from `Roblox-squishy/data/raw/` into import-ready DataTable CSVs (`Docs/data/{PuddingHills,GooCoast,MoonlitHollow}_Friends.csv`) + provenance README.
- **Committed** Phase 0 + the **newly-created `SquishySmashUE` UE 5.8 Third Person project** as the baseline (`599a983`); `.gitignore` correctly excluded Binaries/Intermediate/Saved. *(Commit message body says "no .uproject yet" — stale: the project landed in the same `git add -A`.)*
- **Project state:** Chris created `SquishySmashUE` and **enabled `ModelContextProtocol` + `AllToolsets`** in the `.uproject`. No MCP auto-start in committed Config → server needs a manual start.
- **Remaining bootstrap (runtime):** restart editor (load plugins) → `ModelContextProtocol.StartServer` (confirm port 8000) → restart Claude Code (loads `.mcp.json`, approve `unreal`).
- **Next:** run the MCP smoke test, then start **M1** (Pudding Hills core squish loop) from `NEXT_STEPS.md`.
