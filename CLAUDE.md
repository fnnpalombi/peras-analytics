# CLAUDE.md — Peras Analytics

Context for Claude Code working in this repo. Read this before proposing changes.

## What this project is
EDA of **Peras**, an AI startup that generates custom textbooks and education tools.
Dual purpose, from the same work:
1. **For Peras:** clean their PostHog data, run EDA across the acquisition → activation
   → engagement → retention funnel, review experiment rigor, hand back action items.
2. **For the owner:** a **public portfolio piece** showing EDA in Python + SQL and
   visualization in Tableau.

The governing tension: the portfolio must be public, but Peras's data must never be
exposed unsafely. Everything is built around a **two-tier** model — private raw vs.
public aggregated/cleared.

## Current state
- Repo scaffolded; `notebooks/01_extract.ipynb` (reproducible PostHog HogQL pull) is built.
- **Next up:** `02_clean_audit.ipynb` — dedupe, nulls/types, bot/internal traffic,
  session definition, and a short data-quality report.
- Not started: `03_eda.ipynb`, Tableau dashboards, findings memo.

## Non-negotiable safety rules (do not violate)
- **Never commit `data/raw/`, `.env`, or any user-level data.** These are gitignored;
  keep them that way. Raw data is the private tier and never gets published.
- **Never hardcode secrets.** The PostHog API key lives only in a gitignored `.env`,
  read via `python-dotenv`. Don't print it, don't write it to a committed file.
  (Recommend adding a Claude Code Read deny-rule for `.env` so it never enters context.)
- **Only `data/aggregated/` is publishable** — and only after it's anonymized,
  small-cell-suppressed (k-anonymity, drop/bucket groups under ~5 users), and cleared.
- **Index or relativize sensitive business metrics** (revenue, absolute counts) in any
  public artifact — show trends/indexed values, not Peras's real figures.
- **Nothing goes public without written clearance from Peras.** Repo stays private
  until then; Tableau Public is public by default, so treat it accordingly.

## Toolchain (agreed decisions)
- **Extraction:** PostHog Query API (HogQL) called from Python — reproducible, not
  manual CSV. Land raw as parquet in `data/raw/`.
- **SQL:** DuckDB locally over the extracts, and/or HogQL in PostHog. Show SQL
  deliberately — use it for heavy aggregation/joins/funnels.
- **Python:** pandas/numpy for wrangling; scipy/statsmodels for experiment stats.
- **Viz:** matplotlib/seaborn in-notebook; Tableau (Public) for the portfolio dashboards.

## Conventions
- **Split SQL and Python on purpose** so both skills are visible — don't let one do
  everything by accident.
- **Narrate notebooks.** Markdown context around every analytical step; this is a
  portfolio artifact, so legibility matters.
- **Keep the extract re-runnable** end to end (one script/notebook refreshes it).
- **Every finding earns a "so what"** — end analysis in an action item.
- Run "Restart & Run All" before committing a notebook so outputs render cleanly on
  GitHub; make sure no cell output prints real user-level rows before committing.

## Repo layout
```
data/raw/         # gitignored — private tier, never committed
data/aggregated/  # cleared, publish-safe extracts
notebooks/        # 01_extract, 02_clean_audit, 03_eda
sql/              # DuckDB / HogQL queries
reports/          # data_quality.md, findings_memo.md
tableau/          # workbook + Tableau Public link
```

## Setup
- `cp .env.example .env`, fill in PostHog key / host / project ID.
- `pip install requests pandas pyarrow python-dotenv duckdb jupyter`
- Run notebooks in order.

See @README.md for the public-facing project overview.
