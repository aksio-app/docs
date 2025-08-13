# ADR-001: Choice of Go for Backend Development

> **Technical Note**: Performance metrics and specifications in this document are rough estimates based on industry benchmarks and official documentation as of August 2025. Numbers like "10-100x faster" are order-of-magnitude approximations meant to illustrate relative performance differences, not precise measurements. Actual performance varies significantly based on implementation, hardware, and workload. For current benchmarks, consult official documentation.

## Status
Accepted

## Date
2025-08-13

## Context
The Aksio platform requires a robust, performant backend service to handle educational data processing, user authentication, and API services. The backend must scale efficiently, handle high concurrency, and maintain low latency for real-time features.

Key requirements include:
- **Performance**: Sub-100ms API response times for real-time learning features
- **Concurrency**: Handle thousands of simultaneous users per instance
- **Scalability**: Horizontal scaling capability for growing user base
- **Developer Productivity**: Fast compilation and testing cycles
- **Operational Excellence**: Simple deployment and monitoring
- **Type Safety**: Prevent runtime errors in production
- **Security**: Robust handling of student data (FERPA/GDPR compliance)

## Decision
We will use Go (Golang) as the primary programming language for the Aksio platform backend services.

## Rationale
Go provides several advantages that make it ideal for a high-performance educational platform:

### Performance & Scalability
1. **Compiled Language**: Native binaries with near-C performance, crucial for low-latency APIs
2. **Efficient Memory Management**: Garbage collection optimized for low pause times (~1ms)
3. **Small Memory Footprint**: Typically 10-50MB per service vs 100-500MB for JVM-based services
4. **Fast Startup Times**: Services start in milliseconds, enabling rapid scaling and deployment

### Concurrency Excellence
5. **Goroutines**: Lightweight threads (2KB stack) allow millions of concurrent operations
6. **Channels**: Built-in CSP model for safe concurrent communication
7. **No Thread Management**: Runtime handles scheduling, eliminating complex thread pool tuning

### Developer Productivity
8. **Fast Compilation**: Full rebuilds in seconds, not minutes (10-100x faster than Java/C++)
9. **Simple Language**: Only 25 keywords, learned in days not months
10. **Built-in Testing**: Testing, benchmarking, and profiling in standard library
11. **Single Binary Deployment**: No dependency hell, just copy and run

### Operational Benefits
12. **Cross-compilation**: Build for any OS/architecture from any machine
13. **Static Linking**: No runtime dependencies to manage in production
14. **Excellent Observability**: Built-in profiling (CPU, memory, goroutines, blocking)

### Code Quality
15. **Type Safety**: Catches errors at compile time, not in production
16. **Explicit Error Handling**: No hidden exceptions, clear error paths
17. **Standard Formatting**: `gofmt` eliminates style debates
18. **Security**: Memory-safe with bounds checking, no buffer overflows

## Options Considered

### Option 1: Go (Selected)
**Pros:**
- **Performance**: 10-100x faster than interpreted languages, comparable to C/C++
- **Concurrency**: Goroutines handle 100,000+ concurrent connections per instance
- **Build Speed**: 5-second builds vs minutes for other compiled languages
- **Memory Efficiency**: 10MB services vs 100MB+ for JVM
- **Deployment**: Single binary, no runtime dependencies
- **Learning Curve**: Productive in 1-2 weeks vs months for complex frameworks

**Cons:**
- Less mature ecosystem for some specialized libraries
- No traditional inheritance (composition-only can be unfamiliar)
- Verbose error handling (though explicit is often better)

### Option 2: Node.js/TypeScript
**Pros:**
- Large ecosystem and community
- Good for rapid development
- TypeScript provides type safety
- Familiar to many developers
- Excellent for API development

**Cons:**
- Single-threaded nature less ideal for CPU-intensive tasks
- Runtime errors more common than compiled languages
- Dependency management complexity
- Less predictable performance characteristics

### Option 3: Python/Django
**Pros:**
- Excellent AI/ML ecosystem (TensorFlow, PyTorch, scikit-learn)
- Rapid development with batteries-included framework
- Large talent pool and community
- Extensive libraries for educational/scientific computing
- Great for data processing and analysis

**Cons:**
- **Performance**: 10-100x slower than Go for API operations
- **Interpreted**: Runtime errors only discovered in production
- **GIL Limitation**: True parallelism requires multiple processes
- **Memory Usage**: 100-500MB per process vs 10-50MB for Go
- **Deployment Complexity**: Managing Python versions, virtualenvs, dependencies
- **Type Safety**: Optional typing doesn't prevent runtime errors

### Option 4: Java/Spring Boot
**Pros:**
- Mature ecosystem and extensive libraries
- Strong enterprise support
- Excellent tooling and IDE support
- Well-established security frameworks
- Large talent pool

**Cons:**
- Higher memory usage and startup times
- More complex deployment and configuration
- Verbose syntax can obscure security-critical logic
- JVM overhead for lightweight services

## Consequences

### Positive
- **10x Performance Gain**: Sub-10ms p99 latency achievable for most API endpoints
- **Cost Reduction**: 5-10x fewer servers needed vs JVM or interpreted languages
- **Developer Velocity**: 10-second test cycles, 30-second deploy cycles
- **Operational Simplicity**: Single binary deployment, no runtime version management
- **Scaling Confidence**: Proven to handle millions of concurrent users (Docker, Kubernetes, Uber)
- **Fast Onboarding**: New developers productive within first week

### Negative
- **Library Gaps**: May need to build custom integrations for niche educational tools
- **Talent Pool**: Smaller than JavaScript/Python, though growing rapidly
- **ORM Limitations**: Less magic, more explicit SQL (can be positive for performance)

### Neutral
- **Different Paradigm**: Composition over inheritance requires adjustment
- **Explicit Error Handling**: More verbose but catches more bugs
- **Static Binaries**: 10-20MB files vs 1MB scripts (irrelevant with modern infrastructure)

## Technical Implications

### Performance Metrics
- **Latency**: p50 < 5ms, p99 < 50ms achievable for typical API calls
- **Throughput**: 10,000+ RPS per instance on modest hardware
- **Memory**: 10-50MB per service under load
- **CPU**: Efficient use of multi-core systems via goroutines

### Security Benefits
- **Memory Safety**: No buffer overflows or use-after-free vulnerabilities
- **Type Safety**: Compile-time prevention of type confusion attacks
- **Crypto Libraries**: Excellent standard library for TLS, JWT, encryption
- **Explicit Error Handling**: Security issues can't be silently ignored

## Implementation
- **Environment**: Go 1.24+ with module support
- **HTTP Framework**: See ADR-002 for HTTP framework selection
- **Database**: pgx driver for PostgreSQL (3x faster than standard library)
- **Testing**: Table-driven tests with >80% coverage requirement
- **Benchmarking**: Performance benchmarks for critical paths
- **Profiling**: pprof integration for production debugging
- **Monitoring**: Prometheus metrics and OpenTelemetry tracing

## Risks and Mitigation
| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|-------------|-------------------|
| Team learning curve | Medium | Medium | Go tour + 1 week ramp-up, pair programming |
| Library gaps | Low | Medium | Evaluate needs early, budget time for custom code |
| Goroutine leaks | Medium | Low | Use context.Context, implement leak detection |
| GC pause times | Low | Low | Monitor with pprof, tune GOGC if needed |

## Dependencies
- Go 1.24 or later runtime
- Database drivers (PostgreSQL with Apache AGE)
- Security middleware packages
- Testing framework and tools

## Related Decisions
- ADR-002: Go HTTP Framework Selection (pending)

## Notes
Go was chosen primarily for its exceptional performance characteristics, concurrency model, and developer productivity benefits. The language's simplicity (25 keywords) and fast compilation times (5-10 seconds) enable rapid iteration while maintaining production-grade performance. Companies like Google, Uber, Docker, and Kubernetes have proven Go's ability to scale to millions of users while maintaining sub-millisecond latencies.

While Python/Django was seriously considered for its excellent AI/ML ecosystem which could benefit our learning analytics features, the 10-100x performance penalty and lack of true concurrency (due to the GIL) made it unsuitable for our real-time, high-concurrency requirements. We may still use Python for offline ML model training and data analysis tasks, but the core API serving layer needs the performance characteristics that only compiled languages like Go can provide.

The decision aligns with our need for a platform that can grow from hundreds to millions of students while maintaining consistent performance and operational simplicity.

## Approval
**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-13
**Next Review Date:** 2026-02-13
