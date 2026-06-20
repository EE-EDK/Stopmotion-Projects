# GROK.md — Stopmotion-Projects

## Grok's Role in This Pipeline

Grok (SuperGrok, $0 credit) handles all image generation via `GenerateImage` (text→image).
It does not maintain memory between calls — identity consistency is carried entirely by the
**cast lock text** injected into every prompt.

## Execution Context

- Run Grok CLI from your own PowerShell terminal (not via Claude/Gemini agent)
- Do NOT set `XAI_API_KEY` — SuperGrok login uses the free allowance
- Frame scripts: `.\scripts\run-frames.ps1` in each sub-project

## What Grok Reviews

In QA sessions, Grok can review assembled cuts (watch video + read intent docs).
See `toGROK/INDEX.md` in any project that uses this review pattern.
Focus areas: motion artifacts, lip-sync, flow, audio nuance, failure modes beyond still-frame analysis.

## Known Grok Failure Modes

See `docs/specs/prompt-hazards.md` in each sub-project. Common ones:
- Literal "laser-line" artifacts across eyes for eye-effect prompts
- Solid rendering of translucent/ghost characters
- Invented text on background objects (suppress with "no text, no signage")
- Off-screen characters lip-synced onto visible actors
