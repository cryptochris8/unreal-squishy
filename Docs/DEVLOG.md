# DEVLOG — Squishy Smash UE

Newest first. Format: **date — task — files/assets — what worked — what broke — next.**

## 2026-06-19 — Character input scaffolding (reparent + input assets)  (Claude)
- **`BP_SquishyCharacter` reparented** to `/Game/ThirdPerson/Blueprints/BP_ThirdPersonCharacter` → inherits camera, spring-arm, gentle movement, and Enhanced Input (Move/Look/Jump) for free. Compiles clean.
- **Input assets:** template input lives in `/Game/Input/` (`IMC_Default`, `IA_Move/Look/Jump`). Created `IA_Squish` (duplicate of `IA_Jump` — a digital/bool action) and `IMC_Squishy` (duplicate of `IMC_Default`) in `/Game/SquishySmash/Input/`. Mapped **`IA_Squish` → Left Mouse Button** in `IMC_Squishy` (verified).
- **MCP findings (important):**
  - `BlueprintTools.create` will NOT make non-Blueprint assets (e.g. `InputAction`) — it errors with a modal dialog (hung the server until dismissed). Use `AssetTools.duplicate` of an existing asset of that type instead.
  - `ObjectTools.set_properties` on an array property **replaces** the whole array (so `IMC_Squishy` is now squish-only — intended; it rides alongside the inherited `IMC_Default`).
  - `get_properties` can't read `IMC_Default.Mappings` (returns `[]` even though the template clearly has them) — its mappings aren't API-exposed, so never edit the template IMC; work on duplicates.
  - Enhanced Input events read as `(event EnhancedInputActionIA_Jump (ActionValue ElapsedSeconds TriggeredSeconds InputAction) …)`. The template adds its mapping context in the **PlayerController**, not the character graph.
- **Next (character graph, next session):** add `EnhancedInputActionIA_Squish` handler → camera-forward line trace (~400 cm) → if hit is `BP_SquishyFriend`, call its `Squish`; ensure `IMC_Squishy` is added (BeginPlay override w/ EnhancedInput subsystem, or via the PC). Then `BP_SquishyGameMode` (DefaultPawn=BP_SquishyCharacter, GameState=BP_SquishyGameState), `load_level` LVL_PuddingHills, strip template blocks, place ~12 friends, **PIE**.

## 2026-06-19 — BP_SquishyFriend squish loop authored (graph DSL)  (Claude)
- **Authored the friend's core loop** in the EventGraph via `write_graph_dsl` — compiles clean with **warnings-as-errors**:
  - `Squish` (custom event, public — callable by the character): if `not Popped` AND `GameTime - LastSquishTime >= SquishCooldown(0.12)` → set LastSquishTime, `Joy += JoyPerSquish(0.34)`; when `Joy >= 1.0` → set `Popped`, `SetActorHiddenInGame(true)`, `SetActorEnableCollision(false)`, `SetTimerByFunctionName("Respawn", RespawnDelay 1.2)`, then cast GameState→`BP_SquishyGameState` and `AddSparkleCoins(CoinReward)`.
  - `Respawn` (custom event): `Joy=0`, `Popped=false`, un-hide, re-enable collision. Pop visuals run *before* the cast, so the friend still pops even if the GameState isn't our class yet (pre-GameMode-wiring).
- **DSL lessons (recorded for reuse):** custom events must be pre-created with `add_event` then referenced as `(event Custom|<Name> …)` — `(event <Name>)` fails. Bool var `bPopped` → accessors `Get/SetPopped` (the `b` is stripped). `SetActorHiddenInGame` pin is `bNewHidden` (not `NewHidden`). A multi-exec node (e.g. Cast) **terminates** the enclosing exec flow, so anything that must run regardless goes *before* it. Discover names with `find_node_types` / pins with `get_node_type_pins`.
- **Deferred (juice, not blocking):** squash-stretch on squish + idle breathing bob → feel pass.
- **Next:** `BP_SquishyCharacter` (camera, gentle movement, Enhanced Input) with a look line-trace that calls the hit friend's `Squish`; reuse the ThirdPerson template's input assets + add `IA_Squish`. Then set GameMode, `load_level` LVL_PuddingHills, place friends, **PIE**.

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
