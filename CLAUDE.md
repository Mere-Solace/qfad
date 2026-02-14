# Finance — Script-Based Analysis Toolset

## What This Is
A collection of Python scripts, Jupyter notebooks, and a master Excel workbook for macro-financial analysis. Data comes from FRED (Federal Reserve Economic Data). No web server, no database — just files.

## Project Structure
```
config/series.json       — FRED series definitions (~45 series across 9 categories)
lib/fred_client.py       — FRED API wrapper (uses fredapi + python-dotenv)
lib/csv_store.py         — CSV read/write/append utilities for per-series files
scripts/pull_fred.py     — Fetch FRED data → CSVs, then rebuild Excel workbook
scripts/build_master_sheet.py — Build/update master_workbook.xlsx from CSVs
scripts/*.py             — Analysis stubs (recession, yield curve, correlation)
notebooks/               — Jupyter notebooks for exploration
r_analysis/              — R scripts
data/raw/fred/           — Per-series CSV files (date, value)
data/master_workbook.xlsx — Master Excel workbook with all data aligned
output/                  — Analysis outputs (plots, exports)
```

## Common Commands
```bash
python scripts/pull_fred.py          # Incremental FRED data pull + rebuild workbook
python scripts/pull_fred.py --full   # Full re-sync from scratch
python scripts/build_master_sheet.py # Rebuild workbook only (no API calls)
```

## How Data Flows
1. `pull_fred.py` reads `config/series.json` for the list of FRED series
2. For each series, checks `data/raw/fred/{id}.csv` for the last date
3. Fetches only newer observations from FRED API (incremental)
4. Appends new rows to CSV files
5. Calls `build_master_sheet.py` to rebuild the Excel workbook

## Master Workbook Strategy
- "All Data" sheet: every series outer-joined on date, forward-filled
- Per-category sheets: Treasury Rates, Spreads, Inflation, etc.
- "Metadata" sheet: series info, last date, row count
- User-created sheets (Charts, Analysis, etc.) are preserved on rebuild

## Adding a New FRED Series
1. Find the series ID on https://fred.stlouisfed.org
2. Add it to the appropriate category in `config/series.json`
3. Run `python scripts/pull_fred.py`

## Environment
- Python 3.11+
- `FRED_API_KEY` in `.env` (get one at https://fred.stlouisfed.org/docs/api/api_key.html)
  - The `.env` file **must** be saved with UTF-8 or ASCII encoding, otherwise the key cannot be parsed.
- Install deps: `pip install -e .` or `pip install -e ".[dev]"`
