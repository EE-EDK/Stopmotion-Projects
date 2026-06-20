# AGENTS.md — Stopmotion-Projects

## Role of This Repo

Umbrella for AI stop-motion photorealism horror projects. Each sub-project is a submodule.
This root carries the canonical pipeline (WORKFLOW.md) and governing standards.

## Startup Brief

1. Read `WORKFLOW.md` to understand the 6-phase production pipeline.
2. Open the sub-project's `README.md` for its current phase + build commands.
3. Character locks are the identity source of truth — never alter `character-locks.json`
   without regenerating canonicals and QA-ing existing frames.

## Agent Assignments

| Task | Agent |
|------|-------|
| Pipeline design, shot bible generation, prompt architecture | Claude (SRE/architect) |
| World Bible review, screenplay QA, large-doc analysis | Gemini (Lead Architect) |
| Frame generation, batch automation | Run locally via PowerShell / Grok CLI |
| Voice synthesis, audio mix | Run locally via `scripts/build-audio.py` |

## Submodule Policy

- Root commits only `.gitmodules` entries and governing docs — never project assets.
- Sub-project commits are independent; update root pointer with `git submodule update --remote`.
- Add new projects via: `git submodule add <url> "<Name>"` from root.
