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
- All three notebooks built and working: `01_extract.ipynb` (batched HogQL pull,
  explicit `LIMIT`s), `02_clean_audit.ipynb` (dedupe, bot/traffic filtering, session
  definition, data-quality report), `03_eda.ipynb` (acquisition, signup, activation,
  engagement, retention, experiments, small-cell-suppressed aggregated output).
- Repo is public on GitHub with Peras's written clearance. First commit pushed.
- **Next up:** Tableau dashboards on `data/aggregated/`, then `reports/findings_memo.md`.

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

## Pre-commit safety checklist (run every time, before every commit)

Gitignore protects data *files* — it does not protect notebook *outputs*. A `.ipynb`
is committed with every saved cell output embedded as JSON, so anything printed or
displayed in a notebook is subject to the same public-safety bar as
`data/aggregated/`, even though the notebook file itself isn't gitignored. Found and
fixed real instances of every issue below in this repo's own history — this isn't
hypothetical.

1. **Dump every saved output in every notebook, not just the last displayed
   expression.** A cell can have several `print()`/`display()` calls; only checking
   the final auto-rendered output misses earlier ones. Look for raw person-level
   values (`person_id`, `distinct_id`, `account_id`, URLs, timestamps tied to an
   individual) — not just property *names*, actual *values*.
2. **Small-cell suppression must gate every printed count or rate, not just
   tables.** It's not enough to suppress a distribution/table — a narrative
   "headline" print (e.g. `f"Activated: {n} of {total} people"`) placed *before* the
   threshold check leaks the exact small count regardless of what happens below it.
3. **A "suppressed" fallback message must never contain the exact small count.**
   `"Only 3 people activated"` defeats the point — say `"Fewer than 5 people
   activated"` instead, with no number in it at all.
4. **Treat 0 as always safe to show; treat 1 through (threshold − 1) as always
   hidden, in any form.** This includes rates: a rate computed from a known
   denominator and a small numerator (e.g. "2.6% of 116 activated") can be
   back-calculated to reveal the exact small count, so gate the numerator's size,
   not just whether the rate itself "looks fine."
5. **Confirm `data/aggregated/*.csv` and `reports/*.md` were regenerated from the
   current, correct run before committing.** Stale outputs from an earlier or
   buggy pipeline run (wrong event definitions, an unfixed extraction bug, a
   partial re-run) are a real trust problem for a public repo, not just cosmetic —
   check the numbers match what the notebook just produced, not an old cached run.
6. **Run `git add -A --dry-run` (or `git status`) and read the file list before
   staging anything.** Confirm `.env` and everything under `data/raw/` are absent.
   Don't assume `.gitignore` is doing its job — check it every time.

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
