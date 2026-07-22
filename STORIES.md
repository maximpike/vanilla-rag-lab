# Stories - historical - To Be Deleted

## Story 1: Project Scaffolding & Dev Environment
**Commit:** Initial scaffolding

- Express 5 server with static file serving (`public/`)
- ES module configuration (`"type": "module"`)
- CORS middleware
- `npm run dev` with `--watch` for auto-restart
- Frontend hello-world: `ui-orchestrator.js` → `api-client.js` → `GET /api/hello`
- README, ROADMAP, LICENSE, .gitignore

## Story 2: Three-Panel Layout & Theming
**Commit:** feat: Add three-panel layout with mint theme and component CSS

- Three-panel CSS Grid layout: left sidebar (260px), main content (flex), right sidebar (304px)
- Mint theme via CSS custom properties (backgrounds, accents, text, borders, spacing, radii)
- Component CSS files: `sidebar-left.css`, `sidebar-right.css`, `main-content.css`, `footer.css`
- Left sidebar: logo/branding, nav links (Home, Dashboard, RAG Chat), settings + collapse footer
- Right sidebar: header, upload button, document list structure
- Collapsed sidebar CSS (visual rules only, JS toggle in later story)
- Footer with status indicator and version display
- CSS reset and base typography
- JS files moved to `public/js/` directory

## Story 3: Left Sidebar — Collapse Toggle
**Commit:** feat: Add sidebar collapse toggle

- `sidebar-left.js`: collapse button toggles `.collapsed` class on sidebar
- CSS transition from Story 2 handles the animation
- Imported as side-effect module in `ui-orchestrator.js`

## Story 4: Right Sidebar — Collections System
**Commit:** feat: Add collections system to right sidebar

- Three-state right sidebar: empty state → create input → active collection
- Collection creation with inline input, confirm/cancel buttons, Enter/Escape shortcuts
- Collection selector displays active collection name and document count
- Dropdown with collection switching, active item hidden, "New Collection" option
- Outside-click dismissal for dropdown
- Truncation on long collection names via `text-overflow: ellipsis`
- CSS sibling combinators hide content when dropdown is open
- Collections stored in local JS array (no backend persistence yet)
- Removed `transition: all` from base button rule, cleaned `transform: none` overrides

## Story 5: Backend Infrastructure & Collections Persistence
**Commit:** feat: Add SQLite database and collections persistence

- SQLite via `better-sqlite3` with WAL mode and foreign key enforcement
- SQL migration file (`001-init.sql`) for collections and documents tables
- `collection-service.js`: CRUD operations with UUID keys, disk directory management
- `collection-routes.js`: REST endpoints with validation and generic error responses
- `collection-client.js`: API wrappers with separated network/HTTP error handling
- `sidebar-right.js` refactored from local array to backend API calls
- `renderDropdown` uses `createElement`/`textContent` to prevent XSS injection
- Collections survive page refreshes via `init()` fetch on load
- Upload directories created per-collection in `uploads/<collection_id>/`

## Story 6: File Upload & Document Management
**Commit:** feat: Add file upload and document management

- `document-service.js`: store, list, delete documents with SQLite + disk storage
- `document-routes.js`: REST endpoints scoped by collection ID, multer middleware
- `document-client.js`: API wrappers with separated network/HTTP error handling
- Multi-file upload support (max 10 files, 10MB limit per file)
- Files stored at `uploads/<collection_id>/<uuid>_originalname.pdf` (collision-safe)
- Upload button triggers hidden file input, change event sends files to API
- `refreshDocuments()` re-fetches and re-renders document list after upload/delete
- Document count synced in collection selector and dropdown
- Delete button with hover reveal per document item

## Story 7: Document Embedding Pipeline
**Commit:** feat: Add document embedding pipeline with PDF extraction, chunking, and vector storage

- `pdf-extractor.js`: text extraction from PDFs via `pdfjs-dist` legacy build (Node.js compatible)
- `chunker.js`: recursive text splitting with overlap (500 char chunks, 50 char overlap)
- `embedding-service.js`: Ollama `nomic-embed-text` embeddings (768-d), single + batch, health check
- `vector-store.js`: LanceDB wrapper for vector insert, similarity search, document/collection deletion
- `embedding-pipeline.js`: orchestrates extract → chunk → embed → store with SQLite transaction
- `embedding-routes.js`: `POST /:documentId` triggers pipeline, `POST /search/:collectionId` queries vectors
- `model-routes.js`: `GET /ollama/status` health check, separated for future model integrations (Gemini)
- SQL migration `002-chunks.sql`: chunks table with document FK, content, token estimate, index
- `database.js` updated to run migrations sequentially from array
- `server.js` mounts `/api/embeddings` and `/api/models` routes
- New dependencies: `pdfjs-dist`, `@lancedb/lancedb`, `@lancedb/lancedb-darwin-x64`
- Vector data stored at `data/lancedb/`, chunk metadata in SQLite
- Re-embedding a document clears old chunks before reprocessing
- Known gap: LanceDB vector cleanup not yet wired into document/collection delete services

## Story 8: Client-Side Page Routing
**Commit:** feat: Add client-side page routing between Home, Dashboard, and RAG

- `data-page` attributes on nav links and `<section>` elements
- `public/js/router.js` — show/hide page sections, toggle `active` class on nav
- Refactor `main-content.css`: move layout properties (max-width, padding, margin) from `.main-content` to `.page-home` so each page owns its own layout
- Three page shells: `page-home`, `page-dashboard` (placeholder), `page-rag`
- Wire router into `ui-orchestrator.js`

## Story 9: Right Sidebar Collapse
**Commit:** feat: Add right sidebar collapse toggle

- Collapse icon button in the sidebar-right header (top-right corner)
- CSS collapsed state for `.sidebar-right` (mirror left sidebar pattern)
- JS toggle in `sidebar-right.js`
- Transition animation matching left sidebar (width 0.3s ease-in-out)

## Story 10: Manage Collections Page
**Commit:** feat: Add dedicated collections management page with rename and delete

- New "Collections" page in main content (`data-page="collections"`)
- Left sidebar nav link with `dataset` icon, ordered after Home
- Table layout: name (with folder icon), document count, created date, actions
- Inline rename: edit button → name cell becomes input with Enter/Escape/confirm/cancel
- Delete with confirmation dialogue backdrop ("Delete 'X' and all its documents?")
- "New Collection" button → inline create row in table footer
- Wired to existing `PUT /api/collections/:id` and `DELETE /api/collections/:id`
- Wired LanceDB vector cleanup into collection delete (`deleteByCollection`)
- `deleteCollection` service now async, route handler updated to `await`
- Custom events for cross-module sync: `collections-changed` (page ↔ sidebar), `page-changed` (router → page)
- Sidebar-right stays focused: quick-switch collections, upload files, view documents
- Right sidebar listens for `collections-changed` and re-fetches automatically
- New CSS file: `collections-page.css` (table, inline rename, create row, empty state, dialog)

## Story 11: Document Delete Confirmation
**Commit:** feat: Add confirmation prompt before document deletion

- Reusable confirm dialogue module (`confirm-dialogue.js`), shared with Story 10
- Show document name in prompt: "Delete report.pdf?"
- Confirm / Cancel buttons, styled consistently with app theme
- Wire LanceDB vector cleanup into individual document delete (known gap from Story 7)
- Replace current instant-delete with confirmation flow

## Story 12: RAG Chat Page — UI
**Commit:** feat: Add RAG chat interface with stats display

- `public/css/rag-page.css` — stats cards, chat container, message bubbles, input bar
- Stats row: Documents, Chunks, Embeddings counts (fetched from API)
- Chat message rendering: user bubbles (right-aligned) and assistant bubbles (left-aligned)
- Source references section below assistant messages
- Loading/typing indicator during generation
- Auto-scroll to latest message
- `public/js/rag-page.js` — chat UI logic
- API endpoint: `GET /api/rag/stats/:collectionId`

## Story 13: RAG Chat — Query, Retrieval & Generation
**Commit:** feat: Add RAG query flow with retrieval and LLM generation

- `POST /api/rag/query` — accepts `{ query, collectionId }`
- `services/rag-query-service.js` — orchestrate: embed query → vector search → build prompt → LLM call
- Prompt template: system context + retrieved chunks + user question
- Ollama chat completion (e.g. `llama3.2` or configurable model)
- Response includes generated answer + source references (document name, chunk excerpt)
- Wire frontend chat input to POST endpoint
- Display sources as clickable references below the answer


# Stories

## Story 1: Project Scaffolding & Dev Environment
**Commit:** Initial scaffolding

- Express 5 server with static file serving (`public/`)
- ES module configuration (`"type": "module"`)
- CORS middleware
- `npm run dev` with `--watch` for auto-restart
- Frontend hello-world: `ui-orchestrator.js` → `api-client.js` → `GET /api/hello`
- README, ROADMAP, LICENSE, .gitignore

## Story 2: Three-Panel Layout & Theming
**Commit:** feat: Add three-panel layout with mint theme and component CSS

- Three-panel CSS Grid layout: left sidebar (260px), main content (flex), right sidebar (304px)
- Mint theme via CSS custom properties (backgrounds, accents, text, borders, spacing, radii)
- Component CSS files: `sidebar-left.css`, `sidebar-right.css`, `main-content.css`, `footer.css`
- Left sidebar: logo/branding, nav links (Home, Dashboard, RAG Chat), settings + collapse footer
- Right sidebar: header, upload button, document list structure
- Collapsed sidebar CSS (visual rules only, JS toggle in later story)
- Footer with status indicator and version display
- CSS reset and base typography
- JS files moved to `public/js/` directory

---

## Tech Debt

### TD-1: Audit route parameter handling across all route files
**Priority:** Low
**Context:** During Story 10, a bug surfaced in the DELETE `/api/collections/:id` handler where destructured `req.params` was passed incorrectly to a downstream service. Fixed by using `req.params.id` directly.

**Scope:**
- Review all handlers in `collection-routes.js`, `document-routes.js`, `embedding-routes.js`, `model-routes.js`
- Ensure `req.params.*` and `req.body.*` values are accessed inline rather than destructured into intermediate variables where unnecessary
- Confirm each parameter reaches its service function as the expected primitive type (string, number)
- Add guard clauses for missing or malformed params where absent (e.g. `embedding-routes.js` POST has no check for missing `documentId`)
- Standardise the pattern: validate early, return 400 with a clear message, then call the service

### TD-2: Store timestamps as UTC with timezone indicator
**Priority:** Low
**Context:** SQLite `datetime('now')` produces strings like `"2025-02-15 14:30:00"` with no timezone suffix. The frontend has to guess UTC by appending `"Z"`, which broke when client-generated ISO strings (which already include `Z`) were passed through the same `formatDate` function. A defensive check was added, but the root cause remains in the schema.

**Scope:**
- Update migration default expressions to use `strftime('%Y-%m-%dT%H:%M:%SZ', 'now')` so all stored timestamps are unambiguous ISO 8601 with `Z` suffix
- Audit all tables: `collections.created_at`, `collections.updated_at`, `documents.created_at`, `chunks.created_at`
- Simplify `formatDate` on the frontend — remove the conditional `Z` append once all timestamps include it
- Consider returning `created_at` from the `addCollection` service so the frontend doesn't need to fabricate it