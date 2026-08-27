# AGENTS.md — Stopmotion-Projects

## Role of This Repo

Umbrella for AI stop-motion photorealism horror projects. Each sub-project is a submodule.
This root carries the canonical pipeline (WORKFLOW.md) and governing standards.

## Startup Brief

1. For work on **Whisperer in the Wire**: read `Whisperer in the Wire/docs/production-plan-v3.md` first — it is the canonical plan and overrides this WORKFLOW.md on conflict.
2. Check `Whisperer in the Wire/README.md` for current phase and open decisions.
3. Read `WORKFLOW.md` for the base pipeline; note that Whisperer extends it with a two-register grammar and Phase 3B.
4. Character locks in `src/character-locks.json` are law — do not change without regenerating canonicals and QA-ing existing frames.

## Agent Assignments

| Task | Agent |
|------|-------|
| Pipeline design, shot bible generation, prompt architecture, file ops | Claude (SRE/architect) |
| Screenplay QA, world bible coherence, large-doc analysis, beat/tone audit | Gemini (Lead Architect) |
| Frame generation (Register A), batch automation | Run locally via PowerShell / Grok CLI |
| Register B source images, image-to-video live holds | Run locally (tool TBD in Phase 3) |
| Voice synthesis, audio mix | Run locally via `scripts/build-audio.py` |
| Environmental/tooling troubleshooting | Claude (SRE — Gemini defers to Claude for this) |

## Submodule Policy

- Root commits only `.gitmodules` entries and governing docs — never project assets.
- Sub-project commits are independent; update root pointer with `git submodule update --remote`.
- Add new projects via: `git submodule add <url> "<Name>"` from root.
