# FinAlly — AI Trading Workstation

A dark, data-dense trading terminal with live streaming prices, a simulated $10k portfolio,
and an AI copilot that can analyze positions and execute trades from natural language.

Built entirely by coding agents as the capstone project for an agentic AI coding course.
The full specification lives in [`planning/PLAN.md`](planning/PLAN.md).

## Features

- **Live prices** streamed over SSE, with green/red flash on every tick
- **Simulated portfolio** — market orders, instant fills, no fees
- **Visualizations** — position heatmap, P&L chart, positions table, trade blotter
- **AI chat** — suggests and auto-executes trades and watchlist changes
- **Free to run** — a free OpenRouter model and a built-in market simulator, no paid APIs

## Architecture

One Docker container, one port (8000):

| Layer | Choice |
|---|---|
| Frontend | Next.js static export (TypeScript, Tailwind, Recharts) |
| Backend | FastAPI on `uv`, serving the API and the static frontend |
| Database | SQLite at `db/finally.db`, lazily created and seeded |
| Real-time | Server-Sent Events (`/api/stream/prices`) |
| AI | LiteLLM → OpenRouter, forced tool calling for structured results |
| Market data | GBM simulator by default; Massive (Polygon.io) if a key is set |

## Quick Start

```bash
cp .env.example .env        # then add your OPENROUTER_API_KEY

docker build -t finally .
docker run -v "$PWD/db:/app/db" -p 8000:8000 --env-file .env finally
# open http://localhost:8000
```

`scripts/start_mac.sh` and `scripts/start_windows.ps1` wrap the same commands.

## Local Development

```bash
cd backend && uv run uvicorn app.main:app --reload --port 8000
cd frontend && npm run dev      # :3000, proxies /api/* to :8000
```

Tests: `uv run pytest` in `backend/`, `npm test` in `frontend/`, Playwright E2E in `test/`.
Run anything that touches chat with `LLM_MOCK=true`.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENROUTER_API_KEY` | yes | Key for the AI chat |
| `OPENROUTER_MODEL` | no | Defaults to `openrouter/nvidia/nemotron-3.5-lightning:free` |
| `MASSIVE_API_KEY` | no | Real market data; omit to use the simulator |
| `LLM_MOCK` | no | `true` returns deterministic mock LLM responses |

Names must be uppercase — the Linux container is case-sensitive even though Windows is not.

## The Free Model

The chat runs on a free OpenRouter model, so the app costs nothing to operate. In exchange,
responses take **7-58 seconds** (typically 20-30) and the account is limited to **20 requests
per minute and 50 per day**. The chat panel shows an elapsed counter and says so plainly
rather than hiding the wait. Pointing `OPENROUTER_MODEL` at a paid model is faster and
entirely optional.

## Project Structure

```
finally/
├── backend/     # FastAPI uv project (market data built; see planning/MARKET_DATA_SUMMARY.md)
├── frontend/    # Next.js static export
├── planning/    # PLAN.md and agent contracts
├── test/        # Playwright E2E tests
├── scripts/     # Start/stop helpers
└── db/          # SQLite bind mount (runtime)
```

Only `backend/` and `planning/` exist today; the rest is created as each component is built.

## Troubleshooting

**Certificate errors** (`CERTIFICATE_VERIFY_FAILED`) mean your network inspects TLS traffic,
common on corporate networks. Use `uv --system-certs` and add `truststore`.

## License

See [LICENSE](LICENSE).
