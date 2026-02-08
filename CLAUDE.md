# Finance Project — Developer Guide

## Quick Start (both servers must run)

```bash
# One command — launches both servers in separate windows
start.bat
```

Or manually in two terminals:

```bash
# Terminal 1 — Backend (must start FIRST, run from project root)
.venv/Scripts/python.exe -m uvicorn backend.main:app --reload --reload-dir backend --port 8000

# Terminal 2 — Frontend (only after backend is up)
cd frontend
npm run dev
```

The Vite dev server on :3000 proxies `/api/*` to the FastAPI backend on :8000.
**If the backend is not running, every `/api` request will fail with ECONNREFUSED.**

## Before Changing API Endpoints

1. Every route under `/api/macro/` **must return HTTP 200 even when the database is empty** (return `[]` or a zero-value response, never 500).
2. After adding/modifying any backend route, run the endpoint smoke tests:
   ```bash
   .venv/Scripts/python.exe -m pytest tests/test_api.py -v
   ```
3. The frontend axios client (`frontend/src/api/client.ts`) sets `baseURL: "/api"` — all frontend API functions use paths like `/macro/indicators`, which become `/api/macro/indicators` via the Vite proxy. Never use absolute URLs in frontend API calls.

## Testing

```bash
# Run all tests
.venv/Scripts/python.exe -m pytest tests/ -v

# Run only API endpoint tests
.venv/Scripts/python.exe -m pytest tests/test_api.py -v

# Run data pipeline tests
.venv/Scripts/python.exe -m pytest tests/test_data_pipeline.py -v
```

## Common Pitfalls

- **ECONNREFUSED on `/api/*`**: Backend is not running. Start it before the frontend.
- **Empty responses from macro endpoints**: The DB has no observations yet. Run the FRED ingestion or start the backend (it auto-syncs on startup).
- **Vite proxy config** is in `frontend/vite.config.ts` under `server.proxy`. The target must match the backend port (default 8000).
- **FRED_API_KEY** must be set in `.env` for data ingestion to work.
- **uvicorn --reload restarts on CSV writes**: Always use `--reload-dir backend` so the file watcher only watches Python source, not `data/raw/` CSVs written during FRED sync.

## Architecture

- Backend: FastAPI + SQLAlchemy + SQLite (`data/finance.db`)
- Frontend: React 18 + TypeScript + Vite + Tailwind CSS 4
- API proxy: Vite `:3000` → FastAPI `:8000` (path prefix `/api`)
- All API routes are mounted under `/api` in `backend/main.py`
- Macro routes: `/api/macro/indicators`, `/api/macro/catalog`, `/api/macro/yield-curve`, `/api/macro/recession-risk`, `/api/macro/series/{id}`, POST `/api/macro/multi-series`, POST `/api/macro/correlation`
