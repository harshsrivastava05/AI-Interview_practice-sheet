# AI Interview Preparation Guide

**Compiled from this chat**  
**Topics covered:** MCP, LLM APIs, vector databases, fine-tuning vs RAG, AI frameworks, model internals, multimodal AI, and AI product sense.

---

## How to use this guide

This document is designed for AI/LLM engineering interviews. It focuses on:

1. What each concept means.
2. How it is used in real production systems.
3. Where it fails.
4. What tradeoffs matter.
5. Common interview questions with strong answers.

The strongest interview pattern is:

> Do not talk about LLMs as isolated API calls. Talk about production systems: permissions, schemas, retrieval quality, latency, cost, observability, retries, fallbacks, evals, and user trust.

---

# Table of Contents

1. [MCP — Model Context Protocol](#1-mcp--model-context-protocol)
2. [OpenAI / Anthropic / Gemini APIs](#2-openai--anthropic--gemini-apis)
3. [Vector Databases](#3-vector-databases)
4. [Fine-tuning vs RAG](#4-fine-tuning-vs-rag)
5. [Framework Alternatives](#5-framework-alternatives)
6. [Model Internals](#6-model-internals)
7. [Multimodal AI](#7-multimodal-ai)
8. [AI Product Sense](#8-ai-product-sense)
9. [Final Production Architecture Lens](#9-final-production-architecture-lens)
10. [High-Impact Interview Revision Checklist](#10-high-impact-interview-revision-checklist)

---

# 1. MCP — Model Context Protocol

## 1.1 What MCP is

MCP stands for **Model Context Protocol**.

It is an open protocol for connecting AI applications to external systems such as:

- Tools
- Files
- Databases
- APIs
- Prompts
- SaaS products
- Internal services
- Knowledge stores

Simple interview explanation:

> MCP is a standardized protocol that lets an AI host discover and interact with external tools and context sources without every AI app needing custom integrations for every external system.

Without MCP, each AI application must build its own custom integration with every tool.

```text
Without MCP:

AI App A → custom GitHub integration
AI App A → custom Slack integration
AI App A → custom database integration

AI App B → custom GitHub integration
AI App B → custom Slack integration
AI App B → custom database integration
```

With MCP:

```text
With MCP:

AI Host → MCP Client → MCP Server → GitHub
AI Host → MCP Client → MCP Server → Slack
AI Host → MCP Client → MCP Server → Database
```

This reduces duplicate integration work and improves agent-tool interoperability.

---

## 1.2 MCP host-client-server model

MCP uses a **host-client-server** model.

```text
User
  ↓
AI Host / AI App
  ↓
MCP Client
  ↓
MCP Server
  ↓
External system
```

### Host

The host is the AI application the user interacts with.

Examples:

- ChatGPT-like app
- Claude Desktop
- Cursor
- VS Code extension
- Internal enterprise assistant
- Your own AI tutor app

### Client

The MCP client lives inside the host. It manages communication with one MCP server.

Usually, one client connects to one server.

```text
Host
  ├── MCP Client → GitHub MCP Server
  ├── MCP Client → Postgres MCP Server
  ├── MCP Client → Slack MCP Server
  └── MCP Client → Filesystem MCP Server
```

### Server

The MCP server exposes capabilities to the client.

These capabilities can include:

- Tools
- Resources
- Prompts

A server usually wraps some external system.

Example:

```text
Postgres MCP Server
  exposes:
    - list_tables
    - describe_schema
    - query_readonly_data
```

---

## 1.3 MCP tools

Tools are callable operations.

They are used when the model needs to **do something**.

Examples:

```text
search_interview_experiences(company, role)
get_user_progress(user_id)
create_recommendation_event(user_id, problem_id)
fetch_leetcode_submission(submission_id)
create_jira_ticket(title, description)
send_slack_message(channel, message)
```

A tool should have:

- Name
- Description
- Input schema
- Output schema or expected result format
- Permission checks
- Error behavior
- Rate limits
- Audit logging

### Production example

Bad tool:

```text
run_sql(query: string)
```

This is dangerous because the model can generate arbitrary SQL.

Better tools:

```text
get_user_topic_progress(user_id, topic_slug)
list_active_recommendations(user_id)
create_recommendation_event(user_id, problem_slug, reason_code)
```

Why this is better:

- Narrower action surface
- Easier validation
- Easier authorization
- Easier logging
- Lower blast radius

### Production rule

> Expose domain-specific tools, not arbitrary execution tools.

---

## 1.4 MCP resources

Resources are readable context.

They are used when the model needs to **know something**, not perform an action.

Examples:

```text
user://profile/{user_id}
catalog://dsa/topics
company://interview-patterns/google/backend
file://resume.pdf
db-schema://recommendation_tables
```

A resource can represent:

- File content
- Database records
- User profile
- App state
- API result
- Schema
- Knowledge base document

### Resource vs tool

| Capability | Purpose | Example |
|---|---|---|
| Resource | Read context | `user://progress/{user_id}` |
| Tool | Perform action | `create_recommendation_event(...)` |

In production, resources should usually be read-only.

If something changes state, represent it as a tool.

---

## 1.5 MCP prompts

Prompts are reusable prompt templates exposed by MCP servers.

They help standardize workflows across clients.

Examples:

```text
prompt://generate-company-study-plan
prompt://summarize-user-weaknesses
prompt://create-revision-session
prompt://extract-interview-rounds
```

A prompt may accept arguments:

```json
{
  "company": "Google",
  "role": "Backend Engineer",
  "time_window": "2 months",
  "weak_topics": ["Graphs", "Dynamic Programming"]
}
```

Then it generates a standardized prompt workflow.

### Production use

Prompts are useful when multiple clients or agents need consistent behavior.

For example, your AI tutor may have a standard prompt for:

- Generating a study plan
- Summarizing weaknesses
- Producing mock interview feedback
- Creating revision tasks
- Extracting interview round data

Instead of duplicating prompt text in backend, frontend, worker jobs, and agents, expose reusable prompts through MCP.

---

## 1.6 Access boundaries in MCP

This is one of the most important interview topics.

MCP standardizes connectivity, but it does **not automatically make tools safe**.

Security still depends on:

- Authentication
- Authorization
- Tenant isolation
- Permission scopes
- Tool allowlists
- Human approval
- Input validation
- Output filtering
- Audit logs
- Rate limits

### Production access boundaries

| Boundary | Purpose |
|---|---|
| User identity | Know which user initiated the action |
| Tenant isolation | Prevent cross-user or cross-company data leakage |
| Scope-based permissions | Restrict what tools/resources can be used |
| Tool allowlist | Only expose approved tools |
| Approval gates | Require confirmation for sensitive actions |
| Input validation | Prevent malformed or malicious arguments |
| Output filtering | Remove secrets, tokens, PII, excessive data |
| Rate limiting | Prevent loops, abuse, and runaway costs |
| Audit logging | Record what happened, when, and why |
| Idempotency | Avoid duplicate writes on retries |

### Example: safe recommendation tool

```json
{
  "tool": "create_recommendation_event",
  "arguments": {
    "user_id": "current_user_only",
    "problem_slug": "coin-change",
    "reason_code": "weak_dynamic_programming",
    "idempotency_key": "user123-coin-change-2026-06-28"
  }
}
```

Before execution, backend should check:

```text
1. Is this the authenticated user?
2. Is this tool allowed for this user?
3. Is the problem valid?
4. Has the same recommendation already been created?
5. Does this action need user approval?
6. Can the event be safely logged?
```

---

## 1.7 Agent-tool interoperability

MCP solves the **N × M integration problem**.

Without MCP:

```text
N AI clients × M tools = N*M custom integrations
```

With MCP:

```text
Any MCP-compatible client can connect to any MCP-compatible server.
```

This matters because the same tool server can potentially work with:

- ChatGPT-style app
- Claude Desktop
- Cursor
- VS Code
- Internal agent runtime
- LangGraph orchestrator
- Enterprise assistant

### Example for an AI tutor product

```text
Crackedin AI Host
  ├── MCP Client → Interview Corpus MCP Server
  ├── MCP Client → User Progress MCP Server
  ├── MCP Client → Recommendation MCP Server
  └── MCP Client → LeetCode Sync MCP Server
```

Each server exposes only its own domain.

### Why split servers?

Do not build one “god MCP server” with access to everything.

Better:

| Server | Exposes |
|---|---|
| Interview Corpus MCP Server | Search interview data |
| User Progress MCP Server | Read learner progress |
| Recommendation MCP Server | Create/list recommendation events |
| LeetCode Sync MCP Server | Read submission sync state |
| Admin MCP Server | Admin-only tools |

This gives cleaner access boundaries and easier debugging.

---

## 1.8 How MCP is used in production

### Use case 1: Enterprise assistant

The assistant connects to:

- Google Drive
- Slack
- Jira
- GitHub
- Postgres
- Internal docs

User asks:

```text
Summarize open bugs related to the login service and draft a Slack update.
```

The agent can:

1. Search Jira issues.
2. Inspect GitHub PRs.
3. Read Slack threads.
4. Draft a response.
5. Ask for approval before sending.

### Use case 2: AI coding assistant

An IDE host connects to:

- File system server
- Git server
- Terminal server
- Documentation server

The model can:

- Search files
- Inspect code
- Propose patches
- Run safe tests
- Summarize diffs

### Use case 3: AI tutor/interview system

MCP can expose:

- Interview corpus search
- User progress resources
- DSA skill graph resources
- Recommendation tools
- LeetCode sync tools
- Study plan prompts

This allows different agents or clients to reuse the same integration layer.

---

## 1.9 MCP production failure modes

| Failure mode | Cause | Mitigation |
|---|---|---|
| Tool abuse | Model repeatedly calls expensive tools | Tool budget, rate limits |
| Data leakage | Resource returns data from wrong tenant | Auth checks, tenant filters |
| Arbitrary execution | Generic tools like `run_sql` or `bash` | Narrow tools, allowlists |
| Duplicate writes | Retries without idempotency | Idempotency keys |
| Hidden destructive action | Model performs action without user consent | Approval gates |
| Prompt injection | Tool result tells model to ignore instructions | Treat external data as untrusted |
| Poor observability | No trace of tool calls | Structured logs/tracing |
| Over-broad resources | Server returns too much data | Pagination, filtering, redaction |

---

## 1.10 Interview questions — MCP

### Q1. What is MCP and why is it useful?

**Answer:**  
MCP is a protocol for connecting AI applications to external tools, resources, and prompts. It standardizes how AI hosts discover and interact with external systems. Its value is interoperability: instead of building custom integrations between every AI app and every tool, you expose capabilities through MCP servers and connect MCP-compatible clients to them.

---

### Q2. What is the difference between MCP tools, resources, and prompts?

**Answer:**  

- **Tools** are callable actions, such as searching a database or creating a recommendation.
- **Resources** are readable context, such as files, user progress, schemas, or documents.
- **Prompts** are reusable workflow templates exposed by the server.

In production, I treat resources as mostly read-only, tools as controlled execution paths, and prompts as standardized workflows.

---

### Q3. Is MCP enough for security?

**Answer:**  
No. MCP gives a protocol-level structure, but application security still requires authentication, authorization, tenant isolation, permission scopes, tool allowlists, input validation, output filtering, rate limits, audit logs, and human approval for sensitive actions.

---

### Q4. How would you design an MCP server for an AI tutor?

**Answer:**  
I would split servers by domain: interview corpus, learner progress, recommendations, and platform sync. Each server would expose narrow resources and tools. For example, the recommendation server would expose `list_active_recommendations` as a resource and `create_recommendation_event` as a tool. Every call would include user identity, permission checks, idempotency keys, structured errors, and audit logs.

---

### Q5. Why should you avoid generic tools like `run_sql`?

**Answer:**  
Generic tools give the model too much power. A model can generate destructive, expensive, or unauthorized queries. Narrow domain tools are safer because they constrain inputs, permissions, and business logic.

---

# 2. OpenAI / Anthropic / Gemini APIs

## 2.1 What to understand for interviews

For OpenAI, Anthropic, and Gemini APIs, do not only memorize endpoint names.

Understand these production capabilities:

| Capability | Why it matters |
|---|---|
| Function calling / tool use | Lets models request external actions |
| Streaming | Improves perceived latency |
| Structured output | Makes responses machine-parseable |
| Embeddings | Enables semantic search and RAG |
| Multimodal input | Supports images, PDFs, audio, video |
| Rate limits | Required for stability |
| Retries | Required for transient API failures |
| Observability | Needed for debugging, cost, and quality |
| Fallbacks | Needed when model/API fails |

A strong answer:

> LLM APIs are not just completion endpoints. In production they become part of a controlled pipeline with schemas, tools, validation, retries, timeouts, streaming, telemetry, cost controls, and fallbacks.

---

## 2.2 Function calling / tool use

### What it is

Function calling means the model can request that your application call a function.

The model does not execute the function directly.

Example model output:

```json
{
  "tool_name": "search_interview_questions",
  "arguments": {
    "company": "Google",
    "role": "Backend Engineer",
    "topic": "System Design"
  }
}
```

Your backend then:

1. Parses the tool call.
2. Validates arguments.
3. Checks permissions.
4. Executes the function.
5. Returns the tool result to the model.
6. Asks the model to produce the final answer.

### Production flow

```text
User asks question
  ↓
Model decides tool is needed
  ↓
Model returns structured tool call
  ↓
Backend validates args
  ↓
Backend checks permissions
  ↓
Backend executes tool
  ↓
Tool result returned to model
  ↓
Model generates final answer
```

### Tool calling examples for an AI tutor

```text
search_interview_corpus(company, role, topic)
get_user_dsa_progress(user_id)
fetch_active_recommendations(user_id)
create_learning_event(user_id, event_type, topic_id)
retrieve_code_submission(user_id, problem_slug)
```

### Why function calling matters

Without tools, the model may hallucinate facts.

With tools, the model can access real systems:

- Database
- Search index
- Calendar
- Email
- Internal APIs
- Recommendation engine
- User progress store
- Code execution sandbox

---

## 2.3 Function calling production concerns

| Concern | What can go wrong | Mitigation |
|---|---|---|
| Invalid args | Model produces bad JSON or invalid fields | Schema validation |
| Tool hallucination | Model asks for non-existent tool | Registered tool allowlist |
| Dangerous action | Model tries destructive operation | Approval gates |
| Duplicate writes | Retry creates duplicate event | Idempotency keys |
| Slow tools | Tool blocks response | Timeout, async jobs |
| Tool loops | Model keeps calling tools | Max tool-call budget |
| Unauthorized access | Model requests another user's data | Permission checks |
| Sensitive output | Tool returns secrets or PII | Output filtering |
| Poor debugging | Cannot inspect tool path | Tracing/logging |

### Bad design

```text
tool: execute_anything(command: string)
```

### Better design

```text
tool: search_company_interview_patterns(company, role, topic)
tool: get_current_user_progress(topic_slug)
tool: create_study_plan_event(plan_id, action)
```

---

## 2.4 Streaming

### What streaming is

Streaming returns model output incrementally.

Instead of waiting for the full response:

```text
Wait 8 seconds → full response appears
```

The user sees partial output:

```text
0.8s → first tokens appear
2s → answer continues
8s → answer completes
```

### Why streaming matters

Streaming improves perceived latency.

A non-streamed 8-second response feels slow. A streamed 8-second response can feel acceptable because the user sees progress.

### Production streaming flow

```text
Backend calls model with streaming enabled
  ↓
Model emits events/tokens
  ↓
Backend forwards stream to frontend
  ↓
Frontend renders partial response
  ↓
Final event contains completion metadata
```

### Streaming use cases

Good for:

- Long explanations
- Tutoring answers
- Coding help
- Report generation
- Step-by-step reasoning summaries
- Chat UX

Less ideal for:

- Strict JSON used by backend logic
- Small classification tasks
- Workflows needing complete validation before display

### Streaming complexities

Streaming complicates production systems:

- Partial tokens
- Tool call deltas
- Cancel requests
- Network disconnects
- Final usage accounting
- Safety checks
- Post-stream validation
- Reconstructing final response
- Handling retries without duplicating output

### Practical rule

> Stream human-facing text. Prefer non-streamed calls for strict backend JSON unless the API provides reliable structured streaming support.

---

## 2.5 Structured output

### What it is

Structured output means asking the model to return data matching a schema.

Example:

```json
{
  "intent": "dsa_help",
  "topic": "dynamic_programming",
  "needs_retrieval": true,
  "depth": "deep_fast",
  "confidence": 0.86
}
```

This allows LLM outputs to be used by software.

### Where structured outputs are used

Use structured output for:

- Routing
- Classification
- Extraction
- Evaluation
- Recommendation generation
- Tool arguments
- Safety labels
- Agent state updates
- Query rewriting
- Data normalization

### Example: tutor router

```json
{
  "intent": "dsa_help",
  "sub_intent": "concept_explanation",
  "topic": "graphs",
  "difficulty": "medium",
  "needs_user_progress": true,
  "needs_retrieval": true,
  "response_depth": "deep"
}
```

### Validation pipeline

Never blindly trust model JSON.

```text
LLM output
  ↓
JSON parse
  ↓
Schema validation
  ↓
Business validation
  ↓
Retry/repair if invalid
  ↓
Fallback if repeated failure
```

### Valid JSON can still be wrong

Example:

```json
{
  "difficulty": "easy",
  "estimated_minutes": -40
}
```

This may pass JSON parsing but fail business validation.

### Business validation examples

| Field | Validation |
|---|---|
| `difficulty` | Must be `easy`, `medium`, or `hard` |
| `estimated_minutes` | Must be positive |
| `topic_slug` | Must exist in catalog |
| `user_id` | Must match authenticated user |
| `confidence` | Must be between 0 and 1 |

---

## 2.6 Embeddings

### What embeddings are

Embeddings convert text or other content into vectors.

Example:

```text
"Dynamic programming uses overlapping subproblems"
→ [0.12, -0.44, 0.91, ...]
```

Similar meanings should produce vectors close to each other.

### What embeddings are used for

- RAG retrieval
- Semantic search
- Similar question matching
- Duplicate detection
- Clustering
- Recommendation systems
- User memory retrieval
- Topic mapping
- Interview trend analysis

### Embedding flow in RAG

```text
Documents
  ↓
Chunk documents
  ↓
Generate embeddings
  ↓
Store vectors + metadata
  ↓
User query
  ↓
Embed query
  ↓
Vector search
  ↓
Rerank
  ↓
LLM answer with retrieved context
```

### Example in AI interview system

User asks:

```text
What graph questions are asked in Google backend interviews?
```

System:

```text
Embed query
  ↓
Search vector DB
  ↓
Filter company=Google, role=Backend, topic=Graphs
  ↓
Rerank results
  ↓
Generate answer with citations
```

---

## 2.7 Multimodal inputs

### What multimodal means

Multimodal models can process inputs beyond text.

Examples:

- Images
- Screenshots
- PDFs
- Audio
- Video
- Documents
- Tables
- Charts
- Mixed text + image

### Production use cases

| Input | Use case |
|---|---|
| Image | UI screenshot debugging |
| PDF | Resume/job description parsing |
| Chart | Explain trend or numbers |
| Audio | Mock interview transcription |
| Video | Lecture or interview analysis |
| Diagram | System design review |
| Code screenshot | Explain or debug code |

### Example for interview prep

A user uploads a system design diagram.

The model can identify:

- Services
- Databases
- Queues
- Caches
- Bottlenecks
- Missing retries
- Missing DLQ
- Single points of failure
- Scaling risks

### Production risks

| Risk | Mitigation |
|---|---|
| High cost | File limits, sampling, caching |
| High latency | Preprocessing, async jobs |
| Sensitive documents | Consent, redaction, deletion controls |
| OCR errors | Combine native extraction + vision |
| Hallucinated visual details | Ask model to cite image regions/pages |
| Large files | Chunking/page-level processing |
| Security | MIME validation, malware scanning |

---

## 2.8 OpenAI vs Anthropic vs Gemini — practical lens

| Area | OpenAI | Anthropic | Gemini |
|---|---|---|---|
| Tool calling | Strong function/tool calling support | Strong tool-use model | Function calling/tools |
| Streaming | Strong SSE streaming | Strong streaming | Strong streaming |
| Structured output | Strong schema-oriented workflows | Good tool/schema patterns | Good structured output support |
| Embeddings | First-party embedding models | Often used with external embeddings | Strong text/multimodal embeddings |
| Multimodal | Text/image/audio depending on model | Strong text + image/PDF workflows | Strong image/audio/video/document support |
| Typical strength | Broad platform maturity | Reasoning and safety patterns | Multimodal and long context |

### Practical model selection rule

Do not pick a provider only by benchmark.

Evaluate:

- Quality on your golden dataset
- Latency
- Cost
- Context length
- Tool-calling reliability
- Structured output validity
- Streaming UX
- Rate limits
- Data retention/privacy requirements
- Region/compliance needs
- Fallback availability

---

## 2.9 Interview questions — LLM APIs

### Q1. What is function calling and how is it different from prompting?

**Answer:**  
Prompting asks the model to answer directly using prompt context and internal knowledge. Function calling lets the model request an external action through structured arguments. The application validates and executes the function, then returns the result to the model. This is safer and more factual because external data comes from real systems.

---

### Q2. How do you make tool calling safe in production?

**Answer:**  
Use strict schemas, tool allowlists, argument validation, permission checks, timeouts, rate limits, idempotency keys, audit logs, and human approval for sensitive actions. Avoid generic tools like `run_sql` and expose narrow domain tools instead.

---

### Q3. Why is streaming useful?

**Answer:**  
Streaming improves perceived latency by sending partial output as the model generates it. It is especially useful for long chat responses, tutoring, and code explanations. However, it complicates backend handling because of partial events, disconnects, cancellation, final usage accounting, and validation.

---

### Q4. When should you use structured outputs?

**Answer:**  
Use structured outputs when the result is consumed by software, such as routing, extraction, classification, tool arguments, recommendation generation, or eval scoring. Always validate structured outputs with schema and business rules.

---

### Q5. What are embeddings used for in RAG?

**Answer:**  
Embeddings convert documents and queries into vectors. At query time, the system embeds the query, retrieves nearby document vectors, optionally applies metadata filters and reranking, and passes the retrieved context to the LLM. This grounds the answer in application data.

---

# 3. Vector Databases

## 3.1 What a vector DB does

A vector database stores embeddings and supports similarity search.

It answers questions like:

```text
Which stored chunks are semantically closest to this query?
```

In RAG, the flow is:

```text
Documents
  ↓
Chunking
  ↓
Embedding
  ↓
Store vectors + metadata
  ↓
User query
  ↓
Query embedding
  ↓
Nearest-neighbor search
  ↓
Metadata filtering
  ↓
Reranking
  ↓
LLM answer with citations
```

Important point:

> A vector database is not magic memory. Retrieval quality depends on chunking, metadata, embedding quality, filters, reranking, and evals.

---

## 3.2 Core vector DB concepts

### Vector

A vector is a numeric representation of content.

```text
"Graph traversal uses BFS or DFS"
→ [0.21, -0.77, 0.35, ...]
```

### Similarity metric

Common metrics:

| Metric | Meaning |
|---|---|
| Cosine similarity | Measures direction similarity |
| Dot product | Often used for normalized embeddings |
| Euclidean/L2 distance | Measures geometric distance |

Use the metric recommended by the embedding model.

### Metadata

Metadata is structured data stored with vectors.

Example:

```json
{
  "company": "Google",
  "role": "Backend Engineer",
  "topic": "Graphs",
  "difficulty": "Medium",
  "source": "interview_experience",
  "created_at": "2026-06-01"
}
```

### ANN index

ANN means **Approximate Nearest Neighbor**.

Instead of scanning every vector exactly, the database uses an index to quickly find likely nearest vectors.

Common indexes:

| Index | Best for | Tradeoff |
|---|---|---|
| HNSW | Fast high-recall search | More memory |
| IVF/IVFFlat | Large datasets | Needs tuning |
| Flat/brute force | Small datasets, exact search | Slow at scale |
| Quantized indexes | Lower memory | Possible recall loss |

---

## 3.3 pgvector

pgvector adds vector search to PostgreSQL.

### Why pgvector is attractive

It lets you store:

- Relational data
- Metadata
- Vectors
- User data
- Application data

inside the same Postgres system.

### Example table

```sql
CREATE TABLE interview_chunks (
  id BIGSERIAL PRIMARY KEY,
  company TEXT,
  role TEXT,
  topic TEXT,
  source_id BIGINT,
  chunk_text TEXT,
  embedding VECTOR(1536),
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Example query

```sql
SELECT *
FROM interview_chunks
WHERE company = 'Google'
  AND topic = 'Graphs'
ORDER BY embedding <=> :query_embedding
LIMIT 10;
```

### When pgvector is a good choice

Use pgvector when:

- You already use Postgres.
- Dataset is small to medium.
- You need relational joins.
- You need simple infrastructure.
- You want transactional consistency.
- You are building an MVP or early production version.

### Production strengths

| Strength | Explanation |
|---|---|
| Simplicity | One database instead of two |
| Joins | Easy joins with users/topics/progress |
| Transactions | Vector rows can update with app data |
| Backups | Same backup process as Postgres |
| MVP-friendly | Fewer moving parts |

### Production risks

| Risk | Explanation |
|---|---|
| Scaling limits | Very large vector workloads can stress Postgres |
| QPS pressure | OLTP and retrieval compete for resources |
| ANN tuning | Index choices matter |
| Filtering complexity | Filtered vector search can get expensive |
| Operational coupling | DB handles both app data and retrieval |

### Practical advice

Start with pgvector if it meets your requirements.

Move to a dedicated vector DB only when metrics show bottlenecks.

---

## 3.4 Pinecone

Pinecone is a managed vector database.

### When Pinecone is useful

Use Pinecone when:

- You want managed vector infrastructure.
- You expect high QPS.
- You do not want to operate vector DB infrastructure.
- You need namespaces for tenant/domain separation.
- You care about production reliability and scaling.

### Strengths

| Strength | Explanation |
|---|---|
| Managed | Less operational burden |
| Scale | Designed for vector workloads |
| Namespaces | Useful for tenant or dataset isolation |
| Metadata filtering | Built into search flow |
| Availability | Managed reliability features |

### Risks

| Risk | Explanation |
|---|---|
| Cost | Managed infra can be expensive |
| Vendor lock-in | Migration may be non-trivial |
| Source of truth | You still need Postgres/app DB |
| Consistency | Need sync between primary DB and Pinecone |
| Debugging | More distributed system complexity |

### Production pattern

```text
Postgres = source of truth
Pinecone = retrieval index
Event/outbox = keeps Pinecone in sync
```

---

## 3.5 Weaviate

Weaviate is a vector database with schema-aware object storage, vector indexing, and hybrid search.

### When Weaviate is useful

Use Weaviate when:

- You want a full-featured vector search engine.
- You need hybrid search.
- You want schema-aware objects.
- You want self-hosted or managed deployment.
- You need more vector-specific features than pgvector.

### Strengths

| Strength | Explanation |
|---|---|
| Hybrid search | Combines lexical and vector retrieval |
| Schema model | Objects/classes can be modeled |
| ANN indexes | HNSW and other index options |
| Managed/self-hosted | Flexible deployment |
| RAG-focused | Designed for AI retrieval use cases |

### Risks

| Risk | Explanation |
|---|---|
| Operational complexity | More services to run |
| Schema design | Poor schema hurts retrieval |
| Memory tuning | HNSW can be memory-heavy |
| Sync consistency | Still needs source-of-truth sync |

---

## 3.6 Qdrant

Qdrant is a high-performance vector database written in Rust.

It stores vectors as **points** with associated **payloads**.

### When Qdrant is useful

Use Qdrant when:

- You want strong metadata/payload filtering.
- You want open-source/self-hosted vector search.
- You care about performance.
- You need clean vector + payload APIs.
- You want a production-ready dedicated vector engine.

### Strengths

| Strength | Explanation |
|---|---|
| Payload filtering | Strong metadata filtering story |
| Performance | Built for vector search workloads |
| Open source | Self-hosting possible |
| Clean API | Good developer ergonomics |
| Production-ready | Suitable beyond prototypes |

### Risks

| Risk | Explanation |
|---|---|
| Extra infra | Another database to run |
| Sync consistency | Need sync from Postgres |
| Backup/rebuild | Need index rebuild process |
| Operational skill | Team must understand vector infra |

---

## 3.7 Chroma

Chroma is an open-source embedding/vector database popular for prototypes and local AI apps.

### When Chroma is useful

Use Chroma when:

- You are prototyping.
- You need quick local development.
- You are building demos.
- You want simple collection APIs.
- You do not need high-scale production guarantees yet.

### Strengths

| Strength | Explanation |
|---|---|
| Simple | Easy to start |
| Local-friendly | Good for development |
| AI-native API | Designed around embeddings |
| Metadata filters | Useful for basic RAG |

### Risks

| Risk | Explanation |
|---|---|
| Scale | May not be best for high-scale production |
| Multi-tenancy | Needs careful design |
| Durability | Compare carefully before mission-critical use |
| Migration | May need later move to pgvector/Qdrant/Weaviate/Pinecone |

---

## 3.8 Metadata filtering

Metadata filtering restricts vector search using structured fields.

Example:

```text
Find semantically similar chunks
WHERE company = "Google"
AND role = "Backend Engineer"
AND topic = "Graphs"
```

### Why it matters

Without metadata filtering, vector search may return semantically related but contextually wrong chunks.

Bad retrieval:

```text
Query: Google backend graph interview questions

Returned:
- Amazon graph question
- Google frontend question
- Generic graph tutorial
- Old irrelevant forum thread
```

Good retrieval:

```text
Filter:
company = Google
role = Backend
topic = Graphs
source_type = interview_experience

Then vector search within relevant candidates.
```

### Filtering strategies

| Strategy | How it works | Risk |
|---|---|---|
| Pre-filter | Filter first, then vector search | Candidate pool may become too small |
| Post-filter | Vector search first, then filter | May return too few valid results |
| Integrated filtering | DB combines vector search + filters | DB-specific behavior |
| Hybrid retrieval | BM25 + vector + metadata + rerank | More complex but stronger |

---

## 3.9 ANN indexes

### HNSW

HNSW means **Hierarchical Navigable Small World**.

It builds a graph of nearby vectors.

Search navigates the graph instead of scanning all vectors.

#### Strengths

- Fast
- High recall
- Common default
- Good for many production systems

#### Weaknesses

- Memory-heavy
- Index build can be expensive
- Tuning matters

Important tuning concepts:

| Parameter | Meaning |
|---|---|
| `M` | Number of graph connections |
| `ef_construction` | Build-time search depth |
| `ef_search` | Query-time search depth |

Higher values usually improve recall but increase memory/latency.

---

### IVFFlat / IVF

IVF means **Inverted File Index**.

It clusters vectors and searches selected clusters.

#### Strengths

- Can work well on large datasets
- Can reduce search space
- Lower memory than HNSW in some cases

#### Weaknesses

- Needs training/clustering
- Recall depends on probe count
- Poor tuning can miss relevant results

---

### Flat search

Flat search compares the query against every vector.

#### Strengths

- Exact
- Simple
- Good for small datasets
- Useful as baseline for eval

#### Weaknesses

- Slow at scale
- Expensive for large datasets

---

## 3.10 Choosing a vector DB

### MVP / early production

Use **Postgres + pgvector** when:

- You already use Postgres.
- Dataset is manageable.
- You need relational filtering.
- You want fewer moving parts.
- You need fast iteration.

### Scaling phase

Move to **Qdrant** or **Weaviate** when:

- Vector QPS grows.
- Filtering becomes expensive.
- Retrieval workload affects OLTP database.
- ANN performance matters more.
- You need dedicated vector scaling.

### Managed scale

Use **Pinecone** when:

- You want managed infrastructure.
- You can pay for convenience.
- You need production vector scaling quickly.
- You want to avoid operating vector infra.

### Prototyping

Use **Chroma** when:

- You are experimenting.
- You need local RAG.
- You are validating retrieval workflows.
- Production scale is not the immediate concern.

---

## 3.11 Common RAG retrieval failure modes

| Failure | Cause | Fix |
|---|---|---|
| Irrelevant chunks | Weak query or no filters | Query rewrite, metadata filters |
| Missing answer | Bad chunking or indexing | Better chunking, overlap, hierarchy |
| Too much context | Large irrelevant retrieval | Reranking, top-k tuning |
| Old data | Stale index | Freshness metadata, reindexing |
| Duplicate chunks | Poor ingestion | Deduplication |
| Cross-user leakage | Missing permission filters | Tenant-aware retrieval |
| Hallucinated answer | Weak grounding | Citations, faithfulness eval |
| Slow retrieval | Bad index or too many filters | Index tuning, caching |
| Poor exact keyword match | Vector-only retrieval | Hybrid BM25 + vector |

---

## 3.12 Interview questions — Vector DBs

### Q1. What is a vector database and why is it used in RAG?

**Answer:**  
A vector database stores embeddings and supports similarity search. In RAG, documents are chunked, embedded, stored with metadata, and retrieved at query time based on semantic similarity. Retrieved chunks are passed to the LLM as grounding context, reducing hallucination and allowing answers over private or changing data.

---

### Q2. What is the difference between pgvector and Pinecone/Qdrant/Weaviate?

**Answer:**  
pgvector brings vector search into Postgres, which is great for simplicity, relational joins, and transactional consistency. Pinecone, Qdrant, and Weaviate are dedicated vector databases, usually better for high vector QPS, large-scale ANN search, and advanced filtering. I would start with pgvector for MVP and move only when metrics justify it.

---

### Q3. What is metadata filtering and why is it important?

**Answer:**  
Metadata filtering restricts retrieval using structured fields like company, role, topic, user ID, document type, date, or permission. It improves relevance and prevents data leakage. In production RAG, metadata filtering is essential for both accuracy and access control.

---

### Q4. What is HNSW?

**Answer:**  
HNSW is a graph-based approximate nearest neighbor index. It connects nearby vectors in a navigable graph, allowing fast search without scanning all vectors. It usually gives strong recall and latency but uses more memory and needs tuning.

---

### Q5. Why can RAG retrieval fail?

**Answer:**  
RAG can fail due to bad chunking, weak embeddings, missing metadata, no reranking, overly broad retrieval, overly strict filters, stale documents, duplicate chunks, poor query rewriting, or no permission filtering. Strong production RAG usually combines vector search, BM25/hybrid search, metadata filters, reranking, citations, evals, and regression tests.

---

# 4. Fine-tuning vs RAG

## 4.1 Core difference

**RAG** gives the model external knowledge at inference time.  
**Fine-tuning** changes model behavior through training.

Simple interview answer:

> Use RAG when the model needs new, changing, private, or user-specific facts. Use fine-tuning when the model already has the needed information but needs to behave more consistently.

---

## 4.2 What RAG is

RAG stands for **Retrieval-Augmented Generation**.

The system retrieves relevant context and gives it to the model.

```text
User query
  ↓
Retrieve relevant documents/chunks
  ↓
Pass retrieved context to LLM
  ↓
Generate grounded answer
```

RAG is an inference-time architecture.

The model weights do not change.

---

## 4.3 When to use RAG

Use RAG when the issue is missing or changing knowledge.

### Good RAG cases

| Use case | Why RAG is better |
|---|---|
| Company interview questions | Data changes over time |
| User progress | User-specific and dynamic |
| Internal docs | Need citations/source traceability |
| Product pricing | Frequently changing |
| Legal/policy docs | Need freshness and auditability |
| Support knowledge base | Large and frequently updated |
| Interview corpus | Too large and dynamic for weights |

### In an AI tutor system

Use RAG for:

- Interview experience search
- Company-wise interview trends
- DSA explanations from curated content
- User-specific progress
- Study plan grounding
- Tutor memory retrieval
- Source-backed answers

---

## 4.4 When not to use RAG

RAG may not fix:

- Bad output format
- Poor classification behavior
- Inconsistent tone
- Weak tool-use decisions
- Bad extraction style
- Poor adherence to a rubric

Those are often behavior problems, not knowledge problems.

---

## 4.5 What fine-tuning is

Fine-tuning trains a model further on task-specific examples.

The model weights or adapter weights are updated.

```text
Training examples
  ↓
Fine-tuning process
  ↓
Adapted model
```

Fine-tuning is useful when you want the model to learn a pattern.

---

## 4.6 When to fine-tune

Use fine-tuning when the issue is behavioral consistency.

### Good fine-tuning cases

| Use case | Why fine-tuning helps |
|---|---|
| Structured extraction | Learns output pattern |
| Query classification | Learns decision boundary |
| Custom grading rubric | Learns scoring behavior |
| Tool selection | Learns when to call which tool |
| Consistent tone/style | Reduces prompt complexity |
| Repeated short tasks | May reduce latency/cost |
| Domain-specific normalization | Learns mapping rules |

### In an AI tutor system

Fine-tune for:

- Intent classification
- Interview round extraction
- Query routing
- Tutor feedback style
- Explanation grading
- Weakness label generation
- Structured data normalization

---

## 4.7 When not to fine-tune

Do not fine-tune when:

- You do not have an eval set.
- The problem is stale/missing knowledge.
- The issue is bad retrieval.
- The issue is poor prompt design.
- The issue is wrong source data.
- The target behavior is not clearly defined.
- The dataset is noisy or too small.
- You need citations and source traceability.

Fine-tuning is not a replacement for retrieval when facts change.

---

## 4.8 Decision rule

```text
Does the model need new facts?
  → Use RAG.

Does the model need consistent behavior?
  → Fine-tune.

Does it need both?
  → Use RAG + fine-tuned model.

Do you lack evals?
  → Do not fine-tune yet.
```

---

## 4.9 RAG vs fine-tuning comparison

| Dimension | RAG | Fine-tuning |
|---|---|---|
| Best for | Knowledge injection | Behavior adaptation |
| Freshness | Easy to update | Requires retraining |
| Citations | Strong | Weak |
| Debugging | Inspect retrieval | Harder to inspect |
| Cost | Retrieval + longer prompts | Training cost, possibly cheaper inference |
| Latency | Retrieval adds latency | Can reduce prompt length |
| Rollback | Change index/config | Revert model version |
| Data need | Documents/chunks | High-quality labeled examples |
| User-specific data | Excellent | Usually inappropriate |
| Dynamic facts | Excellent | Poor |
| Repeated format | Moderate | Strong |

---

## 4.10 RAG + fine-tuning together

Many production systems use both.

Example:

```text
RAG:
  Retrieves current user progress and interview corpus.

Fine-tuned model:
  Follows tutor rubric and produces consistent structured feedback.
```

Architecture:

```text
User query
  ↓
Router/classifier
  ↓
RAG retrieval
  ↓
Fine-tuned response model
  ↓
Validated answer
```

---

## 4.11 LoRA

LoRA means **Low-Rank Adaptation**.

Instead of updating all model weights, LoRA freezes the base model and trains small low-rank adapter matrices.

### Full fine-tuning

```text
Update all model parameters.
```

### LoRA

```text
Freeze base model.
Train small adapter matrices.
Load/merge adapter during inference.
```

### Why LoRA matters

LoRA is useful when:

- You use open-source models.
- You want cheaper fine-tuning.
- You need multiple task/domain variants.
- You cannot afford full training.
- You want swappable domain adapters.

Example:

```text
Base model:
  Llama / Mistral / Qwen

Adapters:
  - DSA tutor adapter
  - Interview extraction adapter
  - Resume parser adapter
  - Mock interview adapter
```

### Pros

- Lower training cost
- Lower GPU memory
- Easier versioning
- Multiple adapters possible
- Faster experimentation

### Cons

- May underperform full fine-tuning for some tasks
- Adapter management adds complexity
- Serving multiple adapters can complicate infra
- Not relevant if you only use closed hosted APIs

---

## 4.12 Adapters

Adapters are small trainable modules added to a frozen base model.

LoRA is one adapter-style technique.

General idea:

```text
Base model remains mostly unchanged.
Small trainable modules learn task-specific behavior.
```

### Adapter benefits

- Efficient training
- Modular behavior
- Easier rollback
- Multiple domain specializations
- Lower risk than full fine-tuning

---

## 4.13 Synthetic data

Synthetic data is model-generated training data.

Example:

```json
{
  "input": "I keep failing DP tabulation problems",
  "expected_output": {
    "intent": "weakness_analysis",
    "topic": "dynamic_programming",
    "recommended_action": "review bottom-up state transitions"
  }
}
```

### When synthetic data helps

- Bootstrapping a classifier
- Creating formatting examples
- Generating edge cases
- Expanding sparse datasets
- Creating negative examples
- Simulating user queries
- Training extraction patterns

### Risks

| Risk | Explanation |
|---|---|
| Too clean | Synthetic examples may not match real users |
| Wrong assumptions | Generated labels may be incorrect |
| Overfitting | Model learns artificial patterns |
| Hallucinated facts | Synthetic data may contain false content |
| Distribution mismatch | Real traffic behaves differently |
| Bias amplification | Generator model's bias enters dataset |

### Better production pattern

```text
Real examples
  + synthetic augmentation
  + human review for high-impact examples
  + eval set from real traffic
  + regression tests
```

Synthetic data should augment real data, not replace it entirely.

---

## 4.14 Eval before training

This is one of the most important production points.

Do not fine-tune because “quality feels low.”

First build evals.

### Eval-before-training pipeline

```text
1. Collect real failure cases.
2. Label expected output.
3. Build golden dataset.
4. Run baseline model.
5. Categorize errors.
6. Try prompt/RAG/tool/schema improvements.
7. Fine-tune only if failure is behavioral.
8. Re-run eval.
9. Compare quality, latency, cost, and failure modes.
```

### What to measure

| Task | Metrics |
|---|---|
| Classification | Accuracy, precision, recall, confusion matrix |
| Extraction | Field-level F1, exact match, invalid JSON rate |
| RAG answer | Faithfulness, answer relevance, citation accuracy |
| Tutor feedback | Rubric score, usefulness, hallucination rate |
| Tool calling | Correct tool rate, invalid args, unnecessary tool calls |
| Production | Latency, cost, retry rate, user satisfaction |

### Why evals matter

Fine-tuning can make the model worse if:

- Data is noisy.
- Failure is actually retrieval-related.
- Labels are inconsistent.
- Training examples do not match production distribution.
- The model overfits to synthetic examples.
- The output behavior is not clearly defined.

---

## 4.15 Interview questions — Fine-tuning vs RAG

### Q1. When would you use RAG instead of fine-tuning?

**Answer:**  
Use RAG when the model needs access to external, private, changing, or user-specific knowledge. Examples include company interview experiences, user progress, pricing, internal docs, or recent data. RAG keeps knowledge outside the model and retrieves it at inference time.

---

### Q2. When would you fine-tune instead of using RAG?

**Answer:**  
Fine-tune when the model already has enough knowledge but does not behave consistently. Examples include structured extraction, classification, custom grading, consistent tone, output formatting, and tool-selection behavior.

---

### Q3. Can RAG and fine-tuning be used together?

**Answer:**  
Yes. A common production pattern is using RAG for dynamic knowledge and fine-tuning for behavior. For example, an AI tutor can retrieve user progress and interview data with RAG while using a fine-tuned model to follow a consistent tutoring rubric.

---

### Q4. Why should you run evals before fine-tuning?

**Answer:**  
Because fine-tuning is expensive and can worsen performance if the real issue is retrieval, prompt design, tool calling, or bad data. Evals identify the failure mode and provide a baseline to prove whether fine-tuning improved quality.

---

### Q5. What is LoRA?

**Answer:**  
LoRA is Low-Rank Adaptation. It freezes the base model and trains small low-rank adapter matrices instead of updating all parameters. It reduces training cost and memory and allows task-specific adapters to be swapped or versioned.

---

# 5. Framework Alternatives

## 5.1 Big-picture comparison

These frameworks are not all direct replacements for one another.

They solve different layers of the AI application stack.

| Framework | Best for | Avoid when |
|---|---|---|
| LlamaIndex | RAG, indexing, data connectors, retrieval workflows | You only need simple API calls |
| Semantic Kernel | Enterprise orchestration, plugins, Microsoft ecosystem | You want lightweight experimentation |
| PydanticAI | Type-safe agents and structured outputs | You need complex multi-agent simulation |
| CrewAI | Role-based multi-agent workflows | Simple deterministic workflows |
| AutoGen | Research-style multi-agent conversations | You need strict production control from day one |
| DSPy | Optimizing prompts/programs with metrics | You do not have evals/metrics |

---

## 5.2 LlamaIndex

LlamaIndex is strongest for **data + RAG**.

It helps with:

- Document loading
- Indexing
- Chunking
- Query engines
- Chat engines
- Data connectors
- Retrieval workflows
- RAG evaluation patterns

### Production use

Use LlamaIndex when the system is retrieval-heavy.

Example:

```text
Interview corpus
  ↓
Load documents
  ↓
Chunk with metadata
  ↓
Index into vector DB
  ↓
Retrieve by company/topic/role
  ↓
Rerank
  ↓
Generate answer with citations
```

### In an AI tutor product

LlamaIndex can help with:

- Interview corpus retrieval
- Company-specific search
- Topic-wise explanation retrieval
- RAG over uploaded documents
- Retrieval pipeline experimentation

### Strengths

| Strength | Explanation |
|---|---|
| RAG-focused | Many retrieval abstractions |
| Data connectors | Easier ingestion |
| Query engines | Faster prototyping |
| Index abstractions | Convenient for RAG workflows |

### Risks

| Risk | Explanation |
|---|---|
| Abstraction leakage | Hidden retrieval decisions can hurt quality |
| Latency | Framework defaults may not be optimized |
| Debugging | Need to inspect actual chunks/results |
| Vendor lock-in | Harder migration if deeply coupled |

### Production advice

Use LlamaIndex to move fast, but do not treat it as a black box. Inspect:

- Chunking
- Metadata
- Retrieved results
- Reranking
- Token usage
- Latency
- Eval scores

---

## 5.3 Semantic Kernel

Semantic Kernel is Microsoft’s AI orchestration framework.

It focuses on:

- Plugins
- Connectors
- Function orchestration
- Enterprise integrations
- Planner-style workflows
- Microsoft/Azure ecosystem compatibility

### Production use

Semantic Kernel fits when:

- You are in .NET/Azure/Microsoft ecosystem.
- You need plugin-style function orchestration.
- You need enterprise connectors.
- You want maintainable AI workflows.

Example:

```text
Enterprise AI assistant
  ├── Calendar plugin
  ├── Email plugin
  ├── CRM plugin
  ├── Search plugin
  └── Database plugin
```

### Strengths

| Strength | Explanation |
|---|---|
| Enterprise-friendly | Fits Microsoft stack |
| Plugin model | Wraps existing APIs |
| Orchestration | Useful for business workflows |
| Multi-language ecosystem | Good for .NET/Python/Java environments |

### Risks

| Risk | Explanation |
|---|---|
| Heavier stack | May be too much for startup MVP |
| Ecosystem bias | Best if using Microsoft tools |
| Complexity | Overkill for simple agent apps |

---

## 5.4 PydanticAI

PydanticAI is focused on typed Python agents and structured validation.

It fits backend engineering well because it emphasizes:

- Pydantic models
- Type-safe outputs
- Validation
- Dependency injection
- Tools
- Retries on validation failure
- Testable agent behavior

### Example

```python
from pydantic import BaseModel
from typing import Literal

class TutorRoute(BaseModel):
    intent: Literal["dsa", "interview", "system_design"]
    needs_retrieval: bool
    depth: Literal["fast", "deep"]
```

The model must return a valid `TutorRoute`.

### Production use

Use PydanticAI for:

- Routing
- Extraction
- Classification
- Structured backend workflows
- Tool-calling with validation
- Typed agent outputs

### Why it is strong

It matches production backend thinking:

```text
LLM output
  ↓
Pydantic validation
  ↓
Retry if invalid
  ↓
Business validation
  ↓
Tool execution
```

### Strengths

| Strength | Explanation |
|---|---|
| Type safety | Better backend correctness |
| Validation | Catches bad outputs |
| Python-native | Fits Python services |
| Testing | Easier to test structured behavior |

### Risks

| Risk | Explanation |
|---|---|
| Less multi-agent focused | Not ideal for complex agent societies |
| Python-specific | Less useful if stack is not Python |
| Still needs orchestration | Validation is not full workflow management |

---

## 5.5 CrewAI

CrewAI is designed for role-based multi-agent workflows.

It models:

- Agents
- Roles
- Tasks
- Crews
- Flows

### Example

```text
Researcher agent
  → retrieves sources

Analyst agent
  → compares patterns

Writer agent
  → creates final report

Reviewer agent
  → checks quality
```

### Production use

CrewAI can help with:

- Deep research reports
- Automated content generation
- Multi-agent analysis
- Company interview trend reports
- Workflow decomposition

### Strengths

| Strength | Explanation |
|---|---|
| Role-based design | Easy to conceptualize |
| Multi-step workflows | Useful for research tasks |
| Agent collaboration | Good for non-latency-sensitive work |
| Fast prototyping | Quick multi-agent demos |

### Risks

| Risk | Explanation |
|---|---|
| Latency | Multiple agents increase time |
| Cost | More LLM calls |
| Nondeterminism | Agents can drift |
| Debugging | Harder than deterministic workflows |
| Overengineering | Many tasks do not need agents |

### Production advice

Use CrewAI for offline/deep workflows, not your critical low-latency chat path unless necessary.

---

## 5.6 AutoGen

AutoGen is a framework for multi-agent AI systems.

It is useful for:

- Conversational agents
- Agent-to-agent collaboration
- Human-in-the-loop workflows
- Tool-using agents
- Research-style automation
- Coding/review/execution loops

### Example

```text
Planner agent
  ↓
Coder agent
  ↓
Reviewer agent
  ↓
Executor agent
  ↓
Human approval
```

### Strengths

| Strength | Explanation |
|---|---|
| Multi-agent conversations | Good for complex task decomposition |
| Human-in-loop | Useful for review workflows |
| Tool integration | Agents can use tools |
| Research-friendly | Good for experimentation |

### Risks

| Risk | Explanation |
|---|---|
| Agent loops | Conversations can run too long |
| Drift | Agents may lose task focus |
| Cost | Many model calls |
| Debugging | Hard to trace failures |
| Production control | Needs strict limits |

### Production guardrails

If using AutoGen in production:

- Set max turns
- Set tool budgets
- Define stop conditions
- Persist state
- Log every tool call
- Require approval for writes
- Add timeouts
- Add failure fallback

---

## 5.7 DSPy

DSPy is different from the others.

It focuses on programming and optimizing LLM workflows using metrics.

Instead of manually prompt-engineering, you define modules and metrics, then optimize.

### Example workflow

```text
Retrieve → Generate → Judge
```

DSPy can optimize instructions/examples for each step based on your metric.

### When DSPy is useful

Use DSPy when:

- You have eval data.
- You want systematic prompt optimization.
- You want to optimize RAG pipelines.
- You want smaller models to perform better.
- You want less manual prompt hacking.

### Strengths

| Strength | Explanation |
|---|---|
| Metric-driven | Optimizes against evals |
| Less manual prompting | Systematic improvement |
| RAG optimization | Useful for retrieval/generation workflows |
| Modular | Treats LLM pipelines as programs |

### Risks

| Risk | Explanation |
|---|---|
| Requires good metrics | Weak metrics produce weak optimization |
| Overfitting | Can overfit to eval set |
| Learning curve | Different mental model |
| Not magic | Still needs good data and evaluation |

---

## 5.8 Framework choice for an AI tutor system

Recommended practical stack:

| Component | Suggested tool |
|---|---|
| Core tutor orchestrator | LangGraph/custom state machine |
| RAG over interview corpus | LlamaIndex or custom retrieval |
| Typed routing/output | PydanticAI or custom Pydantic layer |
| Enterprise plugin workflows | Semantic Kernel |
| Deep research reports | CrewAI or AutoGen if needed |
| Prompt/RAG optimization | DSPy after evals exist |

### Strategic recommendation

For core user-facing tutoring:

```text
Use deterministic orchestration first.
Add multi-agent workflows only where they create measurable value.
```

Do not start with many agents unless the task truly requires it.

---

## 5.9 Interview questions — Framework alternatives

### Q1. How is LlamaIndex different from LangChain?

**Answer:**  
LlamaIndex is more focused on data ingestion, indexing, retrieval, and RAG workflows. LangChain is broader and covers chains, agents, tools, prompts, and integrations. I would choose LlamaIndex when retrieval quality and data indexing are central.

---

### Q2. When would you use PydanticAI?

**Answer:**  
I would use PydanticAI when I need type-safe outputs, schema validation, and backend correctness. It is useful for routing, extraction, classification, and workflows where invalid model output must be caught before business logic runs.

---

### Q3. When are CrewAI or AutoGen useful?

**Answer:**  
They are useful when the task naturally requires multiple specialized agents, such as research, coding-review loops, or complex analysis. However, they increase latency, cost, nondeterminism, and debugging complexity, so I would avoid them for simple production chat.

---

### Q4. What makes DSPy different?

**Answer:**  
DSPy treats LLM applications as optimizable programs rather than hand-written prompts. You define modules and metrics, then DSPy optimizes instructions/examples. It is powerful when you have eval sets, but not useful if your metrics are weak.

---

### Q5. How would you choose an AI framework for production?

**Answer:**  
I would choose based on the problem layer: LlamaIndex for RAG, PydanticAI for typed structured workflows, Semantic Kernel for enterprise plugin orchestration, DSPy for metric-driven optimization, and multi-agent frameworks only for workflows that genuinely require agent collaboration. I would avoid adding framework complexity before measuring need.

---

# 6. Model Internals

## 6.1 Why model internals matter for production engineers

You usually do not implement transformers in backend AI product roles.

But you must understand their consequences:

| Internal concept | Production consequence |
|---|---|
| Tokenization | Cost, latency, chunking |
| Context window | Long prompts cost more and may dilute attention |
| Attention | Model can miss details in long contexts |
| Next-token prediction | Hallucination risk |
| Pretraining | Knowledge is broad but stale |
| Instruction tuning | Better assistant behavior |
| RLHF/DPO | Safety/refusal/preference behavior |
| Sampling | Determinism vs creativity |

---

## 6.2 Transformer basics

Modern LLMs are based on the Transformer architecture.

A decoder-only LLM roughly works like this:

```text
Input text
  ↓
Tokenization
  ↓
Token embeddings
  ↓
Positional information
  ↓
Transformer blocks
    ├── self-attention
    ├── feed-forward network
    ├── residual connections
    └── normalization
  ↓
Logits over vocabulary
  ↓
Next-token prediction
```

The model predicts the next token repeatedly.

```text
Input: "A binary search tree has"
Model predicts: "left"
Then predicts next token...
```

---

## 6.3 Tokens

Models do not read words directly. They read tokens.

Example:

```text
"dynamic programming"
→ ["dynamic", " programming"]
```

Tokenization affects:

- Cost
- Latency
- Context length
- Chunk size
- Multilingual quality
- Code handling
- Streaming granularity

### Production consequence

If your prompt is too long, you pay more and increase latency.

For RAG, chunking should consider token count, not character count.

---

## 6.4 Embeddings inside models

Tokens are converted into vectors called embeddings.

```text
Token ID
  ↓
Embedding lookup
  ↓
Vector representation
```

These vectors are then processed by transformer layers.

Embeddings inside the LLM are not the same as API embeddings used for vector search, but the concept is similar: represent discrete content in numeric space.

---

## 6.5 Positional information

Transformers need positional information because attention itself does not naturally know token order.

Without positions:

```text
"dog bites man"
"man bites dog"
```

could look similar.

Position encodings/embeddings help the model understand order.

Production consequence:

- Long-context behavior depends heavily on positional handling.
- Models may perform worse on information buried in the middle of long context.
- RAG should avoid dumping excessive irrelevant context.

---

## 6.6 Self-attention

Attention lets each token look at other tokens and decide what matters.

Example:

```text
The student failed the graph problem because it required cycle detection.
```

The token `it` should attend strongly to `graph problem`.

### Query, key, value

Self-attention uses three learned projections:

| Term | Meaning |
|---|---|
| Query | What this token is looking for |
| Key | What each token offers |
| Value | Information to copy/mix if relevant |

Simplified:

```text
attention_score = query · key
attention_weights = softmax(scores)
output = weighted sum of values
```

---

## 6.7 Multi-head attention

Instead of one attention pattern, transformers use multiple attention heads.

Different heads can learn different relationships:

- Subject-object relation
- Coreference
- Code indentation
- Function-call arguments
- Mathematical dependencies
- Topic transitions
- Long-range references

### Production consequence

Attention is powerful, but not perfect.

A model can still miss important details in long prompts, especially if the context is noisy or overloaded.

---

## 6.8 Causal attention

Decoder-only LLMs use causal attention.

That means a token can only attend to previous tokens, not future tokens.

```text
Allowed:
token 5 attends to tokens 1-4

Not allowed:
token 5 attends to token 8
```

This enables autoregressive next-token generation.

---

## 6.9 Feed-forward network

Each transformer block also has a feed-forward network.

Self-attention mixes information across tokens. The feed-forward network transforms each token representation.

A simplified transformer block:

```text
Input
  ↓
Self-attention
  ↓
Residual connection + normalization
  ↓
Feed-forward network
  ↓
Residual connection + normalization
  ↓
Output
```

---

## 6.10 Pretraining

Pretraining teaches the model general language/statistical knowledge.

Common objective for decoder LLMs:

```text
Given previous tokens, predict the next token.
```

Example:

```text
Input: "A binary search tree has"
Target: "left"
```

Through massive pretraining, the model learns:

- Grammar
- Facts
- Code patterns
- Reasoning patterns
- Style
- Common concepts
- Statistical associations

But pretraining alone creates a text completion model, not necessarily a helpful assistant.

---

## 6.11 Instruction tuning

Instruction tuning trains the model on examples of instructions and ideal responses.

Example:

```text
Instruction:
Explain binary search in simple terms.

Ideal response:
Binary search is an algorithm that repeatedly halves a sorted search space...
```

Instruction tuning makes the model better at:

- Following user commands
- Answering questions
- Explaining
- Refusing inappropriate requests
- Formatting answers
- Acting like an assistant

---

## 6.12 RLHF

RLHF means **Reinforcement Learning from Human Feedback**.

Typical pipeline:

```text
1. Pretrain base model.
2. Supervised fine-tune on instruction examples.
3. Generate multiple answers.
4. Humans rank answers.
5. Train reward model.
6. Optimize model to produce preferred answers.
```

### What RLHF improves

- Helpfulness
- Harmlessness
- Conversational quality
- Instruction following
- Refusal behavior
- Preference alignment

### RLHF risks

| Risk | Explanation |
|---|---|
| Reward hacking | Model learns to satisfy reward model, not true quality |
| Over-refusal | Model refuses safe requests |
| Verbosity bias | Model learns longer answers are preferred |
| Preference bias | Human preferences may not generalize |
| Hard debugging | Behavior changes can be opaque |

---

## 6.13 DPO

DPO means **Direct Preference Optimization**.

It trains directly on preference pairs:

```text
Prompt
  Chosen answer
  Rejected answer
```

Unlike classic RLHF, DPO does not require a separate reward model and RL optimization loop.

### RLHF vs DPO

| Area | RLHF | DPO |
|---|---|---|
| Data | Preference rankings | Preference pairs |
| Reward model | Usually yes | No separate reward model |
| Complexity | Higher | Lower |
| Stability | Harder | Usually simpler |
| Use case | Large alignment pipelines | Preference fine-tuning |

### Why DPO is popular

- Simpler implementation
- More stable training
- Easier preference optimization
- Good for aligning open-source models

---

## 6.14 Sampling: temperature and top-p

LLMs produce probabilities over next tokens.

Sampling controls how tokens are selected.

### Temperature

Lower temperature:

- More deterministic
- More conservative
- Better for factual/structured tasks

Higher temperature:

- More diverse
- More creative
- More variable
- Higher hallucination risk

### Top-p

Top-p nucleus sampling restricts selection to the smallest group of tokens whose cumulative probability exceeds `p`.

Example:

```text
top_p = 0.9
Consider only tokens covering 90% probability mass.
```

### Production rule

| Task | Suggested behavior |
|---|---|
| JSON extraction | Low temperature |
| Classification | Low temperature |
| Creative writing | Higher temperature |
| Brainstorming | Moderate/high temperature |
| RAG factual answer | Low/moderate temperature |
| Tool calling | Low temperature |

---

## 6.15 Why models hallucinate

LLMs predict plausible next tokens. They do not inherently know whether a claim is true.

Hallucinations happen because:

- Model lacks current/private facts.
- Prompt asks beyond available context.
- Retrieved context is weak.
- Model overgeneralizes from training data.
- User asks ambiguous question.
- Decoding allows plausible but false text.
- There is no verification step.

### Mitigation

- RAG with citations
- Tool use for factual data
- Structured output validation
- Faithfulness evals
- Refusal when evidence is missing
- Explicit uncertainty
- Retrieval quality checks

---

## 6.16 Interview questions — Model internals

### Q1. What is a Transformer?

**Answer:**  
A Transformer is a neural network architecture based on attention. It processes tokens using self-attention, feed-forward layers, residual connections, and normalization. Modern LLMs use transformers because they can model relationships across long sequences and train efficiently in parallel.

---

### Q2. What is attention?

**Answer:**  
Attention lets each token decide which other tokens are relevant. It computes similarity between queries and keys, then uses the resulting weights to combine values. In simple terms, attention tells the model where to look in the context.

---

### Q3. What is the difference between pretraining and instruction tuning?

**Answer:**  
Pretraining teaches the model to predict the next token from massive raw text/code data. Instruction tuning trains the model on instruction-response examples so it behaves like an assistant. Pretraining gives general capability; instruction tuning improves usability.

---

### Q4. What is RLHF?

**Answer:**  
RLHF aligns a model using human preference feedback. Humans rank model outputs, a reward model learns those preferences, and the model is optimized to produce preferred outputs. It improves helpfulness and safety but can introduce reward hacking, over-refusal, or preference bias.

---

### Q5. What is DPO?

**Answer:**  
DPO directly trains a model using chosen vs rejected response pairs. It avoids the separate reward model and RL loop used in classic RLHF, making it simpler and often more stable.

---

### Q6. Why do LLMs hallucinate?

**Answer:**  
LLMs generate plausible next tokens, not verified facts. They hallucinate when they lack reliable context, retrieve poor evidence, overgeneralize from training data, or are asked for information beyond their knowledge. Production systems reduce hallucination with RAG, tools, citations, validation, and uncertainty handling.

---

# 7. Multimodal AI

## 7.1 What multimodal AI means

Multimodal AI can process more than text.

Modalities include:

- Text
- Images
- PDFs
- Screenshots
- Charts
- Tables
- Audio
- Video
- Documents
- Diagrams

A multimodal model can reason over mixed inputs.

Example:

```text
User uploads a system design diagram and asks:
"What are the bottlenecks here?"
```

The model can analyze the visual diagram and produce feedback.

---

## 7.2 Image understanding

Image understanding means the model reasons over pixels and visual structure.

Use cases:

- UI screenshot debugging
- Architecture diagram review
- Whiteboard explanation
- Chart interpretation
- Form understanding
- Resume screenshot parsing
- Code screenshot explanation

### Example: system design diagram

The model can identify:

- Services
- API gateway
- Load balancer
- Queues
- Databases
- Cache
- Bottlenecks
- Missing retry path
- Missing DLQ
- Single point of failure

### Production pipeline

```text
User uploads image
  ↓
Validate file type and size
  ↓
Run safety/security checks
  ↓
Send to vision model
  ↓
Extract structured output
  ↓
Validate output
  ↓
Store derived metadata
```

---

## 7.3 Document understanding

Document understanding is more than OCR.

It includes:

- Text extraction
- Layout understanding
- Table parsing
- Chart interpretation
- Section hierarchy
- Page references
- Image understanding
- Citation/page grounding

### Examples

- Resume parsing
- Job description analysis
- Interview transcript extraction
- Company prep PDF summarization
- Whiteboard diagram review
- Legal/policy document QA

### Production document pipeline

```text
PDF upload
  ↓
Extract native text if available
  ↓
Render pages as images if layout matters
  ↓
Chunk by section/page/table
  ↓
Generate text embeddings
  ↓
Generate image/page embeddings if needed
  ↓
Store page metadata
  ↓
Retrieve relevant pages/chunks
  ↓
Answer with page citations
```

---

## 7.4 OCR limitations

OCR means Optical Character Recognition.

It extracts text from images.

OCR is useful, but limited.

### Common OCR problems

| Problem | Example |
|---|---|
| Layout loss | Two-column PDF text gets mixed |
| Table breakage | Rows/columns merge incorrectly |
| Chart weakness | OCR reads labels but misses trend |
| Handwriting errors | Letters/numbers misread |
| Low-quality scans | Blurry text extraction fails |
| Code indentation | Python code loses structure |
| Math notation | Symbols are misread |
| Diagrams | OCR sees labels but not relationships |

### OCR vs vision model

| OCR | Vision model |
|---|---|
| Extracts visible text | Understands visual structure |
| Cheaper for simple text | Better for charts/diagrams |
| Weak on layout reasoning | Better spatial reasoning |
| More deterministic | More probabilistic |
| Good for clean scans | Better for mixed visual content |

### Best production approach

```text
Use native text extraction/OCR for text-heavy documents.
Use vision models when layout, tables, charts, screenshots, or diagrams matter.
Use both for high-value documents.
```

---

## 7.5 Audio models

Audio AI can support:

- Speech-to-text
- Speaker diarization
- Voice interaction
- Call summarization
- Mock interview analysis
- Pronunciation feedback
- Fluency analysis
- Filler word detection

### Interview prep example

```text
Mock interview audio
  ↓
Transcription
  ↓
Speaker segmentation
  ↓
Answer quality analysis
  ↓
Filler word detection
  ↓
Confidence/clarity feedback
  ↓
Suggested improved answer
```

### Production concerns

| Concern | Why it matters |
|---|---|
| Consent | Users must know audio is recorded |
| Storage | Audio can contain sensitive data |
| Latency | Real-time voice needs low latency |
| Accent robustness | Transcription quality varies |
| Background noise | Can degrade accuracy |
| Speaker separation | Needed for interviews/calls |
| PII redaction | Transcripts may expose sensitive info |
| Deletion controls | Users need control over recordings |

---

## 7.6 Video models

Video understanding can include:

- Keyframe extraction
- Scene understanding
- Audio transcription
- OCR from frames
- Temporal reasoning
- Gesture/body-language cues

### Practical production pipeline

Do not send every frame to a model unless necessary.

Better:

```text
Video
  ↓
Extract audio transcript
  ↓
Sample keyframes
  ↓
Run OCR/vision on important frames
  ↓
Combine transcript + visual metadata
  ↓
Search/reason over derived representation
```

### Use cases

- Lecture summarization
- Mock interview analysis
- UI walkthrough analysis
- Meeting summary
- Training video search
- Whiteboard lecture extraction

### Risks

- High cost
- High latency
- Large file sizes
- Difficult evaluation
- Privacy issues
- Temporal reasoning errors
- Storage complexity

---

## 7.7 Multimodal RAG

Multimodal RAG extends RAG beyond text.

It retrieves relevant evidence across:

- Text chunks
- Images
- Tables
- Charts
- PDF pages
- Audio segments
- Video frames

### Text-only RAG

```text
Question
  ↓
Text embedding
  ↓
Retrieve text chunks
  ↓
Generate answer
```

### Multimodal RAG

```text
Question / image / audio / video
  ↓
Text + visual + audio embeddings
  ↓
Retrieve relevant chunks/pages/images/frames
  ↓
Fuse evidence
  ↓
Generate answer with citations
```

### Example

User asks:

```text
What is wrong with this system design diagram?
```

System retrieves:

- The uploaded diagram
- Similar architecture diagrams
- Internal system design checklist
- Queue/retry best practices
- User’s previous design mistakes

Then the model produces grounded feedback.

---

## 7.8 Multimodal RAG architecture

```text
Ingestion:
  PDF/image/audio/video
    ↓
  Extract text
    ↓
  Extract visual/audio metadata
    ↓
  Chunk by page/scene/section
    ↓
  Generate embeddings
    ↓
  Store vectors + metadata

Query:
  User query/input
    ↓
  Embed query/input
    ↓
  Retrieve relevant multimodal evidence
    ↓
  Rerank
    ↓
  Generate answer
    ↓
  Cite page/frame/source
```

### Metadata examples

```json
{
  "source_type": "pdf",
  "page": 12,
  "section": "System Design",
  "contains_table": true,
  "contains_diagram": true,
  "created_at": "2026-06-28"
}
```

---

## 7.9 Production risks in multimodal AI

| Risk | Mitigation |
|---|---|
| High cost | Sampling, compression, caching |
| High latency | Async processing, pre-indexing |
| OCR errors | Native text + OCR + vision |
| Visual hallucination | Ask for page/region evidence |
| Large files | Chunking and file limits |
| Privacy | Consent, encryption, retention policy |
| Evaluation difficulty | Human-labeled test sets |
| Poor citations | Page/frame-level metadata |
| Security | MIME validation, malware scanning |
| Data leakage | Tenant-aware storage and retrieval |

---

## 7.10 Interview questions — Multimodal AI

### Q1. How is document understanding different from OCR?

**Answer:**  
OCR extracts text from images or scanned documents. Document understanding goes further by understanding layout, tables, charts, images, section hierarchy, and relationships between elements. OCR alone is often insufficient for complex PDFs.

---

### Q2. When would you use a vision model instead of OCR?

**Answer:**  
Use a vision model when the answer depends on layout, visual structure, charts, diagrams, screenshots, handwriting, forms, or spatial relationships. Use OCR/native extraction when the document is mostly clean text and cost matters.

---

### Q3. What is multimodal RAG?

**Answer:**  
Multimodal RAG retrieves evidence across multiple modalities such as text, images, tables, audio, video, or document pages before generating an answer. It is useful when knowledge is not purely textual, such as PDFs with charts, diagrams, screenshots, or video lectures.

---

### Q4. What are the main production risks of multimodal AI?

**Answer:**  
The main risks are cost, latency, file size limits, privacy, OCR errors, hallucinated visual details, weak chart/table extraction, and difficult evaluation. Production systems need validation, citations, confidence handling, and careful file processing.

---

### Q5. How would you process a PDF for RAG?

**Answer:**  
I would extract native text where available, render pages as images if layout matters, split the document by page/section/table, generate embeddings, store page-level metadata, retrieve relevant chunks/pages at query time, and answer with citations. For visually complex PDFs, I would combine OCR/text extraction with vision-model analysis.

---

# 8. AI Product Sense

## 8.1 Why AI product sense matters

Many candidates talk only about models.

Strong candidates talk about product behavior.

AI products are probabilistic, so UX must handle:

- Uncertainty
- Hallucination
- Latency
- Partial correctness
- User correction
- Source visibility
- Permission boundaries
- Action safety

A strong interview answer:

> AI products should be designed around uncertainty, trust, correction, and failure recovery. The model will sometimes be wrong, so the UX must make uncertainty visible, provide sources, allow correction, and prevent unsafe actions.

---

## 8.2 Traditional software vs AI software

Traditional software is deterministic:

```text
Click button → predictable result
```

AI software is probabilistic:

```text
Ask question → likely useful but not guaranteed result
```

Therefore AI UX needs:

- Confidence indicators
- Source grounding
- Correction flows
- Undo
- Human approval
- Progress states
- Clear limitations
- Guardrails

---

## 8.3 UX patterns for AI

### Pattern 1: Show sources where factuality matters

For RAG answers:

```text
Answer
  + citations
  + retrieved sources
  + timestamp/freshness
```

Example:

```text
Based on 2025-2026 interview reports, graph traversal and DP appear frequently for Google backend roles. Evidence is limited to 8 retrieved reports.
```

This is better than:

```text
Google definitely asks graph traversal and DP.
```

---

### Pattern 2: Let users correct the model

Example:

```text
User: This was not DP, it was greedy.
System: Got it. I’ll update the topic label.
```

Correction should become data:

```json
{
  "event_type": "label_correction",
  "user_id": "user_123",
  "old_label": "dynamic_programming",
  "new_label": "greedy",
  "source": "user_feedback"
}
```

This improves future recommendations.

---

### Pattern 3: Use progressive disclosure

Do not dump everything at once.

For tutor UX:

```text
Short answer
  ↓
Example
  ↓
Deeper explanation
  ↓
Practice problem
  ↓
Quiz
```

This is better than a huge explanation every time.

---

### Pattern 4: Make latency visible

For longer tasks, show progress.

```text
Searching interview corpus...
Checking your solved problems...
Generating study plan...
```

Users tolerate latency better when the system explains what is happening.

---

### Pattern 5: Separate answer from action

Bad:

```text
Model automatically deletes recommendations.
```

Good:

```text
Model suggests:
"Dismiss these 3 recommendations?"

User confirms.
```

Agents should not silently perform sensitive actions.

---

## 8.4 Uncertainty handling

AI systems should not pretend certainty.

Bad:

```text
This is definitely the latest Google interview pattern.
```

Better:

```text
Based on the retrieved 2025-2026 interview reports, graph traversal and DP appear frequently. The data may be incomplete because interview reports are self-reported.
```

### Production uncertainty signals

| Situation | UX response |
|---|---|
| Low retrieval score | Say evidence is weak |
| Conflicting sources | Present both views |
| No source found | Clearly say no evidence was found |
| Old data | Mention date/freshness |
| Ambiguous user query | Ask targeted follow-up or state assumption |
| High-impact action | Require confirmation |
| Model confidence low | Provide caveat and safer alternative |

---

## 8.5 Guardrails

Guardrails are controls around model behavior.

### Types of guardrails

| Guardrail | Example |
|---|---|
| Input validation | Reject invalid file type |
| Output validation | JSON schema validation |
| Retrieval guardrail | Permission-aware filtering |
| Tool guardrail | Tool allowlist and approval |
| Safety guardrail | Block harmful requests |
| Business guardrail | Do not recommend solved problems |
| Cost guardrail | Token/tool budget |
| Rate guardrail | Per-user request limits |
| Evaluation guardrail | Regression tests before deploy |

### Guardrails for AI tutor/recommendation system

```text
Do not recommend already-solved problems.
Do not expose another user's progress.
Do not fabricate company interview data.
Do not call expensive deep research for trivial queries.
Do not mutate user study plan without confirmation.
Do not store sensitive user data without consent.
```

---

## 8.6 Explaining limitations

Generic disclaimers are weak.

Bad:

```text
AI can make mistakes.
```

Better:

```text
I found only 4 recent reports for this company-role combination, so the pattern may be incomplete.
```

Or:

```text
This recommendation is based on your LeetCode history, but it does not include Codeforces/GFG yet.
```

Specific limitations build trust.

---

## 8.7 User trust

Trust comes from repeated correctness and recoverability.

### Trust-building mechanisms

| Mechanism | Why it helps |
|---|---|
| Citations | User can verify |
| Confidence labels | Avoids false certainty |
| Correction buttons | User can fix wrong output |
| Undo | Reduces risk |
| Human approval | Prevents unsafe actions |
| Memory controls | Prevents creepy personalization |
| Stable behavior | Reduces surprise |
| Fast feedback | Improves perceived reliability |
| Clear limitations | Sets expectations |
| Data visibility | User knows what was used |

---

## 8.8 AI product anti-patterns

Avoid these:

| Anti-pattern | Why it fails |
|---|---|
| Magic agent UX | User does not know what happened |
| No citations | Hard to trust |
| Over-personalization | Feels creepy |
| Hidden actions | Dangerous |
| Long answers by default | Cognitive overload |
| No correction loop | Errors repeat |
| No evals | Quality silently regresses |
| No fallback | Failures become dead ends |
| Too many agents | High latency and nondeterminism |
| Generic disclaimers | Users ignore them |

---

## 8.9 AI product metrics

Useful metrics include:

| Area | Metrics |
|---|---|
| Quality | Answer helpfulness, accuracy, faithfulness |
| Retrieval | Hit rate, MRR, recall@k, citation accuracy |
| UX | Time to first token, completion rate |
| Trust | Correction rate, source click rate |
| Safety | Block rate, unsafe output rate |
| Cost | Cost/request, token usage |
| Reliability | Error rate, retry rate, timeout rate |
| Engagement | Return usage, task completion |
| Personalization | Recommendation acceptance/dismissal |

---

## 8.10 Interview questions — AI product sense

### Q1. How do you design UX for an AI product differently from normal software?

**Answer:**  
Normal software is deterministic, while AI is probabilistic. AI UX must handle uncertainty, partial correctness, hallucination, latency, correction, sources, and user control. The product should show what the AI used, how confident it is, and what the user can do if it is wrong.

---

### Q2. How should an AI product handle uncertainty?

**Answer:**  
It should expose uncertainty in context-specific ways: show citations, mention weak evidence, ask clarifying questions when needed, present assumptions, and avoid overconfident claims. For high-impact actions, require confirmation.

---

### Q3. What are guardrails in AI products?

**Answer:**  
Guardrails are controls around inputs, outputs, retrieval, tools, permissions, cost, and safety. Examples include schema validation, permission-aware retrieval, tool allowlists, approval gates, rate limits, and regression tests.

---

### Q4. How do you build user trust in an AI system?

**Answer:**  
Trust comes from accurate answers, citations, predictable behavior, transparency, correction mechanisms, and safe action boundaries. Users should be able to verify, correct, undo, and understand what data the system used.

---

### Q5. What is a bad AI product pattern?

**Answer:**  
A bad pattern is a “magic agent” that silently performs actions without showing sources, assumptions, or approval steps. It may look impressive in a demo but fails in production because users cannot trust, inspect, or correct it.

---

# 9. Final Production Architecture Lens

## 9.1 Strong AI production answer

When asked about any AI topic, frame your answer around production constraints.

```text
Model capability
  + data/retrieval
  + tool access
  + validation
  + permissions
  + retries/fallbacks
  + observability
  + evals
  + UX trust
```

---

## 9.2 Example AI tutor architecture

```text
User query
  ↓
Deterministic pre-router
  ↓
LLM router with structured output
  ↓
Policy/depth decision
  ↓
Optional retrieval:
    - interview corpus
    - DSA content
    - user progress
    - recommendation state
  ↓
Tool calls if needed:
    - get_user_progress
    - search_interview_corpus
    - create_learning_event
  ↓
Answer generation
  ↓
Output validation
  ↓
Citation/uncertainty layer
  ↓
Telemetry + feedback logging
```

---

## 9.3 Where each topic fits

| Topic | Role in production system |
|---|---|
| MCP | Standardizes access to tools/resources/prompts |
| Function calling | Lets model request external actions |
| Structured output | Makes LLM output usable by backend |
| Streaming | Improves user-perceived latency |
| Embeddings | Enables semantic retrieval |
| Vector DBs | Store/search embeddings |
| RAG | Injects fresh/private knowledge |
| Fine-tuning | Improves behavior consistency |
| LoRA/adapters | Efficient model specialization |
| Synthetic data | Expands training/eval data carefully |
| LlamaIndex | RAG/data framework |
| PydanticAI | Typed output and validation |
| CrewAI/AutoGen | Multi-agent workflows |
| DSPy | Metric-driven prompt/program optimization |
| Transformers | Explain model capabilities/limits |
| Multimodal AI | Handles images/docs/audio/video |
| Guardrails | Prevent unsafe/bad behavior |
| UX trust | Makes AI usable in real product |

---

## 9.4 Best answer pattern for interviews

Use this structure:

```text
1. Define the concept.
2. Explain where it fits in production.
3. Explain tradeoffs.
4. Mention failure modes.
5. Describe mitigations.
6. Give a concrete example.
```

Example:

> RAG retrieves relevant external context at inference time. In production, I use it when facts are dynamic, private, or too large to fit in the prompt. The tradeoff is retrieval latency and retrieval quality. It can fail due to poor chunking, stale data, weak metadata, or no reranking. I mitigate that with hybrid search, metadata filters, reranking, citations, and evals. For an interview-prep system, I would use RAG to retrieve company-specific interview reports and user progress before generating a study plan.

---

# 10. High-Impact Interview Revision Checklist

## 10.1 MCP

Know:

- Tools vs resources vs prompts
- Host-client-server model
- Access boundaries
- Agent-tool interoperability
- Why MCP is not security by itself
- Why narrow tools are safer than generic tools

Be ready to answer:

- What is MCP?
- How would you design an MCP server?
- What are the risks of exposing tools?
- How do you prevent cross-user leakage?

---

## 10.2 LLM APIs

Know:

- Function calling/tool use
- Streaming
- Structured outputs
- Embeddings
- Multimodal inputs
- Rate limits/retries
- Observability
- Fallbacks

Be ready to answer:

- How do tools work?
- How do you validate tool calls?
- Why use streaming?
- When use structured outputs?
- How do embeddings support RAG?

---

## 10.3 Vector DBs

Know:

- Vector similarity
- Metadata filtering
- ANN indexes
- HNSW vs IVF vs flat
- pgvector vs Pinecone vs Weaviate vs Qdrant vs Chroma
- Hybrid retrieval
- Reranking

Be ready to answer:

- Why use vector DB?
- Why metadata filtering matters?
- When is pgvector enough?
- When move to dedicated vector DB?
- Why does RAG fail?

---

## 10.4 Fine-tuning vs RAG

Know:

- RAG = knowledge
- Fine-tuning = behavior
- LoRA/adapters
- Synthetic data risks
- Eval before training
- RAG + fine-tune combined systems

Be ready to answer:

- When use RAG?
- When fine-tune?
- What is LoRA?
- Why eval before fine-tuning?
- Can synthetic data hurt?

---

## 10.5 Frameworks

Know:

- LlamaIndex = RAG/data
- Semantic Kernel = enterprise plugin orchestration
- PydanticAI = typed agents/validation
- CrewAI = role-based agents
- AutoGen = multi-agent conversations
- DSPy = metric-driven optimization

Be ready to answer:

- Which framework would you choose and why?
- Why avoid multi-agent systems for simple workflows?
- When is DSPy useful?
- How does PydanticAI help production safety?

---

## 10.6 Model internals

Know:

- Tokenization
- Transformer blocks
- Self-attention
- Causal attention
- Pretraining
- Instruction tuning
- RLHF
- DPO
- Hallucination causes

Be ready to answer:

- What is attention?
- What is pretraining vs instruction tuning?
- Why do LLMs hallucinate?
- What is RLHF?
- What is DPO?

---

## 10.7 Multimodal AI

Know:

- Image understanding
- Document understanding
- OCR limitations
- Audio/video pipelines
- Multimodal RAG
- Page/frame-level citations
- Privacy/cost risks

Be ready to answer:

- OCR vs document understanding?
- When use vision model?
- How process a PDF?
- What is multimodal RAG?
- What are production risks?

---

## 10.8 AI product sense

Know:

- AI UX is probabilistic
- Uncertainty handling
- Citations
- Guardrails
- Correction loops
- Human approval
- User trust
- Product anti-patterns

Be ready to answer:

- How do you design AI UX?
- How do you handle uncertainty?
- What guardrails are needed?
- How do you build trust?
- What makes an AI product fail?

---

# Final condensed summary

Use this as your final mental model:

```text
RAG is for knowledge.
Fine-tuning is for behavior.
MCP standardizes tool/context access.
Function calling gives models controlled actions.
Structured outputs make LLMs backend-safe.
Vector DBs support semantic retrieval, but retrieval quality depends on chunking, metadata, reranking, and evals.
Frameworks are orchestration choices, not magic.
Transformers predict tokens using attention.
Multimodal AI adds image/document/audio/video grounding but increases cost and evaluation complexity.
AI product quality depends on uncertainty handling, guardrails, source visibility, and correction loops.
```

For production systems:

```text
Deterministic orchestrator
  + RAG for dynamic knowledge
  + structured outputs for backend safety
  + tool calls with permissions
  + evals before fine-tuning
  + guardrails around actions
  + citations and uncertainty for trust
```

That is the strongest interview framing: not just knowing the terms, but knowing when to use them, where they fail, and how to ship them safely.
