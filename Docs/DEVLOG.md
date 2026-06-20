# DEVLOG — Squishy Smash UE

Newest first. Format: **date — task — files/assets — what worked — what broke — next.**

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
