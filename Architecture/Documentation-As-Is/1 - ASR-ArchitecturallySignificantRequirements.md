# Architecturally Significant Requirements (ASR)

**Project**: Library Management System (PSOFT-G1)
**Architecture Type**: Layered Architecture (3-Tier)
**Date**: 2025-10-29
**Version**: 1.0

---

## Table of Contents
1. [Introduction](#introduction)
2. [Quality Attributes](#quality-attributes)
3. [Security ASRs](#security-asrs)
4. [Performance & Scalability ASRs](#performance--scalability-asrs)
5. [Maintainability ASRs](#maintainability-asrs)
6. [Usability ASRs](#usability-asrs)
7. [Integration ASRs](#integration-asrs)
8. [Data Integrity ASRs](#data-integrity-asrs)
9. [Architectural Decisions](#architectural-decisions)
10. [ASR Traceability Matrix](#asr-traceability-matrix)

---

## Introduction

This document identifies and describes the Architecturally Significant Requirements (ASRs) for the Library Management System. ASRs are requirements that have a profound impact on the architecture of the system. They represent the key quality attributes, constraints, and concerns that shaped the architectural decisions.

The system is a RESTful API backend for managing library operations including authors, books, readers, and lending transactions.

---

## Quality Attributes

The following quality attributes have been identified as architecturally significant for this system:

| Quality Attribute | Priority | Rationale |
|-------------------|----------|-----------|
| Security | High | Handles sensitive user data, financial transactions (fines), and access control |
| Maintainability | High | Academic project requiring clear structure and extensibility |
| Performance | Medium | Must handle concurrent user operations efficiently |
| Usability | Medium | API must be well-documented and easy to integrate |
| Reliability | Medium | Data consistency critical for lending operations and fine calculations |
| Testability | High | Academic project requiring comprehensive testing |

---

## Security ASRs

### ASR-SEC-01: JWT-Based Authentication
**Quality Attribute**: Security, Performance
**Priority**: High

**Scenario**:
- **Source**: External client (web/mobile application)
- **Stimulus**: User attempts to access protected resources
- **Artifact**: Security subsystem
- **Environment**: Normal operation
- **Response**: System validates JWT token and grants/denies access
- **Response Measure**: Authentication decision made within 100ms, zero unauthorized access

**Architectural Decision**:
- Implemented stateless JWT authentication using RSA asymmetric keys
- Public/private key pair stored in classpath resources
- JWT tokens contain user roles and claims
- Token validation on every request to protected endpoints

**Implementation Evidence**:
- [SecurityConfig.java:59-224](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L59-L224)
- RSA key configuration: [application.properties:18-19](Original/src/main/resources/application.properties#L18-L19)
- JWT encoder/decoder beans defined in SecurityConfig

**Rationale**:
- Stateless authentication enables horizontal scaling
- RSA keys provide strong cryptographic security
- Eliminates need for session storage on server
- JWT tokens can be validated independently by multiple server instances

---

### ASR-SEC-02: Role-Based Access Control (RBAC)
**Quality Attribute**: Security
**Priority**: High

**Scenario**:
- **Source**: Authenticated user
- **Stimulus**: Attempts to perform operation (create, read, update, delete)
- **Artifact**: API endpoints
- **Environment**: Normal operation
- **Response**: System checks user role and permits/denies operation
- **Response Measure**: 100% enforcement of role-based permissions, zero privilege escalation

**Architectural Decision**:
- Three distinct roles: ADMIN, LIBRARIAN, READER
- Fine-grained endpoint-level authorization rules
- Method-level security annotations enabled
- Anonymous access only for reader registration

**Implementation Evidence**:
- Role definitions: [Role.java](Original/src/main/java/pt/psoft/g1/psoftg1/usermanagement/model/Role.java)
- Security rules: [SecurityConfig.java:110-172](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L110-L172)
- Example rules:
  - POST /api/authors → LIBRARIAN only
  - GET /api/authors → READER or LIBRARIAN
  - POST /api/readers → Anonymous (registration)
  - PATCH /api/readers → READER (self only)

**Rationale**:
- Implements principle of least privilege
- Separates concerns between librarians (administrative) and readers (borrowing)
- Protects sensitive operations (book management, reporting) from unauthorized access

---

### ASR-SEC-03: Password Security
**Quality Attribute**: Security
**Priority**: High

**Scenario**:
- **Source**: User during registration/authentication
- **Stimulus**: Provides password credentials
- **Artifact**: User management subsystem
- **Environment**: Normal operation
- **Response**: System stores/validates password securely
- **Response Measure**: Passwords never stored in plaintext, bcrypt algorithm used

**Architectural Decision**:
- BCrypt password hashing with salt
- Password strength validation in domain model
- Passwords never logged or exposed in API responses

**Implementation Evidence**:
- Password encoder bean: [SecurityConfig.java:206-209](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L206-L209)
- Password value object: [Password.java](Original/src/main/java/pt/psoft/g1/psoftg1/usermanagement/model/Password.java)

**Rationale**:
- Industry-standard password security
- BCrypt is computationally expensive, resistant to brute-force attacks
- Salting prevents rainbow table attacks

---

### ASR-SEC-04: CORS and API Security
**Quality Attribute**: Security
**Priority**: Medium

**Scenario**:
- **Source**: Browser-based client application
- **Stimulus**: Cross-origin API request
- **Artifact**: HTTP security filters
- **Environment**: Production deployment with web frontend
- **Response**: System validates origin and applies CORS policy
- **Response Measure**: Legitimate origins allowed, others blocked

**Architectural Decision**:
- CORS configuration allowing controlled cross-origin access
- CSRF protection disabled (appropriate for stateless JWT APIs)
- Bearer token authentication on all protected endpoints

**Implementation Evidence**:
- CORS filter: [SecurityConfig.java:211-222](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L211-L222)
- CSRF disabled: [SecurityConfig.java:99](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L99)

**Rationale**:
- Enables browser-based frontend applications
- CSRF not needed for stateless bearer token authentication
- Allows API consumption from authorized web applications

---

## Performance & Scalability ASRs

### ASR-PERF-01: Optimistic Locking for Concurrency
**Quality Attribute**: Performance, Data Integrity
**Priority**: High

**Scenario**:
- **Source**: Multiple concurrent users
- **Stimulus**: Attempt to update the same entity (book, author, lending) simultaneously
- **Artifact**: Domain entities
- **Environment**: High concurrency during peak library hours
- **Response**: System detects concurrent modification and prevents lost updates
- **Response Measure**: Zero lost updates, conflicts detected and reported to users

**Architectural Decision**:
- JPA `@Version` annotation on all mutable entities
- Version checking in domain model before updates
- StaleObjectStateException thrown on version mismatch
- Clients must provide expected version when updating

**Implementation Evidence**:
- Book versioning: [Book.java:27-29](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L27-L29)
- Author versioning: [Author.java:19-20](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java#L19-L20)
- Lending versioning: [Lending.java:96-101](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L96-L101)
- Version check example: [Book.java:94-96](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L94-L96)

**Rationale**:
- Optimistic locking provides better performance than pessimistic locking
- Conflicts are rare in library domain (same book updated simultaneously)
- Avoids database-level locks, improving throughput
- Aligns with stateless API design

---

### ASR-PERF-02: Database Connection Pooling
**Quality Attribute**: Performance, Scalability
**Priority**: High

**Scenario**:
- **Source**: Multiple concurrent API requests
- **Stimulus**: Database queries required for operations
- **Artifact**: Database connection layer
- **Environment**: Normal to peak load
- **Response**: System efficiently manages database connections
- **Response Measure**: Connection acquisition < 50ms, supports 50+ concurrent connections

**Architectural Decision**:
- H2 database with connection pooling (Spring Boot defaults)
- JPA/Hibernate as ORM layer
- Lazy/Eager fetching strategies configured per relationship

**Implementation Evidence**:
- Database configuration: [application.properties:22-33](Original/src/main/resources/application.properties#L22-L33)
- Eager fetching example: [Lending.java:57-58](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L57-L58)

**Rationale**:
- Connection pooling reuses connections, reducing overhead
- Appropriate for moderate concurrency expected in library system
- H2 embedded database suitable for development and small deployments

---

### ASR-PERF-03: Pagination Support
**Quality Attribute**: Performance, Usability
**Priority**: Medium

**Scenario**:
- **Source**: Client application
- **Stimulus**: Requests large result sets (all books, all lendings)
- **Artifact**: Repository layer, API endpoints
- **Environment**: Normal operation with large data volumes
- **Response**: System returns paginated results
- **Response Measure**: Response time < 500ms regardless of total result set size

**Architectural Decision**:
- Custom Page class for pagination metadata
- SearchRequest pattern for queries with pagination
- Consistent pagination across all search endpoints

**Implementation Evidence**:
- Page model: [Page.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/Page.java)
- SearchRequest pattern: [SearchRequest.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/SearchRequest.java)
- Usage example: [GenreController.java:21-26](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L21-L26)

**Rationale**:
- Prevents loading large datasets into memory
- Improves response times for large result sets
- Reduces bandwidth usage
- Better user experience with progressive loading

---

## Maintainability ASRs

### ASR-MAINT-01: Layered Architecture
**Quality Attribute**: Maintainability, Modifiability
**Priority**: High

**Scenario**:
- **Source**: Developer
- **Stimulus**: Needs to modify business logic or change database
- **Artifact**: System architecture
- **Environment**: Development/maintenance
- **Response**: Changes isolated to specific layers without cascading effects
- **Response Measure**: Average change impacts 1-2 layers, not entire system

**Architectural Decision**:
- Three-layer architecture: Presentation (API) → Business (Service) → Data (Repository)
- Clear separation of concerns with distinct responsibilities per layer
- Dependencies flow downward only (API → Service → Repository)
- No direct access from API to Repository layer

**Implementation Evidence**:
- Consistent package structure across all modules:
  ```
  authormanagement/
  ├── api/              ← Presentation Layer
  ├── services/         ← Business Layer
  ├── repositories/     ← Data Layer
  └── model/            ← Domain Model
  ```
- Same pattern in: bookmanagement, lendingmanagement, readermanagement, genremanagement

**Rationale**:
- Well-understood pattern, reduces learning curve
- Enables independent evolution of layers
- Facilitates testing (can mock layer boundaries)
- Supports future migration (e.g., replace H2 with PostgreSQL)

---

### ASR-MAINT-02: Domain-Driven Design (DDD)
**Quality Attribute**: Maintainability, Understandability
**Priority**: High

**Scenario**:
- **Source**: Developer (new team member)
- **Stimulus**: Needs to understand system and implement new feature
- **Artifact**: Domain model, code organization
- **Environment**: Development
- **Response**: Developer can understand domain concepts and locate relevant code
- **Response Measure**: New developer productive within 2 days

**Architectural Decision**:
- Organized around domain aggregates (Author, Book, Lending, Reader)
- Rich domain models with business logic encapsulated in entities
- Value Objects for domain concepts (ISBN, Title, Name, Email, etc.)
- Repository pattern for data access
- Bounded contexts per domain module

**Implementation Evidence**:
- Aggregates documented: [Docs/GlobalArtifacts/Aggregates/](Original/Docs/GlobalArtifacts/Aggregates/)
- Value Objects:
  - [ISBN.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Isbn.java)
  - [Title.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Title.java)
  - [Name.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/model/Name.java)
  - [EmailAddress.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/EmailAddress.java)
- Business logic in entities: [Lending.java:157-170](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L157-L170) (setReturned method)
- Domain glossary: [Glossary.md](Original/Docs/GlobalArtifacts/Glossary.md)

**Rationale**:
- Aligns code structure with business domain (ubiquitous language)
- Makes system easier to understand for both developers and domain experts
- Rich domain models prevent anemic models, centralizing business rules
- Educational value for learning DDD principles

---

### ASR-MAINT-03: Dependency Injection and IoC
**Quality Attribute**: Maintainability, Testability
**Priority**: High

**Scenario**:
- **Source**: Developer
- **Stimulus**: Needs to test service layer or replace implementation
- **Artifact**: Service classes, controllers
- **Environment**: Development, testing
- **Response**: Dependencies can be injected without code changes
- **Response Measure**: 100% of dependencies injectable, all layers testable in isolation

**Architectural Decision**:
- Spring Framework's Dependency Injection container
- Constructor injection for required dependencies
- Interface-based programming (Service interfaces with Impl classes)
- `@Service`, `@RestController`, `@Repository` annotations for component scanning

**Implementation Evidence**:
- Service interfaces: [GenreService.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreService.java)
- Service implementations: [GenreServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Constructor injection: [GenreController.java:17-19](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L17-L19)

**Rationale**:
- Loose coupling between components
- Facilitates unit testing with mock dependencies
- Easy to swap implementations (e.g., caching layer, different repository)
- Spring IoC container manages object lifecycle

---

### ASR-MAINT-04: Validation at Domain Boundaries
**Quality Attribute**: Maintainability, Reliability
**Priority**: High

**Scenario**:
- **Source**: Client application
- **Stimulus**: Submits invalid data (null, empty, wrong format)
- **Artifact**: Domain model, DTOs
- **Environment**: Normal operation
- **Response**: System rejects invalid data with clear error messages
- **Response Measure**: Zero invalid data persisted, meaningful error messages

**Architectural Decision**:
- Jakarta Bean Validation annotations on DTOs and entities
- Domain logic validation in entity constructors and setters
- Business rule validation in services
- GlobalExceptionHandler for consistent error responses

**Implementation Evidence**:
- Genre validation: [Genre.java:16-34](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/model/Genre.java#L16-L34)
- Lending commentary validation: [Lending.java:107-109](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L107-L109)
- Exception handler: [GlobalExceptionHandler.java](Original/src/main/java/pt/psoft/g1/psoftg1/exceptions/GlobalExceptionHandler.java)

**Rationale**:
- Fail-fast principle prevents corruption of data
- Clear error messages improve API usability
- Validation at domain level ensures consistency regardless of entry point
- Prevents invalid state in domain objects

---

## Usability ASRs

### ASR-USE-01: OpenAPI/Swagger Documentation
**Quality Attribute**: Usability
**Priority**: Medium

**Scenario**:
- **Source**: API consumer (frontend developer)
- **Stimulus**: Needs to understand available endpoints and request/response formats
- **Artifact**: API documentation
- **Environment**: Development, integration
- **Response**: System provides interactive, up-to-date API documentation
- **Response Measure**: 100% endpoint coverage, interactive testing capability

**Architectural Decision**:
- SpringDoc OpenAPI 3 integration
- Swagger UI available at `/swagger-ui`
- API docs available at `/api-docs`
- Tag-based organization by domain

**Implementation Evidence**:
- OpenAPI configuration: [application.properties:9-13](Original/src/main/resources/application.properties#L9-L13)
- Controller tags: [GenreController.java:13](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L13)

**Rationale**:
- Self-documenting API reduces integration time
- Interactive testing through Swagger UI improves developer experience
- Auto-generated from code, always up-to-date
- Industry standard for REST API documentation

---

### ASR-USE-02: RESTful API Design
**Quality Attribute**: Usability, Interoperability
**Priority**: High

**Scenario**:
- **Source**: API consumer (client application)
- **Stimulus**: Needs to perform CRUD operations on resources
- **Artifact**: API endpoints
- **Environment**: Normal operation
- **Response**: System provides intuitive, standard REST endpoints
- **Response Measure**: API adheres to REST principles, predictable resource URLs

**Architectural Decision**:
- Resource-based URLs (e.g., `/api/books/{isbn}`)
- HTTP verbs mapped to operations (GET, POST, PUT, PATCH, DELETE)
- Consistent response formats with ListResponse wrapper
- Hypermedia links for related resources

**Implementation Evidence**:
- REST mapping documentation: [RestMapping.md](Original/Docs/Phase2/RestMapping.md)
- Consistent URL patterns:
  - GET /api/authors → list authors
  - GET /api/authors/{id} → get specific author
  - POST /api/authors → create author
  - PATCH /api/authors/{id} → update author
- ListResponse wrapper: [ListResponse.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/api/ListResponse.java)

**Rationale**:
- RESTful design is widely understood and expected
- Stateless interactions support scalability
- HTTP semantics provide clear operation meanings
- Easy to consume from any HTTP client

---

## Integration ASRs

### ASR-INT-01: External API Integration
**Quality Attribute**: Interoperability, Extensibility
**Priority**: Low

**Scenario**:
- **Source**: System (enrichment feature)
- **Stimulus**: Needs historical event data for author birthdays
- **Artifact**: External service integration
- **Environment**: Normal operation with internet connectivity
- **Response**: System fetches data from external API and enriches response
- **Response Measure**: External API response integrated within 2 seconds

**Architectural Decision**:
- Dedicated external service package
- API Ninjas service for historical events
- API key configuration in properties file
- Loose coupling via service interface

**Implementation Evidence**:
- API Ninjas service: [ApiNinjasService.java](Original/src/main/java/pt/psoft/g1/psoftg1/external/service/ApiNinjasService.java)
- API key configuration: [application.properties:74](Original/src/main/resources/application.properties#L74)
- Config class: [ApiNinjasConfig.java](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/ApiNinjasConfig.java)

**Rationale**:
- Demonstrates integration capability
- Isolated in dedicated package for easy modification
- Configurable via properties (different keys per environment)
- Can be easily replaced or extended with other services

---

### ASR-INT-02: File Upload and Storage
**Quality Attribute**: Usability, Extensibility
**Priority**: Medium

**Scenario**:
- **Source**: Client application
- **Stimulus**: Uploads photo for author, reader, or book
- **Artifact**: File storage service
- **Environment**: Normal operation
- **Response**: System stores file and associates with entity
- **Response Measure**: Files up to 200MB supported, photos up to 20KB, files stored securely

**Architectural Decision**:
- Multipart file upload support (Spring)
- Dedicated file storage directory configured via properties
- FileStorageService in shared layer
- EntityWithPhoto base class for entities with photos
- Photo value object encapsulating photo URI

**Implementation Evidence**:
- File upload config: [application.properties:54-71](Original/src/main/resources/application.properties#L54-L71)
- FileStorageService: [FileStorageService.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/FileStorageService.java)
- EntityWithPhoto base class: [EntityWithPhoto.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/model/EntityWithPhoto.java)
- Photo model: [Photo.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/model/Photo.java)
- Upload directory: `uploads-psoft-g1/`

**Rationale**:
- Centralized file handling ensures consistency
- Configurable storage location supports different environments
- Size limits prevent abuse and resource exhaustion
- Base class promotes code reuse across entities with photos

---

## Data Integrity ASRs

### ASR-DATA-01: Business Rules Configuration
**Quality Attribute**: Maintainability, Flexibility
**Priority**: Medium

**Scenario**:
- **Source**: Library administrator
- **Stimulus**: Needs to change lending duration or fine values
- **Artifact**: Business rules configuration
- **Environment**: Deployment, maintenance
- **Response**: Rules can be changed without code modification
- **Response Measure**: Configuration change applied without recompilation

**Architectural Decision**:
- Externalized business rules in `library.properties` file
- Rules injected into services at runtime
- Properties include: lending duration, fine values, reader minimum age, suggestion limits

**Implementation Evidence**:
- Configuration file: [library.properties](Original/src/main/resources/config/library.properties)
  - lendingDurationInDays=15
  - fineValuePerDayInCents=200
  - minimumReaderAge=12
  - suggestionsLimitPerGenre=2

**Rationale**:
- Separates policy from implementation
- Allows business rules to change without code changes
- Different configurations per environment (dev, test, prod)
- Makes business rules visible and auditable

---

### ASR-DATA-02: Data Consistency and Validation
**Quality Attribute**: Reliability, Data Integrity
**Priority**: High

**Scenario**:
- **Source**: System
- **Stimulus**: Database operations (insert, update)
- **Artifact**: Database schema, domain model
- **Environment**: Normal operation
- **Response**: System enforces data constraints and relationships
- **Response Measure**: Zero orphaned records, all foreign key constraints enforced

**Architectural Decision**:
- JPA relationship annotations with cascading rules
- Unique constraints on natural keys (ISBN, reader number, lending number)
- Not-null constraints on required fields
- Embedded value objects for validation
- JPA schema generation from entities

**Implementation Evidence**:
- Unique constraints: [Book.java:19-21](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L19-L21)
- Not-null constraints: [Lending.java:55-58](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L55-L58)
- Schema generation: [application.properties:42-44](Original/src/main/resources/application.properties#L42-L44)

**Rationale**:
- Database-level constraints provide last line of defense
- JPA mapping ensures domain model matches database schema
- Prevents data corruption at multiple levels
- Schema generation ensures consistency between code and database

---

### ASR-DATA-03: Forbidden Names Validation
**Quality Attribute**: Data Quality, Compliance
**Priority**: Low

**Scenario**:
- **Source**: User
- **Stimulus**: Attempts to create author/reader with forbidden name
- **Artifact**: Name validation service
- **Environment**: Normal operation
- **Response**: System rejects forbidden names
- **Response Measure**: 100% forbidden names blocked

**Architectural Decision**:
- Centralized ForbiddenNameService
- Configurable forbidden names list loaded from file
- Shared across all name-containing entities

**Implementation Evidence**:
- Service: [ForbiddenNameServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/ForbiddenNameServiceImpl.java)
- Model: [ForbiddenName.java](Original/src/main/java/pt/psoft/g1/psoftg1/shared/model/ForbiddenName.java)
- Configuration: `forbiddenNames.txt`

**Rationale**:
- Enforces content policy consistently
- Configurable list allows updates without code changes
- Centralized service prevents duplication
- Demonstrates validation extensibility

---

## Architectural Decisions

### AD-01: Selection of Layered Architecture

**Decision**: Adopt a three-layer (presentation, business, data) architecture pattern.

**Context**: Educational project for software architecture course, requiring clear structure, maintainability, and demonstration of architectural principles.

**Alternatives Considered**:
1. **Microservices**: Rejected due to unnecessary complexity for project scope, operational overhead, and lack of distinct scaling requirements per bounded context.
2. **Hexagonal/Ports & Adapters**: Considered but rejected as overly complex for academic timeline and scope.
3. **Transaction Script**: Rejected as inappropriate for learning domain-driven design principles.

**Consequences**:
- **Positive**:
  - Clear separation of concerns
  - Easy to understand and navigate
  - Suitable for single-team development
  - Supports educational goals of demonstrating layered architecture
  - Enables independent testing of layers

- **Negative**:
  - Cannot scale individual layers independently
  - All components deployed as single unit
  - Changes to cross-cutting concerns affect multiple layers

**Validation**: Layered structure consistently applied across all 7 domain modules, enabling maintainability and clarity.

---

### AD-02: JWT with RSA Keys for Authentication

**Decision**: Use JWT tokens with RSA asymmetric encryption for authentication.

**Context**: RESTful API requiring stateless authentication that supports distributed deployment.

**Alternatives Considered**:
1. **Session-based authentication**: Rejected due to scalability limitations and requirement for stateless API
2. **JWT with HMAC**: Considered but RSA preferred for stronger security and key distribution
3. **OAuth2 with external provider**: Rejected as unnecessarily complex for project scope

**Consequences**:
- **Positive**:
  - Stateless authentication enables horizontal scaling
  - RSA keys provide strong security
  - JWT self-contained, reduces database queries
  - Industry-standard approach

- **Negative**:
  - Cannot revoke individual tokens (until expiration)
  - Requires key management infrastructure
  - Token size larger than session ID

**Validation**: Successfully implemented with working authentication flow, configurable key locations.

---

### AD-03: H2 Embedded Database

**Decision**: Use H2 embedded database for persistence.

**Context**: Academic project with moderate data volumes, development and testing focus.

**Alternatives Considered**:
1. **PostgreSQL**: Considered but adds deployment complexity
2. **MySQL**: Similar complexity to PostgreSQL
3. **In-memory only**: Rejected due to need for data persistence

**Consequences**:
- **Positive**:
  - Zero-configuration setup
  - Fast development iterations
  - Suitable for demonstration purposes
  - File or in-memory modes available

- **Negative**:
  - Limited scalability compared to production databases
  - Not suitable for production deployment
  - Fewer advanced features than PostgreSQL

**Validation**: Configuration supports multiple modes (TCP, file, in-memory), adequate for project requirements.

---

### AD-04: Domain-Driven Design Patterns

**Decision**: Apply DDD patterns including aggregates, value objects, repositories, and domain services.

**Context**: Complex business domain (library operations) requiring maintainable, expressive code structure.

**Alternatives Considered**:
1. **Anemic domain model**: Rejected as anti-pattern, leads to service layer bloat
2. **Transaction script**: Rejected as inappropriate for complex domain logic

**Consequences**:
- **Positive**:
  - Code structure mirrors business domain
  - Rich domain models with encapsulated behavior
  - Ubiquitous language established (glossary)
  - Better communication with domain experts
  - Educational value for learning DDD

- **Negative**:
  - Steeper learning curve for team
  - More classes/objects than anemic model
  - Requires understanding of DDD concepts

**Validation**: Comprehensive domain artifacts documented, value objects consistently used, business logic in entities.

---

### AD-05: Optimistic Locking Strategy

**Decision**: Use optimistic locking (@Version) instead of pessimistic locking for concurrency control.

**Context**: Multiple users may attempt to update same entities concurrently.

**Alternatives Considered**:
1. **Pessimistic locking**: Rejected due to performance overhead and reduced concurrency
2. **No concurrency control**: Rejected due to risk of lost updates

**Consequences**:
- **Positive**:
  - Better performance (no database locks)
  - Higher concurrency throughput
  - Aligns with stateless API design
  - Simpler implementation with JPA @Version

- **Negative**:
  - Clients must handle conflicts (retry logic)
  - May result in wasted work if conflict occurs

**Validation**: Version fields on all mutable entities, version checks in update operations.

---

## ASR Traceability Matrix

This matrix traces ASRs to implementation artifacts, architectural decisions, and test evidence.

| ASR ID | Quality Attribute | Implementation Artifacts | Architectural Decision | Test Evidence |
|--------|-------------------|--------------------------|------------------------|---------------|
| ASR-SEC-01 | Security | SecurityConfig.java, JWT beans | AD-02 | AuthApi tests |
| ASR-SEC-02 | Security | SecurityConfig authorization rules, Role.java | AD-02 | Integration tests |
| ASR-SEC-03 | Security | BCryptPasswordEncoder, Password.java | AD-02 | UserService tests |
| ASR-SEC-04 | Security | CorsFilter, SecurityFilterChain | AD-02 | Manual testing |
| ASR-PERF-01 | Performance | @Version annotations, applyPatch methods | AD-05 | Concurrency tests |
| ASR-PERF-02 | Performance | application.properties datasource config | AD-03 | Performance testing |
| ASR-PERF-03 | Performance | Page.java, SearchRequest.java | AD-01 | Repository tests |
| ASR-MAINT-01 | Maintainability | Package structure, layered modules | AD-01 | Architecture review |
| ASR-MAINT-02 | Maintainability | Domain model, value objects, aggregates | AD-04 | Domain model tests |
| ASR-MAINT-03 | Maintainability | Service interfaces, @Service/@Controller | AD-01, AD-04 | Unit tests |
| ASR-MAINT-04 | Maintainability | Validation annotations, GlobalExceptionHandler | AD-04 | Validation tests |
| ASR-USE-01 | Usability | SpringDoc OpenAPI, @Tag annotations | AD-01 | Manual API testing |
| ASR-USE-02 | Usability | REST controllers, consistent URL patterns | AD-01 | Postman collections |
| ASR-INT-01 | Interoperability | ApiNinjasService, external package | AD-01 | External API tests |
| ASR-INT-02 | Usability | FileStorageService, EntityWithPhoto | AD-01 | File upload tests |
| ASR-DATA-01 | Maintainability | library.properties, @Value injection | AD-01 | Configuration tests |
| ASR-DATA-02 | Reliability | JPA constraints, schema generation | AD-03, AD-04 | Integration tests |
| ASR-DATA-03 | Data Quality | ForbiddenNameService, validation | AD-04 | Name validation tests |

---

## Validation and Review

**Validation Method**: Reverse engineering from implemented codebase, cross-referenced with existing documentation.

**Evidence Sources**:
- Source code analysis (144 Java files)
- Configuration files (application.properties, library.properties)
- Existing documentation (Docs/GlobalArtifacts/, Phase1/, Phase2/)
- REST API mappings
- Domain model documentation

**Review Status**: ✅ Completed
**Reviewer**: Architecture analysis based on codebase reverse engineering
**Review Date**: 2025-10-29

---

## References

- [Domain Model](../GlobalArtifacts/DomainModel.puml)
- [Glossary](../GlobalArtifacts/Glossary.md)
- [REST API Mapping - Phase 2](../Phase2/RestMapping.md)
- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/)
- [Domain-Driven Design by Eric Evans](https://www.domainlanguage.com/ddd/)

---

**Document Control**
Version: 1.0
Last Updated: 2025-10-29
Status: Complete
Next Review: Upon major architectural changes
