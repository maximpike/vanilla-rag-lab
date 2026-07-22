# Modules

## Entry Points

### server.js
The HTTP server bootstrap. Its only job is to import the configured Express app from `app.js`, bind it to a port, and
start listening. The port is read from `process.env.PORT` (set via `.env` / `config/env.js`) and falls back to `3000`.

This file is intentionally minimal - its owns *only* the `listen()` call, keeping the Express configuration testable 
in isolation via `app.js`

### app.js
The application's configuration hub. Responsible for:

- **Environment & database initialisation** - Imports `config/env.js` (which loads `.env` via dotenv) and `db/database.js`
    (which runs SQLite migrations) as side-effect imports. These run first because ES module import order is deterministic
    — they execute top-to-bottom before any application code.
- **Express setup** - Creates and exports the Express instance so `server.js` (and test harnesses) can import it without 
    triggering `listen()`.
- **Middleware** - Applies CORS (cross-origin requests), JSON body parsing, and static file serving from `public/`.
- **Route mounting** - Connects each API namespace to its route module under `/api/*`.

The exported `app` object is the boundary between "configuration" and "runtime" - everything above `listen()` lives here, 
and `server.js` handles the rest.

---

## Public — Client Modules

The `public/js/` directory contains all frontend code, split into two layers: **clients** (networking) and **UI** 
(DOM interaction). This separation means UI modules never construct URLs or call `fetch()` directly, and client modules 
never touch the DOM.

### Client Layer — Shared Pattern

All client modules follow the same structural pattern:

1. **Base URL constant** — Each client defines a `BASE` path (e.g., `"/api/collections"`) so route URLs are defined in  
one place.
2. **Two-tier error handling** — Every method uses a `try/catch` around the `fetch()` call to catch *network* errors 
(server unreachable, DNS failure), then checks `res.ok` to catch *HTTP* errors (400s, 500s). This distinction matters: a
network error means the request never reached the server, while an HTTP error means the server responded with a problem.
3. **Error message isolation** — On HTTP errors, the client reads the response body for an error message but wraps it in
a *new* `Error`. This prevents raw backend error details (stack traces, SQL errors) from leaking to the UI layer or the user.
4. **No DOM knowledge** — Clients return data or throw errors. They never reference `document`, elements, or any rendering
logic.

This pattern is consistent across all client modules below, so individual descriptions focus on what makes each module 
unique rather than repeating the shared structure.

### api-client.js
A minimal client with a single `fetchHello()` function used during initial project scaffolding to verify the frontend-to
-backend connection. Calls `GET /api/hello` and returns the JSON response. Unlike other clients, it uses a hardcoded 
absolute URL (`http://localhost:3000/api/hello`) rather than a relative `BASE` path — a leftover from Story 1 scaffolding.

### collection-client.js
CRUD operations for collections:

- `createCollection(name)` — `POST /api/collections` with a JSON body.
- `fetchCollections()` — `GET /api/collections`.
- `updateCollection(name, id)` — `PUT /api/collections/:id` to rename a collection.
- `deleteCollection(id)` — `DELETE /api/collections/:id`. No return value on success (void).

### document-client.js
Manages documents within collections. Includes client-side validation before the request is sent:

- `fetchDocuments(collectionId)` — `GET /api/documents/:collectionId`
- `fetchDocumentsWithEmbedStatus(collectionId)` — `GET /api/documents/:collectionId/embed-status`   
  - Returns documents annotated with whether they have been embedded into the vector store, allowing the UI to show embedding state.
- `uploadDocuments(files, collectionId)` — `POST /api/documents/:collectionId/upload`.  
  - Performs two validations before sending: rejects if no files are selected, and checks each file against a `MAX_FILE_SIZE` of 10MB.
  Sends files as `FormData` (multipart), not JSON. Currently limited to 10 files per upload (both limits are practical constraints to be revisited).
- `deleteDocument(id)` — `DELETE /api/documents/:id`  
  - Removes the document from its collection. No return value on success.

### embedding-client.js
Single-purpose client for triggering the embedding pipeline:

- `embedDocument(documentId)` — `POST /api/embeddings/:documentId`
  - Kicks off the server-side pipeline that extracts text, chunks it, generates embeddings via Ollama, and stores vectors
  in LanceDB. Returns `{ documentId, chunksCreated, dimensions }` on success.

### rag-client.js
Interfaces with the RAG query pipeline:

- `getCollectionStats(collectionId)` — `GET /api/rag/stats/:collectionId`  
  - Returns document count, chunk count, and embedding count for the selected collection. Used by the UI to show collection readiness.
- `queryRag(query, collectionId)` — `POST /api/rag/query` with JSON body `{ query, collectionId }`
  - Sends the user's question to the RAG pipeline, which embeds the query, performs semantic search, builds a prompt with retrieved 
  context, and returns `{ answer, sources }` where sources include document names and relevant excerpts.


## Public — UI Layer

### ui-orchestrator.js
The frontend's bootstrap module. When the browser loads `index.html`, this is the entry point that wires everything together. It:

- **Imports and initialises all page modules** — `router.js`, `sidebar-left.js`, `sidebar-right.js`, `collections-page.js`, and 
`rag-page.js` are imported as side-effect modules, meaning their top-level code (DOM queries, event listeners) runs on import.
- **Wires up the hello-world demo** — Attaches a click handler on `#helloBtn` that delegates to `fetchHello()` from `api-client.js`
(Story 1 scaffolding).

Despite its name suggesting orchestration, this module currently acts more as a **module bootstrapper** — it ensures all page 
modules are loaded and initialised. The actual UI logic (rendering, event handling, state) lives in the individual page modules
it imports.

---

## Services
The `services/` directory contains all backend business logic. Services have zero knowledge of HTTP - they never read request 
objects or write response objects. They receive plain arguments (strings, objects) and return data or throw errors. Routes are
the thin adapter layer that translates HTTP into service calls.

Services interact with two data stores: **SQLite** (via `better-sqlite3`) for structure metadata and **LanceDB** for vector 
storage. Most services pre-compile their SQL statements as module-level constants using `db.prepare()`, which means the 
statements are  parsed once at startup and reused on every call -  a performance pattern specific to `better-sqlite3`.

### Service Layer — Data Flow
The services form a pipeline that processes documents from upload through to queryable RAG:

```
Upload → Store metadata (document-service)
       → Extract text (pdf-extractor)
       → Split into chunks (chunker)
       → Generate embeddings (embedding-service)
       → Store vectors (vector-store)
       → Query: embed question → search vectors → build prompt → LLM answer (rag-query-service / rag-service)
```

Each step in the pipeline is a standalone module with a single responsibility. The `embedding-pipeline.js` orchestrates
the middle steps (extract → chunk → embed → store), while `rag-query-service.js` orchestrates the query path.

### collection-service.js
CRUD operations for collections. Each collection maps to both a SQLite row and a filesystem directory under `uploads/`:

- `addCollection(name)` — Generates a UUID, inserts a row into `collections`, and creates the upload directory. Returns `{ id, name }`.
- `listCollection()` — Returns all collections with a `doc_count` computed via a `LEFT JOIN` on `documents`. This lets the UI show 
document counts without a separate query.
- `updateCollection(name, id)` — Renames a collection and touches `updated_at`.
- `deleteCollection(id)` — Deletes the SQLite row (which cascades to documents and chunks via foreign keys), removes the upload 
directory from disk, and cleans up associated vectors from LanceDB.

The delete operation is `async` because the LanceDB cleanup (`deleteByCollection`) is asynchronous, while the SQLite and 
filesystem operations are synchronous.

### document-service.js
Manages document metadata and file storage within collections:

- `storeDocument(file, collectionId)` — Receives a multer file object, generates a UUID, moves the temp file to the collection's 
upload directory (renamed as `{uuid}_{originalname}`), and inserts metadata into SQLite. Returns a full document descriptor with 
id, collectionId, fileName, originalName, size, and mimeType.
- `listDocuments(collectionId)` — Returns all documents in a collection.
- `listDocumentsWithEmbedStatus(collectionId)` — Extends the basic listing with a `chunk_count` computed via a `LEFT JOIN` on 
- `chunks`. A `chunk_count` of 0 means the document has not been embedded; greater than 0 means it is queryable.
- `deleteDocument(id)` — Looks up the document, removes the file from disk (`unlinkSync`), deletes the SQLite row, and cleans 
up vectors from LanceDB. Returns silently if the document doesn't exist.

The file naming convention (`{uuid}_{originalname}`) prevents collisions while preserving the original name for display purposes.

### pdf-extractor.js
Single-purpose module that extracts text from PDF files on disk:

- `extractText(collectionId, fileName)` — Uses `pdfjs-dist` (Mozilla's PDF.js, legacy build for Node.js compatibility) to open 
the PDF, iterate through each page, and concatenate all text content. Respects end-of-line markers from the PDF structure (`item.hasEOL`)
to preserve paragraph boundaries. Pages are joined with double newlines, which the chunker later uses as its primary split point.

### chunker.js
Splits extracted text into overlapping chunks suitable for embedding. This is a pure function module with no I/O or database 
dependencies — it takes a string and returns chunk objects.

- `chunkText(text, opts)` — The public entry point. Accepts configurable `chunkSize` (default 500 characters) and `chunkOverlap` 
- (default 50 characters). Returns an array of `{ content, tokenEstimate }` objects.

The chunking algorithm works in two phases:
1. **Recursive splitting** (`recursiveSplit`) — Tries progressively finer separators: paragraph breaks (`\n\n`), line breaks (`\n`),
sentence endings (`. `, `! `, `? `), commas, then spaces. If text exceeds `chunkSize`, it splits on the coarsest available separator
and recurses on any pieces that are still too large, using the next-finer separator. This preserves semantic boundaries — paragraphs
stay together when possible, then sentences, then words.
2. **Merge with overlap** (`mergeWithOverlap`) — Reassembles small fragments into chunks up to `chunkSize`, then prepends 
`chunkOverlap` characters from the previous chunk to each subsequent chunk. The overlap ensures that information spanning a chunk 
boundary appears in at least one complete chunk, preventing context loss during retrieval.

The `tokenEstimate` uses a rough heuristic of 4 characters per token, which is a common approximation for English text.

### embedding-service.js
Interfaces with Ollama's embedding API using raw `fetch()` calls to the local Ollama instance:

- `embed(text)` — Generates a single embedding vector (768 dimensions with `nomic-embed-text`). Calls `POST /api/embed` with the 
text as input and returns the first vector from the response.
- `embedBatch(texts)` — Generates embeddings for multiple texts in a single API call. Ollama's `/api/embed` endpoint accepts an 
array as `input`, returning one vector per text in the same order. This avoids N round trips for N chunks.
- `checkOllamaStatus(requiredModels)` — Health check that verifies Ollama is reachable and that all required models are installed.
Calls `/api/tags` to list installed models, then checks against the required list. Returns a structured result with `available`,
`models`, and `reason` fields. Handles the `:latest` tag convention where Ollama may store a model as `nomic-embed-text:latest`
while you request `nomic-embed-text`.

Note: This service talks to the *local* Ollama instance (`localhost:11434`) for embeddings, while `rag-query-service.js` uses the
Ollama JS client for chat completions — two different connection patterns to Ollama.

### embedding-pipeline.js
The orchestrator that ties together the embedding workflow for a single document. This is where the individual service modules 
are composed into a complete pipeline:

- `embedDocument(documentId)` — Executes the full pipeline:
    1. **Fetch metadata** from SQLite to get the file path.
    2. **Check for re-embedding** — If chunks already exist for this document, deletes them first to allow clean re-processing.
    3. **Extract text** via `pdf-extractor.js`.
    4. **Chunk text** via `chunker.js`.
    5. **Generate embeddings** via `embedding-service.js` (batch call).
    6. **Store chunk metadata** in SQLite — Uses a `db.transaction()` to insert all chunks atomically, ensuring either all or none are persisted.
    7. **Store vectors** in LanceDB via `vector-store.js`.

Returns `{ documentId, chunksCreated, dimensions }` on success. Each step validates its output and throws descriptive errors on failure,
so callers get clear feedback about where the pipeline broke.

### vector-store.js

Manages vector storage and retrieval via LanceDB:

- `addVectors(rows)` — Inserts chunk vectors into the `chunks` table. Uses a try/catch pattern to either `openTable` (if it exists)
and `add`, or `createTable` on first use. This means the LanceDB table is lazily created on the first embedding, with no explicit
schema definition — LanceDB infers the schema from the first batch of records.
- `search(queryVector, collectionId, limit)` — Performs nearest-neighbour vector search scoped to a collection. Calls LanceDB's
`.search()` with a `.where()` filter on `collection_id` and returns results mapped to a clean object shape. The `score` field is
LanceDB's `_distance` (L2 distance by default — lower is more similar).
- `deleteByDocument(documentId)` — Removes all vectors for a document. Called during document deletion to prevent stale vectors
from polluting search results.
- `deleteByCollection(collectionId)` — Removes all vectors for a collection. Called during collection deletion.

The database connection is lazily initialised via `getDb()` and cached in a module-level variable. LanceDB stores data on disk at `data/lancedb/` as an embedded database — no separate server process required.

### rag-service.js

Provides simple aggregate queries against SQLite for collection statistics:

- `getDocumentCount(collectionId)` — Returns the number of documents in a collection.
- `getChunkCount(collectionId)` — Returns the number of chunks across all documents in a collection (joins `chunks` through `documents`).

These are used by the stats endpoint to show collection readiness in the UI.

### rag-query-service.js

Orchestrates the full RAG query flow — the read-side counterpart to `embedding-pipeline.js` (which handles the write side):

- `queryRag(query, collectionId)` — Executes the complete query pipeline:
    1. **Embed the query** using the same model (`nomic-embed-text`) that was used for documents, ensuring vectors are in
    the same embedding space.
    2. **Vector search** — Retrieves the top-K (default 5) most relevant chunks from LanceDB.
    3. **Enrich with metadata** — Looks up document names from SQLite so the LLM can cite sources.
    4. **Build prompt** — Constructs a system prompt that includes the retrieved context blocks, each labelled with a source
    number and document name. The prompt instructs the LLM to answer only from the provided context and to cite sources by document name.
    5. **Chat completion** — Sends the system prompt and user question to Ollama Cloud via the official JS client 
    (`ollama` package). Uses a lazy-initialised client with API key authentication.
    6. **Deduplicate sources** — Returns unique source documents with 120-character excerpts.

This module uses a different Ollama connection than `embedding-service.js`: it connects to Ollama Cloud (remote, 
authenticated via API key) for the chat model, while embeddings use the local Ollama instance. The chat model is configurable
via `CHAT_MODEL` environment variable.

---

## Routes

*To be documented in a subsequent pass.*

---

## Testing

*To be documented in a subsequent pass.*
