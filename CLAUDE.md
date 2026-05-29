# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **harness engineering tutorial** — "Learn Claude Code." It teaches how to build the operational environment (harness) around an LLM agent model through 20 progressive lessons. Each lesson adds one mechanism to a core agent loop. The project is trilingual (Chinese source, English/Japanese translations).

The central thesis: agency comes from model training, not code orchestration. The harness provides tools, knowledge, context management, and permission boundaries — the model decides and acts.

## Commands

### Python (agent lessons)
```sh
pip install -r requirements.txt          # anthropic>=0.25.0, python-dotenv>=1.0.0, pyyaml>=6.0
cp .env.example .env                     # set ANTHROPIC_API_KEY

# Run any lesson standalone
python s01_agent_loop/code.py
python s20_comprehensive/code.py

# Run a legacy agent
python agents/s_full.py
```

### Tests
```sh
python -m pytest tests/test_agents_smoke.py -q        # compile check all agents/*.py
python -m pytest tests/test_s_full_background.py -q   # unit tests for s_full background tasks
```

### Web app (Next.js)
```sh
cd web
npm install
npm run dev        # dev server at http://localhost:3000 (runs extract-content.ts first via predev)
npm run build      # type-check + build (runs prebuild extract step)
npx tsc --noEmit   # type check only
```

The `extract-content.ts` script reads root-level `s01_*` through `s20_*` folders and `agents/` + `docs/` directories, then generates `web/src/data/generated/docs.json` and `versions.json`. It runs automatically before dev/build.

## Architecture

### Two parallel tracks

- **Current track (s01-s20):** root-level folders `s01_agent_loop/` through `s20_comprehensive/`. Each contains `README.md` (Chinese narrative), `README.en.md`, `README.ja.md`, `code.py` (standalone runnable), and optional `images/`. This is the canonical version.
- **Legacy track (s01-s12):** `agents/` has runnable Python files; `docs/` has markdown docs in en/ja/zh. Kept for backward compatibility. The web app currently renders this track.

Chapter numbering does **not** map 1:1 between tracks (see README for the mapping table).

### Agent code pattern

Every lesson's `code.py` is self-contained — no imports between chapters. They all share the same core structure:

```python
def agent_loop(messages):
    while True:
        response = client.messages.create(model=MODEL, tools=TOOLS, messages=messages)
        if response.stop_reason != "tool_use":
            return
        # execute tools, append results, loop
```

Each chapter layers one mechanism on top: tools (s02), permissions (s03), hooks (s04), TodoWrite (s05), subagents (s06), skill loading (s07), context compaction (s08), memory (s09), etc.

### Web app structure

- **Framework:** Next.js 16 + React 19 + Tailwind CSS 4 + TypeScript
- **i18n:** Three locales (en, zh, ja) via `web/src/i18n/messages/`
- **Content pipeline:** `web/scripts/extract-content.ts` reads lesson folders → generates JSON into `web/src/data/generated/`
- **Data files:** Annotations (`web/src/data/annotations/sXX.json`), scenarios (`web/src/data/scenarios/sXX.json`), execution flows (`web/src/data/execution-flows.ts`)
- **Key components:** Visualizations in `web/src/components/visualizations/`, agent loop simulator in `web/src/components/simulator/`, doc renderer in `web/src/components/docs/`
- **Config:** `web/src/lib/constants.ts` has version metadata, ordering, and learning path data

### Skills directory

`skills/` contains skill definition files (SKILL.md + reference scripts) used by the s07 skill-loading lesson. Not a build artifact — referenced by agent code at runtime.

## Conventions

- Each `code.py` is standalone runnable and requires only `ANTHROPIC_API_KEY` in `.env`
- The `.env` file supports Anthropic-compatible providers (GLM, MiniMax, DeepSeek, Kimi) via `ANTHROPIC_BASE_URL` + `MODEL_ID`
- Runtime artifacts (`.memory/`, `.tasks/`, `.teams/`, `.mailboxes/`, `.worktrees/`) are gitignored and created by agent code during execution
- SVG diagrams live in each chapter's `images/` folder with locale variants (`.en.svg`, `.ja.svg`, base = Chinese)
