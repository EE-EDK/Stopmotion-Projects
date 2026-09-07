# Stop-Motion Photorealism Horror — Production Pipeline

Adapted from *The Void is Crimson* AI video pipeline.
Key difference: every Register A frame is a **discrete still** rendered by Grok; the stop-motion
aesthetic is baked into the prompts and timing metadata, not post-processed.

> **Project-level extensions:** sub-projects may add phases, registers, or constraints beyond this
> base pipeline. *Whisperer in the Wire* extends this with a two-register grammar (Register A /
> Register B) and Phase 3B (animatic gate). See `Whisperer in the Wire/docs/production-plan-v3.md`
> for the canonical project plan when working on that sub-project. On conflict, the sub-project
> plan wins over this document.

---

## Two-Register Grammar (Whisperer extension)

Some sub-projects operate in two visual-ontological modes. Understand this before working on Whisperer:

| Register | Medium | Frame rate | Character |
|----------|--------|------------|-----------|
| **A — Mortal** | Stop-motion stills (Grok GenerateImage) | ~8 fps (jitter) | Handmade, imperfect |
| **B — Hyperreal** | Photoreal imagery (high-res + image-to-video) | Smooth | Wrong, over-resolved |

A→B transitions are always **hard cuts**. Never blend. Register B appears only at designated breach beats.

---

## Phase 0 — World Bible

Lock these before generating a single frame. Changes after Phase 1 = expensive regeneration.

### 0-A Character Locks
Each character needs a **cast lock**: a dense text description (~80 words) covering:
- Physical form (scale, material — clay, felt, wire armature, resin, etc.)
- Distinguishing features that must survive every frame
- Costume / color palette
- Horror-specific: decay state, uncanny trait, puppet seam visibility intent

Store in `src/character-locks.json`. These strings inject verbatim into every frame prompt.
Mirror *The Void is Crimson* pattern: one lock per character, versioned in git.

### 0-B Set/Location Bible
- Practical miniature or fully AI-generated environment?
- Lighting mood per location (practical tungsten vs. cold fluorescent vs. total darkness)
- Scale cues: forced perspective tricks that read as miniature

### 0-C Horror Tone Guide
Commit these to `docs/specs/tone-guide.md` before scripting:
- Color palette (desaturated base + single accent — e.g. rust/bile/arterial)
- Frame grain target (ISO equiv for the "practical film" feel)
- Stop-motion artifact intent: deliberate jitter? smooth or staccato motion?
- Uncanny threshold: how photorealistic vs. obviously-artificial

---

## Phase 1 — Story / Script *(hard gate)*

Write the script before touching the pipeline. A locked script is mandatory before any frame generation.

### 1-A Screenplay
Write the screenplay in `docs/screenplay/` — browsable in a Grok/Claude review session. Include:
- Beat markers tying to dialogue JSON
- Register tags: [REGISTER-A] / [REGISTER-B] per scene
- ON-SCREEN vs. OFF-SCREEN voice tags

### 1-B Shot Bible Generator
`src/generate-shot-bible.py` is the **single source of truth generator**. It reads:
- `src/production-data.py` — per-beat metadata (location, characters, emotion, camera, register)
- `src/character-locks.json` — cast descriptions (injected into every Register A frame prompt)
- `src/enriched-descriptions.json` — per-shot flavor text (overrides defaults)
- `src/dialogue.json` — beat → [{speaker, line, direction, timing}]

It emits:
- `output/shot-bible.json` — master manifest
- `output/prompts.csv` — one row per frame with full Grok prompt
- `output/storyboard.html` — browsable storyboard

**Regenerate after every edit** to production data — never hand-edit the output files.

### 1-C Frame Rate Planning
Stop-motion timing is not linear. Document in `docs/specs/timing.md`:
```
fps_register_a = 8     # mortal register — jitter is the point
fps_register_b = 24    # hyperreal register — smoothness is the grammar
hold_frames = {
  "idle":   2,         # character stationary
  "action": 1,         # full-motion
  "impact": 3,         # freeze on a hit/scare
  "frozen": 8,         # stillness-as-dread
}
```
Total frame count = sum(beat_duration_sec × fps / hold_multiplier).

---

## Phase 2 — Design / Look-dev & Pre-production

Both registers must be defined as buildable craft before any production work begins.

Key deliverables to `docs/look-dev/`:
- Lookbook (both registers + approved final-image frame)
- Continuity bible (chair states, replacement-part inventory, set dressing across sessions)
- Voice casting notes

---

## Phase 3 — Pipeline / Technical Proof *(critical de-risk)*

Prove the look before committing to full production.
Deliverable: `output/animatic/whisperer_slice_v1.mp4` — one full breach end to end.

**Kill criterion:** if the hard cut reads as a glitch after two method iterations, stop and re-scope.
See the sub-project production plan for fallback options.

### Frame Generation — Register A (Grok CLI)
**Tool:** `GenerateImage` (text→image) via Grok CLI on SuperGrok login ($0 cost).
Do NOT set `XAI_API_KEY` — that routes to the paid API. Run from your own terminal.

#### Prompt Architecture (Register A frame)
```
[GLOBAL STYLE LOCK]
Stop-motion puppet animation. Photorealistic miniature. Practical lighting.
Film grain ISO 3200. Lens: 50mm macro equivalent.

[CHARACTER LOCK — injected from character-locks.json]
{character_description}

[SHOT DIRECTION — from prompts.csv]
{enriched_shot_description}

[HORROR MODIFIER — from tone guide]
{tone_palette_string}
```

#### Frame Consistency Strategy
Grok has no memory between calls. Identity consistency comes entirely from the cast lock text.
1. Lock descriptions must name specific, *unusual* visual details (a cracked left eye socket,
   specific rust stain pattern) — generic descriptions drift.
2. `src/regenerate-canonical.py` — generates canonical reference frames for QA comparison.
3. Batch frames for the same character in the same session window when possible.

#### Running Frame Generation
```powershell
.\scripts\run-frames.ps1 -Section I -Limit 100     # up to 100 frames for Section I
.\scripts\run-frames.ps1 -Limit 30                  # next 30 needed
.\scripts\run-frames.ps1 -Of 3 -Shard 0 -Limit 50  # parallel shard
```
Output: `generated/frames-generated/<frame_id>.png`. Resumable: skips existing.

### Frame Generation — Register B (Hyperreal)
Register B is NOT generated by Grok GenerateImage. It requires a two-step process:
1. **Source image** — a high-resolution photoreal image generated by a capable model.
2. **Image-to-video** — the source image becomes a barely-moving clip for live holds, or a static
   shot for the final image.

Register B plays **smooth** (no decimation). The smoothness itself is the grammar.
Live holds (Register A technique): image-to-video → **decimated to 8 fps** to stay in mortal register.

---

## Phase 3B — Animatic & Locked Edit *(gate before any animation)*

Cut storyboards/rough frames to the script and lock timing. Production animates **to this**.
Stop-motion has near-zero coverage — discover pacing/timing problems here, not on the table.

Deliverable: `output/animatic/whisperer_animatic_v1.mp4` (locked timing).

---

## Phase 4 — Production

Build and capture, animating to the locked animatic.

### Scheduling Discipline
- Batch shots by shared set/puppet-state to avoid teardown/rebuild
- Sessions are **non-resumable** mid-shot (lighting/position continuity) — plan around complete shots
- Guard momentum/burnout on the long solo animate

---

## Phase 5 — Audio

### Narration / Dialogue (Kokoro TTS)
`scripts/build-audio.py` reads `src/dialogue.json` → synthesizes per-line WAV files →
`generated/audio/<beat_id>-<speaker>.wav`.

### Foley Design
```python
# Foley cues keyed to beat IDs in src/foley-cues.json
```
Stop-motion foley that suggests the puppet medium:
- Clay/resin creak on movement
- Wire armature tick on joint flex
- Environment: miniature room resonance (small-space reverb)

### Register B — "Clarity" Sound Design
The audio register switch mirrors the visual. In Register B moments:
- Over-clean audio — wrong room tone
- Impossible latency (sound arrives before its source)
- Absence of ambient noise that should be present

---

## Phase 6 — Post / Final Assembly / QA

```powershell
.\scripts\assemble-sequence.ps1 -Final
```

Final QA checklist (`docs/qa/final-review.md`):
- [ ] Four breaches only — no fifth
- [ ] Every A→B is a hard cut (no blends)
- [ ] Register B smoothness vs. Register A jitter reads as ontologically different
- [ ] Character identity stable across Register A sections
- [ ] Horror tone consistent (no accidental warmth/comedy frames)
- [ ] Sound design of "clarity" lands on Register B beats
- [ ] Narration timing matches intended beats
- [ ] HDR/legal-levels on the final image (no illegal clipping)
- [ ] Caption track present
- [ ] No text artifacts in Register A frames

---

## Phase 7 — Festival / Release / Scale

See sub-project plan for premiere vs. post decision, AI-method framing, and festival strategy.

---

## Project Bootstrap

When starting a new project:
```
1. mkdir "Stopmotion-AI/<Project Name>"
2. git init "Stopmotion-AI/<Project Name>"
3. gh repo create EE-EDK/<project-slug> --private
4. Copy structure from Whisperer in the Wire/ as template
5. Edit src/production-data.py + src/character-locks.json with new project data
6. Run: python src/generate-shot-bible.py   ← verify output before any generation
7. From Stopmotion-AI/: git submodule add https://github.com/EE-EDK/<project-slug>.git "<Project Name>"
```
