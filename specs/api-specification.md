---
title: API Specification
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: API Contract
---

# API Specification

**Version:** 1.0.0-draft
**Base URL:** `https://api.example.com/api`
**Content Type:** `application/json`
**Auth:** Bearer token in `Authorization` header

> **Note:** This specification is framework and implementation agnostic. All endpoints are defined by their HTTP interface contract, not internal implementation details.

## Table of Contents

- [Authentication](#authentication)
- [Documents](#documents)
- [Conversations](#conversations)
- [Messages](#messages)
- [Search](#search)
- [Summaries](#summaries)
- [User Profile](#user-profile)
- [Configuration](#configuration)
- [Health Check](#health-check)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)

---

## Authentication

All authenticated endpoints require a valid JWT token in the `Authorization` header:

```http
Authorization: Bearer <token>
```

### POST /api/auth/register

Register a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword",
  "username": "johndoe"
}
```

**Response:** `201 Created`
```json
{
  "user": {
    "id": "usr_123abc",
    "email": "user@example.com",
    "username": "johndoe",
    "createdAt": "2024-01-15T10:00:00Z"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### POST /api/auth/login

Authenticate an existing user.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securepassword"
}
```

**Response:** `200 OK`
```json
{
  "user": {
    "id": "usr_123abc",
    "email": "user@example.com",
    "username": "johndoe"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Documents

### POST /api/documents

Upload or create a new document.

**Auth Required:** Yes

**Request:**
- For file upload, use `multipart/form-data`
- For text document, use `application/json`

**Multipart Upload:**
```
POST /api/documents
Content-Type: multipart/form-data

file: [PDF/TXT/MD file]
title: "My Document" (optional)
tags: ["research", "ai"] (optional)
```

**JSON Request (text document):**
```json
{
  "title": "My Notes",
  "content": "Document content in plain text or markdown...",
  "contentType": "text/markdown",
  "tags": ["notes", "personal"]
}
```

**Response:** `201 Created`
```json
{
  "document": {
    "id": "doc_456def",
    "userId": "usr_123abc",
    "title": "My Document",
    "contentType": "application/pdf",
    "size": 245680,
    "status": "processing",
    "tags": ["research", "ai"],
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z",
    "processedAt": null,
    "chunkCount": null,
    "url": "https://storage.example.com/documents/doc_456def.pdf"
  }
}
```

**Status Field:**
- `processing` - Document is being processed/embedded
- `ready` - Document is ready for queries
- `failed` - Processing failed

### GET /api/documents

List all documents for the authenticated user.

**Auth Required:** Yes

**Query Parameters:**
- `limit` (optional, default: 20, max: 100) - Number of documents per page
- `offset` (optional, default: 0) - Pagination offset
- `tag` (optional) - Filter by tag
- `status` (optional) - Filter by status (processing, ready, failed)
- `sortBy` (optional, default: createdAt) - Sort field (createdAt, updatedAt, title)
- `sortOrder` (optional, default: desc) - Sort order (asc, desc)

**Response:** `200 OK`
```json
{
  "documents": [
    {
      "id": "doc_456def",
      "title": "My Document",
      "contentType": "application/pdf",
      "size": 245680,
      "status": "ready",
      "tags": ["research", "ai"],
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T10:30:00Z",
      "processedAt": "2024-01-15T10:31:45Z",
      "chunkCount": 23
    }
  ],
  "pagination": {
    "total": 45,
    "limit": 20,
    "offset": 0,
    "hasMore": true
  }
}
```

### GET /api/documents/:id

Get a single document by ID.

**Auth Required:** Yes

**Response:** `200 OK`
```json
{
  "document": {
    "id": "doc_456def",
    "userId": "usr_123abc",
    "title": "My Document",
    "content": "Full text content if text-based...",
    "contentType": "application/pdf",
    "size": 245680,
    "status": "ready",
    "tags": ["research", "ai"],
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z",
    "processedAt": "2024-01-15T10:31:45Z",
    "chunkCount": 23,
    "url": "https://storage.example.com/documents/doc_456def.pdf",
    "metadata": {
      "author": "John Doe",
      "pages": 12
    }
  }
}
```

### PUT /api/documents/:id

Update a document's metadata or content.

**Auth Required:** Yes

**Request Body:**
```json
{
  "title": "Updated Title",
  "content": "Updated content for text documents",
  "tags": ["updated", "tags"]
}
```

**Response:** `200 OK`
```json
{
  "document": {
    "id": "doc_456def",
    "title": "Updated Title",
    "status": "processing",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### DELETE /api/documents/:id

Delete a document and all associated embeddings.

**Auth Required:** Yes

**Response:** `204 No Content`

---

## Conversations

### POST /api/conversations

Create a new conversation thread.

**Auth Required:** Yes

**Request Body:**
```json
{
  "title": "Questions about AI",
  "documentIds": ["doc_456def", "doc_789ghi"]
}
```

**Note:** `documentIds` is optional. If provided, conversation will be scoped to those documents.

**Response:** `201 Created`
```json
{
  "conversation": {
    "id": "conv_abc123",
    "userId": "usr_123abc",
    "title": "Questions about AI",
    "documentIds": ["doc_456def", "doc_789ghi"],
    "messageCount": 0,
    "createdAt": "2024-01-15T11:00:00Z",
    "updatedAt": "2024-01-15T11:00:00Z"
  }
}
```

### GET /api/conversations

List all conversations for the authenticated user.

**Auth Required:** Yes

**Query Parameters:**
- `limit` (optional, default: 20, max: 100)
- `offset` (optional, default: 0)
- `sortBy` (optional, default: updatedAt)
- `sortOrder` (optional, default: desc)

**Response:** `200 OK`
```json
{
  "conversations": [
    {
      "id": "conv_abc123",
      "title": "Questions about AI",
      "messageCount": 12,
      "lastMessage": {
        "role": "assistant",
        "content": "Based on your documents...",
        "createdAt": "2024-01-15T11:05:00Z"
      },
      "createdAt": "2024-01-15T11:00:00Z",
      "updatedAt": "2024-01-15T11:05:00Z"
    }
  ],
  "pagination": {
    "total": 8,
    "limit": 20,
    "offset": 0,
    "hasMore": false
  }
}
```

### GET /api/conversations/:id

Get a conversation with all messages.

**Auth Required:** Yes

**Query Parameters:**
- `limit` (optional, default: 50) - Number of messages to return
- `before` (optional) - Return messages before this message ID
- `after` (optional) - Return messages after this message ID

**Response:** `200 OK`
```json
{
  "conversation": {
    "id": "conv_abc123",
    "userId": "usr_123abc",
    "title": "Questions about AI",
    "documentIds": ["doc_456def"],
    "createdAt": "2024-01-15T11:00:00Z",
    "updatedAt": "2024-01-15T11:05:00Z"
  },
  "messages": [
    {
      "id": "msg_001",
      "conversationId": "conv_abc123",
      "role": "user",
      "content": "What does my document say about neural networks?",
      "createdAt": "2024-01-15T11:01:00Z"
    },
    {
      "id": "msg_002",
      "conversationId": "conv_abc123",
      "role": "assistant",
      "content": "Based on your document, neural networks are...",
      "citations": [
        {
          "documentId": "doc_456def",
          "documentTitle": "My Document",
          "chunkId": "chunk_123",
          "excerpt": "Neural networks are composed of layers...",
          "relevanceScore": 0.92,
          "page": 5
        }
      ],
      "tokenUsage": {
        "prompt": 1250,
        "completion": 180,
        "total": 1430
      },
      "createdAt": "2024-01-15T11:01:05Z"
    }
  ],
  "pagination": {
    "hasMore": false,
    "before": null,
    "after": null
  }
}
```

### PUT /api/conversations/:id

Update conversation metadata.

**Auth Required:** Yes

**Request Body:**
```json
{
  "title": "Updated Title",
  "documentIds": ["doc_456def", "doc_999zzz"]
}
```

**Response:** `200 OK`
```json
{
  "conversation": {
    "id": "conv_abc123",
    "title": "Updated Title",
    "documentIds": ["doc_456def", "doc_999zzz"],
    "updatedAt": "2024-01-15T11:10:00Z"
  }
}
```

### DELETE /api/conversations/:id

Delete a conversation and all its messages.

**Auth Required:** Yes

**Response:** `204 No Content`

---

## Messages

### POST /api/conversations/:id/messages

Send a message in a conversation and get AI response.

**Auth Required:** Yes

**Request Body:**
```json
{
  "content": "What are the main themes in my documents?",
  "documentIds": ["doc_456def"],
  "stream": true
}
```

**Note:**
- `documentIds` (optional) - Override conversation's default documents
- `stream` (optional, default: false) - Enable Server-Sent Events streaming

**Response (Non-Streaming):** `201 Created`
```json
{
  "userMessage": {
    "id": "msg_003",
    "conversationId": "conv_abc123",
    "role": "user",
    "content": "What are the main themes in my documents?",
    "createdAt": "2024-01-15T11:15:00Z"
  },
  "assistantMessage": {
    "id": "msg_004",
    "conversationId": "conv_abc123",
    "role": "assistant",
    "content": "The main themes in your documents are...",
    "citations": [
      {
        "documentId": "doc_456def",
        "documentTitle": "My Document",
        "chunkId": "chunk_456",
        "excerpt": "The primary focus is on...",
        "relevanceScore": 0.88,
        "page": 3
      }
    ],
    "relatedDocuments": ["doc_789ghi"],
    "tokenUsage": {
      "prompt": 2340,
      "completion": 245,
      "total": 2585
    },
    "createdAt": "2024-01-15T11:15:08Z"
  }
}
```

**Response (Streaming):** `200 OK`

Content-Type: `text/event-stream`

```
event: message_start
data: {"messageId":"msg_004","conversationId":"conv_abc123"}

event: content_delta
data: {"delta":"The"}

event: content_delta
data: {"delta":" main"}

event: content_delta
data: {"delta":" themes"}

event: citations
data: {"citations":[{"documentId":"doc_456def","documentTitle":"My Document","excerpt":"...","relevanceScore":0.88}]}

event: message_end
data: {"messageId":"msg_004","tokenUsage":{"prompt":2340,"completion":245,"total":2585},"finishReason":"stop"}

event: done
data: {}
```

**Streaming Event Types:**
- `message_start` - Message generation started
- `content_delta` - Incremental content chunk
- `citations` - Citations found (sent once after content)
- `message_end` - Message complete with metadata
- `error` - Error occurred
- `done` - Stream complete

---

## Search

### POST /api/search

Perform semantic search across documents.

**Auth Required:** Yes

**Request Body:**
```json
{
  "query": "machine learning techniques",
  "documentIds": ["doc_456def", "doc_789ghi"],
  "limit": 10,
  "minRelevance": 0.7
}
```

**Note:**
- `documentIds` (optional) - Limit search to specific documents
- `limit` (optional, default: 10, max: 50) - Number of results
- `minRelevance` (optional, default: 0.0, range: 0-1) - Minimum relevance score

**Response:** `200 OK`
```json
{
  "results": [
    {
      "documentId": "doc_456def",
      "documentTitle": "My Document",
      "chunkId": "chunk_789",
      "content": "Machine learning techniques include supervised...",
      "relevanceScore": 0.94,
      "metadata": {
        "page": 7,
        "section": "Chapter 3"
      }
    },
    {
      "documentId": "doc_789ghi",
      "documentTitle": "Another Doc",
      "chunkId": "chunk_234",
      "content": "Various ML approaches can be categorized...",
      "relevanceScore": 0.86,
      "metadata": {
        "page": 12
      }
    }
  ],
  "query": "machine learning techniques",
  "total": 2
}
```

---

## Summaries

### POST /api/documents/:id/summarize

Generate a summary of a single document.

**Auth Required:** Yes

**Request Body:**
```json
{
  "length": "medium",
  "focus": "key_points"
}
```

**Note:**
- `length` (optional, default: medium) - Values: `short`, `medium`, `long`
- `focus` (optional, default: general) - Values: `general`, `key_points`, `technical`, `conclusions`

**Response:** `200 OK`
```json
{
  "summary": {
    "id": "sum_xyz789",
    "documentId": "doc_456def",
    "content": "This document discusses the fundamentals of neural networks...",
    "length": "medium",
    "focus": "key_points",
    "tokenUsage": {
      "prompt": 3450,
      "completion": 280,
      "total": 3730
    },
    "createdAt": "2024-01-15T11:20:00Z"
  }
}
```

### POST /api/summaries

Generate a summary across multiple documents or a topic.

**Auth Required:** Yes

**Request Body:**
```json
{
  "documentIds": ["doc_456def", "doc_789ghi"],
  "query": "How do these documents discuss AI safety?",
  "length": "long"
}
```

**Note:**
- `documentIds` (required) - Documents to summarize
- `query` (optional) - Focus the summary on a specific question/topic
- `length` (optional, default: medium)

**Response:** `200 OK`
```json
{
  "summary": {
    "id": "sum_multi_123",
    "documentIds": ["doc_456def", "doc_789ghi"],
    "query": "How do these documents discuss AI safety?",
    "content": "Across your documents, AI safety is approached from...",
    "citations": [
      {
        "documentId": "doc_456def",
        "excerpt": "Safety considerations include...",
        "page": 15
      }
    ],
    "length": "long",
    "tokenUsage": {
      "prompt": 5680,
      "completion": 420,
      "total": 6100
    },
    "createdAt": "2024-01-15T11:25:00Z"
  }
}
```

---

## User Profile

### GET /api/user

Get current user profile and preferences.

**Auth Required:** Yes

**Response:** `200 OK`
```json
{
  "user": {
    "id": "usr_123abc",
    "email": "user@example.com",
    "username": "johndoe",
    "preferences": {
      "defaultDocumentScope": "all",
      "summaryLength": "medium",
      "streamingEnabled": true
    },
    "memory": {
      "facts": [
        "User is researching neural networks for thesis",
        "Interested in AI safety and ethics"
      ]
    },
    "usage": {
      "documentsCount": 45,
      "conversationsCount": 8,
      "totalTokensUsed": 145230
    },
    "createdAt": "2024-01-10T08:00:00Z"
  }
}
```

### PUT /api/user

Update user profile and preferences.

**Auth Required:** Yes

**Request Body:**
```json
{
  "username": "newusername",
  "preferences": {
    "defaultDocumentScope": "recent",
    "summaryLength": "short"
  }
}
```

**Response:** `200 OK`
```json
{
  "user": {
    "id": "usr_123abc",
    "username": "newusername",
    "preferences": {
      "defaultDocumentScope": "recent",
      "summaryLength": "short",
      "streamingEnabled": true
    },
    "updatedAt": "2024-01-15T11:30:00Z"
  }
}
```

---

## Configuration

### GET /api/config

Get system configuration and AI provider information.

**Auth Required:** No (public endpoint)

**Response:** `200 OK`
```json
{
  "config": {
    "embeddingModel": "text-embedding-3-small",
    "embeddingDimension": 1536,
    "chatModel": "gpt-4-turbo",
    "vectorStore": "pgvector",
    "chunkSize": 1000,
    "chunkOverlap": 200,
    "version": "1.0.0"
  }
}
```

**Purpose:**
- Allows frontends to display provider information
- Helps with debugging and transparency
- Enables users to understand what models are being used
- Useful for comparing different implementations

**Note:** Implementations MUST expose this information. It reveals which providers are being used, making architectural decisions visible.

---

## Health Check

### GET /api/health

Check health status of the service and its dependencies.

**Auth Required:** No (public endpoint)

**Response (Healthy):** `200 OK`
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "timestamp": "2024-01-15T12:00:00Z",
  "checks": {
    "database": "healthy",
    "embedder": "healthy",
    "vectorStore": "healthy",
    "llm": "healthy"
  }
}
```

**Response (Unhealthy):** `503 Service Unavailable`
```json
{
  "status": "unhealthy",
  "version": "1.0.0",
  "timestamp": "2024-01-15T12:00:00Z",
  "checks": {
    "database": "healthy",
    "embedder": "unhealthy",
    "vectorStore": "healthy",
    "llm": "degraded"
  },
  "errors": {
    "embedder": "Connection timeout to embedding API"
  }
}
```

**Check Status Values:**
- `healthy` - Service is functioning normally
- `degraded` - Service is functioning but with reduced performance
- `unhealthy` - Service is not functioning

**Requirements:**
- Implementations SHOULD check actual connectivity to dependencies
- Implementations SHOULD NOT just check configuration
- Health checks SHOULD complete in < 5 seconds
- Health checks MAY cache results for 30-60 seconds

**Purpose:**
- Enables monitoring and alerting
- Helps diagnose issues
- Required for production deployments
- Reveals architectural dependencies

---

## Error Handling

All errors follow a consistent format:

**Error Response Structure:**
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "Additional context if applicable"
    }
  }
}
```

**Common Error Codes:**

| Status | Code | Description |
|--------|------|-------------|
| 400 | `INVALID_REQUEST` | Malformed request body or parameters |
| 401 | `UNAUTHORIZED` | Missing or invalid authentication token |
| 403 | `FORBIDDEN` | User doesn't have access to resource |
| 404 | `NOT_FOUND` | Resource doesn't exist |
| 409 | `CONFLICT` | Resource conflict (e.g., duplicate) |
| 413 | `PAYLOAD_TOO_LARGE` | File or request too large |
| 422 | `VALIDATION_ERROR` | Request validation failed |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |
| 503 | `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

**Example Error:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Document title is required",
    "details": {
      "field": "title",
      "validation": "required"
    }
  }
}
```

---

## Rate Limiting

All endpoints are subject to rate limiting. Rate limit information is returned in response headers:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705320000
```

When rate limit is exceeded, the API returns `429 Too Many Requests`:

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 60 seconds.",
    "details": {
      "retryAfter": 60,
      "limit": 1000,
      "window": "1h"
    }
  }
}
```

**Suggested Rate Limits** (implementations may vary):
- Authenticated requests: 1000 requests per hour per user
- Document uploads: 100 per hour per user
- Message sends: 500 per hour per user
- Search requests: 200 per hour per user

---

## Implementation Notes

**For Backend Implementers:**

1. **Chunking Strategy** - How documents are split into chunks is implementation-specific, but should support citation at chunk level.

2. **Embedding Model** - Choice of embedding model is flexible, but dimensions should be consistent within an implementation.

3. **LLM Provider** - Any LLM (OpenAI, Anthropic, Gemini, local models) that can handle context and generate responses.

4. **Vector Database** - Any vector store that supports similarity search (Pinecone, Weaviate, pgvector, Chroma, etc.).

5. **Streaming** - SSE (Server-Sent Events) is recommended for broader compatibility, but WebSockets are acceptable.

6. **Authentication** - JWT is recommended but any secure token-based auth is acceptable.

7. **Storage** - Document files can be stored in any blob storage (S3, local filesystem, etc.).

**For Frontend Implementers:**

1. **Handle Streaming** - Must support SSE or WebSocket streaming for real-time responses.

2. **Citations UI** - Must display citations with clickable links to source documents.

3. **Loading States** - Show appropriate loading/processing states for async operations.

4. **Error Handling** - Display user-friendly error messages.

5. **Responsive Design** - Work on desktop and mobile devices.

---

**Next:** See [data-models.md](./data-models.md) for detailed schema definitions.
