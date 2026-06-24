# BUGS — open issues & gotchas

Format: `[date] severity — symptom — repro — suspected cause — status`.

### Open
- **[2026-06-22] minor — squish pops on 2 clicks, expected 3** — Walk up to a friend, tap squish: it Happy-Pops one tap early (e.g. JoyPerSquish 0.34 → 2 taps; 0.5 → 1 tap). **Repro:** real LMB *or* F input, in PIE **and** Standalone; both keys. **NOT** reproducible under synthetic Slate `ProcessMouseButtonDownEvent` injection (always single-fire). — **Suspected cause:** the real key press delivers a duplicate "Pressed" ~0.4s later (a phantom second press *after* release), double-incrementing Joy. The Blueprint graph is verified single-fire at the pin level (one `LMB Pressed → OnSquishPressed`, one `Squish` call, threshold genuinely 1.0); removing the dead `IMC_Squishy` context, a per-press disarm lock, and a 0.35s cooldown all failed to stop it. — **Status: PARKED (deprioritized).** Loop is fully playable at 2 taps. Tried (all failed): IMC_Squishy removal, arm-on-release gate, disarm-on-squish lock, cooldown 0.35. Did NOT yet try: a UMG-button trigger, Enhanced-Input-only (no legacy), or measuring the exact phantom gap with timestamped logs + real input. Revisit fresh.

### Watch-list (anticipated, from sibling project + MCP notes)
- MCP tools missing after enabling plugins → editor **not restarted**, or **`AllToolsets`** not enabled (only ~4 tools show). 
- MCP tools absent in Claude → **Claude Code not restarted** after `.mcp.json`, or trust prompt not approved, or **server not started** (no `port 8000` log line).
- `call_tool "Unknown tool"` → pass the **bare** `tool_name` + separate `toolset_name` (not the fully-qualified path).
- Port **8000** collision if the Gnarly editor is open simultaneously → run one editor, or use `:8001` (see BOOTSTRAP step 3).
- `set_actor_transform` silently resetting fields → always pass full **location + rotation + scale**.
