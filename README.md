# Peras Analytics — Product & Marketing EDA

> Exploratory data analysis of an AI education startup's product and marketing data,
> built in Python + SQL with a Tableau dashboard. Public portfolio version uses
> aggregated / indexed figures only; raw data is private and never committed.

**Status:** in progress · **Stack:** PostHog (HogQL) · Python (pandas) · DuckDB · Tableau

## The problem
Peras generates custom AI textbooks and runs frequent feature and marketing
experiments. This project inspects and cleans their PostHog event data, runs EDA
across the acquisition → activation → engagement → retention funnel, reviews
experiment rigor, and turns the findings into action items and visualizations.

## Approach
1. **Extract** — reproducible pull from PostHog via the HogQL Query API (`notebooks/01_extract.ipynb`)
2. **Clean & audit** — data-quality pass + report (`notebooks/02_clean_audit.ipynb`)
3. **EDA** — funnels, retention cohorts, segmentation, experiment analysis (`notebooks/03_eda.ipynb`)
4. **Visualize** — Tableau dashboards on aggregated extracts (`tableau/`)
5. **Report** — findings + recommendations (`reports/findings_memo.md`)

## Headline findings
_TBD — filled in after analysis._

## Data safety
Raw, user-level data lives in `data/raw/` and is **gitignored**. Only aggregated,
anonymized, small-cell-suppressed extracts in `data/aggregated/` are eligible to be
committed, and only after written clearance from Peras. Sensitive business metrics
are shown as indexed / relative values in any public artifact.

## Repo layout
```
data/raw/         # gitignored — private tier
data/aggregated/  # cleared, publish-safe extracts
notebooks/        # 01_extract, 02_clean_audit, 03_eda
sql/              # DuckDB / HogQL queries
reports/          # data_quality.md, findings_memo.md
tableau/          # workbook + Tableau Public link
```

## Reproduce it
1. `cp .env.example .env` and fill in your PostHog key, host, and project ID
2. `pip install requests pandas pyarrow python-dotenv duckdb jupyter`
3. Run the notebooks in order.
