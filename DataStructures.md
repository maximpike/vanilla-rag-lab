# Data structures
synchronous vs asynchronous calls (and promises in javascript)
disk vs sqlite database
// Ask about sqlite persisting vs disk
what do we define __direnam with __
why doesnt export default collectionRoutes; work in routers
BUG: When you create a new collection it will display whatever the previous collections files are

- Should get collection stats live in the Rag Client module? and subsequent modules.
- Create relevant javascript google docs to note high level learnings.
- Should we rename ui-orchestrator? to bootstrap?
- 


// Coordinates the full pipeline for converting a document into searchable vectors:
//
//   PDF on disk → extract text → chunk → embed → store
//
// This module doesn't know HOW any of those steps work — it just calls each service in order and passes data
// between them. If we swap LanceDB for Qdrant or Ollama for Gemini, this file doesn't change.
// services/embedding-pipeline.js
//
// Orchestrates the full document embedding pipeline:
//   1. Look up the document in SQLite
//   2. Extract text from the PDF
//   3. Split text into overlapping chunks
//   4. Generate embeddings via Ollama
//   5. Store chunks in SQLite (metadata) + LanceDB (vectors)
//
// Each step is handled by a dedicated module — this file only
// coordinates the flow and manages the database transaction.


// service -rag-servie
// Handles RAG stats queries and the full RAG query flow:
//   1. Embed the user's question using the same model that embedded the documents
//   2. Vector search in LanceDB scoped to the active collection
//   3. Build a prompt with retrieved context chunks
//   4. Call Ollama Cloud's chat completion endpoint
//   5. Return the generated answer + source references
//
// The embedding model (nomic-embed-text) runs locally via Ollama.
// The chat model runs on Ollama Cloud — requires OLLAMA_API_KEY in .env


// ── Ollama Cloud client (lazy) ──────────────────────────
// Created on first use so process.env is populated by the time
// the client reads OLLAMA_API_KEY. ES module imports are hoisted
// and evaluated before top-level code (like dotenv.config()) runs,
// so constructing the client at module load time would capture
// undefined env vars.

https://fonts.google.com/icons
- https://fontawesome.com/
- https://lucide.dev/
- https://getbootstrap.com/
- https://heroicons.com/
- https://simpleicons.org/


- https://htmlreference.io/
- https://cssreference.io/

## Collection
```Collection
{ 
    id: "uuid" [PK]
    name: "Research Papers"
    "createdAt": "2026-02-13T...",
    "updatedAt": "2026-02-13T..."
}
```
Add icon: future enhancement

## Document
```Document
{
    id: "uuid" [PK]
    collection_id: "uuid" [FK]
    fileName(change to filename): "a]b2c3_file.pdf",
    "originalName": "My Research Paper.pdf",
    "size": 1048576,
    "mimeType": "application/pdf",
    "createdAt": "2026-02-13T..."
}
```

Bugs
WHen uploading Bert.pdf 16 pages I get the following error:

Warning: UnknownErrorException: Ensure that the `standardFontDataUrl` API parameter is provided.
Embedding pipeline failed: TypeError: fetch failed
at node:internal/deps/undici/undici:13510:13
at async embedBatch (file:///Users/maxpike/Workspace/vanilla-ai-lab/services/embedding-service.js:37:22)
at async embedDocument (file:///Users/maxpike/Workspace/vanilla-ai-lab/services/embedding-pipeline.js:41:21)
at async file:///Users/maxpike/Workspace/vanilla-ai-lab/routes/embedding-routes.js:14:24 {
[cause]: HeadersTimeoutError: Headers Timeout Error
at FastTimer.onParserTimeout [as _onTimeout] (node:internal/deps/undici/undici:6249:32)
at Timeout.onTick [as _onTimeout] (node:internal/deps/undici/undici:2210:17)
at listOnTimeout (node:internal/timers:588:17)
at process.processTimers (node:internal/timers:523:7) {
code: 'UND_ERR_HEADERS_TIMEOUT'
}
}

[Log] orchestrator initialised and modules linked. (ui-orchestrator.js, line 20)
> Selected Element
< <main class="main-content">…</main>
[Error] Failed to load resource: the server responded with a status of 500 (Internal Server Error) (fb7a6230-3192-470a-a9ff-df77e58b8e2f, line 0)
[Error] embedding-client error:
Error: Internal server error
(anonymous function) — embedding-client.js:15
(anonymous function) (embedding-client.js:16)
[Error] Embed failed:
Error: Internal server error
(anonymous function) — embedding-client.js:15
(anonymous function) (sidebar-right.js:251)
