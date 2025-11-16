# Solution: End-to-End Request Flow

## High-Level Architecture

```
┌─────────────┐
│   User      │
│  Browser    │
└──────┬──────┘
       │
       ├─────── 1. DNS Resolution
       │
       ▼
┌─────────────┐
│     CDN     │ ◄─── 2. Check for cached static content
│ (Cloudflare)│
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Load     │ ◄─── 3. Distribute traffic
│  Balancer   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│     API     │ ◄─── 4. Authentication, rate limiting, routing
│   Gateway   │
└──────┬──────┘
       │
       ├──────┬──────┬──────┐
       ▼      ▼      ▼      ▼
    ┌────┐ ┌────┐ ┌────┐ ┌────┐
    │S #1│ │S #2│ │S #3│ │... │ ◄─── 5. Application servers (stateless)
    └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘
       │      │      │      │
       └──────┴──────┴──────┘
              │
              ▼
       ┌─────────────┐
       │    Redis    │ ◄─── 6. Check cache
       │    Cache    │
       └──────┬──────┘
              │
              ▼ (cache miss)
       ┌─────────────┐
       │  Database   │ ◄─── 7. Persistent storage
       │  (Primary)  │
       └─────────────┘
```

## Detailed Request Flow

### Step 1: DNS Resolution

**User action**: Types `www.twitter.com` in browser

**What happens**:
1. Browser checks local DNS cache
2. If not cached, queries DNS resolver (ISP or 8.8.8.8)
3. DNS resolver queries authoritative name servers
4. Returns IP address: `104.244.42.1`
5. Browser caches result (TTL: typically 5 minutes - 1 hour)

**Time**: ~20-50ms (or ~0ms if cached)

### Step 2: CDN Layer

**Request arrives at**: Nearest CDN edge server (e.g., New York data center)

**CDN decision logic**:
```
IF request is for static content (images, CSS, JS):
    IF content is cached AND not expired:
        RETURN from CDN cache
    ELSE:
        FETCH from origin server
        CACHE the content
        RETURN to user
ELSE (dynamic content):
    FORWARD to origin server (load balancer)
```

**What gets cached:**
- Profile pictures
- Tweet images/videos
- JavaScript bundles
- CSS files
- Static HTML

**Time**: ~5-20ms (if cached), ~100-200ms (if fetching from origin)

### Step 3: Load Balancer

**Request arrives at**: Primary load balancer

**Load balancer responsibilities**:

1. **Health checks**: Only route to healthy servers
```
Server #42: ✓ Healthy (response time: 50ms)
Server #43: ✓ Healthy (response time: 45ms)
Server #44: ✗ Unhealthy (timeout) - SKIP
```

2. **Distribution algorithm**:
   - **Round-robin**: Each request goes to next server in sequence
   - **Least connections**: Route to server with fewest active connections
   - **IP hash**: Same client IP always goes to same server (if needed)

3. **SSL termination**: Decrypt HTTPS traffic here (CPU-intensive)

**Decision**: Routes request to Server #42

**Time**: ~1-5ms

### Step 4: API Gateway

**Request arrives at**: API Gateway (single entry point for all services)

**Gateway operations**:

1. **Authentication**:
```python
# Pseudocode
token = extract_token_from_header(request)
user = validate_token(token)  # Check with auth service or Redis
if not user:
    return 401 Unauthorized
```

2. **Rate limiting**:
```python
# Check request count for this user
key = f"rate_limit:{user.id}:{current_minute}"
count = redis.incr(key)
redis.expire(key, 60)  # Expire after 60 seconds

if count > 100:  # Max 100 requests per minute
    return 429 Too Many Requests
```

3. **Routing to microservice**:
```
/api/tweets/timeline    → Timeline Service
/api/tweets/create      → Tweet Service
/api/users/profile      → User Service
/api/notifications      → Notification Service
```

**Time**: ~5-10ms

### Step 5: Application Server

**Request arrives at**: Server #42 (one of 100+ servers)

**Example request**: `GET /api/tweets/timeline`

**Application logic**:

```python
def get_timeline(user_id):
    # 1. Authorization check
    if not user.is_authenticated():
        return 401

    # 2. Build cache key
    cache_key = f"timeline:{user_id}"

    # 3. Check cache first
    cached_data = redis.get(cache_key)
    if cached_data:
        return cached_data  # Cache hit!

    # 4. Cache miss - query database
    timeline = fetch_timeline_from_db(user_id)

    # 5. Store in cache for next time
    redis.setex(cache_key, 60, timeline)  # TTL: 60 seconds

    # 6. Return response
    return timeline
```

**Time**: ~10-50ms (application logic)

### Step 6: Cache Layer (Redis)

**Cache check**: Look for `timeline:user_123`

**Scenario A - Cache Hit** (90% of requests):
```
Redis: "Found! Returning cached timeline"
Time: ~1-2ms
```

**Scenario B - Cache Miss** (10% of requests):
```
Redis: "Not found, need to query database"
Time: Proceed to database
```

**Cache structure**:
```
Key: "timeline:user_123"
Value: JSON array of tweet IDs
TTL: 60 seconds

Key: "tweet:456789"
Value: JSON object with tweet data
TTL: 5 minutes

Key: "session:abc123"
Value: User session data
TTL: 1 hour
```

### Step 7: Database Query

**If cache miss**, query the database:

```sql
-- Get tweets from users that user_123 follows
SELECT t.*
FROM tweets t
JOIN follows f ON t.user_id = f.following_id
WHERE f.follower_id = 123
ORDER BY t.created_at DESC
LIMIT 50
```

**Database architecture**:
- **Primary database**: Handles writes
- **Read replicas**: Handle reads (scaled horizontally)
- **Sharding**: Data split across multiple databases by user_id

**Time**: ~20-100ms (database query)

---

## Response Journey (Return Path)

### Step 1: Store in Cache

Before returning, cache the result:
```python
redis.setex(f"timeline:{user_id}", 60, timeline_data)
```

**Why cache on the way back:**
- Next request for same data will be instant
- Reduces database load
- Improves response time for subsequent requests

### Step 2: Format Response

Application server formats the data:
```json
{
  "tweets": [
    {
      "id": 789,
      "user": "@alice",
      "text": "Hello world!",
      "created_at": "2025-11-16T10:30:00Z"
    },
    // ... more tweets
  ],
  "next_cursor": "abc123"
}
```

### Step 3: Return Through Layers

```
Database → Application Server → API Gateway → Load Balancer → CDN → User
```

Each layer:
- **Application server**: Formats response
- **API Gateway**: Adds headers, logging
- **Load Balancer**: Routes back to user connection
- **CDN**: May cache if response is cacheable
- **Browser**: Receives and renders

**Total time**: ~100-300ms (end-to-end)

---

## Component Interactions Diagram

```
┌──────────────────────────────────────────────────────┐
│                    REQUEST FLOW                       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  User Browser                                         │
│       │                                               │
│       ├─► DNS: www.twitter.com → 104.244.42.1        │
│       │                                               │
│       ├─► CDN: Check cache (static content)          │
│       │        ├─► HIT: Return immediately            │
│       │        └─► MISS: Forward to origin            │
│       │                                               │
│       ├─► Load Balancer: Distribute to Server #42    │
│       │                                               │
│       ├─► API Gateway:                                │
│       │        ├─► Authenticate (check token)         │
│       │        ├─► Rate limit (100 req/min)           │
│       │        └─► Route to Timeline Service          │
│       │                                               │
│       ├─► Application Server #42:                     │
│       │        ├─► Parse request                      │
│       │        ├─► Check authorization                │
│       │        └─► Execute business logic             │
│       │                                               │
│       ├─► Redis Cache:                                │
│       │        ├─► HIT (90%): Return cached data      │
│       │        └─► MISS (10%): Query database         │
│       │                                               │
│       └─► Database (if cache miss):                   │
│              └─► Execute SQL query                    │
│                                                       │
├──────────────────────────────────────────────────────┤
│                   RESPONSE FLOW                       │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Database                                             │
│       │                                               │
│       ├─► Application Server:                         │
│       │        ├─► Format response                    │
│       │        └─► Store in cache (for next time!)    │
│       │                                               │
│       ├─► API Gateway: Add headers, log               │
│       │                                               │
│       ├─► Load Balancer: Route back                   │
│       │                                               │
│       ├─► CDN: Cache if static                        │
│       │                                               │
│       └─► User Browser: Render page                   │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## Scalability Considerations

### Horizontal Scaling (Adding More Servers)

**Easy to scale** (stateless design):
```
Initially: 100 servers handling 1M requests/sec
           = 10,000 requests/sec per server

Peak time: Add 50 more servers → 150 total
           = 6,666 requests/sec per server
```

**Why stateless enables this:**
- New servers can immediately handle any request
- No need to migrate sessions
- Load balancer instantly recognizes new servers

### Vertical Scaling (Bigger Servers)

**Limits:**
- Single server max capacity: ~50,000 requests/sec
- Expensive hardware
- Single point of failure
- **Conclusion**: Horizontal scaling is preferred

### Database Scaling Strategies

1. **Read Replicas**:
```
1 Primary (writes) + 9 Replicas (reads)
= 10x read capacity
```

2. **Sharding** (split by user_id):
```
Users 0-10M    → Database Shard 1
Users 10M-20M  → Database Shard 2
Users 20M-30M  → Database Shard 3
```

3. **Caching**:
- Cache hit rate: 90%
- Reduces database load by 10x

---

## Reliability & Fault Tolerance

### What Happens When Components Fail?

| Component | Failure Impact | Mitigation |
|-----------|---------------|------------|
| **Single Server** | No impact | Load balancer routes to other servers |
| **Load Balancer** | Major outage | Multiple load balancers (active-passive) |
| **Redis Cache** | Slower responses | Traffic goes to database (degraded but functional) |
| **Database Replica** | No impact | Route reads to other replicas |
| **Database Primary** | Can't write | Promote replica to primary (automatic failover) |
| **CDN** | Slower for static content | Traffic goes to origin servers |
| **Entire Data Center** | Regional outage | Route traffic to other geographic regions |

### Health Checks

Load balancer continuously monitors servers:
```python
# Every 5 seconds
for server in servers:
    response = ping(server)
    if response.time > 1000ms or response.failed:
        mark_unhealthy(server)
        stop_routing_to(server)
    else:
        mark_healthy(server)
```

---

## Performance Optimizations

### Latency Breakdown

```
DNS Resolution:          20ms   (cached: ~0ms)
CDN Check:               10ms   (hit: 5ms)
Load Balancer:            5ms
API Gateway:             10ms
Application Logic:       20ms
Cache Lookup:             2ms   (miss: continue to DB)
Database Query:          50ms   (if cache miss)
Network Round-trip:      20ms
─────────────────────────────
Total (cache hit):      ~87ms
Total (cache miss):    ~137ms
```

### Optimization Strategies

1. **Caching** (biggest impact):
   - 90% cache hit rate
   - Saves 50ms per request
   - Reduces database load by 10x

2. **CDN for static content**:
   - Serves from nearby location
   - Reduces latency by ~100ms globally

3. **Connection pooling**:
   - Reuse database connections
   - Saves ~10ms per request

4. **Async processing**:
   - Non-critical tasks (analytics, notifications) handled asynchronously
   - User doesn't wait for these

---

## Capacity Estimation

### Traffic Estimation

**Assumptions:**
- 500 million active users
- Each user makes 20 requests per day
- Peak traffic is 3x average

**Calculations:**
```
Daily requests:  500M users × 20 requests = 10 billion requests/day
Requests/second: 10B / 86,400 seconds ≈ 115,000 requests/sec (average)
Peak requests:   115K × 3 = 345,000 requests/sec
```

### Server Capacity

**Assumptions:**
- Each server handles 10,000 requests/sec

**Calculations:**
```
Servers needed (average): 115,000 / 10,000 = 12 servers
Servers needed (peak):    345,000 / 10,000 = 35 servers
Actual deployment:        50 servers (with headroom for failures)
```

### Cache Memory

**Assumptions:**
- Cache 10M most active users' timelines
- Each timeline: 50 tweets × 200 bytes = 10 KB
- 90% cache hit rate desired

**Calculations:**
```
Cache size: 10M users × 10 KB = 100 GB
Redis servers: 100 GB / 25 GB per server = 4 servers
With replication: 4 × 2 = 8 Redis servers
```

### Database Storage

**Assumptions:**
- 500M users
- Each user posts 1 tweet per day
- Each tweet: 500 bytes

**Calculations:**
```
Daily new data:  500M tweets × 500 bytes = 250 GB/day
Yearly data:     250 GB × 365 = 91 TB/year
5-year storage:  91 TB × 5 = 455 TB
```

---

## Bottlenecks and Failure Modes

### Potential Bottlenecks

1. **Database**:
   - Single primary can't handle all writes
   - **Solution**: Sharding, write-through caching

2. **Network bandwidth**:
   - Transferring images/videos
   - **Solution**: CDN, compression

3. **Load balancer**:
   - Single load balancer is SPOF
   - **Solution**: Multiple load balancers with failover

4. **Cache capacity**:
   - Can't cache everything
   - **Solution**: LRU eviction, tiered caching

### Common Failure Scenarios

**Scenario 1: Cache failure (Redis crash)**
```
Impact:  All requests hit database
Result:  Response time increases from 87ms → 137ms
         Database load increases 10x
Action:  Database read replicas handle the load
         Restart Redis, warm up cache
```

**Scenario 2: Database replica failure**
```
Impact:  Remaining replicas handle traffic
Result:  Each replica gets more load
Action:  Automatic - load balancer routes around failed replica
         Alert ops team to replace replica
```

**Scenario 3: Traffic spike (10x normal)**
```
Impact:  Servers overloaded
Result:  Response times increase, some requests fail
Action:  Auto-scaling adds more servers
         Rate limiting prevents total collapse
         Queue less critical requests
```

---

## Security Considerations

### Defense in Depth

1. **CDN layer**:
   - DDoS protection
   - Web Application Firewall (WAF)

2. **Load balancer**:
   - SSL/TLS termination
   - Rate limiting by IP

3. **API Gateway**:
   - Authentication (JWT validation)
   - Rate limiting by user
   - Input validation

4. **Application server**:
   - Authorization checks
   - Sanitize inputs (prevent SQL injection, XSS)
   - Parameterized queries

5. **Database**:
   - Least privilege access
   - Encrypted at rest
   - Network isolation

### Common Attack Mitigations

| Attack | Mitigation |
|--------|-----------|
| **DDoS** | CDN, rate limiting, auto-scaling |
| **SQL Injection** | Parameterized queries, input validation |
| **XSS** | Output encoding, Content Security Policy |
| **Credential stuffing** | Rate limiting, CAPTCHA, account lockout |
| **Man-in-the-middle** | SSL/TLS everywhere |

---

## Summary

### Key Design Decisions

1. **Stateless architecture**: Enables easy scaling and fault tolerance
2. **Multi-layered caching**: CDN + Redis dramatically improves performance
3. **Horizontal scaling**: Add more servers rather than bigger servers
4. **Microservices**: Independent services scale separately
5. **Multiple data centers**: Geographic distribution for low latency globally

### Request Flow Summary

```
Request:  User → DNS → CDN → LB → API Gateway → Server → Cache → DB
Response: DB → Server (cache it!) → API Gateway → LB → CDN → User

Time: ~87ms (cache hit), ~137ms (cache miss)
```

### Why This Design Scales

- **Stateless servers**: Add/remove instantly
- **Caching**: 90% of requests never hit database
- **CDN**: Static content served from edge
- **Load balancing**: Traffic distributed evenly
- **Fault tolerance**: No single point of failure
- **Geographic distribution**: Serve users globally with low latency
