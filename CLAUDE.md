# CLAUDE.md — Squishy Smash: The Lost Sparkle (Unreal Engine 5.8 + MCP)

Persistent project memory. **Read this before acting.** Deeper detail lives in `Docs/` — follow the §13 pointers instead of re-reading everything each session.

## 1. Project

**Squishy Smash: The Lost Sparkle** — a wholesome **Unreal Engine 5.8 + MCP** game targeting **Steam (PC)**. It is a **fresh Unreal rebuild of the shipped Roblox game "Squishy Smash," NOT a 1:1 port** — recreate the smallest genuinely-fun core experience first, then grow it. Owner: **Chris Campbell** (chriscam8@gmail.com), AppalachAI Studios.

The game is a **cozy, no-combat collector/explorer** for young players and a gentle general Steam audience. Despite the word **"Smash," there is zero fighting** — you wander a storybook world and **squish sleepy plush "squishy friends"** to fill their Joy, popping them into sparkles + coins, then collect them into a book. **"The Lost Sparkle" is Book #2 of the real Squishy Smash storybook** (a finished 18-spread tale): the world's Sparkle splits into three shards that scatter across three lands — reuniting them is the game's quest spine.

Project name (as created in UE): **`SquishySmashUE`**. Its `.uproject` lives at **`SquishySmashUE/SquishySmashUE.uproject`** inside this repo root (`C:\Users\chris\Unreal-squishy`), mirroring the sibling Gnarly layout. If you create it under a different name, update this section.

## 2. Golden rule — build the Pudding Hills vertical slice first

A single-player, single-land **vertical slice** that is always playable:

> gentle third-person character → walk up to a sleepy **squishy friend** → **squish** it (Joy fills in ~3 squishes) → **Happy Pop** into a sparkle burst + **Sparkle Coins** → open a **free Sparkle Capsule** → **Discover** a friend into the **Squishy Book** → **equip** it as a floating **buddy** → wake enough friends so the **first Sparkle Shard** appears → recover it.

Tune by feel; keep it playable every step. Multiplayer, the other two lands (Goo Coast, Moonlit Hollow), and monetization come **after** this slice is fun. **The mechanics are simple — no shot physics/ragdoll like Gnarly Nutmeg — so build in BIG BATCHES, not slow step-by-step.**

## 3. Current state (honest, 2026-06-19)

- **Phase 0 scaffolding complete** (this repo): config, this brain, design (`Docs/GAME_BLUEPRINT.md`), backlog (`Docs/NEXT_STEPS.md`), deep research (`Docs/research/`).
- **UE 5.8 project `SquishySmashUE` created + committed** (Third Person template; `.uproject` at `SquishySmashUE/SquishySmashUE.uproject`, baseline commit `599a983`). **`ModelContextProtocol` + `AllToolsets` are enabled** in the `.uproject`. **Remaining bootstrap (runtime only):** restart the editor so the plugins load → **start the MCP server** (console `ModelContextProtocol.StartServer`, confirm `port 8000` log) → **restart Claude Code** so it loads `.mcp.json` + approve the `unreal` trust prompt. Until then, this session has only the unrelated **global** `Roblox_Studio` MCP. Steps: `Docs/BOOTSTRAP_HANDOFF_PROMPT.md`.
- The Unreal MCP is Epic's built-in **`ModelContextProtocol`** plugin (Experimental, ships in UE 5.8): an HTTP MCP server **running inside the live editor** at `http://127.0.0.1:8000/mcp`. It does **not** auto-start. `.mcp.json` (repo root) registers it; **`AllToolsets`** must also be enabled to get world-building tools. Full detail: `Docs/MCP_NOTES.md`.

## 4. READ-ONLY sources of truth (never modify)

- **IP / story / art:** `C:\Users\chris\Squishy-smash\` — locked Story Bible, all 48 card arts, branding, and **the Lost Sparkle book final spreads** (`book2_final_spreads/`). → `Docs/research/02-story-character-art-bible.md`.
- **Gameplay / systems:** `C:\Users\chris\Roblox-squishy\` — the shipped game's design docs, proven tuning, and 48-friend data (`src/ReplicatedStorage/Shared/SquishyDefinitions.lua`, `data/raw/`). → `Docs/research/01-roblox-gameplay-systems.md`.
- **Sibling UE+MCP project (workflow/conventions only):** `C:\Users\chris\Unreal-Gnarly\`.

## 5. Non-negotiable creative rule (brand identity — load-bearing, not flavor)

Kid-friendly and storybook-safe. Build around **squish, squeeze, bounce, boop, pop, sparkle, discover, collect, decorate, help, friendship**. **NO** combat, weapons, damage, hazards, death, horror, vulgarity, romance/dating, blood/gore, or mature content. **No villain, ever** — tension is situational, never a bad guy.

Player-facing terminology (use the soft words):

| Use | Not |
|---|---|
| Joy Meter | health |
| Squish / Squish Power | attack |
| Happy Pop | defeat / burst |
| Squishy Friends | enemies |
| Play Zone | arena / battle |
| Discovered | won / pulled |

(Internal code/data may keep legacy field names like `burstThreshold`; only the **UI** must be soft.)

## 6. Hard guardrails — DO NOT (until the single-player fun loop is proven)

multiplayer/replication, monetization, accounts, leaderboards, cosmetics store, season pass, Steamworks/online subsystems, GAS, AI behavior trees, analytics SDK. **Forbidden by the brand at ANY time:** paid randomness / coin packs (**Sparkle Capsules are FREE forever**), trading, and anything in §5.

## 7. Ask-before list (require Chris's approval)

Deleting assets · overwriting major Blueprints · changing engine version · converting to C++ · installing third-party plugins · importing large Marketplace/Fab packs · changing packaging settings · renaming the project · spending money or running API-cost-heavy asset generation.

## 8. Workflow loop

1. Plan the smallest meaningful change — or a **batch**, since mechanics are simple. 2. Make it (MCP-first; §11). 3. Test in PIE. 4. Save. 5. Commit after milestones (Git). 6. Update `Docs/DEVLOG.md` (date · task · files/assets · what worked · what broke · next).

Output style: clear steps, exact paths, small diffs, test + rollback instructions.

## 9. Naming conventions

`BP_` blueprints · `WBP_` widgets · `M_` materials · `MI_` material instances · `IA_` input actions · `IMC_` input mapping contexts · `LVL_` levels · `GM_` game modes · `PC_` player controllers · `DA_` data assets · `DT_` data tables · `NS_` Niagara systems · `SG_` save game. Avoid `NewBlueprint3`, `test_final`, `copy_of_X_REALFINAL`.

## 10. Where things live

- Game content under **`SquishySmashUE/Content/SquishySmash/`** — subfolders `Blueprints/{Character,Friends,Gameplay,UI}`, `Maps`, `Materials`, `Meshes`, `Niagara`, `Audio`, `Data`, `Input`, `Cards`. Created in-editor / via MCP. See `Docs/CONTENT_STRUCTURE_PLAN.md`.
- **`Docs/`** at repo root: `research/`, `GAME_BLUEPRINT.md`, `NEXT_STEPS.md`, `MCP_NOTES.md`, `DEVLOG.md`, `BUILD_NOTES.md`, `BUGS.md`, `CONTENT_STRUCTURE_PLAN.md`, `BOOTSTRAP_HANDOFF_PROMPT.md`.

## 11. MCP rules (UE 5.8 — verified in the sibling Gnarly project; re-verify here after bootstrap)

With **`AllToolsets`** enabled, MCP covers inspection, camera, **PIE**, screenshots, config, AND world-building: spawn/remove actors (`SceneTools.add_to_scene_from_class` / `add_to_scene_from_asset` / `remove_from_scene`), create/edit Blueprints (`BlueprintTools`), materials (`MaterialTools`), meshes (`StaticMeshTools`), data tables (`DataTableTools`), Niagara/UMG, set properties (`ObjectTools`). **MCP-first**; fall back to Unreal **Python** for gaps; manual Blueprint *graph* work + Claude guidance for logic/feel. **Before any mutating op:** under Git, current level saved, change small & reversible. Gotcha: `call_tool` wants the **bare** `tool_name` plus a separate `toolset_name`. **Never via MCP:** mass deletes, whole-project C++ conversion, huge imports, packaging.

## 12. Definition of done (vertical slice)

Open the project, press Play, and within ~60 seconds: walk → squish a friend → watch it Happy Pop into sparkles + coins → open a capsule → see a new friend in the Squishy Book → equip it as a floating buddy. Record a 15–30s clip into `Docs/MARKETING_CLIPS.md` notes.

## 13. Pointers

- Design + proven tuning (master): **`Docs/GAME_BLUEPRINT.md`**
- Backlog / milestones: **`Docs/NEXT_STEPS.md`**
- Bring the MCP live (do this next): **`Docs/BOOTSTRAP_HANDOFF_PROMPT.md`**
- MCP capability map: **`Docs/MCP_NOTES.md`** · Build/install notes: **`Docs/BUILD_NOTES.md`**
- Deep research: **`Docs/research/{01-roblox-gameplay-systems, 02-story-character-art-bible, 03-unreal-mcp-workflow-bootstrap}.md`**
