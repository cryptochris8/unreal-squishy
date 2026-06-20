# Squishy Smash — Roblox Gameplay & Systems Teardown (for the UE5 / Steam port)

**Source project:** `C:\Users\chris\Roblox-squishy`
**Target:** *Squishy Smash: The Lost Sparkle* — Unreal Engine 5, Steam.
**Author of this doc:** reverse-engineering pass over the Luau codebase + design docs.
**Date:** 2026-06-19.

> **Single most important framing for the port team:** Despite the title "Smash", **this is NOT a fighting / brawler / PvP game.** There is no combat, no damage, no PvP. "Smash" = giving a sleepy plush "squishy friend" a *silly squish* (a click) to fill its **Joy Meter** until it **Happy Pops** into sparkles and coins. The genre is a **wholesome, cozy, server-shared collector/explorer/sim for ages ~4–8** (broadened to general Roblox/Steam audience). Every player-facing word is deliberately soft (see `docs/02_KID_FRIENDLY_RULES.md`). Keep that DNA or it stops being recognizably Squishy Smash.

---

## 0. Project at a glance

| Aspect | Value |
|---|---|
| Engine / tooling | Roblox + Rojo 7.6.1 (`rokit.toml`), synced from local files |
| Place name | `SquishySmash` (`default.project.json`), `servePlaceIds: 105594294243426` |
| Architecture | **Server-authoritative**, `FilteringEnabled = true`, `StreamingEnabled` on (distant lands replicate only when near) |
| Player count | Small shared server (sized as "1→N scales", validated as a 4-player family playtest; no hard cap enforced in code — Roblox default ~50) |
| Persistence | DataStore (`SquishyPlayerData_v1`) with ProfileStore-style session locking |
| Content size | 48 launch friends + 8 event friends + 3 "Family" friends = **59 collectible characters**; 3 lands; ~18k lines of Luau |
| Monetization | Cosmetics + 3 QoL Game Passes + 6 premium-cosmetic Dev Products. **Capsules are FREE forever** (no paid randomness — deliberate, for the 6–9 audience / "Paid Random Items" policy) |
| Code layout | `src/ReplicatedStorage/Shared` (configs + data), `src/ServerScriptService/Server` (authoritative services), `src/StarterPlayer/StarterPlayerScripts` (client UI + local physics/FX) |

The repo's `CLAUDE.md` is effectively a self-authored architecture map and the best secondary source after the code itself. `archive/qb1/` is an **unrelated, abandoned** football-throwing prototype (RoundService/ThrowService/TargetService) — ignore it. `assets/` holds only 8 sample trading-card images (covered by the art agent).

---

## 1. Core gameplay loop

### 1.1 The one-paragraph loop
The player spawns in **Pudding Hills**, walks around a cozy storybook land, finds **sleepy squishy friends** dotted on pads, and **clicks each one to "squish" it** — every click adds **Joy**; ~3 clicks fill its **Joy Meter** and it **Happy Pops** into sparkles, paying out **Sparkle Coins** and waking a new sleepy friend. Coins are spent at a **Sparkle Capsule** machine to **Discover** new friend **cards** (a kind, free, never-gambling gacha) which fill the **Squishy Book** (48-card album). Threaded through this is **The Lost Sparkle** quest: each land hides one of three **Sparkle Shards** — wake enough friends in a land → its shard appears at a landmark → hold-to-recover it → the **next land unlocks** (Pudding Hills → Goo Coast → Moonlit Hollow). Recovering all 3 shards triggers the **Restore the Sparkle** finale.

### 1.2 Layered goal structure (the retention design, `docs/11_GAMEPLAY_V2_DESIGN.md`)
- **This minute:** fill one friend's Joy Meter → Happy Pop.
- **This session:** a daily quest, the free daily capsule, find today's hidden Sparkle Bits / Story Pages, climb the Sparkle Surge meter.
- **This week+:** recover a shard, finish a land, restore the Sparkle, complete the 48-card Book, save up for the Friend of the Week.

### 1.3 "Win / lose" conditions
There is **no lose state and no fail state by design** — you cannot die (ragdoll/falling-down states are disabled, see §3.4), cannot lose progress, cannot be attacked. The closest thing to "winning" is **completing The Lost Sparkle** (all 3 shards → finale, one-time **+1000 coins**, `GameConfig.FinaleRewardCoins`) and **completing the 48-card Squishy Book**. The game is explicitly **"solo-completable end to end, better together."**

### 1.4 Match / round structure
**None.** This is a **persistent shared world**, not a round/match game. Players come and go; per-player progress persists across sessions via DataStore. The only timed/recurring beats are server-wide events:
- **Sparkle Surge** — a shared meter every Happy Pop fills; full = **60s of x2 coins for everyone** (`SocialConfig.Surge*`).
- **"Everybody Squish!"** — every ~7 min, **golden friends** spawn at the busiest land; a shared golden-pop goal → +150 coins for everyone online (`SocialConfig.Event*`).

---

## 2. Controls & movement

**This is the biggest surprise for the port and must be understood precisely.**

### 2.1 Movement
- **Stock Roblox humanoid locomotion** — default WASD + mouse-look camera, default jump (Space / mobile jump button). **No custom movement code at all.** Camera occlusion is `Invisicam` (`default.project.json`).
- `ScreenOrientation: LandscapeSensor` (mobile-friendly). Game runs on PC, mobile, tablet.

### 2.2 The "squish" / smash action — **NOT a player input action**
- A squish is a **`ClickDetector.MouseClick`** on the friend's *Model* (so ears/wings/toppings are all clickable). See `SquishService.buildSquishy` (`src/ServerScriptService/Server/SquishService.lua:62-67`).
- **The click fires on the SERVER.** The client never decides a squish happened; it only plays FX in response to the server's `SquishResult` broadcast. There is **no keyboard attack, no melee, no aim, no projectile.**
- `ClickDetector.MaxActivationDistance = 32` studs — you can squish from a short distance, you don't have to be touching.
- **Implication for UE5:** "attack" maps to a **click/tap-to-interact on a world actor within range**, server-validated, not to a combat input. (You may *choose* to upgrade this to a satisfying physical "boop/punch" melee in UE5 — see §10 — but the original is click-to-interact.)

### 2.3 Other interactions
All non-squish interactions are **`ProximityPrompt`** (walk up, hold a key — `E` on PC):
- Open a **Sparkle Capsule** (hold).
- Talk to a land **Guide** (gives the shard clue).
- **Recover a Shard** (`HoldDuration = 0.4`, `MaxActivationDistance = 16`).
- **Travel Pad** to another land (hold 0.3s).
- **Boutique** stall ("Browse"), **Friend of the Week** tent ("Befriend (X coins)"), **Family** pedestals ("Say hello"), **Gift** prompt on every other player ("🎁 Give a Gift", hold 0.25), playground ride boarding prompts, Story Pages / Sparkle Bits (walk-into pickup, see §3.5).
- **HUD buttons** (mouse/touch) open panels (Book, Daily Quests, Magic Words, Storybook, gift picker, etc.) — see §6.4.

### 2.4 Abilities / specials
**None.** No dashes, no powers, no cooldown abilities, no inventory-of-moves. The only "specials" are passive perks bought with Robux (Coin Boost, second buddy, VIP aura) and cosmetic buddies/trails. This is intentional: the kindness loop is never gated behind skill.

---

## 3. "Combat" / physics mechanics

There is no combat. This section instead documents the **squish/Joy/Pop loop**, the **squish FX**, and the **playground/ride physics** that give the game its tactile "squishy" feel — the things a UE5 port must reproduce with Chaos.

### 3.1 The squish → Joy → Happy Pop loop (server-authoritative)
File: `src/ServerScriptService/Server/SquishService.lua`. Tunables: `src/ReplicatedStorage/Shared/GameConfig.lua`.

Per friend the server tracks attributes: `Joy` (0..1), `Sleepy` (bool), `DefId`, `PadIndex`, `Popped`.

On each `MouseClick` (`SquishService.handleSquish`):
1. Ignore if friend already `Popped`.
2. **Per-player, per-friend cooldown** = `GameConfig.SquishCooldownSeconds = 0.12s` (gentle anti-spam, stored on the friend so it self-cleans on pop).
3. `incSquish(player)` (lifetime `TotalSquishes`).
4. `Joy = min(1, Joy + GameConfig.JoyPerSquish)`, where **`JoyPerSquish = 0.34`** → ~3 squishes to fill. Set `Sleepy = false`.
5. If `Joy >= 1` → **Happy Pop:**
   - `coins = def.CoinReward` (per-character, 8–140; see §4) `× goldenMult (×3 if golden) × coinMultiplier(player)` where `coinMultiplier = SurgeSurge(×2) × CoinBoostPass(×1.25)` — these stack, so surge+boost = **×2.5** (`Main.server.lua:99-101`).
   - `addCoins`, `incHappyPop` (lifetime `TotalHappyPops`).
   - Broadcast `SquishResult {objectId, defId, joy=1, popped=true, byUserId, coins}` to **all clients** (everyone sees the pop).
   - Free the pad, set `Popped=true`, disable the ClickDetector, destroy the model after `HappyPopHoldSeconds = 1.1s` (lets the pop animation finish), schedule a respawn after `HappyPopRespawnSeconds = 1.2s`.
   - Fire hooks: `onHappyPop` (→ tutorial, shard quest, daily quest, surge meter), `onGoldenPop` (→ Everybody Squish event).
6. Else (not yet full) → broadcast `SquishResult {joy, popped=false}` so the friend visibly wobbles for everyone.

**Spawning:** `SquishService.init(zoneGroups)` builds a flat pad list across all lands; each pad keeps a `packId`+`zone`; a sleepy friend is picked randomly from that pad's pack (`pickDefForPack`), built via `SquishyModelFactory`, scaled `0.92–1.12` for hand-placed variety, and parented under a `Squishies` folder. Golden event friends are spawned off-pad (the event owns their lifetime) and recolored via `SquishyModelFactory.applyGolden`.

> There is **no health, no knockback, no hitboxes, no damage**. "Joy" is the only per-friend state and it only goes up. Map "Joy Meter" to a simple 0..1 float component per interactable in UE5; it is the spiritual inverse of an HP bar.

### 3.2 The squishy *body* (what makes them feel squishy)
File: `src/ServerScriptService/Server/SquishyModelFactory.lua` (~675 lines). Each friend has a **part-built 3D body** from **~17 parameterized archetypes** (dumpling, bun, mochi, cube, puff, rice ball, flan, blob, orb, pad, capsule, drop, pop-ball, bunny, bat, ghost, kitty, critter), hand-skinned per friend (name-true palettes; e.g. Strawberry Dumpling is pink with a calyx knot, Moon Bat Blob has wedge ears + wings + a neon moon).
- Each model: `PrimaryPart` is a `Ball` part named **"Body"**, centered at origin; shape base sits ~2 studs below Body center (pads place pivots at y=2). A **`HatOffset`** attribute marks where cosmetic hats sit.
- (In the live build, the 48 launch friends also have **Meshy image-to-3D meshes** resolved into `ServerStorage.MeshBodies`; the part-built archetypes are the fallback and what the 8 event friends use. The art/mesh agent owns this — the meshes themselves are not in `src/`, they live in the published place.)

### 3.3 Squish / Pop visual FX (client)
File: `src/StarterPlayer/StarterPlayerScripts/SquishFx.lua` (~400 lines). Three layered effects driven off the server's `SquishResult`:
- **Breathing bob** (always): whole-model `PivotTo` vertical sine, `sin(t*1.9 + phase) * 0.22` studs, per-friend phase offset so they don't bob in sync.
- **Squish-spring** (on squish): a ~0.45s scale animation — compress to 84% in 0.07s, then a 7-cycle damped cosine spring back (`1 - 0.16*(1-k)*cos(k*7)`). Applied to the whole model. The Joy bar fills instantly; the face flips from sleepy (closed eyes, zZz) to awake (bead eyes + shine, rosy cheeks, smile).
- **Swell-and-fade pop** (on pop): 0.3s scale up to ~1.5× while all parts fade transparency to 1 ("evaporates into a happy puff"), plus a **26-particle sparkle burst** colored by rarity, the **Happy Pop** sound (per-friend signature variant, ±8% pitch), and a floating `+X` coin label.

A BillboardGui per friend shows the name, rarity-colored Joy bar, and "zZz" (visible only at Joy=0).

### 3.4 No ragdoll — safety by design
File: `ClientController.client.lua` (~lines 180-190) disables `FallingDown` and `Ragdoll` humanoid states on every respawn, on the owning client. Kids never trip, never ragdoll, never get stuck mid-bounce. **This is a core feel decision** — adventures stay gentle.

### 3.5 Hidden collectibles: client-rendered, server-validated
File: `src/StarterPlayer/StarterPlayerScripts/SparkleBits.lua` (+ `SparkleBitService.lua`, `StoryPageService.lua`).
- **Sparkle Bits** (26 across all lands, `GameConfig.SparkleBit*`): glowing neon balls the **client renders and detects** — every Heartbeat it checks if the local root is within `SparkleBitPickupRadius = 7` studs, then fires `CollectSparkleBit`. The **server validates** the pickup with a sanity range `SparkleBitClaimRange = 18`, awards `SparkleBitCoins = 25`, marks it collected, and broadcasts. Finding all → `SparkleBitAllBonus = 300`. **Bits refresh daily.** A 2.5s client watchdog restores a gem if the server never confirms.
- **Story Pages** (18 watercolor book spreads, 6/land): identical pattern, +25 coins each, full-set +300, opens a page-turning viewer (the book *inside* the game).

### 3.6 Playground & ride physics (the tactile heart for UE5)
The lands are dotted with **rideable attractions**. The Roblox technique is **server-authoritative kinematic motion** (anchored parts `CFrame`d every Heartbeat along **arc-length-parameterized splines**), with riders held by **native Seat welds**, plus a **client-owned bounce** system. None of it is rigid-body physics — it is scripted, deterministic, frame-rate-independent, and "kid-gentle" by careful number choices. Details:

**Bounce system (the one genuinely physics-y thing), `PlaygroundService.lua` + `BouncePads.lua`:**
- Server tags a surface `SquishyBouncy` and sets attributes `BounceVelocity` (Vector3), optionally `PartyUntil` (server time) + `PartyVelocity` (boosted launch during a "together" window).
- **The CLIENT applies the bounce** (`BouncePads.lua`): on local `Touched`, debounce 0.45s, read `BounceVelocity` (use `PartyVelocity` if `now < PartyUntil`), then `humanoid:ChangeState(Jumping)` + `root.AssemblyLinearVelocity = velocity`. **Why client-side:** velocity writes and humanoid-state changes only "stick" on the part owner (the client owns its character). The server's job is the *juice* (squash tween, sparkles, the party-window bonus), published via attributes.
- Example: **Bounce Bog** (Goo Coast) = a jelly trampoline drum, `BounceVelocity (0,82,0)`, party bonus `(0,102,0)` when 2+ bounces land within 0.6s, opening a 4s party window. Each bounce squashes the drum and bursts sparkles.
- **Mushroom Hop Trail** (Moonlit) = 6 spring caps that bounce you *toward the next cap* with a directional impulse (`~24 horizontal + 62+altitude×6 vertical`), pitch rising per cap.

**Rideable transport (all the same arc-length spline pattern):**
- **The Sparkle Express coaster** (`CoasterService.lua`, ~580 lines): a **closed Catmull-Rom spline** sampled at ~2-stud intervals into an arc-length table; cars are anchored parts `CFrame`d each Heartbeat at `(rideDist - carSpacing*i) % totalLength`, with **gentle banking** computed from the turn rate and clamped to **≤10°** (`MaxBank`), lerped smoothly. Riders sit in native Seats (welds carry them). Kid-gentle numbers: **cruise 16 studs/s, accel ~6, dwell 14s, 1–2 laps/ride.** **Safe exit:** if a rider leaves the seat mid-ride they "poof" back to the platform with a friendly toast.
- **Pudding Plunge** twin racing slides (open splines, riders in jelly "ride rings", accelerate 8→30 studs/s into splash pools), **Lazy Goo River** (closed-loop convoy of 6 drifting rings at 5 studs/s, bobbing), **Firefly Zip Line** (open spline trolley, 20 studs/s), **Sparkle-Pop Cannon** (parabolic ballistic flight — `Lerp + 4k(1-k)` apex curve, 1.5s, 22-stud apex), **Spoon Seesaw** (occupancy-driven tilt, no mass physics so nobody gets pinned), **Pudding-Cup Spinner**, **orchard/pier swings** (independent sine pendulums), **Bounce Bog**.
- **Safe-exit & no-fail** is the consistent rule across all rides: a rider who bails is gently re-placed on solid ground; you can never get stuck or fall to a fail.

---

## 4. Characters (the collectible roster)

Friends are **collectible cards/companions, not avatars** — the player avatar is the stock Roblox character. Data source of truth: `src/ReplicatedStorage/Shared/SquishyDefinitions.lua` (generated from `data/raw/*.json`), queried via `SquishyData.lua`.

### 4.1 Roster size & structure
- **48 launch friends** (`ReleaseType = "launch"`), the Squishy Book.
- **8 event friends** (`ReleaseType = "event"`-ish; Dumpling Squishy pack) — the **Friend of the Week** rotation pool, shown under the Book's "Events" tab.
- **3 Family friends** (`ReleaseType = "family"`) — Chris's daughters, one guardian per land, earned by restoring each shard (the "⭐ Family" Book tab). **Never sold, never random** (Rarity `family`, weight 0).
- Total **59** characters.

### 4.2 Per-character data fields (`SquishyDefinitions.lua`)
`Id`, `DisplayName`, `CardNumber` ("001/048"), `Rarity`, `PackId`/`PackName`, `Zone`, `Feeling`, `SignatureSound` (Pmf/Sploink/Thup), `Category`, `ThemeTag`, `CoinReward`, `UnlockTier`, `ParticlePreset`, `DecalPreset`, `ImageAssetId` (card art), `ReleaseType`. (Legacy iOS fields like `burstSound`/`deformability`/`elasticity`/`gooLevel` are noted in `docs/04` as preserve-for-later; not all are present in the generated table.)

### 4.3 Differences between characters
Characters are **not mechanically asymmetric in play** — they all squish identically. They differ in:
- **3D shape & skin** (archetype/palette), **card art**, **signature squish sound**, **particle/decal preset on pop**.
- **`CoinReward`** (the only "stat"): commons 8–18, rares 16–24, epics 28–36, mythics 75–140. Higher rarity = more coins per pop *and* rarer in the capsule.
- **Zone/pack membership** (which land they wander / which capsule discovers them).

### 4.4 Rarity tiers (`RarityConfig.lua`)
common (weight 55), rare (25), epic (12), legendary (6), mythic (2), family (0 = unrollable). Note the capsule's own weight table (`CapsuleConfig.lua`) is slightly different and omits legendary: `{common 50, rare 26, epic 14, mythic 7, legendary 3}` — and `pickRarity` auto-skips any rarity a pack lacks and renormalizes.

### 4.5 The three lands' packs (`PackConfig.lua`, `docs/01_UNIVERSE_CANON.md`)
| Pack | Land | Feeling | Sound | Guide | Card range |
|---|---|---|---|---|---|
| Squishy Foods (`launch_squishy_foods`) | Pudding Hills | Comfort | Pmf | Soft Dumpling | 001–016 |
| Goo & Fidgets (`goo_fidgets_drop_01`) | Goo Coast | Surprise | Sploink | Goo Ball | 017–032 |
| Creepy-Cute Creatures (`creepy_cute_pack_01`) | Moonlit Hollow | Brave-Cuddle | Thup | Blushy Bun Bunny | 033–048 |

**Mythic "core" guardians** (1 per land, storybook-special, not enemies): Celestial Dumpling Core (016), Singularity Goo Core (032), Mythic Plush Familiar (048). **Family:** Apple Addy (Pudding), Eggy Ellie (Goo), Hot Dog Heidi (Moonlit).

### 4.6 Unlocks / progression of characters
- Friends are **Discovered** by opening Sparkle Capsules (per-land capsule draws from that land's pack), by befriending the Friend of the Week (fixed 400-coin price), or by restoring shards (Family).
- **Duplicate → variant** (`VariantConfig.lua`): a duplicate isn't a miss — it "shines up" the friend **Discovered → ✨Sparkly (lvl 1, +30 coins) → 🌈Rainbow (lvl 2, +60 coins)**, capped at 2; further dupes pay `MaxDuplicateCoins = 25`. Variant buddies get a visible particle aura. This is long-tail depth **with no new art**.
- **Equip a buddy:** any discovered friend can be equipped as a floating companion (1 slot free, 2nd slot via the Extra Buddy Slot pass).

---

## 5. Game modes, maps / arenas

### 5.1 "Modes"
There is **one mode: the persistent shared cozy world.** No competitive/alternate modes. Layered on top are two **server-wide co-op events** (Sparkle Surge, Everybody Squish — §1.4) and **cross-server leaderboards** (Top Friend Finders / Joy Champions).

### 5.2 The three lands (the "arenas") — `WorldService.lua` (~2475 lines), `ZoneConfig.lua`
Lands sit on **separate 320×320 ground plates, 600 studs apart** (centers at x = 0 / 600 / 1200, z = 0), connected by **Travel Pads**. With `StreamingEnabled`, a distant land only replicates when you're near. Each land is bespoke-built and stretched by `ZoneConfig.Spread = 1.45` so districts reach the plate edges ("use the WHOLE land"). Each land has: a spawn, ~12 friend pads (a 3-pad starter cluster at spawn + pockets behind landmarks), its own **Sparkle Capsule**, a **Guide** NPC, a **Shard pedestal** + **Family pedestal**, a **travel hub**, and **wayfinding paths** for the 6-year-old.

| Land | Theme | Spawn | Shard spot | Wake goal → reward | Landmarks | Attractions |
|---|---|---|---|---|---|---|
| **Pudding Hills** (always unlocked) | Warm golden food valley | (0, .5, 34) | (68,0,-58) orchard edge | **8** → **150c** | The Sparkle orb, pudding mountain, cottage village, soda-falls, orchard, candy-cane arches, travel hub w/ leaderboards | Sparkle Express coaster, Pudding Plunge slides, orchard swings, Spoon Seesaw, Cup Spinner; Boutique stall; Friend-of-the-Week tent |
| **Goo Coast** (unlock after Pudding shard) | Glossy bouncy beach + goo sea | (600,.5,34) | (668,0,-58) | **10** → **250c** | Jelly dunes, wooden pier (friend naps at the END), sandcastle, tide pools, beach huts, rocky cove, candy-striped lighthouse | Bounce Bog, pier swing, Firefly Zip Line, Lazy Goo River, Sparkle-Pop Cannon |
| **Moonlit Hollow** (unlock after Goo shard) | Soft-spooky twilight glade (never scary) | (1200,.5,34) | (1268,0,-58) | **12** → **400c** | Moonpool + crescent moon, giant glowing-mushroom grove, mushroom cottages, stargazing stone circle, lanterns, fallen log, fireflies, sparkle-fall | Mushroom Hop Trail, Firefly Zip Line, Crescent Swing |

### 5.3 Hazards / interactive elements
**No hazards.** Nothing damages or threatens the player. "Interactive elements" are all positive: friend pads, capsule machines, guides, shard pedestals, travel pads, the boutique, the weekly tent, family pedestals, gift prompts, hidden Sparkle Bits & Story Pages, and the rideable playground attractions (§3.6).

---

## 6. Meta systems

### 6.1 The player profile (the save schema) — `PlayerDataService.lua` (~909 lines)
DataStore `SquishyPlayerData_v1`, key `Player_<UserId>`, `DATA_VERSION = 1`. **ProfileStore-style session locking:** `_lock = {id, ts}`, `LOCK_TTL = 240s`, acquire via `UpdateAsync` (7 attempts, 5s wait); refuses live cross-server sessions, steals stale locks. **Autosave every 90s** (refreshes lock); `BindToClose` flushes + releases on shutdown. **Backward/forward-compatible** serialization (unknown keys round-trip).

**Full profile fields:**
`SparkleCoins`, `TotalSquishes`, `TotalHappyPops`, `Discovered{id→bool}` + `DiscoveredCount`, `EquippedBuddyId`, `EquippedBuddyId2`, `TutorialDone`, `FirstCapsuleClaimed`, `Shards{zone→{progress, collected}}`, `SparkleBits{id→bool}` (daily reset), `Variants{id→level}`, `LastDailyCapsuleDay`, `StreakDays`, `LastPlayDay`, `DailyQuests{day, progress, claimed}`, `SparkleRestored`, `Cosmetics{Owned, Equipped}`, `RedeemedCodes{}`, `Room{Owned, Placed}`, `FirstDayPaid{}`, `StoryPages{}`, `Gifting{Day, Sent}`, `PremiumReceipts{}` (idempotence guard).

**Leaderstats (the in-game roblox leaderboard):** "Sparkle Coins", "Friends" (DiscoveredCount), "Happy Pops".

### 6.2 Currency — Sparkle Coins (the single soft currency)
**Earn:** Happy Pops (`CoinReward × multipliers`), capsule duplicates (+25/30/60), Sparkle Bits (+25, +300 all), Story Pages (+25, +300 all), daily quests, login streak (`StreakBaseBonus 20 + StreakPerDay 10 × min(streak,7)`), shards (150/250/400), finale (+1000), promo codes (150–300), received gifts, the Surge ×2 / CoinBoost ×1.25 multipliers.
**Spend:** Sparkle Capsules (100, first free), Boutique cosmetics (150–600), Friend of the Week (400), gifting coins to others. **There is no way to buy coins with Robux** (deliberate economy-integrity choice).

### 6.3 The Sparkle Capsule (free, kind gacha) — `CapsuleService.lua` + `CapsuleConfig.lua`
One capsule per land (draws that land's pack). `tryOpen(player, key, freeOverride)`: build rarity pool → weighted `pickRarity` → **validate a friend can be given BEFORE charging** → free if first capsule / daily / override, else spend `Cost = 100` → pick random friend in that rarity → `discoverCard` returns isNew. Duplicate → variant upgrade + bonus coins. Fires `CapsuleResult` (the reveal). **No pity counter in MVP.** Player-facing language strictly avoids "pull/gamble/loot".

### 6.4 HUD & panels (client) — `HudUI.lua`, `ClientController.client.lua`
Always-on pills: **Sparkle Coins**, **Friends X/48** (launch only), **✨ Bits X/N**, a **quest banner** (current objective). Buttons → panels: **📕 Squishy Book** (`CollectionBookUI` — 720×560, tabs All/Pudding/Goo/Moonlit/Events/⭐Family, locked friends show "?" silhouette, detail modal with **Equip Buddy**), **🎁 Free Daily Gift** (claims daily capsule, pulses when ready), **📋 Daily Quests** (`DailyUI` — streak + 3 quests w/ progress bars), **🎟️ Magic Words** (`CodesUI` — type a code), **📖 Storybook** (`StoryPagesUI` — page viewer). Plus event-opened panels: **Boutique**, **Gift picker**, **Room catalog**. Capsule opens show a full-screen **`CapsuleRevealUI`** (wobble → flip → card reveal → headline "New Friend Discovered!" / "Friendship Bonus!" / "A gift from …!"). `ToastUI` shows soft top-center banners. Layout has **desktop and compact/phone variants**. Owner-only debug buttons (Reset, force Event/Surge, grant pass) are double-gated to the place owner.

### 6.5 Monetization (Phase D — ethical, cosmetics + convenience only)
`MonetizationConfig.lua`, `MonetizationService.lua`, `BoutiqueService.lua`.
- **3 Game Passes** (live ids): **Extra Buddy Slot** (99 R$ — walk with 2 buddies), **Coin Boost** (149 R$ — `CoinBoostMultiplier = 1.25` on every pop), **Sparkle Club VIP** (249 R$ — golden buddy crown/aura + welcome + `VipExtraDailyGifts = 1`). Ownership checked on join (retried 3×) + instant in-session grant on purchase.
- **6 premium cosmetics** as **Developer Products** (79–249 R$): Strawberry Beret, Rainbow Heart Balloon, Unicorn Horn, Comet Trail, Golden Halo, Aurora Ribbon. `ProcessReceipt` is **idempotent** (PremiumReceipts guard), grants into the same `Cosmetics.Owned`/`Equipped` as coin items, auto-wears, **saves before consuming the receipt**, hands temp/unknown receipts back to Roblox.
- **Coin-only Boutique cosmetics** (`CosmeticsConfig.lua`): 6 hats (150–400), 4 trails (250–600), 2 balloons (200) — one slot per type (hat+trail+balloon simultaneously), auto-worn on purchase, replicated so everyone sees them.
- **Hard rules** (`docs/11`): **capsules stay free**, **no coin packs**, **nothing random sold for Robux**, **every friend earnable F2P**, **sell style & convenience, never power**.

### 6.6 Daily / weekly / social meta
- **DailyService:** free daily capsule (`ClaimDailyCapsule`), 3 rotating daily quests (`DailyQuestConfig.forDay`, auto-grant on completion), gentle login **streak** (resets to Day 1 not 0 on a miss). All UTC-day based. Sparkle Bits + Story Pages refresh daily.
- **WeeklyService — Friend of the Week:** a visiting tent; 1 of the 8 event friends rotates in by UTC week (`WeeklyConfig.weekIndex`), befriended at a **known fixed 400-coin price** (never random), full reveal + server shout, countdown sign.
- **CodeService — Magic Words:** server-only promo table (client can't datamine): SPLOINK/THUP 150, PMF 200, EVERYSQUISH 250, LOSTSPARKLE 300; one-time, normalized input, persisted per profile.
- **GiftService — Gifting v1:** a 🎁 prompt on every other player; give preset coins **or SHARE a discovered friend** (recipient gets the discovery + reveal, **giver keeps theirs** — sharing, never trading). 5 gifts/day (+1 VIP), same-server walk-up only, range + cooldown + daily-limit validated server-side. **No trading of any kind** (deliberate: you can never be talked out of your collection).
- **LeaderboardService:** two **OrderedDataStore** cross-server boards ("Top Friend Finders" = DiscoveredCount, "Joy Champions" = TotalHappyPops) rendered on physical signs at the Pudding Hills hub, refreshed every 120s.
- **Squishy Room** (`RoomService.lua`, `RoomConfig.lua`) — a personal decoratable room (furniture Owned/Placed, a sky neighborhood) — present but secondary.
- **My First Day** (`FirstDayService.lua`) — a small first-session checklist that watches 5 signals (first squish, room visit, etc.).

---

## 7. Multiplayer / networking model

- **Fully server-authoritative.** The server owns: all squish/Joy/Pop logic & coins, capsule RNG, all purchases & receipts, shard quest state, gift/code/cosmetic validation, ride motion, and persistence. `FilteringEnabled = true`. Clients only render and *request* (via Remotes).
- **Shared persistent world, individual progress.** No matchmaking, no lobby, no rounds — players join the running world; progress persists per-player via DataStore with session locking (one live session per profile).
- **Replication patterns:**
  - **`StreamingEnabled`** — distant lands replicate only when near; some objects pinned `Persistent` (coaster cars, family shrines) so they survive streaming.
  - Friends, buddies, shards, the world, leaderboards, cosmetics on buddies are **server-spawned** → everyone sees them.
  - **Remotes** (`Remotes.lua`, one `SquishyRemotes` folder of RemoteEvents). C→S: `RequestInitialState, EquipBuddyRequest, CollectSparkleBit, ClaimDailyCapsule, BuyCosmetic, EquipCosmetic, RedeemCode, VisitRoom, PlaceRoomItem, CollectStoryPage, SendGift, BuyPremium, BuyPass, ResetProgress, OwnerDebug`. S→C: `StateSync` (full snapshot), `SocialSync` (surge meter + event), `SquishResult, CapsuleResult, SparkleBitCollected, SparkleRestored, StoryPageCollected, OpenBoutique, OpenRoomCatalog, OpenGiftUI, GiftReceived, Toast`.
  - **`SquishResult` is FireAllClients** — every client plays the squish/pop FX for visible friends.
- **The one client-authoritative exception:** **bounce launches** (the client owns its character's velocity/state, §3.6); server only does the juice + party window via attributes. **Squish input** is also a client `ClickDetector` event but it *fires on and is adjudicated by the server* — the client cannot self-award a pop.
- **Server tick:** ride/coaster/seesaw/buddy motion runs on `RunService.Heartbeat` server-side.

---

## 8. Code architecture map

### 8.1 Layout
```
src/ReplicatedStorage/Shared/   -- configs + data (required by both sides)
src/ServerScriptService/Server/ -- authoritative services (one ModuleScript each)
src/StarterPlayer/StarterPlayerScripts/ -- client UI + local physics/FX
```

### 8.2 Boot order (`Main.server.lua` — the orchestrator)
`Remotes.setupServer()` → require all services → `init()` each → `WorldService.build()` → `QuestService/Coaster/Playground/Family init` → `SquishService.init(zoneGroups)` → wire **hooks** (services expose `onX` callbacks Main connects, so services stay decoupled: e.g. `SquishService.onHappyPop` fans out to tutorial/quest/daily/surge; `coinMultiplier` composes surge×boost) → connect world ProximityPrompts → on `RequestInitialState`, sync + welcome the player.

### 8.3 Server services (the important ones)
| File | Role |
|---|---|
| `Main.server.lua` | Entry/orchestrator + hook wiring + owner-debug |
| **`PlayerDataService.lua`** | **Profile schema, session-locked DataStore, leaderstats, snapshot/`StateSync`** — the spine |
| **`SquishService.lua`** | **The core squish→Joy→Pop loop**, friend spawning |
| `SquishyModelFactory.lua` | 17 part-built squishy body archetypes + per-friend skins |
| `WorldService.lua` | Builds all 3 bespoke lands (geometry, pads, props, paths, travel hubs) |
| `QuestService.lua` | The Lost Sparkle: per-land shard (clue→wake N→appear→recover→unlock next) |
| `CapsuleService.lua` | Free capsule: weighted rarity, discover, duplicate→variant |
| `CollectionService.lua` | Equip buddy (validated toggle, 1–2 slots) |
| `BuddyService.lua` | Spawns equipped friend(s) as floating companion(s) + cosmetics + auras |
| `TutorialService.lua` | "Wake 3 sleepy friends" → 100 coins + first free capsule |
| `TravelService.lua` | Travel Pads between lands, gated by shard progress |
| `FinaleService.lua` | All 3 shards → Restore the Sparkle (one-time +1000, brightens world orb) |
| `FamilyService.lua` | 3 daughter guardians, earned by restoring each shard |
| `DailyService.lua` | Free daily capsule, rotating daily quests, login streak |
| `WeeklyService.lua` | Friend of the Week (fixed-price visitor) |
| `CodeService.lua` | Magic Words promo codes (server-side table) |
| `GiftService.lua` | Gifting v1 (coins or share-a-friend; no trading) |
| `MonetizationService.lua` | Passes + Dev Products, idempotent ProcessReceipt |
| `BoutiqueService.lua` | Coin cosmetics stall (+ premium shelves) |
| `SurgeService.lua` | Server-wide Sparkle Surge meter (×2 coins) |
| `GroupEventService.lua` | "Everybody Squish!" golden-friend co-op event |
| `LeaderboardService.lua` | OrderedDataStore boards on physical signs |
| `SparkleBitService.lua` / `StoryPageService.lua` | Hidden collectibles (range-validated) |
| `CoasterService.lua` / `PlaygroundService.lua` | Spline rides + bounce attractions |
| `RoomService.lua` / `FirstDayService.lua` | Personal room; first-session checklist |

### 8.4 Shared configs (the tunables — single source of truth)
`GameConfig` (squish/joy/tutorial/streak/finale numbers), `ZoneConfig` (the 3 lands + shard chain + Spread), `RarityConfig`/`CapsuleConfig`/`PackConfig` (gacha), `VariantConfig` (duplicates), `SquishyDefinitions`+`SquishyData` (roster), `CosmeticsConfig`/`MonetizationConfig` (shop/products), `SocialConfig` (surge/event/boards), `SoundConfig` (music+SFX ids), `SparkleBitConfig`/`StoryPageConfig` (collectible spots), `Remotes` (the contract), `DailyQuestConfig`/`WeeklyConfig`/`GiftConfig`/`RoomConfig`/`CoasterConfig`/`UiTheme`.

### 8.5 Client modules
`ClientController.client.lua` (boot + the server→client message router), `HudUI`, `CollectionBookUI`, `CapsuleRevealUI`, `DailyUI`, `BoutiqueUI`, `GiftUI`, `CodesUI`, `StoryPagesUI`, `RoomUI`, `FinaleUI`, `SocialUI`, `ToastUI`, `UiTheme`; `SquishFx` (squish/pop animation), `SparkleBits` (collectible render+detect), `BouncePads` (client bounce physics), `SoundScape` (per-land music crossfade).

### 8.6 Patterns worth replicating (or noting) in UE5
**Replicate:**
- **Config-as-source-of-truth** — all tunables centralized; rebalancing never touches logic. Mirror with `UDataAsset`/`UDataTable` + a `UDeveloperSettings`.
- **Hook-based decoupling** — services expose `onX` callbacks the orchestrator wires (no service `require`s another's behavior). Mirror with delegates / a lightweight event bus / GameplayMessageSubsystem.
- **Server validates everything; client requests + renders.** Especially: validate *before* charging; idempotent receipts; range-check client-reported pickups.
- **"Discover before charge"** ordering in the capsule (never take coins then fail to give a friend).
- **Daily-index (UTC) resets** for all daily/weekly state.
- **Generated roster data** from JSON.
**Note / improve for UE5:**
- Roblox's scripted-kinematic rides + Seat welds are a workaround for not having a great physics-animation story; UE5's spline components + sequencer/timelines (or Chaos for genuine squishy bodies) can do this more naturally.
- `ClickDetector`-only "attack" is thin; UE5 can make squishing far more tactile (a real boop with soft-body deformation) without changing the loop.
- No automated tests; balancing was playtest-driven (the 3 daughters). Bring real telemetry on Steam.

---

## 9. MVP feature list for the Unreal port (PRIORITIZED)

> Goal of MVP: be **recognizably Squishy Smash** — the cozy squish-collect loop in at least one beautiful land, with the squishy feel and the Book — then layer depth. Ordered tiers.

### TIER 0 — Must-have to be recognizably Squishy Smash (the irreducible core)
1. **Stock-feel third-person character** with gentle WASD/gamepad movement + jump; **no fall damage / no ragdoll / no death** (safety-first feel). (Enhanced Input; CharacterMovementComponent.)
2. **Squishy friends on pads** in a cozy land (start with **Pudding Hills**), each a soft-body-feeling 3D blob with a cute face, breathing bob, and "zZz" when sleepy.
3. **The squish loop:** click/tap a friend within range → +Joy (≈3 squishes, `JoyPerSquish≈0.34`) → **Happy Pop** (squash-spring + swell-fade-into-sparkles, signature sound, `+coins` popup) → it despawns and a new sleepy friend wakes after ~1.2s. **Server-authoritative Joy + coins.**
4. **Sparkle Coins** currency (earned from pops) with a HUD coin pill.
5. **The Sparkle Capsule** (free, kind gacha): walk up → open → weighted-rarity **Discover a friend** → full-screen **reveal** (wobble→flip→card); **duplicate → ✨Sparkly/🌈Rainbow + bonus coins**. **Never gambling-flavored language.**
6. **The Squishy Book** (collection album): grid of all launch cards, locked = silhouette, unlocked = card art; tabs by land; **Equip Buddy** from a card.
7. **Equipped buddy** — your favorite friend floats and follows you (cosmetic), visible to others.
8. **Tutorial** ("wake 3 sleepy friends" → 100 coins + first free capsule) and a **Guide NPC**.
9. **Persistent save** (coins, discovered set, variants, equipped buddy, tutorial/quest flags). Server-authoritative profile. (On Steam this is your own backend or Steam-cloud-backed save, see §10.)
10. **Kid-safe content rules enforced** — soft vocabulary everywhere, no hazards, no scary visuals (apply `docs/02_KID_FRIENDLY_RULES.md` verbatim).

### TIER 1 — The full identity (what makes it *the* game, not a tech demo)
11. **The Lost Sparkle quest** across **3 lands** (Pudding Hills → Goo Coast → Moonlit Hollow): wake N friends (8/10/12) → **shard appears at a landmark** → hold-to-recover → **next land unlocks** → all 3 → **Restore the Sparkle finale** (+1000, world brightens). Travel Pads gated by shard progress.
12. **Three distinct, explorable lands** with wayfinding, a starter cluster + friends hidden behind landmarks ("I found one my sister missed"). Each with its own capsule, guide, pack of friends.
13. **Hidden Sparkle Bits** (explore collectibles, +25 each, all-found bonus, daily refresh) — client-detect/server-validate.
14. **Daily loop:** free daily capsule, 3 rotating daily quests, gentle login streak.
15. **Squishy playground feel** — at least the **Bounce Bog trampoline** + one ride (the **Sparkle Express coaster** is the signature) and gentle bounce pads. This is a big part of "squishy" tactility.
16. **Full 48-friend roster** with real 3D bodies + card art + signature Pmf/Sploink/Thup sounds + per-land music.
17. **Family friends** (3, earned by restoring shards) — the personal/heart beat.

### TIER 2 — Shared-world & social (better-together)
18. **Sparkle Surge** server-wide ×2-coins meter + **"Everybody Squish!"** golden co-op event.
19. **Leaderboards** (Friend Finders / Joy Champions).
20. **Show-off buddies** (owner tag + variant aura) + server-wide discovery/shard shout-outs.
21. **Friend of the Week** fixed-price visitor; **Magic Words** promo codes; **Gifting** (coins / share-a-friend, no trading); **Story Pages** (the book-in-game).

### TIER 3 — Monetization & long-tail (Steam-appropriate, post-fun)
22. **Coin cosmetics Boutique** (hats/trails/balloons — the first coin sink).
23. **Premium cosmetics + convenience** (Steam equivalents of the passes — Extra Buddy Slot, Coin Boost, VIP aura — and the 6 premium cosmetics). **Keep capsules free; sell style & convenience, never power; no paid randomness.**
24. **Squishy Room** decoration; **My First Day** checklist.

### Explicitly OUT (don't add — they'd break the brand)
Combat / PvP / damage, weapons, hazards, paid randomness / coin packs, **trading**, horror/scary, romance — all deliberately excluded.

---

## 10. Unreal Engine 5 translation notes (per major system)

| Roblox concept | UE5 mapping |
|---|---|
| **Stock humanoid + WASD + Invisicam, no ragdoll** | `ACharacter` + `UCharacterMovementComponent` (walking only, modest speed/jump), **Enhanced Input** mapping context. Disable death/ragdoll; clamp fall damage to 0; gentle landing. A simple spring-arm third-person camera with occlusion fade. |
| **Squishy friend (`SquishService` + `SquishyModelFactory`)** | An `AActor`/`APawn` "SquishyFriend" with a `UJoyComponent` (replicated 0..1 float, server-owned). Body: skeletal mesh with a squash/stretch anim or a **Chaos soft-body / cloth / mesh-deformer** for genuine squish. Data-driven from a `USquishyFriendData` `UPrimaryDataAsset` (the `SquishyDefinitions` rows). Spawner = a `UPadSpawnerComponent` / spawn-volume. |
| **Squish input (`ClickDetector.MouseClick`, server-adjudicated)** | Click/tap or **interact-on-cursor** within range → client RPC `ServerSquish(FriendActor)`; **server** runs the cooldown (0.12s) + Joy add + pop, then **multicast** the FX. Optionally upgrade to a tactile melee "boop" that still routes through the server for the Joy award. Consider `GameplayAbilitySystem` only if it pays for itself (likely overkill here — a plain interaction component is cleaner). |
| **Joy / Happy Pop / coins** | `JoyComponent` server-authoritative; on reaching 1.0 → server applies coins (`CoinReward × surge × boost`), multicasts a `OnHappyPop` event; despawn timer + respawn timer as server timers. No HP/damage types. |
| **Squish/Pop FX (`SquishFx`)** | Niagara sparkle burst + a squash-spring anim (timeline/`UCurveFloat` cosine spring) + Metasound for signature/pop SFX + a floating `+coins` widget. Breathing bob = a looping local anim/timeline. |
| **Bounce pads (client-owned velocity)** | A `UBouncePadComponent` with replicated `BounceVelocity`/party-window properties; on overlap, **client** applies `LaunchCharacter(velocity, true, true)` for responsiveness (or just do it server-side with `bForceClient` — UE replication is friendlier than Roblox here, so you may not need the client-ownership hack). Squash + Niagara juice multicast. |
| **Coaster / rides (arc-length splines + Seat welds)** | `USplineComponent` paths; ride cars = actors moved along the spline by distance each tick (`GetLocationAtDistanceAlongSpline`), banking from spline tangent/roll, clamped. Riders **attached** to a seat socket (disable their movement while seated). Sequencer or a movement component drives it. Replicate car distance (or run deterministically + replicate seat occupancy). Keep the **safe-exit** rule. |
| **Three lands (`WorldService`, 600 studs apart, `StreamingEnabled`)** | **World Partition** + level streaming (or sub-levels) per land; data-driven landmark/pad placement. Travel Pads = trigger volumes that teleport + gate on shard progress. Author the lands as real UE levels (much nicer than procedural part-building) but keep the **layout intent** (starter cluster + hidden pockets + wayfinding). |
| **Quest (`QuestService`, per-land shard chain)** | A `UQuestSubsystem` tracking `Shards[land]={progress,collected}` on the player save; wake-count increments on Happy Pop in that land; shard actor spawns at goal; hold-interact recovers; unlock gate flips. Drive UI via a quest-state delegate. |
| **Capsule gacha (`CapsuleService`, free)** | Server-side weighted RNG (`FRandomStream`), **validate-before-charge**, duplicate→variant. A reveal `UUserWidget` sequence. Keep it **free** and never use gambling visuals/words. |
| **Collection Book / cards** | A `UCollectionSubsystem` over the discovered set; a tabbed `UUserWidget` grid (UMG) with locked silhouettes; Equip-Buddy button → `ServerEquipBuddy(id)`. |
| **Buddy follower (`BuddyService`)** | A cosmetic follower pawn/actor spawned **on the server** (so it replicates to all), AI/spline follow with a bob; cosmetics attached to sockets (`HatOffset` → a head socket). |
| **Persistence (`PlayerDataService`, DataStore + session lock)** | On Steam there's no DataStore — use **your own backend** (e.g. a small REST/Postgres or PlayFab/Nakama) **or** `USaveGame` + Steam Cloud for solo. Keep **server-authoritative writes**, idempotent purchase grants, and a **session-lock equivalent** if you run dedicated servers. Preserve the exact field set (§6.1). |
| **Monetization (passes + Dev Products)** | **Steam DLC / microtransactions / Steam Inventory Service** for cosmetics + convenience. Mirror the **idempotent receipt → grant-then-persist** pattern. **Keep capsules free; no paid randomness; sell style/convenience only** (this is both brand-safe and avoids loot-box regulation). |
| **Shared world / social (Surge, Everybody Squish, leaderboards, gifting)** | Dedicated UE servers (or a relay) for the shared hub; replicate the surge meter + event via a `UGameStateComponent` / replicated subsystem. Leaderboards via Steam Leaderboards or your backend. Gifting = same-session server-validated; **no trading**. |
| **Config-as-source-of-truth + hook decoupling** | `UDataAsset`/`UDataTable` for all tunables; `UDeveloperSettings` for globals; **delegates / GameplayMessageSubsystem** for the `onX` hook pattern so systems stay decoupled. |
| **Kid-safe vocabulary & no-fail design** | Bake the terminology table (`docs/02`) into all UI text and the style guide; enforce no-hazard, no-death, no-scary as a content rule. |

### Net guidance for the port
1. **Build Tier 0 in one gorgeous land first** — prove the squishy feel (soft-body squish + pop + sparkle + sound) and the capsule→Book loop. That single vertical slice *is* the game's soul.
2. **Lean into UE5's strengths the Roblox version couldn't have:** genuine soft-body/deformable squishing (Chaos / mesh deformers), real lighting (Lumen) for the storybook mood, hand-authored levels, and a more tactile "boop." None of this changes the loop; it elevates it.
3. **Preserve the ethics & feel verbatim:** server-authoritative, no-fail, no-ragdoll, free capsules, no trading, soft language, "sell style/convenience never power." These aren't incidental — they're the product's identity and its regulatory/brand safety.
4. **Keep the data model and tunables 1:1** where possible (roster, rarities, coin rewards, Joy/streak/finale numbers) so balance carries over and the two products stay canon-consistent with the storybook + iOS app.

---

### Key source files to revisit (all under `C:\Users\chris\Roblox-squishy`)
- **Core loop:** `src/ServerScriptService/Server/SquishService.lua`; tunables `src/ReplicatedStorage/Shared/GameConfig.lua`.
- **Squishy bodies & FX:** `src/ServerScriptService/Server/SquishyModelFactory.lua`; `src/StarterPlayer/StarterPlayerScripts/SquishFx.lua`.
- **Persistence (schema):** `src/ServerScriptService/Server/PlayerDataService.lua`.
- **Quest spine:** `src/ServerScriptService/Server/QuestService.lua`; `src/ReplicatedStorage/Shared/ZoneConfig.lua`.
- **Gacha:** `src/ServerScriptService/Server/CapsuleService.lua`; `CapsuleConfig.lua`; `RarityConfig.lua`; `VariantConfig.lua`.
- **Roster (source of truth):** `src/ReplicatedStorage/Shared/SquishyDefinitions.lua` (+ `SquishyData.lua`); roster table in `docs/04_CHARACTER_DATA_AND_ROSTER.md`.
- **Physics/rides:** `src/ServerScriptService/Server/CoasterService.lua`, `PlaygroundService.lua`; client `src/StarterPlayer/StarterPlayerScripts/BouncePads.lua`.
- **Orchestration & contract:** `src/ServerScriptService/Server/Main.server.lua`; `src/ReplicatedStorage/Shared/Remotes.lua`.
- **Monetization:** `src/ReplicatedStorage/Shared/MonetizationConfig.lua`, `CosmeticsConfig.lua`; `src/ServerScriptService/Server/MonetizationService.lua`, `BoutiqueService.lua`.
- **Design intent:** `docs/11_GAMEPLAY_V2_DESIGN.md`, `docs/01_UNIVERSE_CANON.md`, `docs/02_KID_FRIENDLY_RULES.md`, `docs/03_ROBLOX_GAME_DESIGN_MVP.md`, `docs/05_COLLECTION_AND_CAPSULE_SYSTEM.md`; narrative `docs/book2_the_lost_sparkle_manuscript_draft.md`; and the repo `CLAUDE.md` (the self-authored architecture map).

### Ambiguities / gaps to flag
- **Exact player cap / dedicated-server topology** is not pinned in code (Roblox default; "scales 1→N"). Decide deliberately for Steam.
- **Meshes & card-art assets** referenced by id live in the published Roblox place / `ServerStorage`, not in `src/` (the art agent covers them). The mesh pipeline is `tools/mesh_pipeline/`; card upload pipeline is `tools/card_art/`.
- The **iOS companion app** is referenced as sharing canon but is out of scope here.
- `legendary` rarity exists in `RarityConfig` (weight 6) but the **capsule** weight table omits it and no launch friend is tagged `legendary` (commons/rares/epics/mythics only) — minor inconsistency to clean up in the port.
