# System-as-is (Reverse Engineered Design)

## Library Management System - Group 1

**Document Version:** 1.0  
**Date:** October 27, 2025  
**Project:** PSOFT-G1 - Library Management System

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Technology Stack](#technology-stack)
4. [Architecture Overview](#architecture-overview)
5. [Detailed Component Analysis](#detailed-component-analysis)
6. [Domain Model](#domain-model)
7. [Data Architecture](#data-architecture)
8. [API Design](#api-design)
9. [Security Architecture](#security-architecture)
10. [Quality Attributes Implementation](#quality-attributes-implementation)
11. [Integration Points](#integration-points)
12. [Deployment Architecture](#deployment-architecture)
13. [Technical Debt and Issues](#technical-debt-and-issues)

---

## 1. Executive Summary

This document presents a comprehensive reverse-engineered analysis of the Library Management System developed by Group 1 (PSOFT-G1). The system is a Spring Boot-based web application designed to manage library operations including author management, book cataloging, reader registration, and lending operations.

**Key Characteristics:**
- **Architecture Style:** Layered Architecture with RESTful API
- **Primary Framework:** Spring Boot 3.2.5
- **Persistence:** JPA/Hibernate with H2 Database
- **Security:** JWT-based authentication with OAuth2 Resource Server
- **Documentation:** OpenAPI/Swagger UI enabled

---

## 2. System Overview

### 2.1 Purpose

The Library Management System provides a comprehensive digital solution for managing library operations, including:
- Author and book catalog management
- Reader registration and profile management
- Book lending and return processing
- Fine calculation for overdue books
- Search and reporting capabilities
- Photo management for authors, books, and readers

### 2.2 Stakeholders

- **Librarians:** Full administrative access to all system features
- **Readers:** Self-service access to view catalog, manage profile, and track lendings
- **Anonymous Users:** Limited access to reader registration

### 2.3 Business Context

The system operates in a library domain where:
- Books are lent to registered readers for a defined period
- Multiple authors can co-author books
- Readers must consent to GDPR regulations
- Fines are automatically calculated for overdue returns
- System maintains data integrity through optimistic locking

---

## 3. Technology Stack

### 3.1 Backend Technologies

| Component | Technology | Version |
|-----------|-----------|---------|
| **Language** | Java | 17 |
| **Framework** | Spring Boot | 3.2.5 |
| **Persistence** | Spring Data JPA | 3.2.5 |
| **Database** | H2 Database | Runtime |
| **Security** | Spring Security + OAuth2 | 3.2.5 |
| **Validation** | Jakarta Validation | Latest |
| **Mapping** | MapStruct | 1.5.5.Final |
| **Documentation** | SpringDoc OpenAPI | 2.3.0 |
| **Build Tool** | Maven | - |
| **Sanitization** | OWASP Java HTML Sanitizer | 20220608.1 |

### 3.2 Key Libraries

- **Lombok:** Reduces boilerplate code with annotations
- **BCrypt:** Password encryption
- **JWT (Nimbus):** Token-based authentication
- **H2 Console:** Development database interface
- **Spring Boot DevTools:** Development-time features

### 3.3 Testing Stack

- Spring Boot Test Starter
- Spring Security Test
- JUnit (implied through Spring Boot Test)

---

## 4. Architecture Overview

### 4.1 High-Level Architecture

The system follows a **Layered Architecture** pattern with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│         (REST Controllers + DTOs/Views/Mappers)         │
├─────────────────────────────────────────────────────────┤
│                    Application Layer                     │
│              (Service Interfaces + Impl)                │
├─────────────────────────────────────────────────────────┤
│                     Domain Layer                        │
│            (Entities + Value Objects + Rules)           │
├─────────────────────────────────────────────────────────┤
│                   Persistence Layer                     │
│              (Repository Interfaces + JPA)              │
├─────────────────────────────────────────────────────────┤
│                    Infrastructure                       │
│         (Database, File Storage, Security, Config)      │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Package Structure

The system is organized by **functional modules** (vertical slicing):

```
pt.psoft.g1.psoftg1/
├── PsoftG1Application.java              # Application entry point
├── auth/                                 # Authentication module
│   ├── api/                              # Auth REST endpoints
│   └── services/                         # Auth business logic
├── authormanagement/                     # Author module
│   ├── api/                              # REST controllers & DTOs
│   ├── model/                            # Domain entities & VOs
│   ├── repositories/                     # Data access interfaces
│   ├── services/                         # Business logic
│   └── infrastructure/                   # Implementation details
├── bookmanagement/                       # Book module
│   ├── api/
│   ├── model/
│   ├── repositories/
│   ├── services/
│   └── infrastructure/
├── readermanagement/                     # Reader module
│   ├── api/
│   ├── model/
│   ├── repositories/
│   ├── services/
│   └── infraestructure/                  # [Note: typo in original]
├── lendingmanagement/                    # Lending module
│   ├── api/
│   ├── model/
│   ├── repositories/
│   ├── services/
│   └── infrastructure/
├── genremanagement/                      # Genre module
│   ├── api/
│   ├── model/
│   ├── repositories/
│   └── services/
├── usermanagement/                       # User/Auth module
│   ├── api/
│   ├── model/
│   ├── repositories/
│   ├── services/
│   └── infrastructure/
├── shared/                               # Cross-cutting concerns
│   ├── api/                              # Common DTOs
│   ├── model/                            # Shared value objects
│   ├── repositories/                     # Shared repositories
│   └── services/                         # Shared services
├── configuration/                        # System configuration
│   ├── ApiConfig.java
│   ├── SecurityConfig.java
│   ├── JpaConfig.java
│   └── ApiNinjasConfig.java
├── exceptions/                           # Global exception handling
│   ├── ConflictException.java
│   ├── NotFoundException.java
│   ├── LendingForbiddenException.java
│   ├── FileStorageException.java
│   └── GlobalExceptionHandler.java
├── external/                             # External service integration
│   └── service/
│       └── ApiNinjasService.java
└── bootstrapping/                        # System initialization
    ├── Bootstrapper.java
    └── UserBootstrapper.java
```

### 4.3 Architectural Style Analysis

**Primary Pattern:** Layered Architecture with Domain-Driven Design influences

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

---

## 5. Detailed Component Analysis

### 5.1 Author Management Module

**Responsibilities:**
- Create, update, and query authors
- Manage author photos
- Track author bibliography
- Find co-authors
- Generate top author reports by lending statistics

**Key Components:**

#### 5.1.1 Domain Model
```java
@Entity
public class Author extends EntityWithPhoto {
    @Id @GeneratedValue
    private Long authorNumber;
    
    @Version
    private long version;
    
    @Embedded
    private Name name;
    
    @Embedded
    private Bio bio;
    
    // Business logic methods
    public void applyPatch(long desiredVersion, UpdateAuthorRequest request)
    public void removePhoto(long desiredVersion)
}
```

**Value Objects:**
- `Name`: Encapsulates author name with validation
- `Bio`: Encapsulates biography with business rules

#### 5.1.2 API Layer

**Controller:** `AuthorController`
- `POST /api/authors` - Create author with optional photo
- `PATCH /api/authors/{authorNumber}` - Update author (with optimistic locking)
- `GET /api/authors/{authorNumber}` - Get author details
- `GET /api/authors?name={name}` - Search authors by name
- `GET /api/authors/{authorNumber}/books` - Get books by author
- `GET /api/authors/{authorNumber}/co-authors` - Find co-authors
- `GET /api/authors/top5` - Get top 5 authors by lending count

**DTOs/Views:**
- `CreateAuthorRequest`: Input for author creation
- `UpdateAuthorRequest`: Input for partial updates
- `AuthorView`: Standard author representation
- `AuthorLendingView`: Author with lending statistics
- `CoAuthorView`: Co-author relationship representation

**Mapper:** `AuthorViewMapper` (MapStruct)

#### 5.1.3 Service Layer

**Interface:** `AuthorService`  
**Implementation:** `AuthorServiceImpl`

**Key Operations:**
```java
Author create(CreateAuthorRequest resource)
Author partialUpdate(Long authorNumber, UpdateAuthorRequest request, long desiredVersion)
Optional<Author> findByAuthorNumber(Long authorNumber)
List<Author> findByName(String name)
List<AuthorLendingView> findTopAuthorByLendings()
List<Author> getCoAuthors(Long authorNumber)
```

**Business Rules:**
- Author names must be validated against forbidden names
- Photos are optional but must meet size requirements (max 20KB)
- Updates require version checking (optimistic locking)

#### 5.1.4 Repository Layer

**Interface:** `AuthorRepository`

**Query Methods:**
```java
Optional<Author> findByAuthorNumber(Long authorNumber)
List<Author> searchByNameNameStartsWith(String name)
List<Author> searchByNameName(String name)
Page<AuthorLendingView> findTopAuthorByLendings(Pageable pageableRules)
List<Author> findCoAuthorsByAuthorNumber(Long authorNumber)
```

---

### 5.2 Book Management Module

**Responsibilities:**
- Book catalog management (CRUD operations)
- ISBN validation and uniqueness enforcement
- Genre association
- Author-book relationships (many-to-many)
- Photo management
- Top books reporting

**Key Components:**

#### 5.2.1 Domain Model
```java
@Entity
public class Book extends EntityWithPhoto {
    @Id @GeneratedValue
    private long pk;
    
    @Version
    private Long version;
    
    @Embedded
    private Isbn isbn;  // Business identifier
    
    @Embedded
    private Title title;
    
    @Embedded
    private Description description;
    
    @ManyToOne
    private Genre genre;
    
    @ManyToMany
    private List<Author> authors;
    
    // Business rules
    - At least one author required
    - Genre is mandatory
    - ISBN must be unique
}
```

**Value Objects:**
- `Isbn`: International Standard Book Number with validation
- `Title`: Book title with business rules
- `Description`: Optional book description

#### 5.2.2 API Layer

**Controller:** `BookController`
- `PUT /api/books/{isbn}` - Create book (idempotent)
- `PATCH /api/books/{isbn}` - Update book
- `GET /api/books/{isbn}` - Get book details
- `GET /api/books?genre={genre}` - Search by genre
- `GET /api/books?title={title}` - Search by title
- `GET /api/books/top5` - Get top 5 most lent books

**DTOs/Views:**
- `CreateBookRequest`
- `UpdateBookRequest`
- `BookView`
- `BookCountView`: Book with lending count

#### 5.2.3 Service Layer

**Key Business Rules:**
- ISBN uniqueness validation
- Author list must not be empty
- Genre must exist before book creation
- Photo optional, max 20KB

**Notable Implementation:**
- Uses `Genre` entity reference (not just ID)
- Supports multiple authors per book
- Implements optimistic locking for concurrent updates

---

### 5.3 Reader Management Module

**Responsibilities:**
- Reader registration and profile management
- GDPR consent management
- Marketing and third-party consent tracking
- Reader interests (genre preferences)
- Photo management
- Reader number generation

**Key Components:**

#### 5.3.1 Domain Model
```java
@Entity
public class ReaderDetails extends EntityWithPhoto {
    @Id @GeneratedValue
    private Long pk;
    
    @Version
    private Long version;
    
    @OneToOne
    private Reader reader;  // Link to User
    
    private ReaderNumber readerNumber;  // Business identifier
    
    @Embedded
    private BirthDate birthDate;
    
    @Embedded
    private PhoneNumber phoneNumber;
    
    private boolean gdprConsent;           // Must be true
    private boolean marketingConsent;
    private boolean thirdPartySharingConsent;
    
    @ManyToMany
    private List<Genre> interestList;
}
```

**Separation of Concerns:**
- `User` (in usermanagement): Authentication credentials
- `Reader` (extends User): Reader-specific user type
- `ReaderDetails`: Reader profile information

**Value Objects:**
- `ReaderNumber`: Format YYYY/NNNN (year/sequential)
- `BirthDate`: Date of birth with validation
- `PhoneNumber`: Phone number with format validation

#### 5.3.2 API Layer

**Controller:** `ReaderController`
- `POST /api/readers` - Register new reader (public access)
- `GET /api/readers/{year}/{seq}` - Get reader details
- `PATCH /api/readers/{year}/{seq}` - Update reader profile
- `GET /api/readers?name={name}` - Search readers (librarian only)
- `GET /api/readers/{year}/{seq}/lendings` - Get reader lending history

**Security Rules:**
- Registration is public
- Readers can only view/update their own profile
- Librarians have full access

#### 5.3.3 Service Layer

**Key Business Rules:**
- GDPR consent is mandatory (must be true)
- Reader number automatically generated (year/sequential format)
- Marketing and third-party consents are optional
- Photo optional, max 20KB
- Age validation through birthDate

---

### 5.4 Lending Management Module

**Responsibilities:**
- Book lending operations
- Return processing
- Fine calculation and management
- Lending history tracking
- Overdue detection
- Reader commentary collection

**Key Components:**

#### 5.4.1 Domain Model
```java
@Entity
public class Lending {
    @Id @GeneratedValue
    private Long pk;
    
    @Version
    private long version;
    
    private LendingNumber lendingNumber;  // YYYY/NNNN format
    
    @ManyToOne(fetch = EAGER)
    private Book book;
    
    @ManyToOne(fetch = EAGER)
    private ReaderDetails readerDetails;
    
    private LocalDate startDate;          // Creation date
    private LocalDate limitDate;          // Due date
    private LocalDate returnedDate;       // Actual return (null if not returned)
    
    private String commentary;            // Optional reader comment
    
    @OneToOne(cascade = ALL, orphanRemoval = true)
    private Fine fine;                    // Calculated fine
    
    // Business logic methods
    public void returnBook(String commentary, LocalDate returnDate, int fineValuePerDayInCents)
    public Fine calculateFine(LocalDate returnDate, int fineValuePerDayInCents)
}
```

**Fine Entity:**
```java
@Entity
public class Fine {
    @Id @GeneratedValue
    private Long pk;
    
    private int fineValueInCents;
    
    // Calculated based on days overdue × fine rate
}
```

**Value Objects:**
- `LendingNumber`: Unique identifier in format YYYY/NNNN

#### 5.4.2 Business Rules

**Lending Creation:**
- Reader must not have overdue books
- Reader must not have unpaid fines
- Book must be available (not currently lent)
- Duration configured in system properties

**Return Processing:**
- Only the reader who borrowed can return
- Fine automatically calculated if overdue
- Optional commentary accepted
- Return date recorded

**Fine Calculation:**
```
Fine = max(0, days_overdue × fine_rate_per_day)
where days_overdue = return_date - limit_date
```

#### 5.4.3 API Layer

**Controller:** `LendingController`
- `POST /api/lendings` - Create lending (librarian only)
- `GET /api/lendings/{year}/{seq}` - Get lending details
- `PATCH /api/lendings/{year}/{seq}` - Return book
- `GET /api/lendings/average-duration` - Calculate average lending duration
- `GET /api/lendings/overdue` - List overdue lendings

**Security:**
- Librarians can create lendings
- Readers can view their own lendings
- Readers can return their own books
- Librarians have full visibility

---

### 5.5 Genre Management Module

**Responsibilities:**
- Genre catalog maintenance
- Genre lookup for books and reader interests

**Key Components:**

#### 5.5.1 Domain Model
```java
@Entity
public class Genre {
    @Id @GeneratedValue
    private Long pk;
    
    @Column(unique = true, nullable = false)
    private String genre;  // Business identifier
    
    @Version
    private long version;
}
```

**Characteristics:**
- Simple entity with single business attribute
- Genre name is unique and serves as natural key
- Used as reference in Book and ReaderDetails

#### 5.5.2 API Layer

**Controller:** `GenreController`
- `GET /api/genres` - List all genres
- `GET /api/genres/top5` - Top 5 genres by book count

**DTOs:**
- `GenreView`: Simple representation
- `GenreBookCountView`: Genre with book count

---

### 5.6 User Management Module

**Responsibilities:**
- User authentication and authorization
- Role-based access control (RBAC)
- Password management
- User lifecycle management

**Key Components:**

#### 5.6.1 Domain Model

**User Hierarchy:**
```java
@Entity
@Table(name = "T_USER")
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class User implements UserDetails {
    @Id @GeneratedValue
    private Long id;
    
    @Version
    private Long version;
    
    @Embedded
    private Name name;
    
    @Column(unique = true)
    private String username;
    
    private String password;  // BCrypt encrypted
    
    @ElementCollection(fetch = EAGER)
    private Set<Role> authorities;
    
    private boolean enabled;
    private boolean accountNonExpired;
    private boolean accountNonLocked;
    private boolean credentialsNonExpired;
    
    // Spring Security UserDetails implementation
}

public class Librarian extends User { }

public class Reader extends User {
    @OneToOne(mappedBy = "reader")
    private ReaderDetails readerDetails;
}
```

**Role Enum:**
```java
public enum Role {
    LIBRARIAN,
    READER,
    ADMIN
}
```

#### 5.6.2 Security Integration

**Features:**
- Implements Spring Security `UserDetails` interface
- Password encryption using BCrypt
- Account status flags (expired, locked, enabled)
- Role-based authorization
- Auditing fields (@CreatedBy, @CreatedDate, @LastModifiedBy, @LastModifiedDate)

---

### 5.7 Shared Module

**Purpose:** Cross-cutting concerns and reusable components

#### 5.7.1 Value Objects

**EntityWithPhoto (Abstract Base Class):**
```java
@MappedSuperclass
public abstract class EntityWithPhoto {
    @OneToOne(cascade = ALL, orphanRemoval = true)
    private Photo photo;
    
    protected void setPhotoInternal(String photoURI);
    public Optional<Photo> getPhoto();
}
```

**Photo Entity:**
```java
@Entity
public class Photo {
    @Id @GeneratedValue
    private Long pk;
    
    private String photoFile;  // File path
}
```

**Name Value Object:**
```java
@Embeddable
public class Name {
    @Column(nullable = false)
    private String name;
    
    // Validation against forbidden names
    // HTML sanitization
}
```

**Other Shared Value Objects:**
- `StringUtilsCustom`: String manipulation utilities
- `FileUtils`: File handling utilities

#### 5.7.2 Shared Services

**FileStorageService:**
- Photo upload and storage
- File size validation (max 20KB for photos)
- File retrieval and serving
- UUID-based filename generation
- Supports PNG, JPG, GIF formats

**ForbiddenNameService:**
- Loads forbidden names from `forbiddenNames.txt`
- Validates user input against forbidden list
- Used by Name value object

**ConcurrencyService:**
- Extracts version from `If-Match` HTTP header
- Supports optimistic locking pattern

#### 5.7.3 Shared Repositories

**PhotoRepository:**
- CRUD operations for Photo entity

**ForbiddenNameRepository:**
- Manages forbidden name list

---

### 5.8 Configuration Module

#### 5.8.1 SecurityConfig

**Key Features:**
- JWT-based authentication using RSA keys
- OAuth2 Resource Server configuration
- CORS configuration
- Role-based endpoint protection
- Stateless session management
- BCrypt password encoder

**Authentication Flow:**
1. User logs in with credentials
2. System validates credentials
3. JWT token generated with RSA private key
4. Client includes token in Authorization header
5. System validates token with RSA public key

**Protected Endpoints:**
- Public: `/api/public/**`, `/login`, `/api/readers` (POST)
- Librarian: Author management, book management, lending creation
- Reader: Own profile, own lendings
- Admin: User management

#### 5.8.2 JpaConfig

**Features:**
- JPA Auditing enabled (`@EnableJpaAuditing`)
- Custom auditor provider for created/modified tracking
- Maps authenticated user to audit fields

#### 5.8.3 ApiConfig

**Swagger/OpenAPI Configuration:**
- API documentation generation
- JWT security scheme configuration
- API metadata (title, version, description)

#### 5.8.4 ApiNinjasConfig

**External API Integration:**
- Configuration for API Ninjas service
- API key management
- RestTemplate configuration

---

### 5.9 Exception Handling

**Global Exception Handler:** `GlobalExceptionHandler`

**Custom Exceptions:**
- `NotFoundException`: Resource not found (404)
- `ConflictException`: Version conflict or business rule violation (409)
- `LendingForbiddenException`: Lending not allowed (403)
- `FileStorageException`: File upload/storage issues

**Handled Exceptions:**
- `IllegalArgumentException`: Bad request (400)
- `ValidationException`: Validation errors (400)
- `StaleObjectStateException`: Optimistic locking failure (409)
- `AccessDeniedException`: Authorization failure (403)

---

### 5.10 Bootstrapping

**Purpose:** Initialize system with test data

**Components:**
- `UserBootstrapper`: Creates default users (admin, librarians, readers)
- `Bootstrapper`: Can be extended for additional initialization

**Configuration:**
- Runs on profile `bootstrap`
- Configured in `application.properties`

---

## 6. Domain Model

### 6.1 Aggregates

The system identifies four primary aggregates:

#### 6.1.1 Author Aggregate
- **Root:** Author
- **Entities:** Author
- **Value Objects:** Name, Bio, Photo
- **Identifier:** authorNumber (Long)

#### 6.1.2 Book Aggregate
- **Root:** Book
- **Entities:** Book
- **Value Objects:** Isbn, Title, Description, Photo
- **References:** Genre (external), List<Author> (external)
- **Identifier:** ISBN (business key), pk (technical key)

#### 6.1.3 Reader Aggregate
- **Root:** ReaderDetails
- **Entities:** ReaderDetails, Reader (User)
- **Value Objects:** ReaderNumber, BirthDate, PhoneNumber, Photo, Name
- **References:** List<Genre> (interests)
- **Identifier:** ReaderNumber (YYYY/NNNN format)

#### 6.1.4 Lending Aggregate
- **Root:** Lending
- **Entities:** Lending, Fine
- **Value Objects:** LendingNumber
- **References:** Book (external), ReaderDetails (external)
- **Identifier:** LendingNumber (YYYY/NNNN format)

### 6.2 Entity Relationships

```
User (abstract)
├── Librarian
└── Reader ──────1:1────── ReaderDetails
                            │
                            │ M:N
                            ↓
                           Genre
                            ↑
                            │ M:1
Book ────M:N──── Author    │
│                           │
│ M:1───────────────────────┘
│
│ M:1
└────> Lending ──1:1──> Fine
       │ M:1
       └────> ReaderDetails
```

### 6.3 Key Relationships

| From | Relationship | To | Type | Cardinality |
|------|--------------|-----|------|-------------|
| Book | written by | Author | ManyToMany | M:N |
| Book | belongs to | Genre | ManyToOne | M:1 |
| Reader | extends | User | Inheritance | 1:1 |
| Reader | has | ReaderDetails | OneToOne | 1:1 |
| ReaderDetails | interested in | Genre | ManyToMany | M:N |
| Lending | for | Book | ManyToOne | M:1 |
| Lending | by | ReaderDetails | ManyToOne | M:1 |
| Lending | has | Fine | OneToOne | 1:0..1 |
| Author | has | Photo | OneToOne | 1:0..1 |
| Book | has | Photo | OneToOne | 1:0..1 |
| ReaderDetails | has | Photo | OneToOne | 1:0..1 |

---

## 7. Data Architecture

### 7.1 Database Configuration

**Database:** H2 (In-Memory/File-Based)

**Connection Modes:**
```properties
# TCP mode (H2 Console must be running)
jdbc:h2:tcp://localhost/~/psoft-g1;IGNORECASE=TRUE

# File mode (H2 Console must be closed)
jdbc:h2:~/psoft-g1

# In-memory mode (no prerequisites)
jdbc:h2:mem:testdb
```

**Schema Management:**
- Strategy: `spring.jpa.hibernate.ddl-auto=update`
- Automatic DDL generation enabled
- Schema evolves with entity changes

### 7.2 Persistence Patterns

#### 7.2.1 Identity Generation

**Strategies Used:**
- **Technical PKs:** `@GeneratedValue(strategy = GenerationType.AUTO)` for all entities
- **Business Keys:** Separate fields (ISBN, authorNumber, readerNumber, lendingNumber)

**Rationale:**
- Technical PKs for database efficiency
- Business keys for domain identity
- Separation allows key evolution

#### 7.2.2 Optimistic Locking

**Implementation:**
```java
@Version
private Long version;
```

**Usage:**
- All mutable entities have version field
- Version checked on update operations
- `StaleObjectStateException` on conflict
- Client must provide `If-Match` header with version

**Process:**
1. Client reads entity (receives version in ETag header)
2. Client sends update with `If-Match: {version}` header
3. Server validates version matches current
4. Update succeeds or throws `ConflictException`

#### 7.2.3 Embedded Value Objects

**Pattern:**
```java
@Embedded
private Name name;

@Embeddable
public class Name {
    @Column(nullable = false)
    private String name;
}
```

**Value Objects:**
- Name, Bio (Author)
- Isbn, Title, Description (Book)
- ReaderNumber, BirthDate, PhoneNumber (Reader)
- LendingNumber (Lending)

**Benefits:**
- Encapsulation of business rules
- Type safety
- Reusability

#### 7.2.4 Inheritance Strategies

**User Hierarchy:**
```java
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class User { }
```

**Strategy:** JOINED (normalized, separate tables for each subclass)

**Tables Generated:**
- T_USER (base table)
- LIBRARIAN (subclass table)
- READER (subclass table)

#### 7.2.5 Cascade and Orphan Removal

**Photo Management:**
```java
@OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
private Photo photo;
```

**Fine Management:**
```java
@OneToOne(cascade = CascadeType.ALL, orphanRemoval = true)
private Fine fine;
```

**Benefit:** Automatic lifecycle management of dependent entities

---

## 8. API Design

### 8.1 RESTful Principles

**Conventions:**
- Resource-based URLs
- HTTP verbs for operations (POST, GET, PATCH, PUT)
- HTTP status codes for responses
- HATEOAS links where applicable
- JSON as primary media type
- Multipart/form-data for file uploads

### 8.2 Endpoint Catalog

#### 8.2.1 Author Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/authors` | Create author | Librarian |
| GET | `/api/authors/{id}` | Get author details | Librarian, Reader |
| PATCH | `/api/authors/{id}` | Update author | Librarian |
| DELETE | `/api/authors/{id}/photo` | Remove photo | Librarian |
| GET | `/api/authors?name={name}` | Search by name | Librarian, Reader |
| GET | `/api/authors/{id}/books` | Get author's books | Librarian, Reader |
| GET | `/api/authors/{id}/co-authors` | Find co-authors | Librarian, Reader |
| GET | `/api/authors/top5` | Top 5 by lendings | Librarian, Reader |

#### 8.2.2 Book Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| PUT | `/api/books/{isbn}` | Create/Update book (idempotent) | Librarian |
| GET | `/api/books/{isbn}` | Get book details | Librarian, Reader |
| PATCH | `/api/books/{isbn}` | Partial update | Librarian |
| DELETE | `/api/books/{isbn}/photo` | Remove photo | Librarian |
| GET | `/api/books?genre={genre}` | Search by genre | Librarian, Reader |
| GET | `/api/books?title={title}` | Search by title | Librarian, Reader |
| GET | `/api/books/top5` | Top 5 most lent | Librarian, Reader |

#### 8.2.3 Reader Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/readers` | Register reader | Public |
| GET | `/api/readers/{year}/{seq}` | Get reader details | Librarian, Self |
| PATCH | `/api/readers/{year}/{seq}` | Update reader | Self |
| DELETE | `/api/readers/{year}/{seq}/photo` | Remove photo | Self |
| GET | `/api/readers?name={name}` | Search by name | Librarian |
| GET | `/api/readers/{year}/{seq}/lendings` | Reader's lendings | Librarian, Self |
| GET | `/api/readers/top5` | Top 5 readers | Librarian |

#### 8.2.4 Lending Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/lendings` | Create lending | Librarian |
| GET | `/api/lendings/{year}/{seq}` | Get lending details | Librarian, Self |
| PATCH | `/api/lendings/{year}/{seq}` | Return book | Self |
| GET | `/api/lendings/overdue` | List overdue | Librarian |
| GET | `/api/lendings/average` | Average duration | Librarian |
| GET | `/api/lendings/per-month` | Monthly stats | Librarian |

#### 8.2.5 Genre Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/genres` | List all genres | Librarian, Reader |
| GET | `/api/genres/top5` | Top 5 by book count | Librarian, Reader |

#### 8.2.6 Authentication Endpoints

| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/login` | Authenticate user | Public |
| POST | `/api/public/register` | Register user | Public |

### 8.3 Request/Response Patterns

#### 8.3.1 Create Operations (POST)

**Pattern:**
```http
POST /api/authors
Content-Type: multipart/form-data

{
    "name": "John Doe",
    "bio": "Famous author...",
    "photo": [binary]
}

Response:
201 Created
Location: /api/authors/1
ETag: "0"
{
    "authorNumber": 1,
    "name": "John Doe",
    "bio": "Famous author...",
    "photoUrl": "http://.../photos/uuid.jpg",
    "_links": { ... }
}
```

#### 8.3.2 Update Operations (PATCH)

**Pattern:**
```http
PATCH /api/authors/1
If-Match: "0"
Content-Type: multipart/form-data

{
    "bio": "Updated biography"
}

Response:
200 OK
ETag: "1"
{
    "authorNumber": 1,
    "name": "John Doe",
    "bio": "Updated biography",
    ...
}
```

**Optimistic Locking:**
- Client must provide `If-Match` header with current version
- Server returns new version in `ETag` header
- 409 Conflict if version mismatch

#### 8.3.3 Query Operations (GET)

**Single Resource:**
```http
GET /api/authors/1

Response:
200 OK
ETag: "1"
{
    "authorNumber": 1,
    ...
}
```

**Collection with Search:**
```http
GET /api/authors?name=John&page=0&size=10

Response:
200 OK
{
    "content": [ ... ],
    "page": 0,
    "size": 10,
    "totalElements": 25,
    "totalPages": 3
}
```

### 8.4 Error Responses

**Standard Error Format:**
```json
{
    "timestamp": "2025-10-27T10:30:00",
    "status": 404,
    "error": "Not Found",
    "message": "Author not found with id: 999",
    "path": "/api/authors/999"
}
```

**HTTP Status Codes:**
- 200 OK: Successful GET
- 201 Created: Successful POST
- 204 No Content: Successful DELETE
- 400 Bad Request: Validation error
- 401 Unauthorized: Missing/invalid authentication
- 403 Forbidden: Insufficient permissions
- 404 Not Found: Resource doesn't exist
- 409 Conflict: Version mismatch or business rule violation
- 500 Internal Server Error: Unexpected error

### 8.5 API Documentation

**Tool:** SpringDoc OpenAPI 3

**Access:**
- API Docs: `http://localhost:8080/api-docs`
- Swagger UI: `http://localhost:8080/swagger-ui`

**Features:**
- Interactive API testing
- Request/response schemas
- Authentication configuration
- Example values

---

## 9. Security Architecture

### 9.1 Authentication Mechanism

**Strategy:** JWT (JSON Web Token) with RSA asymmetric encryption

#### 9.1.1 Token Generation

**Process:**
1. User submits credentials (username/password) to `/api/login`
2. System validates against database (BCrypt password check)
3. System generates JWT signed with RSA private key
4. Token includes:
   - Subject (username)
   - Authorities (roles)
   - Issue time
   - Expiration time
5. Client receives token

**Keys:**
- Private Key: `classpath:rsa.private.key` (for signing)
- Public Key: `classpath:rsa.public.key` (for verification)

#### 9.1.2 Token Validation

**Process:**
1. Client includes token in request: `Authorization: Bearer {token}`
2. OAuth2 Resource Server intercepts request
3. Token signature verified with RSA public key
4. Token claims extracted (username, roles)
5. Spring Security context populated
6. Request proceeds to controller

### 9.2 Authorization Model

**Mechanism:** Role-Based Access Control (RBAC)

#### 9.2.1 Roles

| Role | Description | Capabilities |
|------|-------------|--------------|
| **ADMIN** | System administrator | Full system access |
| **LIBRARIAN** | Library staff | Manage authors, books, genres, create lendings |
| **READER** | Library patron | View catalog, manage own profile, borrow/return books |

#### 9.2.2 Authorization Rules

**Implementation Methods:**
1. **Method Security:** `@PreAuthorize` annotations
2. **HTTP Security:** Security filter chain configuration

**Examples:**

```java
// Method-level
@PreAuthorize("hasRole('LIBRARIAN')")
public Author create(CreateAuthorRequest request) { }

// HTTP Security
http.authorizeHttpRequests(auth -> auth
    .requestMatchers(HttpMethod.POST, "/api/authors").hasRole("LIBRARIAN")
    .requestMatchers(HttpMethod.GET, "/api/books/**").hasAnyRole("LIBRARIAN", "READER")
);
```

#### 9.2.3 Access Matrix

| Resource | Operation | ADMIN | LIBRARIAN | READER | Anonymous |
|----------|-----------|-------|-----------|--------|-----------|
| Authors | Create/Update | ✓ | ✓ | ✗ | ✗ |
| Authors | Read | ✓ | ✓ | ✓ | ✗ |
| Books | Create/Update | ✓ | ✓ | ✗ | ✗ |
| Books | Read | ✓ | ✓ | ✓ | ✗ |
| Readers | Create | ✓ | ✓ | ✗ | ✓ |
| Readers | Read | ✓ | ✓ | Self | ✗ |
| Readers | Update | ✓ | ✗ | Self | ✗ |
| Lendings | Create | ✓ | ✓ | ✗ | ✗ |
| Lendings | Read | ✓ | ✓ | Self | ✗ |
| Lendings | Return | ✓ | ✗ | Self | ✗ |
| Genres | All | ✓ | ✓ | ✓ | ✗ |

### 9.3 Password Security

**Hashing Algorithm:** BCrypt with default strength (10 rounds)

**Configuration:**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

**Features:**
- Salt automatically generated per password
- Configurable work factor (computational cost)
- Resistant to rainbow table attacks

### 9.4 Input Validation and Sanitization

#### 9.4.1 Validation

**Mechanism:** Jakarta Bean Validation (JSR 380)

**Annotations Used:**
- `@NotNull`: Field cannot be null
- `@NotBlank`: String cannot be empty
- `@Email`: Valid email format
- `@Size`: String length constraints
- `@Valid`: Nested object validation

**Example:**
```java
public class CreateAuthorRequest {
    @NotBlank(message = "Name is mandatory")
    private String name;
    
    @NotBlank(message = "Bio is mandatory")
    @Size(max = 4096, message = "Bio too long")
    private String bio;
}
```

#### 9.4.2 HTML Sanitization

**Library:** OWASP Java HTML Sanitizer

**Purpose:** Prevent XSS attacks in user-generated content

**Usage:** Applied to text fields that may contain formatting (bio, description, commentary)

### 9.5 Forbidden Names

**Mechanism:** Blacklist-based name filtering

**Implementation:**
- Forbidden names stored in `forbiddenNames.txt`
- Loaded at startup by `ForbiddenNameService`
- Validated in `Name` value object
- Throws `IllegalArgumentException` if match found

### 9.6 CORS Configuration

**Configuration:**
```java
@Bean
public CorsFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowCredentials(true);
    config.addAllowedOriginPattern("*");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    return new CorsFilter(source);
}
```

**Current State:** Permissive (allows all origins)  
**Recommendation:** Restrict to known frontend origins in production

### 9.7 CSRF Protection

**Status:** Disabled for stateless JWT authentication

**Rationale:**
- JWT stored client-side (not in cookies)
- Stateless session management
- No server-side session to hijack

---

## 10. Quality Attributes Implementation

### 10.1 Maintainability

**Achieved Through:**

1. **Modular Architecture**
   - Clear module boundaries
   - High cohesion within modules
   - Low coupling between modules

2. **Separation of Concerns**
   - API layer: Request/response handling
   - Service layer: Business logic
   - Domain layer: Business rules
   - Repository layer: Data access

3. **Code Generation**
   - Lombok: Reduces boilerplate (getters, setters, constructors)
   - MapStruct: Automated DTO-Entity mapping

4. **Naming Conventions**
   - Consistent naming across modules
   - Self-documenting code
   - Package structure reflects architecture

5. **Documentation**
   - OpenAPI/Swagger for API docs
   - JavaDoc for complex logic (where present)
   - Domain model documentation in `/Docs`

**Metrics:**
- Package coupling: Low (modules largely independent)
- Cyclomatic complexity: Generally low
- Code duplication: Minimal (shared module)

### 10.2 Testability

**Achieved Through:**

1. **Dependency Injection**
   - Constructor-based injection (recommended by Lombok's `@RequiredArgsConstructor`)
   - Easy to mock dependencies

2. **Interface Segregation**
   - Services defined as interfaces
   - Implementations can be swapped

3. **Repository Abstraction**
   - JPA repositories easily mocked
   - Custom queries testable independently

4. **Test Infrastructure**
   - Separate test profile (`application-integrationtest.properties`)
   - In-memory H2 database for integration tests
   - Spring Security Test support

**Test Structure:**
```
test/java/pt/psoft/g1/psoftg1/
├── authormanagement/
│   ├── services/
│   │   └── AuthorServiceImplIntegrationTest.java
│   └── model/
│       └── AuthorTest.java
├── bookmanagement/
├── lendingmanagement/
├── readermanagement/
└── testutils/
```

**Testing Levels:**
- **Unit Tests:** Domain model validation (AuthorTest, BookTest, etc.)
- **Integration Tests:** Service layer with database (AuthorServiceImplIntegrationTest)
- **API Tests:** Postman collections in `/resources/assets`

### 10.3 Scalability Considerations

**Current State:** Single-instance application

**Scalability Constraints:**
1. **H2 File Database**
   - File-based H2 doesn't support multiple writers
   - TCP mode requires single H2 Console instance

2. **File Storage**
   - Local filesystem storage for photos
   - Not shared across instances

3. **Sequential Number Generation**
   - Reader numbers: YYYY/NNNN
   - Lending numbers: YYYY/NNNN
   - Potential race condition with concurrent requests

**Scalability Enablers:**
1. **Stateless Design**
   - JWT authentication (no server-side sessions)
   - Each request self-contained

2. **Optimistic Locking**
   - Supports concurrent updates
   - No pessimistic locks

3. **Database Independence**
   - JPA abstraction
   - Easy to migrate to PostgreSQL/MySQL

**Recommendations for Scale:**
1. Replace H2 with production database (PostgreSQL)
2. Implement distributed file storage (S3, Azure Blob)
3. Use database sequences for number generation
4. Add caching layer (Redis) for read-heavy operations
5. Implement proper database connection pooling

### 10.4 Performance

**Current Optimizations:**

1. **Eager Fetching** (Selective)
   ```java
   @ManyToOne(fetch = FetchType.EAGER)
   private Book book;
   ```
   - Used in Lending for commonly accessed relationships
   - Reduces N+1 query problem

2. **Query Optimization**
   - Custom JPQL queries for complex operations
   - Pagination support for large result sets

3. **Caching** (Implicit)
   - JPA first-level cache (session)
   - Second-level cache not configured

**Performance Considerations:**

1. **Photo Storage**
   - Files stored on filesystem (fast)
   - 20KB size limit (reasonable)
   - UUID filenames prevent collisions

2. **Database Queries**
   - Most queries by primary key (fast)
   - Some full table scans for statistics (Top 5 queries)

**Recommendations:**
1. Add database indexes on frequently queried fields
2. Implement second-level cache (Caffeine/EhCache)
3. Add query result pagination everywhere
4. Profile slow queries and optimize
5. Consider database views for complex reports

### 10.5 Reliability

**Achieved Through:**

1. **Data Integrity**
   - Database constraints (unique, not null)
   - Foreign key relationships
   - Optimistic locking for concurrent updates

2. **Transaction Management**
   - Declarative transactions (`@Transactional`)
   - ACID guarantees from database

3. **Exception Handling**
   - Global exception handler
   - Consistent error responses
   - Logging (implicit through Spring Boot)

4. **Validation**
   - Input validation at API layer
   - Business rule validation in domain
   - Database constraint validation

**Reliability Gaps:**

1. **No Circuit Breakers**
   - External API calls (ApiNinjas) not protected
   - No fallback mechanisms

2. **Limited Error Recovery**
   - No retry logic
   - No compensation transactions

3. **No Health Checks**
   - No actuator endpoints
   - No monitoring configured

**Recommendations:**
1. Add Spring Boot Actuator for health checks
2. Implement circuit breaker (Resilience4j)
3. Add comprehensive logging
4. Implement retry logic for external services
5. Add database backup strategy

### 10.6 Security (Quality Attribute)

**Strengths:**

1. **Authentication & Authorization**
   - Industry-standard JWT
   - RSA encryption (secure)
   - Role-based access control

2. **Password Security**
   - BCrypt hashing
   - Salting automatic

3. **Input Validation**
   - Jakarta Validation
   - HTML sanitization
   - Forbidden name filtering

4. **API Security**
   - Stateless (no session fixation)
   - CORS configured
   - CSRF not needed (JWT)

**Weaknesses:**

1. **CORS Too Permissive**
   - Allows all origins
   - Should restrict in production

2. **No Rate Limiting**
   - Vulnerable to brute force
   - No throttling

3. **No Audit Logging**
   - Limited audit trail
   - Who did what when?

4. **RSA Keys in Classpath**
   - Keys hardcoded in repository
   - Should use external key management

5. **No HTTPS Enforcement**
   - HTTP allowed
   - Credentials sent in clear (in dev)

6. **No Input Length Limits**
   - Some fields lack size validation
   - Potential DoS via large inputs

**Recommendations:**
1. Restrict CORS to known origins
2. Implement rate limiting (Bucket4j)
3. Add comprehensive audit logging
4. Externalize RSA keys (environment variables, vault)
5. Enforce HTTPS in production
6. Add input length validation everywhere
7. Implement account lockout after failed logins
8. Add API key rotation mechanism

---

## 11. Integration Points

### 11.1 External API Integration

#### 11.1.1 API Ninjas

**Purpose:** External service for enriching data (specific use case not evident from code)

**Configuration:**
```properties
my.ninjas-key=a5nSlaa4JxIubY09H+NYuQ==cY9FegnFmAvYi6fN
```

**Implementation:** `ApiNinjasService`

**Integration Pattern:**
- RestTemplate-based HTTP client
- API key authentication
- Configured via `ApiNinjasConfig`

**Considerations:**
- No error handling visible
- No circuit breaker
- No fallback strategy
- Synchronous calls (blocking)

### 11.2 File System Integration

**Purpose:** Photo storage

**Configuration:**
```properties
file.upload-dir=uploads-psoft-g1
file.photo_max_size=20000
```

**Implementation:** `FileStorageService`

**Operations:**
- Upload: `storeFile(MultipartFile file)`
- Retrieve: `loadFileAsResource(String fileName)`
- Delete: Cascade delete via entity removal

**Storage Strategy:**
- Local filesystem
- UUID-based filenames
- Relative path storage in database

**Limitations:**
- Not suitable for distributed deployment
- No CDN integration
- No image transformation

### 11.3 Database Integration

**Current:** H2 (embedded)

**Migration Path:**
- JPA abstraction enables easy database change
- Hibernate DDL generation
- No database-specific queries (portable)

**Recommended Production Databases:**
- PostgreSQL (recommended)
- MySQL
- SQL Server
- Oracle

---

## 12. Deployment Architecture

### 12.1 Application Packaging

**Type:** Executable JAR (Spring Boot)

**Build:**
```bash
mvn clean package
```

**Execution:**
```bash
java -jar target/psoft-g1-0.0.1-SNAPSHOT.jar
```

**Embedded Components:**
- Tomcat web server (embedded)
- H2 database (embedded)
- All dependencies (fat JAR)

### 12.2 Configuration Management

**Mechanism:** Spring Boot externalized configuration

**Files:**
- `application.properties`: Base configuration
- `application-integrationtest.properties`: Test-specific overrides
- `config/library.properties`: Domain-specific configuration

**Profiles:**
- `bootstrap`: Includes test data initialization

**Override Order:**
1. Command-line arguments
2. Environment variables
3. application-{profile}.properties
4. application.properties

### 12.3 Required Resources

**Runtime Requirements:**
- **Java:** JDK 17 or higher
- **Memory:** 512MB minimum (JVM heap)
- **Disk:** 
  - 100MB for application
  - Variable for file uploads (photos)
  - Variable for H2 database file
- **Network:**
  - Port 8080 (HTTP)
  - Outbound HTTPS (for API Ninjas)

**Development Requirements:**
- Maven 3.6+
- IDE with Lombok support
- H2 Console (optional)

### 12.4 Deployment Options

#### 12.4.1 Standalone Deployment
```bash
java -jar psoft-g1.jar
```

**Characteristics:**
- Single process
- Embedded Tomcat
- File-based H2
- Local file storage

**Suitable For:** Development, demos, small deployments

#### 12.4.2 Containerized Deployment (Docker)

**Dockerfile Example:**
```dockerfile
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY target/psoft-g1.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Requirements:**
- External database (H2 file doesn't work well in containers)
- Persistent volume for photos
- Environment-based configuration

#### 12.4.3 Cloud Deployment

**Suitable Platforms:**
- AWS Elastic Beanstalk
- Azure App Service
- Google Cloud Run
- Heroku

**Modifications Needed:**
1. Replace H2 with cloud database (RDS, Azure SQL, Cloud SQL)
2. Use cloud storage for files (S3, Azure Blob, GCS)
3. Externalize configuration
4. Add health check endpoints

### 12.5 Monitoring and Observability

**Current State:** Minimal

**Available:**
- Spring Boot logging (console/file)
- H2 Console (development only)

**Missing:**
- Health check endpoints
- Metrics (CPU, memory, request rates)
- Distributed tracing
- Log aggregation

**Recommendations:**
1. Add Spring Boot Actuator
2. Integrate Prometheus for metrics
3. Add ELK stack for log aggregation
4. Implement distributed tracing (Sleuth/Zipkin)

---

## 13. Technical Debt and Issues

### 13.1 Identified Technical Debt

#### 13.1.1 Architecture-Level

1. **Inconsistent Package Naming**
   - Most modules use `infrastructure`
   - Reader module uses `infraestructure` (typo)
   - **Impact:** Low (cosmetic)
   - **Effort:** Low (simple rename)

2. **Mixed Responsibilities in Services**
   - Some services handle both business logic and orchestration
   - Example: Photo validation in controller AND service
   - **Impact:** Medium (maintainability)
   - **Effort:** Medium (refactor services)

3. **Lack of Proper DTO Layer**
   - Direct entity exposure in some places
   - Mixing concerns between API and domain
   - **Impact:** Medium (tight coupling)
   - **Effort:** Medium (introduce DTOs consistently)

#### 13.1.2 Code-Level

1. **Hardcoded Values**
   - Fine calculation logic hardcoded
   - Magic numbers in validation
   - **Impact:** Medium (flexibility)
   - **Effort:** Low (externalize to configuration)

2. **Insufficient Error Handling**
   - External API calls not wrapped in try-catch
   - File operations may throw unhandled exceptions
   - **Impact:** High (reliability)
   - **Effort:** Medium (add proper error handling)

3. **Missing Javadoc**
   - Business logic methods lack documentation
   - Complex algorithms not explained
   - **Impact:** Medium (maintainability)
   - **Effort:** Medium (add documentation)

4. **Inconsistent Null Handling**
   - Some methods return null
   - Others return Optional
   - **Impact:** Low (minor confusion)
   - **Effort:** Low (standardize on Optional)

#### 13.1.3 Data-Level

1. **Primitive Obsession**
   - Some fields should be value objects
   - Example: `commentary` as String (could be Comment VO)
   - **Impact:** Low (type safety)
   - **Effort:** Low (introduce VOs)

2. **Cascading Deletes Not Fully Specified**
   - What happens when deleting a book with lendings?
   - What happens when deleting an author with books?
   - **Impact:** High (data integrity)
   - **Effort:** Medium (define and implement cascades)

3. **Sequential Number Generation**
   - ReaderNumber and LendingNumber use YYYY/NNNN format
   - Sequential number may have race conditions
   - **Impact:** High (in concurrent environment)
   - **Effort:** Medium (use database sequences)

#### 13.1.4 Security-Level

1. **RSA Keys in Repository**
   - Private key checked into source control
   - **Impact:** CRITICAL (security breach)
   - **Effort:** Low (externalize keys)

2. **No Rate Limiting**
   - Login endpoint vulnerable to brute force
   - **Impact:** High (security)
   - **Effort:** Medium (implement rate limiting)

3. **CORS Too Permissive**
   - Allows all origins
   - **Impact:** Medium (security)
   - **Effort:** Low (restrict origins)

4. **No HTTPS Enforcement**
   - HTTP allowed in all environments
   - **Impact:** High (security in production)
   - **Effort:** Low (configuration change)

#### 13.1.5 Testing-Level

1. **Insufficient Test Coverage**
   - Mostly unit tests for model
   - Few integration tests
   - No API tests automated (only Postman)
   - **Impact:** High (quality assurance)
   - **Effort:** High (write comprehensive tests)

2. **No Performance Tests**
   - No load testing
   - No stress testing
   - **Impact:** Medium (unknown performance characteristics)
   - **Effort:** Medium (implement performance test suite)

### 13.2 Known Bugs and Limitations

1. **H2 Database Limitations**
   - File mode requires no concurrent access
   - TCP mode requires H2 Console running
   - Not production-ready
   - **Resolution:** Migrate to production database

2. **File Upload Size**
   - Max 200MB per request (configured)
   - Max 20KB per photo (configured)
   - Inconsistent limits
   - **Resolution:** Clarify and align limits

3. **Concurrent Lending Creation**
   - Two librarians may lend same book simultaneously
   - Optimistic locking on Lending doesn't prevent this
   - Need optimistic locking or checking on Book
   - **Resolution:** Add book availability check in transaction

4. **Reader Registration**
   - No email verification
   - Anyone can register
   - **Resolution:** Add email verification flow

### 13.3 Missing Features

1. **Password Reset**
   - No "forgot password" functionality
   - **Impact:** High (usability)

2. **Email Notifications**
   - No reminder emails for due dates
   - No overdue notifications
   - **Impact:** Medium (user experience)

3. **Audit Logging**
   - No comprehensive audit trail
   - **Impact:** High (compliance, debugging)

4. **Reporting**
   - Limited reporting capabilities
   - No export functionality
   - **Impact:** Medium (business requirements)

5. **Book Reservation**
   - Cannot reserve books that are lent out
   - **Impact:** Medium (user experience)

6. **Multiple Copies**
   - System assumes one copy per book
   - Cannot handle multiple copies of same ISBN
   - **Impact:** High (business requirement)

### 13.4 Refactoring Opportunities

1. **Extract Configuration Class**
   - Move magic numbers to configuration
   - Create `LibraryProperties` class
   - **Benefit:** Easier configuration management

2. **Introduce Domain Services**
   - Move complex business logic from entities
   - Example: Fine calculation to `FineCalculationService`
   - **Benefit:** Better separation of concerns

3. **Implement Repository Pattern Consistently**
   - Some repositories are interfaces, some have implementations
   - Standardize approach
   - **Benefit:** Consistency

4. **Consolidate Photo Management**
   - Photo logic scattered across entities and services
   - Create `PhotoManagementService`
   - **Benefit:** Single responsibility

5. **Introduce Command/Query Separation (CQRS Light)**
   - Separate read and write models
   - Optimize queries independently
   - **Benefit:** Performance and clarity

---

## Conclusion

The Library Management System represents a well-structured Spring Boot application with clear architectural patterns and separation of concerns. The system successfully implements core library operations with appropriate security and data integrity measures.

**Strengths:**
- Clean layered architecture
- Modular design by business capability
- Strong domain model with value objects
- Comprehensive REST API
- JWT-based security
- Optimistic locking for concurrency

**Areas for Improvement:**
- Production database required (replace H2)
- Enhanced security (externalize keys, rate limiting)
- Comprehensive testing
- Better error handling and monitoring
- Scalability considerations (file storage, number generation)
- Complete feature set (multiple copies, reservations)

**Recommended Next Steps:**
1. **Immediate:** Fix security vulnerabilities (RSA keys, CORS, HTTPS)
2. **Short-term:** Migrate to production database, add monitoring
3. **Medium-term:** Implement missing features, comprehensive testing
4. **Long-term:** Scalability enhancements, microservices consideration

The system provides a solid foundation for a library management solution and demonstrates good software engineering practices. With the recommended improvements, it can evolve into a production-ready system capable of serving real-world library operations.

---

**Document Prepared By:** System Analysis  
**Date:** October 27, 2025  
**Version:** 1.0
