# When to Split a Monolith into Microservices?

## Question

How do you decide when to split a monolithic application into microservices? What are the key factors and signals that indicate it's time to make this architectural change?

## Context

Many applications start as monoliths (single unified codebase where all components run together). At some point, teams consider splitting into microservices (separate, independent services). Understanding when and why to make this transition is crucial for building scalable systems.

## Requirements to Consider

### Functional Requirements
- Application must continue to function during and after the split
- Services need to communicate effectively
- Data consistency must be maintained across service boundaries

### Non-Functional Requirements
- **Scalability**: Ability to scale different components independently
- **Availability**: Minimize downtime during transitions
- **Performance**: Network latency from service-to-service calls
- **Maintainability**: Balance between coordination overhead and team independence

## Constraints

- Team size and structure
- Current traffic and growth projections
- Budget for infrastructure and operational overhead
- Existing technical debt
- Organizational ability to manage distributed systems
