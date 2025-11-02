# Architecturally Significant Requirements (ASR)

**Project**: Library Management System (PSOFT-G1)
**Architecture Type**: Layered Architecture (3-Tier)
**Date**: 2025-10-29
**Version**: 2.0

---

## Table of Contents
1. [Introduction](#introduction)
2. [Quality Attributes Overview](#quality-attributes-overview)
3. [Security Requirements](#security-requirements)
4. [Performance & Scalability](#performance--scalability)
5. [Maintainability](#maintainability)
6. [Usability & Integration](#usability--integration)
7. [Data Integrity](#data-integrity)
8. [Key Architectural Decisions](#key-architectural-decisions)
9. [Traceability Matrix](#traceability-matrix)

---

## Introduction

This document identifies the Architecturally Significant Requirements (ASRs) for the Library Management System. ASRs represent requirements that have a profound impact on the system architecture, encompassing key quality attributes, constraints, and concerns that shaped our architectural decisions.

The system provides a RESTful API backend for managing library operations including authors, books, readers, and lending transactions. Our architecture emphasizes security, maintainability, and educational value while supporting typical library workflows.

---

## Quality Attributes Overview

The following quality attributes drive our architectural decisions, prioritized based on system requirements and constraints:

| Quality Attribute | Priority | Rationale |
|-------------------|----------|-----------|
| Security | High | Handles sensitive user data, financial transactions (fines), and role-based access |
| Maintainability | High | Academic project requiring clear structure, extensibility, and learning value |
| Testability | High | Comprehensive testing required for validation and quality assurance |
| Performance | Medium | Must support concurrent operations efficiently without over-engineering |
| Reliability | Medium | Data consistency critical for lending operations and fine calculations |
| Usability | Medium | API must be well-documented and intuitive for client integration |

---

## Security Requirements

### ASR-SEC-01: Stateless JWT Authentication

The system implements stateless authentication using JWT tokens with RSA asymmetric encryption to enable secure, scalable access control. This approach eliminates server-side session storage, allowing horizontal scaling and reducing server memory overhead.

**Implementation approach**:
- RSA key pair (public/private) stored securely in classpath resources
- JWT tokens contain user roles and claims for authorization decisions
- Token validation occurs on every request to protected endpoints
- Authentication decisions complete within 100ms

**Evidence**: [SecurityConfig.java:59-224](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L59-L224)

**Benefits**: This stateless design enables multiple server instances to validate tokens independently, supports horizontal scaling, and provides strong cryptographic security without requiring session synchronization across servers.

---

### ASR-SEC-02: Role-Based Access Control (RBAC)

Fine-grained authorization is enforced through three distinct roles (ADMIN, LIBRARIAN, READER) with endpoint-level access control. The system implements the principle of least privilege, ensuring users can only perform actions appropriate to their role.

**Role-based restrictions**:
- LIBRARIAN: Manages books, authors, and processes lendings
- READER: Borrows books, views suggestions, updates own profile
- ADMIN: Full system access for administrative operations
- Anonymous: Limited to reader registration only

**Evidence**: [SecurityConfig.java:110-172](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L110-L172)

**Benefits**: This separation protects sensitive operations from unauthorized access, prevents privilege escalation, and aligns with library operational workflows where librarians and readers have distinct responsibilities.

---

### ASR-SEC-03: Password Security & CORS

Passwords are secured using BCrypt hashing with salt, providing industry-standard protection against brute-force and rainbow table attacks. Additionally, CORS policies control cross-origin access for browser-based clients.

**Security measures**:
- BCrypt hashing is computationally expensive, slowing down attack attempts
- Unique salt per password prevents rainbow table attacks
- Passwords never logged, exposed in responses, or stored in plaintext
- CORS configuration allows controlled cross-origin access for legitimate frontends
- CSRF protection disabled (appropriate for stateless JWT APIs)

**Evidence**: Password encoder at [SecurityConfig.java:206-209](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L206-L209)

---

## Performance & Scalability

### ASR-PERF-01: Optimistic Locking for Concurrency

The system uses JPA's optimistic locking mechanism (`@Version`) to handle concurrent updates without database-level locks. This approach assumes conflicts are rare but detects them when they occur, providing better throughput than pessimistic locking.

**How it works**:
- Every mutable entity has a version field automatically incremented on updates
- Clients must provide expected version when submitting updates
- Version mismatch throws `StaleObjectStateException`, signaling concurrent modification
- Clients handle conflicts by retrying with fresh data

**Evidence**: Version fields in [Book.java:27-29](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L27-L29), [Author.java:19-20](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java#L19-L20), [Lending.java:96-101](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L96-L101)

**Rationale**: In a library system, simultaneous updates to the same entity are rare. Optimistic locking avoids the performance penalty of database locks while still preventing lost updates, aligning perfectly with our stateless API design.

---

### ASR-PERF-02: Pagination & Connection Pooling

Large result sets are handled through consistent pagination support, preventing memory exhaustion and reducing response times. Database connection pooling ensures efficient resource utilization under concurrent load.

**Implementation**:
- Custom `Page` class provides pagination metadata (page number, size, total)
- `SearchRequest` pattern standardizes paginated queries across endpoints
- H2 connection pooling (Spring Boot defaults) supports 50+ concurrent connections
- Response times maintain < 500ms regardless of total dataset size

**Evidence**: [Page.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/Page.java), [SearchRequest.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/SearchRequest.java)

**Benefits**: Pagination improves user experience with progressive loading, reduces bandwidth usage, and prevents loading thousands of records into memory. Connection pooling reuses database connections, significantly reducing overhead during peak usage.

---

## Maintainability

### ASR-MAINT-01: Layered Architecture Pattern

The system adopts a strict three-layer architecture (Presentation, Business, Data) to enable independent evolution of concerns and reduce coupling between components. Each layer has distinct responsibilities and dependencies flow downward only.

**Layer responsibilities**:
- **Presentation (API)**: REST controllers, request/response handling, OpenAPI documentation
- **Business (Service)**: Domain logic, business rules, transaction management, orchestration
- **Data (Repository)**: Database access, query implementation, entity persistence

**Package structure** (consistent across all modules):
```
{domain}management/
├── api/              ← Presentation Layer
├── services/         ← Business Layer
├── repositories/     ← Data Layer
└── model/            ← Domain Model
```

**Evidence**: Applied consistently in authormanagement, bookmanagement, lendingmanagement, readermanagement, genremanagement

**Benefits**: This well-understood pattern reduces learning curve, enables testing by mocking layer boundaries, facilitates future migrations (e.g., database changes), and allows independent layer evolution without cascading effects.

---

### ASR-MAINT-02: Domain-Driven Design Principles

The codebase is organized around domain aggregates (Author, Book, Lending, Reader) with rich domain models that encapsulate business logic. This approach aligns code structure with business concepts, making the system more understandable and maintainable.

**DDD patterns applied**:
- **Aggregates**: Documented domain boundaries with clear ownership
- **Value Objects**: ISBN, Title, Name, Email enforce validation and immutability
- **Repository Pattern**: Data access abstraction per aggregate
- **Ubiquitous Language**: Consistent terminology between code and domain experts

**Evidence**: Value objects like [ISBN.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Isbn.java), [Title.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Title.java), business logic in [Lending.java:157-170](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L157-L170)

**Benefits**: Rich domain models prevent anemic models by centralizing business rules in entities. This makes the system intuitive for both developers and domain experts, provides educational value for learning DDD, and ensures business logic isn't scattered across service layers.

---

### ASR-MAINT-03: Dependency Injection & Validation

Spring's dependency injection container manages component lifecycle and dependencies, enabling loose coupling and testability. Comprehensive validation at domain boundaries prevents invalid data from entering the system.

**Dependency injection approach**:
- Constructor injection for required dependencies (immutable, testable)
- Interface-based programming (Service interfaces with Impl classes)
- Component scanning with `@Service`, `@RestController`, `@Repository` annotations

**Validation strategy** (multi-layered):
- Jakarta Bean Validation annotations on DTOs and entities
- Domain logic validation in entity constructors and setters
- Business rule validation in service layer
- `GlobalExceptionHandler` for consistent error responses

**Evidence**: [GenreService.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreService.java) interface with [GenreServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java) implementation

**Benefits**: Loose coupling facilitates unit testing with mock dependencies, allows easy implementation swapping, and ensures invalid data is rejected early with meaningful error messages.

---

## Usability & Integration

### ASR-USE-01: RESTful API Design with OpenAPI Documentation

The API follows REST principles with resource-based URLs, standard HTTP verbs, and consistent response formats. Interactive documentation through Swagger UI enables easy exploration and testing.

**REST conventions**:
- Resource-based URLs: `/api/books/{isbn}`, `/api/authors/{id}`
- HTTP verbs map to operations: GET (read), POST (create), PATCH (update), DELETE (remove)
- Consistent `ListResponse` wrapper for collections
- Hypermedia links for related resources

**Documentation features**:
- SpringDoc OpenAPI 3 generates up-to-date documentation from code
- Swagger UI at `/swagger-ui` enables interactive testing
- Tag-based organization by domain module
- 100% endpoint coverage with request/response examples

**Evidence**: [application.properties:9-13](Original/src/main/resources/application.properties#L9-L13), [RestMapping.md](Original/Docs/Phase2/RestMapping.md)

**Benefits**: Self-documenting API reduces integration time, interactive testing improves developer experience, and RESTful design is widely understood and easy to consume from any HTTP client.

---

### ASR-INT-01: External Integration & File Upload

The system demonstrates extensibility through external API integration (API Ninjas for historical events) and file upload support for photos. Both features are isolated in dedicated packages for easy modification.

**External integration**:
- Dedicated service package for external APIs
- API key configuration externalized to properties file
- Loose coupling via service interfaces enables easy provider replacement

**File upload support**:
- Multipart file upload (Spring) with configurable size limits (200MB files, 20KB photos)
- `FileStorageService` in shared layer ensures consistent handling
- `EntityWithPhoto` base class promotes code reuse across entities
- `Photo` value object encapsulates photo URI and validation

**Evidence**: [ApiNinjasService.java](Original/src/main/java/pt/psoft/g1/psoftg1/external/service/ApiNinjasService.java), [FileStorageService.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/FileStorageService.java)

**Benefits**: Isolation in dedicated packages makes external dependencies easy to modify or replace. Centralized file handling ensures consistency, and configurable limits prevent resource exhaustion.

---

## Data Integrity

### ASR-DATA-01: Externalized Business Rules

Business rules (lending duration, fine values, age restrictions) are externalized to configuration files, allowing policy changes without code modification. This separates business policy from implementation logic.

**Configurable rules** (`library.properties`):
- Lending duration: 15 days
- Fine value per day: 200 cents
- Minimum reader age: 12 years
- Suggestion limits per genre: 2

**Evidence**: [library.properties](Original/src/main/resources/config/library.properties)

**Benefits**: Different configurations per environment (dev, test, prod), business rules are visible and auditable, and policy changes don't require recompilation or redeployment.

---

### ASR-DATA-02: Data Consistency Enforcement

Multi-level data validation ensures data integrity through database constraints, JPA relationship annotations, and domain-level validation. This defense-in-depth approach prevents data corruption.

**Consistency mechanisms**:
- JPA unique constraints on natural keys (ISBN, reader number, lending number)
- Not-null constraints on required fields
- Foreign key relationships with appropriate cascade rules
- Embedded value objects enforce format validation
- Schema auto-generation from entities ensures code-database alignment

**Special validations**:
- `ForbiddenNameService` blocks inappropriate names (centralized, configurable)
- Business rule validation in service layer (lending limits, eligibility)
- Domain validation in entity constructors (invariant enforcement)

**Evidence**: Unique constraints in [Book.java:19-21](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L19-L21), [ForbiddenNameServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/ForbiddenNameServiceImpl.java)

**Benefits**: Database-level constraints provide last line of defense, JPA mapping ensures consistency between code and schema, and fail-fast validation prevents invalid data from propagating.

---

## Key Architectural Decisions

### AD-01: Selection of Layered Architecture

**Decision**: Adopt three-layer (presentation, business, data) architecture pattern over microservices or hexagonal architecture.

**Rationale**: For an educational project with a single development team, layered architecture provides the optimal balance of clarity, maintainability, and learning value. Microservices would introduce unnecessary complexity and operational overhead without corresponding benefits. The well-understood layered pattern enables focus on domain modeling and business logic rather than distributed systems concerns.

**Trade-offs**: While we cannot scale individual layers independently and must deploy as a single unit, this is acceptable for the project scope. The simplicity gains outweigh the scalability limitations for a library management system.

---

### AD-02: JWT with RSA Keys for Authentication

**Decision**: Use JWT tokens with RSA asymmetric encryption rather than session-based authentication or OAuth2.

**Rationale**: Stateless JWT authentication enables horizontal scaling without session synchronization, aligns with REST principles, and provides strong cryptographic security. RSA keys are preferred over HMAC for better key distribution properties and stronger security guarantees.

**Trade-offs**: Individual tokens cannot be revoked before expiration (requires token blacklisting for logout), and RSA tokens are larger than session IDs. However, the scalability and security benefits justify these trade-offs.

---

### AD-03: H2 Embedded Database

**Decision**: Use H2 embedded database for persistence instead of PostgreSQL or MySQL.

**Rationale**: For an academic project with moderate data volumes, H2 provides zero-configuration setup, fast development iterations, and suitability for demonstration purposes. The embedded database supports multiple modes (TCP, file, in-memory) for different testing scenarios.

**Trade-offs**: Limited scalability compared to production databases and fewer advanced features. However, JPA abstraction allows easy migration to PostgreSQL in future if needed.

---

### AD-04: Domain-Driven Design Patterns

**Decision**: Apply DDD patterns (aggregates, value objects, repositories, domain services) rather than anemic domain models or transaction scripts.

**Rationale**: The library domain has sufficient complexity to benefit from rich domain models with encapsulated behavior. DDD provides educational value, improves communication with domain experts through ubiquitous language, and prevents service layer bloat.

**Trade-offs**: Steeper learning curve and more classes than anemic models, but the improved maintainability and expressiveness justify the additional complexity.

---

### AD-05: Optimistic Locking Strategy

**Decision**: Use optimistic locking (`@Version`) rather than pessimistic locking for concurrency control.

**Rationale**: In a library system, concurrent updates to the same entity are rare. Optimistic locking provides better performance and higher throughput without database locks, aligning perfectly with stateless API design.

**Trade-offs**: Clients must implement retry logic for conflicts, and work may be wasted if conflicts occur. However, the performance benefits and simplicity outweigh these concerns.

---

## Traceability Matrix

| ASR ID | Quality Attribute | Key Implementation | Architectural Decision |
|--------|-------------------|-------------------|------------------------|
| ASR-SEC-01 | Security | JWT with RSA keys, SecurityConfig | AD-02 |
| ASR-SEC-02 | Security | RBAC rules, Role-based endpoints | AD-02 |
| ASR-SEC-03 | Security | BCrypt hashing, CORS configuration | AD-02 |
| ASR-PERF-01 | Performance | @Version optimistic locking | AD-05 |
| ASR-PERF-02 | Performance | Pagination, connection pooling | AD-03 |
| ASR-MAINT-01 | Maintainability | Three-layer package structure | AD-01 |
| ASR-MAINT-02 | Maintainability | DDD aggregates, value objects | AD-04 |
| ASR-MAINT-03 | Maintainability | Dependency injection, validation | AD-01, AD-04 |
| ASR-USE-01 | Usability | RESTful design, OpenAPI/Swagger | AD-01 |
| ASR-INT-01 | Interoperability | External API service, file uploads | AD-01 |
| ASR-DATA-01 | Flexibility | Externalized business rules | AD-01 |
| ASR-DATA-02 | Reliability | JPA constraints, forbidden names | AD-03, AD-04 |

---

## Validation and Review

**Validation Method**: Reverse engineering from implemented codebase, cross-referenced with existing documentation and 144 Java source files.

**Evidence Sources**: Source code analysis, configuration files (application.properties, library.properties), existing documentation (domain model, aggregates, REST mappings).

**Review Status**: Completed
**Review Date**: 2025-10-29

---

## References

- [Domain Model](../GlobalArtifacts/DomainModel.puml)
- [Glossary](../GlobalArtifacts/Glossary.md)
- [REST API Mapping](../Phase2/RestMapping.md)
- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/)
- [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)

---

**Document Control**
Version: 2.0
Last Updated: 2025-11-02
Status: Complete (Summarized Edition)
Next Review: Upon major architectural changes
