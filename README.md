# Additional Resources

## Core Concepts

### Microservices Architecture
- **Martin Fowler - Microservices**: https://martinfowler.com/articles/microservices.html
  - Definitive guide to microservices by one of the thought leaders
  - Covers characteristics, benefits, and costs

- **Sam Newman - Building Microservices**: Book
  - Comprehensive guide to designing, deploying, and managing microservices
  - Practical patterns and anti-patterns

### Monolith First Approach
- **Martin Fowler - Monolith First**: https://martinfowler.com/bliki/MonolithFirst.html
  - Argues for starting with monolith and splitting later
  - Discusses risks of premature microservices

### API Design and Versioning
- **Stripe API Design Guide**: https://stripe.com/docs/api/versioning
  - Real-world example of API versioning done well
  - Backward compatibility strategies

- **REST API Versioning Strategies**:
  - URL versioning (`/v1/products`)
  - Header versioning (`Accept: application/vnd.api.v1+json`)
  - Query parameter versioning (`?version=1`)

## Patterns and Practices

### Strangler Fig Pattern
- **Martin Fowler - Strangler Fig Application**: https://martinfowler.com/bliki/StranglerFigApplication.html
  - Incrementally migrate from monolith to microservices
  - Reduce risk by gradual transition

### Service Communication

**Synchronous Communication:**
- REST APIs
- gRPC (Google's RPC framework)
- GraphQL

**Asynchronous Communication:**
- Message Queues: RabbitMQ, Apache Kafka
- Event-Driven Architecture
- Publish-Subscribe patterns

### Distributed Data Management
- **Saga Pattern**: https://microservices.io/patterns/data/saga.html
  - Manage transactions across multiple services
  - Choreography vs Orchestration

- **Event Sourcing**: https://martinfowler.com/eaaDev/EventSourcing.html
  - Store state as sequence of events
  - Rebuild state by replaying events

### Circuit Breaker Pattern
- **Martin Fowler - Circuit Breaker**: https://martinfowler.com/bliki/CircuitBreaker.html
  - Prevent cascading failures
  - Fail fast when downstream service is unhealthy

## Real-World Case Studies

### Companies That Migrated

**Netflix:**
- Migrated from monolith to microservices over several years
- Now runs hundreds of microservices
- Blog: https://netflixtechblog.com/

**Amazon:**
- Started as monolith
- Split into services as they scaled
- Two-pizza team rule (team size)

**Uber:**
- Started with monolith
- Split as team grew to thousands of engineers
- Engineering blog: https://eng.uber.com/

### Companies That Stayed Monolith

**Shopify:**
- Serves millions of merchants with a modular monolith
- Blog posts on why they chose monolith: https://shopify.engineering/

**Stack Overflow:**
- Runs on monolith architecture
- Handles massive scale with vertical scaling

## Tools and Technologies

### Service Mesh
- **Istio**: Traffic management, security, observability
- **Linkerd**: Lightweight service mesh
- Purpose: Handle service-to-service communication

### API Gateways
- **Kong**: Open-source API gateway
- **AWS API Gateway**: Managed service
- Purpose: Single entry point, rate limiting, authentication

### Monitoring and Observability
- **Prometheus + Grafana**: Metrics collection and visualization
- **Jaeger / Zipkin**: Distributed tracing
- **ELK Stack**: Centralized logging (Elasticsearch, Logstash, Kibana)

### Container Orchestration
- **Kubernetes**: Container orchestration platform
- **Docker Compose**: Local multi-container development
- Purpose: Deploy, scale, and manage containerized services

## Related System Design Patterns

### Database Patterns
- **Database per Service**: Each microservice has own database
- **Shared Database**: Multiple services share database (anti-pattern in microservices)
- **CQRS (Command Query Responsibility Segregation)**: Separate read and write models

### Deployment Patterns
- **Blue-Green Deployment**: Run two identical environments, switch traffic
- **Canary Deployment**: Gradually roll out to subset of users
- **Feature Flags**: Enable/disable features without deployment

### Resilience Patterns
- **Retry with Exponential Backoff**: Retry failed requests with increasing delays
- **Bulkhead Pattern**: Isolate resources to prevent total failure
- **Timeout Pattern**: Set maximum wait time for responses

## Books

1. **"Building Microservices"** by Sam Newman
   - Comprehensive guide to microservices architecture

2. **"Designing Data-Intensive Applications"** by Martin Kleppmann
   - Deep dive into distributed systems, consistency, scalability

3. **"The Phoenix Project"** by Gene Kim
   - Novel about DevOps transformation, includes microservices discussion

4. **"Monolith to Microservices"** by Sam Newman
   - Specifically about migration strategies

## Online Courses

- **Microservices with Node JS and React** (Udemy)
- **Microservices Architecture** (Coursera)
- **AWS Certified Solutions Architect** - Covers microservices on AWS

## Key Takeaways

### Remember These Principles

1. **YAGNI** (You Aren't Gonna Need It): Don't build for hypothetical future
2. **Conway's Law**: Architecture mirrors organization structure
3. **Evidence-based decisions**: Use metrics, not gut feelings
4. **Incremental migration**: Strangler pattern over big bang
5. **API contracts**: Version from day one
6. **Monitor everything**: You can't improve what you don't measure

### Common Pitfalls to Avoid

1. ❌ Starting with microservices for small team/low traffic
2. ❌ Splitting without clear pain points
3. ❌ Ignoring operational complexity
4. ❌ Not planning for service communication
5. ❌ Breaking API contracts without versioning
6. ❌ Shared databases between services
7. ❌ Not investing in monitoring/observability

### Questions to Ask Before Splitting

1. What specific problem are we solving?
2. Do we have metrics showing the pain?
3. Can we solve it within the monolith?
4. Do we have the operational maturity for microservices?
5. What's the migration plan?
6. How will we handle API evolution?
7. What's our rollback strategy?

If you can't answer these confidently, you're not ready to split.
