# Changelog

All notable changes to the LEAF Specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
- [ ] Reference implementation (Next.js + Vercel AI SDK + Supabase)
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
