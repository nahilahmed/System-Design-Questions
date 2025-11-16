# Additional Resources

## Core Concepts

### DNS and Networking
- **How DNS Works**: Understanding domain name resolution
  - [DNS Explained - Cloudflare](https://www.cloudflare.com/learning/dns/what-is-dns/)
  - Covers DNS hierarchy, caching, and record types

- **HTTP Request/Response Cycle**
  - Understanding headers, status codes, and protocols
  - Learn about HTTP/1.1, HTTP/2, and HTTP/3 differences

### Load Balancing
- **Load Balancing Algorithms**
  - Round-robin, least connections, IP hash
  - Consistent hashing for distributed systems

- **Layer 4 vs Layer 7 Load Balancing**
  - TCP vs HTTP load balancing trade-offs
  - When to use each approach

- **Popular Load Balancers**
  - NGINX
  - HAProxy
  - AWS Elastic Load Balancer (ELB)

### Caching Strategies
- **Redis Documentation**
  - In-memory data structures
  - Persistence options
  - Cluster mode vs Sentinel

- **Cache Invalidation**
  - "There are only two hard things in Computer Science: cache invalidation and naming things" - Phil Karlton
  - Strategies: TTL, write-through, cache-aside

- **Cache Eviction Policies**
  - LRU (Least Recently Used)
  - LFU (Least Frequently Used)
  - FIFO (First In First Out)

### Content Delivery Networks
- **How CDNs Work**
  - Edge locations and PoPs (Points of Presence)
  - Origin servers vs edge servers
  - Cache hierarchy

- **Popular CDN Providers**
  - Cloudflare (also provides DDoS protection)
  - AWS CloudFront
  - Akamai
  - Fastly

## Architecture Patterns

### Stateless vs Stateful
- **The Twelve-Factor App - Processes**
  - Best practices for building stateless applications
  - Why stateless is crucial for modern cloud apps

- **Session Management**
  - Token-based authentication (JWT)
  - OAuth 2.0 and OpenID Connect
  - Where to store session data

### Microservices
- **Microservices Architecture**
  - Breaking down monoliths
  - Service boundaries and domain-driven design
  - Inter-service communication patterns

- **API Gateway Pattern**
  - Kong, AWS API Gateway, Azure API Management
  - Rate limiting, authentication, transformation

## Real-World Implementations

### Case Studies
- **Twitter Architecture**
  - How Twitter handles millions of requests per second
  - Timeline generation at scale
  - Caching strategies

- **Netflix Architecture**
  - Edge services and caching
  - Content delivery optimization
  - Chaos engineering (testing failure scenarios)

- **Facebook/Meta Infrastructure**
  - TAO (The Associations and Objects) caching layer
  - Memcached at scale
  - Edge computing

### Engineering Blogs
- **High Scalability Blog**
  - Real-world architecture breakdowns
  - System design case studies

- **Company Engineering Blogs**
  - Netflix Tech Blog
  - Uber Engineering
  - Airbnb Engineering
  - LinkedIn Engineering
  - Dropbox Tech Blog

## Performance and Optimization

### Latency and Performance
- **Understanding Latency**
  - Network latency, application latency, database latency
  - The importance of measuring at percentiles (p50, p95, p99)

- **Performance Monitoring**
  - Application Performance Monitoring (APM) tools
  - Distributed tracing (Jaeger, Zipkin)
  - Metrics collection (Prometheus, Grafana)

### Database Optimization
- **Database Indexing**
  - B-tree indexes
  - When to use indexes
  - Index overhead

- **Query Optimization**
  - EXPLAIN plans
  - N+1 query problem
  - Pagination strategies

- **Database Replication**
  - Primary-replica replication
  - Multi-primary replication
  - Replication lag

## Reliability and Fault Tolerance

### High Availability
- **Designing for Failure**
  - Assume everything will fail
  - Graceful degradation
  - Circuit breaker pattern

- **Health Checks**
  - Active vs passive health checks
  - Heartbeat mechanisms

- **Failover Strategies**
  - Active-passive vs active-active
  - Automatic vs manual failover
  - Database failover considerations

### Observability
- **The Three Pillars**
  - Metrics (time-series data)
  - Logs (events and errors)
  - Traces (request flow through system)

- **Tools**
  - Prometheus + Grafana (metrics)
  - ELK Stack (Elasticsearch, Logstash, Kibana) for logs
  - Jaeger for distributed tracing

## Security

### Authentication and Authorization
- **JWT (JSON Web Tokens)**
  - How they work
  - Stateless authentication
  - Token expiration and refresh

- **OAuth 2.0**
  - Authorization flows
  - Third-party integrations

- **API Security**
  - API keys vs tokens
  - Rate limiting strategies
  - CORS (Cross-Origin Resource Sharing)

### Common Vulnerabilities
- **OWASP Top 10**
  - Injection attacks (SQL, NoSQL, Command)
  - Broken authentication
  - XSS (Cross-Site Scripting)
  - CSRF (Cross-Site Request Forgery)

- **DDoS Protection**
  - Rate limiting
  - CDN-based protection
  - Auto-scaling to absorb traffic

## Capacity Planning

### Back-of-Envelope Calculations
- **Numbers Every Programmer Should Know**
  - L1 cache: 0.5 ns
  - RAM: 100 ns
  - SSD: 150,000 ns
  - Network within data center: 500,000 ns
  - Disk: 10,000,000 ns

- **Estimating System Capacity**
  - QPS (Queries Per Second) calculations
  - Storage requirements
  - Bandwidth requirements
  - Memory requirements for caching

### Scaling Strategies
- **Horizontal vs Vertical Scaling**
  - When to use each
  - Cost implications
  - Limitations of each approach

- **Database Sharding**
  - Sharding strategies (range, hash, geographic)
  - Challenges: joins, transactions across shards
  - Resharding considerations

## Tools and Technologies

### Infrastructure
- **Containerization**
  - Docker for application packaging
  - Container orchestration with Kubernetes

- **Cloud Providers**
  - AWS (EC2, RDS, ElastiCache, CloudFront)
  - Google Cloud Platform
  - Microsoft Azure

- **Infrastructure as Code**
  - Terraform
  - CloudFormation
  - Ansible

### Development Tools
- **Load Testing**
  - Apache JMeter
  - Gatling
  - k6
  - Locust

- **Monitoring and Debugging**
  - New Relic
  - DataDog
  - Sentry (error tracking)

## Books

### System Design
- **"Designing Data-Intensive Applications" by Martin Kleppmann**
  - Comprehensive guide to distributed systems
  - Covers databases, replication, partitioning, consistency

- **"System Design Interview" by Alex Xu**
  - Popular interview preparation book
  - Real-world system design examples

### Performance
- **"High Performance Browser Networking" by Ilya Grigorik**
  - Understanding latency and bandwidth
  - Optimizing web performance

### Distributed Systems
- **"Distributed Systems" by Maarten van Steen**
  - Academic perspective on distributed computing
  - Consistency, replication, fault tolerance

## Related System Design Patterns

### Patterns to Explore Next
1. **Database Design and Sharding**
   - When to shard
   - Sharding strategies
   - Handling distributed transactions

2. **Message Queues and Async Processing**
   - Decoupling services
   - RabbitMQ, Kafka, SQS
   - Event-driven architecture

3. **Rate Limiting and Throttling**
   - Token bucket algorithm
   - Leaky bucket algorithm
   - Distributed rate limiting

4. **Consistent Hashing**
   - Used in load balancers and distributed caches
   - Minimizes data movement when nodes are added/removed

5. **Circuit Breaker Pattern**
   - Preventing cascading failures
   - Graceful degradation

6. **CQRS (Command Query Responsibility Segregation)**
   - Separating read and write operations
   - Optimizing for different access patterns

## Practice Problems

### Related System Design Questions
1. Design a URL shortener (bit.ly)
2. Design Twitter
3. Design Instagram
4. Design a web crawler
5. Design a notification system
6. Design a rate limiter
7. Design a distributed cache
8. Design a search autocomplete system

### Key Concepts to Practice
- Calculating QPS and storage needs
- Identifying bottlenecks
- Choosing appropriate databases
- Designing for high availability
- Making trade-off decisions

## Online Resources

### Interactive Learning
- **System Design Primer (GitHub)**
  - Comprehensive collection of system design topics
  - Includes examples and exercises

- **Gaurav Sen's YouTube Channel**
  - System design interview videos
  - Clear explanations of complex concepts

### Communities
- **r/systemdesign on Reddit**
  - Discussions and resources

- **Stack Overflow**
  - Specific technical questions

- **Hacker News**
  - Industry discussions and architecture posts

---

## Next Steps

After mastering request flow, consider exploring:
1. **Database internals**: How databases actually work under the hood
2. **Distributed consensus**: Raft, Paxos algorithms
3. **Event-driven architecture**: Moving beyond request-response
4. **Serverless architecture**: When to use vs traditional servers
5. **Edge computing**: Processing closer to users
