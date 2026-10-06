# Embeddings + Neo4j RAG

A Retrieval-Augmented Generation (RAG) pipeline in Node.js + TypeScript that ingests a PDF about tensors and TensorFlow.js, stores its chunks as vector embeddings in Neo4j, and answers questions in Portuguese using only the retrieved context.

Embeddings are generated **locally** with Transformers.js (`all-MiniLM-L6-v2`), so no embedding API is needed. Only the final answer generation calls an LLM through OpenRouter.

## How it works

1. **Load & split** — `tensores.pdf` is loaded with LangChain's `PDFLoader` and split into overlapping chunks (`RecursiveCharacterTextSplitter`, 1000 chars / 200 overlap).
2. **Embed** — each chunk is converted into a 384-dimension vector with `HuggingFaceTransformersEmbeddings` running on-device.
3. **Index** — previous `Chunk` nodes are removed and the new chunks are stored in Neo4j as `Chunk` nodes with a vector index (`tensors_index`).
4. **Retrieve** — for each question, a similarity search returns the top-K (3) chunks; chunks with score ≤ 0.5 are discarded.
5. **Generate** — the retrieved context is injected into a prompt template (`prompts/template.txt`) driven by a JSON prompt config (`prompts/answerPrompt.json`) and sent to the LLM via a LangChain `RunnableSequence`.
6. **Persist** — each answer is printed and saved as Markdown in `respostas/`.

```
PDF ──► chunks ──► embeddings (local) ──► Neo4j vector index
                                               │
question ──► embedding ──► similarity search ──┘
                                │
                     top-K context + prompt ──► LLM (OpenRouter) ──► answer.md
```

## Tech Stack

- Node.js 22 with native TypeScript (`--experimental-strip-types`, no build step)
- LangChain.js (document loaders, text splitter, runnables, prompt templates)
- Transformers.js — local embeddings with `Xenova/all-MiniLM-L6-v2`
- Neo4j 5 (Docker) as vector store
- OpenRouter (OpenAI-compatible API) for the chat model

## Project Structure

- `src/index.ts` - Entry point: ingestion pipeline, sample questions, and answer persistence
- `src/config.ts` - Centralized configuration (Neo4j, OpenRouter, splitter, embedding model, top-K)
- `src/documentProcessor.ts` - Loads the PDF and splits it into chunks
- `src/ai.ts` - RAG chain: vector retrieval + LLM answer generation
- `src/util.ts` - Helper to pretty-print retrieved chunks
- `prompts/answerPrompt.json` - Role, task, constraints, and instructions for the assistant
- `prompts/template.txt` - Prompt template filled with the config, question, and context
- `tensores.pdf` - Source document (knowledge base)
- `docker-compose.yml` - Neo4j service with APOC

## Requirements

- Node.js 22+
- Docker with Compose v2 (`docker compose`)
- An [OpenRouter](https://openrouter.ai) API key

> **Using a `:free` model?** OpenRouter only routes free models if your account allows it. Enable the free-model data policy at <https://openrouter.ai/settings/privacy>, otherwise requests fail with `404 ... Free model training violation`. Alternatively, set `NLP_MODEL` to any paid model.

## Setup and Run

1. Install dependencies:
```
npm install
```

2. Create your `.env` from the example and fill in your OpenRouter key:
```
cp .env.example .env
```

3. Start Neo4j (waits until the database is healthy):
```
npm run infra:up
```

4. Run the pipeline:
```
npm start
```

The first run downloads the embedding model (~90 MB) to the local cache. Answers are written to `respostas/`.

5. (Optional) Explore the graph at <http://localhost:7474> (user `neo4j`, password `password`):
```cypher
MATCH (c:Chunk) RETURN c.text, c.source LIMIT 10
```

6. Stop and remove the database:
```
npm run infra:down
```

## Customization

- **Questions** — edit the `questions` array in `src/index.ts`.
- **Knowledge base** — replace `tensores.pdf` or change `CONFIG.pdf.path`.
- **Retrieval** — tune `textSplitter` (chunk size/overlap) and `similarity.topK` in `src/config.ts`.
- **Assistant behavior** — change role, tone, and rules in `prompts/answerPrompt.json` without touching code.
- **Models** — swap `EMBEDDING_MODEL` or `NLP_MODEL` in `.env`.
