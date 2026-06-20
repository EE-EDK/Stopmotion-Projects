# Stopmotion-Projects

Umbrella repository for AI-assisted stop-motion photorealism horror productions.
Each sub-project is an independent git repo wired in as a submodule here.
Governing pipeline, world-building standards, and AI tooling live at root.

## Projects

| Submodule | Status | Logline |
|-----------|--------|---------|
| [Whisperer in the Wire](Whisperer%20in%20the%20Wire/) | 🟡 Pre-production | TBD |

## Shared Pipeline

See [`WORKFLOW.md`](WORKFLOW.md) — the canonical stop-motion photorealism horror pipeline,
adapted from *The Void is Crimson* (AI video) and extended for frame-by-frame puppet aesthetics.

## Tool Stack

| Tool | Role |
|------|------|
| Grok CLI (SuperGrok) | Frame generation — `GenerateImage` text→image |
| `paths.py` (per-project) | Single source of truth for all file paths |
| Shot bible generator (Python) | Turns production data → JSON / CSV / storyboard HTML |
| PowerShell scripts | Drive Grok CLI batch jobs, FFmpeg assembly |
| Kokoro TTS | Narration / voice synthesis |
| FFmpeg | Frame sequence → video, audio mux, concat |

## Repo Map

```
Stopmotion-AI/                    ← root (this repo → EE-EDK/Stopmotion-Projects.git)
├── README.md
├── WORKFLOW.md                   ← canonical pipeline
├── CLAUDE.md / AGENTS.md etc.   ← AI agent context
└── Whisperer in the Wire/        ← submodule → EE-EDK/Whisperer-in-the-Wire.git
```

## Adding a New Project

1. Init a local git repo under `Stopmotion-AI/<Project Name>/`
2. Create `EE-EDK/<project-slug>` on GitHub (private)
3. Scaffold with the project template (see `WORKFLOW.md § Project Bootstrap`)
4. `git submodule add https://github.com/EE-EDK/<project-slug>.git "<Project Name>"`
5. Push both repos
6. Add a row to the Projects table above
