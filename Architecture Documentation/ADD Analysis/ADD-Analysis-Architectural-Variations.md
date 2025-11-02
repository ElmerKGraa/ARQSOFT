# ADD Analysis: Architectural Variations Adoption

**Project:** Library Management System (psoft-g1)
**Purpose:** Analyze adoption of configurable architectural alternatives
**Date:** November 2025
**Version:** 1.0

---

## Executive Summary

This document applies the **Attribute-Driven Design (ADD)** methodology to analyze and design the adoption of configurable architectural variations in the Library Management System. The variations include:

1. **Multi-persistence support** (SQL+Redis, MongoDB+Redis)
2. **External ISBN lookup services** (ISBNdb, Google Books API)
3. **Pluggable ID generation strategies**
4. **Configuration-time architectural decisions**

---

## Table of Contents

1. [Requirements Identification and Classification](#1-requirements-identification-and-classification)
2. [ADD Process Application](#2-add-process-application)
3. [Architectural Tactics](#3-architectural-tactics)
4. [Reference Architectures](#4-reference-architectures)
5. [Design Patterns](#5-design-patterns)
6. [Architectural Alternatives and Rationale](#6-architectural-alternatives-and-rationale)
7. [Implementation Roadmap](#7-implementation-roadmap)

---

## 1. Requirements Identification and Classification

### 1.1 New Functional Requirements

| ID | Requirement | Type | Priority |
|----|-------------|------|----------|
| **NFR-P1** | Support SQL (H2/PostgreSQL) + Redis persistence | Functional | High |
| **NFR-P2** | Support MongoDB + Redis persistence | Functional | High |
| **NFR-E1** | Integrate ISBNdb for ISBN lookup by title | Functional | Medium |
| **NFR-E2** | Integrate Google Books API for ISBN lookup | Functional | Medium |
| **NFR-I1** | Support pluggable ID generation strategies | Functional | Medium |
| **NFR-C1** | Configuration-time selection of alternatives | Functional | High |

### 1.2 Quality Attribute Requirements

| Quality Attribute | Scenario | Current State | Target State |
|-------------------|----------|---------------|--------------|
| **Modifiability** | Switch persistence strategy without code changes | Not supported | Configuration-based switching |
| **Performance** | Read-heavy operations (Top 5 queries, search) | Database only | Redis caching layer |
| **Interoperability** | ISBN enrichment from external sources | Not available | Multiple provider support |
| **Configurability** | Runtime behavior based on setup-time config | Limited | Comprehensive configuration support |
| **Testability** | Test with different persistence backends | Single H2 implementation | Multiple implementation testing |
| **Scalability** | Handle increased read load | Limited by database | Distributed caching with Redis |

### 1.3 Architectural Drivers (ASRs - Architecturally Significant Requirements)

**ASR-1: Multi-Persistence Strategy Support**
- **Stimulus:** System administrator selects persistence strategy at deployment time
- **Response:** System uses selected persistence implementation (SQL, MongoDB) with Redis caching
- **Quality Attributes:** Modifiability, Performance, Configurability
- **Business Value:** Support different deployment scenarios (enterprise SQL, modern NoSQL)

**ASR-2: External Service Integration**
- **Stimulus:** User requests book by title without ISBN
- **Response:** System queries configured external ISBN provider (ISBNdb or Google Books)
- **Quality Attributes:** Interoperability, Modifiability, Availability
- **Business Value:** Enrich book catalog with external data

**ASR-3: Pluggable ID Generation**
- **Stimulus:** Business rules change ID format (e.g., ReaderNumber from YYYY/SEQ to UUID)
- **Response:** System uses configured ID generator without code changes
- **Quality Attributes:** Modifiability, Configurability
- **Business Value:** Adapt to changing business requirements

**ASR-4: Configuration-Driven Architecture**
- **Stimulus:** DevOps team deploys to different environments (dev, staging, production)
- **Response:** System behavior adapts based on configuration profiles
- **Quality Attributes:** Configurability, Portability, Testability
- **Business Value:** Reduce deployment complexity, support A/B testing

### 1.4 Constraints

| Type | Constraint | Impact |
|------|-----------|--------|
| **Technical** | Spring Boot 3.2.5 framework | Must use Spring abstractions (Boot, Data, Cache) |
| **Technical** | Existing JPA entity model | Cannot completely decouple from JPA annotations initially |
| **Technical** | Current service layer contract | Repository interfaces must remain compatible |
| **Organizational** | Academic project timeline | Prioritize high-value, demonstrable features |
| **Business** | Backward compatibility | Existing H2 implementation must continue to work |

---

## 2. ADD Process Application

### 2.1 ADD Iteration Overview

We'll apply 3 ADD iterations focusing on different architectural concerns:

```
Iteration 1: Multi-Persistence Support (SQL + Redis, MongoDB + Redis)
    ├── Focus: Repository layer abstraction and caching
    └── Drivers: ASR-1 (Multi-Persistence), Performance, Modifiability

Iteration 2: External ISBN Lookup Integration
    ├── Focus: External service abstraction and resilience
    └── Drivers: ASR-2 (External Services), Interoperability, Availability

Iteration 3: Pluggable ID Generation & Configuration Strategy
    ├── Focus: Configuration management and runtime strategy selection
    └── Drivers: ASR-3 (ID Generation), ASR-4 (Configuration), Modifiability
```

---

### 2.2 ADD Iteration 1: Multi-Persistence Support

#### Step 1: Review Inputs
- **Design Purpose:** Support multiple persistence backends with Redis caching
- **Primary Functional Requirements:** NFR-P1, NFR-P2
- **Quality Attributes:** Performance (caching), Modifiability (strategy switching)
- **Constraints:** Existing service layer contract, Spring Boot ecosystem

#### Step 2: Establish Iteration Goal
**Goal:** Design a repository abstraction layer that supports:
1. SQL-based persistence (H2, PostgreSQL, MySQL)
2. MongoDB document storage
3. Redis caching layer (read-through, write-through cache)
4. Configuration-time selection of persistence strategy

#### Step 3: Choose Elements to Refine
**Primary Elements:**
- Repository layer (all modules: Book, Author, Reader, Lending, Genre)
- Caching layer (currently only in UserRepository)
- Configuration management (Spring profiles, properties)

#### Step 4: Choose Design Concepts

**Selected Tactics:**
- **Repository Pattern** (already in use, will be enhanced)
- **Strategy Pattern** for persistence implementation selection
- **Cache-Aside Pattern** for Redis integration
- **Abstract Factory Pattern** for repository creation
- **Spring Configuration Profiles** for deployment-time selection

**Reference Architecture:**
- **Polyglot Persistence Architecture** (microservices pattern)
- **CQRS lite** (separate read/write concerns for caching)

#### Step 5: Instantiate Architectural Elements

**New Components:**

1. **Persistence Strategy Abstraction**
```
pt.psoft.g1.psoftg1.infrastructure.persistence/
├── PersistenceStrategy (enum: SQL, MONGODB)
├── config/
│   ├── SqlPersistenceConfig.java
│   ├── MongoDbPersistenceConfig.java
│   └── RedisCacheConfig.java
└── repositories/
    ├── sql/          # Spring Data JPA implementations
    ├── mongodb/      # Spring Data MongoDB implementations
    └── cache/        # Redis cache decorators
```

2. **Cache Abstraction Layer**
```
pt.psoft.g1.psoftg1.shared.caching/
├── CacheStrategy (interface)
├── RedisCacheStrategy (implementation)
├── CacheableRepository (decorator pattern)
└── CacheConfig.java
```

#### Step 6: Define Interfaces

**Enhanced Repository Pattern:**
```java
// Domain layer - unchanged
public interface BookRepository {
    Optional<Book> findByIsbn(Isbn isbn);
    List<Book> searchBooks(Page page, SearchBooksQuery query);
    Page<Book> findTop5BooksLent(Pageable pageable);
    // ... existing methods
}

// Infrastructure layer - SQL implementation
@Repository
@Profile("persistence-sql")
public class SqlBookRepository implements BookRepository {
    // Spring Data JPA implementation
}

// Infrastructure layer - MongoDB implementation
@Repository
@Profile("persistence-mongodb")
public class MongoBookRepository implements BookRepository {
    // Spring Data MongoDB implementation
}

// Caching decorator
@Component
@Primary
@ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
public class CachedBookRepository implements BookRepository {
    private final BookRepository delegate;
    private final CacheStrategy cacheStrategy;

    // Cache-aside implementation
}
```

#### Step 7: Sketch Views and Record Design Decisions

**Decision 1: Use Spring Profiles for Persistence Selection**
- **Rationale:** Leverages Spring Boot's configuration management
- **Alternatives:** Custom factory, service locator pattern
- **Trade-offs:** Profile-based is less flexible at runtime but simpler

**Decision 2: Decorator Pattern for Caching**
- **Rationale:** Non-invasive, follows Open/Closed Principle
- **Alternatives:** AOP-based caching, manual cache management
- **Trade-offs:** Slight complexity vs. clean separation

**Decision 3: Maintain Domain Model with JPA Annotations**
- **Rationale:** Pragmatic approach for academic timeline
- **Alternatives:** Separate persistence models, ORM-free domain
- **Trade-offs:** JPA coupling vs. development speed

#### Step 8: Analyze Current Design

**Refined Elements:**
- ✅ Repository layer (all 9 repositories)
- ✅ Configuration layer (persistence profiles)
- ✅ Cache layer (Redis integration)

**Remaining Elements:**
- External service integration (Iteration 2)
- ID generation strategies (Iteration 3)

---

### 2.3 ADD Iteration 2: External ISBN Lookup Integration

#### Step 1: Review Inputs
- **Design Purpose:** Enrich book data with ISBN from external providers
- **Primary Functional Requirements:** NFR-E1, NFR-E2
- **Quality Attributes:** Interoperability, Availability, Modifiability

#### Step 2: Establish Iteration Goal
**Goal:** Design an external service abstraction supporting:
1. ISBNdb API integration
2. Google Books API integration
3. Failover/fallback strategy
4. Configuration-time provider selection

#### Step 3: Choose Elements to Refine
**Primary Elements:**
- External service layer (new)
- Book service (enhancement for ISBN lookup)
- Configuration management (API keys, endpoints)

#### Step 4: Choose Design Concepts

**Selected Tactics:**
- **Strategy Pattern** for provider selection
- **Circuit Breaker Pattern** for resilience
- **Facade Pattern** for external API complexity
- **Adapter Pattern** for API response normalization

**Reference Architecture:**
- **API Gateway Pattern** (simplified, no dedicated gateway)
- **Anti-Corruption Layer** (DDD pattern for external integration)

#### Step 5: Instantiate Architectural Elements

**New Components:**

```
pt.psoft.g1.psoftg1.external.isbn/
├── IsbnLookupService (interface)
├── IsbnLookupProvider (enum: ISBNDB, GOOGLE_BOOKS)
├── providers/
│   ├── IsbnDbProvider.java
│   ├── GoogleBooksProvider.java
│   └── FallbackProvider.java (composite)
├── model/
│   ├── IsbnLookupRequest.java
│   └── IsbnLookupResponse.java
└── config/
    └── IsbnLookupConfig.java
```

#### Step 6: Define Interfaces

```java
// External service abstraction
public interface IsbnLookupService {
    Optional<String> findIsbnByTitle(String title, String author);
    List<IsbnLookupResponse> searchByTitle(String title);
}

// Provider implementations
@Service
@Profile("isbn-provider-isbndb")
public class IsbnDbProvider implements IsbnLookupService {
    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;
    // ISBNdb-specific implementation
}

@Service
@Profile("isbn-provider-googlebooks")
public class GoogleBooksProvider implements IsbnLookupService {
    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;
    // Google Books API implementation
}

// Composite fallback strategy
@Service
@Primary
@ConditionalOnProperty(name = "isbn.lookup.fallback", havingValue = "true")
public class FallbackIsbnLookupService implements IsbnLookupService {
    private final List<IsbnLookupService> providers;

    @Override
    public Optional<String> findIsbnByTitle(String title, String author) {
        for (IsbnLookupService provider : providers) {
            try {
                Optional<String> result = provider.findIsbnByTitle(title, author);
                if (result.isPresent()) return result;
            } catch (Exception e) {
                // Log and try next provider
            }
        }
        return Optional.empty();
    }
}
```

#### Step 7: Record Design Decisions

**Decision 4: Use Resilience4j Circuit Breaker**
- **Rationale:** Prevent cascading failures from external APIs
- **Configuration:** 50% failure threshold, 60s timeout

**Decision 5: Fallback Chain Strategy**
- **Rationale:** High availability by trying multiple providers
- **Trade-offs:** Increased latency vs. availability

---

### 2.4 ADD Iteration 3: Pluggable ID Generation & Configuration

#### Step 1: Review Inputs
- **Design Purpose:** Support multiple ID generation strategies
- **Primary Functional Requirements:** NFR-I1, NFR-C1
- **Quality Attributes:** Modifiability, Configurability

#### Step 2: Establish Iteration Goal
**Goal:** Design ID generation abstraction supporting:
1. Auto-increment (current)
2. UUID/GUID
3. Year-based sequences (YYYY/SEQ)
4. Custom business logic

#### Step 3: Choose Elements to Refine
**Primary Elements:**
- Entity ID generation (all entities)
- Configuration management (centralized)

#### Step 4: Choose Design Concepts

**Selected Tactics:**
- **Strategy Pattern** for ID generation
- **Factory Pattern** for ID generator creation
- **Template Method Pattern** for common ID logic

#### Step 5: Instantiate Architectural Elements

**New Components:**

```
pt.psoft.g1.psoftg1.shared.idgeneration/
├── IdGenerator<T> (interface)
├── IdGenerationStrategy (enum)
├── generators/
│   ├── AutoIncrementIdGenerator.java
│   ├── UuidIdGenerator.java
│   ├── YearSequenceIdGenerator.java
│   └── CustomIdGenerator.java
└── config/
    └── IdGenerationConfig.java
```

#### Step 6: Define Interfaces

```java
public interface IdGenerator<T> {
    T generateId(Object context);
    boolean validate(T id);
}

@Component
@ConditionalOnProperty(name = "id.generation.reader.strategy", havingValue = "year-sequence")
public class ReaderNumberGenerator implements IdGenerator<ReaderNumber> {
    @Override
    public ReaderNumber generateId(Object context) {
        int year = LocalDate.now().getYear();
        int sequence = getNextSequence(year);
        return new ReaderNumber(year, sequence);
    }
}
```

#### Step 7: Record Design Decisions

**Decision 6: Separate ID Generation from JPA**
- **Rationale:** Decouple business logic from persistence
- **Implementation:** Use `@PrePersist` callbacks or service layer

**Decision 7: Configuration Properties per Entity**
- **Rationale:** Fine-grained control over each entity type
- **Example:**
```properties
id.generation.reader.strategy=year-sequence
id.generation.lending.strategy=year-sequence
id.generation.author.strategy=auto-increment
```

---

## 3. Architectural Tactics

### 3.1 Modifiability Tactics

| Tactic | Application | Benefit |
|--------|-------------|---------|
| **Increase Semantic Coherence** | Separate persistence strategies into distinct modules | Clear boundaries, easier to modify |
| **Abstract Common Services** | Repository interface abstraction | Swap implementations without service layer changes |
| **Use Intermediary** | Cache decorator, ISBN lookup facade | Decouple clients from implementation details |
| **Restrict Dependencies** | Domain layer independent of infrastructure | Changes to persistence don't affect domain |
| **Configuration Files** | Spring profiles, application.properties | Runtime behavior modification without recompilation |

### 3.2 Performance Tactics

| Tactic | Application | Benefit |
|--------|-------------|---------|
| **Introduce Concurrency** | Redis async operations | Non-blocking cache reads/writes |
| **Maintain Multiple Copies** | Redis cache for read-heavy data | Reduced database load |
| **Increase Available Resources** | Distributed Redis cache | Horizontal scalability |
| **Reduce Computational Overhead** | Cache Top 5 queries (expensive aggregations) | Avoid repeated expensive calculations |

### 3.3 Availability Tactics

| Tactic | Application | Benefit |
|--------|-------------|---------|
| **Exception Detection** | Circuit breaker for external ISBN APIs | Detect and handle failures |
| **Exception Prevention** | Input validation before external calls | Avoid unnecessary failures |
| **Exception Recovery** | Fallback chain for ISBN lookup | Graceful degradation |
| **Redundancy** | Multiple ISBN providers | No single point of failure |

### 3.4 Testability Tactics

| Tactic | Application | Benefit |
|--------|-------------|---------|
| **Separate Interface from Implementation** | Repository pattern already in place | Easy to mock implementations |
| **Record/Playback** | Cache external API responses for tests | Deterministic testing |
| **Specialized Interfaces** | Separate read and write concerns | Isolated testing of operations |

---

## 4. Reference Architectures

### 4.1 Polyglot Persistence Architecture

**Source:** Microservices architecture patterns (Martin Fowler, Sam Newman)

**Application to Project:**
```
┌─────────────────────────────────────────────────────┐
│              Service Layer (unchanged)              │
│  BookService, AuthorService, ReaderService, etc.   │
└──────────────────┬──────────────────────────────────┘
                   │ Depends on domain interfaces
┌──────────────────▼──────────────────────────────────┐
│           Repository Interfaces (domain)            │
│  BookRepository, AuthorRepository, etc.             │
└──────────────────┬──────────────────────────────────┘
                   │ Implemented by
       ┌───────────┴───────────┐
       │                       │
┌──────▼──────────┐   ┌────────▼────────────┐
│  SQL Strategy   │   │  MongoDB Strategy   │
│  (H2, PgSQL)    │   │  (Documents)        │
└──────┬──────────┘   └────────┬────────────┘
       │                       │
       └───────────┬───────────┘
                   │ Both use
         ┌─────────▼──────────┐
         │  Redis Cache Layer │
         │  (Performance)     │
         └────────────────────┘
```

**Benefits:**
- Choose optimal storage for each data type
- SQL for transactional data (Lendings)
- MongoDB for flexible schemas (future: reader activity logs)
- Redis for read-heavy aggregations (Top 5 queries)

### 4.2 Hexagonal Architecture (Ports and Adapters)

**Source:** Alistair Cockburn

**Current Alignment:**
```
        ┌────────────────────────────┐
        │      Domain Core           │
        │  (Entities, Value Objects) │
        │  (Repository Interfaces)   │
        └────────────┬───────────────┘
                     │ Ports
        ┌────────────┴───────────────┐
        │                            │
┌───────▼────────┐          ┌────────▼────────┐
│  Driving Side  │          │  Driven Side    │
│  (Controllers) │          │  (Adapters)     │
│  REST API      │          │  - JPA          │
│  OpenAPI       │          │  - MongoDB      │
└────────────────┘          │  - Redis        │
                            │  - ISBN APIs    │
                            └─────────────────┘
```

**Enhancement Needed:**
- Formalize port interfaces (already partially done with repositories)
- Add explicit adapter layer for external services
- Document port/adapter boundaries

### 4.3 CQRS Lite (Read/Write Separation)

**Source:** Command Query Responsibility Segregation (Greg Young)

**Simplified Application:**
```
Write Path (Commands):
  BookService.createBook()
      ↓
  BookRepository.save()
      ↓
  Primary Store (SQL/MongoDB)
      ↓
  Cache Invalidation (Redis)

Read Path (Queries):
  BookService.findTop5Books()
      ↓
  CachedBookRepository.findTop5()
      ↓
  Redis Cache (if hit)
      ↓
  Primary Store (if miss)
      ↓
  Cache Update (write-through)
```

**Benefits for Project:**
- Top 5 queries are read-heavy, benefit from caching
- Write operations go directly to primary store
- Clear separation of concerns

---

## 5. Design Patterns

### 5.1 Repository Pattern (Enhanced)

**Current State:** Domain interfaces with Spring Data implementations

**Enhancement:**
```java
// Domain interface (unchanged)
public interface BookRepository {
    Optional<Book> findByIsbn(Isbn isbn);
    // ... methods
}

// Abstract base for common logic
public abstract class AbstractBookRepository implements BookRepository {
    protected final CacheStrategy cache;

    @Override
    public Optional<Book> findByIsbn(Isbn isbn) {
        return cache.get(isbn.toString(), () -> doFindByIsbn(isbn));
    }

    protected abstract Optional<Book> doFindByIsbn(Isbn isbn);
}

// SQL implementation
@Repository
@Profile("persistence-sql")
public class SqlBookRepository extends AbstractBookRepository {
    private final JpaBookRepository jpaRepo;

    @Override
    protected Optional<Book> doFindByIsbn(Isbn isbn) {
        return jpaRepo.findByIsbn(isbn);
    }
}

// MongoDB implementation
@Repository
@Profile("persistence-mongodb")
public class MongoBookRepository extends AbstractBookRepository {
    private final MongoTemplate mongoTemplate;

    @Override
    protected Optional<Book> doFindByIsbn(Isbn isbn) {
        Query query = new Query(Criteria.where("isbn").is(isbn.toString()));
        return Optional.ofNullable(mongoTemplate.findOne(query, Book.class));
    }
}
```

### 5.2 Strategy Pattern (Multiple Applications)

**Application 1: Persistence Strategy**
```java
public enum PersistenceStrategy {
    SQL, MONGODB;
}

@Configuration
public class PersistenceConfig {
    @Bean
    @ConditionalOnProperty(name = "persistence.strategy", havingValue = "sql")
    public BookRepository sqlBookRepository() {
        return new SqlBookRepository(...);
    }

    @Bean
    @ConditionalOnProperty(name = "persistence.strategy", havingValue = "mongodb")
    public BookRepository mongoBookRepository() {
        return new MongoBookRepository(...);
    }
}
```

**Application 2: ISBN Lookup Strategy**
```java
public interface IsbnLookupStrategy {
    Optional<String> lookup(String title, String author);
}

@Component("isbndb")
public class IsbnDbStrategy implements IsbnLookupStrategy { ... }

@Component("googlebooks")
public class GoogleBooksStrategy implements IsbnLookupStrategy { ... }

@Service
public class IsbnLookupService {
    private final Map<String, IsbnLookupStrategy> strategies;

    @Value("${isbn.lookup.provider}")
    private String provider;

    public Optional<String> lookup(String title, String author) {
        return strategies.get(provider).lookup(title, author);
    }
}
```

### 5.3 Decorator Pattern (Caching Layer)

```java
@Component
@Primary
@ConditionalOnProperty(name = "cache.enabled", havingValue = "true")
public class CachedBookRepository implements BookRepository {
    private final BookRepository delegate;
    private final RedisTemplate<String, Book> redisTemplate;
    private final Duration ttl;

    public CachedBookRepository(
            @Qualifier("actualBookRepository") BookRepository delegate,
            RedisTemplate<String, Book> redisTemplate) {
        this.delegate = delegate;
        this.redisTemplate = redisTemplate;
        this.ttl = Duration.ofMinutes(10);
    }

    @Override
    public Optional<Book> findByIsbn(Isbn isbn) {
        String cacheKey = "book:isbn:" + isbn.toString();

        // Try cache first
        Book cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            return Optional.of(cached);
        }

        // Cache miss - fetch from delegate
        Optional<Book> result = delegate.findByIsbn(isbn);

        // Update cache
        result.ifPresent(book ->
            redisTemplate.opsForValue().set(cacheKey, book, ttl)
        );

        return result;
    }

    @Override
    public Book save(Book book) {
        Book saved = delegate.save(book);

        // Cache invalidation
        String cacheKey = "book:isbn:" + book.getIsbn().toString();
        redisTemplate.delete(cacheKey);

        // Invalidate related queries
        redisTemplate.delete("book:top5");

        return saved;
    }
}
```

### 5.4 Adapter Pattern (External APIs)

```java
// Common interface
public interface ExternalBookProvider {
    Optional<BookExternalData> findByIsbn(String isbn);
    List<BookExternalData> searchByTitle(String title);
}

// ISBNdb adapter
@Component
@ConditionalOnProperty(name = "external.book.provider", havingValue = "isbndb")
public class IsbnDbAdapter implements ExternalBookProvider {
    private final WebClient webClient;

    @Override
    public Optional<BookExternalData> findByIsbn(String isbn) {
        IsbnDbResponse response = webClient.get()
            .uri("/book/{isbn}", isbn)
            .retrieve()
            .bodyToMono(IsbnDbResponse.class)
            .block();

        return Optional.ofNullable(response)
            .map(this::convertToCommonFormat);
    }

    private BookExternalData convertToCommonFormat(IsbnDbResponse response) {
        return BookExternalData.builder()
            .isbn(response.getIsbn13())
            .title(response.getTitle())
            .authors(response.getAuthors())
            .publisher(response.getPublisher())
            .build();
    }
}

// Google Books adapter
@Component
@ConditionalOnProperty(name = "external.book.provider", havingValue = "googlebooks")
public class GoogleBooksAdapter implements ExternalBookProvider {
    private final WebClient webClient;

    @Override
    public Optional<BookExternalData> findByIsbn(String isbn) {
        GoogleBooksResponse response = webClient.get()
            .uri("/volumes?q=isbn:{isbn}", isbn)
            .retrieve()
            .bodyToMono(GoogleBooksResponse.class)
            .block();

        return Optional.ofNullable(response)
            .map(this::convertToCommonFormat);
    }

    private BookExternalData convertToCommonFormat(GoogleBooksResponse response) {
        VolumeInfo info = response.getItems().get(0).getVolumeInfo();
        return BookExternalData.builder()
            .isbn(extractIsbn(info.getIndustryIdentifiers()))
            .title(info.getTitle())
            .authors(info.getAuthors())
            .publisher(info.getPublisher())
            .build();
    }
}
```

### 5.5 Factory Pattern (ID Generators)

```java
@Configuration
public class IdGeneratorFactory {

    @Bean
    public IdGenerator<ReaderNumber> readerNumberGenerator(
            @Value("${id.generation.reader.strategy}") String strategy) {
        return switch (strategy) {
            case "year-sequence" -> new YearSequenceReaderNumberGenerator();
            case "uuid" -> new UuidReaderNumberGenerator();
            case "auto-increment" -> new AutoIncrementReaderNumberGenerator();
            default -> throw new IllegalArgumentException("Unknown strategy: " + strategy);
        };
    }

    @Bean
    public IdGenerator<LendingNumber> lendingNumberGenerator(
            @Value("${id.generation.lending.strategy}") String strategy) {
        return switch (strategy) {
            case "year-sequence" -> new YearSequenceLendingNumberGenerator();
            case "uuid" -> new UuidLendingNumberGenerator();
            default -> new YearSequenceLendingNumberGenerator(); // default
        };
    }
}
```

---

## 6. Architectural Alternatives and Rationale

### 6.1 Alternative 1: SQL + Redis Persistence

#### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    REST API Layer                           │
│               (BookController, etc.)                        │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   Service Layer                             │
│         (BookService, AuthorService, etc.)                  │
└────────────────────────┬────────────────────────────────────┘
                         │ Uses domain interfaces
┌────────────────────────▼────────────────────────────────────┐
│              Repository Interfaces (Domain)                 │
│   BookRepository, AuthorRepository, ReaderRepository, etc.  │
└────────────────────────┬────────────────────────────────────┘
                         │ Implemented by
                         │
        ┌────────────────┴─────────────────┐
        │                                  │
┌───────▼──────────┐            ┌─────────▼──────────┐
│  Cache Decorator │            │  SQL Repository    │
│  (Redis Layer)   │────────────│  (JPA/Hibernate)   │
└──────────────────┘   delegate └────────────────────┘
        │                                  │
        │                                  │
┌───────▼──────────┐            ┌─────────▼──────────┐
│  Redis Cluster   │            │  PostgreSQL        │
│  (Cache Store)   │            │  (Primary Store)   │
│                  │            │                    │
│  - Key patterns  │            │  - Relational      │
│  - TTL: 10 min   │            │  - ACID            │
│  - Eviction: LRU │            │  - Transactions    │
└──────────────────┘            └────────────────────┘
```

#### Implementation Strategy

**Phase 1: Redis Integration**
1. Add Spring Data Redis dependency
2. Configure Redis connection (standalone/cluster)
3. Implement cache decorator for each repository
4. Define caching strategy per query type

**Phase 2: SQL Database Migration (H2 → PostgreSQL)**
1. Update datasource configuration
2. Test schema migration (JPA DDL)
3. Migrate native SQL queries (H2-specific functions)
4. Performance testing and optimization

**Cache Key Design:**
```
Pattern: {entity}:{operation}:{identifier}

Examples:
- book:isbn:978-0134685991          # Single book by ISBN
- book:top5                          # Top 5 books (short TTL)
- author:{authorNumber}              # Author details
- lending:overdue                    # Overdue lendings list
- reader:{year}/{seq}                # Reader by ReaderNumber
- genre:top5                         # Top 5 genres
```

**Cache Invalidation Strategy:**
```java
@Component
public class CacheInvalidationStrategy {
    private final RedisTemplate<String, Object> redisTemplate;

    public void invalidateBookCaches(Book book) {
        // Invalidate specific book
        redisTemplate.delete("book:isbn:" + book.getIsbn());

        // Invalidate related aggregations
        redisTemplate.delete("book:top5");
        redisTemplate.delete("book:genre:" + book.getGenre().getGenre());

        // Invalidate author caches
        book.getAuthors().forEach(author ->
            redisTemplate.delete("author:" + author.getAuthorNumber() + ":books")
        );
    }

    public void invalidateLendingCaches(Lending lending) {
        redisTemplate.delete("lending:" + lending.getLendingNumber());
        redisTemplate.delete("lending:overdue");
        redisTemplate.delete("book:isbn:" + lending.getBook().getIsbn() + ":avg-duration");
        redisTemplate.delete("reader:" + lending.getReaderDetails().getReaderNumber() + ":lendings");
    }
}
```

#### Configuration Example

```yaml
# application-sql-redis.yml
spring:
  profiles:
    active: sql-redis

  datasource:
    url: jdbc:postgresql://localhost:5432/library
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver

  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: validate  # Use Flyway/Liquibase in production

  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      cluster:
        nodes: ${REDIS_CLUSTER_NODES:}
      lettuce:
        pool:
          max-active: 10
          max-idle: 5
          min-idle: 2

# Caching configuration
cache:
  enabled: true
  ttl:
    default: 600  # 10 minutes
    top5: 300     # 5 minutes (more volatile)
    book: 1800    # 30 minutes (less volatile)

# Persistence strategy
persistence:
  strategy: sql
```

#### Pros and Cons

**Pros:**
- ✅ Proven technology stack (SQL + Redis)
- ✅ ACID guarantees for transactional data
- ✅ Rich query capabilities (JPA, SQL)
- ✅ Significant performance improvement for read-heavy operations (Top 5, search)
- ✅ Existing team knowledge
- ✅ Minimal changes to current codebase

**Cons:**
- ❌ Schema rigidity (migrations needed for changes)
- ❌ Vertical scaling limitations (single primary database)
- ❌ Native SQL queries need database-specific adaptation
- ❌ Cache consistency complexity

**Best For:**
- Production deployment with strong consistency requirements
- Read-heavy workloads (reporting, dashboards)
- Teams familiar with relational databases

---

### 6.2 Alternative 2: MongoDB + Redis Persistence

#### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    REST API Layer                           │
│               (BookController, etc.)                        │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│                   Service Layer                             │
│         (BookService, AuthorService, etc.)                  │
└────────────────────────┬────────────────────────────────────┘
                         │ Uses domain interfaces
┌────────────────────────▼────────────────────────────────────┐
│              Repository Interfaces (Domain)                 │
│   BookRepository, AuthorRepository, ReaderRepository, etc.  │
└────────────────────────┬────────────────────────────────────┘
                         │ Implemented by
                         │
        ┌────────────────┴─────────────────┐
        │                                  │
┌───────▼──────────┐            ┌─────────▼──────────┐
│  Cache Decorator │            │  MongoDB Repository│
│  (Redis Layer)   │────────────│  (Spring Data)     │
└──────────────────┘   delegate └────────────────────┘
        │                                  │
        │                                  │
┌───────▼──────────┐            ┌─────────▼──────────┐
│  Redis Cluster   │            │  MongoDB Cluster   │
│  (Cache Store)   │            │  (Primary Store)   │
│                  │            │                    │
│  - Same as SQL   │            │  - Documents       │
│  - TTL: 10 min   │            │  - Flexible schema │
│  - Eviction: LRU │            │  - Aggregations    │
└──────────────────┘            └────────────────────┘
```

#### Document Model Design

**Book Collection:**
```json
{
  "_id": "ObjectId(...)",
  "isbn": "978-0134685991",
  "title": "Effective Java",
  "description": "Best practices for Java...",
  "genre": {
    "id": "ObjectId(...)",
    "name": "Programming"
  },
  "authors": [
    {
      "authorNumber": 1,
      "name": "Joshua Bloch",
      "bio": "Java platform architect..."
    }
  ],
  "photo": {
    "path": "/uploads/book-123.jpg"
  },
  "metadata": {
    "createdAt": "ISODate(...)",
    "modifiedAt": "ISODate(...)",
    "version": 1
  }
}
```

**Reader Collection:**
```json
{
  "_id": "ObjectId(...)",
  "readerNumber": "2024/1",
  "user": {
    "email": "john.doe@example.com",
    "name": "John Doe",
    "roles": ["READER"]
  },
  "birthDate": "1990-05-15",
  "phoneNumber": "+351912345678",
  "gdprConsent": true,
  "interests": [
    {"genreId": "ObjectId(...)", "name": "Science Fiction"},
    {"genreId": "ObjectId(...)", "name": "Programming"}
  ],
  "photo": {"path": "/uploads/reader-2024-1.jpg"},
  "metadata": {
    "createdAt": "ISODate(...)",
    "modifiedAt": "ISODate(...)",
    "version": 1
  }
}
```

**Lending Collection:**
```json
{
  "_id": "ObjectId(...)",
  "lendingNumber": "2024/15",
  "book": {
    "isbn": "978-0134685991",
    "title": "Effective Java"  // Denormalized for performance
  },
  "reader": {
    "readerNumber": "2024/1",
    "name": "John Doe"  // Denormalized
  },
  "dates": {
    "start": "ISODate(2024-01-15)",
    "limit": "ISODate(2024-01-30)",
    "returned": null
  },
  "fine": {
    "valuePerDayInCents": 50,
    "totalCents": 0
  },
  "commentary": "",
  "status": "ACTIVE",  // ACTIVE, RETURNED, OVERDUE
  "metadata": {
    "createdAt": "ISODate(...)",
    "version": 1
  }
}
```

#### Implementation Strategy

**Phase 1: Document Model Design**
1. Map JPA entities to MongoDB documents
2. Decide denormalization strategy (embedded vs. references)
3. Design indexes for query performance

**Phase 2: Repository Implementation**
```java
@Repository
@Profile("persistence-mongodb")
public class MongoBookRepository implements BookRepository {
    private final MongoTemplate mongoTemplate;

    @Override
    public Optional<Book> findByIsbn(Isbn isbn) {
        Query query = new Query(Criteria.where("isbn").is(isbn.toString()));
        return Optional.ofNullable(mongoTemplate.findOne(query, Book.class));
    }

    @Override
    public Page<Book> findTop5BooksLent(Pageable pageable) {
        // MongoDB aggregation pipeline
        Aggregation aggregation = Aggregation.newAggregation(
            Aggregation.lookup("lending", "_id", "book._id", "lendings"),
            Aggregation.group("_id").count().as("lendCount"),
            Aggregation.sort(Sort.Direction.DESC, "lendCount"),
            Aggregation.limit(5)
        );

        AggregationResults<BookLendingCount> results =
            mongoTemplate.aggregate(aggregation, "book", BookLendingCount.class);

        // Convert to Page<Book>
        // ...
    }
}
```

**Phase 3: Data Migration**
1. Export data from H2/SQL
2. Transform to document format
3. Import to MongoDB
4. Validate data integrity

#### Configuration Example

```yaml
# application-mongodb-redis.yml
spring:
  profiles:
    active: mongodb-redis

  data:
    mongodb:
      uri: ${MONGODB_URI:mongodb://localhost:27017/library}
      database: library
      auto-index-creation: true

    redis:
      # Same as SQL+Redis configuration
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}

# Persistence strategy
persistence:
  strategy: mongodb

# Caching configuration (same as SQL)
cache:
  enabled: true
  ttl:
    default: 600
```

**Index Strategy:**
```javascript
// MongoDB indexes
db.book.createIndex({ "isbn": 1 }, { unique: true });
db.book.createIndex({ "title": "text" });  // Full-text search
db.book.createIndex({ "genre.name": 1 });
db.book.createIndex({ "authors.authorNumber": 1 });

db.reader.createIndex({ "readerNumber": 1 }, { unique: true });
db.reader.createIndex({ "user.email": 1 }, { unique: true });
db.reader.createIndex({ "phoneNumber": 1 });

db.lending.createIndex({ "lendingNumber": 1 }, { unique: true });
db.lending.createIndex({ "status": 1 });
db.lending.createIndex({ "dates.limit": 1 });  // For overdue queries
db.lending.createIndex({ "reader.readerNumber": 1 });
db.lending.createIndex({ "book.isbn": 1 });
```

#### Pros and Cons

**Pros:**
- ✅ Flexible schema (easier to add fields)
- ✅ Horizontal scaling (sharding support)
- ✅ Rich aggregation framework (replaces complex SQL)
- ✅ Natural fit for JSON-based REST API
- ✅ No ORM impedance mismatch
- ✅ Embedded documents reduce joins

**Cons:**
- ❌ No built-in ACID transactions across documents (before v4.0)
- ❌ Denormalization requires careful consistency management
- ❌ Team learning curve (if unfamiliar with NoSQL)
- ❌ Loss of referential integrity enforcement
- ❌ Requires data migration from current SQL

**Best For:**
- Rapid prototyping with evolving schema
- Scalability requirements (large datasets, high throughput)
- Document-centric data (e.g., reader activity logs, audit trails)
- Teams with NoSQL expertise

---

### 6.3 Alternative 3: ISBN Lookup Integration

#### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                  BookService                                │
│              (registerBookByTitle)                          │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ Uses
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              IsbnLookupService                              │
│              (Facade + Fallback)                            │
└────────┬───────────────────────────────────────┬────────────┘
         │                                       │
         │ Circuit Breaker                       │ Circuit Breaker
         │ (50% threshold)                       │ (50% threshold)
         │                                       │
┌────────▼──────────┐                  ┌─────────▼────────────┐
│  ISBNdb Provider  │                  │ Google Books Provider│
│  (Primary)        │                  │ (Fallback)           │
└────────┬──────────┘                  └─────────┬────────────┘
         │                                       │
         │ HTTPS                                 │ HTTPS
         ▼                                       ▼
┌────────────────────┐                ┌──────────────────────┐
│  ISBNdb API        │                │ Google Books API     │
│  api.isbndb.com    │                │ googleapis.com       │
└────────────────────┘                └──────────────────────┘
```

#### Implementation Strategy

**Provider Interface:**
```java
public interface IsbnLookupProvider {
    Optional<IsbnLookupResult> findIsbnByTitle(String title, String author);
    List<IsbnLookupResult> searchByTitle(String title);
    String getProviderName();
}

@Data
@Builder
public class IsbnLookupResult {
    private String isbn;
    private String title;
    private List<String> authors;
    private String publisher;
    private Integer publishYear;
    private String providerSource;
}
```

**ISBNdb Implementation:**
```java
@Service
@Profile("isbn-provider-isbndb")
public class IsbnDbProvider implements IsbnLookupProvider {
    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;

    @Value("${isbn.isbndb.api-key}")
    private String apiKey;

    public IsbnDbProvider(WebClient.Builder webClientBuilder,
                         CircuitBreakerRegistry circuitBreakerRegistry) {
        this.webClient = webClientBuilder
            .baseUrl("https://api2.isbndb.com")
            .defaultHeader("Authorization", apiKey)
            .build();

        this.circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("isbndb");
    }

    @Override
    public Optional<IsbnLookupResult> findIsbnByTitle(String title, String author) {
        return Try.of(() -> circuitBreaker.executeSupplier(() -> {
            IsbnDbResponse response = webClient.get()
                .uri("/book/{title}?author={author}", title, author)
                .retrieve()
                .bodyToMono(IsbnDbResponse.class)
                .block(Duration.ofSeconds(5));

            return Optional.ofNullable(response)
                .map(this::toIsbnLookupResult);
        }))
        .recover(throwable -> {
            log.warn("ISBNdb lookup failed for title: {}", title, throwable);
            return Optional.empty();
        })
        .get();
    }

    private IsbnLookupResult toIsbnLookupResult(IsbnDbResponse response) {
        return IsbnLookupResult.builder()
            .isbn(response.getIsbn13())
            .title(response.getTitle())
            .authors(response.getAuthors())
            .publisher(response.getPublisher())
            .publishYear(response.getDate_published())
            .providerSource("ISBNdb")
            .build();
    }
}
```

**Google Books Implementation:**
```java
@Service
@Profile("isbn-provider-googlebooks")
public class GoogleBooksProvider implements IsbnLookupProvider {
    private final WebClient webClient;
    private final CircuitBreaker circuitBreaker;

    @Value("${isbn.googlebooks.api-key:}")
    private String apiKey;

    public GoogleBooksProvider(WebClient.Builder webClientBuilder,
                              CircuitBreakerRegistry circuitBreakerRegistry) {
        this.webClient = webClientBuilder
            .baseUrl("https://www.googleapis.com/books/v1")
            .build();

        this.circuitBreaker = circuitBreakerRegistry
            .circuitBreaker("googlebooks");
    }

    @Override
    public Optional<IsbnLookupResult> findIsbnByTitle(String title, String author) {
        return Try.of(() -> circuitBreaker.executeSupplier(() -> {
            String query = String.format("intitle:%s+inauthor:%s", title, author);

            GoogleBooksResponse response = webClient.get()
                .uri(uriBuilder -> uriBuilder
                    .path("/volumes")
                    .queryParam("q", query)
                    .queryParam("key", apiKey)
                    .build())
                .retrieve()
                .bodyToMono(GoogleBooksResponse.class)
                .block(Duration.ofSeconds(5));

            return Optional.ofNullable(response)
                .filter(r -> r.getTotalItems() > 0)
                .map(r -> r.getItems().get(0))
                .map(this::toIsbnLookupResult);
        }))
        .recover(throwable -> {
            log.warn("Google Books lookup failed for title: {}", title, throwable);
            return Optional.empty();
        })
        .get();
    }

    private IsbnLookupResult toIsbnLookupResult(GoogleBooksItem item) {
        VolumeInfo info = item.getVolumeInfo();
        return IsbnLookupResult.builder()
            .isbn(extractIsbn13(info.getIndustryIdentifiers()))
            .title(info.getTitle())
            .authors(info.getAuthors())
            .publisher(info.getPublisher())
            .publishYear(extractYear(info.getPublishedDate()))
            .providerSource("Google Books")
            .build();
    }
}
```

**Fallback Service:**
```java
@Service
@Primary
public class FallbackIsbnLookupService implements IsbnLookupProvider {
    private final List<IsbnLookupProvider> providers;

    public FallbackIsbnLookupService(List<IsbnLookupProvider> providers) {
        this.providers = providers.stream()
            .sorted(Comparator.comparing(IsbnLookupProvider::getProviderName))
            .collect(Collectors.toList());
    }

    @Override
    public Optional<IsbnLookupResult> findIsbnByTitle(String title, String author) {
        for (IsbnLookupProvider provider : providers) {
            try {
                Optional<IsbnLookupResult> result =
                    provider.findIsbnByTitle(title, author);

                if (result.isPresent()) {
                    log.info("ISBN found using provider: {}",
                        provider.getProviderName());
                    return result;
                }
            } catch (Exception e) {
                log.warn("Provider {} failed, trying next",
                    provider.getProviderName(), e);
            }
        }

        log.warn("All ISBN providers failed for title: {}", title);
        return Optional.empty();
    }
}
```

#### Configuration Example

```yaml
# application.yml
isbn:
  lookup:
    enabled: true
    fallback: true
    providers:
      - isbndb
      - googlebooks

  isbndb:
    api-key: ${ISBNDB_API_KEY}
    base-url: https://api2.isbndb.com

  googlebooks:
    api-key: ${GOOGLE_BOOKS_API_KEY:}  # Optional
    base-url: https://www.googleapis.com/books/v1

# Resilience4j Circuit Breaker
resilience4j:
  circuitbreaker:
    instances:
      isbndb:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 60s
        sliding-window-size: 10
        permitted-number-of-calls-in-half-open-state: 3
      googlebooks:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 60s
        sliding-window-size: 10
```

#### Enhanced BookService

```java
@Service
@RequiredArgsConstructor
public class BookServiceImpl implements BookService {
    private final BookRepository bookRepository;
    private final IsbnLookupProvider isbnLookup;

    @Value("${isbn.lookup.enabled:false}")
    private boolean isbnLookupEnabled;

    /**
     * New method: Register book by title (ISBN lookup)
     */
    public Book registerBookByTitle(RegisterBookByTitleRequest request) {
        // Lookup ISBN from external provider
        if (!isbnLookupEnabled) {
            throw new UnsupportedOperationException(
                "ISBN lookup is not enabled");
        }

        Optional<IsbnLookupResult> isbnResult =
            isbnLookup.findIsbnByTitle(
                request.getTitle(),
                request.getAuthor()
            );

        if (isbnResult.isEmpty()) {
            throw new NotFoundException(
                "Could not find ISBN for title: " + request.getTitle());
        }

        // Use found ISBN to register book
        Isbn isbn = new Isbn(isbnResult.get().getIsbn());

        // Check if book already exists
        if (bookRepository.findByIsbn(isbn).isPresent()) {
            throw new ConflictException(
                "Book with ISBN " + isbn + " already exists");
        }

        // Create book with enriched data
        Book book = Book.builder()
            .isbn(isbn)
            .title(new Title(isbnResult.get().getTitle()))
            .description(new Description(
                "Publisher: " + isbnResult.get().getPublisher()))
            .genre(request.getGenre())
            .authors(request.getAuthors())
            .build();

        return bookRepository.save(book);
    }
}
```

#### REST Endpoint

```java
@RestController
@RequestMapping("/api/books")
public class BookController {

    @PostMapping("/by-title")
    @PreAuthorize("hasRole('LIBRARIAN')")
    public ResponseEntity<BookView> registerBookByTitle(
            @Valid @RequestBody RegisterBookByTitleRequest request) {

        Book book = bookService.registerBookByTitle(request);
        BookView view = bookViewMapper.toBookView(book);

        return ResponseEntity
            .created(URI.create("/api/books/" + book.getIsbn()))
            .body(view);
    }
}

@Data
public class RegisterBookByTitleRequest {
    @NotBlank
    private String title;

    @NotBlank
    private String author;

    @NotNull
    private Genre genre;

    private Set<Author> authors;
}
```

---

### 6.4 Alternative 4: Pluggable ID Generation

#### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                  Service Layer                              │
│         (BookService, ReaderService, etc.)                  │
└────────────────────────┬────────────────────────────────────┘
                         │ Uses
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              IdGeneratorFactory                             │
│         (Strategy selection by entity type)                 │
└────────┬────────────────────────────────────────┬───────────┘
         │                                        │
         │ Configuration-based                    │
         │ strategy selection                     │
         │                                        │
┌────────▼──────────┐    ┌──────────┐   ┌────────▼──────────┐
│ Year-Sequence Gen │    │ UUID Gen │   │ Auto-Increment Gen│
│ (ReaderNumber,    │    │ (Future) │   │ (Author Number)   │
│  LendingNumber)   │    │          │   │                   │
└───────────────────┘    └──────────┘   └───────────────────┘
```

#### Implementation Strategy

**Generic ID Generator Interface:**
```java
public interface IdGenerator<T> {
    T generate(GenerationContext context);
    boolean validate(T id);
    Class<T> getIdType();
}

@Data
@Builder
public class GenerationContext {
    private Object entity;
    private Map<String, Object> parameters;
    private EntityManager entityManager;
}
```

**Year-Sequence Implementation:**
```java
@Component
@ConditionalOnProperty(
    name = "id.generation.reader.strategy",
    havingValue = "year-sequence"
)
public class YearSequenceReaderNumberGenerator
        implements IdGenerator<ReaderNumber> {

    private final EntityManager entityManager;

    @Override
    public ReaderNumber generate(GenerationContext context) {
        int year = LocalDate.now().getYear();
        int sequence = getNextSequence(year);
        return new ReaderNumber(year, sequence);
    }

    private int getNextSequence(int year) {
        String query = "SELECT MAX(CAST(SUBSTRING(r.readerNumber.number, 6) AS int)) " +
                      "FROM ReaderDetails r " +
                      "WHERE r.readerNumber.number LIKE :yearPattern";

        Integer maxSeq = entityManager.createQuery(query, Integer.class)
            .setParameter("yearPattern", year + "/%")
            .getSingleResult();

        return (maxSeq != null ? maxSeq : 0) + 1;
    }

    @Override
    public boolean validate(ReaderNumber id) {
        return id != null &&
               id.toString().matches("\\d{4}/\\d+");
    }
}
```

**UUID Implementation:**
```java
@Component
@ConditionalOnProperty(
    name = "id.generation.reader.strategy",
    havingValue = "uuid"
)
public class UuidReaderNumberGenerator
        implements IdGenerator<ReaderNumber> {

    @Override
    public ReaderNumber generate(GenerationContext context) {
        String uuid = UUID.randomUUID().toString();
        return new ReaderNumber(uuid);  // Assumes ReaderNumber accepts String
    }

    @Override
    public boolean validate(ReaderNumber id) {
        try {
            UUID.fromString(id.toString());
            return true;
        } catch (IllegalArgumentException e) {
            return false;
        }
    }
}
```

**Factory Configuration:**
```java
@Configuration
public class IdGenerationConfig {

    @Bean
    public IdGeneratorRegistry idGeneratorRegistry(
            List<IdGenerator<?>> generators) {
        return new IdGeneratorRegistry(generators);
    }
}

@Component
public class IdGeneratorRegistry {
    private final Map<Class<?>, IdGenerator<?>> generators;

    public IdGeneratorRegistry(List<IdGenerator<?>> generators) {
        this.generators = generators.stream()
            .collect(Collectors.toMap(
                IdGenerator::getIdType,
                Function.identity()
            ));
    }

    @SuppressWarnings("unchecked")
    public <T> IdGenerator<T> getGenerator(Class<T> idType) {
        IdGenerator<T> generator = (IdGenerator<T>) generators.get(idType);
        if (generator == null) {
            throw new IllegalArgumentException(
                "No ID generator found for type: " + idType.getName());
        }
        return generator;
    }
}
```

**Service Layer Integration:**
```java
@Service
@RequiredArgsConstructor
public class ReaderServiceImpl implements ReaderService {
    private final ReaderRepository readerRepository;
    private final IdGeneratorRegistry idGeneratorRegistry;

    @Transactional
    public ReaderDetails registerReader(RegisterReaderRequest request) {
        // Generate reader number using configured strategy
        IdGenerator<ReaderNumber> generator =
            idGeneratorRegistry.getGenerator(ReaderNumber.class);

        ReaderNumber readerNumber = generator.generate(
            GenerationContext.builder()
                .parameters(Map.of("year", LocalDate.now().getYear()))
                .build()
        );

        // Create reader with generated ID
        ReaderDetails reader = ReaderDetails.builder()
            .readerNumber(readerNumber)
            .birthDate(request.getBirthDate())
            .phoneNumber(request.getPhoneNumber())
            .gdprConsent(request.getGdprConsent())
            .build();

        return readerRepository.save(reader);
    }
}
```

#### Configuration Example

```yaml
# application.yml
id:
  generation:
    reader:
      strategy: year-sequence  # Options: year-sequence, uuid, auto-increment
      prefix: ""               # Optional prefix

    lending:
      strategy: year-sequence
      prefix: "L-"

    author:
      strategy: auto-increment
      start: 1000  # Starting number
```

---

## 7. Implementation Roadmap

### 7.1 Module Prioritization

Based on current architecture analysis, prioritize modules by:
1. **Ease of implementation** (existing abstraction quality)
2. **Business value** (performance improvement potential)
3. **Risk** (complexity, external dependencies)

| Module | Priority | Rationale | Estimated Effort |
|--------|----------|-----------|------------------|
| **Book Repository + Redis Cache** | 🔴 High | Most queried entity (Top 5, search), good abstraction exists | 2-3 days |
| **Genre Repository + Redis Cache** | 🔴 High | Simple entity, high read frequency (Top 5 genres) | 1 day |
| **Author Repository + Redis Cache** | 🟡 Medium | Moderate complexity, co-author relationships | 2 days |
| **Reader Repository + MongoDB** | 🟡 Medium | Flexible schema beneficial (interests, activity logs) | 3-4 days |
| **Lending Repository + MongoDB** | 🟢 Low | Complex aggregations, but SQL works well | 4-5 days |
| **ISBN Lookup Integration** | 🟡 Medium | New feature, external dependency, circuit breaker needed | 2-3 days |
| **ID Generation Abstraction** | 🟢 Low | Nice-to-have, current approach works | 2 days |

### 7.2 Phased Implementation Plan

#### Phase 1: Caching Layer (Week 1-2)
**Goal:** Improve read performance with Redis caching

**Tasks:**
1. Add Spring Data Redis dependencies
2. Configure Redis connection (local/Docker)
3. Implement `CacheStrategy` interface
4. Create `CachedBookRepository` decorator
5. Implement cache invalidation strategy
6. Test with Book, Genre, Author repositories
7. Performance benchmarking (before/after)

**Deliverables:**
- Redis configuration (Docker Compose)
- Cached repository implementations
- Cache key design documentation
- Performance test results

**Configuration Files:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes

volumes:
  redis-data:
```

#### Phase 2: SQL + Redis Alternative (Week 3)
**Goal:** Migrate from H2 to PostgreSQL with Redis caching

**Tasks:**
1. Set up PostgreSQL (Docker)
2. Update datasource configuration
3. Migrate native SQL queries (H2 → PostgreSQL)
4. Test schema generation (Flyway/Liquibase)
5. Integration testing (all CRUD + queries)
6. Load testing with Redis cache

**Deliverables:**
- PostgreSQL Docker configuration
- Migration scripts (if using Flyway)
- Updated native queries
- Integration test suite

#### Phase 3: MongoDB Alternative (Week 4-5)
**Goal:** Implement MongoDB persistence option

**Tasks:**
1. Design document models (Book, Reader, Lending)
2. Add Spring Data MongoDB dependencies
3. Implement MongoDB repository adapters
4. Create profile-based configuration (`persistence-mongodb`)
5. Data migration utility (H2 → MongoDB)
6. Integration testing
7. Performance comparison (SQL vs. MongoDB)

**Deliverables:**
- Document model specifications
- MongoDB repository implementations
- Data migration script
- Performance comparison report

#### Phase 4: ISBN Lookup Integration (Week 6)
**Goal:** External ISBN enrichment

**Tasks:**
1. Design `IsbnLookupService` abstraction
2. Implement ISBNdb provider
3. Implement Google Books provider
4. Add Resilience4j circuit breaker
5. Implement fallback strategy
6. Create REST endpoint (`POST /api/books/by-title`)
7. API key management (environment variables)

**Deliverables:**
- ISBN lookup service implementations
- Circuit breaker configuration
- API documentation (OpenAPI)
- POSTMAN collection updates

#### Phase 5: Configuration Management (Week 7)
**Goal:** Centralized configuration strategy

**Tasks:**
1. Create Spring profiles for each alternative:
   - `sql-redis`
   - `mongodb-redis`
   - `isbn-isbndb`
   - `isbn-googlebooks`
2. Implement ID generation factory
3. Document configuration options
4. Create deployment guides

**Deliverables:**
- Profile configuration files
- Configuration documentation
- Deployment guide (Docker Compose for all alternatives)

### 7.3 Testing Strategy

**Unit Tests:**
- Mock repository implementations
- Test cache decorator logic
- Test ID generation strategies
- Test ISBN provider adapters

**Integration Tests:**
- Test each persistence alternative (SQL, MongoDB)
- Test cache invalidation
- Test external API integration (with WireMock)
- Test fallback strategies

**Performance Tests:**
- Benchmark Top 5 queries (cached vs. uncached)
- Load testing with JMeter/Gatling
- Compare SQL vs. MongoDB performance

**Configuration Tests:**
- Test profile activation
- Test property injection
- Test conditional bean creation

### 7.4 Documentation Requirements

1. **Architecture Documentation** (UPDATE arc42)
   - Section 4: Solution Strategy (add alternatives)
   - Section 7: Deployment View (add configurations)
   - Section 9: Architecture Decisions (add ADRs)

2. **Configuration Guide**
   - How to select persistence strategy
   - How to configure Redis
   - How to configure external ISBN APIs
   - Environment variable reference

3. **Deployment Guide**
   - Docker Compose configurations
   - Environment-specific profiles
   - Scaling considerations

4. **API Documentation**
   - New endpoint: `POST /api/books/by-title`
   - Updated OpenAPI specification
   - POSTMAN collection updates

---

## 8. Risk Mitigation

### 8.1 Technical Risks

| Risk | Mitigation Strategy |
|------|---------------------|
| **Cache consistency issues** | Implement robust invalidation strategy, use TTL, monitor cache hit rate |
| **MongoDB data migration failures** | Create rollback plan, test migration on staging data first |
| **External API rate limiting** | Implement caching of ISBN lookups, fallback to multiple providers |
| **Circuit breaker false positives** | Tune thresholds based on production metrics, implement manual override |
| **Performance degradation** | Comprehensive benchmarking before deployment, canary releases |

### 8.2 Project Risks

| Risk | Mitigation Strategy |
|------|---------------------|
| **Timeline overrun** | Prioritize high-value features (Phase 1-2), defer low-priority items |
| **Team skill gaps** | Provide training resources, pair programming for complex tasks |
| **Scope creep** | Strict adherence to defined alternatives, no additional features |

---

## 9. Success Criteria

### 9.1 Functional Criteria

- ✅ Support at least 2 persistence strategies (SQL+Redis, MongoDB+Redis)
- ✅ Support at least 2 ISBN providers (ISBNdb, Google Books)
- ✅ Configuration-time strategy selection (no code changes)
- ✅ Backward compatibility with existing H2 implementation
- ✅ All existing tests pass with each alternative

### 9.2 Quality Criteria

- ✅ **Performance:** 50%+ reduction in Top 5 query latency (with Redis)
- ✅ **Availability:** Circuit breaker prevents cascading failures
- ✅ **Modifiability:** Add new persistence/ISBN provider in <1 day
- ✅ **Testability:** 80%+ code coverage for new components

### 9.3 Documentation Criteria

- ✅ Updated arc42 architecture document
- ✅ Configuration guide with examples
- ✅ Deployment guide with Docker Compose
- ✅ Updated API documentation (OpenAPI)

---

## Appendix A: Technology Comparison Matrix

| Criterion | SQL + Redis | MongoDB + Redis |
|-----------|-------------|-----------------|
| **Data Consistency** | Strong (ACID) | Eventual (unless using transactions) |
| **Schema Flexibility** | Low (migrations needed) | High (dynamic schema) |
| **Query Capabilities** | Rich (SQL, JPA) | Good (aggregation framework) |
| **Horizontal Scalability** | Limited (read replicas) | Excellent (sharding) |
| **Team Familiarity** | High (current stack) | Low (learning curve) |
| **Performance (reads)** | Good (with cache) | Good (with cache) |
| **Performance (writes)** | Moderate (transactions) | Faster (no transactions) |
| **Operational Complexity** | Low (mature tooling) | Moderate (cluster management) |
| **Best For** | Production deployment | Rapid prototyping, scalability |

---

## Appendix B: Configuration Reference

### B.1 Profile Summary

| Profile | Persistence | Cache | ISBN Lookup |
|---------|-------------|-------|-------------|
| `default` | H2 (embedded) | None | Disabled |
| `sql-redis` | PostgreSQL | Redis | Optional |
| `mongodb-redis` | MongoDB | Redis | Optional |
| `isbn-isbndb` | - | - | ISBNdb |
| `isbn-googlebooks` | - | - | Google Books |

**Activation Example:**
```bash
# SQL + Redis + ISBNdb
java -jar psoft-g1.jar --spring.profiles.active=sql-redis,isbn-isbndb

# MongoDB + Redis + Google Books
java -jar psoft-g1.jar --spring.profiles.active=mongodb-redis,isbn-googlebooks
```

### B.2 Environment Variables

```bash
# Database
DB_USERNAME=library_user
DB_PASSWORD=secure_password
MONGODB_URI=mongodb://localhost:27017/library

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# External APIs
ISBNDB_API_KEY=your_isbndb_key
GOOGLE_BOOKS_API_KEY=your_google_books_key
```

---

**END OF ADD ANALYSIS**
