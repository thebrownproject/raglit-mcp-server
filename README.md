# RagLit MCP Server

![Status](https://img.shields.io/badge/status-complete-green)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?logo=typescript&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?logo=postgresql&logoColor=white)

> Model Context Protocol server enabling AI agents to query and interact with PostgREST APIs

---

## Overview

RagLit is a Model Context Protocol (MCP) server that bridges AI agents like Claude Desktop with PostgREST-compatible backends for RAG (Retrieval-Augmented Generation) pipelines. The server implements the MCP specification to expose document chunking, embedding generation, and semantic search capabilities, enabling AI agents to ingest documents into PostgreSQL databases and query them using vector similarity search. Built with TypeScript using repository pattern for database abstraction, demonstrating understanding of emerging AI agent architectures and standardized tool protocols.

---

## Tech Stack

**Runtime:** Node.js · TypeScript <br>
**AI/ML:** OpenAI API (text-embedding-3-small) · pgvector <br>
**Database:** PostgreSQL · PostgREST <br>
**Architecture:** Model Context Protocol (MCP) · Repository Pattern <br>
**Tools:** Zod (validation) · ESLint

---

## Features

- MCP-compliant server implementing standardized tool protocol for AI agent communication
- Document ingestion pipeline with fixed-size word-based chunking and configurable overlap
- OpenAI embedding generation service with text-embedding-3-small model integration
- Semantic search using PostgreSQL pgvector extension via PostgREST RPC functions
- Metadata filtering with JSONB containment queries for exact-match document retrieval
- Repository pattern abstraction layer for PostgREST API with interface-based design
- Three MCP tools exposed: chunk_document, search_chunks, filter_metadata
- Environment-based configuration supporting Supabase and self-hosted PostgREST instances
- Schema validation using Zod for type-safe request/response handling

---

## Architecture & Tech Decisions

Built using the Model Context Protocol (MCP) specification, an emerging standard for AI agent tool integration pioneered by Anthropic. Implements stdio-based communication pattern where Claude Desktop launches the Node process and communicates via standard input/output. Chose repository pattern with abstract ChunkRepository interface to decouple business logic from PostgREST implementation details, enabling future adapter implementations for different backends. PostgREST integration leverages PostgreSQL's pgvector extension through RPC function calls (match_chunks, filter_chunks_by_meta) rather than direct SQL, providing API-level abstraction. Document chunking uses fixed-size word-count strategy with overlap to maintain semantic context across chunk boundaries. OpenAI embedding service encapsulates API calls with error handling and retry logic. Environment configuration validation using Zod ensures runtime type safety for required variables (EXTERNAL_API_URL, OPENAI_API_KEY).

---

## Learnings & Challenges

**Key Learnings:**

- Implementing Model Context Protocol (MCP) specification for AI agent tool integration
- Designing repository pattern abstractions for REST API interactions with type-safe interfaces
- Understanding pgvector cosine similarity search with PostgreSQL RPC function integration
- Building document chunking strategies balancing semantic coherence with embedding costs
- Using Zod for runtime schema validation and environment configuration

**Challenges Overcome:**

- Debugging PostgREST schema cache issues requiring exact column name matching (camelCase)
- Designing MCP tool schemas that balance flexibility with type safety using Zod
- Implementing proper error handling for cascading failures (OpenAI → PostgreSQL → PostgREST)
- Structuring repository interface to support both Supabase and self-hosted PostgREST
- Understanding MCP stdio communication pattern and Claude Desktop integration requirements

---

## Quick Start

```bash
npm install
npm run build
npm run start
# Server communicates via stdio for MCP clients
```

**Claude Desktop Integration:**
Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "raglit-postgrest": {
      "command": "node",
      "args": ["/absolute/path/to/raglit-fixed-mcp/dist/index.js"],
      "env": {
        "EXTERNAL_API_URL": "https://your-project.supabase.co",
        "OPENAI_API_KEY": "sk-your-key",
        "EXTERNAL_API_KEY": "your-postgrest-key"
      }
    }
  }
}
```

**PostgreSQL Setup:**

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create chunks table
CREATE TABLE public.chunks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    "documentId" TEXT NOT NULL,
    content TEXT NOT NULL,
    "chunkIndex" INTEGER NOT NULL,
    "chunkSize" INTEGER NOT NULL,
    "chunkOverlap" INTEGER NOT NULL,
    "chunkStrategy" TEXT DEFAULT 'fixed-size' NOT NULL,
    metadata JSONB DEFAULT '{}',
    embedding VECTOR(1536)
);

-- Create RPC functions for search and filter
-- (See full README for complete SQL)
```

**Environment Variables:**

```env
EXTERNAL_API_URL=https://your-postgrest-url
OPENAI_API_KEY=sk-your-openai-key
EXTERNAL_API_KEY=your-postgrest-api-key
EMBEDDING_MODEL=text-embedding-3-small
```
