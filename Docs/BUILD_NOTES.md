# BUILD_NOTES — engine, install, packaging

## Engine
- **Unreal Engine 5.8** (the **`ModelContextProtocol`** MCP plugin ships with 5.8+; the sibling Gnarly project runs 5.8.0, CL 55116800). Use the same to stay compatible.
- Project type: **Third Person template, Blueprint, Starter Content ON, Raytracing OFF.**

## Known install gotcha
- **`II-E1003`** during UE install/launch = missing **Microsoft Visual C++ x64 runtime**. Fix: install **`vc_redist.x64.exe`** (Gnarly resolved it at runtime `14.50.35719`), then retry. Logged because it cost the sibling project time.

## MCP server (recap — full detail in MCP_NOTES.md)
- Runs **inside the live editor**, `http://127.0.0.1:8000/mcp`. Does not auto-start: console `ModelContextProtocol.StartServer` or Editor Prefs → Auto Start. Needs **`AllToolsets`** enabled for world-building tools. Restart Claude Code after `.mcp.json` exists.

## Source control
- `git init` at repo root `C:\Users\chris\Unreal-squishy`. `.gitignore` excludes UE-generated `Binaries/ Build/ DerivedDataCache/ Intermediate/ Saved/`; **`Content/` (.uasset/.umap), `Config/`, and `*.uproject` ARE tracked.** Commit after milestones (CLAUDE.md §8).

## Packaging (LATER — do not do early)
- Steam (PC) target. Steamworks/packaging deferred until the vertical slice is fun (CLAUDE.md §6). Revisit shipping config, build size, and `Saved/` exclusion then.
