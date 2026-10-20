# AI/LLM Application Development — Reference Implementations

Fullstack TypeScript developer building production AI/LLM application patterns with Google Cloud. These repositories explore AI application development progressively — embeddings, RAG, agents, multi-turn conversations, an MCP server, a data platform, and production optimizations.

They are proof-of-concept implementations built for learning and portfolio, not production systems. The goal is to understand each AI/LLM pattern in isolation and have working references demonstrating real-world use cases.

All POCs use the same **Financial Assistant** concept: AI-powered Q&A over SEC quarterly filings and stock market data, evolving from basic semantic search to multi-source agents with conversation memory.

**Framework:** Gemini 2.5 Flash (Vertex AI) · Fastify (TypeScript backend) · React  
**Data:** BigQuery (SEC filings, daily stock prices) · Firestore vector search  
**Deployment:** Cloud Run (GCP) · Docker containers

---

## Phase 1: MVP (Interview-ready)

### [llm-search](https://github.com/pawel-janus/llm-search) ✅

Semantic search engine over SEC quarterly filings. User searches by company/topic, returns most similar documents ranked by cosine similarity. Deployed to Cloud Run with 411ms warm latency, similarity scores 0.82-0.85.

**Patterns:** Vertex AI Text Embeddings API (text-embedding-004, 768D) · Firestore vector search (FieldValue.vector wrapper) · COSINE distance · BigQuery data loading (275 SEC filings) · Region optimization (europe-west4 + europe-central2)

**Key learnings:** FieldValue.vector() required for Firestore indexing · Region selection matters (400ms vs 900ms) · Egress cost negligible (~$0.003/month)

`TypeScript` `Fastify` `Vertex AI` `Firestore` `BigQuery` `Cloud Run` `Deployed`

---

### [llm-rag](https://github.com/pawel-janus/llm-rag) ✅

RAG (Retrieval-Augmented Generation) pipeline extending semantic search with Gemini 2.5 Flash. User asks questions in natural language ("What was Apple's net income in 2019?"), system retrieves top 5 SEC filings via vector search, injects formatted context into prompt, generates natural language answer with citations. Deployed to Cloud Run with ~2s warm latency.

**Patterns:** RAG pipeline (retrieve → inject → generate) · Gemini 2.5 Flash via Vertex AI · Prompt engineering (example-driven, human-readable context) · Citation tracking · Human-readable formatting (NetIncomeLoss → Net Income, $10.04B) · Q&A UI with example questions

**Key learnings:** Context formatting critical (tag translation, value formatting) · Prompt structure matters (example-driven vs restrictive) · Model availability varies by region (gemini-2.5-flash in europe-west4) · Cold start ~5s, warm ~2s

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Firestore` `BigQuery` `Cloud Run` `Deployed`

---

### llm-data ✅ *(private repository)*

ETL / data layer of the Financial Assistant, the source of the data the later POCs query. For a configurable list of companies it loads SEC quarterly facts and daily stock prices into BigQuery. Raw files are kept in Cloud Storage exactly as delivered; one Cloud Run Job per data source validates, transforms and loads them idempotently; rows that fail validation go to a quarantine table instead of being dropped. **The repository is private** (it handles data with licence restrictions), so there is no link. Superseded by `llm-data-platform` (see Phase 1b), which carries the same ideas further.

**GCP services:** BigQuery (load jobs, MERGE, datasets per data domain, `INFORMATION_SCHEMA` for usage tracking) · Cloud Storage (raw layer, private bucket) · Cloud Run Jobs (one per source, no public URL) · Cloud Build and Artifact Registry (image built without local Docker) · IAM (dedicated service accounts, least privilege)

**Patterns:** Raw layer exactly as delivered · One processor per source · Idempotent loads (MERGE, content hash) · Validation with quarantine · Incremental fetch with watermarks · Data source evaluation (public dataset vs EDGAR) · Cost control inside free tiers

`TypeScript` `BigQuery` `Cloud Storage` `Cloud Run Jobs` `Cloud Build` `Artifact Registry` `IAM` `Private repo`

---

### [llm-agent](https://github.com/pawel-janus/llm-agent) ✅

Agent using Gemini function calling over the BigQuery data layer. Five tools (company list, SEC facts, tag list, stock price, price change) are declared with JSON Schema and validated with Zod; the model decides which to call, in parallel when it can, and combines SEC figures with daily prices in one answer. Empty results carry a note and the nearest available periods, so the agent reports data gaps instead of estimating. The UI shows each tool call (arguments, duration, result). Deployed privately to Cloud Run with a dedicated service account and no public access.

**Patterns:** Gemini function calling · Multi-tool orchestration (agent loop with round limit) · Multi-source data fusion (SEC + prices) · Tool argument validation (Zod) · Honest "no data" behaviour enforced in code · Cost-capped BigQuery queries · Tool-call transparency in the UI

**Key learnings:** Behaviour that must happen belongs in code, not in the prompt · Fiscal-year labels and sparse filings need a derived data layer · Adjusted prices and as-reported EPS make P/E unreliable, so it was left out

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `BigQuery` `Cloud Run` `Deployed privately`

---

### [llm-chat](https://github.com/pawel-janus/llm-chat) ✅

Multi-turn version of the agent: the user can ask follow-ups ("And Walmart's?") that refer to earlier turns. Conversations are stored in Firestore (one document per session, messages in a subcollection), so the backend keeps no state and survives scale-to-zero and several instances. The whole text history is sent to Gemini with each question and the model resolves references itself; history handling lives in code around the agent loop, not in the tools. The UI has a session list, message history, a tool-call log and an alert for long conversations. Deployed privately to Cloud Run with a dedicated service account and no public access.

**Patterns:** Conversation storage (Firestore) · Session management (list, open, delete) · Multi-turn dialogue with history in the prompt · Reference resolution by the model · Stateless backend · Long-conversation alert (trimming and summarizing left for later)

**Key learnings:** History is application state handled in code, not a model tool · Storing only text turns keeps messages small but a follow-up needing a number calls the tool again · Sending the whole history works for short chats and needs trimming or summarizing for long ones

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Firestore` `BigQuery` `Cloud Run` `Deployed privately`

---

## Phase 1b: Platform and data

### [llm-mcp](https://github.com/pawel-janus/llm-mcp) ✅

MCP (Model Context Protocol) server that exposes the Financial Assistant data as read-only tools: five tools over SEC filing figures and daily stock prices in BigQuery (company list, facts, tag list, price, price change), in two groups. Any MCP client can call them instead of each app carrying its own copy of the data access code. One codebase serves two transports: stdio for local use (MCP Inspector, Claude Code) and stateless Streamable HTTP, deployed privately to Cloud Run with IAM-only access and a read-only service account. Tool schemas are Zod 4 and reach clients as JSON Schema; invalid arguments come back as a tool result the model can act on.

**Patterns:** MCP server (tools, annotations) · Stateless Streamable HTTP (a new server per request) · stdio and HTTP from one codebase · Schema validation reused as the tool contract · Private service behind IAM, reached locally through an authenticated proxy · Least-privilege service account

**Key learnings:** MCP SDK v2 is a set of small packages and the Fastify adapter only wraps the app, the protocol is served by `createMcpHandler` · Stateless servers fit Cloud Run: create the server and transport per request · Tool descriptions are part of the data contract, so the caveats about the data live there

`TypeScript` `MCP` `Fastify` `BigQuery` `Cloud Run` `Deployed privately`

---

### llm-data-platform ✅ *(private repository)*

Data platform behind the Financial Assistant, successor of `llm-data`: an npm workspaces monorepo with one ETL per source, one MCP server per data domain and a shared schema package per domain. SEC EDGAR facts of nine companies and daily prices are loaded into BigQuery by Cloud Run Jobs (incremental fetch, content hashes, validation with quarantine); views give the current value of each fact, the gaps of the data and 15 metrics with derived standalone quarters. Two private MCP servers on Cloud Run serve the data to a model, including growth, ratios, trailing twelve months and company comparison computed in code, and every answer carries data-quality flags (gaps, restated figures, derived quarters). 40 SQL data checks guard the loaded data. **The repository is private** (licence-restricted price data), so there is no link.

**Patterns:** Monorepo with shared schemas · ETL per source, MCP per domain · Derived views over restated filings · Standalone quarters by subtraction · Data-quality flags in tool results · SQL data checks with golden cases · Least-privilege service account per app

`TypeScript` `MCP` `BigQuery` `Cloud Storage` `Cloud Run Jobs` `Cloud Run` `Cloud Build` `IAM` `Private repo`

---

## Phase 2: Advanced Patterns (Future)

### llm-advanced-rag *(planned)*

Production RAG patterns: text chunking strategies, metadata filtering, reranking after vector search, hybrid search (vector + keyword). Improves retrieval precision for complex queries over large document collections.

**Patterns:** Text chunking (500-1000 tokens) · Chunk overlap · Metadata filtering (sector, company size) · Reranking (Cohere API) · Hybrid search (vector + BM25) · Query expansion

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Firestore` `Cohere` `Cloud Run`

---

### llm-multi-step-agent *(planned)*

Multi-step agent with chain-of-thought reasoning and planning. Agent breaks complex tasks into steps, executes sequentially, handles tool chaining (output of tool1 → input to tool2), recovers from errors.

**Patterns:** Chain of thought prompting · Agent planning · Tool chaining · Multi-step orchestration · ReAct pattern (Reason → Act → Observe) · Error recovery

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Cloud Run`

---

### llm-caching *(planned)*

Semantic caching for cost optimization. Cache LLM responses based on embedding similarity (not exact string match). Same question paraphrased → cached answer. 10x cost reduction for common queries.

**Patterns:** Semantic caching · Redis vector search (RediSearch) · Embedding-based cache key · TTL strategies · Cache invalidation · Cost optimization (90% cache hit rate target)

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Redis` `Memorystore` `Cloud Run`

---

### llm-guardrails *(planned)*

Content filtering, PII detection, prompt injection prevention. Input validation (Google Cloud DLP API for PII), toxicity detection (Perspective API), output filtering (sensitive data patterns), security against adversarial prompts.

**Patterns:** PII detection (Cloud DLP API) · Toxicity filtering (Perspective API) · Prompt injection prevention · Output validation · Rate limiting · Security guardrails

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Cloud DLP` `Perspective API` `Cloud Run`

---

### llm-multimodal *(planned)*

Multi-modal RAG over financial charts and diagrams. Gemini 2.5 Flash vision extracts text from images, embeds descriptions, retrieves based on visual content. User can upload charts/screenshots, ask questions about visual data.

**Patterns:** Gemini Vision API · Multimodal embeddings (image + text) · Image understanding · Cloud Storage integration · Visual Q&A · Chart/diagram analysis

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Vertex AI` `Cloud Storage` `Cloud Run`

---

### llm-llmops *(planned)*

Production LLMOps patterns: structured logging, monitoring, prompt versioning, A/B testing, cost tracking, user feedback collection. Cloud Logging for prompts/responses/tokens, Grafana dashboards, budget alerts.

**Patterns:** Structured logging (Cloud Logging) · Latency monitoring · Token usage tracking · Prompt versioning (Git) · A/B testing (split traffic) · User feedback (thumbs up/down) · Cost attribution · Grafana dashboards

`TypeScript` `Fastify` `Gemini 2.5 Flash` `Cloud Logging` `Cloud Monitoring` `Grafana` `Cloud Run`

---

### llm-vectordb-benchmark *(planned)*

Benchmark of three vector database options on the same SEC filings dataset: Firestore vector search, Supabase pgvector and Vertex AI Vector Search. Compares latency (p50/p95/p99), accuracy (recall@5), setup complexity, cost at several scales and production readiness, ending with a data-driven recommendation.

**Patterns:** Vector DB abstraction layer · Latency benchmarking · Recall@5 comparison · Cost projection · Production-readiness assessment

`TypeScript` `Firestore` `Supabase` `pgvector` `Vertex AI Vector Search`

---

> These implementations prioritize pattern clarity over production completeness. Focus is on understanding AI/LLM application patterns (not ML engineering: fine-tuning, training, model evaluation). Uses pre-trained models (Gemini, GPT-4) via API, not custom model development.
