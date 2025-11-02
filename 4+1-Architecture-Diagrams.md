# 4+1 Architectural View Model
## Library Management System - Complete Diagrams

**System:** Library Management System (PSOFT-G1)  
**Date:** October 27, 2025  
**Architecture Model:** 4+1 View (Philippe Kruchten)

---

## Table of Contents

1. [Logical View](#1-logical-view)
   - [Level 1: System Context](#level-1-system-context)
   - [Level 2: Module Structure](#level-2-module-structure)
   - [Level 3: Class Relationships](#level-3-class-relationships)
2. [Process View](#2-process-view)
   - [Level 1: System Processes](#level-1-system-processes)
   - [Level 2: Component Interactions](#level-2-component-interactions)
   - [Level 3: Detailed Sequence Flows](#level-3-detailed-sequence-flows)
3. [Development View](#3-development-view)
   - [Level 1: Layer Architecture](#level-1-layer-architecture)
   - [Level 2: Package Organization](#level-2-package-organization)
   - [Level 3: Module Dependencies](#level-3-module-dependencies)
4. [Physical View](#4-physical-view)
   - [Level 1: Deployment Overview](#level-1-deployment-overview)
   - [Level 2: Infrastructure Components](#level-2-infrastructure-components)
   - [Level 3: Detailed Deployment](#level-3-detailed-deployment)
5. [Scenarios/Use Cases](#5-scenarios-use-cases)
   - [Level 1: Key Use Cases](#level-1-key-use-cases)
   - [Level 2: Detailed Use Case Flows](#level-2-detailed-use-case-flows)
   - [Level 3: End-to-End Scenarios](#level-3-end-to-end-scenarios)

---

## 1. Logical View

The Logical View describes the system's functionality from the end-user perspective, showing key abstractions and their relationships.

### Level 1: System Context

**High-level system context showing main functional areas and external actors.**

```mermaid
graph TB
    subgraph "Library Management System"
        AUTH[Authentication & Authorization]
        AUTHOR[Author Management]
        BOOK[Book Management]
        READER[Reader Management]
        LENDING[Lending Management]
        GENRE[Genre Management]
    end
    
    LIBRARIAN[Librarian]
    READER_USER[Reader User]
    ANONYMOUS[Anonymous User]
    EXT_API[External API<br/>API Ninjas]
    
    LIBRARIAN -->|Full Access| AUTH
    LIBRARIAN -->|Manage| AUTHOR
    LIBRARIAN -->|Manage| BOOK
    LIBRARIAN -->|Manage| READER
    LIBRARIAN -->|Manage| LENDING
    LIBRARIAN -->|Manage| GENRE
    
    READER_USER -->|Login| AUTH
    READER_USER -->|View Profile| READER
    READER_USER -->|Browse| BOOK
    READER_USER -->|View Lendings| LENDING
    
    ANONYMOUS -->|Register| READER
    ANONYMOUS -->|View Catalog| BOOK
    
    AUTHOR -.->|Quote Service| EXT_API
    
    style AUTH fill:#e1f5ff
    style AUTHOR fill:#fff4e1
    style BOOK fill:#ffe1f5
    style READER fill:#e1ffe1
    style LENDING fill:#f5e1ff
    style GENRE fill:#ffffcc
```

### Level 2: Module Structure

**Detailed view of core modules showing their internal components and relationships.**

```mermaid
graph TB
    subgraph "Author Management"
        AUTHOR_API[Author Controller]
        AUTHOR_SRV[Author Service]
        AUTHOR_DOM[Author Entity]
        AUTHOR_REPO[Author Repository]
        AUTHOR_DTO[Author DTOs/Views]
    end
    
    subgraph "Book Management"
        BOOK_API[Book Controller]
        BOOK_SRV[Book Service]
        BOOK_DOM[Book Entity]
        BOOK_REPO[Book Repository]
        BOOK_DTO[Book DTOs/Views]
    end
    
    subgraph "Reader Management"
        READER_API[Reader Controller]
        READER_SRV[Reader Service]
        READER_DOM[Reader Entity]
        READER_REPO[Reader Repository]
        READER_DTO[Reader DTOs/Views]
    end
    
    subgraph "Lending Management"
        LENDING_API[Lending Controller]
        LENDING_SRV[Lending Service]
        LENDING_DOM[Lending Entity]
        LENDING_REPO[Lending Repository]
        LENDING_DTO[Lending DTOs/Views]
    end
    
    subgraph "Shared Components"
        PHOTO_SRV[Photo Service]
        FILE_STORE[File Storage Service]
        VALUE_OBJ[Value Objects]
    end
    
    subgraph "Security Layer"
        JWT[JWT Service]
        AUTH_SRV[Auth Service]
        USER_MGT[User Management]
    end
    
    AUTHOR_API --> AUTHOR_SRV
    AUTHOR_SRV --> AUTHOR_DOM
    AUTHOR_SRV --> AUTHOR_REPO
    AUTHOR_API --> AUTHOR_DTO
    
    BOOK_API --> BOOK_SRV
    BOOK_SRV --> BOOK_DOM
    BOOK_SRV --> BOOK_REPO
    BOOK_API --> BOOK_DTO
    
    READER_API --> READER_SRV
    READER_SRV --> READER_DOM
    READER_SRV --> READER_REPO
    READER_API --> READER_DTO
    
    LENDING_API --> LENDING_SRV
    LENDING_SRV --> LENDING_DOM
    LENDING_SRV --> LENDING_REPO
    LENDING_API --> LENDING_DTO
    
    BOOK_DOM -.->|references| AUTHOR_DOM
    LENDING_DOM -.->|references| BOOK_DOM
    LENDING_DOM -.->|references| READER_DOM
    
    AUTHOR_SRV --> PHOTO_SRV
    BOOK_SRV --> PHOTO_SRV
    READER_SRV --> PHOTO_SRV
    
    PHOTO_SRV --> FILE_STORE
    
    AUTHOR_DOM --> VALUE_OBJ
    BOOK_DOM --> VALUE_OBJ
    READER_DOM --> VALUE_OBJ
    
    AUTHOR_API --> JWT
    BOOK_API --> JWT
    READER_API --> JWT
    LENDING_API --> JWT
    
    JWT --> AUTH_SRV
    AUTH_SRV --> USER_MGT
```

### Level 3: Class Relationships

**Detailed class diagram showing domain entities, value objects, and key relationships.**

```mermaid
classDiagram
    class Author {
        -Long id
        -Long version
        -AuthorNumber authorNumber
        -Name name
        -Bio bio
        -Photo photo
        -List~Book~ books
        +addBook(Book)
        +removeBook(Book)
        +applyPatch(EditAuthorRequest)
    }
    
    class Book {
        -Long id
        -Long version
        -Isbn isbn
        -Title title
        -Description description
        -Genre genre
        -Photo photo
        -List~Author~ authors
        -List~Lending~ lendings
        +addAuthor(Author)
        +removeAuthor(Author)
        +isAvailable() boolean
        +lendTo(Reader) Lending
    }
    
    class Reader {
        -Long id
        -Long version
        -ReaderNumber readerNumber
        -Name name
        -BirthDate birthDate
        -PhoneNumber phoneNumber
        -Photo photo
        -Boolean gdprConsent
        -List~Lending~ lendings
        -List~String~ interestList
        -User user
        +canBorrow() boolean
        +hasOverdueBooks() boolean
        +getTotalFines() double
    }
    
    class Lending {
        -Long id
        -Long version
        -LendingNumber lendingNumber
        -Book book
        -Reader reader
        -LocalDate startDate
        -Integer daysUntilReturn
        -LocalDate returnedDate
        -String commentary
        -LocalDate dueDate
        -Fine fine
        +isOverdue() boolean
        +returned(LocalDate, String)
        +calculateFine() Fine
        +getDaysDelayed() int
    }
    
    class Genre {
        -Long pk
        -String genre
        +valueOf(String) Genre
    }
    
    class User {
        -Long id
        -Long version
        -Username username
        -Password password
        -Name fullName
        -Set~Role~ authorities
        +isEnabled() boolean
    }
    
    class AuthorNumber {
        <<Value Object>>
        -String number
        +generate() AuthorNumber
    }
    
    class ReaderNumber {
        <<Value Object>>
        -String number
        +generate(int) ReaderNumber
    }
    
    class LendingNumber {
        <<Value Object>>
        -String number
        +generate(int) LendingNumber
    }
    
    class Isbn {
        <<Value Object>>
        -String isbn
        +validate(String)
    }
    
    class Title {
        <<Value Object>>
        -String title
        +validate(String)
    }
    
    class Name {
        <<Value Object>>
        -String name
        +validate(String)
    }
    
    class Photo {
        <<Value Object>>
        -String photoURI
        -String photoFile
        +validate(MultipartFile)
    }
    
    class Fine {
        <<Value Object>>
        -double amount
        +calculate(int, LocalDate) Fine
    }
    
    class Bio {
        <<Value Object>>
        -String bio
        +validate(String)
    }
    
    Author "1" -- "*" Book : co-authors
    Book "1" -- "*" Lending : lent via
    Reader "1" -- "*" Lending : borrows
    Book "*" -- "1" Genre : categorized by
    Reader "1" -- "1" User : has account
    
    Author ..> AuthorNumber : uses
    Author ..> Name : uses
    Author ..> Bio : uses
    Author ..> Photo : uses
    
    Book ..> Isbn : uses
    Book ..> Title : uses
    Book ..> Photo : uses
    
    Reader ..> ReaderNumber : uses
    Reader ..> Name : uses
    Reader ..> Photo : uses
    
    Lending ..> LendingNumber : uses
    Lending ..> Fine : calculates
    
    User ..> Name : uses
```

---

## 2. Process View

The Process View describes the runtime behavior, concurrency, and communication aspects of the system.

### Level 1: System Processes

**High-level process view showing main runtime components and their interactions.**

```mermaid
graph LR
    subgraph "Client Layer"
        WEB[Web Browser]
        MOBILE[Mobile App]
        API_CLIENT[API Client]
    end
    
    subgraph "Application Server"
        REST[REST API Layer]
        AUTH_PROC[Auth Process]
        BIZ_PROC[Business Logic Process]
        DATA_PROC[Data Access Process]
    end
    
    subgraph "Data Storage"
        DB[(H2 Database)]
        FILE_SYS[File System<br/>Photo Storage]
    end
    
    subgraph "External Services"
        EXT[API Ninjas<br/>Quote Service]
    end
    
    WEB -->|HTTPS/JSON| REST
    MOBILE -->|HTTPS/JSON| REST
    API_CLIENT -->|HTTPS/JSON| REST
    
    REST -->|JWT Validation| AUTH_PROC
    REST -->|Process Request| BIZ_PROC
    
    BIZ_PROC -->|Read/Write| DATA_PROC
    DATA_PROC -->|JDBC| DB
    DATA_PROC -->|I/O| FILE_SYS
    
    BIZ_PROC -.->|HTTP| EXT
```

### Level 2: Component Interactions

**Detailed process interactions showing concurrent operations and message flows.**

```mermaid
sequenceDiagram
    participant Client
    participant Gateway as API Gateway
    participant Auth as Auth Filter
    participant Controller
    participant Service
    participant Repository
    participant Database
    participant FileStore as File Storage
    
    Note over Client,FileStore: Concurrent Request Handling
    
    par Request 1: Create Book
        Client->>Gateway: POST /books (JWT)
        Gateway->>Auth: Validate Token
        Auth->>Controller: Authorized Request
        Controller->>Service: createBook(dto)
        Service->>Repository: save(book)
        Repository->>Database: INSERT
        Database-->>Repository: book entity
        Repository-->>Service: saved book
        Service-->>Controller: book view
        Controller-->>Client: 201 Created
    and Request 2: Upload Photo
        Client->>Gateway: POST /books/{isbn}/photo
        Gateway->>Auth: Validate Token
        Auth->>Controller: Authorized Request
        Controller->>Service: uploadPhoto(file)
        Service->>FileStore: store(file)
        FileStore-->>Service: file path
        Service->>Repository: update photo
        Repository->>Database: UPDATE
        Database-->>Repository: success
        Repository-->>Service: updated book
        Service-->>Controller: success
        Controller-->>Client: 200 OK
    end
    
    Note over Service,Database: Optimistic Locking
    Service->>Repository: save(entity, version)
    Repository->>Database: UPDATE WHERE version=?
    alt Version Conflict
        Database-->>Repository: 0 rows updated
        Repository-->>Service: OptimisticLockException
        Service-->>Controller: ConflictException
        Controller-->>Client: 409 Conflict
    else Success
        Database-->>Repository: 1 row updated
        Repository-->>Service: success
    end
```

### Level 3: Detailed Sequence Flows

**Detailed sequence diagrams for critical business processes.**

#### 3.1 Book Lending Process with Concurrency Control

```mermaid
sequenceDiagram
    participant L1 as Librarian 1
    participant L2 as Librarian 2
    participant API as REST API
    participant LendSrv as Lending Service
    participant BookSrv as Book Service
    participant ReaderSrv as Reader Service
    participant BookRepo as Book Repository
    participant LendRepo as Lending Repository
    participant DB as Database
    
    Note over L1,DB: Concurrent Lending Scenario
    
    par Librarian 1 attempts to lend book
        L1->>API: POST /lendings {book, reader, days}
        API->>LendSrv: create(request)
        LendSrv->>BookSrv: getBookByIsbn(isbn)
        BookSrv->>BookRepo: findByIsbn(isbn)
        BookRepo->>DB: SELECT (version=5)
        DB-->>BookRepo: Book(version=5)
        BookRepo-->>BookSrv: Book(version=5)
        BookSrv-->>LendSrv: Book(version=5)
        
        LendSrv->>LendSrv: Check if available
        Note over LendSrv: Book shows available
        
        LendSrv->>ReaderSrv: canBorrow(reader)
        ReaderSrv-->>LendSrv: true
        
        LendSrv->>LendSrv: Create Lending entity
        LendSrv->>LendRepo: save(lending)
        
    and Librarian 2 attempts to lend same book
        L2->>API: POST /lendings {same book, different reader}
        API->>LendSrv: create(request)
        LendSrv->>BookSrv: getBookByIsbn(isbn)
        BookSrv->>BookRepo: findByIsbn(isbn)
        BookRepo->>DB: SELECT (version=5)
        DB-->>BookRepo: Book(version=5)
        BookRepo-->>BookSrv: Book(version=5)
        BookSrv-->>LendSrv: Book(version=5)
        
        LendSrv->>LendSrv: Check if available
        Note over LendSrv: Book shows available (race condition)
        
        LendSrv->>ReaderSrv: canBorrow(reader)
        ReaderSrv-->>LendSrv: true
        
        LendSrv->>LendSrv: Create Lending entity
        LendSrv->>LendRepo: save(lending)
    end
    
    LendRepo->>DB: BEGIN TRANSACTION
    DB-->>LendRepo: TX Started
    LendRepo->>DB: INSERT INTO lending
    DB-->>LendRepo: Lending created
    LendRepo->>DB: UPDATE book SET version=6 WHERE version=5
    
    alt L1 commits first
        DB-->>LendRepo: 1 row updated (success)
        LendRepo->>DB: COMMIT
        LendRepo-->>LendSrv: Lending created
        LendSrv-->>API: Success
        API-->>L1: 201 Created
        
        Note over L2,DB: L2's transaction continues
        LendRepo->>DB: INSERT INTO lending
        DB-->>LendRepo: Lending created
        LendRepo->>DB: UPDATE book SET version=6 WHERE version=5
        DB-->>LendRepo: 0 rows updated (version changed)
        LendRepo->>DB: ROLLBACK
        LendRepo-->>LendSrv: OptimisticLockException
        LendSrv-->>API: ConflictException
        API-->>L2: 409 Conflict - Book already lent
    end
```

#### 3.2 Fine Calculation and Book Return Process

```mermaid
sequenceDiagram
    participant Librarian
    participant API as REST API
    participant LendSrv as Lending Service
    participant FineCalc as Fine Calculator
    participant LendRepo as Lending Repository
    participant BookRepo as Book Repository
    participant DB as Database
    
    Librarian->>API: PUT /lendings/{id}/return
    API->>LendSrv: returnLending(id, commentary)
    
    LendSrv->>LendRepo: findById(id)
    LendRepo->>DB: SELECT lending
    DB-->>LendRepo: Lending entity
    LendRepo-->>LendSrv: Lending
    
    LendSrv->>LendSrv: Check if already returned
    
    alt Already Returned
        LendSrv-->>API: ConflictException
        API-->>Librarian: 409 - Already returned
    else Not Returned
        LendSrv->>FineCalc: calculateFine(lending)
        
        FineCalc->>FineCalc: getDaysDelayed()
        Note over FineCalc: returnDate - dueDate
        
        alt No Delay
            FineCalc-->>LendSrv: Fine(0.0)
        else Has Delay
            FineCalc->>FineCalc: calculate()
            Note over FineCalc: (daysDelayed * 0.5) + 1.0
            FineCalc-->>LendSrv: Fine(amount)
        end
        
        LendSrv->>LendSrv: Update lending
        Note over LendSrv: Set returnedDate, commentary, fine
        
        LendSrv->>LendRepo: save(lending)
        LendRepo->>DB: BEGIN TRANSACTION
        LendRepo->>DB: UPDATE lending SET returned_date=?, fine=?, version=version+1
        DB-->>LendRepo: Success
        
        LendSrv->>BookRepo: Update book availability
        BookRepo->>DB: UPDATE book SET version=version+1
        DB-->>BookRepo: Success
        
        LendRepo->>DB: COMMIT
        
        LendRepo-->>LendSrv: Updated lending
        LendSrv-->>API: LendingView with fine
        API-->>Librarian: 200 OK with fine amount
    end
```

#### 3.3 Authentication and Authorization Flow

```mermaid
sequenceDiagram
    participant Client
    participant API as REST API
    participant AuthFilter as JWT Filter
    participant AuthSrv as Auth Service
    participant JwtSrv as JWT Service
    participant UserRepo as User Repository
    participant DB as Database
    participant BCrypt as Password Encoder
    
    Note over Client,DB: Login Process
    
    Client->>API: POST /auth/login {username, password}
    API->>AuthSrv: login(credentials)
    
    AuthSrv->>UserRepo: findByUsername(username)
    UserRepo->>DB: SELECT user
    DB-->>UserRepo: User entity
    UserRepo-->>AuthSrv: User
    
    AuthSrv->>BCrypt: matches(rawPassword, encodedPassword)
    BCrypt-->>AuthSrv: boolean
    
    alt Invalid Credentials
        AuthSrv-->>API: AuthenticationException
        API-->>Client: 401 Unauthorized
    else Valid Credentials
        AuthSrv->>JwtSrv: generateToken(user)
        
        JwtSrv->>JwtSrv: Create JWT Claims
        Note over JwtSrv: Subject: username<br/>Roles: authorities<br/>Expiry: 24h
        
        JwtSrv->>JwtSrv: Sign with RSA Private Key
        JwtSrv-->>AuthSrv: JWT Token
        
        AuthSrv-->>API: JwtResponse
        API-->>Client: 200 OK {token, username, roles}
    end
    
    Note over Client,DB: Subsequent Authenticated Request
    
    Client->>API: GET /books (Authorization: Bearer {token})
    API->>AuthFilter: doFilterInternal(request)
    
    AuthFilter->>AuthFilter: Extract token from header
    AuthFilter->>JwtSrv: validateToken(token)
    
    JwtSrv->>JwtSrv: Verify signature with RSA Public Key
    JwtSrv->>JwtSrv: Check expiration
    
    alt Invalid/Expired Token
        JwtSrv-->>AuthFilter: Invalid
        AuthFilter-->>API: 401 Unauthorized
        API-->>Client: 401 Unauthorized
    else Valid Token
        JwtSrv-->>AuthFilter: Valid
        AuthFilter->>JwtSrv: extractUsername(token)
        JwtSrv-->>AuthFilter: username
        
        AuthFilter->>UserRepo: loadUserByUsername(username)
        UserRepo->>DB: SELECT user
        DB-->>UserRepo: User
        UserRepo-->>AuthFilter: UserDetails
        
        AuthFilter->>AuthFilter: Set SecurityContext
        AuthFilter->>API: Continue filter chain
        
        API->>API: Check @PreAuthorize
        Note over API: hasRole('LIBRARIAN')
        
        alt Insufficient Permissions
            API-->>Client: 403 Forbidden
        else Authorized
            API->>API: Execute business logic
            API-->>Client: 200 OK with data
        end
    end
```

---

## 3. Development View

The Development View shows the software module organization, package structure, and dependencies from a programmer's perspective.

### Level 1: Layer Architecture

**High-level layered architecture showing separation of concerns.**

```mermaid
graph TB
    subgraph "Presentation Layer"
        REST[REST Controllers]
        DTOs[DTOs & Views]
        MAPPERS[MapStruct Mappers]
        VALIDATORS[Request Validators]
    end
    
    subgraph "Application Layer"
        SERVICES[Service Interfaces]
        SERVICE_IMPL[Service Implementations]
        FACADES[Service Facades]
    end
    
    subgraph "Domain Layer"
        ENTITIES[Domain Entities]
        VALUE_OBJ[Value Objects]
        DOMAIN_RULES[Business Rules]
        DOMAIN_EVENTS[Domain Events]
    end
    
    subgraph "Infrastructure Layer"
        REPOS[Repository Interfaces]
        REPO_IMPL[JPA Implementations]
        FILE_STORAGE[File Storage]
        EXTERNAL[External API Clients]
    end
    
    subgraph "Cross-Cutting Concerns"
        SECURITY[Security Config]
        EXCEPTIONS[Exception Handlers]
        CONFIG[Application Config]
        BOOTSTRAP[Data Bootstrapping]
    end
    
    REST --> SERVICES
    DTOs -.-> ENTITIES
    MAPPERS -.-> DTOs
    MAPPERS -.-> ENTITIES
    
    SERVICES --> ENTITIES
    SERVICE_IMPL --> ENTITIES
    SERVICE_IMPL --> VALUE_OBJ
    SERVICE_IMPL --> REPOS
    
    ENTITIES --> VALUE_OBJ
    ENTITIES --> DOMAIN_RULES
    
    REPOS --> ENTITIES
    REPO_IMPL -.-> REPOS
    
    SERVICE_IMPL --> FILE_STORAGE
    SERVICE_IMPL --> EXTERNAL
    
    REST --> SECURITY
    SERVICES --> EXCEPTIONS
    
    style "Presentation Layer" fill:#e3f2fd
    style "Application Layer" fill:#fff3e0
    style "Domain Layer" fill:#f3e5f5
    style "Infrastructure Layer" fill:#e8f5e9
    style "Cross-Cutting Concerns" fill:#fce4ec
```

### Level 2: Package Organization

**Detailed package structure showing modular organization by business capability.**

```mermaid
graph TB
    subgraph "pt.psoft.g1.psoftg1"
        MAIN[PsoftG1Application]
        
        subgraph "authormanagement"
            AUTH_API[api/]
            AUTH_MODEL[model/]
            AUTH_REPO[repositories/]
            AUTH_SRV[services/]
            AUTH_INFRA[infrastructure/]
        end
        
        subgraph "bookmanagement"
            BOOK_API[api/]
            BOOK_MODEL[model/]
            BOOK_REPO[repositories/]
            BOOK_SRV[services/]
            BOOK_INFRA[infrastructure/]
        end
        
        subgraph "readermanagement"
            READER_API[api/]
            READER_MODEL[model/]
            READER_REPO[repositories/]
            READER_SRV[services/]
            READER_INFRA[infraestructure/]
        end
        
        subgraph "lendingmanagement"
            LEND_API[api/]
            LEND_MODEL[model/]
            LEND_REPO[repositories/]
            LEND_SRV[services/]
            LEND_INFRA[infrastructure/]
        end
        
        subgraph "genremanagement"
            GENRE_API[api/]
            GENRE_MODEL[model/]
            GENRE_REPO[repositories/]
            GENRE_SRV[services/]
        end
        
        subgraph "usermanagement"
            USER_API[api/]
            USER_MODEL[model/]
            USER_REPO[repositories/]
            USER_SRV[services/]
            USER_INFRA[infrastructure/]
        end
        
        subgraph "auth"
            AUTH_API2[api/AuthController]
            AUTH_SRV2[services/AuthService]
        end
        
        subgraph "shared"
            SHARED_API[api/]
            SHARED_MODEL[model/]
            SHARED_REPO[repositories/]
            SHARED_SRV[services/]
        end
        
        subgraph "configuration"
            SEC_CFG[SecurityConfig]
            API_CFG[ApiConfig]
            JPA_CFG[JpaConfig]
            NINJA_CFG[ApiNinjasConfig]
        end
        
        subgraph "exceptions"
            CONFLICT[ConflictException]
            NOT_FOUND[NotFoundException]
            LENDING_FORB[LendingForbiddenException]
            FILE_STORE_EX[FileStorageException]
            GLOBAL_HANDLER[GlobalExceptionHandler]
        end
        
        subgraph "external"
            EXT_SRV[service/ApiNinjasService]
        end
        
        subgraph "bootstrapping"
            BOOT[Bootstrapper]
            USER_BOOT[UserBootstrapper]
        end
    end
    
    MAIN --> AUTH_API
    MAIN --> BOOK_API
    MAIN --> READER_API
    MAIN --> LEND_API
    MAIN --> GENRE_API
    MAIN --> USER_API
    
    AUTH_API --> AUTH_SRV
    AUTH_SRV --> AUTH_MODEL
    AUTH_SRV --> AUTH_REPO
    
    BOOK_API --> BOOK_SRV
    BOOK_SRV --> BOOK_MODEL
    BOOK_SRV --> BOOK_REPO
    BOOK_MODEL -.->|references| AUTH_MODEL
    
    LEND_API --> LEND_SRV
    LEND_SRV --> LEND_MODEL
    LEND_SRV --> LEND_REPO
    LEND_MODEL -.->|references| BOOK_MODEL
    LEND_MODEL -.->|references| READER_MODEL
    
    AUTH_SRV --> SHARED_SRV
    BOOK_SRV --> SHARED_SRV
    READER_SRV --> SHARED_SRV
    
    AUTH_API --> SEC_CFG
    BOOK_API --> SEC_CFG
    READER_API --> SEC_CFG
    LEND_API --> SEC_CFG
    
    AUTH_SRV --> EXT_SRV
    
    style authormanagement fill:#ffe0b2
    style bookmanagement fill:#f8bbd0
    style readermanagement fill:#c5e1a5
    style lendingmanagement fill:#ce93d8
    style shared fill:#90caf9
    style configuration fill:#fff59d
```

### Level 3: Module Dependencies

**Detailed dependency diagram showing inter-module relationships and shared components.**

```mermaid
graph LR
    subgraph "Core Domain Modules"
        direction TB
        AUTHOR[Author Management<br/>────────────<br/>AuthorController<br/>AuthorService<br/>AuthorServiceImpl<br/>Author Entity<br/>AuthorRepository]
        
        BOOK[Book Management<br/>────────────<br/>BookController<br/>BookService<br/>BookServiceImpl<br/>Book Entity<br/>BookRepository]
        
        READER[Reader Management<br/>────────────<br/>ReaderController<br/>ReaderService<br/>ReaderServiceImpl<br/>Reader Entity<br/>ReaderRepository]
        
        LENDING[Lending Management<br/>────────────<br/>LendingController<br/>LendingService<br/>LendingServiceImpl<br/>Lending Entity<br/>LendingRepository]
        
        GENRE[Genre Management<br/>────────────<br/>GenreController<br/>GenreService<br/>GenreServiceImpl<br/>Genre Entity<br/>GenreRepository]
    end
    
    subgraph "Shared Infrastructure"
        direction TB
        PHOTO[Photo Service<br/>────────────<br/>PhotoService<br/>FileStorageService<br/>PhotoValidator]
        
        VO[Value Objects<br/>────────────<br/>AuthorNumber<br/>ReaderNumber<br/>LendingNumber<br/>Isbn, Title, Name<br/>Bio, Photo, Fine]
        
        SECURITY[Security<br/>────────────<br/>JwtService<br/>AuthService<br/>User Entity<br/>UserRepository]
        
        EXTERNAL[External Services<br/>────────────<br/>ApiNinjasService<br/>QuoteApiClient]
    end
    
    BOOK -->|uses| AUTHOR
    BOOK -->|references| GENRE
    LENDING -->|references| BOOK
    LENDING -->|references| READER
    
    AUTHOR -->|uses| PHOTO
    BOOK -->|uses| PHOTO
    READER -->|uses| PHOTO
    
    AUTHOR -->|uses| VO
    BOOK -->|uses| VO
    READER -->|uses| VO
    LENDING -->|uses| VO
    
    AUTHOR -->|secured by| SECURITY
    BOOK -->|secured by| SECURITY
    READER -->|secured by| SECURITY
    LENDING -->|secured by| SECURITY
    GENRE -->|secured by| SECURITY
    
    READER -->|has| SECURITY
    
    AUTHOR -->|calls| EXTERNAL
    
    style AUTHOR fill:#ffccbc
    style BOOK fill:#f48fb1
    style READER fill:#aed581
    style LENDING fill:#ba68c8
    style GENRE fill:#fff176
    style PHOTO fill:#81d4fa
    style VO fill:#80deea
    style SECURITY fill:#ffab91
    style EXTERNAL fill:#a5d6a7
```

---

## 4. Physical View

The Physical View describes the deployment of software components on hardware infrastructure.

### Level 1: Deployment Overview

**High-level deployment architecture showing system topology.**

```mermaid
graph TB
    subgraph "Client Tier"
        WEB_BROWSER[Web Browser]
        MOBILE_APP[Mobile App]
        POSTMAN[API Testing Tools]
    end
    
    subgraph "Application Tier"
        direction TB
        APP_SERVER[Spring Boot<br/>Embedded Tomcat<br/>Port: 8080]
    end
    
    subgraph "Data Tier"
        direction TB
        H2_DB[(H2 Database<br/>File: ./data/psoft-g1.mv.db<br/>Console: 8080/h2-console)]
        FILE_SYS[File System Storage<br/>./photos/]
    end
    
    subgraph "External Services"
        API_NINJAS[API Ninjas<br/>Quote Service<br/>api-ninjas.com]
    end
    
    WEB_BROWSER -->|HTTPS:8080| APP_SERVER
    MOBILE_APP -->|HTTPS:8080| APP_SERVER
    POSTMAN -->|HTTP/HTTPS:8080| APP_SERVER
    
    APP_SERVER -->|JDBC| H2_DB
    APP_SERVER -->|File I/O| FILE_SYS
    APP_SERVER -.->|REST API| API_NINJAS
    
    style "Client Tier" fill:#e3f2fd
    style "Application Tier" fill:#fff3e0
    style "Data Tier" fill:#e8f5e9
    style "External Services" fill:#fce4ec
```

### Level 2: Infrastructure Components

**Detailed infrastructure view showing runtime components and configurations.**

```mermaid
graph TB
    subgraph "Client Layer"
        BROWSER[Web Clients<br/>────────<br/>Modern Browsers<br/>CORS Enabled]
        SWAGGER[Swagger UI<br/>────────<br/>/swagger-ui.html<br/>API Documentation]
    end
    
    subgraph "Application Server"
        direction TB
        
        subgraph "Spring Boot Container"
            TOMCAT[Embedded Tomcat<br/>────────<br/>Port: 8080<br/>Max Threads: Default<br/>Connection Pool: HikariCP]
            
            SECURITY_FILTER[Security Filter Chain<br/>────────<br/>JWT Filter<br/>CORS Filter<br/>CSRF Disabled]
            
            REST_LAYER[REST Controllers<br/>────────<br/>@RestController<br/>JSON Serialization]
            
            SERVICE_LAYER[Service Layer<br/>────────<br/>@Service<br/>@Transactional]
            
            JPA_LAYER[JPA Layer<br/>────────<br/>Hibernate<br/>Entity Manager]
        end
        
        subgraph "Configuration"
            APP_PROPS[application.yml<br/>────────<br/>Server Config<br/>Database Config<br/>JWT Config<br/>File Upload Config]
            
            SECURITY_CONFIG[SecurityConfig<br/>────────<br/>JWT Secret Keys<br/>CORS Settings<br/>Auth Rules]
            
            JPA_CONFIG[JPA Config<br/>────────<br/>DDL Auto: update<br/>Show SQL: true<br/>Dialect: H2]
        end
    end
    
    subgraph "Data Layer"
        direction LR
        
        H2_FILE[(H2 Database - File Mode<br/>────────────<br/>Path: ./data/psoft-g1.mv.db<br/>Mode: FILE<br/>Auto Server: false)]
        
        H2_CONSOLE[H2 Console<br/>────────<br/>Path: /h2-console<br/>Enabled: true]
        
        FILE_STORAGE[File Storage<br/>────────<br/>Base Path: ./photos/<br/>Max Size: 20KB<br/>Types: PNG, JPG, JPEG]
    end
    
    subgraph "External Integration"
        EXT_API[API Ninjas<br/>────────<br/>Base URL: api-ninjas.com<br/>API Key: Configured<br/>Timeout: 10s]
    end
    
    BROWSER -->|HTTP/HTTPS| TOMCAT
    SWAGGER -->|HTTP/HTTPS| TOMCAT
    
    TOMCAT --> SECURITY_FILTER
    SECURITY_FILTER --> REST_LAYER
    REST_LAYER --> SERVICE_LAYER
    SERVICE_LAYER --> JPA_LAYER
    
    JPA_LAYER -->|JDBC| H2_FILE
    TOMCAT --> H2_CONSOLE
    H2_CONSOLE --> H2_FILE
    
    SERVICE_LAYER -->|I/O| FILE_STORAGE
    SERVICE_LAYER -.->|HTTPS| EXT_API
    
    APP_PROPS -.->|Config| TOMCAT
    SECURITY_CONFIG -.->|Config| SECURITY_FILTER
    JPA_CONFIG -.->|Config| JPA_LAYER
```

### Level 3: Detailed Deployment

**Comprehensive deployment diagram with all configuration details.**

```mermaid
graph TB
    subgraph "Development Environment"
        DEV_MACHINE[Developer Machine<br/>────────<br/>OS: Windows/Mac/Linux<br/>Java: 17<br/>Maven: 3.x<br/>IDE: IntelliJ/Eclipse]
    end
    
    subgraph "Application Runtime"
        direction TB
        
        subgraph "JVM Process"
            JVM[Java Virtual Machine<br/>────────<br/>Java 17<br/>Heap: -Xmx (default)<br/>Metaspace: -XX:MaxMetaspaceSize]
            
            subgraph "Spring Boot Application"
                MAIN[PsoftG1Application<br/>────────<br/>@SpringBootApplication<br/>Main Method]
                
                CONTEXT[Application Context<br/>────────<br/>Bean Factory<br/>Dependency Injection<br/>Component Scanning]
                
                TOMCAT_EMBED[Embedded Tomcat 10.x<br/>────────<br/>Port: 8080<br/>Protocol: HTTP/1.1<br/>Compression: Enabled<br/>Max Request Size: 200MB]
                
                subgraph "Web Layer"
                    DISPATCHER[Dispatcher Servlet<br/>────────<br/>URL Mapping<br/>Request Routing]
                    
                    FILTER_CHAIN[Filter Chain<br/>────────<br/>1. CORS Filter<br/>2. JWT Authentication Filter<br/>3. Authorization Filter<br/>4. Exception Handler]
                end
                
                subgraph "Security Layer"
                    SEC_CONTEXT[Security Context<br/>────────<br/>Authentication Object<br/>Granted Authorities<br/>Thread-bound]
                    
                    JWT_UTIL[JWT Utility<br/>────────<br/>RSA Key Pair<br/>Private: jwt/private_key.pem<br/>Public: jwt/public_key.pem<br/>Expiry: 24h]
                    
                    PWD_ENC[BCrypt Encoder<br/>────────<br/>Strength: 10<br/>Salt: Random]
                end
                
                subgraph "Business Layer"
                    CONTROLLERS[REST Controllers<br/>────────<br/>@RestController<br/>@RequestMapping<br/>@PreAuthorize]
                    
                    SERVICES[Service Beans<br/>────────<br/>@Service<br/>@Transactional<br/>Business Logic]
                    
                    MAPPERS[MapStruct Mappers<br/>────────<br/>DTO ↔ Entity<br/>Compile-time Generation]
                end
                
                subgraph "Data Access Layer"
                    REPO_PROXIES[Repository Proxies<br/>────────<br/>JPA Interface Impl<br/>Query Method Generation]
                    
                    ENTITY_MANAGER[Entity Manager<br/>────────<br/>Persistence Context<br/>1st Level Cache<br/>Transaction Management]
                    
                    HIKARI_POOL[HikariCP Connection Pool<br/>────────<br/>Min Connections: 10<br/>Max Connections: 10<br/>Timeout: 30s<br/>Leak Detection: 0]
                end
            end
        end
    end
    
    subgraph "Persistence Layer"
        direction LR
        
        H2_ENGINE[(H2 Database Engine<br/>────────────<br/>Version: 2.x<br/>Mode: FILE/AUTO_SERVER<br/>DB File: ./data/psoft-g1.mv.db<br/>Lock File: .lock.db<br/>Trace File: .trace.db)]
        
        H2_WEB_SERVER[H2 Web Console<br/>────────<br/>Port: 8080<br/>Context: /h2-console<br/>JDBC URL: jdbc:h2:file:./data/psoft-g1<br/>Username: sa<br/>Password: (empty)]
    end
    
    subgraph "File System"
        direction TB
        
        APP_ROOT[Application Root Directory<br/>────────<br/>./<br/>psoft-g1-project/]
        
        DATA_DIR[data/<br/>────────<br/>psoft-g1.mv.db<br/>psoft-g1.trace.db]
        
        PHOTO_DIR[photos/<br/>────────<br/>authors/<br/>books/<br/>readers/]
        
        RESOURCES[src/main/resources/<br/>────────<br/>application.yml<br/>jwt/private_key.pem<br/>jwt/public_key.pem]
        
        LOGS[logs/<br/>────────<br/>application.log<br/>spring.log]
    end
    
    subgraph "External Dependencies"
        API_NINJAS_EXT[API Ninjas Service<br/>────────<br/>URL: https://api.api-ninjas.com/v1/quotes<br/>Method: GET<br/>Auth: X-Api-Key header<br/>Response: JSON<br/>Timeout: 10s<br/>Retry: None]
    end
    
    DEV_MACHINE -->|mvn spring-boot:run| MAIN
    MAIN --> CONTEXT
    CONTEXT --> TOMCAT_EMBED
    
    TOMCAT_EMBED --> DISPATCHER
    DISPATCHER --> FILTER_CHAIN
    FILTER_CHAIN --> SEC_CONTEXT
    SEC_CONTEXT --> JWT_UTIL
    
    FILTER_CHAIN --> CONTROLLERS
    CONTROLLERS --> MAPPERS
    CONTROLLERS --> SERVICES
    SERVICES --> REPO_PROXIES
    
    REPO_PROXIES --> ENTITY_MANAGER
    ENTITY_MANAGER --> HIKARI_POOL
    HIKARI_POOL -->|JDBC Connection| H2_ENGINE
    
    TOMCAT_EMBED --> H2_WEB_SERVER
    H2_WEB_SERVER --> H2_ENGINE
    
    H2_ENGINE -.->|Persist| DATA_DIR
    SERVICES -.->|Read/Write| PHOTO_DIR
    CONTEXT -.->|Load| RESOURCES
    SERVICES -.->|Write| LOGS
    
    SERVICES -.->|HTTP Client| API_NINJAS_EXT
    
    DATA_DIR -.-> APP_ROOT
    PHOTO_DIR -.-> APP_ROOT
    LOGS -.-> APP_ROOT
    
    style "Development Environment" fill:#e1f5fe
    style "Application Runtime" fill:#fff9c4
    style "Persistence Layer" fill:#f1f8e9
    style "File System" fill:#fce4ec
    style "External Dependencies" fill:#ede7f6
```

---

## 5. Scenarios / Use Cases

The Scenarios view (the "+1") ties together all other views through use cases and user stories.

### Level 1: Key Use Cases

**High-level use case diagram showing main system functionalities.**

```mermaid
graph TB
    subgraph "Actors"
        LIBRARIAN((Librarian))
        READER((Reader))
        ANONYMOUS((Anonymous User))
        SYSTEM((System))
    end
    
    subgraph "Library Management System"
        direction TB
        
        subgraph "Authentication"
            UC_LOGIN[Login]
            UC_LOGOUT[Logout]
        end
        
        subgraph "Author Management"
            UC_CREATE_AUTHOR[Create Author]
            UC_UPDATE_AUTHOR[Update Author]
            UC_VIEW_AUTHOR[View Author Details]
            UC_LIST_AUTHORS[List Authors]
            UC_PHOTO_AUTHOR[Manage Author Photo]
        end
        
        subgraph "Book Management"
            UC_CREATE_BOOK[Create Book]
            UC_UPDATE_BOOK[Update Book]
            UC_VIEW_BOOK[View Book Details]
            UC_SEARCH_BOOKS[Search Books]
            UC_PHOTO_BOOK[Manage Book Photo]
        end
        
        subgraph "Reader Management"
            UC_REGISTER[Register Reader]
            UC_UPDATE_READER[Update Reader Profile]
            UC_VIEW_READER[View Reader Details]
            UC_LIST_READERS[List Readers]
            UC_PHOTO_READER[Manage Reader Photo]
        end
        
        subgraph "Lending Operations"
            UC_CREATE_LENDING[Create Lending]
            UC_RETURN_BOOK[Return Book]
            UC_VIEW_LENDING[View Lending Details]
            UC_LIST_LENDINGS[List Lendings]
            UC_CALC_FINE[Calculate Fine]
        end
        
        subgraph "Reporting"
            UC_AVG_LEND[Average Lending Duration]
            UC_READER_LEND[Reader Lending History]
            UC_OVERDUE[Overdue Lendings]
        end
    end
    
    LIBRARIAN -->|performs| UC_LOGIN
    LIBRARIAN -->|performs| UC_CREATE_AUTHOR
    LIBRARIAN -->|performs| UC_UPDATE_AUTHOR
    LIBRARIAN -->|performs| UC_CREATE_BOOK
    LIBRARIAN -->|performs| UC_UPDATE_BOOK
    LIBRARIAN -->|performs| UC_CREATE_LENDING
    LIBRARIAN -->|performs| UC_RETURN_BOOK
    LIBRARIAN -->|performs| UC_UPDATE_READER
    LIBRARIAN -->|performs| UC_LIST_AUTHORS
    LIBRARIAN -->|performs| UC_LIST_READERS
    LIBRARIAN -->|performs| UC_LIST_LENDINGS
    LIBRARIAN -->|views| UC_AVG_LEND
    LIBRARIAN -->|views| UC_OVERDUE
    
    READER -->|performs| UC_LOGIN
    READER -->|performs| UC_VIEW_READER
    READER -->|performs| UC_UPDATE_READER
    READER -->|performs| UC_SEARCH_BOOKS
    READER -->|performs| UC_VIEW_BOOK
    READER -->|performs| UC_VIEW_LENDING
    READER -->|views| UC_READER_LEND
    
    ANONYMOUS -->|performs| UC_REGISTER
    ANONYMOUS -->|performs| UC_SEARCH_BOOKS
    ANONYMOUS -->|performs| UC_VIEW_BOOK
    ANONYMOUS -->|performs| UC_VIEW_AUTHOR
    
    UC_RETURN_BOOK -.->|includes| UC_CALC_FINE
    UC_CREATE_LENDING -.->|requires| UC_VIEW_BOOK
    UC_CREATE_LENDING -.->|requires| UC_VIEW_READER
    
    SYSTEM -->|triggers| UC_CALC_FINE
```

### Level 2: Detailed Use Case Flows

**Detailed sequence diagrams for major use case scenarios.**

#### 2.1 Complete Book Lending Scenario

```mermaid
sequenceDiagram
    actor Librarian
    participant UI as Web Interface
    participant API as REST API
    participant AuthSrv as Auth Service
    participant BookSrv as Book Service
    participant ReaderSrv as Reader Service
    participant LendSrv as Lending Service
    participant DB as Database
    
    Note over Librarian,DB: Scenario: Librarian lends a book to a reader
    
    %% Step 1: Login
    rect rgb(200, 230, 255)
        Note over Librarian,AuthSrv: Phase 1: Authentication
        Librarian->>UI: Enter credentials
        UI->>API: POST /auth/login
        API->>AuthSrv: authenticate(username, password)
        AuthSrv->>DB: Verify credentials
        DB-->>AuthSrv: User details
        AuthSrv-->>API: JWT Token
        API-->>UI: Token + User info
        UI-->>Librarian: Login successful
    end
    
    %% Step 2: Search for book
    rect rgb(255, 230, 230)
        Note over Librarian,BookSrv: Phase 2: Find Book
        Librarian->>UI: Search for book by title
        UI->>API: GET /books?title=...
        Note over API: Include JWT in header
        API->>BookSrv: searchBooks(title)
        BookSrv->>DB: Query books
        DB-->>BookSrv: Book list
        BookSrv-->>API: BookView list
        API-->>UI: JSON response
        UI-->>Librarian: Display books
        
        Librarian->>UI: Select book (check availability)
        UI->>API: GET /books/{isbn}
        API->>BookSrv: getBookByIsbn(isbn)
        BookSrv->>DB: Find book with lendings
        DB-->>BookSrv: Book + current lendings
        BookSrv->>BookSrv: Check if available
        BookSrv-->>API: BookView with availability
        API-->>UI: Book details
        UI-->>Librarian: Show "Available for lending"
    end
    
    %% Step 3: Find reader
    rect rgb(230, 255, 230)
        Note over Librarian,ReaderSrv: Phase 3: Find Reader
        Librarian->>UI: Search for reader by name
        UI->>API: GET /readers?name=...
        API->>ReaderSrv: searchReaders(name)
        ReaderSrv->>DB: Query readers
        DB-->>ReaderSrv: Reader list
        ReaderSrv-->>API: ReaderView list
        API-->>UI: JSON response
        UI-->>Librarian: Display readers
        
        Librarian->>UI: Select reader (check eligibility)
        UI->>API: GET /readers/{readerNumber}
        API->>ReaderSrv: getReaderDetails(readerNumber)
        ReaderSrv->>DB: Find reader with lendings
        DB-->>ReaderSrv: Reader + active lendings
        ReaderSrv->>ReaderSrv: Check eligibility
        Note over ReaderSrv: No overdue books?<br/>Within lending limit?
        ReaderSrv-->>API: ReaderView with status
        API-->>UI: Reader details
        UI-->>Librarian: Show "Eligible to borrow"
    end
    
    %% Step 4: Create lending
    rect rgb(255, 255, 200)
        Note over Librarian,LendSrv: Phase 4: Create Lending
        Librarian->>UI: Specify lending days (e.g., 15)
        UI->>API: POST /lendings
        Note over UI,API: {isbn, readerNumber, daysUntilReturn: 15}
        API->>LendSrv: createLending(request)
        
        LendSrv->>BookSrv: Verify book available
        BookSrv->>DB: Check book status
        DB-->>BookSrv: Book available
        BookSrv-->>LendSrv: OK
        
        LendSrv->>ReaderSrv: Verify reader eligible
        ReaderSrv->>DB: Check reader status
        DB-->>ReaderSrv: Reader eligible
        ReaderSrv-->>LendSrv: OK
        
        LendSrv->>LendSrv: Create Lending entity
        Note over LendSrv: Generate lending number<br/>Set start date = today<br/>Calculate due date<br/>Link book & reader
        
        LendSrv->>DB: BEGIN TRANSACTION
        LendSrv->>DB: INSERT lending
        LendSrv->>DB: UPDATE book version (optimistic lock)
        LendSrv->>DB: COMMIT
        DB-->>LendSrv: Lending created
        
        LendSrv-->>API: LendingView
        API-->>UI: 201 Created + lending details
        UI-->>Librarian: Show success + lending number
        Note over UI: Display: Due date, Lending #
    end
    
    %% Step 5: Later - Return book
    rect rgb(230, 200, 255)
        Note over Librarian,LendSrv: Phase 5: Return Book (Later)
        Librarian->>UI: Return book with lending number
        UI->>API: PUT /lendings/{lendingNumber}/return
        Note over UI,API: {commentary: "Good condition"}
        
        API->>LendSrv: returnLending(lendingNumber, commentary)
        LendSrv->>DB: Find lending
        DB-->>LendSrv: Lending entity
        
        LendSrv->>LendSrv: Check if overdue
        Note over LendSrv: returnDate > dueDate?
        
        alt Book is Overdue
            LendSrv->>LendSrv: calculateFine()
            Note over LendSrv: daysDelayed * 0.5€ + 1€
            LendSrv->>DB: UPDATE lending SET fine, returnedDate
            DB-->>LendSrv: Updated
            LendSrv-->>API: LendingView with fine
            API-->>UI: Book returned + fine: X.XX€
            UI-->>Librarian: Show fine amount to collect
        else Book on Time
            LendSrv->>DB: UPDATE lending SET returnedDate
            DB-->>LendSrv: Updated
            LendSrv-->>API: LendingView
            API-->>UI: Book returned successfully
            UI-->>Librarian: Show success message
        end
    end
```

#### 2.2 Reader Self-Registration Scenario

```mermaid
sequenceDiagram
    actor User as New User
    participant UI as Web Interface
    participant API as REST API
    participant ReaderSrv as Reader Service
    participant UserSrv as User Service
    participant FileSrv as File Storage Service
    participant DB as Database
    
    Note over User,DB: Scenario: New user registers as a reader
    
    %% Step 1: Registration form
    rect rgb(200, 230, 255)
        Note over User,UI: Phase 1: Fill Registration Form
        User->>UI: Click "Register"
        UI-->>User: Display registration form
        User->>UI: Enter details
        Note over User,UI: Name, email, birthdate,<br/>phone, password, interests<br/>Accept GDPR consent
        User->>UI: Upload photo (optional)
        User->>UI: Submit form
    end
    
    %% Step 2: Create user account
    rect rgb(255, 230, 230)
        Note over UI,UserSrv: Phase 2: Create User Account
        UI->>API: POST /signup
        Note over UI,API: {username, password, fullName, email}
        
        API->>UserSrv: register(request)
        
        UserSrv->>UserSrv: Validate username unique
        UserSrv->>DB: Check username exists
        DB-->>UserSrv: Not found
        
        UserSrv->>UserSrv: Encrypt password (BCrypt)
        UserSrv->>UserSrv: Assign READER role
        
        UserSrv->>DB: INSERT user
        DB-->>UserSrv: User created
        UserSrv-->>API: User entity
    end
    
    %% Step 3: Create reader profile
    rect rgb(230, 255, 230)
        Note over API,ReaderSrv: Phase 3: Create Reader Profile
        API->>ReaderSrv: createReader(readerRequest, user)
        
        ReaderSrv->>ReaderSrv: Generate reader number
        Note over ReaderSrv: Format: YYYY/NNNN
        
        ReaderSrv->>ReaderSrv: Validate inputs
        Note over ReaderSrv: Name, birthdate, phone<br/>Age >= 12 years<br/>GDPR consent = true
        
        alt Validation Fails
            ReaderSrv-->>API: ValidationException
            API-->>UI: 400 Bad Request
            UI-->>User: Show error message
            Note over User: Fix errors and resubmit
        else Validation Passes
            ReaderSrv->>ReaderSrv: Create Reader entity
            Note over ReaderSrv: Link to User<br/>Set interests<br/>Set initial quotas
        end
    end
    
    %% Step 4: Upload photo
    rect rgb(255, 255, 200)
        Note over ReaderSrv,FileSrv: Phase 4: Process Photo (if provided)
        
        opt Photo Provided
            ReaderSrv->>ReaderSrv: Validate photo
            Note over ReaderSrv: Check: PNG/JPG/JPEG<br/>Max size: 20KB<br/>Dimensions valid
            
            alt Photo Invalid
                ReaderSrv-->>API: ValidationException
                API-->>UI: 400 Bad Request - Photo invalid
                UI-->>User: Photo error, try again
            else Photo Valid
                ReaderSrv->>FileSrv: storePhoto(file, "readers", readerNumber)
                FileSrv->>FileSrv: Generate unique filename
                FileSrv->>FileSrv: Save to ./photos/readers/
                FileSrv-->>ReaderSrv: Photo path
                ReaderSrv->>ReaderSrv: Set photo URI in Reader
            end
        end
    end
    
    %% Step 5: Save and respond
    rect rgb(230, 200, 255)
        Note over ReaderSrv,DB: Phase 5: Persist Reader
        ReaderSrv->>DB: BEGIN TRANSACTION
        ReaderSrv->>DB: INSERT reader
        DB-->>ReaderSrv: Reader saved
        ReaderSrv->>DB: COMMIT
        
        ReaderSrv-->>API: ReaderView
        API-->>UI: 201 Created + reader details
        UI-->>User: Registration successful!
        Note over UI: Display reader number<br/>Provide login instructions
        
        User->>UI: Click "Login"
        UI->>API: POST /auth/login
        Note over UI,API: Use email and password
        API-->>UI: JWT Token
        UI-->>User: Logged in, redirect to dashboard
    end
```

### Level 3: End-to-End Scenarios

**Complete end-to-end scenarios showing system-wide interactions across all architectural views.**

#### 3.1 Complex Lending Scenario with Multiple Concurrent Operations

```mermaid
sequenceDiagram
    actor L1 as Librarian 1
    actor L2 as Librarian 2
    actor R1 as Reader 1
    participant LB as Load Balancer
    participant API as REST API
    participant Auth as JWT Filter
    participant LendCtrl as Lending Controller
    participant BookCtrl as Book Controller
    participant LendSrv as Lending Service
    participant BookSrv as Book Service
    participant ReaderSrv as Reader Service
    participant Cache as Entity Manager Cache
    participant DB as H2 Database
    participant FileStore as File System
    
    Note over L1,FileStore: Scenario: Multiple concurrent operations on books and lendings
    
    %% Concurrent operations begin
    par Librarian 1: Create lending for Book A
        L1->>LB: POST /lendings {bookA, reader1, 15 days}
        LB->>API: Route request
        API->>Auth: Validate JWT
        Auth->>Auth: Extract user & roles
        Auth-->>API: LIBRARIAN role confirmed
        API->>LendCtrl: createLending(request)
        
        LendCtrl->>LendSrv: create(request)
        LendSrv->>BookSrv: getByIsbn(bookA)
        BookSrv->>Cache: Check cache
        Cache-->>BookSrv: Cache miss
        BookSrv->>DB: SELECT * FROM book WHERE isbn=? (version=3)
        DB-->>BookSrv: Book A (version=3, available)
        BookSrv->>Cache: Put in cache
        BookSrv-->>LendSrv: Book A details
        
        LendSrv->>ReaderSrv: canBorrow(reader1)
        ReaderSrv->>DB: SELECT * FROM reader, lendings
        DB-->>ReaderSrv: Reader data + active lendings
        ReaderSrv->>ReaderSrv: Check: No overdue books, within limit
        ReaderSrv-->>LendSrv: Eligible = true
        
        LendSrv->>LendSrv: Create Lending entity
        Note over LendSrv: lendingNumber: 2025/0042<br/>startDate: today<br/>dueDate: today + 15<br/>Link book & reader
        
        LendSrv->>DB: BEGIN TRANSACTION (TX1)
        LendSrv->>DB: INSERT INTO lending VALUES(...)
        
    and Librarian 2: Try to lend same Book A (race condition)
        L2->>LB: POST /lendings {bookA, reader2, 10 days}
        LB->>API: Route request
        API->>Auth: Validate JWT
        Auth-->>API: LIBRARIAN role confirmed
        API->>LendCtrl: createLending(request)
        
        LendCtrl->>LendSrv: create(request)
        LendSrv->>BookSrv: getByIsbn(bookA)
        BookSrv->>Cache: Check cache
        Note over Cache: Cache has Book A (version=3, available)
        Cache-->>BookSrv: Book A from cache
        BookSrv-->>LendSrv: Book A (appears available)
        
        LendSrv->>ReaderSrv: canBorrow(reader2)
        ReaderSrv-->>LendSrv: Eligible = true
        
        LendSrv->>LendSrv: Create Lending entity
        LendSrv->>DB: BEGIN TRANSACTION (TX2)
        LendSrv->>DB: INSERT INTO lending VALUES(...)
        
    and Reader 1: Browse books concurrently
        R1->>LB: GET /books?genre=Fiction
        LB->>API: Route request
        API->>BookCtrl: searchBooks(genre)
        BookCtrl->>BookSrv: search(criteria)
        BookSrv->>DB: SELECT * FROM book WHERE genre=?
        DB-->>BookSrv: List of books
        BookSrv-->>BookCtrl: BookView list
        BookCtrl-->>API: JSON response
        API-->>R1: Book catalog (including Book A as "available")
        Note over R1: Views outdated availability<br/>due to eventual consistency
    end
    
    %% Transaction completion
    Note over LendSrv,DB: Transaction Race Resolution
    
    rect rgb(200, 255, 200)
        Note over L1,DB: Librarian 1's transaction (TX1) commits first
        LendSrv->>DB: UPDATE book SET version=4 WHERE id=? AND version=3
        DB-->>LendSrv: 1 row updated (SUCCESS)
        LendSrv->>DB: COMMIT TX1
        DB-->>LendSrv: Transaction committed
        LendSrv->>Cache: Invalidate Book A
        LendSrv-->>LendCtrl: Lending created
        LendCtrl-->>API: 201 Created
        API-->>L1: Success + lending details
    end
    
    rect rgb(255, 200, 200)
        Note over L2,DB: Librarian 2's transaction (TX2) fails
        LendSrv->>DB: UPDATE book SET version=4 WHERE id=? AND version=3
        DB-->>LendSrv: 0 rows updated (version changed to 4)
        Note over DB: Optimistic locking violation!
        DB-->>LendSrv: OptimisticLockException
        LendSrv->>DB: ROLLBACK TX2
        LendSrv-->>LendCtrl: ConflictException
        LendCtrl-->>API: 409 Conflict
        API-->>L2: Error: Book is no longer available
        Note over L2: Must refresh and retry<br/>if still needed
    end
    
    %% Reader queries for updated info
    rect rgb(200, 200, 255)
        Note over R1,DB: Reader 1 refreshes to see updated status
        R1->>LB: GET /books/{bookA}
        LB->>API: Route request
        API->>BookCtrl: getBook(isbn)
        BookCtrl->>BookSrv: getByIsbn(bookA)
        BookSrv->>Cache: Check cache
        Cache-->>BookSrv: Cache miss (was invalidated)
        BookSrv->>DB: SELECT * FROM book WHERE isbn=?
        DB-->>BookSrv: Book A (version=4, not available - has active lending)
        BookSrv->>Cache: Update cache
        BookSrv-->>BookCtrl: BookView (unavailable)
        BookCtrl-->>API: JSON response
        API-->>R1: Book A is currently unavailable
        Note over R1: Sees correct, updated status
    end
```

#### 3.2 Complete Multi-Module Scenario: From Book Creation to Return with Fine

```mermaid
sequenceDiagram
    actor Librarian
    actor Reader
    participant API as REST API
    participant AuthSrv as Auth Service
    participant AuthorSrv as Author Service
    participant BookSrv as Book Service
    participant ReaderSrv as Reader Service
    participant LendSrv as Lending Service
    participant PhotoSrv as Photo Service
    participant ExtAPI as API Ninjas
    participant DB as Database
    participant FileStore as File System
    
    Note over Librarian,FileStore: Complete Scenario: Book lifecycle from creation to lending and return
    
    %% Phase 1: Create Author with photo and bio from external API
    rect rgb(230, 240, 255)
        Note over Librarian,ExtAPI: Phase 1: Create Author
        Librarian->>API: POST /authors {name, bio}
        API->>AuthSrv: Validate JWT
        AuthSrv-->>API: Authorized (LIBRARIAN)
        
        API->>AuthorSrv: createAuthor(request)
        AuthorSrv->>AuthorSrv: Generate author number
        
        AuthorSrv->>ExtAPI: GET /quotes (for bio inspiration)
        ExtAPI-->>AuthorSrv: Famous quote
        AuthorSrv->>AuthorSrv: Append quote to bio
        
        AuthorSrv->>DB: INSERT author
        DB-->>AuthorSrv: Author created (id=101)
        AuthorSrv-->>API: AuthorView
        API-->>Librarian: 201 Created {authorNumber: "2025-000042"}
        
        Librarian->>API: POST /authors/2025-000042/photo + [photo file]
        API->>AuthorSrv: uploadPhoto(authorNumber, file)
        AuthorSrv->>PhotoSrv: validateAndStore(file, "author")
        
        PhotoSrv->>PhotoSrv: Validate size/type
        PhotoSrv->>FileStore: Write to ./photos/authors/
        FileStore-->>PhotoSrv: File path
        
        PhotoSrv-->>AuthorSrv: Photo URI
        AuthorSrv->>DB: UPDATE author SET photo=?
        DB-->>AuthorSrv: Updated
        AuthorSrv-->>API: Success
        API-->>Librarian: 200 OK - Photo uploaded
    end
    
    %% Phase 2: Create Book linked to Author
    rect rgb(255, 240, 230)
        Note over Librarian,BookSrv: Phase 2: Create Book
        Librarian->>API: POST /books
        Note over Librarian,API: {isbn, title, genre,<br/>authors: ["2025-000042"]}
        
        API->>BookSrv: createBook(request)
        BookSrv->>AuthorSrv: Verify authors exist
        AuthorSrv->>DB: SELECT authors WHERE number IN (...)
        DB-->>AuthorSrv: Author list (id=101)
        AuthorSrv-->>BookSrv: Authors found
        
        BookSrv->>BookSrv: Create Book entity
        BookSrv->>BookSrv: Link authors
        BookSrv->>DB: INSERT book
        DB-->>BookSrv: Book created
        BookSrv->>DB: INSERT book_authors (book_id, author_id)
        DB-->>BookSrv: Relationship created
        
        BookSrv-->>API: BookView
        API-->>Librarian: 201 Created
        
        Librarian->>API: POST /books/{isbn}/photo + [cover image]
        API->>BookSrv: uploadPhoto(isbn, file)
        BookSrv->>PhotoSrv: validateAndStore(file, "book")
        PhotoSrv->>FileStore: Write to ./photos/books/
        FileStore-->>PhotoSrv: File path
        PhotoSrv-->>BookSrv: Photo URI
        BookSrv->>DB: UPDATE book SET photo=?
        DB-->>BookSrv: Updated
        BookSrv-->>API: Success
        API-->>Librarian: 200 OK
    end
    
    %% Phase 3: Reader searches and views book
    rect rgb(240, 255, 240)
        Note over Reader,BookSrv: Phase 3: Reader Discovers Book
        Reader->>API: GET /books?title=...
        Note over API: No authentication required<br/>for book search
        
        API->>BookSrv: searchBooks(criteria)
        BookSrv->>DB: SELECT books with authors and genres
        DB-->>BookSrv: Book list with relationships
        BookSrv-->>API: BookView list
        API-->>Reader: Books catalog
        
        Reader->>API: GET /books/{isbn}
        API->>BookSrv: getDetails(isbn)
        BookSrv->>DB: SELECT book, authors, genre, active lendings
        DB-->>BookSrv: Full book details
        BookSrv->>BookSrv: Calculate availability
        BookSrv-->>API: Detailed BookView
        API-->>Reader: Book details (available)
        Note over Reader: Decides to borrow
    end
    
    %% Phase 4: Create Lending
    rect rgb(255, 240, 255)
        Note over Librarian,LendSrv: Phase 4: Librarian Creates Lending
        Reader->>Librarian: Request to borrow book
        Note over Reader,Librarian: In-person interaction
        
        Librarian->>API: POST /lendings
        Note over Librarian,API: {isbn, readerNumber, daysUntilReturn: 15}
        
        API->>AuthSrv: Validate JWT
        AuthSrv-->>API: Authorized (LIBRARIAN)
        
        API->>LendSrv: createLending(request)
        
        LendSrv->>BookSrv: getByIsbn(isbn)
        BookSrv->>DB: SELECT book (version=1)
        DB-->>BookSrv: Book
        BookSrv-->>LendSrv: Book available
        
        LendSrv->>ReaderSrv: getByNumber(readerNumber)
        ReaderSrv->>DB: SELECT reader with active lendings
        DB-->>ReaderSrv: Reader details
        ReaderSrv->>ReaderSrv: Check eligibility
        Note over ReaderSrv: No overdue books<br/>Within borrowing limit
        ReaderSrv-->>LendSrv: Reader eligible
        
        LendSrv->>LendSrv: Generate lending number
        Note over LendSrv: 2025/0058
        LendSrv->>LendSrv: Create Lending entity
        Note over LendSrv: startDate: 2025-10-27<br/>dueDate: 2025-11-11<br/>daysUntilReturn: 15
        
        LendSrv->>DB: BEGIN TRANSACTION
        LendSrv->>DB: INSERT lending
        LendSrv->>DB: UPDATE book SET version=2 WHERE version=1
        DB-->>LendSrv: Book updated (optimistic lock success)
        LendSrv->>DB: COMMIT
        
        LendSrv-->>API: LendingView
        API-->>Librarian: 201 Created
        Librarian-->>Reader: Here's your book!
        Note over Reader: Due date: November 11, 2025
    end
    
    %% Phase 5: Time passes, book becomes overdue
    rect rgb(255, 230, 230)
        Note over Reader,LendSrv: Phase 5: Book Becomes Overdue (Time passes)
        Note over Reader: November 11 passes...<br/>Returns on November 16<br/>(5 days late)
        
        Reader->>API: GET /lendings/mine
        Note over Reader: Check personal lendings
        API->>AuthSrv: Validate JWT
        AuthSrv-->>API: Authorized (READER role)
        
        API->>LendSrv: getByReader(username)
        LendSrv->>DB: SELECT lendings WHERE reader.user=?
        DB-->>LendSrv: Lending list
        LendSrv->>LendSrv: Calculate overdue status
        Note over LendSrv: Check: returnedDate is null<br/>AND dueDate < today
        LendSrv-->>API: LendingView (overdue=true, daysDelayed=5)
        API-->>Reader: Your lending is 5 days overdue
        Note over Reader: Realizes must return and pay fine
    end
    
    %% Phase 6: Book Return with Fine Calculation
    rect rgb(240, 230, 255)
        Note over Librarian,LendSrv: Phase 6: Return Book and Calculate Fine
        Reader->>Librarian: Return book (5 days late)
        Librarian->>API: PUT /lendings/2025-0058/return
        Note over Librarian,API: {commentary: "Book in good condition"}
        
        API->>AuthSrv: Validate JWT
        AuthSrv-->>API: Authorized (LIBRARIAN)
        
        API->>LendSrv: returnLending(lendingNumber, commentary)
        LendSrv->>DB: SELECT lending
        DB-->>LendSrv: Lending entity
        
        LendSrv->>LendSrv: Check if already returned
        Note over LendSrv: returnedDate is null → OK
        
        LendSrv->>LendSrv: Calculate days delayed
        Note over LendSrv: returnDate(Nov 16) - dueDate(Nov 11)<br/>= 5 days
        
        LendSrv->>LendSrv: Calculate fine
        Note over LendSrv: Fine = (5 days * 0.5€) + 1€<br/>= 2.5€ + 1€ = 3.5€
        
        LendSrv->>LendSrv: Update Lending entity
        Note over LendSrv: Set returnedDate = Nov 16<br/>Set fine = 3.5€<br/>Set commentary
        
        LendSrv->>DB: BEGIN TRANSACTION
        LendSrv->>DB: UPDATE lending SET returned_date=?, fine=?, commentary=?, version=version+1
        LendSrv->>DB: UPDATE book SET version=3 WHERE version=2
        DB-->>LendSrv: Both updates successful
        LendSrv->>DB: COMMIT
        
        LendSrv-->>API: LendingView with fine
        API-->>Librarian: 200 OK {fine: 3.50, returnDate: "2025-11-16"}
        Librarian-->>Reader: Please pay fine of €3.50
        Note over Reader: Pays fine, transaction complete
    end
    
    %% Phase 7: Reporting and Analytics
    rect rgb(230, 255, 255)
        Note over Librarian,DB: Phase 7: Generate Reports
        Librarian->>API: GET /lendings/average/2025/11
        API->>LendSrv: getAverageLendingDuration(year, month)
        LendSrv->>DB: SELECT AVG(returned_date - start_date)
        Note over DB: WHERE YEAR(start_date)=2025<br/>AND MONTH(start_date)=11<br/>AND returned_date IS NOT NULL
        DB-->>LendSrv: Average: 12.5 days
        LendSrv-->>API: AverageLendingView
        API-->>Librarian: Average lending: 12.5 days
        
        Librarian->>API: GET /lendings?overdue=true
        API->>LendSrv: getOverdueJendings()
        LendSrv->>DB: SELECT lendings
        Note over DB: WHERE returned_date IS NULL<br/>AND due_date < CURRENT_DATE
        DB-->>LendSrv: Overdue lending list
        LendSrv-->>API: LendingView list
        API-->>Librarian: List of overdue lendings
        Note over Librarian: Takes action on overdue items
    end
```

---

## Summary

This document provides a comprehensive 4+1 architectural view model for the Library Management System with three levels of depth for each view:

### View Coverage

1. **Logical View** - Shows the system's conceptual structure, domain model, and key abstractions
   - Level 1: System context with main modules
   - Level 2: Detailed module structure with components
   - Level 3: Complete class diagram with relationships

2. **Process View** - Describes runtime behavior, concurrency, and communication
   - Level 1: High-level process overview
   - Level 2: Component interactions with sequence flows
   - Level 3: Detailed concurrent scenarios with transactions

3. **Development View** - Shows the code organization and module dependencies
   - Level 1: Layered architecture
   - Level 2: Package organization by business capability
   - Level 3: Detailed module dependencies

4. **Physical View** - Describes deployment and infrastructure
   - Level 1: Simple deployment topology
   - Level 2: Infrastructure components with configurations
   - Level 3: Complete deployment with all details

5. **Scenarios (Use Cases)** - The "+1" that ties everything together
   - Level 1: Key use case overview
   - Level 2: Detailed use case flows
   - Level 3: Complex end-to-end scenarios

### Key Architectural Patterns Identified

- **Layered Architecture** with clear separation of concerns
- **Domain-Driven Design** influences with rich domain model
- **Repository Pattern** for data access abstraction
- **RESTful API** design
- **JWT-based Security** with role-based access control
- **Optimistic Locking** for concurrency control
- **Value Objects** for type safety
- **Service Layer** for business logic encapsulation

### Technology Highlights

- Spring Boot 3.2.5 ecosystem
- JPA/Hibernate for ORM
- H2 Database (development)
- MapStruct for DTO mapping
- BCrypt for password encryption
- RSA JWT tokens

---

**Document Version:** 1.0  
**Date:** October 27, 2025  
**Architecture Model:** 4+1 View (Kruchten)  
**System:** Library Management System (PSOFT-G1)
