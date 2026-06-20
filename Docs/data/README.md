# Docs/data — pre-staged content data (editor-independent)

Clean, import-ready extracts of the canonical Roblox data, prepared ahead of the editor so M2 content can drop in immediately once the MCP is live.

## PuddingHills_Friends.csv
The full **Pudding Hills / Squishy Foods pack** (cards 001–016: 8 Common / 4 Rare / 3 Epic / 1 Mythic), extracted verbatim from the source of truth:
`C:\Users\chris\Roblox-squishy\data\raw\launch_squishy_foods.json`.

**Columns:** `Name` (row id / snake_case), `DisplayName`, `Pack`, `Zone`, `Rarity`, `CoinReward`, `BodyArchetype` (Dumpling/JellyCube/Mochi — drives the primitive squishy shape), `ParticlePreset` (→ Niagara `NS_HappyPop` color variant), `SfxSet`, `CardNumber`, `CardArtAsset` (expected texture name in `Content/SquishySmash/Cards/`), `SuggestedTintHex`.

- `CoinReward`, `Rarity`, `BodyArchetype`, `ParticlePreset`, `CardNumber` are **from source data** (exact).
- `SuggestedTintHex` is **suggested** (derived from each friend's theme + the pack palette `#FF8FB8 / #FFD36E / #7FE7FF`). **Reconcile against** the real per-friend skins in `C:\Users\chris\Roblox-squishy\src\ServerScriptService\Server\SquishyModelFactory.lua` and the final card art in `C:\Users\chris\Squishy-smash\` before locking.

## How to use (after bootstrap, in-editor / via MCP)
1. Define a struct `F_SquishyFriend` (or `UDA_SquishyFriend`) with members matching the column names above (Rarity as a `ESquishyRarity` UENUM: Common/Rare/Epic/Mythic).
2. Import this CSV as **`DT_SquishyFriends`** into `Content/SquishySmash/Data/` (Content Browser → Import, pick the struct; or via MCP `DataTableTools`). UE uses the first column (`Name`) as the Row Name.
3. The capsule (M2) and Squishy Book read from this table.

## All three land rosters are pre-staged (48 launch friends)
| CSV | Zone / Pack | Cards | Source JSON |
|---|---|---|---|
| `PuddingHills_Friends.csv` | Pudding Hills / Squishy Foods | 001–016 | `launch_squishy_foods.json` |
| `GooCoast_Friends.csv` | Goo Coast / Goo & Fidgets | 017–032 | `goo_fidgets_drop_01.json` |
| `MoonlitHollow_Friends.csv` | Moonlit Hollow / Creepy-Cute | 033–048 | `creepy_cute_pack_01.json` |

Same column schema for all three. `Rarity`, `CoinReward`, `ParticlePreset`, `CardNumber` are **exact from source**. `BodyArchetype` for Goo Coast comes from each friend's `behaviorProfile` (GooBall/StressBall/JellyCube); for Moonlit Hollow the source `behaviorProfile` is the generic `"creature"`, so it's mapped to the specific **visual** archetype (Bunny/Bat/Ghost/Cat/Blob/Slime/Critter/Creature/Familiar) per name + the Roblox `SquishyModelFactory` shapes. Per-pack palettes used to seed `SuggestedTintHex`: Pudding `#FF8FB8 / #FFD36E / #7FE7FF`, Goo `#B6FF5C / #7FE7FF / #FFD36E`, Moonlit `#B084F2 / #FF8FB8 / #B6FF5C`.

Build the slice from `PuddingHills_Friends.csv`; the other two import the same way when those lands land (LATER).

## Not yet extracted (do when those features land — LATER)
- 8 event friends (Friend-of-the-Week pool) + family "Keepsake" cards — in the Roblox data/defs.
- Alt/early set: `data/raw/dumpling_squishy_drop_01.json`.
