# AI_GUIDELINES.md
> Instructions for any AI assistant working on this codebase.
> Repository: https://github.com/ahmadrazakhandanish/mestra-app-context.git

---

## How to Use This Repository

This repo contains context files for AI assistants:

| File | Purpose | Update Frequency |
|------|---------|-----------------|
| `combined_code.txt` | All source code in one file | After every code change |
| `PROJECT_DETAILS.md` | Architecture, API contracts, constants | Occasionally |
| `AI_GUIDELINES.md` | This file. How to approach the project | Rarely |
| `project_structure.txt` | Directory tree | After adding/removing files |
| `PROGRESS.md` | Living changelog, failed approaches | After each session |
| `CHAT.md` | Session summaries with timestamps | When user says "summarize" |

---

## How to Read combined_code.txt

The file concatenates all source files with separator headers. Each file begins with:

==== E:\media-chunk-processing-app<relative_path> ====

### To find a specific file's code:
Search for the header pattern. Examples:

| You need... | Search for... |
|------------|---------------|
| Main UI component | `==== E:\media-chunk-processing-app\components\maestra\workspace.tsx ====` |
| Python automation server | `==== E:\media-chunk-processing-app\py\maestra.py ====` |
| File picker API | `==== E:\media-chunk-processing-app\app\api\browse\route.ts ====` |
| Chunks API | `==== E:\media-chunk-processing-app\app\api\chunks\route.ts ====` |
| Run/SSE API | `==== E:\media-chunk-processing-app\app\api\run\route.ts ====` |
| Download API | `==== E:\media-chunk-processing-app\app\api\download\route.ts ====` |
| Merge video API | `==== E:\media-chunk-processing-app\app\api\merge-video\route.ts ====` |
| Upload API | `==== E:\media-chunk-processing-app\app\api\upload\route.ts ====` |
| Node Python wrapper | `==== E:\media-chunk-processing-app\lib\maestra.ts ====` |
| Types/helpers | `==== E:\media-chunk-processing-app\components\maestra\lib.ts ====` |
| Log pane UI | `==== E:\media-chunk-processing-app\components\maestra\log-pane.tsx ====` |
| Chunk list UI | `==== E:\media-chunk-processing-app\components\maestra\chunk-list.tsx ====` |
| Audio player | `==== E:\media-chunk-processing-app\components\maestra\audio-player.tsx ====` |
| App header | `==== E:\media-chunk-processing-app\components\maestra\app-header.tsx ====` |
| Step section | `==== E:\media-chunk-processing-app\components\maestra\step-section.tsx ====` |
| PowerShell picker | `==== E:\media-chunk-processing-app\py\_pick_file.ps1 ====` |
| Next.js config | `==== E:\media-chunk-processing-app\next.config.mjs ====` |
| Package.json | `==== E:\media-chunk-processing-app\package.json ====` |
| TypeScript config | `==== E:\media-chunk-processing-app\tsconfig.json ====` |
| Root page | `==== E:\media-chunk-processing-app\app\page.tsx ====` |
| Root layout | `==== E:\media-chunk-processing-app\app\layout.tsx ====` |

A file's code starts immediately after its header and ends when the next `====` header appears (or EOF).

---

## Full Directory Structure

E:\media-chunk-processing-app

├── app/

│   ├── page.tsx                          # Root page, renders <Workspace/>

│   ├── layout.tsx                        # Root layout (fonts, metadata)

│   ├── globals.css                       # Tailwind base styles

│   └── api/

│       ├── browse/route.ts               # PowerShell file dialog (non-blocking)

│       ├── chunks/route.ts               # GET: ffmpeg status + list | POST: split file

│       ├── run/route.ts                  # POST: SSE stream, spawns maestra.py

│       ├── download/route.ts             # GET: serve/merge voiceover files

│       ├── merge-video/route.ts          # GET: mux dubbed audio onto video

│       └── upload/route.ts               # POST: file upload

├── components/

│   ├── maestra/

│   │   ├── workspace.tsx                 # Main UI orchestrator (all state lives here)

│   │   ├── lib.ts                        # TypeScript types, interfaces, helpers

│   │   ├── log-pane.tsx                  # Live SSE log display

│   │   ├── chunk-list.tsx                # Chunk selection with checkboxes

│   │   ├── audio-player.tsx              # Audio playback component

│   │   ├── app-header.tsx                # Header bar with mascot sprite

│   │   └── step-section.tsx              # Step progress indicator

│   └── ui/                               # shadcn/ui primitives (don't modify)

│       ├── badge.tsx

│       ├── button.tsx

│       ├── card.tsx

│       ├── checkbox.tsx

│       ├── input.tsx

│       ├── label.tsx

│       ├── radio-group.tsx

│       ├── scroll-area.tsx

│       ├── separator.tsx

│       ├── slider.tsx

│       └── switch.tsx

├── lib/

│   ├── maestra.ts                        # Node wrapper that spawns Python process

│   └── utils.ts                          # cn() utility for shadcn

├── py/

│   ├── maestra.py                        # Core automation (Playwright, ~40KB)

│   └── _pick_file.ps1                    # Windows file dialog script

├── public/

│   └── sprites/

│       ├── vox-idle.png                  # Mascot idle state

│       ├── vox-sing.png                  # Mascot active state

│       └── vox-dizzy.png                 # Mascot error state

├── next.config.mjs                       # allowedDevOrigins for local IPs

├── package.json                          # "my-project@0.1.0"

├── pnpm-workspace.yaml

├── tsconfig.json

├── postcss.config.mjs

└── components.json                       # shadcn/ui config

### Runtime folders (created by maestra.py, NEVER in repo):
├── chunks/                   # Numbered .mp3 segments (1.mp3, 2.mp3...)

├── _profiles/                # Persistent Chromium user data dirs (one per chunk)

├── _debug/                   # Screenshots + HTML + text on errors

├── _state/                   # state.jsonl ledger (append-only)

├── downloads/voiceovers/     # Final dubbed .mp3 files (NEVER deleted by cleanup)

├── .next/                    # Next.js build cache

└── node_modules/             # pnpm packages

---

## Rules for AI Assistants

### Before answering:
1. **Always read the relevant file(s) from combined_code.txt first.** Don't guess what the code looks like.
2. **Check PROJECT_DETAILS.md** for architecture decisions, API contracts, and constants.
3. **Read PROGRESS.md** to see what's been tried and failed. Never suggest a failed approach again.
4. **Cross-reference**: if a bug involves frontend-to-backend communication, read BOTH the component AND the API route.

### When proposing fixes:
1. **Deliver as .bat files** the user can double-click on Windows. Not manual instructions.
2. **One comprehensive fix** per issue. Don't iterate back and forth if avoidable.
3. **Specify exact file paths** relative to `E:\media-chunk-processing-app\`.
4. **Show root cause** in 2-3 sentences, then the fix. Don't over-explain.

### Never do:
- Don't touch `node_modules/`, `.next/`, `chunks/`, `downloads/`, `_profiles/`, `_debug/`, `_state/`
- Don't modify `components/ui/*` (those are shadcn primitives, auto-generated)
- Don't suggest deploying this app (it's local-only, by design)
- Don't rename files or restructure the project without explicit permission
- Don't assume code you h


How to Read combined_code.txt


The file concatenates all source files with separator headers. Each file begins with:



==== E:\media-chunk-processing-app<relative_path> ====



To find a specific file's code:
Search for the header pattern. Examples:



You need...

Search for...

Main UI component

workspace.tsx

Python automation server

py\maestra.py

File picker API

app\api\browse\route.ts

Chunks API

app\api\chunks\route.ts

Run/SSE API

app\api\run\route.ts

Download API

app\api\download\route.ts

Merge video API

app\api\merge-video\route.ts

Upload API

app\api\upload\route.ts

Node Python wrapper

lib\maestra.ts

Types/helpers

components\maestra\lib.ts

Log pane UI

components\maestra\log-pane.tsx

Chunk list UI

components\maestra\chunk-list.tsx

Audio player

components\maestra\audio-player.tsx

App header

components\maestra\app-header.tsx

Step section

components\maestra\step-section.tsx

PowerShell picker

py_pick_file.ps1

Next.js config

next.config.mjs

Package.json

package.json

TypeScript config

tsconfig.json

Root page

app\page.tsx

Root layout

app\layout.tsx

