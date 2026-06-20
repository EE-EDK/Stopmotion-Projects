# Stop-Motion Photorealism Horror — Production Pipeline

Adapted from *The Void is Crimson* AI video pipeline.
Key difference: every frame is a **discrete still** rendered by Grok; the stop-motion
aesthetic is baked into the prompts and timing metadata, not post-processed.

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

## Phase 1 — Script + Shot Bible

### 1-A Screenplay
Write the screenplay as an HTML doc (`docs/screenplay/<title>.html`) — browsable in a
Grok/Claude review session. Include:
- Beat markers tying to dialogue JSON
- ON-SCREEN vs. OFF-SCREEN voice tags (Grok only lip-syncs what it sees)

### 1-B Shot Bible Generator
`src/generate-shot-bible.py` is the **single source of truth generator**. It reads:
- `src/production-data.py` — per-beat metadata (location, characters, emotion, camera)
- `src/character-locks.json` — cast descriptions (injected into every frame prompt)
- `src/enriched-descriptions.json` — per-shot flavor text (overrides defaults)
- `src/dialogue.json` — beat → [{speaker, line, direction, timing}]

It emits:
- `output/<title>-shot-bible.json` — master manifest
- `output/<title>-prompts.csv` — one row per frame with full Grok prompt
- `output/<title>-storyboard.html` — browsable storyboard

**Regenerate after every edit** to production data — never hand-edit the output files.

### 1-C Frame Rate Planning
Stop-motion timing is not linear. Document in `docs/specs/timing.md`:
```
target_fps = 12          # or 8 for more staccato horror feel
hold_frames = {
  "idle": 2,             # character stationary — hold same frame N times
  "action": 1,           # full-motion — new frame every tick
  "impact": 3,           # freeze on a hit/scare
}
```
The shot bible generator uses these to expand beats into discrete frame slots.
Total frame count = sum(beat_duration_sec × fps / hold_multiplier).

---

## Phase 2 — Frame Generation (Grok CLI)

**Tool:** `GenerateImage` (text→image) via Grok CLI on SuperGrok login ($0 cost).
Do NOT set `XAI_API_KEY` — that routes to the paid API. Run from your own terminal.

### Prompt Architecture (per frame)
```
[GLOBAL STYLE LOCK]
Stop-motion puppet animation. Photorealistic miniature. Practical lighting.
Film grain ISO 3200. Lens: 50mm macro equivalent.

[CHARACTER LOCK — injected from character-locks.json]
{character_description}

[SHOT DIRECTION — from prompts.csv column "prompt"]
{enriched_shot_description}

[HORROR MODIFIER — from tone guide]
{tone_palette_string}
```

### Frame Consistency Strategy
Grok has no memory between calls. Identity consistency comes entirely from the cast lock text.
To preserve continuity:
1. Lock descriptions must name specific, *unusual* visual details (a cracked left eye socket,
   specific rust stain pattern) — generic descriptions drift.
2. `src/regenerate-canonical.py` — generates 40 canonical reference frames for QA comparison.
   Run after every lock text change.
3. Batch frames for the same character in the same session window when possible.

### Running Frame Generation
```powershell
.\scripts\run-frames.ps1 -Section I -Limit 100     # generate up to 100 frames for Section I
.\scripts\run-frames.ps1 -Limit 30                  # next 30 needed (any section)
.\scripts\run-frames.ps1 -Of 3 -Shard 0 -Limit 50  # parallel shard (3 windows)
```
Output: `generated/frames-generated/<frame_id>.png`
Resumable: skips frames already present.

### Inter-Frame Drift Suppression
The biggest stop-motion AI problem: frames of the same character look like different puppets.
Mitigation stack (in order of effectiveness):
1. **Specificity in lock text** — the primary lever
2. **Canonical anchors** — QA each batch against `refs/character-locks/` contact sheet
3. **Prompt recycling** — identical prompt structure for frames in the same beat; only
   motion/expression changes
4. **Manual curation** — regenerate outlier frames individually (`-Mode gen` suffix variant)

---

## Phase 3 — Frame QA

Run after each batch. Do not proceed to Phase 4 until QA passes a section.

### 3-A Character Consistency Check
Open `ui/contact-sheet.html` (generated by shot bible script). Compare:
- Character face/body against `refs/character-locks/<character>-lock.png`
- Flag frames where a character has >2 identity deviations (different hair, wrong hand count, etc.)

### 3-B Horror Tone Check
Per frame, ask: does it read as oppressive / uncanny / dreadful — or just "dark scene"?
Reject frames that are merely dark without menace. Note in `docs/qa/batch-<N>-notes.md`.

### 3-C Motion Path Review
Lay flagged frames in sequence in any image viewer. Does the pose progression tell the motion?
Stop-motion works by implying motion between frames — even with AI generation, the sequence
must read as intentional movement, not random drift.

### 3-D Prompt Hazard Check
Known Grok failure modes (from `docs/specs/prompt-hazards.md`):
- Literal red "laser-line" artifacts across eyes when describing eye effects
- Ghost/translucent characters rendered solid
- Invented background text (signs, labels) — prohibit with "no text, no signage" in global lock
- Off-screen characters getting lip-synced onto visible faces (Phase 4 issue, not Phase 2)

---

## Phase 4 — Animation Assembly

### 4-A Sequence → Video
```powershell
.\scripts\assemble-sequence.ps1 -Section I       # FFmpeg concat for one section
.\scripts\assemble-sequence.ps1 -WithTimings     # uses per-frame hold counts from shot bible
```
The script reads `output/<title>-shot-bible.json` for hold counts.
Output: `output/sections/S<N>.mp4` (silent).

### 4-B Color Grading
After assembly, apply LUT or FFmpeg filter chain from `docs/specs/grade-spec.md`.
Minimal target: desaturate 30%, add grain, boost shadow contrast.
```powershell
.\scripts\grade.ps1 -Input output/sections/S1.mp4 -Lut refs/luts/horror-v1.cube
```

---

## Phase 5 — Audio

### 5-A Narration / Dialogue (Kokoro TTS)
`scripts/build-audio.py` reads `src/dialogue.json` → synthesizes per-line WAV files →
`generated/audio/<beat_id>-<speaker>.wav`.
Narrator voice: low, grave, mysterious (LotR-prologue reference). Tune voice blend in script.

### 5-B Foley Design
Stop-motion needs foley that suggests the puppet medium:
- Clay/resin creak on movement
- Wire armature tick on joint flex
- Environment: miniature room resonance (slightly hollow, small-space reverb)
Store reference timing in `src/foley-cues.json` keyed to beat IDs.

### 5-C Mix + Mux
```powershell
.\scripts\build-audio.py --mix-section I        # assemble per-section mix
.\scripts\assemble-sequence.ps1 -WithAudio -Section I   # mux into video
```

---

## Phase 6 — Final Assembly + QA

```powershell
.\scripts\assemble-sequence.ps1 -Final          # concat all sections → output/<title>-final.mp4
```

Final QA checklist (`docs/qa/final-review.md`):
- [ ] Character identity stable across all sections
- [ ] Horror tone consistent (no accidental warmth/comedy frames)
- [ ] Narration timing matches intended beats
- [ ] Foley synchronized to visible movement
- [ ] No text artifacts in frame
- [ ] Motion path reads as intentional stop-motion, not random drift

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
