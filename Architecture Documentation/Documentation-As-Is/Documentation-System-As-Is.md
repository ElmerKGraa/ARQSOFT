# Architecture Documentation - Library Management (System-As-Is)

This document represents the "As-Is" state of the Library Management System as implemented for the ISEP LETI 2023/2024 academic project.

**Date:** November 2025
**Project:** Library Management System
**Version:** As-Implemented Documentation
**Group**: Elmer Graça-1161424; Rafael Perez-1240486
**Class**: ARQSOFT_D
Institution: ISEP - Instituto Superior de Engenharia do Porto - Data Engineering Master

* Across the document, certains points were viewed from the perspective that this was a academic project therefore considering the constraints and requirements that come with it. Points marked with '##'.
* Some images were inconvenient to show in markdown format for which the respective image is referenced following structure:
```             
---> \Views\Context.png                                       

```
---

## 1. Introduction and Objectives

The Library Management System is a backend REST API designed to manage Central City Library's operations, including book cataloging, reader registration, and lending management for thousands of books organized by genre with core requirements of tracking key business metrics including the top readers, books, and genres, as well as monthly average lending per reader and genre, and average lending duration across multiple dimensions.

The system is a RESTful API backend for managing library operations including authors, books, readers, and lending transactions.

### 1.1 Requirements Overview

**Core Business Requirements:**

- **Book Management:** Register and manage books with ISBN, title, genre, description, author, and cover photos
- **Author Management:** Track authors with biographical information, photos, and co-author relationships
- **Reader Management:** Self-service registration with personal data and GDPR consent, profile management, and interest tracking and recommendation
- **Lending Operations:** Book lending workflow with 15-day default duration, overdue fine calculation (per delay day), and business rules (max 3 simultaneous lendings, no lending with overdue books)
- **Reporting & Analytics:**
  - Top rankings (readers, authors, books lent, genres)
  - Monthly lending statistics per reader and per genre
  - Average lending duration metrics per book, reader and genre
  - Last 12 months lending trends by genre
  - Overdue lending tracking

**Key Functional Features:**

- JWT-based authentication with role-based access control (Reader, Librarian, Admin)
- Photo upload support for books, authors, and readers (max 20KB, PNG/JPEG)
- Advanced search capabilities for books, authors and readers (by title, genre, author, phone, email)
- Book suggestions based on reader interests
- External API integration (API Ninjas for historical events/funny quotes)
- Pagination support for long result lists
- Concurrent access control
- API documentation via OpenAPI 3.0 and Swagger UI
- Automated tests with POSTMAN collections
- Frontend Flexibility Requirement: Backend-only system with REST API to support any future frontend technology

**Functional Requirments:**

1. **Librarian**
1.1. As a librarian, I want to manage authors (create, update, view, search by name) so that the library maintains accurate and searchable author records.
1.2. As a librarian, I want to manage books (create, update, view by ISBN, search by genre/title) so that the catalog stays correct and easy to browse.
1.3. As a librarian, I want to manage readers (view by reader number, search by name/phone/email) so that I can quickly locate user profiles.
1.4. As a librarian, I want to manage lendings (create/lend, view by lending number, list overdue ordered by lateness) enforcing rules (no overdue, max 3 books) so that lending complies with library policies.
1.5. As a librarian, I want to enrich resources (authors with optional photo, books with cover) so that the system can present richer information.
1.6. As a librarian, I want analytical views (Top N books, Top N authors, Top N genres, Top N readers, stats per month/genre/reader, average lending duration) so that I can monitor usage and plan the collection.

2. **Reader**
2.1. As a reader, I want to manage my account (register, update personal data, optional photo, optional interests) so that my profile is complete and personalized.
2.2. As a reader, I want to search and discover books (by title, author, genre, by an author’s catalog, by co-authors) so that I can find material relevant to me.
2.3. As a reader, I want to receive recommendations based on my interests so that I get personalized suggestions.
2.4. As a reader, I want to return borrowed books so that I can complete the lending cycle and avoid fines.
2.5. As a reader, I want to view lending details (my loans or a given lending number) so that I know due dates and history.

3. **Librarian or Reader**
3.1. As a librarian or reader, I want to consult details of core entities (author, book, lending) by their identifiers so that I can access complete information when needed.

4. **Guest (Anonymous)**
4.1. Register as a reader (name, email, date of birth, phone number, GDPR consent) with optional photo and optional list of interests (genres) so that I can access the library’s lending services, receive a Reader Number, and have a personalized profile.

### 1.2 Quality Goals

| Priority | Quality Attribute | Description |
|----------|------------------|-------------|
| 1 | **Security & Compliance** | JWT-based authentication, role-based authorization, XSS protection via HTML sanitization, GDPR compliance, BCrypt password hashing |
| 2 | **Data Integrity** | CConcurrent access handling, ETag support, transaction management, forbidden name validation |
| 3 | **Maintainability** | Layered architecture with clear separation of concerns with DDD design, extensive use of Value Objects for type safety |
| 4 | **Interoperability** | RESTful API with HATEOAS links, OpenAPI 3.0 specification, Swagger UI documentation |
| 5 | **Performance** | File-based H2 database, JPA query optimization (JPQL, native SQL, Criteria API), pagination for large datasets |
| 6 | **Testability** | Comprehensive unit tests, POSTMAN collections for API testing, bootstrap data profiles |

### 1.3 Stakeholders

| Role | Expectations | Concerns |
|------|-------------|----------|
| **Librarian** | Complete CRUD operations on books/authors, lending management, comprehensive reporting, overdue tracking | System reliability, data accuracy, ease of use |
| **Reader** | Self-service registration, book search and suggestions, lending/return operations, profile management | Privacy (GDPR), user experience, data security |
| **System Administrators** | System monitoring, user management, database maintenance | Security, performance, backup/recovery |
| **Development Team** | Clear architecture, maintainable code, comprehensive documentation | Technical debt management, testing coverage |

---

## 2. Constraints

### 2.1 Technical Constraints

| Constraint | Description |
|------------|-------------|
| **TC-1: Java 17** | JDK 17 required for Spring Boot 3.2.5 compatibility |
| **TC-2: Spring Boot 3.2.5** | Framework version locked by project dependencies |
| **TC-3: H2 Database** | Embedded file-based H2 database (`./data/psoft-g1`) |
| **TC-4: Maven Build** | Maven as build and dependency management tool |
| **TC-5: JWT Authentication** | RSA-signed JWT tokens (rsa.private.key, rsa.public.key) |
| **TC-6: REST API Only** | Backend-only system, no frontend. Must support any future frontend technology |
| **TC-7: File Storage** | Local filesystem storage |

### 2.2 Organizational Constraints

##*Considering the original project statement provided*

| Constraint | Description |
|------------|-------------|
| **OC-1: Academic Project** | Developed as university coursework (LETI - 2023/2024) - Small team (3-4 students) with individual work package assignments|
| **OC-2: Iterative Development** | Two-phase development (Phase 1: CRUD, Phase 2: Advanced features) |
| **OC-3: Documentation Requirements** | Must include domain model, sequence diagrams, design justifications, unit tests |
| **OC-4: Code Attribution** | Third-party code must be clearly marked with source attribution |

### 2.3 Conventions

| Convention | Description |
|------------|-------------|
| **CV-1: OpenAPI Specification** | API must be documented using OpenAPI 3.0 with Swagger UI |
| **CV-2: POSTMAN Collections** | Sample requests and automated tests provided via POSTMAN |
| **CV-3: DDD Principles** | Domain-Driven Design with Aggregates, Entities, Value Objects |
| **CV-4: REST Principles** | RESTful API design with proper HTTP methods and status codes |

---

## 3. Context and Scope

Libraries face the operational challenge of managing thousands of books while maintaining comprehensive control over reader registrations, lending transactions and other various operations. Beyond basic transaction management, the library must employ data-driven strategies to enhance reader engagement and optimize collection management—including personalized book recommendations based on reader interests and robust analytics to inform future planning decisions. 
To achieve operational efficiency and support strategic planning, the system must systematically collect, track, and analyze critical business metrics such as lending patterns by genre and reader, borrowing duration trends, overdue management, and popularity indicators for books and authors. Hence the necessity of this system that not only streamlines daily operations but also empowers the library with actionable insights derived from its operations and data.

```             
---> Documentation-As-Is\Views\Context.png                                       

```

**Technical Protocols & Formats:**

- **API Protocol:** REST over HTTP/HTTPS
- **Data Format:** JSON (application/json)
- **Authentication:** JWT (RS256 algorithm with RSA keys)
- **File Upload:** Multipart form-data (max 200MB; photos limited to 20KB)
- **Database Protocol:** JDBC (H2 database driver)
- **API Documentation:** OpenAPI 3.0

## 4. Solution Strategy

The system employs a three-layer architecture pattern—Presentation (API/Controllers), Business (Service), and Data (Repository)—to achieve decoupling, maintainability and controlled modifiability. This approach establishes clear separation of concerns where each layer has distinct responsibilities, enabling developers to modify business logic or database implementations without propagating changes across unrelated components. By enforcing unidirectional dependencies that flow downward from API through Service to Repository, the architecture prevents circular dependencies and hidden coupling, ensuring that modifications remain localized to specific layers rather than cascading throughout the entire system. This well-understood pattern facilitates independent layer evolution (such as replacing the H2 database with PostgreSQL), and enables comprehensive testing through mock-based verification at layer boundaries. The consistent package structure replicated across all domain modules—author management, book management, lending management, reader management, and genre management—ensures architectural homogeneity and predictable code organization that developers can quickly navigate.

```
┌────────────────────────────────────────────────────────────┐
│                    Client Layer                            │
│             Future Frontend(External System)               │
└──────────────────────────┬─────────────────────────────────┘
                           │ HTTPS (Port 8080)
                           │ Content-Type: application/json
                           │ Authorization: Role-based <JWT>
┌──────────────────────────▼─────────────────────────────────┐
│          Presentation Layer - Controllers                  │
│  (@RestController, @RequestMapping)                        │
│  - AuthorController   - BookController                     │
│  - ReaderController   - LendingController                  │
└──────────────────────────┬─────────────────────────────────┘
                           │ DTOs (Request/Response)
┌──────────────────────────▼─────────────────────────────────┐
│              Service Layer                                 │
│  (@Service, @Transactional)                                │
│  - Business logic validation                               │
│  - Transaction management                                  │
│  - MapStruct DTO transformations                           │
└──────────────────────────┬─────────────────────────────────┘
                           │ Domain Objects
┌──────────────────────────▼─────────────────────────────────┐
│            Repository Layer                                │
│  (Repository Pattern)                                      │
│  - Domain interfaces (model layer)                         │
│  - Spring Data implementations (infrastructure)            │
└──────────────────────────┬─────────────────────────────────┘
                           │ JPA/Hibernate
┌──────────────────────────▼─────────────────────────────────┐
│                     H2 Database                            │
│                                                            │
└────────────────────────────────────────────────────────────┘

```

**Characteristics:**
1. **Vertical Slicing:** Each business capability (author, book, lending, reader) is organized in its own module
2. **Layered Structure:** Within each module, clear separation between API, Service, Domain, and Repository layers
3. **Dependency Direction:** Dependencies flow downward (API → Service → Domain → Repository)
4. **Shared Kernel:** Common functionality in `shared` package
5. **Infrastructure Separation:** Configuration and cross-cutting concerns isolated

**Benefits:**
- High cohesion within modules
- Clear separation of concerns
- Easy to understand and navigate
- Supports team-based development
- High testability

**Negatives:**
-Lack of persistence of data
- Some duplication across modules
- Cross-module dependencies require careful management

*These drawbacks will be addressed with ADD—for example, we’ll remove duplication from hard-coded statistics by introducing a shared service for books, genres, and authors.*

### 4.1 Technology Decisions

| Decision | Rationale |
|----------|-----------|
| **Spring Boot 3.2.5** | Industry-standard framework for enterprise Java applications, comprehensive ecosystem, auto-configuration, seamless SQL code conversion, production-ready features |
| **Layered Architecture + DDD** | Clear separation of concerns, maintainability, testability, aligns with software engineering best practices |
| **H2 Database** | Embedded database suitable for prototyping, easy setup, no external dependencies (## sufficient for academic prototype project scale) |
| **JWT with RSA** | Stateless authentication suitable for REST APIs, scalable, industry standard, supports role-based access control |
| **MapStruct** | Type-safe DTO-Entity mapping at compile-time, reduces boilerplate, better performance than reflection-based mappers |
| **Lombok** | Reduces boilerplate code (getters/setters/constructors), improves code readability, widely adopted in Spring ecosystem |
| **Value Objects** | Strong typing, encapsulated validation, domain modeling best practice |

### 4.2 Top-Level Decomposition

**Bounded Contexts (DDD-inspired):**

The system is organized around domain aggregates—Author, Book, Lending, and Reader—with rich domain models that encapsulate business logic directly within entities rather than scattering it across utility classes or anemic models. By representing domain concepts as Value Objects (such as ISBN, Title, Name, and Email), the architecture creates self-documenting code where domain intent is explicit and type-safe, reducing misunderstandings and implementation errors. The Repository pattern abstracts data access concerns, allowing domain entities to focus on business behavior while persistence mechanisms remain decoupled and independently testable. Bounded contexts established around each domain module create clear conceptual boundaries that align with organizational responsibility structures, enabling new developers to quickly understand the system's functional areas and locate relevant code without extensive documentation—typically achieving productivity within two days. This domain-centric organization ensures that as the library's business requirements evolve, the codebase remains intuitive and extensible, with new features naturally mapping to domain concepts rather than requiring architectural reorientation.

1. **Author Management** - Author lifecycle, co-author relationships, author analytics
2. **Book Management** - Book catalog, genre classification, book search, recommendations , book analytics
3. **Reader Management** - Reader registration, profile management, interest tracking, reader analytics
4. **Lending Management** - Lending operations, fine calculation, overdue tracking, lending analytics
5. **Genre Management** - Genre catalog, genre-based analytics
6. **User Management** - Authentication, authorization, user lifecycle
7. **Shared** - Cross-cutting utilities (file storage, validation, concurrency, exception handling)

**Package Structure Pattern:**

```
{module}management/
├── api/              # REST controllers, DTOs, ViewMappers
├── model/            # Domain entities, value objects
├── services/         # Business logic, service interfaces
├── repositories/     # Repository interfaces (domain)
└── infrastructure/   # Repository implementations (Spring Data)
```

### 4.3 Key Quality Goals Achievement

| Quality Goal | Approach |
|--------------|----------|
| **Security** | Spring Security with JWT, BCrypt password hashing, role-based access control, HTML sanitization (OWASP), GDPR consent tracking |
| **Data Integrity** | JPA `@Version` for optimistic locking, transaction management (`@Transactional`), constraint validation |
| **Maintainability** | Clean architecture with layers, DDD patterns, extensive use of Value Objects, MapStruct for transformations, comprehensive documentation |
| **Interoperability** | RESTful API design, OpenAPI specification, HATEOAS links, standard HTTP methods and status codes |
| **Performance** | JPA query optimization (JPQL, native SQL, Criteria API), pagination support, file-based database, lazy loading strategies |
| **Testability** | Dependency injection, interface-based design, bootstrap data profiles, unit tests, POSTMAN test collections |

### 4.4 Organizational Decisions

'##'**Development Approach:** (Considering academic context)
- Two-phase iterative development (Phase 1: CRUD, Phase 2: Advanced features)
- Work package division among team members
- Documentation-driven development (analysis → design → implementation → testing)

---

## 5. Building Block View

The system is constitute by main **7 domain modules** (plus Shared utilities) as depicted below:

```
├── AuthorManagement
├── BookManagement
├── ReaderManagement
├── LendingManagement
├── GenreManagement
├── UserManagement
└── Auth
```

Each module follows the same generic internal structure (3 layers) variying only in the specific domain entities, value objects, DTOs, and business logic. As follow the example of the **Author Management** module:
```
authormanagement/
├── api/              ← Presentation Layer
│   ├── AuthorController.java
│   ├── AuthorView.java
│   └── AuthorViewMapper.java
├── services/         ← Business Layer
│   ├── AuthorService.java (interface)
│   └── AuthorServiceImpl.java
├── repositories/     ← Data Layer
│   └── AuthorRepository.java
└── model/            ← Domain Model (shared by all layers)
    ├── Author.java
    └── Bio.java
```

### 5.1 Level 1 - System Context

**Whitebox: Library Management System**

```
---> Documentation-As-Is\Views\Logical View\L1\System.png

```

**Contained Building Blocks:**

| Building Block | Responsibility |
|----------------|----------------|
| **Author Management** | Author CRUD operations, co-author relationship tracking, top 5 authors by lending count, author search |
| **Book Management** | Book CRUD operations, genre assignment, multi-author support, book search (title, genre, author), top 5 books, book suggestions, average lending duration per book |
| **Reader Management** | Self-service registration with GDPR consent, profile management, interest tracking, top 5 readers, reader search (name, email, phone) |
| **Lending Management** | Lending creation with business rules validation, book return with fine calculation, overdue tracking, average lending duration metrics |
| **Genre Management** | Genre catalog, top 5 genres, lending statistics per genre, average lending duration per genre |
| **User Management** | Authentication (JWT generation), authorization (role-based), user lifecycle, password management (BCrypt) |
| **Shared Utilities** | File storage, photo management, concurrency control (ETag), forbidden name validation, HTML sanitization |

### 5.2 Level 2 - Module Internal Structure

**Whitebox: Book Management Module**

```
---> Documentation-As-Is\Views\Logical View\L2\System.png

```

**Similar structure applies to all other management modules.**

### 5.3 Level 3 - Domain Model (Core Entities)

**Key Aggregates and Entities:**

```
---> Documentation-As-Is\Views\Logical View\L3\System.png

```

---

## 6. Runtime View

### 6.1 Create Lending (Business-Critical Scenario)

**Scenario:** Librarian lends a book to a reader with business rule validation

```
┌──────────┐   ┌──────────────┐   ┌─────────────┐   ┌──────────┐   ┌──────────┐
│Librarian │   │ Lending      │   │  Lending    │   │ Reader   │   │  Book    │
│  (User)  │   │ Controller   │   │  Service    │   │  Repo    │   │  Repo    │
└────┬─────┘   └──────┬───────┘   └──────┬──────┘   └────┬─────┘   └────┬─────┘
     │                │                   │                │              │
     │ POST /lendings │                   │                │              │
     │ (JWT, bookISBN,│                   │                │              │
     │  readerNumber) │                   │                │              │
     ├───────────────>│                   │                │              │
     │                │                   │                │              │
     │                │ createLending()   │                │              │
     │                ├──────────────────>│                │              │
     │                │                   │                │              │
     │                │                   │ findByReaderNumber()          │
     │                │                   ├───────────────>│              │
     │                │                   │ ReaderDetails  │              │
     │                │                   │<───────────────┤              │
     │                │                   │                │              │
     │                │                   │ findByIsbn()                  │
     │                │                   ├──────────────────────────────>│
     │                │                   │            Book                │
     │                │                   │<──────────────────────────────┤
     │                │                   │                │              │
     │                │                   │ validateBusinessRules()       │
     │                │                   │ - Check overdue books         │
     │                │                   │ - Check max 3 lendings        │
     │                │                   │ - Calculate limitDate         │
     │                │                   │                │              │
     │                │                   │ save(Lending)  │              │
     │                │                   ├───────────────>│              │
     │                │                   │ Lending        │              │
     │                │                   │<───────────────┤              │
     │                │                   │                │              │
     │                │ LendingView       │                │              │
     │                │<──────────────────┤                │              │
     │                │                   │                │              │
     │ 201 Created    │                   │                │              │
     │ (LendingView + │                   │                │              │
     │  HATEOAS links)│                   │                │              │
     │<───────────────┤                   │                │              │
```

**Business Rules Validated:**
1. Reader must not have overdue books
2. Reader can have at most 3 simultaneous lendings
3. Book must exist and be available
4. Limit date calculated as startDate + lendingDurationInDays (from config)

### 6.2 Return Book with Fine Calculation

```
┌──────────┐   ┌──────────────┐   ┌─────────────┐   ┌──────────┐
│ Reader   │   │  Lending     │   │  Lending    │   │ Lending  │
│  (User)  │   │  Controller  │   │  Service    │   │   Repo   │
└────┬─────┘   └──────┬───────┘   └──────┬──────┘   └────┬─────┘
     │                │                   │                │
     │ PATCH /lendings│                   │                │
     │ /{year}/{seq}  │                   │                │
     │ (commentary)   │                   │                │
     ├───────────────>│                   │                │
     │                │                   │                │
     │                │ returnBook()      │                │
     │                ├──────────────────>│                │
     │                │                   │                │
     │                │                   │ findByLendingNumber()
     │                │                   ├───────────────>│
     │                │                   │ Lending        │
     │                │                   │<───────────────┤
     │                │                   │                │
     │                │                   │ Authorization check
     │                │                   │ (owner only)   │
     │                │                   │                │
     │                │                   │ lending.setReturned(today)
     │                │                   │ - Calculate daysDelayed
     │                │                   │ - Calculate fine
     │                │                   │   (daysDelayed * finePerDay)
     │                │                   │                │
     │                │                   │ save(Lending)  │
     │                │                   ├───────────────>│
     │                │                   │                │
     │                │ LendingView       │                │
     │                │ (with fine info)  │                │
     │                │<──────────────────┤                │
     │                │                   │                │
     │ 200 OK         │                   │                │
     │ (Fine amount   │                   │                │
     │  displayed)    │                   │                │
     │<───────────────┤                   │                │
```

### 6.3 JWT Authentication Flow

```
┌──────────┐   ┌──────────────┐   ┌─────────────┐   ┌──────────┐
│  Client  │   │   Security   │   │    Token     │   │   User   │
│          │   │   Filter     │   │   Service    │   │   Repo   │
└────┬─────┘   └──────┬───────┘   └──────┬───────┘   └────┬─────┘
     │                │                   │                │
     │ POST /login    │                   │                │
     │ (username, pwd)│                   │                │
     ├───────────────>│                   │                │
     │                │                   │                │
     │                │ authenticate()    │                │
     │                ├──────────────────>│                │
     │                │                   │                │
     │                │                   │ findByUsername()
     │                │                   ├───────────────>│
     │                │                   │ User           │
     │                │                   │<───────────────┤
     │                │                   │                │
     │                │                   │ BCrypt.verify(password)
     │                │                   │                │
     │                │                   │ generateJWT()  │
     │                │                   │ - Sign with RSA private key
     │                │                   │ - Include roles in claims
     │                │                   │                │
     │                │ JWT token         │                │
     │                │<──────────────────┤                │
     │                │                   │                │
     │ 200 OK + JWT   │                   │                │
     │<───────────────┤                   │                │
     │                │                   │                │
     │ Subsequent API │                   │                │
     │ calls with     │                   │                │
     │ Authorization: │                   │                │
     │ Bearer <JWT>   │                   │                │
     ├───────────────>│                   │                │
     │                │                   │                │
     │                │ verifyJWT()       │                │
     │                │ (RSA public key)  │                │
     │                ├──────────────────>│                │
     │                │                   │                │
     │                │ Security Context  │                │
     │                │ (with roles)      │                │
     │                │<──────────────────┤                │
     │                │                   │                │
     │                │ @PreAuthorize     │                │
     │                │ role check        │                │
```

---

## 7. Deployment View

### 7.1 Infrastructure Level 1

**Single-Instance Deployment (Current "As-Is" State)**

```
┌────────────────────────────────────────────────────────────┐
│              Development/Local Machine                      │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │         JVM (Java 17)                                 │ │
│  │                                                       │ │
│  │  ┌─────────────────────────────────────────────┐    │ │
│  │  │   Spring Boot Application (port 8080)       │    │ │
│  │  │                                              │    │ │
│  │  │  - Embedded Tomcat                          │    │ │
│  │  │  - Spring MVC Controllers                   │    │ │
│  │  │  - Service Layer                            │    │ │
│  │  │  - JPA/Hibernate                            │    │ │
│  │  │  - Spring Security                          │    │ │
│  │  └─────────────────────────────────────────────┘    │ │
│  │                                                       │ │
│  │  ┌─────────────────────────────────────────────┐    │ │
│  │  │   H2 Database Engine (embedded)             │    │ │
│  │  └─────────────────────────────────────────────┘    │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │         File System                                   │ │
│  │                                                       │ │
│  │  ./data/psoft-g1.*        (Database files)           │ │
│  │  ./uploads-psoft-g1/      (Photo storage)            │ │
│  │  ./src/main/resources/    (RSA keys, config)         │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                            │
│  ┌──────────────────────────────────────────────────────┐ │
│  │         Network                                       │ │
│  │                                                       │ │
│  │  localhost:8080          (REST API)                  │ │
│  │  localhost:8080/swagger-ui (Swagger UI)              │ │
│  │  localhost:8080/h2-console (H2 Console)              │ │
│  └──────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────┘
                         │
                         │ HTTPS
                         ▼
         ┌───────────────────────────────┐
         │   External: API Ninjas        │
         │   (Historical Events API)     │
         └───────────────────────────────┘
```

**Mapping to Infrastructure:**

| Node | Artifacts | Configuration |
|------|-----------|---------------|
| **JVM** | psoft-g1-0.0.1-SNAPSHOT.jar | Java 17, heap size defaults |
| **H2 Database** | psoft-g1.mv.db, psoft-g1.trace.db | File-based, `./data/` directory, AUTO_SERVER=TRUE |
| **File Storage** | Photo files (PNG/JPEG) | `./uploads-psoft-g1/`, max 20KB per file |
| **Configuration** | application.properties, RSA keys | `src/main/resources/` |

### 7.2 Technology Stack per Layer

| Layer | Technologies | Deployment Unit |
|-------|-------------|-----------------|
| **API Layer** | Spring MVC 6.x, Spring Security, Springdoc OpenAPI | Embedded Tomcat (Spring Boot) |
| **Service Layer** | Spring Framework, MapStruct 1.5.5, Jakarta Validation | Same JAR |
| **Persistence Layer** | Spring Data JPA, Hibernate 6.x, H2 JDBC Driver | Same JAR + H2 files |
| **Security** | Spring Security OAuth2 Resource Server, JWT (RSA) | RSA key files |
| **Monitoring** | Spring Boot Actuator, H2 Console | Exposed endpoints |

---

## 8. Cross-Cutting Concepts

┌────────────────────────────────────────────────────────────┐
│          Cross-Cutting Concerns                            │
│  - Spring Security (JWT, BCrypt, OAuth2)                   │
│  - JPA Auditing (createdAt, modifiedAt, etc.)              │
│  - OpenAPI/Swagger (API documentation)                     │
│  - File Storage Service (photo management)                 │
│  - Exception Handling (GlobalExceptionHandler)             │
└────────────────────────────────────────────────────────────┘

### 8.1 Domain Model Patterns

**Value Objects:**

All primitive types are wrapped in Value Objects to enforce validation and provide type safety:

```
Isbn, Title, Description          (Book domain)
Name, Bio                         (Author domain)
EmailAddress, Password            (User domain)
ReaderNumber, BirthDate, PhoneNumber  (Reader domain)
LendingNumber                     (Lending domain)
```

**Benefits:**
- Encapsulated validation logic
- Type safety (cannot accidentally assign ISBN to Title)
- Self-documenting code
- Consistent validation across the application

### 8.2 Security Concepts

**Authentication:**
- JWT-based (RS256 algorithm with RSA 2048-bit keys)
- Stateless sessions
- Token expiration configured
- Refresh token strategy not implemented (academic scope)

**Authorization:**
- Role-based access control (READER, LIBRARIAN, ADMIN)
- Method-level security with `@PreAuthorize`
- Resource-level authorization (readers can only access own data)

**Password Security:**
- BCrypt hashing (strength 12)
- Password value object enforces complexity rules
- No plain-text password storage

**Data Protection:**
- GDPR consent mandatory for reader registration
- HTML sanitization (OWASP library) for XSS prevention
- Forbidden name validation (loaded from forbiddenNames.txt)
- Photo file size restrictions

### 8.3 Persistence & Transaction Management

**Repository Pattern:**
- Domain interfaces defined in model layer
- Infrastructure implementations in infrastructure layer
- Separation allows domain logic independence from persistence technology

**Transaction Boundaries:**
- `@Transactional` at service layer
- Read-only transactions for queries
- Propagation defaults (REQUIRED)

**Optimistic Locking:**
- `@Version` field on all aggregates
- ETag support in REST API (If-Match headers)
- Prevents lost updates in concurrent scenarios

**Auditing:**
- JPA Auditing enabled
- Tracks: createdAt, modifiedAt, createdBy, modifiedBy
- Auditor extracted from Spring Security context

### 8.4 Exception Handling

**Global Exception Handler:**
- `@RestControllerAdvice` for centralized error handling
- Custom exceptions:
  - `NotFoundException` → 404
  - `ConflictException` → 409
  - `LendingForbiddenException` → 403
  - `ValidationException` → 400
- Consistent error response format

### 8.5 DTO Mapping Strategy

**MapStruct Mappers:**
- Compile-time code generation
- Two categories:
  - **ViewMappers** (Entity → View DTO)
  - **ServiceMappers** (Request DTO → Entity)
- Custom mapping methods for complex transformations
- Integration with Lombok via lombok-mapstruct-binding

### 8.6 API Design Principles

**REST Maturity:**
- Level 2: HTTP verbs and status codes
- Level 3: HATEOAS (hypermedia links in responses)

**Conventions:**
- Resource URLs: `/api/{resource}` (plural)
- Natural keys in URLs: `/api/books/{isbn}`, `/api/lendings/{year}/{seq}`
- Query parameters for filtering: `?genre=Fiction&title=Harry`
- Request bodies for complex searches: `POST /api/books/search`

**Pagination:**
- Page number and size parameters
- Response includes totalElements, totalPages, current page

**File Upload:**
- Multipart form-data
- Separate endpoints: `POST /{id}/photo`, `DELETE /{id}/photo`

### 8.7 Configuration Management

**Profiles:**
- `bootstrap` - Pre-loads sample data for development/testing
- Configuration externalized in `application.properties`
- Library-specific config in `config/library.properties`

**Key Configurations:**
```
lendingDurationInDays=15
fineValuePerDayInCents=50
file.photo_max_size=20000
file.upload-dir=uploads-psoft-g1
```

### 8.8 Testing Strategy

**Unit Tests:**
- JUnit 5
- Service layer tests with mocked repositories
- Value object validation tests

**API Tests:**
- POSTMAN collections
- Sample requests for all endpoints
- Automated test scripts

**Integration Tests:**
- Spring Boot Test
- `@SpringBootTest` with embedded H2
- Bootstrap profile for test data

---

## 9. Architecture Decisions

### 9.1 ADR-001: Layered Architecture with DDD Influence

**Characteristics:**
1. **Vertical Slicing:** Each business capability (author, book, lending, reader) is organized in its own module
2. **Layered Structure:** Within each module, clear separation between API, Service, Domain, and Repository layers
3. **Dependency Direction:** Dependencies flow downward (API → Service → Domain → Repository)
4. **Shared Kernel:** Common functionality in `shared` package
5. **Infrastructure Separation:** Configuration and cross-cutting concerns isolated

**Benefits:**
- High cohesion within modules
- Clear separation of concerns
- Easy to understand and navigate
- Supports team-based development

**Drawbacks:**
- Some duplication across modules
- Potential for service layer to become anemic
- Cross-module dependencies require careful management

**Decision:**
Implement a layered architecture with Domain-Driven Design principles:
- API Layer (Controllers, DTOs)
- Application Layer (Services)
- Domain Layer (Entities, Value Objects, Repository Interfaces)
- Infrastructure Layer (Repository Implementations)

**Consequences:**
- **Positive:** Clear separation of concerns, testability, maintainability, aligns with academic requirements
- **Negative:** More complex initial setup, requires understanding of DDD concepts
- **Risks:** Team members need training on DDD principles

### 9.2 ADR-002: Repository Interfaces in Domain Layer

**Status:** Accepted

**Context:**
Need to decouple domain logic from persistence infrastructure.

**Decision:**
Define repository interfaces in the domain layer (`repositories/`) and implement them in the infrastructure layer (`infrastructure/repositories/impl/`) using Spring Data JPA.

**Consequences:**
- **Positive:** Domain logic independent of persistence technology, easier to test, follows DDD
- **Negative:** Additional abstraction layer, Spring Data repositories must implement both interfaces
- **Rationale:** Supports Dependency Inversion Principle, enables future technology changes

### 9.3 ADR-003: Value Objects for All Domain Primitives

**Status:** Accepted

**Context:**
Need to enforce validation consistently and avoid primitive obsession anti-pattern.

**Decision:**
Wrap all domain primitives (strings, numbers, dates) in Value Objects with embedded validation logic.

**Examples:** `Isbn`, `Title`, `EmailAddress`, `PhoneNumber`, `ReaderNumber`, `LendingNumber`

**Consequences:**
- **Positive:** Type safety, encapsulated validation, self-documenting code, consistent validation
- **Negative:** More classes to maintain, initial learning curve
- **Trade-offs:** Verbosity vs. robustness (robustness wins for academic/enterprise context)

### 9.4 ADR-004: JWT with RSA Signing

**Status:** Accepted

**Context:**
Need stateless authentication suitable for REST APIs with potential future microservices architecture.

**Decision:**
Use JWT with RS256 algorithm (RSA public/private key pair) instead of HMAC (symmetric key).

**Rationale:**
- Public key can be shared with other services for token verification
- Private key remains secure on authentication service
- Industry standard for distributed systems

**Consequences:**
- **Positive:** Scalable, supports future microservices, standard approach
- **Negative:** RSA key management required, slightly slower than HMAC
- **Implementation:** Keys stored in `src/main/resources/rsa.private.key` and `rsa.public.key`

### 9.5 ADR-005: H2 Embedded Database

**Status:** Accepted (for prototype phase)

**Context:**
Need rapid development with minimal infrastructure setup for academic project.

**Decision:**
Use H2 embedded file-based database instead of external database (PostgreSQL, MySQL).

**Consequences:**
- **Positive:** Zero configuration, portable, suitable for development and testing
- **Negative:** Not production-ready, single-instance limitation, no advanced features
- **Migration Path:** Easy migration to PostgreSQL/MySQL (JPA abstraction layer)
- **Risks:** File corruption, performance limitations with large datasets

### 9.6 ADR-006: MapStruct for DTO Mapping

**Status:** Accepted

**Context:**
Need efficient DTO-Entity transformation with type safety.

**Decision:**
Use MapStruct (compile-time code generation) instead of ModelMapper or manual mapping.

**Rationale:**
- Compile-time safety (errors caught early)
- Better performance than reflection-based mappers
- Integration with Lombok
- Industry adoption

**Consequences:**
- **Positive:** Type-safe, performant, generates clean code
- **Negative:** Requires annotation processor setup, learning curve
- **Alternatives Rejected:** ModelMapper (runtime reflection), manual mapping (boilerplate)

### 9.7 ADR-007: Natural Keys in REST URLs

**Status:** Accepted

**Context:**
Need user-friendly, meaningful URLs for REST API.

**Decision:**
Use natural keys (business identifiers) in URLs instead of internal database IDs:
- `/api/books/{isbn}` instead of `/api/books/123`
- `/api/readers/{year}/{seq}` instead of `/api/readers/456`
- `/api/lendings/{year}/{seq}` instead of `/api/lendings/789`

**Consequences:**
- **Positive:** User-friendly, meaningful URLs, business-focused API design
- **Negative:** Requires unique natural keys, URL encoding for special characters
- **Trade-offs:** Simplicity vs. technical purity (simplicity wins for user experience)

### 9.8 ADR-008: Optimistic Locking Strategy

**Status:** Accepted

**Context:**
Need to handle concurrent updates without database-level locking overhead.

**Decision:**
Use JPA `@Version` fields with ETag support in REST API (If-Match headers).

**Consequences:**
- **Positive:** No database locks, scalable, supports HTTP caching strategies
- **Negative:** Clients must handle 409 Conflict responses, retry logic required
- **Implementation:** All aggregate roots have `@Version` field

### 9.9 ADR-009: Bootstrap Data via Spring Profiles

**Status:** Accepted

**Context:**
Need sample data for development, testing, and demonstration.

**Decision:**
Use Spring Profiles (`@Profile("bootstrap")`) to conditionally load sample data at startup.

**Consequences:**
- **Positive:** Repeatable data setup, clean separation from production code
- **Negative:** Bootstrap code must be maintained, risk of accidentally running in production
- **Safeguards:** Profile activation explicit in `application.properties`

### 9.10 ADR-010: Local File Storage for Photos

**Status:** Accepted (with limitations)

**Context:**
Need simple file storage solution for photo upload feature in academic prototype.

**Decision:**
Use local filesystem (`uploads-psoft-g1/`) instead of cloud storage (S3, Azure Blob).

**Consequences:**
- **Positive:** Simple implementation, no external dependencies, no costs
- **Negative:** Not cloud-ready, requires shared filesystem for clustering, backup challenges
- **Migration Path:** Abstract file storage behind `FileStorageService` interface for future cloud migration
- **Risks:** File loss if directory not backed up, not suitable for production

---

## 10. Quality Requirements

### 10.1 Quality Tree

```
Quality
├── Security (High)
│   ├── Authentication (JWT, BCrypt)
│   ├── Authorization (Role-based)
│   ├── XSS Prevention (HTML sanitization)
│   └── GDPR Compliance (consent tracking)
│
├── Data Integrity (High)
│   ├── Optimistic Locking (@Version)
│   ├── Transaction Management
│   ├── Constraint Validation (Jakarta Validation)
│   └── Business Rule Enforcement
│
├── Maintainability (High)
│   ├── Layered Architecture
│   ├── DDD Patterns
│   ├── Value Objects
│   └── Comprehensive Documentation
│
├── Interoperability (Medium)
│   ├── RESTful API Design
│   ├── OpenAPI Specification
│   ├── HATEOAS Links
│   └── Standard HTTP Methods
│
├── Performance (Medium)
│   ├── JPA Query Optimization
│   ├── Pagination
│   ├── Lazy Loading
│   └── File-based Database
│
├── Testability (Medium)
│   ├── Dependency Injection
│   ├── Interface-based Design
│   ├── Bootstrap Profiles
│   └── POSTMAN Collections
│
└── Usability (Low - Backend API)
    ├── Swagger UI
    ├── Clear Error Messages
    └── HATEOAS Discoverability
```

### 10.2 Quality Scenarios

#### 10.2.1 Security Scenarios

**QS-SEC-01: Unauthorized Access Prevention**
- **Stimulus:** Unauthenticated user attempts to access protected endpoint
- **Response:** System returns 401 Unauthorized with error message
- **Measure:** 100% of protected endpoints require valid JWT token

**QS-SEC-02: Role-Based Access Control**
- **Stimulus:** Reader attempts to access Librarian-only endpoint (e.g., create lending)
- **Response:** System returns 403 Forbidden
- **Measure:** All endpoints enforce correct role requirements

**QS-SEC-03: Password Security**
- **Stimulus:** User registers with weak password
- **Response:** System rejects password, returns 400 Bad Request with validation errors
- **Measure:** All passwords hashed with BCrypt strength 12

**QS-SEC-04: XSS Attack Prevention**
- **Stimulus:** User submits HTML/JavaScript in input fields
- **Response:** System sanitizes input using OWASP library
- **Measure:** No executable code stored in database

#### 10.2.2 Data Integrity Scenarios

**QS-INT-01: Concurrent Update Conflict**
- **Stimulus:** Two users update same book simultaneously
- **Response:** First update succeeds, second returns 409 Conflict
- **Measure:** Zero lost updates due to optimistic locking

**QS-INT-02: Business Rule Violation (Lending)**
- **Stimulus:** Librarian attempts to lend book to reader with 3 active lendings
- **Response:** System rejects lending, returns 403 Forbidden with explanation
- **Measure:** 100% business rules enforced at service layer

**QS-INT-03: Invalid Data Validation**
- **Stimulus:** User submits invalid ISBN format
- **Response:** System rejects request, returns 400 Bad Request with validation errors
- **Measure:** All Value Objects enforce validation

#### 10.2.3 Performance Scenarios

**QS-PERF-01: Large Dataset Query**
- **Stimulus:** Librarian requests top 5 books from 10,000+ books
- **Response:** Query executes using database aggregation (not in-memory sorting)
- **Measure:** Response time < 2 seconds for top 5 queries

**QS-PERF-02: Pagination**
- **Stimulus:** User requests search results with 5000+ matches
- **Response:** System returns paginated results (default 20 per page)
- **Measure:** Response time consistent regardless of total result size

#### 10.2.4 Maintainability Scenarios

**QS-MAINT-01: Technology Migration**
- **Stimulus:** Need to migrate from H2 to PostgreSQL
- **Response:** Change driver and connection URL, no code changes needed
- **Measure:** Repository abstraction layer isolates persistence technology

**QS-MAINT-02: New Feature Addition**
- **Stimulus:** Add new aggregate (e.g., "Reservation")
- **Response:** Follow existing package structure pattern
- **Measure:** Consistent module structure across all bounded contexts

---

## 11. Risks and Technical Debt

### 11.1 Technical Risks

| ID | Risk | Probability | Impact | Mitigation |
|----|------|-------------|--------|------------|
| **TR-01** | **H2 Database Data Loss** - File corruption or accidental deletion | Medium | High | Regular backups of `./data/` directory; Document migration path to production database |
| **TR-02** | **File Storage Limitations** - Local filesystem not suitable for production | High | Medium | Abstract storage behind `FileStorageService` interface; Document cloud migration path |
| **TR-03** | **JWT Key Compromise** - Private key exposure in version control | Low | Critical | Keys excluded from git via `.gitignore`; Document key rotation procedure |
| **TR-04** | **Scalability Limitations** - Single-instance deployment with file-based database | High | Medium | Document as prototype limitation; Design supports future microservices migration |
| **TR-05** | **Concurrent Access at Scale** - Optimistic locking failures under high contention | Low | Medium | Current design appropriate for library use case (low contention); Monitor conflict rate |
| **TR-06** | **External API Dependency** - API Ninjas service availability | Medium | Low | Bonus feature only; Implement timeout and fallback logic |

### 11.2 Technical Debt

| ID | Debt | Category | Priority | Effort to Fix |
|----|------|----------|----------|---------------|
| **TD-01** | **No Refresh Token Strategy** - JWT expiration requires re-login | Missing Feature | Medium | 2-3 days (implement refresh token endpoint) |
| **TD-02** | **No Async Processing for Reports** - Long-running report queries block HTTP threads | Performance | Low | 5-7 days (implement async with Spring @Async or message queue) |
| **TD-03** | **Limited Error Context** - Some exceptions lack user-friendly messages | Usability | Low | 2-3 days (enhance exception messages) |
| **TD-04** | **No Monitoring/Observability** - Limited production monitoring capabilities | Operations | Medium | 3-5 days (integrate Prometheus, Grafana, or APM tool) |
| **TD-05** | **Bootstrap Data Hard-coded** - Sample data in Java code instead of external files | Maintainability | Low | 1-2 days (move to SQL scripts or JSON files) |
| **TD-06** | **No API Versioning Strategy** - Breaking changes will impact existing clients | Compatibility | Medium | 2-3 days (implement URL-based versioning like `/api/v1/`) |
| **TD-07** | **Limited Photo Format Validation** - File type check by extension only (not magic bytes) | Security | Low | 1 day (implement Apache Tika or similar) |
| **TD-08** | **No Rate Limiting** - API vulnerable to abuse | Security | Medium | 2-3 days (implement Spring Cloud Gateway or Bucket4j) |
| **TD-09** | **H2 Console Enabled in Production Config** - Security risk if deployed as-is | Security | High | 1 hour (externalize config, disable in production profile) |
| **TD-10** | **CORS Allows All Origins** - Security configuration too permissive | Security | Medium | 1 hour (restrict to specific origins in production) |

### 11.3 Architectural Debt

| ID | Issue | Impact | Resolution Strategy |
|----|-------|--------|---------------------|
| **AD-01** | **No Event-Driven Architecture** - Requirement mentions async/events for reporting, but current implementation is synchronous | Future requirement not met | Consider Spring Events, Kafka, or RabbitMQ for reporting data updates |
| **AD-02** | **Monolithic Deployment** - All modules in single JAR despite bounded context separation | Scalability | Current design supports future extraction to microservices (clean module boundaries) |
| **AD-03** | **No Caching Layer** - Repeated queries for same data (e.g., top 5 books) | Performance | Implement Spring Cache with Redis or Caffeine |
| **AD-04** | **No API Gateway** - Direct exposure of backend services | Security, Routing | Consider Spring Cloud Gateway for production |

### 11.4 Known Limitations

1. **Single-Instance Deployment:** H2 file-based database and local file storage prevent horizontal scaling
2. **No Distributed Transactions:** Not applicable to current monolithic design, but required for future microservices
3. **Limited Search Capabilities:** Basic string matching; no full-text search or fuzzy matching
4. **No Soft Deletes:** Entities are hard-deleted; no audit trail for deletions
5. **No Multi-Tenancy:** Single library instance; cannot support multiple independent libraries
6. **No Internationalization (i18n):** Error messages and responses in English only
7. **No Background Jobs:** No scheduled tasks for overdue notifications, fine calculations, etc.

### 11.5 Recommendations for Production Readiness

**Critical (Must-Have):**
1. Migrate to production database (PostgreSQL/MySQL)
2. Implement cloud storage for photos (AWS S3, Azure Blob)
3. Disable H2 Console in production
4. Configure CORS for specific origins only
5. Implement rate limiting
6. Add comprehensive logging (SLF4J + Logback)
7. Implement health checks and readiness probes
8. Externalize RSA keys to secrets management (Vault, AWS Secrets Manager)

**Important (Should-Have):**
1. Implement refresh token strategy
2. Add monitoring and alerting (Prometheus, Grafana)
3. Implement API versioning
4. Add distributed tracing (Jaeger, Zipkin)
5. Implement caching layer (Redis)
6. Add automated backup strategy
7. Implement circuit breakers for external APIs

**Nice-to-Have:**
1. Async processing for reports
2. Full-text search (Elasticsearch)
3. Soft deletes with audit trail
4. Internationalization support
5. Background job scheduling (Quartz, Spring Batch)
6. API Gateway (Spring Cloud Gateway)

---

## 12. Glossary

| Term | Definition |
|------|------------|
| **API** | Application Programming Interface - A software interface that allows two or more computer programs to communicate with each other |
| **Aggregate** | DDD pattern - A cluster of domain objects that can be treated as a single unit (e.g., Book with its Authors) |
| **Author** | A person who wrote books present in the library; can be main author or co-author |
| **Author Bio** | Short biography of an Author |
| **BCrypt** | Password hashing algorithm using adaptive hash function based on Blowfish cipher |
| **Book** | An item stored in the library that is lent to readers by librarians |
| **Bootstrap** | Initial loading of sample data for development/testing purposes |
| **Bounded Context** | DDD pattern - Explicit boundary within which a domain model is defined and applicable |
| **Co-Author** | An author who collaborated with other authors on a book |
| **CORS** | Cross-Origin Resource Sharing - HTTP-header based mechanism allowing servers to specify which origins can access resources |
| **DTO** | Data Transfer Object - Object used to transfer data between processes or layers |
| **Entity** | DDD pattern - Object defined by its identity rather than attributes |
| **ETag** | HTTP response header used for web cache validation and optimistic concurrency control |
| **Fine** | Monetary penalty applied when a lending is overdue (calculated per day of delay) |
| **GDPR** | General Data Protection Regulation - EU regulation on data protection and privacy |
| **Genre** | Category of books (e.g., Science-fiction, Mystery, Law, Medicine) |
| **HATEOAS** | Hypermedia As The Engine Of Application State - REST constraint where responses include links to related resources |
| **ISBN** | International Standard Book Number - Unique identifier for books |
| **JPA** | Jakarta Persistence API (formerly Java Persistence API) - Specification for ORM in Java |
| **JWT** | JSON Web Token - Compact URL-safe means of representing claims between two parties |
| **Lending** | The act of lending a book to a reader; tracked with lending number, dates, and potential fines |
| **Lending Number** | Unique identifier for a lending in format YYYY/SEQ (e.g., "2024/1") |
| **Librarian** | User role with permissions to manage books, authors, and lendings |
| **MapStruct** | Java annotation processor for generating type-safe bean mapping code |
| **Optimistic Locking** | Concurrency control method assuming conflicts are rare, using version numbers to detect conflicts |
| **Reader** | Registered user who can request books for lending and must return them |
| **Reader Number** | Unique identifier for a reader in format YYYY/SEQ (e.g., "2024/1") |
| **Repository Pattern** | Abstraction layer between domain and data mapping layers |
| **REST** | Representational State Transfer - Architectural style for distributed hypermedia systems |
| **Value Object** | DDD pattern - Object defined by its attributes rather than identity; immutable |
| **Version** | Field used for optimistic locking to track entity modifications |

---

## Appendix A: References

### Documentation
- [arc42 Template](https://arc42.org/overview) - Architecture documentation template
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/3.2.5/reference/html/) - Framework reference
- [Domain-Driven Design](https://martinfowler.com/bliki/DomainDrivenDesign.html) - DDD principles by Martin Fowler

### Project Artifacts
- [Docs/GlobalArtifacts/](Docs/GlobalArtifacts/) - Domain model documentation
- [Docs/Phase1/](Docs/Phase1/) - Phase 1 requirements and implementation
- [Docs/Phase2/](Docs/Phase2/) - Phase 2 requirements and implementation
- [pom.xml](pom.xml) - Maven dependencies and build configuration
- [application.properties](src/main/resources/application.properties) - Application configuration

### External APIs
- [API Ninjas](https://api-ninjas.com/) - Historical events API for "funny quotes" feature

---

## Appendix B: Tools and Technologies

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Language** | Java | 17 | Application development |
| **Framework** | Spring Boot | 3.2.5 | Application framework |
| **Build** | Maven | 3.x | Dependency management, build automation |
| **Database** | H2 | Runtime | Embedded database |
| **ORM** | Hibernate | 6.x (via Spring Boot) | JPA implementation |
| **Security** | Spring Security | 6.x (via Spring Boot) | Authentication, authorization |
| **API Docs** | Springdoc OpenAPI | 2.3.0 | OpenAPI 3.0 specification |
| **Mapping** | MapStruct | 1.5.5 | DTO-Entity transformation |
| **Code Gen** | Lombok | 1.18.30 | Boilerplate reduction |
| **Validation** | Jakarta Validation | 3.x | Request validation |
| **Sanitization** | OWASP HTML Sanitizer | 20220608.1 | XSS prevention |
| **HTTP Client** | Spring WebFlux | 6.x (via Spring Boot) | External API calls |
| **Testing** | JUnit | 5.x | Unit testing |
| **API Testing** | POSTMAN | Latest | API testing and documentation |

---