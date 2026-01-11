---
title: Data Models
version: 1.0.0-draft
last_updated: 2026-01-11
status: Draft
category: Data Schema
---

# Data Models

**Version:** 1.0.0-draft

This document defines the data schemas for all entities in the LEAF Specification. Schemas are defined using JSON Schema format but are **implementation-agnostic** - you can map these to any database, ORM, or data structure.

> **Implementation Note:** These are logical schemas defining the data contract. Internal storage format, database schema, indexing, and optimization are implementation details.

## Table of Contents

- [User](#user)
- [Document](#document)
- [Conversation](#conversation)
- [Message](#message)
- [Citation](#citation)
- [SearchResult](#searchresult)
- [Summary](#summary)
- [TokenUsage](#tokenusage)
- [Pagination](#pagination)

---

## User

Represents a user account in the system.

```json
{
  "id": "string (unique identifier)",
  "email": "string (valid email, unique)",
  "username": "string (3-30 characters, unique)",
  "passwordHash": "string (never exposed in API responses)",
  "preferences": {
    "defaultDocumentScope": "string (enum: all|recent|tagged)",
    "summaryLength": "string (enum: short|medium|long)",
    "streamingEnabled": "boolean"
  },
  "memory": {
    "facts": ["string (learned facts about user)"]
  },
  "usage": {
    "documentsCount": "integer",
    "conversationsCount": "integer",
    "totalTokensUsed": "integer"
  },
  "createdAt": "string (ISO 8601 datetime)",
  "updatedAt": "string (ISO 8601 datetime)"
}
```

**Field Requirements:**
- `id` - Required, unique, immutable
- `email` - Required, unique, valid email format
- `username` - Required, unique, 3-30 characters, alphanumeric + underscore/dash
- `passwordHash` - Required for storage, never returned in API responses
- `preferences` - Optional, defaults to system defaults
- `memory.facts` - Optional, empty array by default
- `usage` - System-calculated, read-only
- `createdAt` - Required, set on creation
- `updatedAt` - Required, updated on modification

**Example:**
```json
{
  "id": "usr_a8f3c92b",
  "email": "alice@example.com",
  "username": "alice",
  "preferences": {
    "defaultDocumentScope": "all",
    "summaryLength": "medium",
    "streamingEnabled": true
  },
  "memory": {
    "facts": [
      "Researching machine learning for thesis",
      "Prefers technical summaries"
    ]
  },
  "usage": {
    "documentsCount": 23,
    "conversationsCount": 5,
    "totalTokensUsed": 45230
  },
  "createdAt": "2024-01-10T08:00:00Z",
  "updatedAt": "2024-01-15T11:30:00Z"
}
```

---

## Document

Represents an uploaded or created document.

```json
{
  "id": "string (unique identifier)",
  "userId": "string (foreign key to User.id)",
  "title": "string (1-200 characters)",
  "content": "string (full text content, optional for binary files)",
  "contentType": "string (MIME type)",
  "size": "integer (bytes)",
  "status": "string (enum: processing|ready|failed)",
  "tags": ["string"],
  "url": "string (URL to file if stored externally)",
  "metadata": {
    "author": "string (optional)",
    "pages": "integer (optional)",
    "language": "string (optional, ISO 639-1)"
  },
  "chunkCount": "integer (number of embedded chunks)",
  "createdAt": "string (ISO 8601 datetime)",
  "updatedAt": "string (ISO 8601 datetime)",
  "processedAt": "string (ISO 8601 datetime, nullable)"
}
```

**Field Requirements:**
- `id` - Required, unique, immutable
- `userId` - Required, must reference valid User
- `title` - Required, 1-200 characters
- `content` - Optional (may not exist for binary files like PDFs)
- `contentType` - Required (e.g., `text/markdown`, `application/pdf`)
- `size` - Required, file size in bytes
- `status` - Required, state machine: `processing` → `ready` or `failed`
- `tags` - Optional, array of strings for categorization
- `url` - Optional, URL if file stored externally
- `metadata` - Optional, extracted metadata from document
- `chunkCount` - Required after processing, null during processing
- `processedAt` - Null until processing complete

**Supported Content Types:**
- `text/plain`
- `text/markdown`
- `application/pdf`
- `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (DOCX)
- Others at implementation discretion

**Example:**
```json
{
  "id": "doc_b7e2f91a",
  "userId": "usr_a8f3c92b",
  "title": "Neural Networks Fundamentals",
  "contentType": "application/pdf",
  "size": 2456789,
  "status": "ready",
  "tags": ["machine-learning", "research", "thesis"],
  "url": "https://storage.example.com/documents/doc_b7e2f91a.pdf",
  "metadata": {
    "author": "Dr. Jane Smith",
    "pages": 45,
    "language": "en"
  },
  "chunkCount": 89,
  "createdAt": "2024-01-12T14:20:00Z",
  "updatedAt": "2024-01-12T14:20:00Z",
  "processedAt": "2024-01-12T14:22:30Z"
}
```

---

## Conversation

Represents a chat conversation thread.

```json
{
  "id": "string (unique identifier)",
  "userId": "string (foreign key to User.id)",
  "title": "string (1-200 characters)",
  "documentIds": ["string (array of Document.id)"],
  "messageCount": "integer",
  "createdAt": "string (ISO 8601 datetime)",
  "updatedAt": "string (ISO 8601 datetime)"
}
```

**Field Requirements:**
- `id` - Required, unique, immutable
- `userId` - Required, must reference valid User
- `title` - Required, 1-200 characters
- `documentIds` - Optional, empty array means search all documents
- `messageCount` - System-calculated, read-only
- `createdAt` - Required
- `updatedAt` - Required, updated when messages added

**Example:**
```json
{
  "id": "conv_9d4c3f2a",
  "userId": "usr_a8f3c92b",
  "title": "Questions about Neural Networks",
  "documentIds": ["doc_b7e2f91a", "doc_c3e1a45b"],
  "messageCount": 8,
  "createdAt": "2024-01-14T09:00:00Z",
  "updatedAt": "2024-01-14T09:45:00Z"
}
```

---

## Message

Represents a single message in a conversation.

```json
{
  "id": "string (unique identifier)",
  "conversationId": "string (foreign key to Conversation.id)",
  "role": "string (enum: user|assistant|system)",
  "content": "string (message text)",
  "citations": ["Citation (optional, only for assistant messages)"],
  "relatedDocuments": ["string (array of Document.id, optional)"],
  "tokenUsage": "TokenUsage (optional, only for assistant messages)",
  "createdAt": "string (ISO 8601 datetime)"
}
```

**Field Requirements:**
- `id` - Required, unique, immutable
- `conversationId` - Required, must reference valid Conversation
- `role` - Required, one of: `user`, `assistant`, `system`
- `content` - Required, message text (markdown supported)
- `citations` - Optional, array of Citation objects (assistant only)
- `relatedDocuments` - Optional, suggested related documents
- `tokenUsage` - Optional, token metrics (assistant only)
- `createdAt` - Required, immutable

**Role Descriptions:**
- `user` - Message from the user
- `assistant` - AI-generated response
- `system` - System messages (e.g., "Conversation started")

**Example (User Message):**
```json
{
  "id": "msg_1a2b3c4d",
  "conversationId": "conv_9d4c3f2a",
  "role": "user",
  "content": "What are the main components of a neural network?",
  "createdAt": "2024-01-14T09:15:00Z"
}
```

**Example (Assistant Message):**
```json
{
  "id": "msg_2b3c4d5e",
  "conversationId": "conv_9d4c3f2a",
  "role": "assistant",
  "content": "Based on your documents, the main components of a neural network are:\n\n1. **Input Layer** - Receives the initial data\n2. **Hidden Layers** - Process information through weighted connections\n3. **Output Layer** - Produces the final prediction\n\nEach layer consists of neurons that apply activation functions to transform the data.",
  "citations": [
    {
      "documentId": "doc_b7e2f91a",
      "documentTitle": "Neural Networks Fundamentals",
      "chunkId": "chunk_42",
      "excerpt": "A neural network consists of an input layer, one or more hidden layers, and an output layer. Each layer contains neurons that apply activation functions...",
      "relevanceScore": 0.94,
      "page": 12
    }
  ],
  "relatedDocuments": ["doc_c3e1a45b"],
  "tokenUsage": {
    "prompt": 1450,
    "completion": 120,
    "total": 1570
  },
  "createdAt": "2024-01-14T09:15:05Z"
}
```

---

## Citation

Represents a reference to source material in a document.

```json
{
  "documentId": "string (foreign key to Document.id)",
  "documentTitle": "string",
  "chunkId": "string (identifier for specific chunk)",
  "excerpt": "string (relevant text excerpt)",
  "relevanceScore": "number (0-1, similarity score)",
  "page": "integer (optional, page number if applicable)",
  "section": "string (optional, section/chapter name)",
  "metadata": {
    "startChar": "integer (optional)",
    "endChar": "integer (optional)"
  }
}
```

**Field Requirements:**
- `documentId` - Required, must reference valid Document
- `documentTitle` - Required for display
- `chunkId` - Required, identifies specific chunk in vector store
- `excerpt` - Required, 50-500 characters of relevant text
- `relevanceScore` - Required, 0.0-1.0 similarity score
- `page` - Optional, page number if document has pages
- `section` - Optional, section/chapter if available
- `metadata` - Optional, additional context

**Example:**
```json
{
  "documentId": "doc_b7e2f91a",
  "documentTitle": "Neural Networks Fundamentals",
  "chunkId": "chunk_42",
  "excerpt": "A neural network consists of an input layer, one or more hidden layers, and an output layer. Each layer contains neurons that apply activation functions to transform the incoming data.",
  "relevanceScore": 0.94,
  "page": 12,
  "section": "Chapter 2: Architecture",
  "metadata": {
    "startChar": 3450,
    "endChar": 3680
  }
}
```

---

## SearchResult

Represents a search result from semantic search.

```json
{
  "documentId": "string (foreign key to Document.id)",
  "documentTitle": "string",
  "chunkId": "string",
  "content": "string (chunk content)",
  "relevanceScore": "number (0-1)",
  "metadata": {
    "page": "integer (optional)",
    "section": "string (optional)"
  }
}
```

**Field Requirements:**
- `documentId` - Required
- `documentTitle` - Required for display
- `chunkId` - Required
- `content` - Required, full chunk text
- `relevanceScore` - Required, 0.0-1.0
- `metadata` - Optional, context information

**Example:**
```json
{
  "documentId": "doc_b7e2f91a",
  "documentTitle": "Neural Networks Fundamentals",
  "chunkId": "chunk_56",
  "content": "Backpropagation is the key algorithm used to train neural networks. It calculates gradients by propagating errors backward through the network, allowing weights to be adjusted to minimize loss.",
  "relevanceScore": 0.89,
  "metadata": {
    "page": 28,
    "section": "Chapter 4: Training"
  }
}
```

---

## Summary

Represents a generated summary of one or more documents.

```json
{
  "id": "string (unique identifier)",
  "userId": "string (foreign key to User.id)",
  "documentIds": ["string (array of Document.id)"],
  "query": "string (optional, focus query)",
  "content": "string (summary text)",
  "length": "string (enum: short|medium|long)",
  "focus": "string (enum: general|key_points|technical|conclusions)",
  "citations": ["Citation (optional)"],
  "tokenUsage": "TokenUsage",
  "createdAt": "string (ISO 8601 datetime)"
}
```

**Field Requirements:**
- `id` - Required, unique
- `userId` - Required
- `documentIds` - Required, array of documents summarized
- `query` - Optional, if summary focused on specific question
- `content` - Required, summary text
- `length` - Required, summary size
- `focus` - Required, summary type
- `citations` - Optional, source references
- `tokenUsage` - Required
- `createdAt` - Required

**Length Guidelines:**
- `short` - ~100-200 words
- `medium` - ~300-500 words
- `long` - ~600-1000 words

**Example:**
```json
{
  "id": "sum_7f8e9d0a",
  "userId": "usr_a8f3c92b",
  "documentIds": ["doc_b7e2f91a"],
  "content": "This document provides a comprehensive overview of neural network fundamentals. Key topics include network architecture (input, hidden, and output layers), activation functions (ReLU, sigmoid, tanh), and training algorithms (backpropagation, gradient descent). The document emphasizes practical applications in image recognition and natural language processing.",
  "length": "short",
  "focus": "key_points",
  "tokenUsage": {
    "prompt": 3200,
    "completion": 95,
    "total": 3295
  },
  "createdAt": "2024-01-14T10:00:00Z"
}
```

---

## TokenUsage

Represents LLM token consumption for an operation.

```json
{
  "prompt": "integer (input tokens)",
  "completion": "integer (output tokens)",
  "total": "integer (prompt + completion)"
}
```

**Field Requirements:**
- `prompt` - Required, non-negative integer
- `completion` - Required, non-negative integer
- `total` - Required, must equal prompt + completion

**Example:**
```json
{
  "prompt": 1450,
  "completion": 120,
  "total": 1570
}
```

---

## Pagination

Standard pagination metadata for list endpoints.

```json
{
  "total": "integer (total items available)",
  "limit": "integer (items per page)",
  "offset": "integer (starting position)",
  "hasMore": "boolean (more items available)"
}
```

**Field Requirements:**
- `total` - Required, total count of all items
- `limit` - Required, max items in current response
- `offset` - Required, starting position (0-indexed)
- `hasMore` - Required, `true` if more pages exist

**Example:**
```json
{
  "total": 156,
  "limit": 20,
  "offset": 40,
  "hasMore": true
}
```

---

## Validation Rules

### String Constraints
- **Email**: Valid email format (RFC 5322)
- **Username**: 3-30 characters, alphanumeric + `_` and `-`
- **Title**: 1-200 characters
- **Content**: 1-1000000 characters (1MB text limit)
- **Tags**: Each tag 1-50 characters, max 20 tags per document

### Numeric Constraints
- **File Size**: Max 50MB per document (implementation may vary)
- **Page Limits**: 1-100 for lists (default 20)
- **Relevance Score**: 0.0-1.0 (floating point)

### Date/Time Format
- All dates in ISO 8601 format: `YYYY-MM-DDTHH:mm:ssZ`
- UTC timezone required

---

## Relationships

```
User 1:N Document
User 1:N Conversation
User 1:N Summary

Conversation 1:N Message
Conversation N:M Document (via documentIds)

Message N:M Citation
Citation N:1 Document
```

---

## Implementation Notes

1. **IDs**: Use any unique identifier format (UUID, nanoid, ULID, auto-increment). Prefix with entity type is recommended (e.g., `doc_`, `usr_`) for debugging.

2. **Chunks**: Document chunking is implementation-specific. Store chunk metadata to support citations.

3. **Embeddings**: Not explicitly modeled - internal to vector store implementation.

4. **Soft Deletes**: Consider soft delete patterns for Documents and Conversations to support undo/recovery.

5. **Indexes**: Implementations should index frequently queried fields (userId, status, createdAt, etc.).

6. **Content Storage**: Binary document files can be stored separately from metadata (blob storage, filesystem, etc.).

---

**Next:** See [required-features.md](./required-features.md) for feature requirements.
