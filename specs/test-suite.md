---
title: Test Suite
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: Testing & Validation
---

# Test Suite

**Version:** 1.0.0-draft

This document defines the **automated test suite** for validating LEAF spec implementations. All implementations must pass these tests to be considered compliant with the specification.

> **Challenge:** Unlike traditional REST APIs, AI applications are non-deterministic. This test suite balances strict functional testing with flexible AI quality assessment.

## Test Categories

1. **Functional Tests** - API behavior, data integrity, security
2. **AI Quality Tests** - Response relevance, citation accuracy, hallucination detection
3. **Performance Tests** - Response times, throughput
4. **Integration Tests** - End-to-end workflows

---

## 1. Functional Tests

### 1.1 Authentication & User Management

#### Test: User Registration
```javascript
POST /api/auth/register
Body: {
  "email": "test@example.com",
  "password": "SecurePass123!",
  "displayName": "Test User"  // optional
}

Expected:
- Status: 201
- Response includes: user.id, user.email, user.displayName, token
- Token is valid JWT or equivalent
- Password NOT in response
```

#### Test: Duplicate Registration
```javascript
POST /api/auth/register (with same email)

Expected:
- Status: 400 or 409
- Error code: CONFLICT or VALIDATION_ERROR
- Message indicates duplicate email
```

#### Test: User Login
```javascript
POST /api/auth/login
Body: {
  "email": "test@example.com",
  "password": "SecurePass123!"
}

Expected:
- Status: 200
- Response includes: user, token
- Token works for authenticated endpoints
```

#### Test: Invalid Login
```javascript
POST /api/auth/login
Body: {
  "email": "test@example.com",
  "password": "WrongPassword"
}

Expected:
- Status: 401
- Error code: UNAUTHORIZED
```

#### Test: Get User Profile
```javascript
GET /api/user
Headers: Authorization: Bearer <token>

Expected:
- Status: 200
- Response includes: user.id, user.email, user.preferences, user.usage
- Password NOT in response
```

#### Test: Unauthorized Access
```javascript
GET /api/user
Headers: Authorization: Bearer invalid_token

Expected:
- Status: 401
- Error code: UNAUTHORIZED
```

---

### 1.2 Document Management

#### Test: Upload PDF Document
```javascript
POST /api/documents
Content-Type: multipart/form-data
Body: file=test.pdf, title="Test Document", tags=["test"]

Expected:
- Status: 201
- Response includes: document.id, status="processing", title, contentType, size
- Document ID is unique
```

#### Test: Create Text Document
```javascript
POST /api/documents
Body: {
  "title": "Text Note",
  "content": "This is test content about neural networks.",
  "contentType": "text/markdown",
  "tags": ["test", "notes"]
}

Expected:
- Status: 201
- Response includes: document.id, status="processing"
```

#### Test: Document Processing
```javascript
// Wait for processing (poll or webhook)
GET /api/documents/:id

Expected (within 30s):
- status transitions to "ready"
- chunkCount > 0
- processedAt is set
```

#### Test: List Documents
```javascript
GET /api/documents?limit=10&offset=0

Expected:
- Status: 200
- Response includes: documents[], pagination.total, pagination.hasMore
- Documents sorted by createdAt desc
```

#### Test: Filter Documents by Tag
```javascript
GET /api/documents?tag=test

Expected:
- Status: 200
- All returned documents have "test" tag
```

#### Test: Get Single Document
```javascript
GET /api/documents/:id

Expected:
- Status: 200
- Response includes: all document fields, content (if text-based)
```

#### Test: Update Document
```javascript
PUT /api/documents/:id
Body: {
  "title": "Updated Title"
}

Expected:
- Status: 200
- Document title updated
- updatedAt changed
```

#### Test: Delete Document
```javascript
DELETE /api/documents/:id

Expected:
- Status: 204
- Document no longer retrievable
- Embeddings removed from vector DB
```

#### Test: Access Other User's Document
```javascript
// User A creates document
// User B tries to access it
GET /api/documents/:userA_document_id
Headers: Authorization: Bearer <userB_token>

Expected:
- Status: 404 or 403
- Cannot access other user's data
```

---

### 1.3 Conversation Management

#### Test: Create Conversation
```javascript
POST /api/conversations
Body: {
  "title": "Test Conversation",
  "documentIds": ["doc_123"]
}

Expected:
- Status: 201
- Response includes: conversation.id, title, documentIds, messageCount=0
```

#### Test: List Conversations
```javascript
GET /api/conversations

Expected:
- Status: 200
- Response includes: conversations[], pagination
```

#### Test: Get Conversation with Messages
```javascript
GET /api/conversations/:id

Expected:
- Status: 200
- Response includes: conversation, messages[]
```

#### Test: Update Conversation
```javascript
PUT /api/conversations/:id
Body: {
  "title": "New Title"
}

Expected:
- Status: 200
- Title updated
```

#### Test: Delete Conversation
```javascript
DELETE /api/conversations/:id

Expected:
- Status: 204
- Conversation and all messages deleted
```

---

### 1.4 Messaging & Streaming

#### Test: Send Message (Non-Streaming)
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "What is a neural network?",
  "stream": false
}

Expected:
- Status: 201
- Response includes: userMessage, assistantMessage
- assistantMessage has: content, citations[], tokenUsage
- Citations reference valid documents
```

#### Test: Send Message (Streaming)
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "What is a neural network?",
  "stream": true
}

Expected:
- Status: 200
- Content-Type: text/event-stream
- Events received in order:
  1. message_start
  2. Multiple content_delta
  3. citations (optional)
  4. message_end
  5. done
- Stream closes properly
```

#### Test: Stream Interruption Handling
```javascript
// Client disconnects mid-stream

Expected:
- Server detects disconnect
- Cleanup resources
- No memory leaks
```

---

### 1.5 Search

#### Test: Semantic Search
```javascript
POST /api/search
Body: {
  "query": "neural network architecture",
  "limit": 5
}

Expected:
- Status: 200
- Response includes: results[] with documentId, content, relevanceScore
- Results ranked by relevance (descending)
- relevanceScore between 0.0-1.0
```

#### Test: Search with Document Filter
```javascript
POST /api/search
Body: {
  "query": "machine learning",
  "documentIds": ["doc_123"],
  "limit": 10
}

Expected:
- Status: 200
- All results from doc_123 only
```

#### Test: Search with Minimum Relevance
```javascript
POST /api/search
Body: {
  "query": "neural networks",
  "minRelevance": 0.8
}

Expected:
- Status: 200
- All results have relevanceScore >= 0.8
```

---

### 1.6 Summarization

#### Test: Single Document Summary
```javascript
POST /api/documents/:id/summarize
Body: {
  "length": "medium",
  "focus": "key_points"
}

Expected:
- Status: 200
- Response includes: summary.content, tokenUsage
- Content length appropriate for "medium"
```

#### Test: Multi-Document Summary
```javascript
POST /api/summaries
Body: {
  "documentIds": ["doc_123", "doc_456"],
  "query": "What do these documents say about AI?",
  "length": "long"
}

Expected:
- Status: 200
- Response includes: summary.content, citations[]
- Citations reference both documents
```

---

### 1.7 Error Handling

#### Test: Invalid Request Body
```javascript
POST /api/documents
Body: { invalid: "data" }

Expected:
- Status: 400 or 422
- Error includes: code, message, details
```

#### Test: Missing Required Fields
```javascript
POST /api/auth/register
Body: { "email": "test@example.com" } // missing password

Expected:
- Status: 400 or 422
- Error indicates missing field
```

#### Test: Invalid File Type
```javascript
POST /api/documents
Body: file=image.exe (invalid type)

Expected:
- Status: 400 or 415
- Error indicates unsupported file type
```

#### Test: File Too Large
```javascript
POST /api/documents
Body: file=huge_file.pdf (>50MB)

Expected:
- Status: 413
- Error code: PAYLOAD_TOO_LARGE
```

---

### 1.8 Rate Limiting

#### Test: Rate Limit Enforcement
```javascript
// Send 1001 requests rapidly

Expected:
- First 1000 succeed
- 1001st returns 429
- Response includes: X-RateLimit-* headers
- Error includes retry-after
```

---

## 2. AI Quality Tests

These tests use **standardized test documents** and evaluate AI response quality.

### Test Documents

**Document A: "Neural Networks Basics"**
- Content includes: definitions, architecture, training
- ~2000 words, 5 pages

**Document B: "Machine Learning Overview"**
- Content includes: supervised learning, unsupervised learning, ML types
- ~1500 words, 4 pages

**Document C: "AI Ethics"**
- Content includes: bias, fairness, accountability
- ~1800 words, 4 pages

### 2.1 Citation Accuracy

#### Test: Citation Exists
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "What is a neural network?"
}

Evaluation:
✅ PASS: Response includes at least one citation
✅ PASS: Citation references Document A
✅ PASS: Citation excerpt found in Document A
❌ FAIL: No citations or incorrect document
```

#### Test: Multiple Citation Relevance
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "How do neural networks and machine learning relate?"
}

Evaluation:
✅ PASS: Citations include both Document A and B
✅ PASS: Citations relevant to question
❌ FAIL: Missing one document or irrelevant citations
```

#### Test: No Hallucination in Citations
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "What does the document say about quantum computing?"
}

Evaluation:
✅ PASS: Response says "no information found" or similar
✅ PASS: No citations, or citations explicitly state uncertainty
❌ FAIL: Hallucinates quantum computing content
❌ FAIL: Citations don't match claimed source
```

### 2.2 Response Relevance

#### Test: Relevance to Query
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "Explain the backpropagation algorithm"
}

Evaluation (manual or LLM-as-judge):
✅ PASS: Response discusses backpropagation
✅ PASS: Information from documents
❌ FAIL: Off-topic response
```

#### Test: Context Awareness
```javascript
// Message 1
POST /api/conversations/:id/messages
Body: { "content": "What is a neural network?" }

// Message 2 (follow-up)
POST /api/conversations/:id/messages
Body: { "content": "How is it trained?" }

Evaluation:
✅ PASS: Response understands "it" refers to neural network
✅ PASS: Discusses training methods
❌ FAIL: Loses context, asks "what is 'it'?"
```

### 2.3 Grounding in Documents

#### Test: No Hallucination
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "What datasets are used in the documents?"
}

Evaluation:
✅ PASS: Only mentions datasets actually in documents
✅ PASS: Says "not mentioned" if not in docs
❌ FAIL: Invents datasets not in source
```

### 2.4 Summary Quality

#### Test: Summary Accuracy
```javascript
POST /api/documents/:id/summarize
Body: { "length": "short", "focus": "key_points" }

Evaluation:
✅ PASS: Summary captures main themes
✅ PASS: Length ~100-200 words
✅ PASS: No fabricated information
❌ FAIL: Missing key points or hallucinated content
```

### 2.5 Search Quality

#### Test: Search Result Relevance
```javascript
POST /api/search
Body: {
  "query": "training neural networks",
  "limit": 5
}

Evaluation:
✅ PASS: Top results discuss neural network training
✅ PASS: Relevance scores > 0.6 for top 3
❌ FAIL: Irrelevant results ranked highly
```

---

## 3. Performance Tests

### 3.1 Response Time Benchmarks

#### Test: Document Upload Response
```javascript
Measure time to receive 201 response

Target: < 1 second
Acceptable: < 3 seconds
```

#### Test: First Token Latency (Streaming)
```javascript
Measure time to first content_delta event

Target: < 500ms
Acceptable: < 2 seconds
```

#### Test: Search Response
```javascript
Measure time to receive search results

Target: < 1 second
Acceptable: < 3 seconds
```

### 3.2 Throughput

#### Test: Concurrent Requests
```javascript
Send 10 concurrent message requests

Evaluation:
✅ PASS: All complete successfully
✅ PASS: Average response time < 5s
❌ FAIL: Timeouts or errors
```

---

## 4. Integration Tests

### 4.1 End-to-End: New User Journey
```javascript
1. Register new user
2. Upload 3 documents (PDF, TXT, MD)
3. Wait for processing
4. Create conversation
5. Send question
6. Verify response with citations
7. Send follow-up question
8. Verify context maintained
9. Search across documents
10. Generate summary

Expected:
✅ All steps succeed
✅ Data consistent across operations
```

### 4.2 End-to-End: Document Lifecycle
```javascript
1. Upload document
2. Wait for processing
3. Query document content
4. Update document content
5. Wait for reprocessing
6. Query again (should reflect update)
7. Delete document
8. Verify removed from search

Expected:
✅ All steps succeed
✅ Updates reflected correctly
✅ Deletion complete
```

---

## 5. Security Tests

### 5.1 Data Isolation
```javascript
// User A uploads document
// User B attempts access

Expected:
❌ FAIL if User B can access User A's document
✅ PASS if properly isolated
```

### 5.2 SQL Injection / Injection Attacks
```javascript
POST /api/conversations/:id/messages
Body: {
  "content": "'; DROP TABLE users; --"
}

Expected:
✅ PASS: Treated as regular query
✅ PASS: No database corruption
```

### 5.3 XSS Prevention
```javascript
POST /api/documents
Body: {
  "title": "<script>alert('xss')</script>",
  "content": "Test"
}

Expected:
✅ PASS: Script tags escaped in responses
✅ PASS: Not executed in frontend
```

---

## 6. Edge Case "Turing Tests"

These three scenarios test advanced architectural capabilities that reveal framework maturity. All implementations **MUST** handle these correctly to be considered production-ready.

> **Why "Turing Tests"?** Just as the Turing Test evaluates AI intelligence, these tests evaluate an implementation's ability to handle AI edge cases intelligently - not just the happy path.

---

### 6.1. Hallucination Guard Test

**Scenario:** User asks a question with no relevant documents in the knowledge base.

**Purpose:** Tests business logic intervention in the AI pipeline. Implementations must prevent the LLM from fabricating answers when there's no supporting context.

#### Test Setup
```javascript
// 1. Upload test document
POST /api/documents
Body: file=lotr.pdf  // The Lord of the Rings (full text)

// 2. Wait for processing
GET /api/documents/:id
Expected: status="ready"

// 3. Ask unrelated question
POST /api/conversations/:id/messages
Body: {
  "content": "What is the capital of Mars?"
}
```

#### Expected Behavior

**Retrieval Phase:**
```javascript
// Vector search runs
// Returns chunks about Middle-earth geography
// Highest similarity score: 0.35 (far below threshold)
```

**Logic Check:**
```javascript
if (highestSimilarityScore < 0.75) {
  // MUST NOT call LLM generation
  return {
    "content": "I cannot find this information in your knowledge base.",
    "citations": [],
    "confidence": "none"
  }
}
```

#### Grading Criteria

❌ **FAIL** - Hallucinates an answer
```javascript
{
  "content": "The capital of Mars is Olympus City...",
  "citations": [{ "documentId": "lotr.pdf" }]  // Invalid citation
}
```

⚠️ **PARTIAL** - Returns LLM response with low-quality context
```javascript
{
  "content": "Based on the documents, I don't have information about Mars, but...",
  "citations": []  // Better, but still unnecessary LLM call
}
```

✅ **PASS** - Returns canned "not found" response without LLM call
```javascript
{
  "content": "I cannot find this information in your knowledge base.",
  "citations": [],
  "confidence": "none",
  "retrievedChunks": 5,
  "maxSimilarity": 0.35  // Shows why it failed
}
```

#### Why This Matters

- **Cost:** Prevents expensive LLM calls for irrelevant queries
- **Trust:** Prevents hallucinated citations that break user confidence
- **Architecture:** Tests business logic layer between retrieval and generation

---

### 6.2. Context Squeeze Test

**Scenario:** User asks a broad question that retrieves more context than fits in the model's context window.

**Purpose:** Tests complex orchestration and context management strategies. Implementations must handle token limits gracefully without silently truncating.

#### Test Setup
```javascript
// 1. Upload 20 PDF documents (total ~500 pages)
POST /api/documents (20 times with different files)
// Files: financial_reports_2020-2024.pdf (5 years × 4 quarters)

// 2. Wait for all processing
GET /api/documents
Expected: All 20 documents with status="ready"

// 3. Ask broad question
POST /api/conversations/:id/messages
Body: {
  "content": "Summarize all financial projections across all quarterly reports."
}
```

#### Expected Behavior

**Retrieval Phase:**
```javascript
// Vector search returns 50+ relevant chunks
// Total tokens: ~20,000 (retrieved chunks)
// Model limit: 8,000 tokens (context window)
// Problem: Cannot fit all context
```

**Context Management Strategy (Implementation Must Choose One):**

**Option A: Map-Reduce**
```javascript
1. Summarize each document individually (5 passes)
2. Combine summaries into final answer (1 pass)
3. Total: 6 LLM calls, fits within limits

Response includes:
{
  "content": "Across all quarterly reports...",
  "citations": [...],  // From all 20 documents
  "strategy": "map-reduce",
  "passes": 6
}
```

**Option B: Hierarchical Ranking**
```javascript
1. Rank chunks by similarity
2. Select top N chunks that fit in context
3. Generate answer from selected chunks
4. Document which sources were prioritized

Response includes:
{
  "content": "Based on the most relevant sections...",
  "citations": [...],  // From top 8 documents only
  "strategy": "ranked-selection",
  "selectedDocuments": 8,
  "totalDocuments": 20
}
```

**Option C: Iterative Refinement**
```javascript
1. Generate initial summary from top 10 chunks
2. Review remaining chunks against initial summary
3. Refine answer with new information
4. Multiple passes until convergence

Response includes:
{
  "content": "Financial projections show...",
  "citations": [...],
  "strategy": "iterative-refinement",
  "iterations": 3
}
```

#### Grading Criteria

❌ **FAIL** - Crashes or returns error
```javascript
{
  "error": "CONTEXT_LIMIT_EXCEEDED",
  "message": "Too many results"
}
```

⚠️ **PARTIAL** - Truncates context silently
```javascript
{
  "content": "Based on Q1 2024 report...",  // Only used first chunk
  "citations": [/* only 1 document */]  // Lost 19 documents
}
// No indication that context was limited
```

✅ **PASS** - Implements explicit context management
```javascript
{
  "content": "Comprehensive summary...",
  "citations": [/* from multiple documents */],
  "metadata": {
    "strategy": "map-reduce",  // or other strategy
    "totalChunks": 50,
    "processedChunks": 50,
    "contextManagement": "applied"
  }
}
```

#### Why This Matters

- **Completeness:** Users expect answers that consider all relevant documents
- **Transparency:** Shows how implementation handles real-world constraints
- **Architecture:** Reveals orchestration patterns for multi-step AI workflows

---

### 6.3. Multimodal Ingestion Test ⭕ OPTIONAL EXTENSION

**Scope:** This test is for OPTIONAL EXTENSION implementations only.
- ✅ **Core Certification:** Does NOT require this test - text-only is fully compliant
- 🟢 **Multimodal Certification:** Requires passing this test for extended cert

**Scenario:** User uploads a non-text file (image, chart, diagram) that requires vision model processing.

**Purpose:** Tests chaining different model modalities and extracting knowledge from non-text formats.

#### Test Setup
```javascript
// 1. Upload image file
POST /api/documents
Body: file=quarterly_revenue_chart.png
// PNG chart showing bar graph of Q1-Q4 revenue

Expected:
- Status: 202 Accepted
- Document created with mime_type="image/png"
```

#### Expected Behavior

**Processing Pipeline:**
```javascript
1. Detect MIME type: image/png
2. Route to vision model processor
3. Call vision model (e.g., GPT-4o, Claude 3.5 with vision)
   Prompt: "Describe this image in detail for retrieval purposes.
            Extract all text, data, and visual information."
4. Store description as searchable text
5. Generate embedding from description
6. Mark document as status="ready"
```

**Vision Model Output:**
```
This is a bar chart showing quarterly revenue for 2024:
Q1: $2.3M (blue bar)
Q2: $2.8M (blue bar)
Q3: $3.1M (blue bar)
Q4: $3.5M (blue bar, projected)

The chart shows 35% year-over-year growth.
The title reads "2024 Revenue Projections".
```

**Queryability Test:**
```javascript
// User should be able to query the image content
POST /api/conversations/:id/messages
Body: {
  "content": "What was Q3 revenue in 2024?"
}

Expected:
{
  "content": "According to the revenue chart, Q3 2024 revenue was $3.1M.",
  "citations": [{
    "documentId": "quarterly_revenue_chart.png",
    "content": "Q3: $3.1M (blue bar)..."
  }]
}
```

#### Grading Criteria

❌ **FAIL** - Rejects image files
```javascript
{
  "error": "UNSUPPORTED_FILE_TYPE",
  "message": "Only text and PDF files supported"
}
```

⚠️ **PARTIAL** - Stores filename only
```javascript
// Document created but not processed
// User queries return no results
// No vision model integration
```

✅ **PASS** - Extracts and embeds image description
```javascript
// Document processed with vision model
// Description stored and embedded
// Queries return relevant information
// Citations link back to image file
```

#### Extension Tiers

**Tier 1 (Required for "Multimodal" certification):**
- ✅ PNG, JPG image support
- ✅ Vision model integration
- ✅ Text extraction from images
- ✅ Searchable image descriptions

**Tier 2 (Optional):**
- 🔶 OCR for scanned documents
- 🔶 Chart/diagram understanding
- 🔶 Audio transcription
- 🔶 Video analysis

**Note:** Implementations CAN choose to be "text-only" and skip this test. Mark as:
- 🟡 "Text-Only Implementation" - Valid, but limited scope
- 🟢 "Multimodal Implementation" - Passes this test

#### Why This Matters

- **Real-World Usage:** Users upload screenshots, charts, whiteboards
- **Model Chaining:** Tests ability to orchestrate multiple AI models
- **Format Flexibility:** Shows implementation can handle diverse inputs

---

## 6.4. Summary: Edge Case Grading

| Test | Critical? | Purpose | Passing Score |
|------|-----------|---------|---------------|
| **Hallucination Guard** | 🔴 Required | Prevent fabricated answers | Must pass |
| **Context Squeeze** | 🟡 Important | Handle token limits gracefully | 80% score |
| **Multimodal Ingestion** | 🟢 Optional | Chain vision models (if supported) | N/A for text-only |

**Certification Requirements:**
- ✅ **Text-Only Certification**: Pass Hallucination Guard + Context Squeeze
- ✅ **Multimodal Certification**: Pass all three tests

---

## Test Execution

### Running the Test Suite

```bash
# Install test runner
npm install -g ai-realworld-tests

# Configure API URL
export API_URL=http://localhost:3000/api

# Run full test suite
ai-realworld-tests run --all

# Run specific category
ai-realworld-tests run --functional
ai-realworld-tests run --ai-quality
ai-realworld-tests run --performance

# Run with test documents
ai-realworld-tests run --documents=./test-docs
```

### Test Output

```
LEAF Test Suite v1.0.0
======================

Functional Tests: 45/45 passed ✅
AI Quality Tests: 8/10 passed ⚠️
  - Citation Exists: PASS
  - Multiple Citations: PASS
  - No Hallucination: FAIL (hallucinated quantum computing)
  - Response Relevance: PASS
  - Context Awareness: PASS
  - Grounding: PASS
  - Summary Accuracy: FAIL (too short)
  - Search Relevance: PASS

Performance Tests: 5/6 passed ⚠️
  - Upload Response: PASS (650ms)
  - First Token: PASS (420ms)
  - Search Response: FAIL (3.2s - exceeds 3s acceptable)

Integration Tests: 2/2 passed ✅

Security Tests: 5/5 passed ✅

Overall: 65/68 tests passed (95.6%)

Certification: COMPLIANT ✅
(Minimum 90% required, all critical tests passed)
```

---

## Compliance Thresholds

To be **certified compliant**, implementations must:

- ✅ Pass 100% of functional tests (critical)
- ✅ Pass 80%+ of AI quality tests
- ✅ Pass 100% of security tests (critical)
- ✅ Pass 80%+ of integration tests
- 🔶 Pass 70%+ of performance tests (recommended)

---

## Implementation Grading Rubric

Beyond pass/fail test results, reviewers and evaluators should assess implementations on these architectural dimensions. This rubric helps compare frameworks objectively and reveals hidden complexity.

### 1. Async Architecture Quality

**Question:** How well does the framework handle asynchronous AI operations?

**Test Methodology:**
- Upload 50MB PDF
- Measure API response time (should be < 200ms for 202 Accepted)
- Check if background job system is visible
- Monitor CPU/memory during processing

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Response < 200ms
- Native job queue support
- Observable job status
- No main thread blocking
- Example: FastAPI + Celery, Next.js + BullMQ

⭐⭐⭐⭐ **Good** (7/10 points)
- Response < 1s
- Job queue via library
- Some observability
- Example: Express + custom workers

⭐⭐ **Poor** (3/10 points)
- Response > 5s
- Synchronous processing
- Blocks main thread
- Example: Basic Flask without workers

**Key Insight:** Frameworks with native async primitives (Go, Rust, Node) vs. frameworks requiring external workers (Python WSGI)

---

### 2. Vector Database Integration

**Question:** How verbose and complex is vector similarity code?

**Test Methodology:**
- Count lines of code for implementing similarity search
- Check if ORM has native vector support
- Evaluate if raw SQL is required

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Native ORM vector support (< 10 lines)
- Example: Supabase JS client with pgvector, Prisma with vector extension
```typescript
const results = await db.chunks.findMany({
  orderBy: vector.l2Distance(embedding, 'asc'),
  take: 5
})
```

⭐⭐⭐⭐ **Good** (7/10 points)
- Library abstraction (< 50 lines)
- Example: LangChain vector stores
```python
vectorstore = Chroma(...)
results = vectorstore.similarity_search(query, k=5)
```

⭐⭐ **Poor** (3/10 points)
- Raw SQL required (> 100 lines)
```sql
SELECT id, content,
  1 - (embedding <=> '[0.1,0.2,...]'::vector) AS similarity
FROM chunks
ORDER BY embedding <=> '[0.1,0.2,...]'::vector
LIMIT 5;
```

**Key Insight:** ORMs without vector support force SQL, revealing database abstraction limitations

---

### 3. Streaming Architecture

**Question:** Does the framework fight against or embrace streaming?

**Test Methodology:**
- Implement SSE endpoint for streaming chat
- Check for buffering issues
- Test with different HTTP servers
- Monitor for manual flush calls

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Native streaming support
- No configuration required
- No buffering issues
- Example: FastAPI with StreamingResponse, Next.js Route Handlers
```python
async def stream():
    for chunk in llm.stream():
        yield chunk
return StreamingResponse(stream())
```

⭐⭐⭐⭐ **Good** (7/10 points)
- Requires middleware or configuration
- Works reliably after setup
- Example: Express with compression middleware disabled

⭐⭐ **Poor** (3/10 points)
- Buffering issues
- Requires manual flush calls
- Framework fights against streaming
- Example: Python WSGI servers (Gunicorn, uWSGI)

**Key Insight:** ASGI vs WSGI, frameworks designed for streaming vs retrofitted

---

### 4. Provider Abstraction

**Question:** How easy is it to swap AI providers (OpenAI → Anthropic)?

**Test Methodology:**
- Change from OpenAI to Anthropic
- Count files that need modification
- Evaluate if interface/adapter pattern is natural

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Interface-based design
- Change 1 config value or environment variable
- Example: Implementation uses provider interfaces, swap via DI
```typescript
// .env change: LLM_PROVIDER=anthropic
// No code changes needed
```

⭐⭐⭐⭐ **Good** (7/10 points)
- Adapter pattern
- Change 1-2 classes
- Clear abstraction boundary

⭐⭐ **Poor** (3/10 points)
- Find/replace throughout codebase
- Provider-specific code scattered
- No abstraction layer

**Key Insight:** Languages with interfaces (TypeScript, Java, Go) vs dynamic languages (Python, Ruby)

---

### 5. Error Handling Architecture

**Question:** How are AI errors (rate limits, model failures) surfaced?

**Test Methodology:**
- Trigger rate limit error (too many requests)
- Trigger model error (invalid request)
- Check error messages returned to user
- Evaluate retry logic

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Typed error system (Result<T, E> or similar)
- Graceful degradation
- Retry logic with exponential backoff
- User-friendly error messages
```typescript
return {
  error: {
    code: "RATE_LIMIT_EXCEEDED",
    message: "Too many requests. Try again in 5 minutes.",
    retryAfter: 300
  }
}
```

⭐⭐⭐⭐ **Good** (7/10 points)
- Try/catch with custom messages
- Some retry logic
- Errors logged

⭐⭐ **Poor** (3/10 points)
- Silent failures
- Raw stack traces to user
- No retry mechanism

**Key Insight:** Type systems (Rust, TypeScript) vs runtime errors (Python, JavaScript)

---

### 6. Cost Tracking Implementation

**Question:** Can users see token usage and costs?

**Test Methodology:**
- Check if token counts are returned in API
- Evaluate if costs are calculated
- Check for dashboard or tracking

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- Real-time cost dashboard
- Token counts in every response
- Historical cost tracking
- Alerts for budget limits

⭐⭐⭐⭐ **Good** (7/10 points)
- Token counts in API response metadata
- Basic cost calculation
```json
{
  "content": "...",
  "usage": {
    "promptTokens": 1234,
    "completionTokens": 567,
    "totalCost": 0.0234
  }
}
```

⭐⭐ **Poor** (3/10 points)
- No visibility into usage
- No cost tracking
- Users blind to expenses

**Key Insight:** Production-ready vs proof-of-concept implementations

---

### 7. Code Complexity & Maintainability

**Question:** How much boilerplate and complexity does the framework require?

**Test Methodology:**
- Count total lines of code
- Evaluate dependency count
- Measure cognitive complexity
- Check for framework-specific ceremony

**Grading Scale:**

⭐⭐⭐⭐⭐ **Excellent** (10/10 points)
- < 2000 LOC for complete implementation
- < 20 dependencies
- Clear separation of concerns
- Minimal boilerplate

⭐⭐⭐⭐ **Good** (7/10 points)
- 2000-5000 LOC
- 20-40 dependencies
- Some framework ceremony

⭐⭐ **Poor** (3/10 points)
- > 5000 LOC
- > 40 dependencies
- Heavy framework ceremony
- Excessive abstraction layers

**Key Insight:** Framework magic vs explicit code trade-offs

---

### Overall Implementation Score

**Total Points:** Sum of all categories (Max 70 points)

**Rating Tiers:**

🥇 **Outstanding** (60-70 points)
- Production-ready reference implementation
- Framework strengths clearly demonstrated
- Few compromises or workarounds

🥈 **Strong** (45-59 points)
- Solid implementation with minor rough edges
- Framework mostly suitable for task
- Some workarounds needed

🥉 **Acceptable** (30-44 points)
- Functional but reveals framework limitations
- Significant compromises required
- Educational about trade-offs

⚠️ **Needs Improvement** (< 30 points)
- Framework poorly suited for AI workloads
- Major architectural issues
- Requires heavy customization

---

### Example Scorecards

**Next.js + Vercel AI SDK + Supabase:**
- Async Architecture: 10/10 (native job support)
- Vector Integration: 10/10 (Supabase client)
- Streaming: 10/10 (Route Handlers)
- Provider Abstraction: 10/10 (Vercel AI SDK)
- Error Handling: 8/10 (good try/catch)
- Cost Tracking: 7/10 (manual implementation)
- Code Complexity: 9/10 (< 2000 LOC)
- **Total: 64/70 (Outstanding)**

**Python FastAPI + Raw APIs + pgvector:**
- Async Architecture: 10/10 (FastAPI + Celery)
- Vector Integration: 5/10 (raw SQL required)
- Streaming: 10/10 (StreamingResponse)
- Provider Abstraction: 7/10 (adapter pattern)
- Error Handling: 8/10 (custom exceptions)
- Cost Tracking: 9/10 (detailed tracking)
- Code Complexity: 6/10 (~4000 LOC)
- **Total: 55/70 (Strong)**

**Flask + LangChain:**
- Async Architecture: 3/10 (WSGI blocking)
- Vector Integration: 8/10 (LangChain abstraction)
- Streaming: 3/10 (buffering issues)
- Provider Abstraction: 9/10 (LangChain)
- Error Handling: 5/10 (basic try/catch)
- Cost Tracking: 4/10 (minimal)
- Code Complexity: 7/10 (~3000 LOC)
- **Total: 39/70 (Acceptable)**

---

**Purpose of This Rubric:**

This grading system helps:
1. **Learners** understand which frameworks excel at AI workloads
2. **Framework creators** identify areas for improvement
3. **Teams** make informed technology decisions
4. **Community** compare implementations objectively

The goal is **not** to crown a "winner" but to **reveal architectural trade-offs** - just like RealWorld revealed that Django has more boilerplate than Express, but better ORM than raw Node.

---

## Test Document Repository

Standard test documents and fixtures available at:
```
https://github.com/ai-realworld/test-fixtures
```

Includes:
- Sample PDFs, text files, markdown docs
- Pre-computed embeddings (for validation)
- Expected responses (for quality checks)
- Test conversation transcripts

---

**Next:** See [ui-requirements.md](./ui-requirements.md) for frontend specifications.
