---
title: Design Principles
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: Normative Guidelines
---

# Design Principles

**Version:** 1.0.0-draft

These principles are **normative** - all implementations MUST adhere to them. They define the philosophical approach that makes LEAF spec an effective architectural benchmark.

---

## 1. AI is Essential, Not Decorative

### Principle
The application must not function meaningfully without AI. This specification is not about adding AI features to a traditional CRUD app - AI workflows are fundamental to the application's core value.

### Requirements
- ✅ Documents without AI processing provide no value
- ✅ Primary user interactions involve AI operations
- ✅ Removing AI would break core functionality
- ❌ AI cannot be "optional" or "premium" feature

### Why This Matters
Many "AI applications" are traditional apps with AI bolt-ons. This specification forces honest integration of AI into the application architecture from the ground up, revealing how frameworks handle AI-first design.

---

## 2. Explicit Workflows Over Implicit Magic

### Principle
All AI-related processing must be observable and modeled explicitly in code. No hidden abstractions, no "magic" SDKs that obscure operations, no implicit background jobs that developers can't trace.

### Requirements
Implementations must explicitly model and expose:
- ✅ Document ingestion pipeline stages (upload → chunk → embed → index)
- ✅ Job/task status for all async operations
- ✅ Retrieval steps that occur before generation
- ✅ Token usage and costs per operation
- ✅ Failure modes and retry logic
- ✅ Prompt construction and versioning

### What This Forbids
- ❌ SDKs that hide chunking/embedding behind "auto-ingest"
- ❌ Frameworks that abstract away vector search
- ❌ Libraries that hide prompt construction
- ❌ "Magic" configuration that makes decisions opaque

### Why This Matters
The most important architectural decisions in AI applications happen in the "magic" layer that many frameworks hide. This principle forces implementations to show their work, revealing complexity and design choices.

### Example
**Bad (implicit):**
```python
# What's happening here?
knowledge_base.add_document(pdf_file)
```

**Good (explicit):**
```python
# Upload document
document = storage.upload(pdf_file)
document.status = "uploaded"

# Queue ingestion job
job = ingestion_queue.enqueue(
    task="process_document",
    document_id=document.id
)

# Process async
chunks = chunker.chunk(document.text)
embeddings = embedding_model.embed(chunks)
vector_store.index(embeddings, document_id=document.id)
document.status = "ready"
```

---

## 3. Asynchronous by Default

### Principle
AI operations are long-running (seconds to minutes), expensive (tokens cost money), and failure-prone (rate limits, timeouts, errors). They **must** be modeled as asynchronous jobs, never as synchronous request-response operations.

### Requirements
- ✅ Document processing happens asynchronously
- ✅ All AI generation is non-blocking
- ✅ Job status queryable via API
- ✅ Failures handled gracefully with retry logic
- ✅ Timeout handling for long operations
- ❌ **Synchronous AI calls from HTTP request handlers are FORBIDDEN**

### What This Forbids
```python
# FORBIDDEN: Synchronous AI call in request handler
@app.post("/chat")
def chat(message: str):
    response = llm.generate(message)  # ❌ BLOCKS REQUEST
    return {"response": response}
```

### What This Requires
```python
# REQUIRED: Async job pattern
@app.post("/chat")
def chat(message: str):
    job = job_queue.enqueue(
        task="generate_response",
        message=message,
        user_id=current_user.id
    )
    return {"job_id": job.id, "status": "pending"}

@app.get("/jobs/{job_id}")
def get_job(job_id: str):
    job = job_store.get(job_id)
    return {
        "status": job.status,  # pending | processing | complete | failed
        "result": job.result if job.complete else None
    }
```

### Why This Matters
Frameworks differ dramatically in how they handle async operations, background jobs, and long-running tasks. This principle forces honest modeling of AI's inherent asynchronicity, exposing architectural choices around:
- Job queues (Celery, Bull, custom)
- Status storage and polling
- WebSocket/SSE for real-time updates
- Error handling and retry logic
- Resource limits and timeouts

---

## 4. Provenance and Citations Required

### Principle
Every AI output must be traceable to its sources. No hallucinations, no fabricated citations, no claims without evidence.

### Requirements
- ✅ Retrieved chunks stored with each AI result
- ✅ Citations reference specific documents and locations
- ✅ Relevance scores recorded
- ✅ Users can click citations to view source
- ❌ AI cannot cite documents not actually retrieved
- ❌ AI cannot invent page numbers or quotes

### What This Requires
Every AI response must include:
```json
{
  "content": "Neural networks consist of layers...",
  "citations": [
    {
      "documentId": "doc_123",
      "chunkId": "chunk_456",
      "excerpt": "A neural network consists of layers...",
      "relevanceScore": 0.94,
      "page": 12
    }
  ],
  "retrievalMetadata": {
    "searchQuery": "neural network architecture",
    "documentsSearched": 5,
    "chunksRetrieved": 10,
    "topKUsed": 3
  }
}
```

### Why This Matters
Citation quality is the key difference between toy demos and production AI applications. This principle forces implementations to:
- Model chunk-level granularity
- Implement retrieval tracking
- Design UI for provenance display
- Handle edge cases (no relevant content found)

---

## 5. Non-Determinism is Explicit

### Principle
AI responses vary between runs with identical inputs. The specification embraces this reality rather than hiding it.

### Requirements
- ✅ Each AI execution creates a new, versioned result
- ✅ Re-running produces new result (no automatic reuse)
- ✅ Prompt templates versioned
- ✅ Model identifiers recorded
- ✅ Temperature/parameters stored
- ✅ Users can compare multiple runs

### Data Model
```json
{
  "id": "result_789",
  "actionName": "answer_question",
  "actionVersion": "v2",
  "inputParameters": {
    "question": "What is a neural network?"
  },
  "modelIdentifier": "gpt-4-turbo-2024-04-09",
  "modelParameters": {
    "temperature": 0.7,
    "maxTokens": 500
  },
  "retrievedChunkIds": ["chunk_1", "chunk_2"],
  "output": "...",
  "createdAt": "2024-01-15T10:00:00Z"
}
```

### Why This Matters
Traditional apps are deterministic - same input always produces same output. AI apps are non-deterministic. This principle forces implementations to design for:
- Result versioning and history
- A/B comparison of runs
- Reproducibility tracking
- Debugging non-deterministic behavior

---

## 6. No Framework-Specific Shortcuts

### Principle
Implementations cannot hide architectural complexity behind proprietary abstractions or framework-exclusive features. The specification must work with **any** tech stack.

### Requirements
- ✅ Works with any LLM provider (OpenAI, Anthropic, Gemini, local models)
- ✅ Works with any vector database (Pinecone, Weaviate, pgvector, Chroma)
- ✅ Works with any web framework (Express, FastAPI, Rails, Spring)
- ✅ Works with any language (Python, JavaScript, Go, Rust, Java)
- ❌ Cannot require vendor-specific SDKs (e.g., Vercel AI SDK only)
- ❌ Cannot require proprietary services (e.g., must work on Vercel)
- ❌ Cannot rely on framework-exclusive abstractions

### What This Forbids
```typescript
// FORBIDDEN: Vercel-only implementation
import { OpenAIStream } from 'ai'

export default async function POST(req: Request) {
  const stream = await OpenAIStream(...)  // ❌ Framework-specific
  return new StreamingTextResponse(stream)
}
```

### What This Allows
```typescript
// ALLOWED: Framework-agnostic SSE streaming
export default async function POST(req: Request) {
  const stream = new ReadableStream({
    async start(controller) {
      for await (const chunk of llm.generate(...)) {
        controller.enqueue(`data: ${JSON.stringify(chunk)}\n\n`)
      }
      controller.close()
    }
  })
  return new Response(stream, {
    headers: { 'Content-Type': 'text/event-stream' }
  })
}
```

### Why This Matters
The goal is to compare frameworks fairly. Framework-specific shortcuts hide complexity and make comparisons meaningless. Every implementation must solve the same problems with the same level of explicitness.

---

## 7. Strict Scope Discipline

### Principle
Features not explicitly required in the specification are out of scope. This is a benchmark that reveals architectural decisions, not a feature showcase or production-ready app.

### Requirements
- ✅ Implement only what the spec requires
- ✅ Mark extensions clearly as "beyond spec"
- ✅ Scope creep invalidates comparisons
- ❌ Adding unrequired features makes benchmarking meaningless

### Why This Matters
The original RealWorld works because all implementations solve the **exact same problem**. Adding features breaks comparability. An implementation with real-time collaboration, multimodal input, and agent workflows cannot be fairly compared to one that strictly follows the spec.

---

## Summary

These principles ensure LEAF spec serves its purpose: **exposing architectural truth**.

The specification is deliberately **demanding**. It forces implementations to:
- Model complexity explicitly
- Handle asynchronicity honestly
- Track provenance rigorously
- Embrace non-determinism
- Work universally

If implementing this spec feels verbose or uncomfortable, that's working as intended. The goal is to reveal what AI application development actually requires, without the marketing gloss of "seamless integration" or "just add one line of code."

Just as RealWorld revealed that some frameworks required 10x more boilerplate than others, LEAF spec will reveal which approaches handle AI's unique challenges (async, non-determinism, cost, failures) with clarity versus obscurity.

---

**Next:** See [required-features.md](./required-features.md) for specific feature requirements that embody these principles.
