# CONTENT_STRUCTURE_PLAN — Content/SquishySmash/ layout

All game content lives under **`SquishySmashUE/Content/SquishySmash/`** (created in-editor / via MCP during M1). Engine Starter Content + `/Engine/BasicShapes` are fine as placeholders.

```
Content/SquishySmash/
  Maps/         LVL_PuddingHills            (greybox → art-passed land)
  Blueprints/
    Character/  BP_SquishyCharacter, BP_SquishyPlayerController, GM_SquishySmash
    Friends/    BP_SquishyFriend, BP_FriendPad, BP_BuddyFollower, AC_JoyComponent
    Gameplay/   BP_SquishyGameState, BP_CapsuleStation, BP_QuestManager, BP_SparkleShard
    UI/         (logic-side widget owners if any)
  UI/           WBP_HUD, WBP_SquishyBook, WBP_CapsuleReveal, WBP_Toast, WBP_QuestHint
  Input/        IMC_Squishy, IA_Move, IA_Look, IA_Jump, IA_Squish, IA_Interact
  Materials/    M_JellyPlush, MI_Friend_* (per-friend tints), M_Ground_Pudding, M_Card
  Meshes/       (primitive-built friend parts now; sculpted meshes later)
  Niagara/      NS_HappyPop (sparkle burst ~26), NS_ShardGlow, NS_SquishPuff
  Audio/        SFX squish "Pmf"/Happy-Pop, cozy ambience + menu music (from Roblox assets/)
  Data/         DT_SquishyFriends (ported from SquishyDefinitions.lua), DA_ rarity/zone config
  Cards/        T_Card_001..016 (Pudding Hills card art, imported from C:\Users\chris\Squishy-smash\)
```

## MVP asset checklist (M1–M4)
- **M1:** `LVL_PuddingHills`, `BP_SquishyCharacter`, `IMC_Squishy`+`IA_*`, `BP_SquishyFriend`(+`AC_JoyComponent`), `BP_FriendPad`, `NS_HappyPop`, `BP_SquishyGameState`, `WBP_HUD`, `GM_SquishySmash`.
- **M2:** `DT_SquishyFriends`, `BP_CapsuleStation`, `WBP_CapsuleReveal`, `WBP_SquishyBook`, `Cards/T_Card_001..016`.
- **M3:** `BP_BuddyFollower`, `BP_QuestManager`, `BP_SparkleShard`, `SG_SquishyProfile`.
- **M4:** `M_JellyPlush` + `MI_Friend_*`, kawaii-eye decals, audio, Fredoka UI font, lighting/bloom pass.

Naming per CLAUDE.md §9. Build only what the current milestone needs.
