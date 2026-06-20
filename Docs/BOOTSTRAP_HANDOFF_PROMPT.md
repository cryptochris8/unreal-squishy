# BOOTSTRAP_HANDOFF_PROMPT — bring the Unreal MCP live for Squishy Smash

The repo scaffold (Phase 0) is done. To build in-editor, the Unreal MCP must be connected. Tags: **[CHRIS]** = manual (editor/launcher/restart) · **[CLAUDE]** = I do it once the MCP is live.

> Why manual at all: the Unreal MCP server runs **inside a live UE 5.8 editor**, and only the editor can generate a real `.uproject`/`Config`/`Content`. I can't create those from the shell. After the steps below, I can build the world directly via MCP.

## Step 1 — Create the UE 5.8 project  [CHRIS]
1. Epic Launcher → **Unreal Engine 5.8** → **Launch**. (If install error **`II-E1003`** appears, install **`vc_redist.x64.exe`** (Microsoft VC++ x64 runtime) and retry — see `BUILD_NOTES.md`.)
2. **Games → Third Person**, settings: **Blueprint** (not C++), **Starter Content: ON**, **Raytracing: OFF**, target Desktop.
3. **Project Name:** `SquishySmashUE` · **Location:** `C:\Users\chris\Unreal-squishy`
   → result: `C:\Users\chris\Unreal-squishy\SquishySmashUE\SquishySmashUE.uproject` (sits beside this `Docs/` and `.mcp.json`). **Create.**

## Step 2 — Enable the MCP plugins  [CHRIS]
4. Edit → **Plugins**. Enable **"Unreal MCP" (`ModelContextProtocol`)** and **`AllToolsets`**. *(Do NOT enable "MCP Client Toolset.")* **`AllToolsets` is required** — without it only ~4 useless tools appear.
5. **Restart the editor** (plugins load on restart).

## Step 3 — Start the MCP server  [CHRIS]
6. Open the console (backtick `` ` ``) and run: **`ModelContextProtocol.StartServer`**
   (or set Editor Preferences → *Model Context Protocol* → **Auto Start Server**, then restart.)
7. Confirm the Output Log shows: **`LogModelContextProtocol: Starting MCP server on port 8000`**.
   - Leave this editor **open** — it *is* the MCP server. (If you also run the Gnarly editor at the same time they'll both want port 8000; run them one at a time, or start one with `ModelContextProtocol.StartServer 8001` and point that project's `.mcp.json` at `:8001`.)

## Step 4 — Reconnect Claude Code  [CHRIS]
8. **Restart Claude Code** in `C:\Users\chris\Unreal-squishy` (it reads `.mcp.json` on start). Approve the **`unreal`** server trust prompt.
9. Tell me **"MCP is live"** (or just say go).

## Step 5 — Verify + start building  [CLAUDE]
10. I run the smoke test (`list_toolsets` → spawn/find/remove a test cube → capture viewport → PIE) and log it to `MCP_NOTES.md`.
11. I create `Content/SquishySmash/` and start **M1 — the Pudding Hills core squish loop** (`NEXT_STEPS.md`), then we commit.

---
### Verification prompt to paste after reconnecting (optional)
> "MCP is live. Run the Unreal MCP smoke test (list_toolsets, spawn+find+remove a test cube, capture viewport, PIE start/stop), log results to Docs/MCP_NOTES.md, then begin M1 from Docs/NEXT_STEPS.md."

### Exact `.mcp.json` (already written at repo root — no change needed)
```json
{ "mcpServers": { "unreal": { "type": "http", "url": "http://127.0.0.1:8000/mcp" } } }
```
