# DEVLOG — Squishy Smash UE

Newest first. Format: **date — task — files/assets — what worked — what broke — next.**

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
