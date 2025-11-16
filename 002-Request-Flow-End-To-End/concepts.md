# Key Concepts

## 1. DNS Resolution (Domain Name System)

**What it is:**
DNS is like a phonebook for the internet that translates human-readable domain names (like `www.twitter.com`) into IP addresses (like `104.244.42.1`) that computers use to communicate.

**Why we need it:**
- Domain names are easier to remember than IP addresses
- IP addresses can change without affecting users
- Allows for flexibility in infrastructure changes (moving data centers, changing hosting)

**How it works:**
1. Browser checks local cache
2. If not found, queries DNS resolver
3. DNS resolver queries authoritative DNS servers
4. Returns IP address to browser
5. Browser caches the result

## 2. Load Balancing

**What it is:**
A load balancer distributes incoming requests across multiple servers to ensure no single server becomes overwhelmed.

**Why we need it:**
- Single servers have limited capacity
- Distributes traffic evenly across many servers
- Provides fault tolerance (if one server fails, others continue)
- Enables horizontal scaling

**Key concepts:**
- **Distribution algorithms**: Round-robin, least connections, IP hash
- **Health checks**: Monitors server health and stops routing to failed servers
- **Horizontal scaling**: Adding more servers to handle increased load

**Example calculation:**
- 1 million requests/second ÷ 100 servers = 10,000 requests/second per server

## 3. Stateful vs Stateless Architecture

### Stateful Architecture (Sticky Sessions)

**How it works:**
- Each user is assigned to a specific server
- Server stores session data in its local memory
- Load balancer uses cookies to route same user to same server

**Advantages:**
- Simpler implementation
- Faster access to session data (in local memory)

**Disadvantages:**
- Server crashes = lost sessions for all users on that server
- Difficult to scale (can't easily add/remove servers)
- Uneven load distribution possible
- Server maintenance requires session migration

**When to use:**
- WebSocket connections (chat apps, real-time collaboration)
- Legacy applications that can't be refactored
- Performance-critical applications with heavy session caching

### Stateless Architecture (Recommended for Scale)

**How it works:**
- Requests can go to any server
- Session data stored externally (Redis, database)
- Each server fetches session data as needed

**Advantages:**
- Server crashes don't lose user sessions
- Easy to add/remove servers instantly
- Better load distribution
- Simplified maintenance and deployments

**Disadvantages:**
- Extra network call to fetch session data
- Slightly more complex architecture

**Why stateless is better for large-scale systems:**
- Twitter has millions of concurrent users
- Servers need to be added/removed dynamically based on load
- Must handle server failures gracefully
- Even load distribution is critical

## 4. Caching

**What it is:**
Caching stores frequently accessed data in fast memory (RAM) to avoid slow database queries.

**Storage hierarchy (fast to slow):**
1. **CPU Cache**: Fastest, smallest
2. **RAM (Memory)**: Fast, medium size - **This is where Redis/Memcached live**
3. **SSD (Disk)**: Medium speed, large size
4. **HDD (Disk)**: Slowest, largest - **This is where databases typically store data**

**Cache vs Database:**
- **Cache**: Fast (RAM), temporary, limited size
- **Database**: Slow (disk), permanent, large capacity

**TTL (Time To Live):**
Different data has different freshness requirements:

- **Short TTL (30-60 seconds)**: Timeline feeds, trending topics, notifications
- **Medium TTL (5-15 minutes)**: User profiles, follower counts
- **Long TTL (hours/days)**: Profile pictures, static assets, logos

**Cache strategies:**
1. **Read-through**: Check cache first, if miss then query database and store in cache
2. **Write-through**: Write to both cache and database simultaneously
3. **Cache invalidation**: Remove stale data when underlying data changes

**Trade-offs:**
- Longer TTL = less database load, but more stale data
- Shorter TTL = fresher data, but more database queries

**Popular caching tools:**
- Redis: In-memory data store, supports complex data structures
- Memcached: Simple key-value cache

## 5. Authentication vs Authorization

### Authentication (Who are you?)

**Purpose**: Verify user identity

**Examples:**
- Username/password login
- API key validation
- JWT token verification
- OAuth tokens

**Answer**: "You are user @nahil"

### Authorization (What can you do?)

**Purpose**: Verify user permissions for specific actions

**Examples:**
- Can this user delete this tweet? (only if they wrote it)
- Can this user access admin panel? (only if they're an admin)
- Can this user view this private profile? (only if they're a follower)

**Answer**: "User @nahil is allowed to perform this action"

**Why separate:**
- Authentication happens once per session
- Authorization happens for every action
- Different users have different permissions
- Same user identity, different access levels

## 6. Content Delivery Network (CDN)

**What it is:**
Geographically distributed servers that cache static content close to users.

**Why we need it:**
- **Reduced latency**: Serving from nearby location is faster
- **Reduced origin load**: Origin servers don't serve every static file
- **Better user experience**: Faster page loads globally

**What to cache on CDN:**
- Images, videos, audio files
- CSS, JavaScript files
- Fonts
- Static HTML pages

**What NOT to cache on CDN:**
- User-specific data (timeline, notifications)
- Real-time data
- Authenticated content
- Dynamic API responses

**Popular CDNs:**
- Cloudflare
- AWS CloudFront
- Akamai
- Fastly

## 7. API Gateway

**What it is:**
A single entry point that sits between clients and microservices.

**Responsibilities:**
1. **Routing**: Direct requests to appropriate microservices
2. **Authentication**: Validate API keys/tokens
3. **Rate limiting**: Prevent abuse (e.g., max 100 requests/minute per user)
4. **Request transformation**: Convert formats, add headers
5. **Logging & monitoring**: Track all requests

**Why rate limiting is important:**
- Prevents abuse and DDoS attacks
- Ensures fair resource allocation
- Protects backend services from overload
- Enforces API usage limits for different tiers

**Without rate limiting:**
- Malicious users could overwhelm the system
- Bugs could cause infinite loops making unlimited requests
- Uneven resource consumption (some users monopolize resources)

## 8. Data Flow Patterns

### Request Flow
```
User → DNS → CDN → Load Balancer → API Gateway → Web Server → Cache → Database
```

### Response Flow
```
Database → Web Server → (Store in Cache) → API Gateway → Load Balancer → CDN → User
```

**Key insight**: On the way back, we populate the cache for future requests!

## 9. Microservices Architecture

**What it is:**
Breaking down a large application into smaller, independent services.

**Example for Twitter:**
- **Tweet Service**: Handles creating, reading, deleting tweets
- **User Service**: Manages user profiles, authentication
- **Timeline Service**: Generates user timelines
- **Notification Service**: Sends notifications
- **Media Service**: Handles image/video uploads

**Benefits:**
- Each service can scale independently
- Different teams can work on different services
- Technology stack can vary per service
- Easier to maintain and debug

**Trade-offs:**
- More complex infrastructure
- Network calls between services add latency
- Distributed systems challenges (consistency, transactions)

## 10. Fault Tolerance Concepts

**Single Point of Failure (SPOF):**
Any component whose failure would stop the entire system.

**How to avoid:**
- Multiple load balancers
- Multiple database replicas
- Multiple data centers
- Automatic failover mechanisms

**Graceful degradation:**
- If cache fails → still serve from database (slower but works)
- If CDN fails → serve from origin (slower but works)
- If one server fails → others handle the load

## Trade-Off Analysis Summary

| Decision | Pros | Cons |
|----------|------|------|
| **Stateless vs Stateful** | Better scaling, fault tolerance | Extra latency for session lookup |
| **Long vs Short Cache TTL** | Less DB load | More stale data |
| **CDN caching** | Faster for users, less origin load | Stale content possible |
| **More servers** | Better performance, redundancy | Higher cost |
| **Microservices** | Independent scaling, flexibility | Complexity, network overhead |
