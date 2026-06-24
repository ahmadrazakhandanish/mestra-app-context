# PROJECT_CONTEXT.md

## Purpose
Automated Hindi→Urdu dubbing tool. Takes any audio/video file, splits it into chunks, sends each chunk through Maestra.ai's free voiceover-trial page using headless browser automation, downloads the dubbed audio, and merges everything back together. Runs locally on Windows only.

## Tech Stack
| Layer | Tech |
|-------|------|
| Frontend | Next.js 16.2.6 (Turbopack), TypeScript, Tailwind CSS, shadcn/ui |
| Backend | Python 3, Playwright (headless Chromium) |
| Audio | ffmpeg (segmenting, merging, video→audio extraction) |
| File Picker | PowerShell script (non-blocking spawn from Node) |
| Package Manager | pnpm (pnpm-workspace.yaml) |
| OS | Windows only (E:\media-chunk-processing-app) |

## Directory Structure
E:\media-chunk-processing-app

├── app/

│   ├── page.tsx                  # renders <Workspace/>

│   ├── layout.tsx                # root layout with fonts

│   └── api/

│       ├── browse/route.ts       # PowerShell file dialog

│       ├── chunks/route.ts       # ffmpeg split + chunk listing

│       ├── run/route.ts          # SSE stream, spawns maestra.py

│       ├── download/route.ts     # serve/merge voiceover files

│       ├── merge-video/route.ts  # dubbed audio + original video

│       └── upload/route.ts       # file upload handler

├── components/

│   └── maestra/

│       ├── workspace.tsx         # main UI orchestrator

│       ├── lib.ts                # types, interfaces, helpers

│       ├── log-pane.tsx          # live SSE log display

│       ├── chunk-list.tsx        # chunk selection checkboxes

│       ├── audio-player.tsx      # playback component

│       ├── app-header.tsx        # header with mascot

│       └── step-section.tsx      # step progress UI

├── lib/

│   ├── maestra.ts               # Node wrapper that spawns Python

│   └── utils.ts                 # cn() shadcn utility

├── py/

│   ├── maestra.py               # core automation (40KB+)

│   └── _pick_file.ps1           # Windows file dialog script

├── next.config.mjs              # allowedDevOrigins config

├── package.json                 # "my-project@0.1.0"

└── tsconfig.json

### Runtime folders (created by maestra.py, never commit):
- `chunks/` — numbered .mp3 segments
- `_profiles/` — persistent Chromium user data dirs
- `_debug/` — screenshots + HTML + text dumps on errors
- `_state/` — state.jsonl ledger
- `downloads/voiceovers/` — final dubbed .mp3 files

## Pipeline Flow
[Pick File] → [ffmpeg split ≤55s chunks] → [Stage 1: upload to Maestra] → [Stage 2: poll & download dubbed audio] → [Verify & retry missing] → [Merge all] → [Cleanup]

### Stage 1 (per chunk, headless Chromium):
1. Navigate to https://app.maestra.ai/voiceover-trial
2. Wait for Onboarding dialog
3. Select source language: "Hindi (india)"
4. Select target language: "urdu"
5. Attach chunk .mp3 via file input
6. Click "Upload File"
7. Select voice actor: "Uzma" (may timeout first try, retries)
8. Click "Voiceover" button to confirm

### Stage 2 (per chunk, same persistent profile):
1. Revisit voiceover-trial page
2. Click "AI Dubbing"
3. Wait 20s initial delay
4. Poll network responses for `voiceoverOutput/*.mp3` pattern (NO page reload)
5. Download captured audio URL via HTTP GET
6. Save to downloads/voiceovers/{chunk_number}.mp3

### Concurrency:
- BATCH_SIZE = 3 (ThreadPoolExecutor)
- Each chunk gets its own persistent Chromium profile (unique Firebase user)
- Batches run sequentially, chunks within a batch run in parallel

### Verification & Retry:
- After all batches, compare outputs vs selected chunks
- Missing chunks get retried with fresh profiles (RETRY_PROFILE_BASE=20000)
- retry_chunk_until_success() loops until Stage 2 captures audio
- On full success: cleanup deletes chunks/, _profiles/, _debug/, _state/

## API Contract

### GET /api/browse
Returns: `{ path: "D:\\folder\\file.mp3" }` or `{ path: "" }` on cancel
Behavior: Spawns PowerShell file dialog (non-blocking, 120s timeout), writes selected path to temp file to avoid encoding issues

### GET /api/chunks
Returns: `{ ffmpeg: true, chunks: [{id, index, name, sizeBytes}], sourceFile: "" }`

### POST /api/chunks
Body: `{ filePath: string, segmentSeconds: number }`
Behavior: Copies input to `_temp_input.ext` (avoids special chars), runs ffmpeg segment
Returns: `{ chunks: [...] }` or `{ error: "..." }`

### POST /api/run
Body: `{ filePath, chunks, segmentSeconds, stage2Only, doCleanup }`
Returns: SSE stream with `data: {"type":"log","message":"..."}\n\n` events

### GET /api/download
Params: `?file=3.mp3` | `?merged=true` | `?all=list`

### GET /api/merge-video
Merges `_merged_voiceover.mp3` onto `_source_video.*` using ffmpeg

## Key Constants (maestra.py)
```python
BATCH_SIZE = 3
DEFAULT_SEGMENT_SECONDS = 55
MAX_SEGMENT_SECONDS = 58
CAPTURE_TIMEOUT_S = 120
DUB_FIRST_WAIT_S = 20
POLL_MS = 500
STEP_RETRIES = 2
DEFAULT_TIMEOUT_MS = 30000
NAV_TIMEOUT_MS = 45000
VOICE_TIMEOUT_MS = 35000
PROFILE_PREFIX = "HENG_"
PROFILE_NUMBER_BASE = 10000
RETRY_PROFILE_BASE = 20000


State System
Ledger: _state/state.jsonl — JSONL append-only log
Each line: {"profile": "HENG_10001", "phase": "stage1", "status": "completed", "ts": 1234567890}
Thread-safe writes via LEDGER_LOCK
--resume flag reads ledger to skip completed profiles


Known Quirks
"Uzma" voice button sometimes times out on first click (35s), always works on retry
Each dubbed chunk is ~438-440KB for ~55s audio
Maestra creates unique Firebase users per persistent profile (maestro-218920.appspot.com)
Files with | or ? in names break ffmpeg → must copy to safe temp name first
PowerShell dialogs from background processes don't get focus → need ToHere's a comprehensive PROJECT_CONTEXT.md that any AI will fully grok without needing to ask follow-ups:


# PROJECT_CONTEXT.md
> Auto-context file for AI assistants. Read this + combined_code.txt before responding.

## Project Identity
- **Name:** media-chunk-processing-app (package: my-project@0.1.0)
- **Location:** E:\media-chunk-processing-app
- **Purpose:** Automates Hindi→Urdu audio/video dubbing via Maestra.ai's free voiceover-trial page
- **Status:** Working locally, actively iterated. Not deployed anywhere.
- **Owner:** Solo dev, Engineering role, timezone Asia/Karachi, Windows OS

---

## Tech Stack
| Layer | Tech |
|-------|------|
| Frontend | Next.js 16.2.6 (Turbopack), TypeScript, Tailwind CSS, shadcn/ui |
| Backend (Node) | Next.js API routes (app/api/*) |
| Backend (Python) | Python 3 + Playwright (headless Chromium) |
| File Picker | PowerShell script (py/_pick_file.ps1) |
| Audio Processing | ffmpeg (segment, merge, extract from video) |
| Package Manager | pnpm (pnpm-workspace.yaml) |
| Concurrency | Python ThreadPoolExecutor (3 parallel browsers) |

---

## Architecture & Data Flow

[User clicks Browse] → PowerShell file dialog → returns local path

↓

[POST /api/chunks] → copies file to safe temp name → ffmpeg segments into chunks/1.mp3, 2.mp3...

↓

[POST /api/run] → spawns python py/maestra.py → streams SSE logs to frontend

↓

┌─── BATCH (3 concurrent) ──────────────────────────────────────────┐

│ Stage 1: Navigate maestra.ai/voiceover-trial                       │

│          → Select Hindi (india) source, Urdu target                │

│          → Upload chunk .mp3                                       │

│          → Select voice actor "Uzma"                               │

│          → Click "Voiceover" to confirm                            │

│                                                                    │

│ Stage 2: Revisit same page (persistent profile = same Firebase user)│

│          → Click "AI Dubbing"                                      │

│          → Poll network responses for voiceoverOutput/*.mp3        │

│          → Download dubbed audio via direct HTTP GET                │

│          → NO page reload during polling                           │

└────────────────────────────────────────────────────────────────────┘

↓

[Verification] → Compare outputs vs selected chunks → Retry missing with fresh profiles

↓

[Cleanup] → Delete chunks/, _profiles/, _debug/, _state/ → Keep only downloads/voiceovers/

↓

[Merge] → ffmpeg concat all voiced chunks → optionally mux back onto original video

---

## API Routes

| Method | Route | Input | Output |
|--------|-------|-------|--------|
| GET | /api/browse | — | `{path: "D:\\..."}` (120s timeout) |
| GET | /api/chunks | — | `{ffmpeg: bool, chunks: [...], sourceFile: ""}` |
| POST | /api/chunks | `{filePath, segmentSeconds}` | `{chunks: [...]}` |
| POST | /api/run | `{chunks, stage, cleanup}` | SSE stream: `data: {"type":"log","msg":"..."}\n\n` |
| GET | /api/download?file=3.mp3 | query param | audio/mpeg binary |
| GET | /api/download?merged=true | — | merged voiceover binary |
| GET | /api/download?all=list | — | `{files: [{name, size, url}]}` |
| GET | /api/merge-video | — | dubbed video (maxDuration=300) |
| POST | /api/upload | multipart file | uploaded file path |

---

## File Map (what matters)

app/

page.tsx                    ← renders <Workspace/>

layout.tsx                  ← root layout with fonts

api/

browse/route.tsHere's a comprehensive PROJECT_CONTEXT.md that any AI can use cold without hallucinating:



# PROJECT_CONTEXT.md
> Auto-context file for AI assistants. Read this + combined_code.txt before answering.

## Project
**Name:** media-chunk-processing-app
**Location:** E:\media-chunk-processing-app
**Purpose:** Automates Hindi→Urdu audio/video dubbing using Maestra.ai's free voiceover-trial page. Splits media into chunks, processes each through headless browsers in parallel, downloads dubbed outputs, merges back.
**Status:** Working locally, actively iterated via .bat patches.
**Deployment:** None. Local-only Windows dev tool.

---

## Tech Stack
| Layer | Tech |
|-------|------|
| Frontend | Next.js 16.2.6 (Turbopack), TypeScript, Tailwind CSS, shadcn/ui |
| Backend | Python 3, Playwright (sync API), headless Chromium |
| File Picker | PowerShell script (non-blocking spawn from Node) |
| Audio/Video | ffmpeg (segment, concat, extract, merge) |
| Package Mgr | pnpm (pnpm-workspace.yaml) |
| State | JSONL append-only ledger (_state/state.jsonl) |

---

## Pipeline Flow
[Browse] User picks media file via PowerShell dialog → path written to temp file
[Chunk]  ffmpeg splits audio into ≤55s numbered .mp3 files (chunks/1.mp3, 2.mp3...)
[Stage1] Per chunk: headless Chromium → Maestra → select Hindi/Urdu → upload → select "Uzma" → confirm
[Stage2] Per chunk: revisit page → click "AI Dubbing" → poll network for voiceoverOutput/*.mp3 → download
[Verify] Compare outputs vs selected chunks → retry missing with fresh profiles
[Merge]  ffmpeg concat all dubbed chunks → optionally mux back onto original video
[Clean]  Delete working dirs (chunks/, _profiles/, _debug/, _state/), keep downloads/voiceovers/
---

## API Routes (Next.js, all under app/api/)

| Method | Route | Accepts | Returns |
|--------|-------|---------|---------|
| GET | /api/browse | — | `{path: string}` (120s timeout) |
| GET | /api/chunks | — | `{ffmpeg: bool, chunks: [...], sourceFile: ""}` |
| POST | /api/chunks | `{filePath, segmentSeconds}` | `{chunks: [...]}` or `{error}` |
| POST | /api/run | `{chunks, stage, filePath, segmentSeconds, doCleanup}` | SSE stream: `data: {"type","payload"}\n\n` |
| GET | /api/download?file=3.mp3 | query param | audio/mpeg binary |
| GET | /api/download?merged=true | — | merged voiceover binary |
| GET | /api/download?all=list | — | `{files: [...]}` |
| GET | /api/merge-video | — | dubbed video (maxDuration=300) |
| POST | /api/upload | multipart file | uploaded file path |

---

## File Map (what matters)
app/

page.tsx                    — renders <Workspace />

layout.tsx                  — root layout, fonts, metadata

api/browse/route.ts         — PowerShell file dialog (spawn, non-blocking)

api/chunks/route.ts         — ffmpeg check + chunking (copies to safe temp name first)

api/run/route.ts            — SSE streaming, spawns python py/maestra.py

api/download/route.ts       — serve/merge voiceover files

api/merge-video/route.ts    — mux dubbed audio onto source video

api/upload/route.ts         — file uploaHere's a much more complete PROJECT_CONTEXT.md. Save this in your repo root:



# PROJECT_CONTEXT.md

> This file is the single source of truth for AI assistants working on this codebase.
> Last updated: 2026-06-24

---

## 1. What This App Does

Automates Hindi→Urdu audio/video dubbing using Maestra.ai's free voiceover-trial page.
User picks a media file, app splits it into chunks, processes each chunk through Maestra
using headless browsers (one per chunk), downloads the dubbed audio, and merges everything back.

Runs locally on Windows only. Never deployed. Personal automation tool.

---

## 2. Tech Stack

| Layer | Tech |
|-------|------|
| Frontend | Next.js 16.2.6 (Turbopack), TypeScript, Tailwind CSS, shadcn/ui |
| Backend | Python 3 + Playwright (headless Chromium) |
| File Picker | PowerShell script (py/_pick_file.ps1) |
| Audio Processing | ffmpeg (chunking, merging, video→audio extraction) |
| Package Manager | pnpm (pnpm-workspace.yaml) |
| OS | Windows only |
| Location | E:\media-chunk-processing-app |

---

## 3. Pipeline Flow

[Pick File] → [ffmpeg Split] → [Stage 1: Upload to Maestra] → [Stage 2: Poll & Download Dubbed Audio] → [Verify & Retry Missing] → [Merge All] → [Cleanup]

### Stage 1 (per chunk, per profile):
1. Launch persistent Chromium context (unique profile per chunk)
2. Navigate to https://app.maestra.ai/voiceover-trial
3. Wait for Onboarding dialog
4. Select source: "Hindi (india)", target: "urdu"
5. Attach chunk .mp3 via file input
6. Click "Upload File"
7. Select voice actor "Uzma"
8. Click "Voiceover" button to confirm

### Stage 2 (per chunk, same profile):
1. Revisit the same URL
2. Click "AI Dubbing"
3. Wait 20s initial delay
4. Poll network responses for `voiceoverOutput/*.mp3` pattern (NO page reload)
5. Download captured audio URL via requests
6. Save to downloads/voiceovers/{chunk_number}.mp3

### Concurrency:
- BATCH_SIZE = 3 (processes 3 chunks simultaneously)
- ThreadPoolExecutor manages batches
- threading.Event for graceful interrupt (Ctrl+C safe)

---

## 4. API Routes

| Method | Route | Purpose |
|--------|-------|---------|
| GET | /api/browse | Opens Windows file dialog (PowerShell, non-blocking, 120s timeout). Returns `{path: "..."}` |
| GET | /api/chunks | Returns `{ffmpeg: bool, chunks: [...], sourceFile: ""}` |
| POST | /api/chunks | Body: `{filePath, segmentSeconds}`. Copies to safe temp name, runs ffmpeg segment |
| POST | /api/run | SSE stream (`data: {...}\n\n`). Spawns `python maestra.py` with args |
| GET | /api/download?file=3.mp3 | Serve single dubbed file |
| GET | /api/download?merged=true | Merge all chunks + serve |
| GET | /api/download?all=list | List available files |
| GET | /api/merge-video | Combine dubbed audio with original video (maxDuration=300) |
| POST | /api/upload | File upload handler |

---

## 5. File Structure (relevant files only)

E:\media-chunk-processing-app

├── app/

│   ├── page.tsx                          # Root page, renders <Workspace/>

│   ├── layout.tsx                        # Root layout

│   └── api/

│       ├── browse/route.ts               # PowerShell file picker

│       ├── chunks/route.ts               # ffmpeg chunking

│       ├── run/route.ts                  # SSE pipeline execution

│       ├── download/route.ts             # File serving + merge

│       ├── merge-video/route.ts          # Video+audio merge

│       └── upload/route.ts               # File upload

├── components/

│   └── maestra/

│       ├── workspace.tsx                 # Main UI (state, steps, orchestration)

│       ├── lib.ts                        # Types + helpers

│       ├── log-pane.tsx                  # Live SSE log display

│       ├── chunk-list.tsx                # Chunk selection checkboxes

│       ├── audio-player.tsx              # Playback component

│       ├── app-header.tsx                # Header with mascot