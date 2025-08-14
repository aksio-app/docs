# ADR-002: Choice of Go HTTP Router Package

> **Technical Note**: Performance metrics and specifications in this document are rough estimates based on industry benchmarks and official documentation as of August 2025. Numbers are order-of-magnitude approximations meant to illustrate relative performance differences, not precise measurements. Actual performance varies significantly based on implementation, hardware, and workload.

## Status
- 2025-08-13 - Draft
- 2025-08-14 - **Accepted**

## Context
Following the decision to use Go for our backend (ADR-001), we need to select an HTTP router package for building our REST API. The package will handle routing, middleware, request/response processing, and provide the foundation for our API endpoints.

Key requirements for our HTTP router package:
- **Performance**: Must handle thousands of concurrent requests with low latency
- **Developer Experience**: Clean API, good documentation, active community
- **Middleware Ecosystem**: Authentication, CORS, rate limiting, logging support
- **Stability**: Production-ready with proven track record
- **Testing Support**: Easy to write unit and integration tests
- **JSON Handling**: Efficient serialization/deserialization
- **Error Handling**: Consistent error management patterns
- **WebSocket Support**: For real-time features (future requirement)

## Decision
We have chosen **Gin** as our HTTP router package for the Aksio backend.

### Rationale
After evaluating the options, Gin provides the best balance of:
1. **Maturity and Stability**: Largest community (50k+ GitHub stars) with extensive production usage
2. **Developer Productivity**: Rich middleware ecosystem and built-in conveniences (validation, binding)
3. **Performance**: Excellent performance characteristics suitable for our educational platform needs
4. **Documentation**: Comprehensive documentation with abundant tutorials and examples
5. **Team Familiarity**: Most widely known Go HTTP router package, easier to find developers with experience
6. **Ecosystem**: Extensive collection of middleware and integrations already available

While Echo offers a cleaner API and Chi is more idiomatic, Gin's massive ecosystem and community support make it the pragmatic choice for rapid development while maintaining production quality.

## Options Considered

### Option 1: Gin (Selected)
**Description**: Currently the most popular Go HTTP router package with extensive middleware ecosystem.

**Pros:**
- Largest community and ecosystem (50k+ GitHub stars)
- Extensive middleware collection available
- Battle-tested in production by many companies
- Excellent performance (40k+ requests/second)
- Familiar API design for developers coming from other languages
- Built-in validation and binding
- Comprehensive documentation and tutorials
- GroupRouting for organizing endpoints

**Cons:**
- Larger dependency footprint than minimalist packages
- Some consider the API design less "Go-idiomatic"
- Context handling can be confusing for beginners
- Slightly higher memory usage than bare-bones alternatives

### Option 2: Echo
**Description**: High-performance, minimalist HTTP router package with elegant API design.

**Pros:**
- Cleaner, more intuitive API than Gin
- Excellent performance (similar to Gin)
- Better documentation quality
- Built-in middleware for common needs
- Automatic TLS via Let's Encrypt
- HTTP/2 support out of the box
- Better WebSocket support
- More consistent context handling

**Cons:**
- Smaller community than Gin (28k+ GitHub stars)
- Fewer third-party middleware options
- Less battle-tested at massive scale
- Some API changes between major versions

### Option 3: Fiber
**Description**: Express-inspired HTTP package built on Fasthttp instead of net/http.

**Pros:**
- Fastest performance (up to 40% faster than Gin/Echo)
- Express-like API familiar to Node.js developers
- Built-in support for WebSockets
- Lower memory footprint
- Good for developers transitioning from Node.js
- Rich set of built-in middleware

**Cons:**
- Built on Fasthttp, not standard net/http (compatibility issues)
- Smallest community of major packages
- Less mature ecosystem
- Fasthttp has different semantics that can cause subtle bugs
- Not compatible with standard net/http middleware
- Potential issues with HTTP/2

### Option 4: Chi
**Description**: Lightweight, idiomatic HTTP router that stays close to standard library.

**Pros:**
- Most "Go-idiomatic" design
- Compatible with any standard net/http middleware
- Very lightweight (minimal dependencies)
- Excellent for APIs that need fine control
- Context-based request handling
- Stable API with no breaking changes

**Cons:**
- More verbose than Gin/Echo/Fiber
- Fewer built-in conveniences
- Requires more boilerplate code
- Smaller ecosystem of Chi-specific middleware
- Less structured than full-featured packages

### Option 5: Standard Library (net/http + gorilla/mux or httprouter)
**Description**: Use Go's standard library with a minimal router.

**Pros:**
- No package lock-in
- Smallest possible dependency footprint
- Maximum control and flexibility
- Best for learning and understanding
- Most stable (part of Go standard library)
- Easy to migrate to any router package later

**Cons:**
- Most verbose, requires significant boilerplate
- No built-in conveniences (validation, binding, etc.)
- Need to build own middleware chain
- Slower development initially
- Team needs to establish patterns

### Option 6: Buffalo
**Description**: Full-stack web development package like Ruby on Rails for Go.

**Pros:**
- Batteries-included approach
- Code generation for rapid development
- Built-in ORM, migrations, and asset pipeline
- Good for full-stack applications
- Integrated testing tools

**Cons:**
- Heavyweight for API-only services
- Opinionated structure may not fit our needs
- Smaller community than micro-packages
- Steeper learning curve
- More complex deployment

## Comparison Matrix

| Package   | Performance | Community | Learning Curve | Ecosystem | Production Ready |
|-----------|------------|-----------|----------------|-----------|-----------------|
| Gin       | Excellent  | Largest   | Moderate       | Extensive | Yes            |
| Echo      | Excellent  | Large     | Easy           | Good      | Yes            |
| Fiber     | Best       | Growing   | Easy           | Growing   | Yes            |
| Chi       | Excellent  | Moderate  | Easy           | Moderate  | Yes            |
| net/http  | Excellent  | N/A       | Hard           | Minimal   | Yes            |
| Buffalo   | Good       | Small     | Steep          | Integrated| Yes            |

## Consequences

### With Gin as our chosen HTTP router package:

**Positive Consequences:**
- **Rapid Development**: Extensive middleware and built-in features accelerate feature delivery
- **Large Talent Pool**: Easier to find Go developers familiar with Gin
- **Proven Patterns**: Well-established patterns for authentication, validation, and error handling
- **Community Support**: Large community means quick answers to problems and continuous improvements
- **Production Ready**: Battle-tested in production by companies at scale
- **Learning Resources**: Abundant tutorials, examples, and documentation
- **Integration Ready**: Existing middleware for JWT, OAuth, CORS, rate limiting, etc.

**Negative Consequences:**
- **Package Lock-in**: While migration is possible, we'll be coupled to Gin's patterns
- **Memory Overhead**: Slightly higher memory usage compared to minimalist alternatives
- **Learning Curve**: Developers need to learn Gin-specific patterns and conventions
- **Opinionated Structure**: Must follow Gin's way of doing things, less flexibility than bare net/http

**Mitigation Strategies:**
- Keep business logic separate from HTTP handlers to minimize package coupling
- Use interfaces to abstract Gin-specific code where practical
- Document Gin-specific patterns for team onboarding
- Regular evaluation of package health and community activity

## Implementation Considerations

### Immediate Actions:
1. **Update go.mod**: Add Gin as a dependency to the backend project
2. **Documentation**: Create team guidelines for Gin best practices
3. **Testing setup**: Configure testing utilities for Gin handlers

### Architecture Guidelines:
- Keep HTTP concerns in handler layer only
- Business logic should remain package-agnostic in service layer
- Use Gin's built-in validation but duplicate critical validations in service layer
- Leverage Gin's middleware chain for cross-cutting concerns
- Use route groups to organize endpoints logically

## Risks and Mitigation
| Risk | Impact | Probability | Mitigation Strategy |
|------|--------|-------------|-------------------|
| Package becomes unmaintained | High | Low | Choose package with large community |
| Performance bottlenecks | Medium | Low | Load test early, package change possible |
| Difficult to hire developers | Medium | Medium | Choose popular package with good docs |
| Missing critical feature | Medium | Low | Evaluate feature completeness upfront |

## Dependencies
- Go 1.24 or later (from ADR-001)
- Will influence choice of testing packages
- May affect deployment and monitoring strategies

## Related Decisions
- ADR-001: Choice of Go for Backend Development

## Notes

### Decision Factors:
The choice of Gin balances developer productivity with performance needs. Our educational platform will handle real-time study scheduling and potentially thousands of concurrent students, making Gin's performance characteristics and scalability proven track record crucial.

### Implementation Status:
Gin is already partially integrated into the backend codebase, with authentication endpoints successfully implemented using Gin's router and middleware. This existing implementation validates the package choice and provides a foundation for expanding to other endpoints.

### Future Considerations:
While we're committing to Gin for the foreseeable future, the architecture should remain flexible enough to migrate if needed. The Go ecosystem's interface-based design makes such migrations feasible, though not trivial.

## Approval
**Decision Maker(s):** Olav Selnes Lorentzen  
**Date Approved:** 2025-08-14