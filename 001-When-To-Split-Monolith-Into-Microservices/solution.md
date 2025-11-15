# Solution: Decision Framework for Splitting Monoliths

## High-Level Approach

**Start with a monolith. Split strategically when you have concrete evidence that the benefits outweigh the costs.**

## Step-by-Step Decision Process

### Step 1: Assess Current State

Before considering microservices, gather data:

**Metrics to collect:**
```
Performance Metrics:
- Database CPU/Memory usage (which queries?)
- API endpoint latency (which endpoints?)
- Resource usage by feature
- Error rates by component

Team Metrics:
- Deployment frequency
- Number of developers
- Merge conflicts per week
- Time spent coordinating deployments

Cost Metrics:
- Current server costs
- Projected costs with independent scaling
- Operational overhead (person-hours)
```

### Step 2: Identify Pain Points

Look for these specific signals:

**Signal 1: Performance Bottleneck**
```
Example:
- GPS tracking: 200 writes/second → 70% of DB CPU
- Menu lookups: Slow due to GPS traffic
- Action: Extract GPS to own service + own database
```

**Signal 2: Scaling Inefficiency**
```
Example:
- Image processing needs CPU-heavy servers
- Rest of app needs moderate servers
- Currently: 10 servers @ $500/mo = $5,000/mo
- With split: $2,100/mo (58% savings)
- Action: Extract image processing
```

**Signal 3: Team Conflicts**
```
Example:
- 30 developers, 5 teams
- Teams blocking each other's releases
- 3+ hour deploy coordination meetings
- Action: Split by team boundaries
```

**Signal 4: Technology Mismatch**
```
Example:
- Need Python + TensorFlow for recommendations
- Main app is Java
- Can't mix in monolith
- Action: Extract recommendations service
```

### Step 3: Choose What to Split

**Prioritization matrix:**

| Service | Performance Issue | Scaling Need | Team Conflict | Tech Mismatch | **Priority** |
|---------|------------------|--------------|---------------|---------------|--------------|
| GPS Tracking | High (70% DB CPU) | Medium | Low | Low | **HIGH** |
| Image Processing | Medium | High (10x servers) | Low | Medium (needs GPU) | **HIGH** |
| Auth | Low | Low | Low | Low | **LOW** |
| Menus | Low | Low | Medium | Low | **LOW** |

**Extract in order of priority. Don't split everything at once.**

### Step 4: Design the Service Boundary

For the service you're extracting:

**1. Define the API Contract**
```
Service: GPS Tracking Service

API:
POST /tracking/update
{
  "driverId": "123",
  "location": {"lat": 37.7749, "lng": -122.4194},
  "timestamp": "2024-01-15T12:00:00Z"
}

GET /tracking/driver/{driverId}
Response:
{
  "driverId": "123",
  "currentLocation": {"lat": 37.7749, "lng": -122.4194},
  "lastUpdate": "2024-01-15T12:00:00Z"
}
```

**2. Decide on Data Ownership**
```
GPS Service owns:
- driver_locations table
- tracking_history table

Order Service owns:
- orders table
- order_status table

Communication: Order Service calls GPS Service API to get driver location
```

**3. Plan Migration Strategy**

**Option A: Strangler Pattern** (Recommended)
```
Phase 1: Create new GPS Service, but don't use it yet
Phase 2: Dual-write: Update both monolith DB and GPS Service
Phase 3: Switch reads to GPS Service
Phase 4: Stop writing to monolith, remove old code
```

**Option B: Big Bang**
```
Deploy new service and switch all at once
Riskier, but faster
```

### Step 5: Handle Cross-Service Communication

**Synchronous (HTTP/REST)**
```
Order Service needs driver location:
GET https://gps-service/tracking/driver/123

Pros: Simple, immediate response
Cons: Coupled, if GPS service is down, order service fails
```

**Asynchronous (Message Queue)**
```
GPS Service publishes location updates:
Topic: "driver.location.updated"
Message: {"driverId": "123", "location": {...}}

Order Service subscribes and updates its cache

Pros: Decoupled, resilient to failures
Cons: Eventual consistency, more complex
```

**When to use each:**
- Synchronous: When you need immediate, up-to-date data
- Asynchronous: When eventual consistency is acceptable

### Step 6: Maintain API Compatibility

**Version your APIs from day one:**

```
Current API:
GET /api/v1/products/{id}
Response: {"id": 1, "price": 999}

Need to add currency support:
GET /api/v2/products/{id}
Response: {"id": 1, "price": {"amount": 999, "currency": "USD"}}

Migration plan:
1. Release v2 API
2. Announce: "v1 deprecated, removal in 6 months"
3. Consumers migrate gradually
4. Remove v1 after deadline
```

**Backward compatibility rules:**
- ✅ Add new fields: Safe
- ✅ Add new optional parameters: Safe
- ❌ Remove fields: Breaking change
- ❌ Rename fields: Breaking change
- ❌ Change field types: Breaking change

## Architecture Evolution Example

### Food Delivery App Evolution

**Phase 1: Year 1 (Monolith)**
```
┌─────────────────────────────────┐
│         Monolith App            │
│  ┌──────────────────────────┐   │
│  │ Restaurants              │   │
│  │ Orders                   │   │
│  │ GPS Tracking             │   │
│  │ Payments                 │   │
│  └──────────────────────────┘   │
│              ↓                  │
│  ┌──────────────────────────┐   │
│  │     Single Database      │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘

Team: 8 developers
Traffic: 1,000 orders/day
Cost: $400/month (2 servers)
```

**Phase 2: Year 2 (First Extraction)**
```
Problem: GPS tracking writing 200x/sec, database at 90% CPU

┌──────────────────────┐       ┌──────────────────────┐
│   Monolith App       │       │  GPS Tracking        │
│  ┌────────────────┐  │       │  Service             │
│  │ Restaurants    │  │       │  ┌────────────────┐  │
│  │ Orders         │──┼──HTTP─┼─→│ Track Updates  │  │
│  │ Payments       │  │       │  │ Get Location   │  │
│  └────────────────┘  │       │  └────────────────┘  │
│         ↓            │       │         ↓            │
│  ┌────────────────┐  │       │  ┌────────────────┐  │
│  │   Main DB      │  │       │  │  Redis (fast   │  │
│  │                │  │       │  │  writes)       │  │
│  └────────────────┘  │       │  └────────────────┘  │
└──────────────────────┘       └──────────────────────┘

Team: 15 developers
Traffic: 50,000 orders/day
Cost: $1,200/month (optimized scaling)
```

**Phase 3: Year 3 (Team Growth)**
```
Problem: 30 developers, teams blocking deployments

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Restaurant      │  │   Order          │  │  GPS Tracking    │
│  Service         │  │   Service        │  │  Service         │
│  (Team A)        │  │   (Team B)       │  │  (Team C)        │
└──────────────────┘  └──────────────────┘  └──────────────────┘
         ↓                     ↓                      ↓
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Restaurant DB   │  │   Order DB       │  │   Redis          │
└──────────────────┘  └──────────────────┘  └──────────────────┘

         ┌──────────────────┐
         │  Payment         │
         │  Service         │
         │  (Team D)        │
         └──────────────────┘
                 ↓
         ┌──────────────────┐
         │  Payment DB      │
         └──────────────────┘

Team: 30 developers (4 teams)
Traffic: 500,000 orders/day
Cost: $3,500/month (independent scaling)
Each team deploys independently
```

## Capacity Estimation Example

### Scenario: Image Processing Service

**Requirements:**
- 10,000 product uploads/day
- Each product: 1 original image + 4 thumbnails (small, medium, large, mobile)
- Processing time: 2 seconds per image
- Peak traffic: 3x average during business hours

**Calculations:**

**Daily load:**
```
10,000 uploads × 5 images = 50,000 image operations/day
50,000 / 24 hours = 2,083 ops/hour average
2,083 / 3600 seconds = 0.58 ops/second average
Peak: 0.58 × 3 = 1.74 ops/second
```

**Processing capacity:**
```
1 server: 1 op / 2 seconds = 0.5 ops/second
For 1.74 ops/second peak: Need 4 servers (with buffer)
```

**If this was in monolith:**
```
Must run 4 servers for entire app
Cost: 4 × $200 = $800/month
```

**As microservice:**
```
Image service: 4 × $100 (CPU-optimized) = $400
Main app: 2 × $100 (general purpose) = $200
Total: $600/month (25% savings)

Plus: Can scale image service independently during product launches
```

## Failure Scenarios and Solutions

### Scenario 1: Service Dependency Failure

**Problem:**
```
Order Service → calls Payment Service → Payment Service is down
Result: Can't create orders
```

**Solutions:**

**1. Circuit Breaker Pattern**
```
If Payment Service fails:
- Stop calling it (don't cascade failures)
- Return cached response or degraded service
- Retry after cooldown period
```

**2. Async Processing**
```
Order Service:
1. Create order with status "PENDING_PAYMENT"
2. Publish message to queue: "process_payment"
3. Return to user: "Order received, processing payment"

Payment Service:
- Processes queue when available
- Updates order status to "PAID"
```

### Scenario 2: Data Inconsistency

**Problem:**
```
Order created → Payment processed → Inventory deduction fails
Result: Paid but no inventory reserved
```

**Solutions:**

**1. Saga Pattern**
```
Orchestrator coordinates:
1. Create order
2. Process payment
3. Reserve inventory
If step 3 fails → Compensate:
- Refund payment
- Cancel order
```

**2. Event Sourcing**
```
Store events:
- OrderCreated
- PaymentProcessed
- InventoryReservationFailed

Replay events to reach consistent state
```

## Monitoring and Observability

### Key Metrics to Track

**Service-Level Metrics:**
```
- Request rate (requests/second)
- Error rate (errors/second, % of requests)
- Latency (p50, p95, p99)
- Saturation (CPU, memory, disk I/O)
```

**Business Metrics:**
```
- Orders/day
- Conversion rate
- Payment success rate
- Average delivery time
```

**Distributed Tracing:**
```
Request flow:
User → API Gateway → Order Service → Product Service → Payment Service
         [200ms]      [50ms]         [30ms]           [100ms]

Trace ID: xyz-123
Total latency: 380ms
Bottleneck: Payment Service (100ms)
```

## Summary: When to Split

### Start Here (Monolith)
- Team < 20 developers
- Traffic < 100,000 requests/day
- Can scale vertically (bigger servers)
- Unified tech stack works
- Fast iteration more important than independence

### Consider Splitting When
- ✅ Database CPU > 80% from one feature
- ✅ Scaling costs 2x+ what independent scaling would cost
- ✅ Teams blocking each other's deploys weekly
- ✅ Need different tech for specific features
- ✅ 50+ developers stepping on each other

### Don't Split When
- ❌ Just following trends
- ❌ "Might need it someday"
- ❌ Small team (< 15 devs)
- ❌ Low traffic (< 10,000 requests/day)
- ❌ Can't handle operational complexity

### The Golden Rule

**"Add complexity only when the pain of the current architecture exceeds the pain of the added complexity."**

Evidence-based decisions beat premature optimization every time.
