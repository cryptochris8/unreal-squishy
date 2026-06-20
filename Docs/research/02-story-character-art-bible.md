# Squishy Smash: The Lost Sparkle — Story, Character & Art Bible

**For:** Unreal Engine 5 game *Squishy Smash: The Lost Sparkle* (Steam)
**Authored:** 2026-06-19 (research compiled from existing Squishy Smash IP)
**Status:** Research deliverable. Distinguishes clearly between **[FOUND]** (sourced from existing materials) and **[INFERRED / PROPOSED]** (new synthesis for the UE5 title).

---

## 0. Sources & where the IP actually lives

The Squishy Smash IP is spread across **three sibling project folders** under `C:\Users\chris\`. The Unreal project (`Unreal-squishy`) was **empty** at research time — it inherits everything below.

| Folder | What it is | Value for UE5 |
|---|---|---|
| `C:\Users\chris\Squishy-smash\` | **PRIMARY IP / canon source.** Flutter iOS app + the published storybook line (KDP). Holds the locked Story Bible, the full 48-card art, branding, and Book 2 (*The Lost Sparkle*) manuscript + rendered spreads. | Canonical lore, character art, painterly storybook art, branding |
| `C:\Users\chris\Roblox-squishy\` | Roblox port of the same world. Cleanest **distilled** design docs + reusable game-side assets (SFX, ambience, marketing icons, sample cards). | Game-design distillation, audio, marketing icons |
| `C:\Users\chris\Unreal-squishy\` | **This project (destination).** Was empty except for video files at root. | — |

### The single most important narrative discovery

> **"The Lost Sparkle" IS fully documented.** It is **Book #2** of the Squishy Smash storybook line — a 40-page, 8.5×8.5 KDP picture book by Christopher Ryan Campbell. The complete 18-spread, ~928-word manuscript exists, the canonical Story Bible behind it exists, and all 18 painted spreads have been rendered. The UE5 game's subtitle is drawn directly from this book.

**Canonical "Lost Sparkle" source files (all FOUND):**
- `C:\Users\chris\Roblox-squishy\docs\book2_the_lost_sparkle_manuscript_draft.md` — full 18-spread manuscript with per-spread visual direction (also in `Squishy-smash\...\book\manuscript\book2_manuscript_draft.md`)
- `C:\Users\chris\Squishy-smash\squishy_smash\book\STORY_BIBLE.md` — **the load-bearing canonical mythology** (locked 2026-05-13)
- `C:\Users\chris\Squishy-smash\squishy_smash\book\BOOK2_CONCEPT_DRAFT.md` — locked story spine + format
- `C:\Users\chris\Squishy-smash\squishy_smash\book\STORYBOOK_DISCOVERY.md` — Book 2 production/positioning spike
- `C:\Users\chris\Squishy-smash\squishy_smash\book\cover\cover_copy_book2.md` — official cover + back-cover blurb + palette tokens
- 18 rendered spreads: `C:\Users\chris\Squishy-smash\book2_final_spreads\spread_01.png` … `spread_18.png` (dup at `...\squishy_smash\book\book2_final_spreads\`)

**Note on the four `.mp4` files** in `Roblox-squishy` root (`Moonlit-hollow.mp4`, `Moonlit-hollow_portrait.mp4`, `pudding-hills.mp4`, `Squishy video.mp4`, `Squishy_Smash_store_video.mp4`) and the many `squishy_book2_*.mp4` teaser/read-along videos in `C:\Users\chris\`: these are **environment fly-throughs / trailers / audiobook read-alongs** named after the three world-regions. Per instructions they were **not opened**, but their names confirm the three canonical locations and that animated motion references exist for each region. Recommend the user point these out if motion reference is wanted for UE5 environment animation.

---

## 1. World & Setting

**[FOUND — Story Bible §1–4, Universe Canon]**

### The one-sentence world law (canonical, verbatim)
> **"A squishy is a wish that found a shape. The Squishkeeper sees them, and writes it down, so they stay."**

That sentence is the entire physics of the world. Everything else is a consequence.

### What the world is
Squishy Smash is a **soft, wholesome, kawaii feelings-world** for ages 4–8 (and the cozy/ASMR-adult adjacent audience). It is **not** a combat world. There is **no villain — ever** (this is a hard canon rule; tension always comes from *situation*, never opposition). The promise of the universe is *safety*. Squishies are made of **"wish-stuff"** (canonical term, always lowercase + hyphenated) — soft pockets of feeling that the world grew bodies around so children could hold them. That is why they wobble.

### The cosmology
- **Squishies** = wishes that found a shape. Comfort, surprise, courage, hush — made bodies.
- **The Squishkeeper** = the first soft thing that ever noticed another soft thing. **A witness, not a god.** "Round, probably." Being *seen* is what lets a squishy stay. **CRITICAL VISUAL RULE: the Squishkeeper is NEVER drawn** (no silhouette, no implied figure — the mystery is franchise fuel). Exists only as narrator voice. Acceptable stand-in props only: a quill, a half-open journal, a faint pawprint, a single round shadow.
- **The pop** = "a pop is a hello, not a goodbye." When squeezed, a squishy's packed feeling bursts out as color + sound + a shockwave of joy, then it reforms in **"the great soft elsewhere"** and comes back. (This is officially the **Kirby** model — pop = teleport, not death.)
- **The Sparkle** = the shared light of being-seen, fueled by squishies popping and the Squishkeeper recording. **When pops stop returning, the Sparkle dims.** This one mechanic powers the whole story line.

### The Three Pack-Worlds (the three locations — all FOUND)
Each region formed around one **"first feeling."** This is the spine of both story and level design.

| World / Location | First Feeling | Palette & look | Signature sound | Main guide | Founding "Spilling" |
|---|---|---|---|---|---|
| **Pudding Hills** | Comfort (warm + sweet) | Peach / cream / warm gold. Whipped-cream hills, syrup rivers, sprinkle-snow, steamer baskets, orchard | **"Pmf"** | Soft Dumpling | "spilled warm — syrup pooled, sprinkles fell as snow" |
| **Goo Coast** | Surprise (glossy + bouncy) | Mint / aqua / jelly-blue. Glossy shore, bubble-tide, a sea you bounce on | **"Sploink"** | Goo Ball | "spilled glossy — the tide learned to *sploink*" |
| **Moonlit Hollow** | Brave-Cuddle (a child meeting the dark and finding it soft) | Lavender / purple / silver, **soft dark, never scary**. Silver/glowing mushrooms, a moon "turned down to nightlight-soft," deep groves | **"Thup"** | Blushy Bun Bunny | "spilled quiet — under a moon turned to nightlight-soft" |

Above all three hangs **The Sparkle** (a warm-gold four-point star). The world is officially called **"Pack-Land"** in the manuscript. The three regions had **never visited each other** before the events of *The Lost Sparkle* — crossing the borders is the literal and emotional engine of the story.

> Visual proof: `spread_01.png` paints all three regions in one frame — cream/peach swirl-hills (L), a mint goo river (center), lavender glowing mushrooms + crescent moon (R), gold Sparkle-star above. This is the single best world-establishing reference for UE5.

### Tone & themes
- **Tonal lane (canonical):** *"Adventurous (Pokémon-mythic) with a soft landing."* Open with scope/wonder/journey energy; close intimate, cozy, repeatable.
- **Themes:** being seen; closeness-as-courage; small/everyday feelings carrying the world (a Common saves the day — "mythologically load-bearing"); friendship across difference; the dark made gentle.
- **Rarity is lore, not grind:** rarity = *how much wish-stuff* a squishy is made of. Commons are "closer to the child"; the Legendary Cores are "farther away precisely because they are larger." (Story Bible §5.)

### Canon guardrails to respect in UE5 (do not break)
1. No villain / no antagonist. Tension = situation only.
2. The Squishkeeper is never depicted.
3. "Pop = hello / they always come back" — never frame popping as defeat or death.
4. Soft dark in Moonlit Hollow — never scary/horror.
5. Deliberate incompleteness — don't over-explain the cosmology (Pokémon waited 8 years to name a creator god; Ghibli never explains Totoro — same model).

---

## 2. The Narrative — "The Lost Sparkle" plot

**[FOUND — full manuscript, locked spine]**

### Logline (from the official back-cover blurb)
> *"A sparkle goes missing. Three new friends go looking."*
>
> *"When the last Sparkle of the Squishy World flickers and splits into three, Soft Dumpling sets out into the warm dark to find the missing light. Along the way she meets a curious Goo Ball and a Blushy Bun Bunny — and discovers that some sparkles are only found when three first feelings remember each other."*

### What the Lost Sparkle is, the conflict, and the goal
- **The Lost Sparkle** = the world's shared light of being-seen. For the first time ever, it **flickered, wobbled, and split into three shards**, one drifting into each pack-world. As it fragments, each region goes "a little quieter" — the Sparkle is going out.
- **The conflict (no villain):** the *situation* — the cycle of pop-and-return is breaking, the Sparkle is dimming, and the three regions are isolated from each other. The threat is **absence and forgetting**, not an enemy.
- **The goal:** three Common squishies — one per region — must **cross the borders that no one has ever crossed**, gather the three shards, and reunite them. The resolution is that **friendship across packs makes the Sparkle whole again** (and brighter than before).

### The locked 5-beat spine
1. **Peace.** Pack-Land at peace; the Sparkle holds.
2. **The Flicker.** The Sparkle splits; three shards drift, one into each world.
3. **Three summoned.** Soft Dumpling (Pudding Hills), Goo Ball (Goo Coast), Blushy Bun Bunny (Moonlit Hollow) each leave home for the first time and **meet at the border.**
4. **Three-act middle.** One pack-world per act. The trio traverses Pudding Hills → Goo Coast → Moonlit Hollow. In each: a **Rare** speaks as guide, an **Epic** "gates" the obstacle (resolved by collective ask / bounce / squish — never by fighting), and **Commons** populate the crowd.
5. **Climax + resolution.** In Moonlit Hollow's deepest grove the third shard is nearly out. Blushy Bun Bunny shouts **"EVERYBODY SQUISH!"** — the trio squeeze together, three pops become one, the three shards fuse. The three **Cores** arrive and *bow to the Commons*. The Sparkle returns **brighter**. Everyone carries a little light home; the three regions begin, for the first time, to visit each other.

### The climax mechanic — the "Big Pop"
**"EVERYBODY SQUISH!"** (spreads 12–13). This is the franchise's shout-along moment — the child performs it every read. For UE5 this is the obvious **signature set-piece / button-mash or rhythm climax**.

### The cumulative chant (a key audio/gameplay hook — FOUND)
The three signature sounds build cumulatively, then unify at the climax — a deliberate read-aloud / TikTok-virality device:
- Spread 6 (Pudding Hills): **Pmf**
- Spread 8 (Goo Coast): **Pmf · Sploink**
- Spread 10 (Moonlit Hollow): **Pmf · Sploink · Thup**
- Spread 12 (climax): **PMF! SPLOINK! THUP!** → "the three sounds became one"

### Canonical lines (brand-safe, may be used verbatim in UE5)
- **The Pact:** *"Every pop is a hello. Every hello comes back."* (appears once, near the climax — never twice)
- **The Squishkeeper's job:** *"To see them, and write it down, so they stay."*
- **The Sparkle, defined:** *"The light that comes from being found."*
- **The close (Book 2):** *"And tomorrow, another wobble. They always come back."*

### Series context (for sequel hooks — FOUND, Story Bible §8)
*The Lost Sparkle* is Book 2 of a planned 6-book arc. Book 1 = *Meet the Squishies* (a character catalog, not a plot). Books 3–6 each spotlight a region/Core: *Pudding Hills* (Galaxy Dumpling), *Goo Coast* (Aurora Stretch Cube), *Moonlit Hollow* (Mythic Plush Familiar), *The Three Cores* (approaches but never fully answers "what is the Squishkeeper"). Useful if the UE5 game wants DLC / chapter structure.

---

## 3. Character Roster

**[FOUND — full 48-card roster + bios + card art + JSON stats]**

The complete cast is **48 characters across 3 packs** (16 each = 8 Common, 4 Rare, 3 Epic, 1 Legendary/Mythic Core), plus an 8-character **event pack** (Dumpling Squishy / "Candy Cloud Kitchen") and a few custom/family "Keepsake" cards. Every character has finished hero card art, a warm child-friendly bio, and a JSON stat block (deformability, elasticity, gooLevel, burst sounds, particle/decal presets, coin reward, rarity).

**Universal visual DNA (all characters):** rounded, blobby, toy-like "squishy" silhouettes; oversized glossy anime/kawaii eyes with bright catch-lights; tiny smiling mouths; rosy blush cheeks; soft rim-lighting; high-polish "premium plush/jelly" rendering; sparkle accents that scale up with rarity. The cards read as **2D renders of a consistent 3D-style character** — treat as official canon art (per `docs/08`).

Card-art source (final, 48): `C:\Users\chris\Squishy-smash\squishy_smash\assets\cards\final_48\NNN_Name.webp`
Bios source: `...\book\squishy_smash_featured_character_bio_sheet_for_claude.md`
Stats source: `Roblox-squishy\data\raw\*.json`

### 3A. The Three Protagonists (the heart of the UE5 game)

These three carry *The Lost Sparkle*. Canonical hero reference: `...\assets\website_hero\trio_book2_protagonists.png` and climax `spread_12.png`. In the soft storybook style they are simplified (see Art Direction §4) vs. the glossier card art.

**Soft Dumpling** — `001/048`, Squishy Foods, Pudding Hills, Common, "Pmf", coins 8
- **Appearance:** a cream/pale-peach steamed soup-dumpling (bao/xiaolongbao shape) with a little pleated/swirled topknot at the crown, two tiny stub feet, big dark eyes, rosy cheeks. Card shows her in a bamboo steamer with rising puffs.
- **Personality:** calm, comforting, cozy, gentle, "puffy and playful." The quiet one who surprises herself with bravery (she's the thumbnail-anchor / lead protagonist).
- **Role:** lead protagonist; "comfort." Sets out first; the audience-POV character. Burst type "Puff Pop," trait "Soft Bounce."
- **Signature:** leaves a soft puff of joy; says **"Pmf."**

**Goo Ball** — `017/048`, Goo & Fidgets, Goo Coast, Common, "Sploink", coins 14
- **Appearance:** a translucent glossy blue slime sphere — round, jiggly, see-through with bright highlights and tiny nub feet; big eyes, small smile, blush. (In storybook art: a glassy aqua-blue orb.)
- **Personality:** surprising, silly, stretchy, easygoing — "never bounces the same way twice."
- **Role:** second protagonist; "surprise." Leads the Goo Coast act (the bounce challenge).
- **Signature:** every wobble is a new surprise; says **"Sploink."**

**Blushy Bun Bunny** — `033/048`, Creepy-Cute Creatures, Moonlit Hollow, Common, "Thup", coins 12
- **Appearance:** a small round white/cream plush bunny with long floppy ears, extra-rosy cheeks, tiny paws, big warm eyes. (Storybook art: fluffy lop-eared white bun.)
- **Personality:** sweet, warm, gentle, "leader-by-warmth." The emotional core — the only protagonist who speaks dialogue early; the one who calls the climactic **"EVERYBODY SQUISH!"**
- **Role:** third protagonist; "brave-cuddle." Leads the Moonlit Hollow act; triggers the climax.
- **Signature:** rosy cheeks, hops; says **"Thup."**

### 3B. The Three Cores (Legendary / Mythic — the "frame" / guardians)

**Not enemies. Ancient, protective, Sparkle-level figures.** In Book 2 they appear only at the climax and **bow to the Commons.** Each is "original wish-stuff" — they *bloom* rather than pop.

**Celestial Dumpling Core** — `016/048`, Squishy Foods, Pudding Hills, Mythic, coins 120
- The "warm drop." A radiant golden-celestial dumpling that "taught the stars to glow by example." Mythic & radiant, powerful & kind. (Also the iOS app's menu mascot.) Story role: the sky-dumpling-sized Epic/Core that gates Pudding Hills appears as "a dumpling the size of a sky, full of small quiet stars."

**Singularity Goo Core** — `032/048`, Goo & Fidgets, Goo Coast, Mythic, coins 130
- The "glossy drop." A deep purple cosmic goo with a literal spiral galaxy / black-hole swirl in its body — "so dense with feeling that gravity bends to look at it." Card type "Mythic Goo," burst "Void Burst," trait "Singular Pull." Strange & dazzling.

**Mythic Plush Familiar** — `048/048`, Creepy-Cute Creatures, Moonlit Hollow, Mythic, coins 140
- The "quiet drop." A cream-white plush fox/cat familiar with a golden halo, a star above its head, and a jeweled purple cloak/cape. "Walks the edge of every Hollow making sure no squishy gets lost in its own shadow." Protective, kind, the guardian of lost squishies.

### 3C. Story-flagged Rares & Epics (guides + showpieces)

These nine are specifically flagged as **story characters** — Rares act as **guides** (described-by-feeling, e.g. "a small mochi that waved one shimmery wave"); Epics act as **obstacle-gates** (resolved peacefully).

| Name | Card | Pack / World | Rarity | Story role | Look |
|---|---|---|---|---|---|
| **Sparkle Mochi** | 011 | Squishy Foods / Pudding Hills | Rare | Pudding Hills guide | Shimmery soft mochi, glitter trail |
| **Galaxy Dumpling** | 013 | Squishy Foods / Pudding Hills | Epic | Pudding Hills gate ("sky-dumpling") | Dumpling full of stars/nebula, cosmic-purple, "Starburst Pop," "Nebula Bounce"; future hero of Book 3 |
| **Glitter Goo Ball** | 025 | Goo & Fidgets / Goo Coast | Rare | Goo Coast guide ("opal goo") | Goo with sparkling flecks |
| **Aurora Stretch Cube** | 030 | Goo & Fidgets / Goo Coast | Epic | Goo Coast gate ("cube the color of dawn") | Sky-ribbon iridescent stretchy cube; future hero of Book 4 |
| **Star-Eyed Bunny** | 041 | Creepy-Cute / Moonlit Hollow | Rare | Moonlit Hollow guide ("bunny whose eyes were stars") | White bun with literal star-shaped pupils, on a glowing wish-trail under a moon |
| **Glow Ghost Puff** | 043 | Creepy-Cute / Moonlit Hollow | Rare | Moonlit Hollow guide / grove presence | Soft glowing ghost cloud |

(Epic Moonlit gate in the deepest grove is **Glow Ghost Puff** scaled large — "a feeling shaped like a friendly haunting.")

### 3D. Full 48-card roster (reference table — FOUND)

Source of truth: `Roblox-squishy\docs\04_CHARACTER_DATA_AND_ROSTER.md` + `data\raw\*.json`. Fields per character: rarity, coin reward, unlock tier, card number, deformability/elasticity/gooLevel, impact + burst sounds, particle/decal presets.

**Squishy Foods → Pudding Hills (Comfort, "Pmf"):**
001 Soft Dumpling (C) · 002 Jelly Bun (C) · 003 Peach Mochi (C) · 004 Syrup Cube (C) · 005 Cream Puff (C) · 006 Rice Ball Squish (C) · 007 Marshmallow Puff (C) · 008 Pudding Pop (C) · 009 Strawberry Dumpling (R) · 010 Rainbow Jelly Bun (R) · 011 Sparkle Mochi (R) · 012 Golden Syrup Cube (R) · 013 Galaxy Dumpling (E) · 014 Crystal Mochi (E) · 015 Neon Dessert Blob (E) · 016 Celestial Dumpling Core (M)

**Goo & Fidgets → Goo Coast (Surprise, "Sploink"):**
017 Goo Ball (C) · 018 Bubble Blob (C) · 019 Stretch Cube (C) · 020 Soft Stress Orb (C) · 021 Jelly Pad (C) · 022 Sticky Pop Ball (C) · 023 Wobble Drop (C) · 024 Squish Capsule (C) · 025 Glitter Goo Ball (R) · 026 Shockwave Blob (R) · 027 Frost Gel Cube (R) · 028 Prism Stress Orb (R) · 029 Plasma Goo Ball (E) · 030 Aurora Stretch Cube (E) · 031 Cosmic Jelly Pad (E) · 032 Singularity Goo Core (M)

**Creepy-Cute Creatures → Moonlit Hollow (Brave-Cuddle, "Thup"):**
033 Blushy Bun Bunny (C) · 034 Squish Bat (C) · 035 Puff Ghost (C) · 036 Wobble Kitty (C) · 037 Tiny Blob Monster (C) · 038 Soft Fang Critter (C) · 039 Sleepy Slime Pet (C) · 040 Round Eared Creature (C) · 041 Star-Eyed Bunny (R) · 042 Moon Bat Blob (R) · 043 Glow Ghost Puff (R) · 044 Candy Fang Creature (R) · 045 Dream Eater Squish (E) · 046 Arcane Wobble Kitty (E) · 047 Phantom Jelly Beast (E) · 048 Mythic Plush Familiar (M)

**Event pack — Dumpling Squishy / "Candy Cloud Kitchen" (optional, FOUND):** Boblet (C) · Dimpa (C) · Gobble Puff (R) · Gold Dumplio (M) · Moshi (C) · Puffkin (C) · Soupy Blob (R) · Steamy (C). Plus custom "Keepsake/Family" cards (Apple Addy, Eggy Ellie, Hot Dog Heidi).

> Every one of the 48 has a one-line warm bio in the character bio sheet — full personality copy is ready to lift directly for UE5 codex/collection entries.

---

## 4. Art Direction

**[FOUND for both styles; UE5 translation INFERRED]**

Squishy Smash deliberately runs **two official, co-existing visual languages** (the Squishmallows / Pokémon / Sanrio precedent — same characters, different lanes). The UE5 game should pick a primary and use the other as accent.

### Style A — "Card / Toy" style (the collectible, glossy lane)
*Reference: every `final_48` card; e.g. `001_Soft_Dumpling`, `032_Singularity_Goo_Core`, `041_Star_Eyed_Bunny`, `013_Galaxy_Dumpling`, `048_Mythic_Plush_Familiar`.*
- **Look:** high-polish, hyper-rendered 3D-style kawaii. Glossy pastel finish, rounded forms, soft subsurface glow, strong rim light, lots of sparkle/bloom. "Premium, not cheap." Toy-commercial render quality.
- **Eyes:** large, glassy, anime catch-lights (multiple highlights). Rosy blush. Tiny mouth.
- **Rarity escalates the render:** Common = soft blue/pink, simple bg; Rare = brighter blue/purple + atmospheric flecks; Epic = purple/cosmic glow, room-filling FX; Legendary/Mythic = gold frame + cosmic galaxy backdrops + halos.
- **Best fit for UE5:** the **in-world characters, capsule/gacha reveals, collection codex, UI, marketing/Steam capsule art.** This is the lane the existing 3D-looking renders already imply.

### Style B — "Storybook / Painterly" style (the narrative lane)
*Reference: `spread_01.png` (world establishing) and `spread_12.png` (climax); hero `trio_book2_protagonists.png`.*
- **Look:** soft **watercolor + gouache / chalk-pastel**, **no hard outlines**, forms built from shaded mass. Cited anchor: **Vashti Harrison's *Big*** (Caldecott) and **Christopher Denise's *Knight Owl*** (cinematic dusk-warm chiaroscuro). Klassen-meets-Christian-Robinson register: limited warm palette, simple geometric character forms, flat color blocks with restrained texture, soft north-window light.
- **Characters simplified:** in this lane Soft Dumpling/Goo Ball/Blushy Bun are reduced to clean, single-warm-dot eyes and minimal features (vs. the busy card eyes).
- **Best fit for UE5:** **cutscenes, story-mode backgrounds, loading screens, the storybook framing device, the bedtime/close beats, painted environment skies.** This is the look that says "the player is inside the book."

### Color palette (canonical tokens — FOUND)

**Brand / global**
| Token | Hex | Use |
|---|---|---|
| Deep starry-night base | `#120B17` | Book 1 / catalog base, deep-space backdrops |
| Brand pink (primary) | `#FF8FB8` | Logo, spine band, primary brand accent |
| Sun gold (secondary) | `#FFD36E` | Sparkle, coins, warm highlights |
| Jelly-blue (accent) | `#7FE7FF` | Goo accents, cool highlights |

**Per-region accents (from pack JSON `palette` + canon):**
- **Pudding Hills:** primary `#FF8FB8`, secondary `#FFD36E`, accent `#7FE7FF` — peach/cream/warm-gold overall
- **Goo Coast:** mint/aqua/jelly-blue
- **Moonlit Hollow:** lavender/purple/silver

**Storybook "dusk-warm" sky palette (Book 2 / Knight Owl lane — use for painterly cutscenes & skies):**
| Token | Hex | Use |
|---|---|---|
| `cover_sky_top` | `#1B2440` | Deep teal-indigo (top of sky) |
| `cover_sky_mid` | `#3D3155` | Warm indigo bridge |
| `cover_horizon` | `#E4A56C` | Warm dusk glow at horizon |
| `cover_lowlight` | `#F5C97A` | Brightest horizon / sparkle echo |
| `wordmark_cream` | `#F5E9D0` | Cream type / light values |
| `volume_gold` | `#E4C46C` | "The Lost Sparkle" gold accent |

**Chroma-curve idea (storybook craft, transferable to UE5 story mode):** one signature hue (the Sparkle — candy lavender/mint) drives the whole emotional curve. Full saturation at peace → progressively **flatter/desaturated** (not darker) as the Sparkle goes missing → floods back at maximum saturation at the "EVERYBODY SQUISH!" climax → softens to warm dusk for the close.

### Shapes / silhouette ("the squishy aesthetic")
- Rounded, blobby, **squash-and-stretch** primitives. No sharp edges. Big head/body, tiny stub limbs.
- Strong, simple, readable silhouettes (Steam-thumbnail test: must read tiny).
- Deformation is core identity — characters should visibly squish/jiggle. The JSON even carries per-character `deformability`, `elasticity`, `gooLevel`, `burstThreshold` values to drive this.

### Mood / lighting
- Soft, warm, glowy. Bloom and gentle rim light everywhere. Pastel, **bright but not messy, never dark/muddy.**
- Moonlit Hollow is the only "dark" zone and it must stay **cozy-dark, nightlight-soft, never horror.**
- The Sparkle is the key light motif — a warm-gold four-point star/bloom.

### Concrete UE5 art guidance **[INFERRED / PROPOSED]**
- **Direction:** **stylized, not realistic.** Lean into a cel/soft-stylized hybrid. Two viable pipelines:
  1. **Glossy plush/jelly PBR** for characters (Style A): high roughness variation, **subsurface scattering** (SSS) for the jelly/dumpling translucency, clearcoat for goo gloss, fresnel rim. Bake the catch-light "kawaii eyes" as emissive eye decals/textures for consistency with card art.
  2. **Painterly post / NPR** for story-mode and skies (Style B): paint skies as textured skydomes or use a watercolor post-process (Kuwahara/edge-softening + paper-grain overlay) so cutscene backdrops match the book spreads.
- **Materials/shading notes:** translucent gummy bodies (SSS + thin-film/iridescence on Goo/Aurora/Prism characters); soft cloth/plush shading on bunnies/familiars (fuzz/fur shells, sheen); gold/halo emissive for Mythics; particle bursts + splat decals on "pop" (the JSON's `particlePreset`/`decalPreset` map directly to Niagara systems, e.g. `pink_soup_burst`, `blue_jelly_burst`, `cream_puff_burst`, `purple_monster_burst`).
- **Lighting mood:** warm key + cool fill, heavy bloom, soft contact shadows, no harsh shadows. Per-region lighting LUTs: Pudding Hills = golden-hour warm; Goo Coast = bright mint daylight; Moonlit Hollow = lavender moonlit dusk (the `#1B2440 → #E4A56C → #F5C97A` gradient). The Sparkle as a dynamic warm point light that literally **dims** as the story darkens (gameplay-visible).
- **Fonts (FOUND):** **Fredoka** = display/game/catalog & UI (matches the puffy 3D logo). **EB Garamond / Bookmania** = storybook body/narrator. **Caveat Brush** = hand-letter accent (the "EVERYBODY SQUISH!" shout). Logo wordmark = puffy 3D pink/cream balloon lettering with the bunny mascot peeking over (`branding/logo/squishy_smash_logo_primary.png`).

---

## 5. Audio / Music Identity

**[FOUND — ElevenLabs prompt sheet, SFX libraries, ambience, app store copy]**

### North-star tone
> **"ASMR oddly-satisfying, not cartoony."** Reference: slime-poking, soap-cutting, kinetic-sand TikToks. **Wet, organic, close-mic. Not synthy. Not cute-cartoon-boing.** "The bursts should make someone want to smash again immediately."

### SFX taxonomy (real, mastered files already exist — see §6)
- **Squishes/impacts:** per-character, multi-variant, with anti-repetition (e.g. dumpling = doughy wet squish; jelly = wobbly slap w/ sticky tail; mochi = stretchy sticky pull; slime = squelchy suction-pop).
- **Bursts/pops:** "Happy Pop," "Finale," bigger+brighter for Rare/Mythic.
- **Signature squish vocab (brand audio):** **Pmf** (Pudding), **Sploink** (Goo), **Thup** (Moonlit) — discrete recorded SFX (`sig_pmf_*`, `sig_sploink_*`, `sig_thup_*`). These are the audio brand and the cumulative-chant payload.
- **Sparkle/collection:** `sparkle_bit`, `shard`, `capsule`, coin ding, story-page turn.
- **Creature vocals:** wordless, vowel-y, cute-not-scary giggles/squeaks only (no real words).
- **UI:** soft bubble taps, warm rising confirm chimes, gentle dismiss tones — snappy, never draggy.
- **Per-region ambience (exists):** `amb_pudding_1`, `amb_goo_1`, `amb_moonlit_1` — ready-made room tone for the three UE5 zones. Plus a "train" set (`sfx_trainchug`, `sfx_trainwhistle`) suggesting a Pudding Hills ride/transport element.

### Music
- **[FOUND, thin]:** The iOS app added **"cozy new music on the main menu"** (calm, warm, low-key). A `squishy_promo_music.mp3` and `squishy_book2_audiobook` narration tracks exist. A `Roblox-squishy\marketing\music\raw\` folder exists (was empty at research time).
- **[INFERRED / PROPOSED for UE5]:** cozy, gentle, lullaby-leaning score — soft piano / celeste / music-box / mallet (glockenspiel, marimba), light pads, no aggressive percussion. Adventurous-but-soft per the Story Bible voice: a touch of "Pokémon-mythic" wonder in the world themes, resolving to bedtime-cozy at the close. Per-region motifs (warm/major for Pudding Hills; playful/bouncy for Goo Coast; hushed/twinkly for Moonlit Hollow) that braid together at the "Everybody Squish!" climax — mirroring the cumulative chant.

### Marketing VO tone (FOUND, optional)
Playful/dry-funny/cute-chaos, modern toy-ad delivery, lines under 2s, no reverb. Sample lines: "Squish it. Smash it. Pop it." / "This should not be this satisfying." Parent-friendly reframe leads with the Pact line ("Every pop is a hello").

---

## 6. Asset Inventory (reusable / reference for UE5)

All paths absolute. Card art and audio are the highest-value port candidates.

### 6A. Character / hero art — **PORT THESE**
| Path | Type | Depicts |
|---|---|---|
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\cards\final_48\*.webp` | 48 card images | All 48 characters, finished hero art + frames (canonical) |
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\cards\custom_family\*.webp` | webp | Keepsake/family cards (Apple Addy, Eggy Ellie, Hot Dog Heidi) |
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\website_hero\hero_*.png` | ~24 PNGs | Background-free hero renders of higher-rarity characters (cutout-ready) |
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\website_hero\trio_book2_protagonists.png` | PNG | **Canonical 3-protagonist hero shot** (storybook style) |
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\website_hero\pack_*.png` | PNG | Per-pack group art |
| `C:\Users\chris\Roblox-squishy\assets\card_samples\*.webp` (+ `_png\`) | 8 cards | Sample subset (001, 002, 013, 032, 041, 043, 046, 048) |

### 6B. Storybook art (painterly cutscene / background reference) — **PORT / REFERENCE**
| Path | Type | Depicts |
|---|---|---|
| `C:\Users\chris\Squishy-smash\book2_final_spreads\spread_01.png … spread_18.png` | 18 PNGs | Full painted *Lost Sparkle* spreads (world, journey, climax, close) |
| `C:\Users\chris\Squishy-smash\squishy_smash\book\book2_final_spreads\*` | dup | Same 18 spreads |
| `C:\Users\chris\Squishy-smash\squishy_smash\book\spreads_rendered\`, `spread_poc\`, `regen_2026_05_pipeline\` | folders | Intermediate/POC spreads + generation prompts (style recipes) |

### 6C. Branding / identity — **PORT THESE**
| Path | Type | Depicts |
|---|---|---|
| `C:\Users\chris\Squishy-smash\squishy_smash\branding\logo\squishy_smash_logo_primary.png` | PNG | Primary 3D puffy wordmark + bunny mascot |
| `C:\Users\chris\Squishy-smash\squishy_smash\branding\icon\squishy_smash_icon_*.png` | PNG | App/game icons (bunny + pink variants, 512) |
| `C:\Users\chris\Squishy-smash\squishy_smash\branding\skybox\skyboxreveal1/2.jpg`, `skybox_reveal_portrait.jpg` | JPG | Sparkle-reveal skyboxes (usable as UE5 sky reference) |
| `C:\Users\chris\Squishy-smash\squishy_smash\branding\x_banner.png`, `reddit_ad_hero_1200.*` | img | Social banners / ad heroes |
| `C:\Users\chris\Squishy-smash\squishy_smash\book\assets\wordmark_series.png` | PNG | Locked **storybook** serif series wordmark (Books 2–5) |

### 6D. Audio — **PORT THESE (mastered)**
| Path | Type | Depicts |
|---|---|---|
| `C:\Users\chris\Roblox-squishy\marketing\sfx_master\sfx_*.mp3` | MP3 | Mastered: squish ×3, happy_pop ×3, capsule, finale, shard, sparkle_bit ×2, story_page |
| `C:\Users\chris\Roblox-squishy\marketing\sfx_master2\*.mp3` | MP3 | Mastered: ambience (pudding/goo/moonlit), boing, pop, splash, whoosh, train ×2, **sig_pmf/sploink/thup ×2 each** |
| `C:\Users\chris\Roblox-squishy\marketing\sfx_audition\0X_*.mp3` | MP3 | Audition set (squish, happy_pop, sparkle_bit, capsule, boing, shard) |
| `C:\Users\chris\Squishy-smash\squishy_smash\assets\audio\{food,goo,creature,ui}\` | MP3 | Per-character impacts/bursts + UI (some real, some stubs per ElevenLabs sheet) |
| `C:\Users\chris\squishy_book2_audiobook_v1.mp3`, `squishy_book2_chris/george.mp3`, `squishy_promo_*.mp3` | MP3 | *Lost Sparkle* audiobook narration + promo music/VO |

### 6E. Marketing / store reference
| Path | Type | Depicts |
|---|---|---|
| `C:\Users\chris\Roblox-squishy\marketing\product_icons\*.png` | PNG | Cosmetic/pass icons (aurora ribbon, comet trail, golden halo, unicorn horn, beret, balloon, VIP) — cosmetic design reference |
| `C:\Users\chris\Roblox-squishy\marketing\icon_*.png`, `thumb_*1920x1080.png` | PNG | Game icons + 1080 thumbnails (avatar path, blob buddy) |
| `C:\Users\chris\Squishy-smash\squishy_smash\docs\app_store_submission_copy.md` | md | Full store copy, keywords, tone, the Pact-line positioning |
| `C:\Users\chris\Squishy-smash\squishy_smash\book\cover\cover_copy_book2.md` | md | *Lost Sparkle* cover + back-cover blurb + dusk palette tokens |
| **Video (note only — not opened):** `Roblox-squishy\Moonlit-hollow.mp4`, `pudding-hills.mp4`, `Squishy_Smash_store_video.mp4`, `Squishy video.mp4`, `Moonlit-hollow_portrait.mp4`; `C:\Users\chris\squishy_book2_*.mp4` (teasers v1–v8, read-alongs, TikTok/YouTube shorts) | MP4 | Region fly-throughs, store trailer, Book 2 teasers & audiobook read-alongs — **motion/animation reference for the three zones** |

### 6F. Design docs worth lifting wholesale
- `C:\Users\chris\Squishy-smash\squishy_smash\book\STORY_BIBLE.md` — **the canon; treat as law.**
- `C:\Users\chris\Roblox-squishy\docs\01_UNIVERSE_CANON.md`, `04_CHARACTER_DATA_AND_ROSTER.md`, `06_UI_AND_CARD_STYLE_GUIDE.md` — clean distilled design.
- `...\book\squishy_smash_featured_character_bio_sheet_for_claude.md` — all 48 ready-to-use bios.
- `...\book\research_2026_05_25\00_SYNTHESIS_visual_plan.md` (+ 01–06) — the painterly art pipeline & craft brief.
- `...\book\manuscript\book2_manuscript_draft.md` + `book2_layout_brief.md` — full manuscript + per-spread layout.

---

## 7. Flags / open questions for the user

1. **Storybook location confirmed.** The "storybook" for Squishy Smash is **Book #2: *The Lost Sparkle*** (the UE5 subtitle's namesake). Full manuscript, canonical Story Bible, and all 18 painted spreads are FOUND — **not** inside `Unreal-squishy`, but in `C:\Users\chris\Squishy-smash\squishy_smash\book\` (canon) and mirrored into `C:\Users\chris\Roblox-squishy\docs\`. Recommend the user confirm `Squishy-smash` is the IP source of truth to port from.
2. **Book #1 (*Meet the Squishies*) manuscript** (`...\book\manuscript\01_manuscript.md` / `02_manuscript_v2.md`) was referenced but not deeply read here — it is the **voice canon** if the UE5 narrator/codex copy needs to match Book 1 exactly.
3. **Two art styles** — the UE5 game must decide primary (recommend **Style A glossy for gameplay/characters/UI**, **Style B painterly for story-mode/cutscenes/skies**). Both are official; mixing is on-brand.
4. **Music is the thinnest area** — only "cozy menu music" + audiobook tracks exist; SFX/ambience are rich. Original UE5 score is a net-new need (§5 proposes a direction).
5. **The Unreal project was empty** at research time (only loose `.mp4`s at the `Unreal-squishy` root, none yet inspected) — this bible + the asset inventory in §6 is the starting kit.

---

*Compiled 2026-06-19 from existing Squishy Smash IP across `Squishy-smash\` (canon) and `Roblox-squishy\` (distilled). [FOUND] = sourced & cited; [INFERRED/PROPOSED] = new synthesis for the UE5 title. The Story Bible (`STORY_BIBLE.md`) is the load-bearing canon — if anything here contradicts it, the bible wins.*
