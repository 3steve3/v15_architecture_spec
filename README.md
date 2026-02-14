# V15 Composable Commerce OS - Architectural Overview

V15 is a proprietary, headless, multi-tenant commerce and operations engine designed for high-integrity, high-concurrency environments. The architecture prioritizes data integrity, auditability, and radical composability, allowing it to adapt to diverse industry verticals without code changes.

**Status:** Proprietary / Closed Source (Commercial Potential)
**Stack:** .NET 10 (LTS), PostgreSQL 18, TimescaleDB, gRPC, GraphQL (Hot Chocolate)

---

## Architectural Pillars

The V15 architecture is built on five core principles that ensure security, scalability, and maintainability.

### 1. Schema-First, Domain-Driven Design
The system is segregated into nine distinct database schemas (`iam`, `audit`, `platform`, `catalog`, `inventory`, `finance`, `work`, `events`, `attribution`). This is not just for organization; it enforces strict logical boundaries between different business domains.

- **`iam` (Identity):** Manages the global identity of users versus their tenant-specific memberships and roles. This separation is critical for secure multi-tenancy.
- **`inventory` vs. `finance`:** The physical state of the world (`inventory`) is explicitly decoupled from its financial representation (`finance`), preventing operational events from corrupting the financial ledger.
- **`work`:** Operational processes like service tickets are treated as first-class entities, allowing for deep profitability analysis on service delivery.

This approach ensures that complexity in one domain (e.g., a new inventory tracking method) does not create instability in another (e.g., financial reporting).

### 2. Immutable, Append-Only Auditing via Hypertables
Every state change in the system—every `INSERT`, `UPDATE`, and `DELETE`—is recorded as an immutable event in the `audit.trail` hypertable.

- **Technology:** Leverages **TimescaleDB** to handle billions of events with extreme time-series query performance.
- **Data Integrity:** `audit.trail` is configured with a 7-year retention policy and is the ultimate source of truth for forensic analysis, compliance checks, and debugging.
- **Functionality:** This event log is not just for security; it is the engine that drives the **Attribution** schema, allowing for a historical "time machine" to calculate the true profitability of any job, customer, or product.

### 3. Database-Level Security (Row Level Security)
The system does not trust the application layer to enforce security. The database itself is hardened with **PostgreSQL Row Level Security (RLS)**.

- **Mechanism:** A dynamic `tenant_isolation_policy` is programmatically applied to every table containing a `tenant_id`.
- **Effect:** It is **mathematically impossible** for a user from Tenant A to see, modify, or even know about the existence of data from Tenant B, even in the event of a severe application-level bug.
- **The Guarantee:** This provides the data segregation required for high-trust and regulated industries.

### 4. Radical Composability (The `platform` Schema)
The OS is designed to be a "white-label" engine, not a rigid application. The `platform` schema allows tenants to reconfigure the system to match their specific industry vertical.

- **Custom Fields:** Tenants can extend the core schema with their own data points (`vin_number` for automotive, `lot_code` for F&B) without requiring a new deployment.
- **Terminology Overrides:** A restaurant can rename "Customers" to "Guests" and "Work Orders" to "Tickets" via a simple configuration entry.
- **Feature Gating:** Functionality is controlled by `feature_flags` that are tied to subscription tiers, allowing for a flexible "Starter vs. Enterprise" business model.

### 5. Reliable Integration via the Transactional Outbox Pattern
V15 is built to be a "good citizen" in a microservices ecosystem. The `events` schema implements the Transactional Outbox pattern to guarantee that events are delivered to external systems (like a shipping API or a marketing platform) exactly once.

- **The Problem it Solves:** Prevents "dual-write" failures where a database commit succeeds but the corresponding event fails to send, leaving the system in an inconsistent state.
- **The Result:** The system can guarantee that if an invoice is created, the "Invoice Created" event *will* be delivered, ensuring architectural resilience and reliable integration.

---

## The Hybrid API Gateway: gRPC + GraphQL

V15 utilizes a "best of both worlds" approach for its API layer, acknowledging that no single protocol is perfect for every use case.

- **gRPC (Internal & High-Performance):** Used for service-to-service communication and high-frequency data streams (like the `audit.trail` ingestion). Its strict, protobuf-based contracts ensure maximum performance and type safety for the internal "Fortress" logic.

- **GraphQL (External & Flexible):** The public-facing API for web and mobile clients is a **GraphQL endpoint (via Hot Chocolate)**. This layer sits *in front of* the gRPC services.
    - **Why:** It allows frontend developers to query for exactly the data they need in a single round trip, preventing the over-fetching common with REST.
    - **The Pattern:** A GraphQL resolver receives a query, then makes one or more high-performance gRPC calls to the internal services to fetch the data, composing it into the final response. This provides a flexible "Vibe" for the frontend while maintaining the strict, integrity of the backend.

---

## Contact

[LinkedIn](https://www.linkedin.com/in/steve-naumov/) | [Email](stevennaumov@outlook.com)
