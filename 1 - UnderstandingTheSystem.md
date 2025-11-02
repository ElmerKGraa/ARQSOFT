# Understanding the Library Management System Architecture

**A Comprehensive Guide for Developers New to Software Architecture**

**Target Audience**: Developers with programming experience but limited software architecture knowledge
**Purpose**: Understand the system's architecture, components, and design decisions to explain and justify to instructors

---

## Table of Contents

1. [Introduction: From Code to Architecture](#introduction-from-code-to-architecture)
2. [What is Software Architecture?](#what-is-software-architecture)
3. [Understanding Layered Architecture](#understanding-layered-architecture)
4. [The Big Picture: System Overview](#the-big-picture-system-overview)
5. [Deep Dive: How a Request Flows Through the System](#deep-dive-how-a-request-flows-through-the-system)
6. [Understanding Each Component](#understanding-each-component)
7. [Key Architectural Concepts Explained](#key-architectural-concepts-explained)
8. [Why These Design Decisions Were Made](#why-these-design-decisions-were-made)
9. [How to Navigate the Codebase](#how-to-navigate-the-codebase)
10. [Justifying Your Work to Your Teacher](#justifying-your-work-to-your-teacher)
11. [Common Questions and Answers](#common-questions-and-answers)

---

## Introduction: From Code to Architecture

### What You Already Know (Programming)

As a programmer, you understand:
- **Variables and data types**: `String name`, `int age`
- **Functions/methods**: `public void doSomething()`
- **Classes and objects**: Creating instances, calling methods
- **Control flow**: if/else, loops, exceptions
- **Data structures**: Lists, Maps, Sets

### What's Different About Architecture?

Architecture is about **the big picture**:
- **Not "what does this line do?"** but **"why is this organized this way?"**
- **Not "how do I write this method?"** but **"where should this logic live?"**
- **Not "what's the syntax?"** but **"how do these 144 files work together?"**

### The Analogy: Building a House vs. Building a City

**Programming** = Building a single room
- You focus on details: where the door goes, paint color, furniture placement

**Architecture** = Urban planning for a city
- You focus on: how neighborhoods connect, where roads go, zoning (residential vs. commercial)

Your Library Management System is like a small city with 7 neighborhoods (modules), and we need to understand how they're organized and why.

---

## What is Software Architecture?

### Definition (Simple)

**Software Architecture** = The high-level structure of a software system, showing:
1. What are the **major components** (big pieces)?
2. How do they **connect and communicate**?
3. What **rules and patterns** govern the system?

### Why Architecture Matters

Imagine trying to find a book in a library where:
- Books are randomly scattered on shelves
- No classification system (no genres, no Dewey Decimal)
- No catalog or index

You'd eventually find the book, but it would take hours!

**Good architecture** = Organized library where:
- Books sorted by genre (classification)
- Clear sections and signs (structure)
- Catalog system (navigation)
- Rules about where things go (patterns)

### Architecture vs. Code

| Aspect | Code Level | Architecture Level |
|--------|------------|-------------------|
| **Focus** | Individual methods, classes | Modules, layers, components |
| **Questions** | "How does this work?" | "Why is it organized this way?" |
| **Scale** | 10-100 lines | 1000-10,000 lines |
| **Time** | Minutes to hours | Days to weeks |
| **Analogy** | Individual LEGO bricks | LEGO instruction manual |

---

## Understanding Layered Architecture

### What is Layered Architecture?

Your system uses **3-layer architecture** (also called "3-tier" or "N-tier"). Think of it like a cake with three layers:

```
┌─────────────────────────────────────┐
│     TOP LAYER (Presentation)       │  ← Users interact here
│  "The Restaurant Menu & Waiters"   │
├─────────────────────────────────────┤
│    MIDDLE LAYER (Business Logic)   │  ← Business rules live here
│      "The Kitchen & Chefs"          │
├─────────────────────────────────────┤
│    BOTTOM LAYER (Data Access)      │  ← Database access here
│     "The Storage Room & Pantry"    │
└─────────────────────────────────────┘
```

### Layer 1: Presentation Layer (API Controllers)

**What it does**: Handles communication with the outside world (clients, browsers, apps)

**Real-world analogy**: Restaurant waiters
- Take orders from customers (receive HTTP requests)
- Validate orders (check input)
- Pass orders to kitchen (call business layer)
- Bring food to customers (return HTTP responses)

**Code location**: `api/` packages
- Example: [AuthorController.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)

**What you'll see**:
```java
@RestController
@RequestMapping("/api/authors")
public class AuthorController {

    @PostMapping
    public AuthorView createAuthor(@Valid @RequestBody CreateAuthorRequest request) {
        // 1. Receive HTTP POST request
        // 2. Validate input (Spring does this automatically with @Valid)
        // 3. Call business layer (service)
        // 4. Convert result to JSON response
        // 5. Return to client
    }
}
```

**Key concepts**:
- `@RestController`: "This class handles web requests"
- `@PostMapping`: "This method handles POST requests"
- `@RequestBody`: "Read data from HTTP request body"
- Returns `AuthorView` (a DTO - Data Transfer Object) not the entity

---

### Layer 2: Business Logic Layer (Services)

**What it does**: Contains all business rules and domain logic

**Real-world analogy**: Restaurant kitchen
- Receives orders from waiters (from API layer)
- Follows recipes (business rules)
- Uses ingredients (calls data layer)
- Prepares dishes (processes data)
- Ensures quality (validation)

**Code location**: `services/` packages
- Example: [AuthorServiceImpl.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)

**What you'll see**:
```java
@Service
public class AuthorServiceImpl implements AuthorService {
    private final AuthorRepository authorRepository;
    private final ForbiddenNameService forbiddenNameService;

    public Author create(CreateAuthorRequest request) {
        // 1. Check business rules (e.g., name not forbidden)
        if (forbiddenNameService.isForbidden(request.getName())) {
            throw new IllegalArgumentException("Name is forbidden");
        }

        // 2. Create domain object (Author entity)
        Author author = new Author(request.getName(), request.getBio());

        // 3. Save to database (through repository)
        return authorRepository.save(author);

        // Notice: NO HTTP stuff here! Pure business logic.
    }
}
```

**Key concepts**:
- `@Service`: "This class contains business logic"
- Uses `Repository` to access data (doesn't know about SQL)
- Enforces business rules (forbidden names, validation)
- Works with domain entities (`Author`, `Book`, etc.)

---

### Layer 3: Data Access Layer (Repositories)

**What it does**: Handles all database operations

**Real-world analogy**: Restaurant storage room
- Stores ingredients (data)
- Retrieves items when chef asks (queries)
- Keeps inventory organized (database schema)
- Knows where everything is (indexes, keys)

**Code location**: `repositories/` packages
- Example: [AuthorRepository.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/repositories/AuthorRepository.java)

**What you'll see**:
```java
public interface AuthorRepository {
    Author save(Author author);
    Optional<Author> findById(Long id);
    List<Author> findByName(String name);
    void delete(Author author);
}
```

**Key concepts**:
- Interface-based (implementation hidden)
- Spring Data JPA provides implementation automatically
- No SQL in your code (JPA generates it)
- Returns domain entities (`Author`), not database rows

---

### Why Layers? The Rules

**Rule 1: Layers Only Talk Downward**

```
┌─────────────┐
│ Controller  │ ✅ Can call Service
└──────┬──────┘    ❌ Cannot call Repository directly
       │
       ▼
┌─────────────┐
│  Service    │ ✅ Can call Repository
└──────┬──────┘    ❌ Cannot call Controller
       │
       ▼
┌─────────────┐
│ Repository  │ ✅ Can access Database
└─────────────┘    ❌ Cannot call Service or Controller
```

**Why?** Separation of concerns. Each layer has ONE job.

**Rule 2: Each Layer Uses Different Types**

| Layer | Type Used | Purpose |
|-------|-----------|---------|
| **Controller** | DTOs (AuthorView, CreateAuthorRequest) | Data transfer over network |
| **Service** | Entities (Author, Book) | Domain objects with business logic |
| **Repository** | Entities (Author, Book) | Persistence objects |

**Why?**
- DTOs optimized for JSON (flat, simple)
- Entities optimized for business logic (methods, validation)
- Don't expose internal structure to clients

**Rule 3: No Business Logic in Controllers**

❌ **Bad** (business logic in controller):
```java
@PostMapping
public AuthorView createAuthor(@RequestBody CreateAuthorRequest request) {
    if (request.getName().contains("badword")) {  // ❌ Business rule in controller!
        throw new IllegalArgumentException("Bad name");
    }
    // ...
}
```

✅ **Good** (business logic in service):
```java
// Controller - just coordinates
@PostMapping
public AuthorView createAuthor(@RequestBody CreateAuthorRequest request) {
    Author author = authorService.create(request);  // ✅ Delegates to service
    return mapper.toView(author);
}

// Service - contains business logic
public Author create(CreateAuthorRequest request) {
    if (forbiddenNameService.isForbidden(request.getName())) {  // ✅ Business rule here
        throw new IllegalArgumentException("Forbidden name");
    }
    // ...
}
```

---

## The Big Picture: System Overview

### The 7 Modules (Neighborhoods in Our City)

Your system has **7 domain modules**. Think of each as a neighborhood with its own purpose:

```
Library Management System City
├── Author District      (manages writers)
├── Book District        (manages books)
├── Reader District      (manages patrons)
├── Lending District     (manages borrowing)
├── Genre District       (manages categories)
├── User District        (manages accounts)
└── Auth District        (security checkpoint)
```

Each module has the **same internal structure** (3 layers):

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

**This consistency is intentional!** Once you understand one module, you understand all of them.

---

### Module Relationships: How They Connect

Modules don't work in isolation. Here's how they depend on each other:

```
                  ┌────────────┐
                  │   User     │
                  │ Management │
                  └──────┬─────┘
                         │
           ┌─────────────┴─────────────┐
           │                           │
     ┌─────▼─────┐              ┌─────▼──────┐
     │  Reader   │              │ Librarian  │
     │Management │              │ (implied)  │
     └─────┬─────┘              └─────┬──────┘
           │                           │
           │                           ├─────────────┐
           │                           │             │
     ┌─────▼─────┐         ┌──────────▼──┐    ┌─────▼──────┐
     │  Lending  │────────▶│    Book     │◀───│   Author   │
     │Management │         │ Management  │    │Management  │
     └───────────┘         └──────┬──────┘    └────────────┘
                                  │
                           ┌──────▼──────┐
                           │    Genre    │
                           │ Management  │
                           └─────────────┘
```

**Key relationships**:
1. **User → Reader**: A Reader is a type of User (inheritance)
2. **Book → Genre**: Every Book has exactly one Genre (many-to-one)
3. **Book → Author**: Books have one or more Authors (many-to-many)
4. **Lending → Book + Reader**: A Lending connects a Book to a Reader (association)

---

### Cross-Cutting Concerns: The Utilities

Some components are used by **all modules** (like city utilities: water, electricity):

```
┌──────────────────────────────────────┐
│      Shared Infrastructure           │
├──────────────────────────────────────┤
│  • FileStorageService (photo uploads)│
│  • ForbiddenNameService (validation) │
│  • GlobalExceptionHandler (errors)   │
│  • ApiNinjasService (external API)   │
└──────────────────────────────────────┘
         ▲    ▲    ▲    ▲    ▲
         │    │    │    │    │
    ┌────┴────┴────┴────┴────┴────┐
    │   All Modules Use These     │
    └─────────────────────────────┘
```

**Security Infrastructure** (the city police):

```
┌──────────────────────────────────────┐
│       Security Infrastructure        │
├──────────────────────────────────────┤
│  • SecurityConfig (rules)            │
│  • JWT Authentication (identity)     │
│  • RBAC (role checking)              │
└──────────────────────────────────────┘
         │
         └─────▶ Protects all API endpoints
```

---

## Deep Dive: How a Request Flows Through the System

Let's trace **one complete request** from start to finish. This will show you how all the pieces fit together.

### Example: Creating a New Author

**User action**: Librarian clicks "Create Author" button in web app, enters name and bio.

#### Step 1: HTTP Request Arrives (Client → Server)

```http
POST /api/authors HTTP/1.1
Host: localhost:8080
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "name": "Isaac Asimov",
  "bio": "Science fiction author, known for Foundation series"
}
```

**What just happened?**
- Client sent POST request to `/api/authors` endpoint
- Included JWT token for authentication
- JSON body contains author data

---

#### Step 2: Security Layer Intercepts (Before Controller)

**File**: [SecurityConfig.java:110-172](../../src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L110-L172)

```java
// Spring Security checks BEFORE request reaches controller
.requestMatchers(HttpMethod.POST, "/api/authors").hasRole(Role.LIBRARIAN)
```

**Security checks**:
1. ✅ Is JWT token valid? (signature, expiration)
2. ✅ Does user have LIBRARIAN role?
3. ✅ Is token not expired?

**Result**:
- ✅ Pass → Continue to controller
- ❌ Fail → Return 401 Unauthorized or 403 Forbidden (request stops here)

---

#### Step 3: Controller Receives Request (Presentation Layer)

**File**: [AuthorController.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)

```java
@RestController
@RequestMapping("/api/authors")
public class AuthorController {

    private final AuthorService authorService;  // Injected by Spring
    private final AuthorViewMapper mapper;      // Injected by Spring

    @PostMapping  // Handles POST /api/authors
    public AuthorView createAuthor(@Valid @RequestBody CreateAuthorRequest request) {
        // 1. Spring already validated @Valid annotations
        // 2. Call business layer
        final Author author = authorService.create(request);

        // 3. Convert entity to DTO (for JSON response)
        return mapper.toAuthorView(author);
    }
}
```

**What happens here?**
1. **Deserialization**: Spring converts JSON → `CreateAuthorRequest` object
2. **Validation**: Spring checks `@NotNull`, `@Size`, etc. on `CreateAuthorRequest`
   - If invalid → Return 400 Bad Request (stops here)
3. **Delegation**: Controller calls `authorService.create()`
4. **No business logic**: Controller is just a "traffic cop"

---

#### Step 4: Service Executes Business Logic (Business Layer)

**File**: [AuthorServiceImpl.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)

```java
@Service
@RequiredArgsConstructor  // Lombok: generates constructor for final fields
public class AuthorServiceImpl implements AuthorService {

    private final AuthorRepository authorRepository;
    private final ForbiddenNameService forbiddenNameService;
    private final FileStorageService fileStorageService;

    @Transactional  // Database transaction starts here
    public Author create(CreateAuthorRequest request) {
        // Step 4.1: Validate business rules
        String name = request.getName();
        if (forbiddenNameService.isForbidden(name)) {
            throw new IllegalArgumentException("Author name is forbidden");
        }

        // Step 4.2: Handle photo upload (if provided)
        String photoUri = null;
        if (request.getPhoto() != null) {
            photoUri = fileStorageService.storeFile(request.getPhoto());
        }

        // Step 4.3: Create domain entity (business object)
        Author author = new Author(
            request.getName(),
            request.getBio(),
            photoUri
        );

        // Step 4.4: Persist to database (through repository)
        return authorRepository.save(author);
    }
    // Transaction commits here (if no exception)
}
```

**What happens here?**
1. **Business rules**: Check forbidden names (e.g., "Voldemort" might be forbidden)
2. **File handling**: Upload photo to file system if provided
3. **Domain creation**: Create `Author` entity (with validation in constructor)
4. **Persistence**: Save to database
5. **Transaction**: All or nothing (if photo upload fails, no database save)

---

#### Step 4.5: Domain Entity Validates Itself

**File**: [Author.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java)

```java
@Entity  // JPA: This is a database table
public class Author extends EntityWithPhoto {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long authorNumber;  // Auto-generated ID

    @Version
    private long version;  // For optimistic locking (concurrency)

    @Embedded
    private Name name;  // Value object

    @Embedded
    private Bio bio;    // Value object

    public Author(String name, String bio, String photoURI) {
        setName(name);      // Validates name
        setBio(bio);        // Validates bio
        setPhotoInternal(photoURI);  // From parent class
    }

    private void setName(String name) {
        // Domain validation happens HERE, not in controller
        this.name = new Name(name);  // Name constructor validates
    }

    private void setBio(String bio) {
        this.bio = new Bio(bio);  // Bio constructor validates
    }

    // Notice: Business logic lives in the entity
    // This is "rich domain model" (DDD principle)
}
```

**What happens here?**
1. **Validation**: Name and Bio value objects validate in their constructors
2. **Encapsulation**: Can't create invalid Author (no setters are public)
3. **Self-contained**: Author knows its own rules

---

#### Step 5: Repository Saves to Database (Data Layer)

**File**: [AuthorRepository.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/repositories/AuthorRepository.java)

```java
public interface AuthorRepository {
    Author save(Author author);
    Optional<Author> findById(Long id);
    List<Author> findByName(String name);
    // Spring Data JPA generates implementation automatically
}
```

**What happens here?**
1. **No SQL written**: Spring Data JPA generates SQL automatically
2. **Generated SQL** (you don't write this):
   ```sql
   INSERT INTO AUTHOR (PK, AUTHOR_NUMBER, NAME, BIO, PHOTO_URI, VERSION)
   VALUES (NULL, NULL, 'Isaac Asimov', 'Science fiction author...', '/uploads/photo.jpg', 0);
   ```
3. **Return**: Returns saved Author with generated ID

---

#### Step 6: Service Returns to Controller

Back to [AuthorServiceImpl.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java):

```java
// Method completes, returns Author entity
return authorRepository.save(author);
// Transaction commits (changes saved to database)
```

**Author entity** now has:
- `authorNumber`: 42 (generated by database)
- `name`: "Isaac Asimov"
- `bio`: "Science fiction author..."
- `photoUri`: "/uploads/authors/photo.jpg"
- `version`: 0 (for optimistic locking)

---

#### Step 7: Controller Converts Entity to DTO

Back to [AuthorController.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java):

```java
@PostMapping
public AuthorView createAuthor(@Valid @RequestBody CreateAuthorRequest request) {
    final Author author = authorService.create(request);  // Just completed

    // Convert Author entity → AuthorView DTO
    return mapper.toAuthorView(author);
}
```

**Mapper** converts internal entity to external DTO:

```java
public class AuthorViewMapper {
    public AuthorView toAuthorView(Author author) {
        return new AuthorView(
            author.getId(),           // 42
            author.getName(),         // "Isaac Asimov"
            author.getBio(),          // "Science fiction author..."
            author.getPhotoUri()      // "/uploads/authors/photo.jpg"
            // Notice: version NOT exposed to client (internal detail)
        );
    }
}
```

**Why convert?**
- **Security**: Don't expose internal fields (like `version`)
- **Flexibility**: Can change entity without breaking API
- **Simplification**: DTOs are flat, entities might be complex

---

#### Step 8: HTTP Response Returned (Server → Client)

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Isaac Asimov",
  "bio": "Science fiction author, known for Foundation series",
  "photoUri": "/uploads/authors/photo.jpg"
}
```

**Spring does**:
1. Serializes `AuthorView` → JSON (Jackson library)
2. Sets HTTP status 200 OK
3. Sends response to client

---

### Complete Flow Diagram

```
┌─────────────┐
│   Client    │ POST /api/authors + JWT token
│  (Browser)  │ { "name": "Isaac Asimov", ... }
└──────┬──────┘
       │ HTTP Request
       ▼
┌─────────────────────────────────────────┐
│     Spring Security Filter Chain        │
│  ✓ Validate JWT token                   │
│  ✓ Check user has LIBRARIAN role        │
└──────┬──────────────────────────────────┘
       │ Authorized ✓
       ▼
┌─────────────────────────────────────────┐
│  PRESENTATION LAYER                     │
│  AuthorController.createAuthor()        │
│  • Receive request                      │
│  • Validate input (@Valid)              │
└──────┬──────────────────────────────────┘
       │ Delegate to business layer
       ▼
┌─────────────────────────────────────────┐
│  BUSINESS LAYER                         │
│  AuthorServiceImpl.create()             │
│  • Check forbidden names                │
│  • Upload photo (FileStorageService)    │
│  • Create Author entity                 │
└──────┬──────────────────────────────────┘
       │ Call repository
       ▼
┌─────────────────────────────────────────┐
│  DOMAIN MODEL                           │
│  Author constructor                     │
│  • Validate name (Name value object)    │
│  • Validate bio (Bio value object)      │
└──────┬──────────────────────────────────┘
       │ Valid entity created
       ▼
┌─────────────────────────────────────────┐
│  DATA LAYER                             │
│  AuthorRepository.save()                │
│  • Generate SQL (Spring Data JPA)       │
│  • Execute INSERT statement             │
└──────┬──────────────────────────────────┘
       │ Return saved entity
       ▼
┌─────────────────────────────────────────┐
│  H2 Database                            │
│  AUTHOR table: New row inserted         │
│  authorNumber = 42 (auto-generated)     │
└──────┬──────────────────────────────────┘
       │ Return to service
       ▼
┌─────────────────────────────────────────┐
│  BUSINESS LAYER (return)                │
│  Transaction commits                    │
└──────┬──────────────────────────────────┘
       │ Return to controller
       ▼
┌─────────────────────────────────────────┐
│  PRESENTATION LAYER (return)            │
│  AuthorViewMapper.toAuthorView()        │
│  • Convert Author → AuthorView DTO      │
│  • Serialize to JSON                    │
└──────┬──────────────────────────────────┘
       │ HTTP Response
       ▼
┌─────────────┐
│   Client    │ { "id": 42, "name": "Isaac Asimov", ... }
│  (Browser)  │ Status: 200 OK
└─────────────┘
```

**Total time**: ~50-200ms (depending on photo upload)

---

## Understanding Each Component

### Component 1: Controllers (API Layer)

**Purpose**: Handle HTTP requests and responses

**Responsibilities**:
1. Map URLs to methods (`@RequestMapping`, `@GetMapping`, etc.)
2. Deserialize JSON → Java objects
3. Validate input (`@Valid`)
4. Call business layer (service)
5. Convert results to DTOs
6. Serialize to JSON
7. Set HTTP status codes

**What they DON'T do**:
- ❌ No business logic
- ❌ No database access
- ❌ No calculations or algorithms

**Example patterns**:

```java
// POST - Create
@PostMapping
public AuthorView createAuthor(@RequestBody CreateAuthorRequest request) { }

// GET - Read one
@GetMapping("/{id}")
public AuthorView getAuthor(@PathVariable Long id) { }

// GET - Read many
@GetMapping
public ListResponse<AuthorView> searchAuthors(@RequestParam String name) { }

// PATCH - Update
@PatchMapping("/{id}")
public AuthorView updateAuthor(@PathVariable Long id,
                                @RequestBody UpdateAuthorRequest request) { }

// DELETE - Delete
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteAuthor(@PathVariable Long id) { }
```

---

### Component 2: Services (Business Layer)

**Purpose**: Implement business logic and orchestrate operations

**Responsibilities**:
1. Enforce business rules (forbidden names, age limits, etc.)
2. Coordinate multiple operations (upload photo + save author)
3. Manage transactions (`@Transactional`)
4. Call repositories for data access
5. Call other services if needed
6. Handle domain exceptions

**What they DON'T do**:
- ❌ No HTTP concerns (no status codes, no JSON)
- ❌ No SQL (use repositories)
- ❌ No UI logic

**Example pattern**:

```java
@Service
@RequiredArgsConstructor
public class AuthorServiceImpl implements AuthorService {

    private final AuthorRepository repository;
    private final OtherService otherService;  // Can depend on other services

    @Transactional  // Database transaction
    public Author create(CreateAuthorRequest request) {
        // 1. Validate business rules
        validateBusinessRules(request);

        // 2. Create domain entity
        Author author = new Author(...);

        // 3. Save via repository
        return repository.save(author);
    }

    private void validateBusinessRules(CreateAuthorRequest request) {
        // Business logic here
    }
}
```

---

### Component 3: Repositories (Data Layer)

**Purpose**: Abstract database access

**Responsibilities**:
1. CRUD operations (Create, Read, Update, Delete)
2. Custom queries
3. Generate SQL automatically (Spring Data JPA)

**What they DON'T do**:
- ❌ No business logic
- ❌ No transactions (service handles that)
- ❌ No validation

**Example pattern**:

```java
public interface AuthorRepository {
    // Basic CRUD (provided by Spring Data JPA)
    Author save(Author author);
    Optional<Author> findById(Long id);
    void delete(Author author);

    // Custom queries (Spring generates SQL from method name)
    List<Author> findByName(String name);
    List<Author> findByNameContaining(String keyword);

    // Complex queries (use @Query annotation)
    @Query("SELECT a FROM Author a WHERE SIZE(a.books) > :minBooks")
    List<Author> findProlificAuthors(@Param("minBooks") int minBooks);
}
```

**Spring Data JPA magic**: You write interface, Spring provides implementation!

---

### Component 4: Domain Entities

**Purpose**: Represent business concepts with behavior

**Responsibilities**:
1. Hold data (fields)
2. Enforce invariants (validation)
3. Contain business logic (methods)
4. Map to database tables (JPA annotations)

**Example**:

```java
@Entity
public class Author {
    @Id
    @GeneratedValue
    private Long id;

    @Embedded
    private Name name;  // Value object (validated)

    @Version
    private Long version;  // Optimistic locking

    // Constructor enforces invariants
    public Author(String name, String bio) {
        setName(name);  // Validates
        setBio(bio);    // Validates
    }

    // Business method
    public void updateBio(String newBio) {
        this.bio = new Bio(newBio);  // Validates
    }

    // No public setters! Controlled modification only.
}
```

**Key principle**: **Rich Domain Model**
- Entities are not just data bags
- They contain behavior and enforce rules
- This is Domain-Driven Design (DDD)

---

### Component 5: Value Objects

**Purpose**: Represent immutable domain concepts

**Characteristics**:
1. **Immutable**: Once created, cannot change
2. **Validated**: Constructor ensures validity
3. **Compared by value**: Two Names with same string are equal
4. **No identity**: No ID field

**Example**: [Name.java](../../src/main/java/pt/psoft/g1/psoftg1/shared/model/Name.java)

```java
@Embeddable  // Stored in parent entity's table
public class Name {
    private String name;

    // Constructor validates
    public Name(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        if (name.length() > 150) {
            throw new IllegalArgumentException("Name too long");
        }
        this.name = name;
    }

    // Getter only, no setter (immutable)
    public String toString() {
        return name;
    }

    // Once created, cannot be changed
    // To "change" name, create new Name object
}
```

**Why use value objects?**
- Validation centralized in one place
- Can't create invalid Name
- Reusable across entities (Author, Reader both have Names)

---

### Component 6: DTOs (Data Transfer Objects)

**Purpose**: Transfer data between layers or over network

**Characteristics**:
1. **Simple**: Just data, no behavior
2. **Flat structure**: Easy to serialize to JSON
3. **Tailored**: Different DTOs for different use cases

**Types of DTOs**:

1. **Request DTOs** (input):
   ```java
   public class CreateAuthorRequest {
       @NotBlank
       private String name;

       @Size(max = 4096)
       private String bio;

       private MultipartFile photo;  // File upload

       // Getters only (Spring sets values via reflection)
   }
   ```

2. **View DTOs** (output):
   ```java
   public class AuthorView {
       private Long id;
       private String name;
       private String bio;
       private String photoUri;

       // Getters and setters (for JSON serialization)
   }
   ```

3. **Specialized DTOs** (analytics):
   ```java
   public class AuthorBookCountDTO {
       private String authorName;
       private Long bookCount;
   }
   ```

---

### Component 7: Mappers

**Purpose**: Convert between entities and DTOs

**Why needed?** Entity ≠ DTO
- Entity has version, timestamps, relationships
- DTO is simplified for client

**Example**:

```java
@Component
public class AuthorViewMapper {

    // Entity → DTO (for response)
    public AuthorView toAuthorView(Author author) {
        return new AuthorView(
            author.getId(),
            author.getName(),
            author.getBio(),
            author.getPhotoUri()
        );
    }

    // List of entities → List of DTOs
    public List<AuthorView> toAuthorViewList(List<Author> authors) {
        return authors.stream()
                      .map(this::toAuthorView)
                      .collect(Collectors.toList());
    }
}
```

---

## Key Architectural Concepts Explained

### Concept 1: Dependency Injection (DI)

**What is it?** Spring creates objects and "injects" dependencies automatically.

**Without DI** (manual instantiation):
```java
public class AuthorController {
    private AuthorService service = new AuthorServiceImpl();  // ❌ Manual creation
    // Problem: Controller now tightly coupled to AuthorServiceImpl
    // Can't easily test, can't swap implementations
}
```

**With DI** (Spring does it):
```java
@RestController
public class AuthorController {
    private final AuthorService service;  // Interface, not implementation

    @Autowired  // or use @RequiredArgsConstructor (Lombok)
    public AuthorController(AuthorService service) {
        this.service = service;  // Spring injects implementation
    }

    // ✅ Controller doesn't know which implementation
    // ✅ Easy to test (inject mock)
    // ✅ Can change implementation without touching controller
}
```

**Benefits**:
- Loose coupling
- Easy testing (inject mocks)
- Configuration-driven (Spring decides what to inject)

---

### Concept 2: Interface-Based Programming

**What is it?** Depend on interfaces, not concrete classes.

**Example**:
```java
// Service interface (contract)
public interface AuthorService {
    Author create(CreateAuthorRequest request);
    Author update(Long id, UpdateAuthorRequest request);
    Optional<Author> findById(Long id);
}

// Implementation 1 (production)
@Service
public class AuthorServiceImpl implements AuthorService {
    // Real database access
}

// Implementation 2 (testing)
public class MockAuthorService implements AuthorService {
    // Fake in-memory data
}

// Controller depends on interface
public class AuthorController {
    private final AuthorService service;  // Interface!

    // Spring injects AuthorServiceImpl in production
    // Tests inject MockAuthorService
}
```

**Benefits**:
- Can swap implementations
- Testing made easy
- Follows SOLID principles (Dependency Inversion)

---

### Concept 3: Domain-Driven Design (DDD)

**What is it?** Organize code around business domain concepts, not technical concerns.

**Key ideas**:

1. **Ubiquitous Language**: Code uses same terms as business
   - Business says "Author", code has `Author` class
   - Business says "Lending", code has `Lending` class
   - [Glossary.md](../GlobalArtifacts/Glossary.md) documents terms

2. **Aggregates**: Clusters of related objects treated as a unit
   - **Author Aggregate**: Author (root) + Bio + Photo
   - **Book Aggregate**: Book (root) + ISBN + Title + Authors
   - **Lending Aggregate**: Lending (root) + Fine + Book + Reader
   - See: [Docs/GlobalArtifacts/Aggregates/](../GlobalArtifacts/Aggregates/)

3. **Value Objects**: Immutable, validated domain concepts
   - ISBN, Title, Name, Email, Password
   - See: [Docs/GlobalArtifacts/ValueObjects/](../GlobalArtifacts/ValueObjects/)

4. **Entities**: Objects with identity (ID)
   - Author, Book, Lending, Reader
   - Can be tracked over time

5. **Repositories**: Abstraction for data access
   - Pretend to be "collection of entities in memory"
   - Hide database details

**Why DDD?**
- Code reflects business reality
- Easier for domain experts to understand
- Changes in business map directly to code changes

---

### Concept 4: Optimistic Locking

**What is it?** Handle concurrent updates without database locks.

**The Problem**:
```
Time    User A                          User B
t1      Read Book (version=5)
t2                                      Read Book (version=5)
t3      Change title
t4                                      Change description
t5      Save Book (version=6)
t6                                      Save Book (version=6) ❌ CONFLICT!
```

Without optimistic locking, User B would overwrite User A's changes (lost update).

**The Solution**:
```java
@Entity
public class Book {
    @Version
    private Long version;  // Automatically managed by JPA

    public void applyPatch(Long expectedVersion, UpdateBookRequest request) {
        if (!Objects.equals(this.version, expectedVersion)) {
            throw new StaleObjectStateException("Someone else modified this book");
        }
        // Apply changes
    }
}
```

**How it works**:
1. When reading Book, client gets `version=5`
2. When updating, client sends `version=5`
3. Server checks: is current version still 5?
   - ✅ Yes → Save with `version=6`
   - ❌ No → Reject (someone else updated first)

**Why?**
- Prevents lost updates
- Better performance than pessimistic locks (no database locking)
- User sees clear error: "Book was changed, please refresh"

---

### Concept 5: JWT Authentication

**What is it?** Token-based authentication (no server-side sessions).

**Traditional session-based**:
```
1. User logs in → Server creates session, stores in memory
2. Server returns session ID in cookie
3. Each request sends cookie → Server looks up session
Problem: Server must remember all sessions (memory, clustering issues)
```

**JWT token-based**:
```
1. User logs in → Server generates JWT token (signed)
2. Server returns token to client
3. Client sends token in Authorization header
4. Server validates token signature (no lookup needed)
Benefit: Stateless! Server doesn't store anything
```

**JWT Structure**:
```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header (algorithm)
eyJzdWIiOiJqb2huZG9lIiwicm9sZXMiOlsiTE... ← Payload (user info, roles)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature (tamper-proof)
```

**Payload contains**:
```json
{
  "sub": "johndoe",              // username
  "roles": ["LIBRARIAN"],        // user roles
  "exp": 1698883200              // expiration timestamp
}
```

**Why JWT?**
- Stateless (scales horizontally)
- No session storage needed
- Token contains all needed information
- Cryptographically signed (RSA) - can't be forged

**Implementation**: [SecurityConfig.java](../../src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java)

---

### Concept 6: Repository Pattern

**What is it?** Treat data access like an in-memory collection.

**Conceptual model**:
```java
// Imagine repositories are just lists in memory
List<Author> authors = new ArrayList<>();

// Add
authors.add(new Author("Asimov", "Sci-fi writer"));

// Find
Optional<Author> found = authors.stream()
    .filter(a -> a.getId().equals(42))
    .findFirst();

// Remove
authors.removeIf(a -> a.getId().equals(42));
```

**Actual repository**:
```java
public interface AuthorRepository {
    Author save(Author author);                // Like authors.add()
    Optional<Author> findById(Long id);        // Like authors.stream().filter()
    void delete(Author author);                // Like authors.remove()
}
```

**Behind the scenes**: Spring Data JPA converts to SQL.

**Benefits**:
- Business layer doesn't know about SQL
- Easy to test (use in-memory implementation)
- Can switch databases without changing code

---

## Why These Design Decisions Were Made

### Decision 1: Layered Architecture

**Question**: Why layers? Why not put everything in one place?

**Answer**: **Separation of concerns**

**Without layers** (everything in one class):
```java
public class AuthorHandler {
    public String createAuthor(HttpServletRequest request) {
        // 1. Parse JSON from HTTP request
        String json = readBody(request);

        // 2. Validate input
        if (!isValidName(json.get("name"))) { return "error"; }

        // 3. Check business rules
        if (isForbiddenName(json.get("name"))) { return "error"; }

        // 4. Build SQL query
        String sql = "INSERT INTO AUTHOR (name, bio) VALUES (?, ?)";

        // 5. Execute SQL
        executeSQL(sql, json.get("name"), json.get("bio"));

        // 6. Format response
        return "{\"id\": 42, \"name\": \"" + json.get("name") + "\"}";
    }
}
```

**Problems**:
- ❌ Can't test business logic without HTTP
- ❌ Can't change database without touching HTTP code
- ❌ Can't reuse business logic in different endpoints
- ❌ Mixing 4 different concerns in one method
- ❌ Hard to understand and maintain

**With layers**:
```java
// Layer 1: HTTP concerns only
@RestController
public class AuthorController {
    public AuthorView createAuthor(@RequestBody CreateAuthorRequest request) {
        return service.create(request);  // Delegate!
    }
}

// Layer 2: Business logic only
@Service
public class AuthorService {
    public Author create(CreateAuthorRequest request) {
        validateBusinessRules(request);
        return repository.save(new Author(...));
    }
}

// Layer 3: Database only
public interface AuthorRepository {
    Author save(Author author);
}
```

**Benefits**:
- ✅ Each layer has ONE responsibility
- ✅ Can test business logic without HTTP or database
- ✅ Can change database (PostgreSQL instead of H2) without touching business logic
- ✅ Can reuse business logic in CLI, batch jobs, etc.
- ✅ Easier to understand (each class is simple)

---

### Decision 2: JWT with RSA (Instead of Sessions)

**Question**: Why JWT tokens? Why not traditional sessions?

**Comparison**:

| Aspect | Session-Based | JWT-Based (Our Choice) |
|--------|---------------|------------------------|
| **Storage** | Server stores sessions | Server stores nothing |
| **Scalability** | Hard (session replication) | Easy (stateless) |
| **Performance** | Session lookup on every request | Just validate signature |
| **Clustering** | Need shared session store | No coordination needed |
| **Logout** | Easy (delete session) | Hard (token valid until expiration) |

**Why JWT wins for this project**:
1. **Educational**: Learn modern authentication
2. **Scalable**: Can run multiple servers without session sharing
3. **RESTful**: REST should be stateless
4. **Industry standard**: Most new APIs use tokens

**Why RSA (not HMAC)**:
- RSA uses public/private key pair
- Public key can be shared (for validation)
- Private key kept secret (for signing)
- More secure for distributed systems

**Files**:
- Private key: `rsa.private.key` (signs tokens)
- Public key: `rsa.public.key` (validates tokens)

---

### Decision 3: H2 Database (Instead of PostgreSQL/MySQL)

**Question**: Why H2 embedded database?

**Comparison**:

| Database | Setup | Performance | Production-Ready |
|----------|-------|-------------|------------------|
| **H2** (our choice) | Zero setup | Fast (in-memory) | No |
| **PostgreSQL** | Install server | Fast | Yes |
| **MySQL** | Install server | Fast | Yes |

**Why H2 for this project**:
1. **Zero setup**: Works out of the box, no installation
2. **Development friendly**: Fast restart, easy to reset
3. **Testing**: Can use in-memory mode for tests
4. **Educational**: Focus on code, not database admin
5. **Portable**: Database is just a file

**H2 Modes**:
```properties
# TCP Mode - can use H2 Console
spring.datasource.url=jdbc:h2:tcp://localhost/~/psoft-g1

# File Mode - just a file, no console
spring.datasource.url=jdbc:h2:~/psoft-g1

# In-Memory Mode - super fast, data lost on restart
spring.datasource.url=jdbc:h2:mem:testdb
```

**For production**: Migrate to PostgreSQL (JPA makes this easy!)

---

### Decision 4: Domain-Driven Design (DDD)

**Question**: Why DDD? Why not just simple CRUD?

**Simple CRUD** (anemic model):
```java
// Just data, no behavior
public class Author {
    public Long id;
    public String name;
    public String bio;
    // All public, no validation
}

// Logic in service (away from data)
public class AuthorService {
    public void create(String name, String bio) {
        if (name == null || name.length() > 150) {
            throw new Exception();  // Validation far from Author
        }
        Author author = new Author();
        author.name = name;  // Can set anything!
        author.bio = bio;
        repository.save(author);
    }
}
```

**Problems**:
- Can create invalid Author (no encapsulation)
- Validation logic scattered across services
- No clear business concepts

**DDD approach** (rich model):
```java
// Data + behavior + validation
public class Author {
    private Long id;
    private Name name;  // Value object (validated)
    private Bio bio;    // Value object (validated)

    public Author(String name, String bio) {
        this.name = new Name(name);  // Validates!
        this.bio = new Bio(bio);      // Validates!
    }

    // Business method
    public void changeBio(String newBio) {
        this.bio = new Bio(newBio);  // Validated
    }

    // Can't create invalid Author!
}
```

**Benefits**:
- ✅ Validation close to data
- ✅ Can't bypass validation
- ✅ Clear business concepts (Author, not just data bag)
- ✅ Easier to understand business rules

**DDD is about**: Making code match business reality.

---

### Decision 5: Optimistic Locking (Instead of Pessimistic)

**Question**: Why optimistic locking?

**Pessimistic locking** (database locks):
```sql
SELECT * FROM BOOK WHERE id = 42 FOR UPDATE;  -- Locks row
-- Other users wait here...
UPDATE BOOK SET title = 'New Title' WHERE id = 42;
COMMIT;  -- Unlocks row
```

**Problems**:
- Blocks other users (bad UX)
- Deadlock risk
- Performance impact
- Doesn't work well with stateless APIs

**Optimistic locking** (version checking):
```java
// User A reads book (version=5)
Book book = repository.findById(42);  // version=5

// User B also reads book (version=5)
Book book2 = repository.findById(42);  // version=5

// User A updates
book.setTitle("New Title");
repository.save(book);  // version becomes 6 ✅

// User B tries to update
book2.setDescription("New desc");
repository.save(book2);  // ❌ Fails! Expected version 5, found 6
// Exception: "Book was modified by someone else"
```

**Benefits**:
- ✅ No database locks (better performance)
- ✅ No blocking (better UX for concurrent users)
- ✅ No deadlocks
- ✅ Clear error message
- ✅ Works with stateless APIs

**When is conflict rare?**
- Librarians rarely edit same book simultaneously
- Risk vs. reward: occasional conflict OK for better performance

---

## How to Navigate the Codebase

### Step 1: Start with the Domain

**Always start here**: [Original/Docs/GlobalArtifacts/](../GlobalArtifacts/)

1. **Read Glossary**: [Glossary.md](../GlobalArtifacts/Glossary.md)
   - Understand business terms
   - Author, Book, Lending, Reader, Fine, Genre

2. **Study Aggregates**: [Aggregates/](../GlobalArtifacts/Aggregates/)
   - Author.md, Book.md, Lending.md, Reader.md
   - What is each aggregate responsible for?

3. **Review Value Objects**: [ValueObjects/](../GlobalArtifacts/ValueObjects/)
   - ISBN, Title, Name, Email
   - What rules do they enforce?

**Why domain first?**
- Domain is the heart of the system
- Everything else serves the domain
- Understand business before code

---

### Step 2: Pick One Module to Explore

**Recommendation**: Start with **Author Management** (simplest)

**Path**: `Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/`

**Reading order**:

1. **Domain model**: `model/Author.java`
   ```
   - What fields does Author have?
   - What value objects? (Name, Bio)
   - What methods? (applyPatch, removePhoto)
   - What validations?
   ```

2. **Repository**: `repositories/AuthorRepository.java`
   ```
   - What queries are available?
   - findById, findByName, etc.
   ```

3. **Service**: `services/AuthorServiceImpl.java`
   ```
   - What operations? (create, update, find)
   - What business rules?
   - What dependencies? (repositories, other services)
   ```

4. **Controller**: `api/AuthorController.java`
   ```
   - What endpoints? (POST, GET, PATCH, DELETE)
   - What security? (who can access?)
   - What DTOs? (request/response objects)
   ```

5. **DTOs**: `api/AuthorView.java`, `services/CreateAuthorRequest.java`
   ```
   - What data exposed to clients?
   - What validations?
   ```

---

### Step 3: Trace One Request End-to-End

**Pick a simple operation**: GET author by ID

**Trace path**:

1. **Start**: User requests `GET /api/authors/42`

2. **Controller**: [AuthorController.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
   ```java
   @GetMapping("/{authorNumber}")
   public AuthorView getAuthor(@PathVariable("authorNumber") Long authorNumber) {
       // Calls service
   }
   ```

3. **Service**: [AuthorServiceImpl.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
   ```java
   public Optional<Author> findById(Long id) {
       return authorRepository.findById(id);
   }
   ```

4. **Repository**: [AuthorRepository.java](../../src/main/java/pt/psoft/g1/psoftg1/authormanagement/repositories/AuthorRepository.java)
   ```java
   Optional<Author> findById(Long id);  // Spring generates implementation
   ```

5. **Back to Controller**: Convert to DTO and return

**Use debugger**: Set breakpoints, step through code

---

### Step 4: Understand Module Relationships

**Key relationships**:

1. **Book depends on Genre**
   - [Book.java:40-42](../../src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L40-L42)
   - Every book has one genre

2. **Book depends on Author** (many-to-many)
   - [Book.java:44-46](../../src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L44-L46)
   - Book has list of authors

3. **Lending connects Book and Reader**
   - [Lending.java:55-66](../../src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L55-L66)
   - Lending has book and reader

4. **Reader depends on User**
   - [ReaderDetails.java:23-26](../../src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L23-L26)
   - Reader extends User

**Diagram**: See [LayeredArchitecture.puml](LayeredArchitecture.puml)

---

### Step 5: Explore Security

**Path**: `Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java`

**Key sections**:

1. **Authorization rules**: Lines 110-172
   ```java
   .requestMatchers(HttpMethod.POST, "/api/authors").hasRole(Role.LIBRARIAN)
   .requestMatchers(HttpMethod.GET, "/api/authors/{id}").hasAnyRole(Role.READER, Role.LIBRARIAN)
   ```

2. **JWT configuration**: Lines 179-203
   ```java
   public JwtEncoder jwtEncoder() { }
   public JwtDecoder jwtDecoder() { }
   ```

3. **Password encoding**: Lines 206-209
   ```java
   public PasswordEncoder passwordEncoder() {
       return new BCryptPasswordEncoder();
   }
   ```

**Understand**:
- Who can access what?
- How is authentication enforced?
- Where are credentials checked?

---

### Step 6: Study Shared Infrastructure

**Path**: `Original/src/main/java/pt/psoft/g1/psoftg1/shared/`

**Key components**:

1. **FileStorageService**: Handles photo uploads
2. **ForbiddenNameService**: Validates names
3. **GlobalExceptionHandler**: Handles all errors
4. **Page, SearchRequest**: Pagination utilities

**Understand**:
- What's shared across all modules?
- How do modules use these services?

---

### Navigation Tips

**Use your IDE**:

1. **Find Usages** (Ctrl+Alt+F7 in IntelliJ)
   - Where is this class used?
   - Where is this method called?

2. **Go to Implementation** (Ctrl+Alt+B in IntelliJ)
   - Jump from interface to implementation
   - See what actually executes

3. **Call Hierarchy** (Ctrl+Alt+H in IntelliJ)
   - See call chain: who calls this method?

4. **File Structure** (Ctrl+F12 in IntelliJ)
   - See all methods in a class

5. **Search Everywhere** (Double Shift in IntelliJ)
   - Find classes, files, symbols

**Use documentation**:
- Start with [README.md](README.md) (the index)
- Read architecture diagrams
- Read functional requirements for context

---

## Justifying Your Work to Your Teacher

### What You've Documented

You've created **comprehensive architecture documentation** for a **reverse-engineered system**. Here's what you have:

---

### 1. Architecturally Significant Requirements (ASRs)

**File**: [ASR-ArchitecturallySignificantRequirements.md](ASR-ArchitecturallySignificantRequirements.md)

**What to say**:
> "I identified and documented **18 ASRs** that shaped this system's architecture. ASRs are requirements that have a profound impact on the architecture - they're the 'why' behind design decisions.
>
> For example:
> - **ASR-SEC-01 (JWT Authentication)**: I documented why JWT with RSA keys was chosen for authentication, including the quality attributes it addresses (security, performance) and the architectural scenario it supports.
> - **ASR-PERF-03 (Optimistic Locking)**: I explained how the system handles concurrent updates without database locks, including the trade-offs and measurement criteria.
>
> I also documented **5 major architectural decisions** (AD-01 through AD-05), explaining the alternatives considered and the rationale for each choice. For instance, **AD-01** justifies why layered architecture was chosen over microservices or hexagonal architecture."

**Key points for teacher**:
- Shows understanding of quality attributes (security, performance, maintainability)
- Links requirements to implementation (code references)
- Explains trade-offs and alternatives
- Uses industry-standard format (ISO/IEC 25010 quality model)

---

### 2. Requirements Documentation

**Files**:
- [FunctionalRequirements.md](FunctionalRequirements.md) - 44 functional requirements
- [NonFunctionalRequirements.md](NonFunctionalRequirements.md) - 30 non-functional requirements

**What to say**:
> "I reverse-engineered the requirements from the implemented codebase. This demonstrates understanding of:
>
> **Functional Requirements** (what the system does):
> - Extracted **44 requirements** across 7 modules
> - Each requirement includes: ID, actor, priority, REST endpoint, and implementation evidence
> - Created traceability matrix linking requirements to use cases and code
> - 100% implementation coverage (all Phase 1 and Phase 2 use cases)
>
> **Non-Functional Requirements** (quality attributes):
> - Documented **30 NFRs** using ISO/IEC 25010 quality model
> - Categories: Security, Performance, Reliability, Maintainability, Usability, Scalability, Portability, Compliance, Interoperability
> - Each NFR includes measurement criteria and implementation status
> - Provided recommendations for improvements"

**Key points for teacher**:
- Demonstrates ability to extract requirements from code
- Shows understanding of functional vs. non-functional
- Uses industry standards (ISO/IEC 25010)
- Complete traceability (requirements → code → tests)

---

### 3. Architecture Diagrams

**Files**: 4 PlantUML diagrams
- [SystemArchitecture.puml](SystemArchitecture.puml) - Component diagram
- [DeploymentArchitecture.puml](DeploymentArchitecture.puml) - Deployment view
- [ContainerArchitecture.puml](ContainerArchitecture.puml) - C4 model
- [LayeredArchitecture.puml](LayeredArchitecture.puml) - Detailed class view

**What to say**:
> "I created **4 comprehensive architecture diagrams** at different levels of abstraction:
>
> 1. **Component Diagram**: Shows the 3-layer architecture (Presentation, Business, Data) and 7 domain modules
> 2. **Deployment Diagram**: Shows how the system is deployed (application server, database, file system, external services)
> 3. **Container Diagram (C4 Model)**: High-level view of containers and their interactions
> 4. **Layered Architecture (Detailed)**: Class-level view showing all controllers, services, repositories, and entities
>
> These diagrams follow industry standards (UML, C4 Model) and are version-controlled using PlantUML text format."

**Key points for teacher**:
- Multiple views (4+1 architectural views)
- Different abstraction levels (from high-level to detailed)
- Follows standards (UML, C4)
- Maintainable (text-based PlantUML, not images)

---

### 4. Domain-Specific Documentation

**File**: [GenreClassificationSystem.md](GenreClassificationSystem.md)

**What to say**:
> "I created deep-dive documentation for the **Genre Classification System**, including:
> - Current system analysis (as-is)
> - Domain model and business rules
> - 7 API operations with examples
> - Analytics capabilities
> - **8 prioritized enhancement recommendations**
>
> This demonstrates ability to:
> - Analyze a specific subsystem in depth
> - Identify limitations in current design
> - Propose concrete, feasible enhancements
> - Prioritize improvements based on value and effort"

**Key points for teacher**:
- Shows system thinking (not just code reading)
- Critical analysis (identifying limitations)
- Solution-oriented (enhancement recommendations)
- Business value focus (why enhancements matter)

---

### The Methodology: Reverse Engineering

**What to emphasize**:

> "This documentation was created through **reverse engineering** of the implemented codebase. The methodology was:
>
> 1. **Code Analysis**: Analyzed 144 Java files to understand structure
> 2. **Pattern Recognition**: Identified architectural patterns (layered, DDD, repository)
> 3. **Relationship Mapping**: Traced dependencies and data flows
> 4. **Evidence Gathering**: Linked every claim to code references
> 5. **Documentation Synthesis**: Created comprehensive documentation with traceability
>
> This approach demonstrates:
> - Ability to understand existing systems (crucial for real-world development)
> - Architectural thinking (seeing patterns, not just code)
> - Documentation skills (clear, structured, traceable)
> - Critical analysis (identifying design decisions and trade-offs)"

---

### Academic Value

**What this demonstrates**:

1. **Software Architecture Skills**:
   - Understanding of architectural styles (layered)
   - Knowledge of design patterns (repository, service, DDD)
   - Quality attribute analysis (security, performance, etc.)

2. **Reverse Engineering Skills**:
   - Extract architecture from code
   - Identify patterns and principles
   - Document system-as-is

3. **Documentation Skills**:
   - Clear, structured writing
   - Multiple views (diagrams, text)
   - Complete traceability
   - Industry-standard formats

4. **Critical Thinking**:
   - Analyze design decisions
   - Identify trade-offs
   - Propose improvements

5. **Professional Practice**:
   - Following standards (UML, C4, ISO 25010)
   - Version control (PlantUML, Markdown)
   - Audience-aware (different documents for different readers)

---

### Answers to Expected Questions

**Q1: "How did you identify the ASRs?"**

> "I analyzed the codebase for requirements that had significant architectural impact. I looked for:
> - Security mechanisms (JWT, RBAC) → ASR-SEC-01, ASR-SEC-02, ASR-SEC-03
> - Concurrency handling (`@Version`) → ASR-PERF-03
> - Structural patterns (3-layer architecture) → ASR-MAINT-01
> - Data integrity mechanisms (constraints, validation) → ASR-DATA-01, ASR-REL-01
>
> Each ASR is backed by concrete code evidence and linked to quality attributes."

**Q2: "Why layered architecture? How do you know?"**

> "Evidence from the codebase:
> 1. **Consistent package structure**: Every module has `api/`, `services/`, `repositories/` packages
> 2. **Dependency flow**: Controllers only call services, services only call repositories
> 3. **No cross-layer violations**: No controllers directly accessing repositories
> 4. **DTO pattern**: Different types at each layer (DTOs in API, entities in domain)
>
> This is documented in **AD-01** (Architectural Decision 01) with alternatives considered (microservices, hexagonal) and rationale."

**Q3: "What are the system's strengths and weaknesses?"**

> **Strengths**:
> - Strong security (JWT, RBAC, BCrypt)
> - Clean architecture (well-organized layers)
> - Good concurrency handling (optimistic locking)
> - Comprehensive API (44 functional requirements)
> - Well-documented domain (DDD artifacts)
>
> **Weaknesses** (documented in NFR recommendations):
> - H2 database not production-ready → needs PostgreSQL migration
> - No API versioning strategy
> - Limited audit logging
> - Genre classification could be enhanced (hierarchical, multiple per book)
>
> All documented with recommendations in NonFunctionalRequirements.md."

**Q4: "What would you do differently?"**

> "Based on my analysis, I'd recommend (all documented):
>
> **Priority 1 (High Impact)**:
> 1. Migrate to PostgreSQL for production (currently H2)
> 2. Implement audit logging for compliance (NFR-COMP-03)
> 3. Add API versioning strategy (NFR-USE-04)
>
> **Priority 2 (Enhancements)**:
> 1. Hierarchical genre classification (GenreClassificationSystem.md, Enhancement 1)
> 2. Multiple genres per book (Enhancement 2)
> 3. Genre recommendation engine (Enhancement 4)
>
> These are justified by business value, technical feasibility, and effort estimates."

**Q5: "How does this relate to course concepts?"**

> "This project demonstrates core software architecture concepts:
>
> - **Architectural Styles**: Layered architecture (3-tier)
> - **Design Patterns**: Repository, Service, DTO, Value Object, Aggregate
> - **Quality Attributes**: Security, Performance, Maintainability (ISO 25010)
> - **Architectural Views**: Component, Deployment, Container (4+1 views)
> - **Domain-Driven Design**: Aggregates, Value Objects, Ubiquitous Language
> - **Architectural Decisions**: Trade-offs, alternatives, rationale
> - **Requirements Engineering**: Functional vs. Non-Functional, traceability
>
> All concepts applied to real, working codebase (144 Java files, 30,000+ lines)."

---

## Common Questions and Answers

### Q: What is the difference between architecture and design?

**A**:
- **Architecture** = High-level structure, major components, fundamental patterns
  - Example: "We use 3-layer architecture with 7 modules"
  - Time scale: Weeks to months
  - Scope: Entire system

- **Design** = Low-level details, class relationships, algorithms
  - Example: "This method uses binary search algorithm"
  - Time scale: Hours to days
  - Scope: Individual classes/methods

**Analogy**:
- Architecture = City planning (where neighborhoods, roads, zones go)
- Design = Building design (floor plans, room layouts)

---

### Q: Why is layered architecture good for this project?

**A**:
1. **Educational**: Easy to understand, widely taught
2. **Team size**: Suitable for small team (layered scales better than microservices for 1-5 developers)
3. **Complexity**: Right level for library domain (not too simple, not too complex)
4. **Maintainability**: Clear structure, easy to locate code
5. **Testability**: Can test layers independently

**When NOT to use**:
- Large-scale systems with distinct scaling needs (use microservices)
- Systems with pluggable components (use component-based)

---

### Q: What is Domain-Driven Design (DDD)?

**A**: Organizing code around business concepts, not technical concerns.

**Without DDD**:
```
controllers/
services/
repositories/
models/
utils/
```
→ Organized by technical role (hard to find all code for "Author")

**With DDD** (this project):
```
authormanagement/
├── api/
├── services/
├── repositories/
└── model/
```
→ Organized by business concept (all Author code in one place)

**Key ideas**:
- **Ubiquitous Language**: Code uses business terms
- **Bounded Contexts**: Each module is independent domain
- **Aggregates**: Clusters of related objects (Author + Bio + Photo)
- **Value Objects**: Immutable concepts (ISBN, Name, Email)

---

### Q: Why use DTOs? Why not expose entities directly?

**A**:

**Problem with exposing entities**:
```java
@GetMapping("/{id}")
public Author getAuthor(@PathVariable Long id) {  // ❌ Exposes entity
    return authorService.findById(id);
}
// Response includes: version, createdDate, modifiedDate, internal IDs...
// Client sees too much! Breaking changes if entity changes.
```

**Solution with DTOs**:
```java
@GetMapping("/{id}")
public AuthorView getAuthor(@PathVariable Long id) {  // ✅ Exposes DTO
    Author author = authorService.findById(id);
    return mapper.toView(author);  // Only expose what client needs
}
// Response: { id, name, bio, photoUri }
// Clean! Can change entity without breaking API.
```

**Benefits**:
- **Security**: Don't expose internal fields (version, timestamps)
- **Flexibility**: Change entity without breaking API
- **Simplification**: DTOs are flat, entities might have complex relationships
- **Versioning**: Different DTOs for different API versions

---

### Q: What is optimistic locking? Why use it?

**A**:

**The Problem**: Two users edit the same book simultaneously

```
Time    User A                    User B
t1      Read Book v1
t2                                Read Book v1
t3      Change title
t4                                Change description
t5      Save
t6                                Save (overwrites A's change!) ❌
```

**The Solution**: Version checking

```java
@Entity
public class Book {
    @Version  // JPA manages this automatically
    private Long version;
}

// When saving:
UPDATE BOOK SET title = ?, version = version + 1
WHERE id = ? AND version = ?
// If version changed → UPDATE returns 0 rows → Exception!
```

**Flow with optimistic locking**:
```
Time    User A                    User B
t1      Read Book v1
t2                                Read Book v1
t3      Change title
t4                                Change description
t5      Save (v1→v2) ✅
t6                                Save v1 expected, found v2 ❌ CONFLICT!
```

User B gets error: "Book was modified by someone else. Please refresh."

**Why?**
- Better performance (no database locks)
- Better UX (no waiting for locks)
- Works with stateless APIs

---

### Q: What is dependency injection?

**A**: Spring creates objects and injects dependencies automatically.

**Without DI** (manual):
```java
public class AuthorController {
    private AuthorService service = new AuthorServiceImpl();  // ❌ Tight coupling
    // Problem: Controller creates service, can't swap implementation
}
```

**With DI** (Spring):
```java
@RestController
public class AuthorController {
    private final AuthorService service;  // Interface

    public AuthorController(AuthorService service) {  // Spring injects
        this.service = service;
    }
    // ✅ Controller doesn't know implementation
    // ✅ Easy to test (inject mock)
}
```

**Spring IoC Container**:
1. Scans for `@Component`, `@Service`, `@Controller` annotations
2. Creates instances (singletons by default)
3. Injects dependencies via constructor

**Benefits**:
- Loose coupling
- Testability (inject mocks)
- Configuration-driven

---

### Q: How do I explain the value of this documentation?

**A**:

**For Developer**:
- "With this documentation, a new developer can understand the system in hours instead of weeks"
- "Clear traceability: requirement → implementation → test"
- "Architecture diagrams show the big picture immediately"

**For Maintainer**:
- "Architectural decisions documented: understand WHY, not just WHAT"
- "Enhancement recommendations provide roadmap for improvements"
- "NFRs document quality attributes and measurement criteria"

**For Architect**:
- "System-as-is documented: baseline for future changes"
- "Architectural patterns identified: can replicate good practices"
- "Trade-offs explicit: understand constraints and alternatives"

**For Manager**:
- "Reduces onboarding time (time = money)"
- "Facilitates knowledge transfer (reduce bus factor)"
- "Provides basis for planning (enhancement recommendations prioritized)"

---

## Conclusion: Key Takeaways

### What You've Learned

1. **Layered Architecture**: How layers separate concerns and enable maintainability

2. **Domain-Driven Design**: How to organize code around business concepts

3. **Design Patterns**: Repository, Service, DTO, Value Object, Aggregate

4. **Quality Attributes**: Security, Performance, Maintainability, Usability

5. **Architectural Decisions**: Trade-offs, alternatives, rationale

6. **Documentation**: How to document architecture comprehensively

### What Makes This Architecture Good

1. **Clear Structure**: Consistent 3-layer pattern across all modules
2. **Separation of Concerns**: Each layer has one responsibility
3. **Testability**: Layers can be tested independently
4. **Maintainability**: Easy to locate and modify code
5. **Scalability**: Stateless design enables horizontal scaling
6. **Security**: JWT, RBAC, validation at multiple levels
7. **Domain Alignment**: Code matches business concepts

### What Could Be Improved

1. **Database**: Migrate from H2 to PostgreSQL for production
2. **API Versioning**: Strategy for backwards compatibility
3. **Audit Logging**: Track who changed what when
4. **Genre System**: Enhance with hierarchy and multiple genres per book
5. **Testing**: Increase test coverage (currently 23 test files)

### Final Advice

**For your teacher meeting**:

1. **Start with the big picture**: Show [SystemArchitecture.puml](SystemArchitecture.puml)
2. **Explain one flow end-to-end**: Trace a request through layers
3. **Highlight key decisions**: Why layered? Why JWT? Why DDD?
4. **Show evidence**: Point to code for every claim
5. **Demonstrate critical thinking**: Discuss trade-offs and alternatives

**Remember**:
- Architecture is about **trade-offs**, not "right/wrong"
- Every decision has **rationale** (documented in ASRs)
- Good architecture **evolves** (documented enhancement recommendations)

**You have comprehensive, professional-quality architecture documentation. Be confident!**

---

## Quick Reference

### Key Files

| Document | Purpose |
|----------|---------|
| [README.md](README.md) | Index and navigation |
| [ASR-ArchitecturallySignificantRequirements.md](ASR-ArchitecturallySignificantRequirements.md) | ASRs and architectural decisions |
| [FunctionalRequirements.md](FunctionalRequirements.md) | 44 functional requirements |
| [NonFunctionalRequirements.md](NonFunctionalRequirements.md) | 30 non-functional requirements |
| [GenreClassificationSystem.md](GenreClassificationSystem.md) | Genre system deep-dive |
| [SystemArchitecture.puml](SystemArchitecture.puml) | Component diagram |
| [DeploymentArchitecture.puml](DeploymentArchitecture.puml) | Deployment view |
| [UnderstandingTheSystem.md](UnderstandingTheSystem.md) | This guide |

### Key Concepts

- **Layered Architecture**: 3 layers (Presentation, Business, Data)
- **DDD**: Domain-Driven Design (Aggregates, Value Objects)
- **JWT**: JSON Web Token (stateless authentication)
- **Optimistic Locking**: Version-based concurrency control
- **Dependency Injection**: Spring IoC container
- **Repository Pattern**: Data access abstraction
- **DTO Pattern**: Data transfer objects

### Questions?

Start with [README.md](README.md) or search this document!

---

**Good luck with your teacher meeting!** 🎓
