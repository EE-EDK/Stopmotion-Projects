# GEMINI.md — Stopmotion-Projects

## Gemini's Role

Lead Architect for large-document analysis: screenplay review, world bible coherence,
multi-file structural analysis. Defer to Claude for environmental/tooling troubleshooting.

## When to Invoke Gemini

- Digesting a full screenplay HTML for beat/tone consistency
- Cross-referencing character locks against all frame prompts for drift
- Architectural decisions on pipeline structure or shot bible schema
- Any task requiring >50-page document synthesis

## Do Not Ask Gemini To

- Run PowerShell scripts or git commands (Claude/local terminal handles that)
- Generate Grok prompts for individual frames (Claude + production-data.py handles that)
- Make file edits (use Claude Code for file operations)
