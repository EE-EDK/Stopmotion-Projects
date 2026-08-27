# GROK.md — Stopmotion-Projects

## Grok's Role in This Pipeline

Grok (SuperGrok, $0 credit) handles **Register A image generation** via `GenerateImage` (text→image).
It does not maintain memory between calls — identity consistency is carried entirely by the
**cast lock text** injected into every prompt.

> **Register B (hyperreal) does NOT use Grok GenerateImage** — it requires a high-resolution
> photoreal model and/or image-to-video tooling. See the two-register grammar in WORKFLOW.md
> and `Whisperer in the Wire/docs/specs/tone-guide.md`.

## Execution Context

- Run Grok CLI from your own PowerShell terminal (not via Claude/Gemini agent)
- Do NOT set `XAI_API_KEY` — SuperGrok login uses the free allowance
- Frame scripts: `.\scripts\run-frames.ps1` in each sub-project

## What Grok Generates

| Use | Tool | Register |
|-----|------|----------|
| Stop-motion stills (main production) | GenerateImage | A |
| Concept reference frames (look-dev) | GenerateImage or Grok Imagine | Concept only |
| Register B source images | High-res photoreal model (TBD Phase 3) | B |

## What Grok Reviews

In QA sessions, Grok can review assembled cuts (watch video + read intent docs).
Focus areas: motion artifacts, lip-sync, flow, audio nuance, failure modes beyond still-frame analysis.
See `docs/qa/` in each sub-project for current QA notes.

## Known Grok Failure Modes

See `docs/specs/prompt-hazards.md` in each sub-project. Common ones:
- Literal "laser-line" artifacts across eyes for eye-effect prompts
- Solid rendering of translucent/ghost characters
- Invented text on background objects (suppress with "no text, no signage")
- Off-screen characters lip-synced onto visible actors
- Identity drift from generic character descriptions (use ~80-word specific lock text)
