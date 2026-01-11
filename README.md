# LEAF Specification

**Status:** 🚧 Draft v1.0 - In Progress

> *The framework-agnostic specification for AI-native applications*

![Draft Badge](https://img.shields.io/badge/status-draft%20v1.0-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## What is LEAF?

**LEAF** (Listen, Embed, Augment, Flow) is the fundamental paradigm for AI-native applications.

Just as **CRUD** (Create, Read, Update, Delete) defined the operations for data-driven applications, **LEAF** defines the operations for intelligence-driven applications:

| **CRUD (Data Era)** | **LEAF (AI Era)** |
|---------------------|----------------------|
| **C**reate | **L**isten - Async ingestion & processing |
| **R**ead | **E**mbed - Transform to vector space |
| **U**pdate | **A**ugment - Retrieve & contextualize (RAG) |
| **D**elete | **F**low - Stream responses with provenance |

**LEAF spec** is the reference implementation specification - defining *exactly* what an application supporting these operations should look like.

---

## ⚠️ Early Draft - Feedback Wanted

**This specification is in early draft phase.** We're defining what could become standard terminology for AI-native development, but we need your input to get it right.

**Your feedback is critical:**
- 📝 Challenge the LEAF operations - Are these the right four? Is there a fifth?
- 🤔 Question the AI-Native Knowledge Base benchmark - Right application or too narrow?
- 🔍 Critique the design principles - Too strict? Missing something?
- 💡 Suggest improvements - What patterns should we capture?

**AI moves fast.** If LEAF is going to become industry standard, it needs to be stress-tested **now** while it's still malleable. Don't hesitate to open issues, start discussions, or propose changes.

**We'd rather be wrong in January 2026 than right in January 2027 after everyone's converged on different terminology.**

👉 **[Open a Discussion](https://github.com/leafspec/spec/discussions)** | **[Report an Issue](https://github.com/leafspec/spec/issues)**

---

## Inspiration

Inspired by [RealWorld/Conduit](https://github.com/gothinkster/realworld), which proved that building the same application across frameworks enables objective comparison.

RealWorld tested CRUD on a Medium.com clone. LEAF spec tests LEAF on an AI-Native Knowledge Base application.

---

## The Mission

**Build the same AI-native application in any framework to enable objective comparison.**

LEAF spec defines:
- ✅ **External API contract** (endpoints, JSON schemas, user flows)
- ✅ **Test validation suite** (automated compliance checks)
- ✅ **Design principles** (normative architectural requirements)
- ❌ **NOT internal implementation** (database schema, service architecture, language choice)

### Why This Matters

Today's AI frameworks hide complexity behind "magic" SDKs. LEAF spec reveals:
- How frameworks handle **async document processing** (Listen - background jobs)
- How frameworks integrate **vector databases** (Embed - semantic search)
- How frameworks implement **RAG pipelines** (Augment - retrieval & context)
- How frameworks support **streaming responses** (Flow - real-time LLM output with citations)

---

## The Application: AI-Native Knowledge Base

A personal knowledge management system where users can:

1. **📥 Upload documents** (PDF, Markdown, text files)
2. **💬 Chat with documents** using AI (RAG-powered conversations)
3. **🔍 Semantic search** across their knowledge base
4. **📊 Generate summaries** of documents or conversations
5. **🔗 Track citations** from AI responses to source documents

**This is the reference application for LEAF operations.**

**Demo:** *(coming soon - reference implementation in progress)*

---

## Quick Start

### For Framework Authors

Want to prove your framework can handle AI-native patterns? Build a LEAF-compliant app:

```bash
# 1. Read the specification
git clone https://github.com/leafspec/spec
cd spec

# 2. Review the API contract
cat specs/api-specification.md

# 3. Implement the LEAF operations
# ... build your implementation

# 4. Run the test suite (coming soon)
npm run test-leaf
```

### For Developers

Explore existing implementations to see framework tradeoffs:

- [nextjs-leaf](https://github.com/leafspec/nextjs-leaf) - TypeScript/Next.js *(planned)*
- [python-fastapi-leaf](https://github.com/leafspec/python-fastapi) *(planned)*
- [go-leaf](https://github.com/leafspec/go-leaf) *(planned)*

**Status:** Reference implementation (hopefully) launching Mid February 2026

---

## Specification Structure

### Core Specifications

| Document | Purpose |
|----------|---------|
| [API Specification](specs/api-specification.md) | REST endpoints, JSON schemas, streaming contracts |
| [Data Models](specs/data-models.md) | Entity schemas (User, Document, Conversation, Message, etc.) |
| [Design Principles](specs/design-principles.md) | 7 normative architectural requirements |
| [Test Suite](specs/test-suite.md) | Validation criteria + "Turing Tests" for edge cases |
| [UI Requirements](specs/ui-requirements.md) | Frontend specifications (framework-agnostic) |
| [Provider Interfaces](specs/provider-interfaces.md) | Recommended abstraction patterns (guidance only) |

### The LEAF Operations

Each operation maps to specific architectural patterns:

#### 🎧 **Listen** - Async Ingestion
- 202 Accepted pattern for long-running operations
- Background job processing
- Status polling or WebSocket updates
- **Tests:** Can framework handle heavy compute without blocking?

#### 🧬 **Embed** - Vector Transformation
- Text → embedding vectors
- Vector database integration
- Cosine similarity search
- **Tests:** How verbose is vector database code? Native ORM support?

#### 🔗 **Augment** - RAG Pipeline
- Semantic search for relevant chunks
- Context window management
- Retrieval ranking and filtering
- **Tests:** Can framework orchestrate multi-step AI workflows?

#### 🌊 **Flow** - Streaming Output
- Token-by-token response streaming via SSE
- Citation tracking in real-time
- Progressive UI updates
- **Tests:** Does framework fight against streaming? Buffering issues?

---

## Design Principles

LEAF spec enforces 7 normative principles:

1. ✅ **AI is Essential, Not Decorative** - Core features require LLM/embeddings
2. ✅ **Explicit Workflows Over Implicit Magic** - Show ingestion status, citations, reasoning
3. ✅ **Asynchronous by Default** - Heavy compute as background jobs (202 pattern)
4. ✅ **Provenance and Citations Required** - Track AI responses to source documents
5. ✅ **Non-Determinism is Explicit** - Acknowledge AI variability in UI/UX
6. ✅ **No Framework-Specific Shortcuts** - Only standard web APIs
7. ✅ **Strict Scope Discipline** - Core is text-only, multimodal is optional extension

See [Design Principles](specs/design-principles.md) for detailed requirements.

## Contributing

**This is an early draft - your input shapes the final spec.**

We're currently building the reference implementation, but **specification feedback is welcome right now:**

**Immediate ways to help:**
- 💬 **Challenge assumptions** - Open a [Discussion](https://github.com/leafspec/spec/discussions) about what we got wrong
- 🐛 **Report issues** - Ambiguous requirements, missing edge cases, inconsistencies
- 🔨 **Build early** - Implement in your framework and tell us what breaks
- ⭐ **Star & watch** - Follow progress and join the conversation

**After reference launch (Mid February 2026):**
- Submit implementations in your favorite framework
- Contribute to test suite automation
- Write implementation guides

The best time to influence this specification is **now**, while it's still draft. See [CONTRIBUTING.md](CONTRIBUTING.md) for full details.

---

## The Vision

If LEAF spec succeeds, **LEAF** becomes the standard terminology for AI-native operations - just as CRUD became standard for data operations.

**Imagine a world where:**
- Job postings say "Experience building LEAF applications"
- Framework docs advertise "Full LEAF support"
- Developers say "It's just a LEAF app" the way they say "It's just CRUD"

**That's the goal.** Define the paradigm, then watch it propagate.

---

## Credits

- **LEAF Paradigm:** Inspired by industry patterns, formalized by this specification
- **Specification Model:** Inspired by [RealWorld/Conduit](https://github.com/gothinkster/realworld) by Eric Simons and Thinkster

**RealWorld's Mission (2017):**
> "While most 'todo' demos provide an excellent cursory glance at a framework's capabilities, they typically don't convey the knowledge & perspective required to actually build real applications with it."

**LEAF spec's Mission (2026):**
> "While most AI demos show how to call an LLM, they typically don't convey the architectural patterns required to build production-ready AI-native applications."

---

## License

MIT License - See [LICENSE](LICENSE) for details

---

## Status & Timeline

**Current Phase:** Draft Specification + Reference Implementation
**Next Milestone:** Reference implementation complete (Mid February 2026)
**Official Launch:** Mid February 2026 with working demo

**Follow progress:**
- Specification: This repository (updated continuously)
- Reference Implementation: [nextjs-leaf](https://github.com/leafspec/nextjs-leaf) *(coming soon)*
- Discussion: GitHub Discussions

---

**Built to define a paradigm. CRUD for the AI era.**
