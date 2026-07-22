# Roadmap - Updated/To Be Reviewed

## Overview

vanilla-rag-lab is a full-stack RAG (Retrieval-Augmented Generation) system built with vanilla JavaScript, Express.js, and SQLite. This roadmap tracks what's been completed, what needs fixing, and what's planned.

Detailed task tracking lives in [GitHub Issues](../../issues). This document provides the high-level narrative.

---

## Phase 1: Foundation (Complete)

Core application scaffolding, layout, and navigation.

| Story | Summary                                        | Status |
|-------|------------------------------------------------|--------|
| 1     | Project scaffolding & dev environment          | Done   |
| 2     | Three-panel layout & mint theme                | Done   |
| 3     | Left sidebar collapse toggle                   | Done   |
| 4     | Right sidebar collections system (local state) | Done   |
| 8     | Client-side page routing                       | Done   |
| 9     | Right sidebar collapse toggle                  | Done   |

---

## Phase 2: Data Layer (Complete)

Backend persistence, file management, and collection operations.

| Story | Summary                                            | Status |
|-------|----------------------------------------------------|--------|
| 5     | SQLite database & collections persistence          | Done   |
| 6     | File upload & document management                  | Done   |
| 10    | Manage collections page (rename, delete, table UI) | Done   |
| 11    | Document delete confirmation dialog                | Done   |

---

## Phase 3: RAG Pipeline (Complete)

Document processing, embedding generation, vector storage, and chat interface.

| Story  | Summary                                                              | Status |
|--------|----------------------------------------------------------------------|--------|
| 7      | Embedding pipeline (PDF extraction, chunking, Ollama, LanceDB)       | Done   |
| 12     | RAG chat page UI (stats cards, message bubbles, input bar)           | Done   |
| 13     | RAG query flow (embed query, vector search, LLM generation, sources) | Done   |

---

## Phase 4: Quality & CI (Complete)

Testing infrastructure and deployment pipeline.

| Item            | Summary                                                    | Status  |
|-----------------|------------------------------------------------------------|---------|
| Jest setup      | ES modules support with experimental VM modules            | Done    |
| Unit tests      | Chunker test suite (20 tests)                              | Done    |
| Validators      | Extracted pure validation functions for testability        | Done    |
| GitHub Actions  | dev-ci.yml and production-ci.yml workflows                 | Done    |
| Branch strategy | Git Flow with main/dev/feature branches, branch protection | Done    |

---

## Phase 5: Stabilisation (Current)

Bug fixes, tech debt, code review, and workflow improvements.

Tracked in GitHub Issues with labels: `bug`, `tech-debt`.

**Known tech debt:**
- TD-1: Audit route parameter handling across all route files
- TD-2: Store timestamps as UTC with timezone indicator

**Known bugs from Story 10:**
- Collections page row alignment (name cell elevation)
- Concurrent rename locking (multiple rows editable simultaneously)
- Sidebar-to-page event sync (collection created in sidebar not reflected on page)
- Invalid date on collection creation (double Z suffix)

> These items may have been partially patched during development. Verification needed during code review sessions.

---

## Phase 6: Agentic Component (Planned)

LangGraph.js agent for natural language dataset querying.

**Scope:**
- Dataset ingestion pipeline (CSV/JSON upload → SQLite tables)
- LangGraph.js ReAct agent with SQL query and schema inspection tools
- Conversation memory via LangGraph checkpointer
- Agent chat UI (reuse RAG chat patterns)
- Dataset inspector in right sidebar (schema, row preview)

**Dependencies:** `@langchain/langgraph`, `@langchain/core`, `csv-parse`

---

## Phase 7: Enhancements (Planned)

Feature ideas for future development. Each will become a GitHub Issue when ready.

**RAG improvements:**
- Advanced chunking strategies (semantic, sliding window, hierarchical)
- Hybrid search (vector + BM25 keyword search)
- Re-ranking pipeline (cross-encoder models)
- Conversation memory within RAG chat sessions
- Source attribution with inline citations linking to sidebar documents

**UI:**
- Similarity graph visualisation (HTML5 Canvas, pairwise cosine similarity)
- Dark mode (CSS custom property extension)
- Dashboard page with system metrics

**Infrastructure:**
- Multi-format document support (.txt, .md, .docx, .csv)
- API key management for cloud models (Gemini, OpenAI)
- GPU acceleration for embedding generation
- Persistent configuration (user preferences across restarts)
- Self-hosted vector database server for production
- Model provider abstraction layer (swap Ollama/Gemini/OpenAI)

---

## Phase 8: Deployment (Planned)

Production deployment strategy. Details TBD once application is stable.

---

## Version History

| Version  | Phase     | Notes                         |
|----------|-----------|-------------------------------|
| 0.1.0    | Phase 1-3 | MVP with full RAG pipeline    |
| 0.2.0    | Phase 4   | Testing and CI infrastructure |