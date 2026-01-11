---
title: Provider Interfaces
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: Integration Guidance
---

# Provider Interfaces (Recommended Architecture)

**Version:** 1.0.0-draft

---

## 🚨 Important: These Are Guidelines, Not Requirements

**Unlike the REST API specification, these interfaces are RECOMMENDED guidance, not mandatory architecture.**

### What This Document Provides

This document shows **WHAT** needs to be abstracted (embeddings, vector storage, LLM calls) but **NOT HOW** you must structure your code.

**Implementations are free to:**
- Use different interface patterns
- Build monolithic services without interfaces
- Structure code in language-idiomatic ways
- Create alternative abstraction layers

### Why We Include This

**Historical Context:** The original RealWorld/Conduit specification intentionally did NOT prescribe internal architecture - only external API contracts. We're following that tradition.

**Why This Doc Exists:** AI abstraction points are not obvious to newcomers. Questions like "Where do I swap OpenAI for Anthropic?" or "How do I make my vector database swappable?" are common. This document provides **pedagogical examples** showing common patterns.

**Think of this as:** The architectural equivalent of a "suggested solution" in a textbook - helpful for learning, but you can solve the problem differently.

---

## Why Provider Abstraction Matters

The LEAF specification requires implementations to work with **any** AI provider (OpenAI, Anthropic, local models) and **any** vector database (Pinecone, pgvector, Weaviate). This is only possible if you abstract provider-specific APIs.

**Benefits of abstraction:**
- **Enable comparison** - Swap OpenAI for Anthropic to see performance differences
- **Reveal complexity** - Show what the "magic" SDKs hide
- **Support testing** - Mock providers for unit tests
- **Prevent vendor lock-in** - Change providers without rewriting application logic

**You can achieve this with:**
- Interfaces/protocols (TypeScript, Go, Rust)
- Abstract base classes (Python, Java)
- Dependency injection (any language)
- Higher-order functions (functional languages)
- Configuration-driven factories (any language)
- Or no abstraction layer if you prefer explicit code

---

## Recommended Interfaces

The following interfaces show common abstraction points. Use them as inspiration, not blueprints.

---

## Interface 1: Embedder

Generates vector embeddings from text.

### Contract

```typescript
interface Embedder {
  // Generate embedding for a single text
  embed(text: string): Promise<float[]>

  // Generate embeddings for multiple texts in batch
  embedBatch(texts: string[]): Promise<float[][]>

  // Return the dimension of generated embeddings
  dimension(): number

  // Optional: Return model identifier
  model(): string
}
```

### Requirements

**Consistency:**
- All embeddings from the same Embedder instance MUST have identical dimensions
- Embedding dimension MUST remain constant throughout application lifecycle
- Re-embedding the same text SHOULD produce similar vectors (< 1% variance)

**Error Handling:**
- MUST handle empty strings (return zero vector or throw clear error)
- MUST handle very long texts (truncate or chunk appropriately)
- MUST handle rate limits gracefully
- MUST provide meaningful error messages

**Performance:**
- `embedBatch` SHOULD be used when embedding multiple texts
- Batch operations SHOULD be more efficient than multiple `embed` calls
- Implementations MAY cache embeddings (but MUST handle cache invalidation)

### Example Implementations

**OpenAI:**
```typescript
class OpenAIEmbedder implements Embedder {
  constructor(apiKey: string, model: string = "text-embedding-3-small") {}

  async embed(text: string): Promise<float[]> {
    const response = await openai.embeddings.create({
      model: this.model,
      input: text
    })
    return response.data[0].embedding
  }

  dimension(): number {
    return 1536 // or 3072 for large model
  }

  model(): string {
    return "text-embedding-3-small"
  }
}
```

**Anthropic (Voyage AI):**
```typescript
class VoyageEmbedder implements Embedder {
  constructor(apiKey: string, model: string = "voyage-2") {}

  async embed(text: string): Promise<float[]> {
    const response = await voyage.embed({
      model: this.model,
      input: [text]
    })
    return response.data[0].embedding
  }

  dimension(): number {
    return 1024
  }
}
```

**Local (Ollama):**
```typescript
class OllamaEmbedder implements Embedder {
  constructor(baseUrl: string = "http://localhost:11434",
              model: string = "nomic-embed-text") {}

  async embed(text: string): Promise<float[]> {
    const response = await fetch(`${this.baseUrl}/api/embeddings`, {
      method: 'POST',
      body: JSON.stringify({ model: this.model, prompt: text })
    })
    return (await response.json()).embedding
  }

  dimension(): number {
    return 768
  }
}
```

---

## Interface 2: VectorStore

Stores and retrieves embeddings for similarity search.

### Contract

```typescript
interface VectorStore {
  // Store an embedding with associated document/chunk ID
  store(id: string, embedding: float[], metadata: object): Promise<void>

  // Search for similar embeddings
  search(
    embedding: float[],
    limit: number,
    threshold?: number
  ): Promise<SearchHit[]>

  // Delete an embedding by ID
  delete(id: string): Promise<void>

  // Delete all embeddings for a document
  deleteByDocument(documentId: string): Promise<void>

  // Check if an embedding exists
  exists(id: string): Promise<boolean>

  // Get total count of stored embeddings
  count(): Promise<number>
}

interface SearchHit {
  id: string              // Document or chunk ID
  score: number           // 0.0 to 1.0, higher is more similar
  metadata: object        // Associated metadata (document ID, chunk index, etc.)
}
```

### Requirements

**Similarity Metric:**
- `score` MUST be normalized to 0.0-1.0 range
- 1.0 indicates identical vectors
- 0.0 indicates orthogonal or completely dissimilar
- Implementations SHOULD use cosine similarity
- If using other metrics (dot product, L2 distance), MUST normalize to 0.0-1.0

**Ordering:**
- `search` results MUST be sorted by score (descending)
- Highest similarity first

**Thresholding:**
- `threshold` parameter filters results with score < threshold
- If no results meet threshold, return empty array
- Default threshold is 0.0 (return all results up to limit)

**Metadata:**
- Implementations MUST preserve metadata exactly as stored
- Common metadata fields: documentId, chunkId, chunkIndex, page, section
- Metadata can be used for post-filtering (not part of similarity search)

**Performance:**
- Search SHOULD complete in < 100ms for < 100k vectors
- Search SHOULD complete in < 500ms for < 1M vectors
- Batch operations (store multiple embeddings) SHOULD be supported

### Example Implementations

**PostgreSQL + pgvector:**
```typescript
class PgVectorStore implements VectorStore {
  constructor(pool: pg.Pool, tableName: string = "embeddings") {}

  async store(id: string, embedding: float[], metadata: object): Promise<void> {
    await this.pool.query(`
      INSERT INTO ${this.tableName} (id, embedding, metadata)
      VALUES ($1, $2, $3)
      ON CONFLICT (id) DO UPDATE SET embedding = $2, metadata = $3
    `, [id, embedding, JSON.stringify(metadata)])
  }

  async search(embedding: float[], limit: number, threshold: number = 0.0): Promise<SearchHit[]> {
    const result = await this.pool.query(`
      SELECT id, 1 - (embedding <=> $1) AS score, metadata
      FROM ${this.tableName}
      WHERE 1 - (embedding <=> $1) >= $2
      ORDER BY embedding <=> $1
      LIMIT $3
    `, [embedding, threshold, limit])

    return result.rows.map(row => ({
      id: row.id,
      score: row.score,
      metadata: JSON.parse(row.metadata)
    }))
  }
}
```

**Pinecone:**
```typescript
class PineconeVectorStore implements VectorStore {
  constructor(index: PineconeIndex) {}

  async store(id: string, embedding: float[], metadata: object): Promise<void> {
    await this.index.upsert([{
      id,
      values: embedding,
      metadata
    }])
  }

  async search(embedding: float[], limit: number, threshold: number = 0.0): Promise<SearchHit[]> {
    const results = await this.index.query({
      vector: embedding,
      topK: limit,
      includeMetadata: true
    })

    return results.matches
      .filter(match => match.score >= threshold)
      .map(match => ({
        id: match.id,
        score: match.score,
        metadata: match.metadata
      }))
  }
}
```

**In-Memory (for testing/demo):**
```typescript
class InMemoryVectorStore implements VectorStore {
  private vectors: Map<string, { embedding: float[], metadata: object }> = new Map()

  async store(id: string, embedding: float[], metadata: object): Promise<void> {
    this.vectors.set(id, { embedding, metadata })
  }

  async search(embedding: float[], limit: number, threshold: number = 0.0): Promise<SearchHit[]> {
    const results: SearchHit[] = []

    for (const [id, data] of this.vectors.entries()) {
      const score = cosineSimilarity(embedding, data.embedding)
      if (score >= threshold) {
        results.push({ id, score, metadata: data.metadata })
      }
    }

    return results
      .sort((a, b) => b.score - a.score)
      .slice(0, limit)
  }
}
```

---

## Interface 3: LLM

Generates text responses from prompts.

### Contract

```typescript
interface LLM {
  // Generate a streaming response
  stream(
    messages: Message[],
    systemPrompt?: string,
    options?: LLMOptions
  ): AsyncIterable<string>

  // Generate a complete response (non-streaming)
  complete(
    messages: Message[],
    systemPrompt?: string,
    options?: LLMOptions
  ): Promise<LLMResponse>

  // Return model identifier
  model(): string
}

interface Message {
  role: "user" | "assistant" | "system"
  content: string
}

interface LLMOptions {
  temperature?: number      // 0.0 to 2.0, default ~0.7
  maxTokens?: number        // Max completion tokens
  stopSequences?: string[]  // Stop generation at these sequences
}

interface LLMResponse {
  content: string
  usage?: {
    promptTokens: number
    completionTokens: number
    totalTokens: number
  }
  finishReason?: "stop" | "length" | "content_filter"
}
```

### Requirements

**Streaming:**
- `stream` MUST yield string chunks as they become available
- Chunks MAY be words, partial words, or character sequences
- Stream MUST close when generation completes
- Stream MUST throw if error occurs

**Message Handling:**
- Implementations MUST support conversation history (multiple messages)
- System prompt SHOULD be prepended to conversation
- Messages MUST be processed in order

**Token Management:**
- Implementations SHOULD enforce maxTokens if specified
- Implementations SHOULD track token usage
- Implementations MAY estimate tokens if provider doesn't return counts

**Error Handling:**
- Rate limit errors SHOULD be retried with exponential backoff
- Content filter errors SHOULD be surfaced to user
- Network errors SHOULD be retried (up to 3 times)

### Example Implementations

**OpenAI:**
```typescript
class OpenAILLM implements LLM {
  constructor(apiKey: string, modelName: string = "gpt-4-turbo") {}

  async *stream(messages: Message[], systemPrompt?: string, options?: LLMOptions) {
    const allMessages = systemPrompt
      ? [{ role: "system", content: systemPrompt }, ...messages]
      : messages

    const stream = await openai.chat.completions.create({
      model: this.modelName,
      messages: allMessages,
      stream: true,
      temperature: options?.temperature,
      max_tokens: options?.maxTokens
    })

    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content
      if (content) yield content
    }
  }

  async complete(messages: Message[], systemPrompt?: string, options?: LLMOptions): Promise<LLMResponse> {
    const allMessages = systemPrompt
      ? [{ role: "system", content: systemPrompt }, ...messages]
      : messages

    const response = await openai.chat.completions.create({
      model: this.modelName,
      messages: allMessages,
      temperature: options?.temperature,
      max_tokens: options?.maxTokens
    })

    return {
      content: response.choices[0].message.content,
      usage: response.usage ? {
        promptTokens: response.usage.prompt_tokens,
        completionTokens: response.usage.completion_tokens,
        totalTokens: response.usage.total_tokens
      } : undefined,
      finishReason: response.choices[0].finish_reason as any
    }
  }

  model(): string {
    return this.modelName
  }
}
```

**Anthropic:**
```typescript
class AnthropicLLM implements LLM {
  constructor(apiKey: string, modelName: string = "claude-sonnet-4-20250514") {}

  async *stream(messages: Message[], systemPrompt?: string, options?: LLMOptions) {
    const stream = await anthropic.messages.stream({
      model: this.modelName,
      max_tokens: options?.maxTokens || 4096,
      system: systemPrompt,
      messages: messages.filter(m => m.role !== "system"),
      temperature: options?.temperature
    })

    for await (const event of stream) {
      if (event.type === 'content_block_delta' && event.delta.type === 'text_delta') {
        yield event.delta.text
      }
    }
  }

  model(): string {
    return this.modelName
  }
}
```

---

## Interface 4: DocumentProcessor (Optional)

Extracts text and metadata from uploaded files.

### Contract

```typescript
interface DocumentProcessor {
  // Process a file and extract content
  process(file: Buffer, mimeType: string): Promise<ProcessedDocument>

  // Check if processor supports this file type
  supports(mimeType: string): boolean
}

interface ProcessedDocument {
  text: string              // Extracted text content
  metadata: {
    title?: string          // Extracted or generated title
    author?: string         // Document author
    pageCount?: number      // Number of pages
    createdAt?: Date        // Document creation date
    language?: string       // Detected language (ISO 639-1)
  }
  chunks?: DocumentChunk[]  // Pre-chunked if processor does chunking
}

interface DocumentChunk {
  content: string
  metadata: {
    page?: number
    section?: string
    startChar?: number
    endChar?: number
  }
}
```

### Requirements

**Supported Types:**
- Implementations MUST support: `text/plain`, `text/markdown`
- Implementations SHOULD support: `application/pdf`
- Implementations MAY support: `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (DOCX)

**Text Extraction:**
- MUST preserve paragraph structure
- SHOULD preserve heading hierarchy
- SHOULD preserve lists and formatting where possible
- MUST handle multi-page documents

**Error Handling:**
- Invalid/corrupted files SHOULD throw clear errors
- Unsupported types SHOULD throw with supported types list
- Extraction failures SHOULD include partial results if available

---

## Implementation Checklist

To ensure proper provider abstraction, implementations should:

- [ ] Define provider interfaces in code (class/interface declarations)
- [ ] Accept providers via dependency injection (not hard-coded)
- [ ] Allow provider swapping via configuration
- [ ] Include at least 2 provider options for each interface
- [ ] Document which providers are supported
- [ ] Provide example configurations for different stacks
- [ ] Test with multiple providers
- [ ] Handle provider failures gracefully
- [ ] Log provider information at startup
- [ ] Expose provider info via GET /api/config

---

## Testing Provider Implementations

Implementations SHOULD include unit tests for each provider:

```typescript
describe('Embedder Interface', () => {
  const embedder = new OpenAIEmbedder(process.env.OPENAI_API_KEY)

  test('embeds text to correct dimension', async () => {
    const embedding = await embedder.embed("test text")
    expect(embedding.length).toBe(1536)
  })

  test('similar texts have high similarity', async () => {
    const emb1 = await embedder.embed("neural networks")
    const emb2 = await embedder.embed("artificial neural networks")
    const similarity = cosineSimilarity(emb1, emb2)
    expect(similarity).toBeGreaterThan(0.8)
  })
})
```

---

## Why This Matters

Without these interfaces, implementations would:
- Hard-code provider APIs (OpenAI-only, Anthropic-only)
- Mix business logic with provider logic
- Make swapping providers impossible
- Hide complexity behind "magic" SDKs

With these interfaces, implementations:
- ✅ Clearly separate concerns
- ✅ Enable provider comparison
- ✅ Force explicit decisions
- ✅ Support testing with mocks
- ✅ Reveal true architectural complexity

This is the difference between a framework-specific demo and a true benchmark.

---

**Next Steps:** See [required-features.md](./required-features.md) for compliance requirements and [test-suite.md](./test-suite.md) for validation criteria. A complete implementation guide is coming with the reference implementation (Mid February 2026).
