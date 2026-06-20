# BUGS — open issues & gotchas

Format: `[date] severity — symptom — repro — suspected cause — status`.

_None yet — project not built. Log MCP/editor/gameplay issues here as they appear._

## Watch-list (anticipated, from sibling project + MCP notes)
- MCP tools missing after enabling plugins → editor **not restarted**, or **`AllToolsets`** not enabled (only ~4 tools show). 
- MCP tools absent in Claude → **Claude Code not restarted** after `.mcp.json`, or trust prompt not approved, or **server not started** (no `port 8000` log line).
- `call_tool "Unknown tool"` → pass the **bare** `tool_name` + separate `toolset_name` (not the fully-qualified path).
- Port **8000** collision if the Gnarly editor is open simultaneously → run one editor, or use `:8001` (see BOOTSTRAP step 3).
- `set_actor_transform` silently resetting fields → always pass full **location + rotation + scale**.
