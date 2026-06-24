# PROGRESS.md
> Living changelog. Append after every AI session or major change.
> Any AI reading this will understand the full project history and current state.
> Repository: https://github.com/ahmadrazakhandanish/mestra-app-context.git

---

## How to Use This File

**For the user:** After each session, add a new entry at the bottom with what changed.
**For AI assistants:** Read this ENTIRE file before responding. It tells you:
- What has already been tried (don't suggest it again)
- What failed and why (don't repeat mistakes)
- What's currently broken (focus here)
- What the user wants next

---

## Current State (update this section each time)

**Last updated:** 2026-06-24
**Status:** App runs, pipeline works end-to-end for batches of chunks
**Working:**
- File picker (PowerShell dialog, pops on top)
- ffmpeg chunking (handles special chars via temp copy)
- Stage 1 automation (upload to Maestra, select Uzma, confirm)
- Stage 2 automation (poll for dubbed audio, download, no reload)
- Batch processing (3 concurrent)
- Verification + retry with fresh profiles
- Merge audio chunks
- SSE streaming logs to frontend
- Download individual/merged voiceovers

**Known issues:**
- Page reloads lose all state (no localStorage persistence yet)

**Next up:**
- Fix page reload losing state (persist to localStorage)

---

## Session Log

### Session 1 — 2026-06-24
**Problems reported:**
1. No file picker implementation
2. App keeps refreshing during processing
3. On press refresh, app resets

**Root causes found:**
- Field name mismatch: frontend sent `{filePath, segmentSeconds}` but backend read `{inputFile, chunkLength}`
- No GET handler for /api/chunks (initial load failed silently)
- /api/run streamed raw text but frontend expected SSE `data: {...}\n\n` format
- No file picker existed, just a text input with placeholder paths

**Fix applied:** fix_mega_upgrade.bat
- Aligned field names across all routes
- Added GET /api/chunks handler
- Added SSE format to /api/run
- Added Browse button calling /api/browse

---

### Session 2 — 2026-06-24
**Problem:** Browse button reloads entire app

**Root cause:** `execSync` in /api/browse blocked the Node process. HMR websocket died, browser thought server crashed, reloaded page on reconnect.

**Fix applied:** fix_browse.bat
- Switched to `spawn()` (non-blocking)
- Added `allowedDevOrigins: ['192.168.100.23']` to next.config.mjs

---

### Session 3 — 2026-06-24
**Problem:** Browse button spins forever, nothing happens

**Root cause:** PowerShell `ShowDialog()` opened behind browser window. Background processes don't get foreground focus on Windows.

**Fix applied:** fix_browse_v2.bat
- Created `py/_pick_file.ps1` as standalone script
- Added TopMost owner form trick to force dialog on top
- Route now calls .ps1 file directly (no inline -Command string)

---

### Session 4 — 2026-06-24
**Problem:** Cross-origin warning for second network interface

**Root cause:** Machine has two IPs (192.168.100.23, 172.27.192.1), only first was in allowedDevOrigins.

**Fix applied:** fix_config.bat
- Added both IPs to allowedDevOrigins in next.config.mjs

---

### Session 5 — 2026-06-24
**Problem:** POST /api/chunks returns 500, ffmpeg error code 4294967274

**Root cause:** Filename contained `|` and `?` characters (YouTube download). ffmpeg interprets `|` as pipe protocol and `?` as URL query. Cannot be fixed with quoting.

**Fix applied:** fix_chunks_route.bat
- Added `copyFileSync(filePath, safePath)` before ffmpeg runs
- Safe name: `_temp_input.ext` (no special chars)
- Cleanup: delete temp file after chunking

---

### Session 6 — 2026-06-24
**Problem:** POST /api/chunks returns 400 "File not found" even though file exists

**Root cause:** PowerShell outputs UTF-16 by default. Node reads pipe as UTF-8. Unicode characters in filename get garbled. `existsSync()` fails on the corrupted path string.

**Fix applied:** fix_encoding.bat
- PowerShell now writes selected path to `py/_picked_path.txt` using explicit UTF-8 encoding
- Node reads from temp file directly (no pipe)
- Strips BOM if present

---

### Session 7 — 2026-06-24
**Problem:** Additional UI fixes needed

**Fixes applied:**
- fix_turbopack_crash.bat — stability fix for Turbopack
- fix_logpane.bat / fix_logpane_animated.bat — log pane rendering
- fix_vox_sprites.bat — mascot sprite animation
- fix_fun_animation.bat — UI polish

---

### Session 8 — 2026-06-24
**Status:** Pipeline fully functional. Successfully processed 10 chunks in 4 batches.
**Remaining issue:** Page reload loses all UI state.
**Next:** localStorage persistence for filePath, chunks, step, selected chunks.

---

## Failed Approaches (don't try these again)

| What was tried | Why it failed |
|---------------|---------------|
| execSync for PowerShell file dialog | Blocks entire Node process, kills HMR |
| Inline PowerShell -Command with spawn | Quoting issues, dialog doesn't show |
| Piping file path through stdout | UTF-16 vs UTF-8 encoding mismatch garbles Unicode |
| Passing special-char filenames directly to ffmpeg | `\|` and `?` interpreted as protocol separators |

---

## Template for New Entries

Copy this when adding a new session:

