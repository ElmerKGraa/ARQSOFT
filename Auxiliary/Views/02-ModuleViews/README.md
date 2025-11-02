# Module Views - Library Management System

## Overview

Module views document the system's implementation structure as a collection of code units (modules, packages, classes). These views show how the system is organized in terms of its source code structure and how modules relate to each other through dependencies.

**Primary Presentation**: [LayeredModuleView.puml](LayeredModuleView.puml)

**Supporting Diagrams**:
- [PackageStructure.puml](PackageStructure.puml) - Package organization
- [DomainModuleOrganization.puml](DomainModuleOrganization.puml) - Domain module structure
- [ModuleDependencyGraph.puml](ModuleDependencyGraph.puml) - Inter-module dependencies

## Stakeholders

- **Developers**: Understanding code organization, where to add new features
- **Architects**: Ensuring clean boundaries and dependency rules
- **Maintainers**: Locating components for bug fixes and enhancements
- **New Team Members**: Learning the codebase structure
- **Code Reviewers**: Verifying adherence to architectural patterns

## Quality Attributes

This view primarily addresses:
- **Modifiability**: Clean module boundaries enable isolated changes
- **Maintainability**: Logical organization reduces cognitive load
- **Testability**: Clear dependencies enable unit testing
- **Reusability**: Well-defined modules can be reused
- **Understandability**: Consistent structure aids comprehension

---

## Element Catalog

### 1. Presentation Layer (API Controllers)

**Responsibility**: Handle HTTP requests, validate input, invoke business logic, format responses

| Module | Location | Purpose | Key Classes |
|--------|----------|---------|-------------|
| **Author API** | `authormanagement.api` | Author CRUD and queries | `AuthorController`, `AuthorView`, `CreateAuthorRequest` |
| **Book API** | `bookmanagement.api` | Book CRUD, photo upload, search | `BookController`, `BookView`, `CreateBookRequest` |
| **Reader API** | `readermanagement.api` | Reader management | `ReaderController`, `ReaderView`, `CreateReaderRequest` |
| **Lending API** | `lendingmanagement.api` | Lending operations | `LendingController`, `LendingView`, `CreateLendingRequest` |
| **Genre API** | `genremanagement.api` | Genre operations | `GenreController`, `GenreView` |
| **Auth API** | `usermanagement.api` | Authentication and registration | `UserController`, `AuthRequest`, `AuthResponse` |

**Dependencies**:
- ✅ May depend on: Business Layer (Services)
- ❌ Must NOT depend on: Data Layer (Repositories, Entities directly)

**Design Constraints**:
- All controllers use `@RestController` and `@RequestMapping`
- Use DTOs for requests and responses (never expose entities)
- Apply `@PreAuthorize` for role-based access control
- Handle exceptions via `@ControllerAdvice`

---

### 2. Business Layer (Services)

**Responsibility**: Implement business logic, enforce business rules, coordinate operations, transaction management

| Module | Location | Purpose | Key Classes |
|--------|----------|---------|-------------|
| **Author Services** | `authormanagement.services` | Author business logic, photo management, biography generation | `AuthorServiceImpl`, `CreateAuthorRequest`, `UpdateAuthorRequest` |
| **Book Services** | `bookmanagement.services` | Book business logic, ISBN validation, cover photo, co-authors | `BookServiceImpl`, `CreateBookRequest`, `UpdateBookRequest` |
| **Reader Services** | `readermanagement.services` | Reader registration, management | `ReaderServiceImpl`, `CreateReaderRequest` |
| **Lending Services** | `lendingmanagement.services` | Lending rules, overdue calculation, returns | `LendingServiceImpl`, `CreateLendingRequest` |
| **Genre Services** | `genremanagement.services` | Genre management, statistics | `GenreServiceImpl` |
| **User Services** | `usermanagement.services` | User registration, authentication | `UserService` |

**Dependencies**:
- ✅ May depend on: Data Layer (Repositories), Domain Layer (Entities), External Services
- ✅ May depend on: Other Services (for coordination)
- ❌ Must NOT depend on: Presentation Layer (Controllers)

**Design Constraints**:
- All services implement an interface (e.g., `AuthorService`)
- Use `@Service` annotation
- Use `@Transactional` for multi-step operations
- Throw domain-specific exceptions (e.g., `NotFoundException`)

---

### 3. Data Layer (Repositories)

**Responsibility**: Provide data access abstraction, execute queries, manage persistence

| Module | Location | Purpose | Key Classes |
|--------|----------|---------|-------------|
| **Author Repositories** | `authormanagement.repositories` | Author data access | `AuthorRepository`, `AuthorRepositoryImpl` |
| **Book Repositories** | `bookmanagement.repositories` | Book data access | `BookRepository`, `BookRepositoryImpl` |
| **Reader Repositories** | `readermanagement.repositories` | Reader data access | `ReaderRepository`, `ReaderRepositoryImpl` |
| **Lending Repositories** | `lendingmanagement.repositories` | Lending data access | `LendingRepository`, `LendingRepositoryImpl` |
| **Genre Repositories** | `genremanagement.repositories` | Genre data access | `GenreRepository`, `GenreRepositoryImpl` |
| **User Repositories** | `usermanagement.repositories` | User data access | `UserRepository` |

**Dependencies**:
- ✅ May depend on: Domain Layer (Entities)
- ✅ May depend on: JPA/H2 infrastructure
- ❌ Must NOT depend on: Business Layer (Services)
- ❌ Must NOT depend on: Presentation Layer (Controllers)

**Design Constraints**:
- Repository pattern: Interface + Implementation
- Use Spring Data JPA where possible
- Custom queries use `@Query` annotation or Criteria API
- Return domain entities (not DTOs)

---

### 4. Domain Layer (Entities & Value Objects)

**Responsibility**: Define core business concepts, enforce domain invariants, encapsulate business rules

| Module | Location | Purpose | Key Classes |
|--------|----------|---------|-------------|
| **Author Domain** | `authormanagement.model` | Author aggregate | `Author`, `Bio` (value object) |
| **Book Domain** | `bookmanagement.model` | Book aggregate | `Book`, `Isbn` (value object), `Title` (value object) |
| **Reader Domain** | `readermanagement.model` | Reader aggregate | `Reader`, `ReaderDetails` |
| **Lending Domain** | `lendingmanagement.model` | Lending aggregate | `Lending`, `Fine` (value object) |
| **Genre Domain** | `genremanagement.model` | Genre entity | `Genre` |
| **User Domain** | `usermanagement.model` | User aggregate | `User`, `Role` (enum) |

**Dependencies**:
- ✅ May depend on: Shared utilities, JPA annotations
- ❌ Must NOT depend on: Any other layer

**Design Constraints**:
- Entities use `@Entity`, `@Id`, `@Version`
- Value objects are immutable and use `@Embeddable`
- Business validation in constructors and setters
- Rich domain model (behavior in entities)

---

### 5. Cross-Cutting Modules

**Responsibility**: Provide common functionality used across all layers

| Module | Location | Purpose | Key Classes |
|--------|----------|---------|-------------|
| **Exceptions** | `exceptions` | Domain exceptions | `NotFoundException`, `ConflictException` |
| **File Storage** | `filemanagement` | Photo upload/retrieval | `FileStorageService`, `UploadFileResponse` |
| **Configuration** | `configuration` | Application config, security | `SecurityConfig`, `OpenApiConfig` |
| **Shared Services** | `shared.services` | Pagination, mapping | `Page`, `EntityNotFoundException` |

**Dependencies**:
- ✅ Can be used by: Any layer
- ❌ Must NOT depend on: Domain-specific modules

---

## Module Organization Principles

### 1. Layered Architecture Pattern

The system follows a **strict layered architecture** with dependency rules:

```
┌─────────────────────────────────────────┐
│       Presentation Layer (API)          │
│         Controllers, DTOs               │
└────────────────┬────────────────────────┘
                 │ depends on ↓
┌────────────────▼────────────────────────┐
│       Business Layer (Services)         │
│      Business Logic, Coordination       │
└────────────────┬────────────────────────┘
                 │ depends on ↓
┌────────────────▼────────────────────────┐
│       Data Layer (Repositories)         │
│         Data Access Abstraction         │
└────────────────┬────────────────────────┘
                 │ depends on ↓
┌────────────────▼────────────────────────┐
│       Domain Layer (Entities)           │
│      Core Business Concepts             │
└─────────────────────────────────────────┘
```

**Key Rule**: Dependencies only flow **downward**. Lower layers never depend on higher layers.

### 2. Domain Module Organization

Each domain (Author, Book, Reader, Lending, Genre, User) follows a consistent internal structure:

```
{domain}management/
├── api/              # Presentation layer for this domain
│   ├── *Controller.java
│   ├── *View.java
│   └── *Request.java
├── services/         # Business layer for this domain
│   ├── *Service.java
│   ├── *ServiceImpl.java
│   └── *DTO.java
├── repositories/     # Data layer for this domain
│   ├── *Repository.java
│   └── *RepositoryImpl.java
└── model/            # Domain layer for this domain
    ├── *.java (entities)
    └── *.java (value objects)
```

**Benefits**:
- **High Cohesion**: Related components grouped together
- **Low Coupling**: Clear boundaries between domains
- **Easy Navigation**: Predictable structure
- **Parallel Development**: Teams can work on different domains independently

### 3. Dependency Management

**Allowed Dependencies**:
- Controllers → Services (interface, not implementation)
- Services → Repositories (interface, not implementation)
- Services → Domain entities
- Repositories → Domain entities
- Services → Other Services (for coordination)
- Any layer → Shared utilities

**Forbidden Dependencies**:
- Domain → Any other layer
- Repositories → Services
- Repositories → Controllers
- Controllers → Repositories (must go through Services)
- Services → Controller DTOs (use Service DTOs instead)

**Enforcement**:
- Spring Dependency Injection (constructor injection preferred)
- Interface-based programming
- Package-private classes where appropriate
- Code reviews

---

## Interface Specifications

### 1. Controller → Service Interface

**Contract**: Controllers invoke service methods via interface, receive domain objects or service DTOs

**Example**:
```java
// Controller
@RestController
@RequestMapping("/api/authors")
public class AuthorController {
    private final AuthorService authorService;  // Interface, not Impl

    @PostMapping
    public ResponseEntity<AuthorView> create(@RequestBody CreateAuthorRequest request) {
        Author author = authorService.create(request);
        return ResponseEntity.created(...).body(AuthorViewMapper.toAuthorView(author));
    }
}

// Service Interface
public interface AuthorService {
    Author create(CreateAuthorRequest request);
}
```

**Error Handling**: Services throw domain exceptions, controllers catch via `@ControllerAdvice`

### 2. Service → Repository Interface

**Contract**: Services invoke repository methods via interface, receive domain entities

**Example**:
```java
// Service
@Service
public class AuthorServiceImpl implements AuthorService {
    private final AuthorRepository authorRepository;  // Interface

    public Author create(CreateAuthorRequest request) {
        Author author = new Author(request.getName(), request.getBio());
        return authorRepository.save(author);
    }
}

// Repository Interface
public interface AuthorRepository {
    Author save(Author author);
    Optional<Author> findById(Long id);
}
```

### 3. Repository → JPA Interface

**Contract**: Repository implementations extend Spring Data JPA interfaces or use EntityManager

**Example**:
```java
// JPA Repository (simple cases)
public interface AuthorRepositoryJPA extends JpaRepository<Author, Long> {
    Optional<Author> findByName(String name);
}

// Custom Repository (complex queries)
public class AuthorRepositoryImpl implements AuthorRepository {
    private final AuthorRepositoryJPA jpaRepository;
    private final EntityManager entityManager;

    public List<Author> searchByComplexCriteria(...) {
        // Custom JPQL or Criteria API query
    }
}
```

---

## Variability Guide

### Adding a New Domain Module

**Steps**:
1. Create package structure: `{domain}management/model/`, `services/`, `repositories/`, `api/`
2. Define domain entity in `model/` (with `@Entity`, `@Version`)
3. Create repository interface in `repositories/`
4. Implement repository with Spring Data JPA
5. Create service interface and implementation in `services/`
6. Create controller, view DTOs, and request DTOs in `api/`
7. Add security rules to `SecurityConfig.java`
8. Add API documentation annotations

**Example**: Adding a "Publisher" domain
```
publishermanagement/
├── api/
│   ├── PublisherController.java
│   ├── PublisherView.java
│   └── CreatePublisherRequest.java
├── services/
│   ├── PublisherService.java
│   └── PublisherServiceImpl.java
├── repositories/
│   ├── PublisherRepository.java
│   └── PublisherRepositoryImpl.java
└── model/
    └── Publisher.java
```

### Adding a New Feature to Existing Domain

**Example**: Add "Author Awards" feature

1. **Domain Layer**: Add `Award` value object to `authormanagement.model/`
2. **Data Layer**: Add query methods to `AuthorRepository` interface
3. **Business Layer**: Add methods to `AuthorService` interface and implement in `AuthorServiceImpl`
4. **Presentation Layer**: Add endpoint to `AuthorController`, create `AwardView` DTO

**Modified Files**:
- `authormanagement/model/Author.java` (add `@ElementCollection List<Award>`)
- `authormanagement/repositories/AuthorRepository.java` (add query method)
- `authormanagement/services/AuthorService.java` (add business method)
- `authormanagement/services/AuthorServiceImpl.java` (implement)
- `authormanagement/api/AuthorController.java` (add endpoint)
- `authormanagement/api/AwardView.java` (new DTO)

---

## Rationale

### Why Layered Architecture?

**Decision**: Use strict 3-tier layered architecture (Presentation → Business → Data → Domain)

**Alternatives Considered**:
1. **Microservices**: Rejected due to unnecessary complexity for monolithic deployment
2. **Hexagonal/Ports-and-Adapters**: Rejected due to over-engineering for current needs
3. **Flat Package Structure**: Rejected due to poor maintainability

**Reasons**:
- ✅ Clear separation of concerns
- ✅ Easy to understand for team with limited architecture experience
- ✅ Well-supported by Spring Boot
- ✅ Enables parallel development by layer
- ✅ Simplifies testing (can test layers independently)
- ✅ Standard pattern with abundant resources

### Why Domain-Based Package Structure?

**Decision**: Organize by domain (vertical slice) rather than by layer type (horizontal slice)

**Structure Chosen**:
```
authormanagement/
├── api/
├── services/
├── repositories/
└── model/
bookmanagement/
├── api/
├── services/
├── repositories/
└── model/
```

**Alternative Rejected**:
```
controllers/
├── AuthorController.java
├── BookController.java
services/
├── AuthorService.java
├── BookService.java
```

**Reasons**:
- ✅ High cohesion: All author-related code in one place
- ✅ Low coupling: Clear boundaries between domains
- ✅ Easier to navigate: "Where is author logic?" → `authormanagement/`
- ✅ Enables parallel development: Team A works on `authormanagement/`, Team B on `bookmanagement/`
- ✅ Aligns with domain-driven design principles

### Why Interface-Based Services and Repositories?

**Decision**: All services and repositories defined as interfaces with implementations

**Example**:
```java
public interface AuthorService { ... }
public class AuthorServiceImpl implements AuthorService { ... }
```

**Reasons**:
- ✅ Enables dependency injection (Spring best practice)
- ✅ Facilitates testing (can mock interfaces)
- ✅ Allows multiple implementations (e.g., different repository strategies)
- ✅ Enforces contracts between layers
- ✅ Supports Open/Closed Principle (OCP)

---

## Related Views

- **Context Views** ([01-ContextViews](../01-ContextViews/README.md)): Shows external boundaries; module view shows internal structure within those boundaries
- **Component & Connector Views** ([03-ComponentAndConnectorViews](../03-ComponentAndConnectorViews/README.md)): Shows runtime interactions; module view shows compile-time structure
- **Allocation Views** ([04-AllocationViews](../04-AllocationViews/README.md)): Shows deployment; module view shows source code organization
- **Behavior Models** ([06-BehaviorModels](../06-BehaviorModels/README.md)): Shows dynamic behavior; module view shows static structure

---

## Cross-View Mapping

| Module View Element | Maps to Context View | Maps to C&C View |
|---------------------|----------------------|------------------|
| `authormanagement.api.AuthorController` | REST API Gateway | REST API Component |
| `authormanagement.services.AuthorServiceImpl` | Business Logic | Service Component |
| `authormanagement.repositories.AuthorRepositoryImpl` | Data Access | Repository Component |
| `authormanagement.model.Author` | Domain Model | Entity |

---

## Diagram Key

### In all module view diagrams:

**Package Notation**:
```
┌─────────────────────────┐
│  📦 package.name        │
│  ├─ Class1              │
│  └─ Class2              │
└─────────────────────────┘
```

**Dependency Types**:
- **Solid arrow (─────>)**: Uses/depends on
- **Dashed arrow (╌╌╌╌>)**: Optional dependency
- **Thick line (═════)**: Strong coupling
- **Thin line (─────)**: Loose coupling

**Stereotypes**:
- **<<Controller>>**: REST API controller
- **<<Service>>**: Business service
- **<<Repository>>**: Data repository
- **<<Entity>>**: JPA entity
- **<<ValueObject>>**: Immutable value object
- **<<DTO>>**: Data transfer object

---

## Design Notes

### Module Design Principles Applied

1. **Single Responsibility Principle (SRP)**: Each module has one reason to change
   - `AuthorController`: Handle HTTP for authors only
   - `AuthorService`: Author business logic only
   - `AuthorRepository`: Author data access only

2. **Open/Closed Principle (OCP)**: Open for extension, closed for modification
   - New features added via new implementations of existing interfaces
   - Example: Can add `AuthorCachingService` implementing `AuthorService`

3. **Liskov Substitution Principle (LSP)**: Implementations substitutable for interfaces
   - Any `AuthorService` implementation can replace `AuthorServiceImpl`

4. **Interface Segregation Principle (ISP)**: Clients depend only on methods they use
   - Controllers use `AuthorService`, not `AuthorRepositoryImpl`
   - Repositories expose minimal interface

5. **Dependency Inversion Principle (DIP)**: Depend on abstractions, not concretions
   - Controllers depend on `AuthorService` interface, not `AuthorServiceImpl`
   - Services depend on `AuthorRepository` interface, not `AuthorRepositoryImpl`

### Package Visibility

- **Public**: API interfaces, Controllers, DTOs (need external access)
- **Package-private**: Service implementations, Repository implementations (hide implementation)
- **Private**: Internal helper methods, utility classes

### Module Coupling Metrics

**Ideal Coupling** (based on current architecture):
- **Controllers → Services**: ~1-2 services per controller (✅ Current: 1.2 avg)
- **Services → Repositories**: ~1-3 repositories per service (✅ Current: 1.8 avg)
- **Services → Other Services**: 0-2 (⚠️ Current: Up to 3 for LendingService)

**Improvement Opportunities**:
- Reduce cross-domain service dependencies (e.g., LendingService depends on BookService, ReaderService, GenreService)
- Consider introducing a coordination layer for complex cross-domain operations

---

## References

- **Code Location**: `c:\Git\ARQSOFT\Original\src\main\java\pt\psoft\g1\psoftg1\`
- **Package Naming Convention**: `pt.psoft.g1.psoftg1.{domain}management.{layer}`
- **Spring Boot Documentation**: https://spring.io/projects/spring-boot
- **Domain-Driven Design**: Eric Evans, "Domain-Driven Design: Tackling Complexity in the Heart of Software"
- **Clean Architecture**: Robert C. Martin, "Clean Architecture: A Craftsman's Guide to Software Structure and Design"
