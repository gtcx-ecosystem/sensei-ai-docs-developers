---
description: PostgreSQL, Redis, and Neo4j choices
---

# ADR 0003: Database Selection

## Status

Accepted

## Date

2025-07-01

## Context

The Sensei platform requires persistent storage, caching, and event streaming infrastructure to support the following workloads:

1. **Primary data store.** Migration metadata (plans, mappings, status), connection configurations, user accounts, organization settings, audit logs, and verification certificates. Requires ACID transactions, complex querying (joins, aggregations, full-text search), and strong consistency guarantees.

2. **Caching and transient state.** Session data, rate limiting counters, agent task queues, real-time migration progress, and ephemeral coordination state between agents. Requires sub-millisecond reads, TTL-based expiration, and pub/sub capabilities.

3. **Event streaming.** Migration lifecycle events, agent communication, log aggregation, webhook delivery, and cross-service coordination. Requires durable ordered message delivery, consumer groups, replay capability, and high throughput under sustained load.

The team evaluated candidates for each role against the following criteria: operational maturity, performance characteristics, ecosystem support, hosting flexibility (managed and self-hosted), and team familiarity.

## Decision

### Primary Store: PostgreSQL

PostgreSQL is selected as the primary relational database.

**Rationale:**

- ACID compliance with strong consistency guarantees suitable for migration metadata that must not be lost or corrupted.
- JSONB columns provide schema flexibility for migration plan structures and agent state that varies by migration type, without requiring a separate document store.
- Full-text search capabilities (tsvector/tsquery) eliminate the need for a dedicated search index for log and audit queries at current scale.
- Row-level security supports multi-tenant isolation at the database layer.
- Mature ecosystem: Prisma ORM integration with TypeScript, well-documented operational procedures, extensive extension library (pg_cron, pgvector for future embedding storage).
- Available as managed service on all major cloud providers (RDS, Cloud SQL, Azure Database) and self-hostable for on-premises deployments.

**Alternatives considered:** MySQL (weaker JSONB support, less expressive query planner), MongoDB (eventual consistency model unsuitable for migration metadata), CockroachDB (distributed consistency unnecessary at current scale, higher operational complexity).

### Caching and Queues: Redis

Redis is selected for caching, rate limiting, session management, and lightweight task queuing.

**Rationale:**

- Sub-millisecond read latency for cached migration state and rate limiting counters.
- Native TTL support for session expiration and cache invalidation.
- Redis Streams provide lightweight task queuing for agent coordination without the operational overhead of a full message broker for low-throughput internal queues.
- Pub/sub capabilities support real-time migration progress updates to connected dashboard clients.
- Sentinel and Cluster modes provide high availability without architectural changes.

**Alternatives considered:** Memcached (no persistence, no pub/sub, no streams), DragonflyDB (promising but insufficient production track record), Valkey (viable long-term fork but ecosystem maturity still developing).

### Event Streaming: Apache Kafka

Apache Kafka is selected for durable event streaming.

**Rationale:**

- Durable, ordered message delivery with configurable retention. Migration lifecycle events must be replayable for audit and debugging purposes.
- Consumer groups support independent processing of the same event stream by multiple services (e.g., webhook delivery, log aggregation, analytics).
- Partitioned topics provide horizontal throughput scaling as migration volume grows.
- Schema Registry integration enforces event schema evolution with backward compatibility guarantees.
- Mature ecosystem with connectors for PostgreSQL (CDC via Debezium), monitoring (Prometheus), and cloud-managed offerings (Confluent Cloud, Amazon MSK).

**Alternatives considered:** RabbitMQ (lacks durable replay, weaker at sustained high-throughput streaming), Amazon SQS (no ordering guarantees without FIFO queues, no replay), Apache Pulsar (smaller ecosystem and community, fewer managed offerings), NATS JetStream (promising but less mature for the event sourcing patterns required by the audit trail).

## Consequences

**Benefits:**

- Each technology serves the workload it was designed for: PostgreSQL for transactional consistency, Redis for speed, Kafka for durable streaming.
- All three are available as both managed services and self-hosted deployments, preserving hosting flexibility.
- The combination is well-documented in production at scale by many organizations, reducing operational risk.

**Drawbacks:**

- Three distinct data systems increase operational surface area. Mitigated by using managed services in production and Docker Compose for local development.
- Data must be kept consistent across systems (e.g., migration status in PostgreSQL and progress in Redis). Mitigated by treating PostgreSQL as the source of truth and all other systems as derived state.
- Kafka introduces meaningful operational complexity (broker management, partition rebalancing, consumer lag monitoring). Mitigated by Confluent Cloud or Amazon MSK for production deployments.

**Migration path:** If event volume exceeds Kafka's cost-effectiveness threshold or if the team's operational capacity is constrained, Redpanda is identified as a drop-in compatible alternative with lower operational overhead.

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Architecture](https://redis.io/docs/management/optimization/)
- [Kafka Design](https://kafka.apache.org/documentation/#design)
- ADR-0001: Monorepo Structure
