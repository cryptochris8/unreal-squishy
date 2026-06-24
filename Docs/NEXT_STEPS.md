# NEXT_STEPS — Squishy Smash UE (live backlog)

Status legend: ☐ todo · ◐ in progress · ☑ done. Work top-down; keep it playable.

## M0 — Bootstrap (bring the MCP live) — BLOCKS EVERYTHING
See `Docs/BOOTSTRAP_HANDOFF_PROMPT.md`. **Requires Chris (manual editor steps).**
- ☑ Phase 0 repo scaffold (config, CLAUDE.md, Docs, research) — *Claude, 2026-06-19*
- ☑ Create UE 5.8 project `SquishySmashUE` (Third Person template) at `C:\Users\chris\Unreal-squishy\` → `.uproject` at `SquishySmashUE/SquishySmashUE.uproject` — *committed `599a983`*
- ☑ Enable plugins **Unreal MCP (`ModelContextProtocol`)** + **`AllToolsets`** in the `.uproject` *(restart editor to load them)*
- ☑ Start MCP server (`ModelContextProtocol.StartServer`); confirm log on port 8000
- ☑ Restart Claude Code in repo root; approve `unreal` trust prompt
- ☑ Verify: `list_toolsets` (~50) → `find_actors` → `get_current_level`; `CaptureViewport`; `IsPIERunning` — *Claude, 2026-06-19* (`call_tool` wants bare tool name + separate toolset_name)
- ☑ `git init` + **initial commit `599a983`** (Phase 0 scaffold + UE project baseline + all-48 friend CSVs)

## M1 — Core squish loop (the heart) — Pudding Hills greybox
- ☑ `Content/SquishySmash/` folder tree (per `CONTENT_STRUCTURE_PLAN.md`) — *Claude, 2026-06-19*
- ◐ `LVL_PuddingHills` greybox: duplicated from template (lit/sky/floor/PlayerStart) — *still has template playground blocks; strip + add hills/windmill + bloom pass*
- ☑ `BP_SquishyCharacter` reparented to `BP_ThirdPersonCharacter`; squish input = **legacy LMB/F key events → `OnSquishPressed`** (Enhanced-Input events don't bind via MCP — see `MCP_NOTES.md`). `OnSquishPressed` = `GetAllActorsOfClass(Friend)` → `GetDistanceTo ≤ 350` → `Squish` (proximity, no trace). — *2026-06-21*
- ☑ `BP_SquishyGameMode` (DefaultPawn=`BP_SquishyCharacter`, GameState=`BP_SquishyGameState`, PC=`BP_SquishyPlayerController` which adds IMC_Default/MouseLook + creates HUD) set as `LVL_PuddingHills` GameMode override; **9 friends placed; PIE boots clean** — *2026-06-20/21*
- ☑ **PLAYABLE + tuned (Chris-confirmed 2026-06-21):** walk up → F/LMB → friends Happy Pop on 3rd squish → coins climb on HUD pill → respawn ~1.2s. Debug prints removed; dead input nodes pruned.
- ☑ `BP_SquishyFriend`: data vars + `Body`=`SM_peach_mochi` (scale 0.5, Z+36) + `Squish`/`Respawn` graph (cooldown→Joy→Happy Pop: hide+disable collision+1.2s respawn+award coins) — *deferred juice: squash-stretch + breathing bob*
- ☑ **Imported all 51 squishy 3D meshes** → `/Game/SquishySmash/Meshes/SM_<name>` (geometry-only); mapping = FriendId `Name`→`SM_<Name>`. *Claude, 2026-06-19*
- ☑ `BP_SquishyGameState` coins + `WBP_HUD` coin pill (pink rounded pill, top-left, Tick-reads SparkleCoins) — *2026-06-21*
- ☑ **Squish JUICE — squash-stretch + idle breathing bob** on `BP_SquishyFriend` (Tick-driven `SquishImpulse` spring via FInterpTo + Sin breathing on `Body` scale; no timeline/assets). PIE-clean on all 9 friends. — *Claude, 2026-06-21*
- ☑ **Happy Pop sparkle (batch B):** `NS_HappyPop` (from engine template `Systems/RadialBurst` — omnidirectional sprite burst + ribbon trails, auto-destroy) spawned at the friend's location on pop, before it hides. Compiles UpToDate, wired in `BP_SquishyFriend.Squish` pop branch. — *Claude, 2026-06-21* · *optional polish: recolor to warm pink sparkles + Sparkle Coin pickup feel*
- ☐ **Cleanup:** strip `BP_SquishyCharacter`'s leftover `AddSquishContext` debug prints (`ADD SQUISH CONTEXT RUN`/`…ADDED NOW`); fix coin-pill ✨ (U+2728) font glyph (use a font with it or drop the emoji).
- ☐ `BP_FriendPad` + spawner (optional — friends already self-respawn; pads = nicer placement/spread)
- ◐ `LVL_PuddingHills`: ☑ **stripped 53 template playground blocks** (kept Floor) + ☑ **re-grounded 9 friends to Z=0** (they'd snapped onto block-tops) + ☑ PlayerStart grounded + ☑ **`NS_HappyPop` sparkles recolored warm pink** (all 3 emitters, *Claude 2026-06-23*). ☐ Still: add hills/windmill landmark + warm bloom pass; per-friend tints/`M_JellyPlush` (friends are gray-tan geometry).
- ☐ Capture a 15–30s clip of the working loop → `Docs/MARKETING_CLIPS.md`

## M2 — Collect: capsule + Squishy Book
- ☐ `DT_SquishyFriends` — import the **pre-staged `Docs/data/PuddingHills_Friends.csv`** (001–016, exact rarities/coins; define `F_SquishyFriend` struct first)
- ☐ `BP_CapsuleStation` (free) + weighted-rarity pick + `WBP_CapsuleReveal`
- ☐ `WBP_SquishyBook` grid (locked silhouettes vs discovered card art); import card art to `Cards/`
- ☐ Discover flow wires capsule → book
- ☐ **Checkpoint:** earn coins → open capsule → new friend appears in the book.

## M3 — Buddy + first Sparkle Shard
- ☐ Equip Buddy from the book → `BP_BuddyFollower` floats/follows
- ☐ `BP_QuestManager`: tutorial ("wake 3"), then count woken → goal → `BP_SparkleShard` appears at windmill
- ☐ Hold-to-recover the shard → shard 1 complete (slice's quest beat)
- ☐ `SG_SquishyProfile` save/load (coins, discovered, buddy, shard, woken count) + autosave
- ☐ **Checkpoint:** full slice loop end-to-end, persists across restart.

## M4 — First art + feel pass (still slice)
- ☐ `M_JellyPlush` (subsurface+clearcoat) + kawaii-eye decals; per-friend tints from data
- ☐ Pudding Hills palette/lighting/bloom pass; Fredoka UI font; soft UI theme
- ☐ Squish/Pop SFX ("Pmf") from Roblox `assets/`; cozy ambience/music
- ☐ Guide (Soft Dumpling) + gentle toasts/tutorial copy (soft vocabulary per CLAUDE.md §5)
- ☐ **Vertical slice DONE** per CLAUDE.md §12 — record a 30s marketing clip.

## Later (post-slice, do NOT start early — CLAUDE.md §6)
Goo Coast + Moonlit Hollow + travel · full 48 roster bodies/art *(all 48 friend data already pre-staged in `Docs/data/`)* · duplicate→variant · daily capsule/quests/streak · Restore-the-Sparkle finale · hidden Sparkle Bits · playground/coaster · cosmetics boutique · shared-world/multiplayer + leaderboards · Steam packaging + Steamworks.

## Parking lot / decisions for Chris
- First-Shard goal N (pace ~3–6 min) · squish input scheme confirm · friends-visible count · project name if not `SquishySmashUE`.
