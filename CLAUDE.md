# CLAUDE.md

Guidance for Claude Code (and any AI assistant) working in this repository.

## What this is

`vanilla-rag-lab` — a full-stack RAG (Retrieval-Augmented Generation) system built with **vanilla JavaScript** (no frontend framework), Express 5, SQLite (`better-sqlite3`), and LanceDB for vector storage. Upload documents to collections, embed them via Ollama, and query them through a chat interface with source citations.

## ⚠️ Branching & PR rules (read first)

Both `main` and `dev` are **protected** — direct pushes are rejected. Every change must go through a pull request that passes the **"Quality Checks"** status check.

- **Never** `git push` to `main` or `dev`. Always create a `feature/*` branch and open a PR.
- No force-pushes to protected branches.
- Standard flow: `feature/*` → PR → merge. `dev` is the integration branch; `dev` → `main` is a release.
- See `.github/BRANCHING-STRATEGY.md` for the full policy.

## Commands

| Command | What it does |
|---|---|
| `npm install` | Install dependencies (Node 22+) |
| `docker compose up -d` | Start Ollama and pull models (`nomic-embed-text`, chat model) |
| `npm run dev` | Start the server with auto-restart (`node --watch`), port 3000 |
| `npm run dev:sync` | Browser-sync frontend hot reload (second terminal), proxies to 3001 |
| `npm start` | Start the production server |
| `npm test` | Run all Jest tests (ESM via `--experimental-vm-modules`) |
| `npm run test:unit` | Unit tests only (`tests/unit`) |
| `npm run test:integration` | Integration tests only (`tests/integration`) |

Run the relevant tests before opening a PR — CI runs the same suite as "Quality Checks".

## Architecture

Layered, with a strict rule: **each layer only talks to the one below it.**

- **`server.js`** — entry point; imports the configured app from `app.js` and calls `listen()`. Nothing else.
- **`app.js`** — Express configuration hub: side-effect imports of `config/env.js` (dotenv) and `db/database.js` (SQLite migrations) run first, then middleware (CORS, JSON, static `public/`) and route mounting under `/api/*`. Exports `app` so tests can import it without starting a server.
- **`routes/`** — thin HTTP adapters. They translate HTTP ⇄ service calls and nothing more.
- **`services/`** — all business logic. Services have **zero HTTP knowledge** — they take plain arguments and return data or throw. They pre-compile SQL with `db.prepare()` at module load. Two data stores: SQLite (metadata) and LanceDB (vectors).
- **`public/js/`** — frontend, split into **clients** (networking: build URLs, call `fetch`, never touch the DOM) and **UI/page modules** (DOM + events, never call `fetch` directly).

The RAG pipeline: `upload → extract text (pdf-extractor) → chunk (chunker) → embed (embedding-service) → store vectors (vector-store)`, orchestrated by `embedding-pipeline.js`. Query path: `embed question → vector search → build prompt → LLM answer`, orchestrated by `rag-query-service.js`.

**Embeddings** run on the **local** Ollama instance (`nomic-embed-text`, 768-dim). **Chat** runs on **Ollama Cloud** via the `ollama` JS client, configurable through the `CHAT_MODEL` env var.

Deeper module-by-module docs live in `MODULES.md`. Roadmap and open work are in `ROADMAP.md` and `NEXT_ACTIONS.md`.

## Conventions

- **ES modules** throughout (`"type": "module"`). Use `import`/`export`, not `require`.
- Prefer small, single-responsibility files — aim to keep modules under ~100 lines; split when they grow past it.
- Keep the layer boundaries intact: no `fetch` in UI modules, no DOM in client modules, no HTTP objects in services.
- Pure logic (e.g. `chunker.js`, validators) should stay I/O-free so it's unit-testable.

## Environment

Secrets live in `.env` (gitignored). Copy `.env.example` → `.env` and fill in values:

- `PORT` — server port (defaults to 3000)
- `OLLAMA_API_KEY` — Ollama Cloud auth for chat completions
- `LancdbAPIKey` — LanceDB cloud key
- `HF_API_TOKEN` — Hugging Face API token

`.env` is never committed; only `.env.example` (structure, no values) is.
