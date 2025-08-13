# ADR-002: Choice of Go HTTP Router Package

> **Technical Note**: Performance metrics and specifications in this document are rough estimates based on industry benchmarks and official documentation as of August 2025. Numbers are order-of-magnitude approximations meant to illustrate relative performance differences, not precise measurements. Actual performance varies significantly based on implementation, hardware, and workload.

## Status
**DRAFT** - Under consideration

## Date
2025-08-13

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
**TO BE DETERMINED** - Awaiting evaluation of options

## Options Considered

### Option 1: Gin
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
- Less structured than full frameworks

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

### If choosing a full-featured package (Gin/Echo/Fiber):
**Positive:**
- Faster initial development
- Built-in solutions for common problems
- Established patterns and best practices
- Easier onboarding for new developers

**Negative:**
- Package lock-in
- Potential overhead for simple endpoints
- Need to learn package-specific patterns

### If choosing minimal approach (Chi/net/http):
**Positive:**
- Maximum flexibility and control
- Easier to understand what's happening
- No package lock-in
- Lighter dependency footprint

**Negative:**
- More boilerplate code
- Slower initial development
- Team needs to establish patterns
- Fewer conveniences out of the box

## Implementation Considerations
- Migration path between frameworks (most are relatively easy except Fiber)
- Team familiarity and learning curve
- Long-term maintenance and framework longevity
- Compatibility with our chosen deployment platform (GCP Cloud Run)

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
The package choice should balance developer productivity with performance needs. Since our platform will handle real-time study scheduling and potentially thousands of concurrent students, performance is important but not at the cost of maintainability.

Consider starting with a well-established package (Gin or Echo) and optimizing later if needed, as migration between packages in Go is generally straightforward due to the language's interface-based design.

## Approval
**Decision Maker(s):** [TO BE DETERMINED]  
**Date Approved:** [PENDING]