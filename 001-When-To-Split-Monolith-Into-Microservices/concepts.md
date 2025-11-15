# Key Concepts

## Monolithic Architecture

### What is a Monolith?
A monolithic application is a single unified codebase where all components run together in the same process. When you deploy, you deploy everything as one unit.

**Example structure:**
```
Monolith App
├── User Authentication
├── Product Catalog
├── Order Processing
├── Payment Handling
├── Notifications
└── All share same database
```

### Advantages of Monoliths
1. **Simpler development**: Everything in one codebase, easy to trace and debug
2. **Easier deployment**: Deploy one thing, not multiple services
3. **No network overhead**: Internal function calls (nanoseconds) vs HTTP requests (milliseconds)
4. **Simpler testing**: Test entire flow without setting up multiple services
5. **Single database**: No distributed data consistency issues
6. **Better for small teams**: Less operational overhead

### When Monoliths Work Best
- Small to medium teams (< 20 developers)
- Applications with similar scaling characteristics
- Unified tech stack is sufficient
- Coordinated deployments are acceptable

## Microservices Architecture

### What are Microservices?
Independent services where each handles one specific business capability, runs separately, and communicates over the network.

**Example structure:**
```
Authentication Service ──┐
Product Service ────────┼── Communicate via HTTP/REST
Order Service ──────────┼── or Message Queues
Payment Service ────────┤
Notification Service ────┘
(Each has own database)
```

### Advantages of Microservices
1. **Independent scaling**: Scale only what needs it (e.g., 10 image processing servers, 2 auth servers)
2. **Team independence**: Different teams can deploy without coordinating
3. **Technology flexibility**: Use Python for ML, Go for high-concurrency, Java for legacy systems
4. **Fault isolation**: Bug in one service doesn't crash entire app
5. **Independent deployment**: Ship features on different schedules

### Costs of Microservices
1. **Network latency**: Function call (nanoseconds) → HTTP request (milliseconds = 1000x slower)
2. **Operational complexity**: Multiple deployments, monitoring, debugging
3. **Data consistency**: Keeping data in sync across services is hard
4. **API coordination**: Version management, backward compatibility
5. **Distributed debugging**: "Which service failed?" is harder to answer
6. **Infrastructure overhead**: More servers, load balancers, service discovery

## Independent Scaling

### The Core Benefit
Different parts of your application have different resource needs.

**Example - E-commerce app:**
- Image processing: CPU-intensive, needs beefy servers
- Authentication: Light load, needs small servers
- Checkout: Spiky traffic (Black Friday), needs elastic scaling

**Monolith approach:**
- Must scale everything together
- Cost: 10 beefy servers × $500/month = $5,000/month

**Microservice approach:**
- Image service: 5 CPU-optimized servers @ $300 = $1,500
- Auth service: 2 general servers @ $200 = $400
- Checkout: 3 servers (scale to 20 during holidays) @ $200 = $600
- Total: $2,500/month (50% savings)

## API Versioning and Contracts

### The Challenge
When Service A changes its API, Service B (owned by different team) might break.

**In a monolith:**
```
// Change function signature
getProduct(productId) → getProduct(productId, includeReviews)
// IDE shows all callers, update them all, deploy together
```

**In microservices:**
```
// Service A changes API
GET /api/products/{id} → GET /api/v2/products/{id}?include=reviews
// Service B, C, D still calling old version - they're separate codebases!
```

### Solutions

**1. API Versioning**
```
GET /api/v1/products/{id}  ← Old services use this
GET /api/v2/products/{id}  ← New version
```
Maintain both until everyone migrates.

**2. Backward Compatibility**
Add fields, don't remove or rename:
```
// Old response
{"id": 1, "name": "Laptop", "price": 999}

// New response - additive
{"id": 1, "name": "Laptop", "price": 999, "reviews": [...]}
```

**3. Deprecation Timeline**
- Announce: "v1 deprecated, removed in 6 months"
- Teams migrate gradually
- Remove old version after deadline

**4. Contract-First Design**
Define APIs upfront using OpenAPI, gRPC schemas, or GraphQL. Changes go through review.

## Team Scaling and Conway's Law

### Conway's Law
"Organizations design systems that mirror their communication structure"

### Impact on Architecture
- **Small team (8 developers)**: Can coordinate easily, monolith works fine
- **Large team (50+ developers)**: Communication overhead grows, need clear boundaries
- **Multiple teams**: Each team wants independence, microservices enable autonomy

### Team Boundaries
Microservices often split along team lines:
- Team A owns Authentication Service
- Team B owns Product Service
- Team C owns Order Service

Each team can deploy independently without blocking others.

## The YAGNI Principle

### "You Aren't Gonna Need It"

Don't build for future scale you haven't proven you need.

**Anti-pattern**: "We'll get big someday, let's start with microservices"
**Better approach**: "We have 8 developers and 1000 requests/day - start simple"

### Evidence-Based Decisions
Use metrics to decide when to split:
- Database CPU at 90% (which queries?)
- Deployment conflicts (how often?)
- Server costs (what's the waste?)
- Team conflicts (merge conflicts per week?)

**Real companies**:
- Instagram: Served millions with a monolith before splitting
- Amazon: Started as monolith, split as they scaled
- Netflix: Migrated to microservices over years, not overnight

## Strategic Extraction

### Don't Split Everything at Once

Start with monolith → Extract the most painful service → Monitor → Extract next if needed

**Example progression:**
1. **Year 1**: Monolith with 8 developers, 10,000 orders/day
2. **Year 2**: Extract image processing (CPU bottleneck), keep rest together
3. **Year 3**: Extract order service (deployment frequency), keep auth + catalog together
4. **Year 4**: Further splits as team grows to 100 developers

### Criteria for First Extraction
Look for the service that has:
- Most different scaling needs
- Most different technology requirements
- Clearest boundaries
- Highest pain point (performance, deployment conflicts)

## Trade-off Summary

| Factor | Monolith | Microservices |
|--------|----------|---------------|
| **Development Speed** | Fast (one codebase) | Slower (coordination) |
| **Deployment** | Simple (one unit) | Complex (many services) |
| **Scaling** | Scale everything together | Scale independently |
| **Team Size** | Better for small teams | Better for large teams |
| **Debugging** | Easy (single codebase) | Hard (distributed tracing) |
| **Technology** | Single stack | Multiple stacks |
| **Network** | Function calls | HTTP requests |
| **Data Consistency** | Easy (one database) | Hard (distributed data) |
| **Operational Cost** | Low | High |

## When to Split: Decision Framework

### Clear Signals to Split
1. ✅ **Performance bottleneck**: One feature is starving others (GPS tracking hitting DB hard)
2. ✅ **Scaling inefficiency**: Paying for servers you don't need (image processing needs 10x servers)
3. ✅ **Team conflicts**: Developers blocking each other's deployments
4. ✅ **Technology mismatch**: Need Python for ML but main app is Java
5. ✅ **Different failure tolerance**: Non-critical features can crash critical ones

### Bad Reasons to Split
1. ❌ "Microservices are trendy"
2. ❌ "Might need to scale someday"
3. ❌ "Code is logically separate" (that's just good module design)
4. ❌ "Read it in a blog post"

### The Question to Ask
**"What specific problem are we solving by splitting, and is the complexity worth it?"**

If you can't answer with concrete metrics or pain points, you're not ready.
