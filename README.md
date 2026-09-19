# AI/LLM Application Development — Reference Implementations

Fullstack TypeScript developer building production AI/LLM application patterns with Google Cloud. These repositories explore AI application development progressively — embeddings, RAG, agents, multi-turn conversations, and production optimizations.

They are proof-of-concept implementations built for learning and portfolio, not production systems. The goal is to understand each AI/LLM pattern in isolation and have working references demonstrating real-world use cases.

All POCs use the same **Financial Assistant** concept: AI-powered Q&A over SEC quarterly filings and stock market data, evolving from basic semantic search to multi-source agents with conversation memory.

**Framework:** Gemini 2.0 Flash (Vertex AI) · Fastify (TypeScript backend) · React  
**Data:** BigQuery (SEC filings, IEX stock prices) · Firestore vector search  
**Deployment:** Cloud Run (GCP) · Docker containers

---

## Phase 1: MVP (Interview-ready)

### financial-assistant-search *(next)*

Semantic search engine over SEC quarterly filings. User searches by company/topic, returns most similar documents ranked by cosine similarity. No generation yet — pure vector search demonstrating embeddings and similarity matching.

**Patterns:** Vertex AI Text Embeddings API · Firestore vector search · Cosine similarity · BigQuery data loading · Semantic search vs keyword search

`TypeScript` `Fastify` `Vertex AI` `Firestore` `BigQuery` `Cloud Run`

---

### financial-assistant-rag *(planned)*

RAG (Retrieval-Augmented Generation) pipeline extending semantic search with Gemini 2.0 Flash. User asks questions in natural language, system retrieves relevant SEC filings, injects them into prompt, generates answer with citations. Demonstrates prompt engineering and hallucination prevention.

**Patterns:** RAG pipeline (retrieve → inject → generate) · Gemini API integration · Prompt engineering · Citation tracking · Context window management · Hallucination prevention

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Firestore` `BigQuery` `Cloud Run`

---

### financial-assistant-agent *(planned)*

Multi-source agent using Gemini Function Calling to combine SEC fundamentals with IEX stock prices. Agent orchestrates multiple tools (searchFilings, getStockPrice, calculatePE) to answer complex financial questions requiring data fusion from multiple datasets.

**Patterns:** Gemini Function Calling · Multi-tool orchestration · Multi-source data fusion (SEC + IEX) · Agent decision-making · Tool execution pipeline · Financial metrics calculation (P/E ratio)

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Firestore` `BigQuery` `Cloud Run`

---

### financial-assistant-chat *(planned)*

Multi-turn conversation system with context management. User can ask follow-up questions referencing previous context. Conversation history stored in Firestore, entity extraction (ticker, period) for reference resolution, context window limiting.

**Patterns:** Conversation storage (Firestore) · Context window management · Multi-turn dialogue · Session management · Entity extraction · Reference resolution ("it", "the stock" → AAPL)

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Firestore` `BigQuery` `Cloud Run`

---

## Phase 2: Advanced Patterns (Future)

### financial-assistant-advanced-rag *(planned)*

Production RAG patterns: text chunking strategies, metadata filtering, reranking after vector search, hybrid search (vector + keyword). Improves retrieval precision for complex queries over large document collections.

**Patterns:** Text chunking (500-1000 tokens) · Chunk overlap · Metadata filtering (sector, company size) · Reranking (Cohere API) · Hybrid search (vector + BM25) · Query expansion

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Firestore` `Cohere` `Cloud Run`

---

### financial-assistant-multi-step *(planned)*

Multi-step agent with chain-of-thought reasoning and planning. Agent breaks complex tasks into steps, executes sequentially, handles tool chaining (output of tool1 → input to tool2), recovers from errors.

**Patterns:** Chain of thought prompting · Agent planning · Tool chaining · Multi-step orchestration · ReAct pattern (Reason → Act → Observe) · Error recovery

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Cloud Run`

---

### financial-assistant-caching *(planned)*

Semantic caching for cost optimization. Cache LLM responses based on embedding similarity (not exact string match). Same question paraphrased → cached answer. 10x cost reduction for common queries.

**Patterns:** Semantic caching · Redis vector search (RediSearch) · Embedding-based cache key · TTL strategies · Cache invalidation · Cost optimization (90% cache hit rate target)

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Redis` `Memorystore` `Cloud Run`

---

### financial-assistant-guardrails *(planned)*

Content filtering, PII detection, prompt injection prevention. Input validation (Google Cloud DLP API for PII), toxicity detection (Perspective API), output filtering (sensitive data patterns), security against adversarial prompts.

**Patterns:** PII detection (Cloud DLP API) · Toxicity filtering (Perspective API) · Prompt injection prevention · Output validation · Rate limiting · Security guardrails

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Cloud DLP` `Perspective API` `Cloud Run`

---

### financial-assistant-multimodal *(planned)*

Multi-modal RAG over financial charts and diagrams. Gemini 2.0 Flash vision extracts text from images, embeds descriptions, retrieves based on visual content. User can upload charts/screenshots, ask questions about visual data.

**Patterns:** Gemini Vision API · Multimodal embeddings (image + text) · Image understanding · Cloud Storage integration · Visual Q&A · Chart/diagram analysis

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Vertex AI` `Cloud Storage` `Cloud Run`

---

### financial-assistant-llmops *(planned)*

Production LLMOps patterns: structured logging, monitoring, prompt versioning, A/B testing, cost tracking, user feedback collection. Cloud Logging for prompts/responses/tokens, Grafana dashboards, budget alerts.

**Patterns:** Structured logging (Cloud Logging) · Latency monitoring · Token usage tracking · Prompt versioning (Git) · A/B testing (split traffic) · User feedback (thumbs up/down) · Cost attribution · Grafana dashboards

`TypeScript` `Fastify` `Gemini 2.0 Flash` `Cloud Logging` `Cloud Monitoring` `Grafana` `Cloud Run`

---

> These implementations prioritize pattern clarity over production completeness. Focus is on understanding AI/LLM application patterns (not ML engineering: fine-tuning, training, model evaluation). Uses pre-trained models (Gemini, GPT-4) via API, not custom model development.
