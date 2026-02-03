# ADR 0002: Language Choices

## Status

Accepted

## Date

2025-06-22

## Context

The Sensei platform requires three distinct runtime profiles:

1. **API gateway and orchestration layer.** Handles HTTP requests, manages migration lifecycle, coordinates agent communication, and serves the dashboard. Requires rapid development velocity, strong ecosystem for web services, and good concurrency support for I/O-bound workloads.

2. **Verification engine (kora-core).** Performs behavioral equivalence testing by replaying historical queries against source and target databases, comparing result sets, and generating cryptographic verification certificates. This is CPU-bound, memory-sensitive, and must operate with deterministic performance characteristics under high throughput.

3. **ML inference services.** Runs the LLM-powered semantic pattern recognition models that power MABA's schema analysis, as well as the differential privacy mechanisms for cross-migration learning. Requires access to the Python ML ecosystem (PyTorch, HuggingFace, NumPy) and GPU acceleration.

A single language cannot optimally serve all three profiles. The team evaluated the following options per component.

## Decision

### API Layer: TypeScript (Node.js)

TypeScript is selected for the API gateway, orchestration layer, CLI, and client-facing SDKs.

**Rationale:**

- Mature ecosystem for HTTP services (Fastify, tRPC, Prisma).
- Strong type system reduces integration errors across the API surface.
- Shared language between API, CLI, and TypeScript SDK reduces context switching.
- Async/await model is well-suited to the I/O-bound orchestration workload.
- Large hiring pool relative to alternatives.

**Alternatives considered:** Go (strong concurrency, but weaker ecosystem for rapid API iteration), Rust (excessive development overhead for an I/O-bound service), Python (inadequate type safety for a complex API surface).

### Verification Engine: Rust (kora-core)

Rust is selected for the verification engine.

**Rationale:**

- Deterministic performance with no garbage collection pauses. Verification operations must complete within bounded time to maintain SLA guarantees.
- Memory safety without runtime overhead. The verification engine processes large result sets and must not leak memory under sustained load.
- Strong concurrency primitives (Tokio, Rayon) for parallel query replay across multiple database connections.
- Correct-by-construction guarantees from the type system and borrow checker reduce the risk of subtle data corruption in verification logic.

**Alternatives considered:** Go (GC pauses unacceptable for latency-sensitive verification), C++ (memory safety risk too high for a correctness-critical component), Java (GC tuning overhead, cold start latency for containerized deployments).

### ML Services: Python

Python is selected for all machine learning inference services.

**Rationale:**

- The ML ecosystem (PyTorch, HuggingFace Transformers, scikit-learn, NumPy) is Python-native. No other language provides equivalent library access.
- GPU acceleration via CUDA is best supported through Python bindings.
- Differential privacy libraries (OpenDP, PySyft) are Python-first.
- Model serving frameworks (vLLM, Triton Inference Server) provide Python interfaces.

**Alternatives considered:** Rust (emerging ML ecosystem but insufficient library coverage for LLM inference), Julia (strong numerical computing but limited production deployment tooling).

## Consequences

**Benefits:**

- Each component uses the language best suited to its performance and ecosystem requirements.
- Clear boundaries between components enforce interface discipline via gRPC and protocol buffers.
- Hiring can target language-specific expertise for each component.

**Drawbacks:**

- Three language toolchains increase CI/CD complexity. Mitigated by the monorepo structure (ADR-0001) and shared build scripts.
- Engineers must be proficient in at least two of the three languages to work across components. Mitigated by clear API boundaries that allow most work to occur within a single language context.
- Cross-language debugging requires familiarity with multiple profiling and tracing tools. Mitigated by unified observability via OpenTelemetry.

**Inter-component communication:** All cross-language communication uses gRPC with protocol buffer definitions maintained in `packages/shared/proto/`. This provides type-safe interfaces, efficient serialization, and language-agnostic contract definitions.

## References

- [Rust Performance Characteristics](https://doc.rust-lang.org/book/)
- [TypeScript Design Goals](https://github.com/microsoft/TypeScript/wiki/TypeScript-Design-Goals)
- ADR-0001: Monorepo Structure
