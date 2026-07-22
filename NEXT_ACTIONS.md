# Next Actions

Working handoff for continuing this project on another machine. Last updated **2026-07-22**.

**Context:** Mid-flight documentation overhaul and project rebrand (`vanilla-ai-lab` → `vanilla-rag-lab`).
Task tracking is moving into [GitHub Issues](../../issues); the `.md` docs are becoming high-level narrative only.

---

## 0. Set up on the new laptop (do this first, right after cloning)

`.env` is gitignored, so it is **not** in a fresh clone. You must recreate it.

```bash
git clone <repo-url>
cd vanilla-rag-lab            # (repo may still be named vanilla-ai-lab until the rename lands)
npm install
cp .env.example .env         # then fill in real values (see table below)
docker compose up -d         # starts Ollama + pulls models
npm run dev                  # backend, port 3000
npm run dev:sync             # (second terminal) frontend hot reload, port 3001
```

### Environment variables

Copy the **real** secret values from the old laptop's `.env` — they are not stored in git.

| Variable         | Purpose                                                        |
|------------------|---------------------------------------------------------------|
| `PORT`           | Express server port (defaults to `3000`)                      |
| `OLLAMA_API_KEY` | Ollama Cloud auth for chat completions (`rag-query-service.js`) |
| `LancdbAPIKey`   | LanceDB cloud key                                              |
| `HF_API_TOKEN`   | Hugging Face API token                                         |

---

## 1. Finish the documentation overhaul (where you left off)

- [ ] **`MODULES.md` → Routes section** — currently a `*To be documented in a subsequent pass*` stub. Document each file in `routes/`.
- [ ] **`MODULES.md` → Testing section** — same stub; document the Jest setup, unit/integration split, and the extracted validators.
- [ ] **`.github/BRANCHING-STRATEGY.md`** — currently a 0-byte empty file. Write up the Git Flow model (main / dev / feature branches, branch protection) that ROADMAP Phase 4 already references.
- [ ] **`STORIES.md`** — retitled "historical - To Be Deleted". Migrate anything still useful into GitHub Issues, then delete the file.
- [ ] **`DataStructures.md`** — resolve/clean the raw open questions appended at the top (see §2), then keep only the finalized data-structure notes.

### Propagate the rebrand (`vanilla-ai-lab` → `vanilla-rag-lab`)

- [ ] `package.json` is done. **`README.md` still titled "Vanilla AI Lab"** — update title/description.
- [ ] Check `design.md` and any other doc for stale "AI Lab" references.
- [ ] Repo/remote URL: `package.json` now points at `github.com/maximpike/vanilla-rag-lab`. Rename the GitHub repo (or revert) so the URL is valid before others clone.

### Doc-vs-code consistency to reconcile

- [ ] `README.md` says the chat model is `llama3.2` (local, via `docker compose`), but `rag-query-service.js` defaults `CHAT_MODEL` to `gpt-oss:20b-cloud` on **Ollama Cloud**. Pick one story and make README + code agree.

---

## 2. Open design questions (pulled from `DataStructures.md`)

- Should `getCollectionStats` live in the RAG client module (and where do subsequent stats calls belong)?
- Rename `ui-orchestrator.js` → `bootstrap.js`? (MODULES.md already notes it behaves as a module bootstrapper, not an orchestrator.)
- Create JS/Google-docs notes capturing high-level learnings.
- Bug noted: creating a new collection can display the *previous* collection's files until refresh.

---

## 3. Git housekeeping before you commit

> ⚠️ **Your newest edits are NOT staged.** The MODULES.md expansion (~265 lines) and the entire
> ROADMAP.md rewrite are unstaged. Committing "staged only" would commit the *old* 25-line MODULES.md
> and **drop the ROADMAP rewrite**. Stage the working-tree changes before committing:

```bash
git add -A                                  # or add specific files below
# newer edits currently unstaged:
#   DataStructures.md  MODULES.md  ROADMAP.md  STORIES.md  package.json  services/rag-query-service.js
# new files to include:
git add NEXT_ACTIONS.md .env.example .gitignore
```

- `.DS_Store` is now gitignored (macOS section added). It was previously showing as untracked.
- Confirm nothing sensitive is staged (the real `.env` stays ignored).

---

## 4. Roadmap pointers (Phase 5 — Stabilisation, current)

Tracked in more detail in `ROADMAP.md`. Known items to verify during code review:

- **Tech debt:** audit route parameter handling across all route files; store timestamps as UTC with a timezone indicator.
- **Story 10 bugs:** collections-page row alignment; concurrent rename locking; sidebar↔page event sync on collection create; invalid date (double `Z` suffix) on collection creation.

Next planned phase is **Phase 6 — Agentic component** (LangGraph.js ReAct agent over uploaded datasets).
