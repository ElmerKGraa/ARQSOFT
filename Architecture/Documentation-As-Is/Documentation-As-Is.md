
Architecture Background
=======================

This document describes the background and rationale for the software architecture of the **Library Management System**. It explains constraints and influences that shaped the current design and enumerates the major architectural approaches adopted to meet the system’s functional and quality goals. It also highlights design alternatives and variation points considered during architectural design.

Problem Background
------------------

The system builds upon a previously developed REST-oriented backend that offers endpoints for **Books, Genres, Authors, Readers, and Lendings**. However, the prior solution lacks **extensibility, configurability, and reliability**, especially regarding multi-model persistence, pluggable external ISBN lookups, and ID generation strategies.

System Overview
---------------

The **Library Management System (BMS)** is a web-based platform that:

- Manages **catalog data** (books, authors, genres), **reader accounts**, and **lending workflows**.
- Supports **configurable persistence** across **SQL + Redis**, **MongoDB + Redis**, and **Elasticsearch** profiles.
- Integrates with external services to **retrieve ISBN from title** via **ISBNdb**, **Google Books**, or **Open Library**, selectable at setup time.
- Provides a **Single Page Application (SPA)** frontend and **Python-based analytics** (Pandas/Matplotlib) for operational insights (e.g., popular authors, lending patterns).
- Exposes an authenticated REST API for administrative and client-facing operations.

Context
=======

The architecture supports two primary use cases:

- **Operator/Staff**: master data curation, lending operations, data import/export, analytics.
- **Readers/Clients**: catalog browsing, account management (including GDPR requests), and reservations/loans tracking.

Software architecture plays a central role by enabling:

- **Run-time configurability** (choice of data stores, ISBN gateways, ID formats) with **setup-time selection** that affects behavior at runtime.
- **Reference-architecture guided design** using **ADD (Attribute-Driven Design)**, patterns, and tactics to achieve target quality attributes.

**Technology context**

- **Backend**: Java **Spring Boot** (REST controllers, services, repositories, gateways).
- **Data**: multiple profiles—**SQLite/PostgreSQL + Redis**, **MongoDB + Redis**, **Elasticsearch**.
- **Frontend**: SPA (e.g., React/Angular/Vue) consuming REST APIs.
- **Analytics**: Python (**Pandas**, **Matplotlib**) batch/offline jobs.
- **Deployment**: on-prem and cloud (IaaS/PaaS) with room for horizontal scaling.

Driving Requirements
====================

Functional Requirements (selected)
----------------------------------

- Manage **Books, Authors, Genres, Readers, Lendings** (CRUD, search, filters).
- **Import** and **export** catalog data (CSV/JSON).
- **Lending workflow**: checkout, return, renew, penalties.
- **ISBN retrieval by title** using pluggable gateways: **ISBNdb**, **Google Books**, **Open Library** (choose one or two in a composite strategy).
- **ID generation strategies** (e.g., UUID v7, KSUID, Snowflake-like) configurable per entity type.
- **Client features**: browse/search catalog, manage account, view loans/reservations.
- **Admin features**: data quality checks, audit trails, configuration console.
- **Analytics** (batch): circulation stats, top-N reports, collection growth.

Quality Attributes (FURPS+)
--------------------------

**Functionality**

- **Security & Privacy**: protect data via authentication/authorization; enforce **least privilege**; encrypt data in transit; support **GDPR** requests (export/delete).
- **Integrity & Auditability**: transactional updates; immutable audit logs for key actions.

**Usability**

- SPA must provide cohesive navigation for **master data**, **lending**, **analytics**, and **privacy** operations.

**Reliability**

- **Graceful degradation** on external ISBN gateway failures (retry/backoff/circuit breaker).
- **Cache** via Redis to reduce third-party dependency impact.

**Performance**

- **Search responsiveness** under expected loads; leverage **Elasticsearch** profile for full-text queries.
- **Caching** of hot reads and gateway responses; async tasks for long-running operations.

**Supportability**

- Clear configuration profiles; containerized services; health checks and metrics; extensible gateways.

Design Constraints
------------------

- **Single SPA** frontend accessing the backend services.
- **Multiple persistence models** selectable at setup time; choice affects runtime behavior and deployment topology.
- **External ISBN gateways** must be **pluggable** and **composable** (Strategy + Composite).
- **Automated testing** emphasis: functional (opaque/transparent-box), mutation tests; SUT spans class, subsystem, and system levels.

Implementation Constraints
--------------------------

- **Monorepo** recommended: SPA, Spring Boot, Python analytics.
- **Profiles** for **SQL+Redis**, **MongoDB+Redis**, **Elasticsearch** must be first-class; **feature flags** and **Spring profiles** drive activation.

Interface Constraints
---------------------

- RESTful API, **Richardson Maturity Model levels 0–2** (resources, HTTP verbs, basic HATEOAS where applicable).
- Auth via JWT/OAuth2; role-based access control.
- Analytics interface reads via internal API or data snapshots.

Physical Constraints
--------------------

- **Two application nodes** behind a load balancer for availability; **Redis** as shared cache; data stores per selected profile.
- On-prem + cloud split allowed; secure network configuration (TLS, firewall rules, secrets management).

Solution Background
===================

The architecture is constrained by the need for **configurable persistence** and **external service variability**, mandating **loose coupling** and **well-defined ports/adapters**. The approach aims to maximize **maintainability**, **evolvability**, **testability**, and **operational resilience**, while supporting **evidence-driven evaluation** through automated testing.

Architectural Approaches
========================

Styles and Patterns
-------------------

- **Client–Server**: SPA consumes REST APIs from the backend.
- **Web Application + SPA**: single-page client for usability and responsiveness.
- **SOA/Service-like modularization** within a Spring Boot application using **clean boundaries** and **Gateway** abstractions for external systems.
- **N-Tier**: presentation, application, domain, data/infrastructure tiers; deployable across on-prem/IaaS/PaaS.
- **Layered Architecture (Onion/Clean Architecture)**: domain model at the center; infrastructure and frameworks at the periphery.
- **Ports & Adapters (Hexagonal)**: for persistence and external ISBN gateways to enable runtime selection and testing seams.
- **Strategy + Factory**: select **persistence profile**, **ID generation**, and **ISBN gateway** strategies at startup.
- **Composite**: optionally combine two ISBN gateways (e.g., try Google Books then fallback to Open Library).
- **Circuit Breaker / Retry with Backoff**: resilience tactics for external calls.
- **Cache-Aside** (Redis): performance and resilience improvements.

Significant Alternatives & Rationale
------------------------------------

- **Event-Driven Integration** across modules was considered for extensibility but deferred to keep scope manageable; can be introduced later for analytics or search indexing pipelines.
- **Microservices** rejected for now; a **modular monolith** with clear boundaries meets requirements while reducing operational overhead.

Analysis Results
================

Quantitative analyses are pending. Qualitatively:

- **Onion/Ports & Adapters** + **Strategy** promotes testability and evolution of persistence and gateway choices.
- **Redis caching and circuit breakers** reduce external service risks.
- **Elasticsearch profile** offers superior search UX at the cost of operational complexity; can be toggled per deployment.

Implementation & Testing Strategy
=================================

- **Spring Profiles** drive adapter selection.
- **Feature flags** for composite ISBN gateway and ES search mode.
- **Testing**:
  - Unit (domain & strategies), mutation tests.
  - Integration (controller+service+repo/gateway).
  - System E2E (happy paths, failure injection for gateways).
  - Contract tests for gateway adapters (mocked external APIs).

Mapping Requirements to Architecture
====================================

| Requirement | Architectural Element(s) | Notes |
|---|---|---|
| CRUD for Books/Authors/Genres/Readers/Lendings | REST Controllers → Services → Domain → Repositories | Separation of concerns via layered/onion. |
| Import/Export Data | Application Services; CSV/JSON adapters | Batch endpoints; validation pipeline. |
| Lending Workflow | Domain model (Aggregates: Lending, Reader), Services | Transactions enforce invariants (availability, penalties). |
| ISBN by Title via external systems | **Gateway Port** + Strategies (ISBNdb, Google Books, Open Library) | Circuit breaker, retry, caching; composite gateway possible. |
| ID Generation Strategies | **IDPolicy** Strategy + Factory | Configurable per entity (UUID v7/KSUID/Snowflake-like). |
| Multi-Model Persistence | **Repository Ports** + Adapters for SQL/Mongo/ES; **Spring Profiles** | Switchable at setup; Redis cache shared. |
| SPA Usability | SPA Client + API | Cohesive navigation; consistent UX. |
| Security & GDPR | AuthN/AuthZ (JWT/OAuth2), Audit, Privacy endpoints | Export/delete account; least privilege. |
| Performance | Redis cache, async tasks, ES profile | Cache-aside; bulk endpoints. |
| Reliability | Circuit breaker, retries, fallbacks | Graceful degradation on gateway errors. |
| Testability & Evidence | Unit, integration, mutation, end-to-end tests | Class, subsystem, and system levels. |

Variation Points (Configuration Matrix)
======================================

- **Persistence**: `sql+redis` | `mongo+redis` | `elasticsearch`
- **ISBN Gateway**: `isbndb` | `google-books` | `open-library` | `composite(isbndb+google)`
- **ID Policy**: `uuidv7` | `ksuid` | `snowflake-like`
- **Search Mode**: `db-like` (SQL/Mongo) | `fulltext` (ES)
- **Auth Provider**: `local-jwt` | `oauth2`

Runtime View (High-Level)
=========================

- **SPA** → **REST API (Spring Boot)**
  - Controllers → Services → Domain
  - **Ports**: `RepositoryPort`, `IsbnLookupPort`, `IdPolicyPort`
  - **Adapters**: `JpaRepo/MongoRepo/EsRepo`, `IsbnDbGateway/GoogleBooksGateway/OpenLibraryGateway`, `UuidPolicy/KsuidPolicy/...`
  - **Infra**: Redis cache, DB/Cluster per profile, Observability (metrics, logs, health)

Deployment View (Baseline)
==========================

- **LB** → **App Node A / App Node B** (Spring Boot + SPA static assets)
- **Data Tier** (per profile):
  - SQL (SQLite dev / PostgreSQL prod) + **Redis**
  - or **MongoDB** + **Redis**
  - or **Elasticsearch** (+ Redis optional)
- **Python Analytics** job container (batch), accessing read-only snapshots/APIs.

Data View (Key Schemas / Indexes)
=================================

- **Books** (id, title, author_id, genre_id, isbn, year, copies, metadata…)
- **Authors** (id, name, birth_year, country…)
- **Genres** (id, name)
- **Readers** (id, email, hashed_pwd, preferences, gdpr_flags…)
- **Lendings** (id, book_id, reader_id, checkout_at, due_at, returned_at, penalties)
- **ES Index** (optional): analyzed fields (title, author_name, genre), facets.

Risks & Mitigations
===================

- **External API instability** → circuit breaker, caching, multiple providers.
- **Operational complexity (ES)** → limit to deployments requiring advanced search.
- **Data model divergence (SQL vs Mongo vs ES)** → repository ports + canonical domain model; focused adapters.

Roadmap (Incremental)
=====================

1. Stabilize **domain model** and **ports**.
2. Implement **SQL+Redis** baseline + one ISBN gateway.
3. Add **ID policies** and configuration console.
4. Add **Mongo+Redis** profile.
5. Add **Elasticsearch** profile.
6. Add **composite ISBN** gateway.
7. Integrate **Python analytics** batch.
8. Harden with **mutation tests** and **failure scenarios**.
