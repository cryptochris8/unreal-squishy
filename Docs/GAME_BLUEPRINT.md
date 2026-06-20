# GAME_BLUEPRINT — Squishy Smash: The Lost Sparkle (UE5.8)

Master design + proven tuning for the Unreal build. Distilled from the shipped Roblox game (`Docs/research/01-roblox-gameplay-systems.md`), the story/art bible (`02`), and the MCP workflow (`03`). **Single source of truth for what we build and the numbers we build it to.**

---

## 0. One-paragraph pitch

You arrive in **Pudding Hills**, a soft storybook land. Sleepy plush **squishy friends** dot the hills. Walk up and **squish** one — each squish adds **Joy**; about three squishes fills its **Joy Meter** and it gives a delighted **Happy Pop**, bursting into sparkles and paying **Sparkle Coins** before a new friend wakes nearby. Coins open a **free Sparkle Capsule** that **Discovers** a friend card into your **Squishy Book** (48 to collect); any discovered friend can be **equipped as a floating buddy** that follows you. Wake enough friends and a **Sparkle Shard** appears at a landmark — recover it to advance **The Lost Sparkle** quest. No combat, no losing, no timers — "solo-completable, better together."

## 1. Scope ladder (what's in the slice vs later)

**VERTICAL SLICE (M1–M4, single-player, Pudding Hills only):** character + movement, squish→Joy→Happy Pop→coins, sparkle FX, coins HUD, free Sparkle Capsule + rarity reveal, Squishy Book (locked silhouettes + equip), floating buddy, first Sparkle Shard beat, local save.
**LATER (post-slice):** Goo Coast + Moonlit Hollow + travel, full 48 roster art, duplicate→variant (Sparkly/Rainbow), daily capsule/quests/streak, the Restore-the-Sparkle finale, hidden Sparkle Bits, playground/coaster toys, cosmetics boutique, shared-world/multiplayer + leaderboards, Steam packaging.
**NEVER (brand):** combat/PvP/damage/hazards, weapons, horror, paid randomness/coin packs, trading.

## 2. Core loop & states

```
Explore Pudding Hills
  └─ find a sleepy Squishy Friend (idle: gentle breathing bob, eyes closed)
       └─ SQUISH (look at it within range + press Squish) → Joy += step, squash-stretch + "pmf" sound
            └─ Joy >= 1.0  →  HAPPY POP: swell, 26-particle sparkle burst, award coins, friend fades
                 └─ ~1.2s later a friend re-wakes nearby (respawn)
  └─ coins accrue → free Sparkle Capsule available
       └─ open capsule → weighted-rarity reveal → Discover new friend → enters Squishy Book
            └─ equip a discovered friend → floating Buddy follows you
  └─ wake N friends total → first Sparkle Shard manifests at the windmill/landmark
       └─ hold-to-recover the shard → quest beat complete (slice ends here; later: unlock Goo Coast)
```

No fail state. No health. The only "meter" is per-friend **Joy** (0→1).

## 3. Proven tuning (from the Roblox game — start here, then tune by feel)

| Thing | Value | Notes |
|---|---|---|
| Squish anti-spam cooldown | **0.12 s** | per friend; ignore squishes faster than this |
| Joy per squish | **+0.34** | → **3 squishes** to fill (Joy 0→1) |
| Happy Pop reward | **`CoinReward`** | base only in MVP (later × surge ×2 × coinBoost ×1.25) |
| `CoinReward` by rarity | **~8 / 20 / 50 / 140** | common / rare / epic / mythic (exact per-friend in data — see §6) |
| Respawn delay after pop | **~1.2 s** | a new sleepy friend wakes at/near the pad |
| Sparkle burst | **~26 particles** | Niagara one-shot; soft pastel motes + bloom |
| Movement | stock walk/run + jump | **no fall damage, no ragdoll** (gentle feel) |
| First Shard goal | wake **N** friends in the land | tune N for ~3–6 min to first shard (Roblox used a per-land goal in `GameConfig`/`ZoneConfig`) |

Pull exact per-friend numbers from `C:\Users\chris\Roblox-squishy\src\ReplicatedStorage\Shared\SquishyDefinitions.lua` and `data/raw/`.

## 4. Interaction model (UE decision)

Roblox used `ClickDetector.MouseClick` (mouse on a 3D model, server-adjudicated). For a Steam third-person game that must also feel good on **gamepad**, use:

- **Primary:** "**look + squish**" — a camera/character forward **line trace** (~350–450 cm). The friend under the reticle (or nearest in a small cone) highlights; press **Squish** (`IA_Squish` = LMB / gamepad bottom face button) to squish it. Repeat to fill Joy. This works identically for mouse and pad.
- Friend shows a soft outline + a floating Joy ring when targeted.
- (Optional later) walk-into proximity auto-highlight for the youngest players.

Server-authoritative adjudication is a **later/multiplayer** concern; in single-player the squish resolves locally in the friend actor.

## 5. Systems → Unreal mapping

| System | UE realization |
|---|---|
| Player | `BP_SquishyCharacter` (Character; CharacterMovement, no fall damage; spring-arm + camera). `IMC_Squishy` + `IA_Move/Look/Jump/Squish/Interact` (Enhanced Input). |
| Squishy friend | `BP_SquishyFriend` (Actor): mesh + face decal + breathing **Timeline**; `JoyComponent` (float 0→1, cooldown); on-fill → **Happy Pop** (squash-stretch, `NS_HappyPop` Niagara, award coins via GameState, hide + respawn timer). Driven by a `DA_SquishyFriend` data asset / `DT_SquishyFriends` row. |
| Friend spawning | `BP_FriendPad` actors placed across Pudding Hills; a `FriendSpawner` (in GameMode or per-pad) seeds a friend and respawns after pop. |
| Economy | Sparkle Coins on `BP_SquishyGameState` (single-player) → `WBP_HUD` coin pill. |
| Capsule (gacha) | `BP_CapsuleStation` (proximity-press) → `CapsuleService` logic: weighted rarity → pick an undiscovered (or duplicate) friend → `WBP_CapsuleReveal` animation → mark Discovered. **Free, always.** |
| Collection | `WBP_SquishyBook` grid (48 slots): discovered = card art; undiscovered = locked silhouette. Select → **Equip Buddy**. |
| Buddy | `BP_BuddyFollower` — spawns the equipped friend as a floating companion with smooth follow/bob. |
| Quest (Lost Sparkle) | `BP_QuestManager`: count friends woken in the land; at goal, reveal `BP_SparkleShard` at the landmark; **hold-to-recover** → mark shard 1. (Later: unlock next land / finale.) |
| Save | `SG_SquishyProfile` (`USaveGame`): coins, `discoveredIds[]`, `equippedBuddyId`, `shardProgress`, `friendsWoken`, tutorialDone. Autosave on change + on quit; load on start. |
| Tutorial | `BP_QuestManager` first objective: "wake up 3 sleepy friends" → small coin reward + first capsule. |
| Guide NPC | Soft Dumpling (Pudding Hills guide) gives gentle prompts (`WBP_Toast`). |

## 6. Content: characters & data

- **Roster:** 48 launch friends in 3 packs of 16 (**8 common / 4 rare / 3 epic / 1 mythic** each). Pudding Hills = **Squishy Foods** pack (IDs **001–016**), guide **Soft Dumpling** (001). Friends are **mechanically identical** — they differ only in 3D shape/skin, card art, signature sound (Pmf/Sploink/Thup), and `CoinReward`. Duplicates aren't misses — later they upgrade Discovered → ✨Sparkly → 🌈Rainbow.
- **Data source of truth:** port `SquishyDefinitions.lua` / `data/raw/` into a UE **`DT_SquishyFriends`** DataTable (or per-friend `DA_` assets): `Id, Name, Pack, Zone, Rarity, CoinReward, Sound, BodyArchetype, BaseColor, CardArt`.
- **Bodies (MVP):** mirror the Roblox `SquishyModelFactory` approach — **primitive-built squishy shapes** (sphere/capsule/blob + ears/eyes via decals), not imported meshes. Fast, on-brand, and editable via MCP. Real sculpted meshes are a later art pass.
- **Card art:** the finished 48 trading-card images live in `C:\Users\chris\Squishy-smash\` (and Roblox `tools/card_art/`). Import to `Content/SquishySmash/Cards/` for the Squishy Book.

## 7. Look & feel (from the art bible)

- **Style A (game/characters/UI/cards):** glossy "toy/card" kawaii — big catch-light eyes, soft rounded silhouettes, rarity-escalating sparkle/bloom. **Character material = "jelly plush":** stylized PBR with **subsurface + clearcoat**, emissive **kawaii-eye decals**.
- **Pudding Hills palette:** peach/cream + brand pink `#FF8FB8`, sun-gold `#FFD36E`, jelly-blue `#7FE7FF`. Soft, warm, glowy; heavy bloom; pastel-bright but never muddy.
- **Signature sound:** wet, close-mic, ASMR-satisfying squishes — Pudding Hills cue is **"Pmf."** (Reusable mastered SFX exist in the Roblox `assets/`.)
- Fonts: **Fredoka** (game/UI). Story/cutscene work (Style B painterly) is later.

## 8. Definition of done — see CLAUDE.md §12.

## 9. Open design decisions (flag to Chris when reached)
- Exact **First Shard** friend-count goal N (pace to ~3–6 min).
- Squish input: confirm "look + click/press" vs proximity auto.
- How many friends visible per land at once (Roblox: ~12 pads; pick for UE perf/feel).
- Whether the MVP shows all 16 Pudding Hills friends in the Book or a starter subset.
