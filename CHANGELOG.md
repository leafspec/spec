# Changelog

All notable changes to the LEAF Specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0-draft] - 2026-01-17

### Changed (2026-01-17)

#### User Model Simplification

- **Replaced `username` with optional `displayName`** - The required unique username field has been replaced with an optional `displayName` field that defaults to the user's email if not provided. This simplifies authentication flows while still allowing implementations to show friendly names in the UI.

Affected specs:
- `api-specification.md` - Registration, login, and user profile endpoints
- `data-models.md` - User model definition
- `required-features.md` - Registration validation rules
- `test-suite.md` - Auth test cases
- `ui-requirements.md` - User display references

#### API Specification - Framework Agnosticism Improvements

- **Added "Processing Status" section** - Explicitly documents acceptable approaches for async status notification:
  - Polling (check document status endpoint)
  - Webhooks (callback URL)
  - Real-time subscriptions (WebSocket, SSE, or framework-native like Convex)

- **Clarified streaming flexibility** - Streaming requirement is now "token-by-token delivery with inline citation metadata" rather than mandating SSE specifically. WebSockets and framework-native mechanisms are equally acceptable.

- **Added file upload flexibility** - Documented that implementations may use any upload pattern:
  - Direct multipart upload
  - Presigned URL upload
  - Framework-specific patterns (Convex, Firebase, etc.)

- **Updated frontend streaming requirement** - Changed from "must support SSE or WebSocket" to "must support the implementation's streaming mechanism"

These changes improve framework agnosticism, particularly for reactive frameworks like Convex where subscriptions are more natural than polling.

#### Optional Reasoning Support (Thought Accordion)

- **Added optional reasoning SSE events** to API specification:
  - `reasoning_start` - Reasoning/thinking phase started
  - `reasoning_delta` - Incremental reasoning chunk
  - `reasoning_end` - Reasoning complete, response follows

- **Added optional `reasoning` field** to Message data model for storing AI reasoning/thinking process separately from response content

- Enables "Thought Accordion" UI pattern where reasoning can be shown collapsed/hidden

- Explicitly marked as optional — support depends on underlying LLM capabilities (OpenAI o1, Claude extended thinking, etc.)

---

## [1.0.0-draft] - 2026-01-11

### Initial Draft Release - LEAF Specification

**Name:** LEAF Specification (Listen, Embed, Augment, Flow)
**Goal:** Position LEAF as the CRUD paradigm for AI-native applications
**Status:** Draft specification complete, reference implementation in progress

### Added

#### Core Specifications
- **API Specification** - Complete REST API contract with 15 endpoints
  - Document upload/management (POST /api/documents)
  - Conversation/message endpoints (POST /api/conversations/{id}/messages)
  - Semantic search (GET /api/search)
  - Streaming support via Server-Sent Events
  - Configuration endpoint (GET /api/config)
  - Health check endpoint (GET /api/health)

- **Data Models** - 8 core entities with JSON schemas
  - User, Document, Conversation, Message
  - Citation, SearchResult, Summary, TokenUsage
  - Standardized pagination format

- **Design Principles** - 7 normative architectural requirements
  1. AI is Essential, Not Decorative
  2. Explicit Workflows Over Implicit Magic
  3. Asynchronous by Default
  4. Provenance and Citations Required
  5. Non-Determinism is Explicit
  6. No Framework-Specific Shortcuts
  7. Strict Scope Discipline

- **Test Suite** - Comprehensive validation criteria
  - API endpoint compliance tests
  - Three "Turing Tests" for edge cases:
    - 6.1: Hallucination Guard Test
    - 6.2: Context Squeeze Test
    - 6.3: Multimodal Ingestion Test (optional extension)
  - Implementation grading rubric (7 dimensions, 70 points)

- **UI Requirements** - Framework-agnostic frontend specifications
  - Required screens and user flows
  - Thought Accordion pattern for AI reasoning transparency
  - Citation display and source inspector

- **Provider Interfaces** - Recommended abstraction patterns (guidance only)
  - Embedder interface (text → vectors)
  - VectorStore interface (similarity search)
  - LLM interface (streaming completion)
  - DocumentProcessor interface (file parsing)

#### Key Concepts
- **LEAF Paradigm** - Listen, Embed, Augment, Flow (vs CRUD)
- **202 Accepted Pattern** - Async document processing
- **Citation Provenance** - Track AI responses to sources
- **Scope Tiers** - Core (text-only) vs Extension (multimodal)

### Framework Agnosticism
- ✅ External API contract specified
- ✅ Internal implementation not prescribed
- ✅ Provider interfaces as guidance, not requirements
- ✅ Multiple tech stack options supported

### Documentation Structure
```
/specs/
  api-specification.md       (21KB)
  data-models.md            (14KB)
  design-principles.md       (9KB)
  test-suite.md             (13KB)
  ui-requirements.md        (14KB)
  provider-interfaces.md    (15KB)
```

---

## [Unreleased] - Planned for v1.0.0

### Coming Soon
- [ ] Reference implementation (Next.js + Convex)
- [ ] Automated test suite
- [ ] Implementation guide
- [ ] Live demo deployment

### Future Enhancements (v1.1.0+)
- [ ] GraphQL alternative to REST API
- [ ] Real-time collaboration features
- [ ] Advanced multimodal support (audio, video)
- [ ] Multi-brain management

---

## Version History

- **v1.0.0-draft** (2026-01-11) - Initial specification draft
- **v1.0.0** (planned Mid February 2026) - Official release with reference implementation

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes to this specification.
