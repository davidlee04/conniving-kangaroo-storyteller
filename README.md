# ClueLLM

**Team:** Conniving Kangaroo &nbsp;|&nbsp; **Course:** CS 7634 (Georgia Tech)
**System name:** ClueLLM
**Phase II project template:** Template 1 — Search-Based Drama Management
**Members:** Devam Shrivastava, Chaeyoung Lee, David Lee

---

## What this is

ClueLLM turns a Phase I generated crime mystery into a **playable, text-based interactive
story** governed by a **search-based drama manager (DM)**. The DM treats the mystery
as an abstract plot graph, watches what the player is doing each turn, and chooses an
intervention (hint, glow, temporary denial, soft cause, or no-op) by running short
sampling-based rollouts and scoring the resulting future states for
**solvability, pacing, suspense, and (negative) manipulation** — the four success
criteria from the project specification.

The full pipeline lives in **`main.ipynb`**; the notebook reads top-to-bottom with
markdown headings for every step.

```
┌────────┐  NL action   ┌──────────────────────┐  fired nodes  ┌──────────────────┐
│ Player ├─────────────▶│ Game Engine          ├──────────────▶│ Plot Graph (DAG) │
└────────┘              │ (validate + step     │               │ pre/effect preds │
                        │  trigger plot graph) │               └──────────────────┘
                        └─────────┬────────────┘
                                  │ world state
                                  ▼
                        ┌──────────────────────┐
                        │ Drama Manager        │  ← scores: solvability, pacing,
                        │ (rollouts + score)   │            suspense, anti-manipulation
                        └─────────┬────────────┘
                                  │ chosen intervention
                                  ▼
                        ┌──────────────────────┐
                        │ Renderer (LLM prose) │ → narration shown to player
                        └──────────────────────┘
```

---

## How to run (instructional team)

### 0. Prerequisites

- Python 3.9 or newer
- Jupyter Notebook / JupyterLab
- A Google Gemini API key on a paid tier (the free tier's 20 requests/day quota is
  not enough for a full Phase II run)

### 1. Install dependencies

The first code cell of the notebook runs `pip install -q -U google-genai`, so no
manual install is needed. If you want to install ahead of time:

```bash
pip install -U google-genai jupyter
```

### 2. Provide your API key (no key is hardcoded)

The notebook reads `GOOGLE_API_KEY` from your environment first; if it's missing, it
prompts via `getpass`. Choose **either**:

```bash
export GOOGLE_API_KEY="paste-your-key-here"
jupyter notebook main.ipynb
```

— **or** just launch the notebook and paste your key when the prompt appears.

### 3. Run all cells

`Kernel → Restart & Run All`. Cells are arranged so that running them in order
populates a single `StoryWorld` object that flows through the whole pipeline.

The interactive `play(world)` call at the bottom of Phase II is **commented out** by
default so the notebook can run unattended. Uncomment it if you want to play the
mystery yourself.

---

## Expected runtime and cost

Times measured on a typical laptop with `gemini-2.5-flash`:

| Section                                       | Wall-clock      |
| --------------------------------------------- | --------------- |
| Phase I — crime, characters, clues, countdown | ~30 sec         |
| Phase I — 15 plot points + arc validation     | ~3–5 min        |
| Phase I — consistency check + final narrative | ~30 sec         |
| Phase II — plot-graph extraction              | ~5 sec          |
| Phase II — game-world generation              | ~10 sec         |
| Phase II — eval harness (4 auto-play runs)    | ~30–90 sec      |
| Phase II — verbose demo + DM log dump         | ~20 sec         |
| **Total (no interactive play)**               | **~5–8 min**    |

API cost:
- **~$0.15 – $0.25** per full notebook run on `gemini-2.5-flash`
- Interactive `play()` adds ~$0.01 per turn (one parse call + one render call)

---

## Code map (architecture ↔ cells)

| Architecture component                          | Where it lives in the notebook         |
| ------------------------------------------------ | -------------------------------------- |
| `StoryWorld`, `CrimeDetails`, `Character`, `Clue`, `PlotPoint` dataclasses | Data Structures section, Phase I |
| Crime + character + clue + countdown generation  | Phase I sections                       |
| 15-step iterative meta-controller                | "Phase 3: Iterative Loop / Meta Controller" |
| Consistency check + final-narrative assembly     | "Phase 4: Validation + Final Narrative Assembly" |
| **Phase II — Plot graph (`PlotNode`, extractor)**| Step 1                                 |
| **Phase II — Game world (`Location`, `Item`, `GameWorld`)** | Step 2                       |
| **Phase II — `WorldState`, predicate / effect functions** | Step 3                          |
| **Phase II — `Action`, NL action parser**        | Step 4                                 |
| **Phase II — `GameEngine` (validate, apply, trigger)** | Step 5                           |
| **Phase II — Drama Manager (`Intervention`, `DMTrace`, rollouts, scoring, `decide`)** | Step 6 |
| **Phase II — Intervention rendering (LLM prose)** | Step 7                                |
| **Phase II — Game loop (`play`, verbose panels)** | Step 8                                |
| **Phase II — Evaluation harness + walkthroughs** | Step 9                                 |

---

## How to read the demo / video walkthrough output

The verbose evaluation cell prints, per turn:

- `[USER ACTION]` — the parsed structured action
- `[ENGINE]` — classification (`constituent` / `neutral` / `invalid` / `case_solved`)
  and which plot-graph nodes fired
- `[DRAMA MANAGER ACTION]` — top candidates considered, their rollout scores, and
  the chosen intervention with a one-line reason
- `[STATE]` — current location, clues discovered, plot-graph progress, intervention budget
- `[NARRATION]` — the prose the player would see

A full structured trace is also written to `dm_log.json` for the final video.

---

## Files in this repository

| File | Purpose |
|---|---|
| `main.ipynb` | The complete Phase I + Phase II system. |
| `README.md` | This file. |
| `Example Story.txt` | Final narrative produced by Phase I. |
| `Example Story Plot Points.txt` | The 15 structured plot points behind that story. |
| `Example Story Annotated.md` | The narrative with each plot point inlined. |
| `Phase II Projects for preview.pdf` | Course-provided Phase II spec. |
| `Phase2AISVW.key` | Our team's Phase II proposal slides. |

---

## API-key handling note (for graders)

Per the rubric guidance, **no API key is hardcoded**. The notebook reads `GOOGLE_API_KEY`
from the environment first and falls back to a `getpass` prompt. The only code change
required to grade the system is to set `GOOGLE_API_KEY` either as an env var or via the
prompt.
