# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository nature

This is a **research/documentation repository**, not a codebase — there is no application code, build system, package manifest, linter, or test suite. All content is Markdown. Do not assume or invent build/lint/test commands; if asked to run one, say none exists.

## Purpose

Research notes and an implementation-ready blueprint for a **monthly liquidity dashboard** that monitors AI-bubble risk through three liquidity layers (policy / market credit / speculative) plus an AI-capex-to-EPS bridge, per the source research report. The dashboard is not yet implemented — this repo is the spec stage.

## Structure and content flow

```
docs_references/   → source research (read-only evidence, do not edit conclusions)
  美股AI泡沫-流動性槓桿診斷與破裂點推演.md              full source report
  美股AI泡沫-流動性槓桿診斷與破裂點推演-分析報告.md      structured summary of the above

drafts/             → derived design artifacts (the actual working documents)
  DESIGN.md                          product/design-contract shorthand (goals, IA, components, a11y, states)
  月度流動性儀表板設計圖.md            the authoritative implementation blueprint (see below)
  資料抓取與查證工作流.md              data-acquisition SOP: per-indicator fetch mode (auto/semi/manual),
                                     source-tier handling rules, and the monthly verification checklist
                                     (operationalizes the source report's Appendix B/C; blueprint §9 defines
                                     what to fetch, this SOP defines how to fetch and verify)
```

`docs_references/` is upstream evidence; `drafts/` is downstream synthesis. When updating design decisions, edit `drafts/`, not `docs_references/`. If new source material arrives, add it under `docs_references/` and cite it from `drafts/`.

## The blueprint (`drafts/月度流動性儀表板設計圖.md`) is the single source of truth for:

- **9 core indicators** with id, layer, bear/bull thresholds, and weight (weights sum to 100) — §5.1. Thresholds exist in dual form: human-readable `bear_threshold`/`bull_threshold` (display only) and machine-readable `bear_rule`/`bull_rule` (authoritative for implementation), with `threshold_version` bumped on any change.
- **Scoring rules** — §6: per-indicator `signal_state` → score (`bull_invalidated`=0, `neutral`=50, `bear_watch`=75, `bear_confirmed`=100, `stale`/`missing`=excluded). Key derived rules: "sustained" = two consecutive month-end triggers; `bear_watch` = within 10% of the bear threshold or a justified analyst note; any stale/missing core indicator marks the global score `provisional`; `valid_weight_coverage` < 50% suppresses the global state entirely.
- **Layer scores** — §6.2: three liquidity layers **plus** the AI EPS bridge (`ndx_forward_eps` is its own bucket, not part of the three layers, but its weight counts toward the global score). §6.2 is the authoritative layer→indicator mapping.
- **Scenario engine** — §7: Bear/Base/Bull driven by explicit condition counts, not a single score. Scenarios may share `watch` but `active` is mutually exclusive (simultaneous Bear+Bull active forces both to `watch` + manual review); `stale`/`missing` indicators count as "condition not met".
- **Freshness rules** — §9: fresh/stale/missing definitions with concrete lag limits (daily >7d, weekly >14d, monthly >45d → stale); carried-forward values are always `stale`, never `fresh`.
- **Data model** — §8: JSON shapes for `indicator_definitions`, `monthly_indicator_values`, `monthly_regime_summary` (includes `score_quality` and `valid_weight_coverage`).
- **Monthly report export format** — §11
- **Acceptance criteria** for a first version — §13
- **Open questions** (implementation platform, data licensing/fetch mode, export format, scenario-probability display, update ownership) are explicitly unresolved — §14 is the single authoritative list; DESIGN.md only points to it

`DESIGN.md` restates goals/IA/components/accessibility/interaction-states in a shorter product-design-contract shape; treat it as a companion index into the blueprint, not a competing spec. Both files list `status: Draft` — reconcile any conflict between them by favoring the blueprint's §5–§8 (indicators/scoring/scenarios are the load-bearing detail) and flag the discrepancy rather than silently picking one.

## Content conventions (preserve these when editing)

- Language: Traditional Chinese (繁體中文). Do not switch to Simplified Chinese or English prose in these documents.
- Tone: 冷靜、研究導向、可稽核、避免交易喊單語氣 — analytical, hedged, auditable; never phrase output as investment advice or a confident price/timing prediction (see DESIGN.md "Brand" and "Content voice").
- Every threshold/signal claim must be traceable to a source indicator, formula, and data-as-of date — this is a hard constraint from the design principles (DESIGN.md §Design principles, blueprint §1/§10.3), not a style preference.
- `stale`/`missing` data must never be silently coerced into `neutral` — this is an explicit acceptance criterion (blueprint §13).
- State naming is fixed to six values — `bear_confirmed`, `bear_watch`, `neutral`, `bull_invalidated`, `stale`, `missing` (blueprint §6.1/§10.1) — do not introduce aliases like "normal"/"warning" in either document.
- Numeric defaults added as research assumptions (bear_watch 10% proximity, 50% coverage cutoff, freshness lag days, sustain = 2 months) are marked for Phase 3 calibration — when changing them, bump `threshold_version` and keep the old definition for backtest comparison.

## When asked to implement the dashboard

The blueprint's §14 open questions (platform, data source licensing, scenario-probability display, update ownership) are unresolved. Do not pick a framework/platform unilaterally — surface these as decisions for the user, per Phase 1 in blueprint §12 (local MVP: CSV/JSON input, 9 indicators, overview + indicator table + Markdown report export).
