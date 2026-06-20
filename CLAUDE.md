# CLAUDE.md — Stopmotion-Projects Root

## What This Is

Umbrella repo for AI-assisted stop-motion photorealism horror projects.
Each subdirectory is a submodule (independent git repo).
Root carries the governing pipeline, shared standards, and AI agent context.

## Directory Structure

```
Stopmotion-AI/                    ← root (EE-EDK/Stopmotion-Projects.git)
├── README.md                     ← project index + tool stack
├── WORKFLOW.md                   ← canonical 6-phase pipeline
├── CLAUDE.md / AGENTS.md etc.   ← AI context docs
├── .gitmodules                   ← submodule registry
└── Whisperer in the Wire/        ← submodule → EE-EDK/Whisperer-in-the-Wire.git
```

## Key Files

| File | Purpose |
|------|---------|
| `WORKFLOW.md` | The canonical stop-motion photorealism horror pipeline — read before any production work |
| `.gitmodules` | Submodule registry — update when adding new projects |
| `README.md` | Project index and tool stack reference |

## Working with Submodules

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/EE-EDK/Stopmotion-Projects.git

# Update all submodules to latest
git submodule update --remote --merge

# Add a new project as submodule
git submodule add https://github.com/EE-EDK/<slug>.git "<Project Name>"
git commit -m "add <Project Name> submodule"
git push
```

## Domain Rules

- **Never generate frames without a locked shot bible** — regenerating 500+ frames because
  the story changed is expensive.
- **Character lock text is law** — changes to `character-locks.json` require re-running
  all canonicals and QA-ing previous batches.
- **`generated/`, `refs/`, `output/` are gitignored** in every sub-project — they are
  on-disk and regenerable. Only source and scripts are tracked.
- **Grok CLI runs on SuperGrok login** — do NOT set `XAI_API_KEY`, that routes to the
  paid API. Run frame generation scripts from your own terminal.

## Build Commands (per sub-project)

```bash
# Regenerate shot bible (run after any story/character edit)
python src/generate-shot-bible.py

# Generate frames (run from own PowerShell terminal)
.\scripts\run-frames.ps1 -Section I -Limit 100

# Assemble video section
.\scripts\assemble-sequence.ps1 -Section I -WithAudio
```

## Related Projects

- `The Void is Crimson` (parent dir) — AI video (not stop-motion); the pipeline ancestor
- `ENGINEERING-PROJECTS/ACTIVE-PROJECTS/ai-video-photo/` — workspace root
