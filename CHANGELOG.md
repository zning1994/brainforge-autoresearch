# Changelog

All notable changes to this project will be documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] - 2026-03-24

### Added

- Core `autoresearch.py` script — autonomous skill prompt optimizer
  - Zero external dependencies (Python 3.9+ stdlib only)
  - LLM providers: MiniMax, OpenAI, Anthropic (auto-detect from env vars)
  - Custom endpoint support via `OPENAI_BASE_URL`
  - `--model` flag to override default model per provider
- Hybrid eval system
  - Rule-based evals: `regex`, `banned_phrases`, `word_count`, `contains`, `not_contains`
  - `contains` supports `match: "all"` and `match: "any"` modes
  - LLM-as-judge evals with binary YES/NO scoring
  - CJK-aware word counting for Chinese/Japanese/Korean text
- Experiment loop
  - Baseline measurement before any changes
  - One mutation per experiment (targeted, not bulk rewrites)
  - Automatic keep/discard based on score comparison
  - Convergence exit: 95%+ pass rate for 3 consecutive experiments
  - Budget cap via `--max-experiments`
- Output artifacts
  - `results.tsv` — tab-separated score log
  - `changelog.md` — detailed mutation history with reasoning
  - `results.json` — structured data for tooling
  - `dashboard.html` — self-contained Chart.js dashboard (opt-in via `--dashboard`)
  - `SKILL.md.baseline` — original skill backup
- `SKILL.md` — agent instructions for OpenClaw / Claude Code / Cursor / Cline
- `eval-guide.md` — practical guide for writing binary eval criteria
- Example eval configs: `weekly-report.json`, `search-skill.json`
- Compatible with `npx skills add` (Vercel Labs skills ecosystem)

[0.1.0]: https://github.com/zning1994/openclaw-autoresearch/releases/tag/v0.1.0
