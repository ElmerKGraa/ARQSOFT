# Genre Classification System

**Project**: Library Management System (PSOFT-G1)
**Date**: 2025-10-29
**Version**: 1.0
**Status**: Documentation and Enhancement

---

## Table of Contents
1. [Introduction](#introduction)
2. [Current System (As-Is)](#current-system-as-is)
3. [Domain Model](#domain-model)
4. [Classification Rules](#classification-rules)
5. [API Operations](#api-operations)
6. [Reporting and Analytics](#reporting-and-analytics)
7. [Data Model](#data-model)
8. [Business Rules](#business-rules)
9. [Enhancement Recommendations](#enhancement-recommendations)
10. [Integration with Other Modules](#integration-with-other-modules)

---

## Introduction

The Genre Classification System is a core component of the Library Management System that organizes books into categories (genres) to facilitate discovery, browsing, and analytics. This document provides comprehensive documentation of the current system and proposes enhancements.

### Purpose

- **Book Organization**: Categorize books for easier discovery
- **Search Facilitation**: Enable genre-based search and filtering
- **Analytics**: Support lending analytics and reporting by genre
- **User Personalization**: Enable interest-based book suggestions for readers

### Scope

The Genre Classification System encompasses:
- Genre entity and value object
- Genre management operations (CRUD)
- Genre-based book categorization
- Genre-based analytics and reporting
- Reader interest management

---

## Current System (As-Is)

### Overview

The current implementation uses a **simple, flat genre classification** where each book is assigned to exactly one genre. Genres are represented as simple string values with validation.

### Key Characteristics

1. **Single Genre per Book**: Each book belongs to exactly one genre
2. **String-Based Classification**: Genre represented as a simple string (max 100 characters)
3. **No Hierarchy**: Flat structure with no parent-child relationships
4. **Manual Assignment**: Librarians assign genres when creating/updating books
5. **Bootstrapped Data**: Initial genres loaded via bootstrapping process
6. **Reader Interests**: Readers can select multiple genres as interests

### Architecture

```
┌─────────────────────────────────────┐
│     Genre Management Module         │
├─────────────────────────────────────┤
│  API Layer                          │
│  - GenreController                  │
│  - GenreView, GenreViewMapper      │
├─────────────────────────────────────┤
│  Business Layer                     │
│  - GenreService                     │
│  - GenreServiceImpl                 │
├─────────────────────────────────────┤
│  Data Layer                         │
│  - GenreRepository                  │
│  - SpringDataGenreRepository        │
├─────────────────────────────────────┤
│  Domain Model                       │
│  - Genre (Entity)                   │
└─────────────────────────────────────┘
```

### Implementation Details

**Core Components**:
- Entity: [Genre.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/model/Genre.java)
- Repository: [GenreRepository.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/repositories/GenreRepository.java)
- Service: [GenreServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Controller: [GenreController.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java)

---

## Domain Model

### Genre Entity

**Purpose**: Represents a book genre/category in the system.

**Attributes**:

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| pk | Long | Primary Key, Auto-generated | Internal database identifier |
| genre | String | Unique, Not Null, 1-100 chars | Genre name/label |

**Implementation**: [Genre.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/model/Genre.java)

```java
@Entity
@Table
public class Genre {
    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    long pk;

    @Size(min = 1, max = 100, message = "Genre name must be between 1 and 100 characters")
    @Column(unique=true, nullable=false, length = 100)
    @Getter
    String genre;

    public Genre(String genre) {
        setGenre(genre);
    }

    private void setGenre(String genre) {
        if(genre == null)
            throw new IllegalArgumentException("Genre cannot be null");
        if(genre.isBlank())
            throw new IllegalArgumentException("Genre cannot be blank");
        if(genre.length() > 100)
            throw new IllegalArgumentException("Genre has a maximum of 100 characters");
        this.genre = genre;
    }
}
```

**Design Patterns**:
- **Value Object Pattern**: Genre behaves like a value object (immutable after creation)
- **Entity Pattern**: Managed by JPA as an entity with identity
- **Validation Pattern**: Business rules enforced in constructor and setter

---

### Relationships

#### Book → Genre (Many-to-One)

**Cardinality**: Many books belong to one genre

**Implementation**: [Book.java:40-42](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L40-L42)

```java
@Getter
@ManyToOne
@NotNull
Genre genre;
```

**Business Rules**:
- Genre is mandatory for books
- Cannot create book without genre
- Genre cannot be null

#### ReaderDetails → Genre (Many-to-Many)

**Cardinality**: Readers can have multiple genre interests, genres can interest multiple readers

**Implementation**: [ReaderDetails.java:56-59](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L56-L59)

```java
@Getter
@Setter
@ManyToMany
private List<Genre> interestList;
```

**Business Rules**:
- Readers can have zero or more genre interests
- Used for personalized book suggestions
- Can be updated by reader

---

### Domain Model Diagram

```
┌────────────────┐
│     Genre      │
│  <<Entity>>    │
├────────────────┤
│ - pk: Long     │
│ - genre: String│
├────────────────┤
│ + Genre(String)│
│ + toString()   │
└────────┬───────┘
         │
         │ 1
         │
         │ *
┌────────▼───────────────────┐
│         Book               │
│      <<Aggregate>>         │
├────────────────────────────┤
│ - isbn: Isbn               │
│ - title: Title             │
│ - genre: Genre             │◄────── Mandatory
│ - authors: List<Author>    │
│ - description: Description │
└────────────────────────────┘

         │ *
         │
         │
         │ *
┌────────▼───────────────────┐
│    ReaderDetails           │
│     <<Aggregate>>          │
├────────────────────────────┤
│ - readerNumber: Number     │
│ - reader: Reader           │
│ - interestList: List<Genre>│◄────── Optional (0..*)
│ - birthDate: BirthDate     │
└────────────────────────────┘
```

---

## Classification Rules

### Genre Assignment Rules

#### Rule 1: Single Genre per Book
**Rule**: Each book MUST be assigned to exactly one genre.

**Rationale**:
- Simplifies classification and browsing
- Reduces complexity in reporting
- Easier for users to understand

**Implementation**: Enforced by database schema (NOT NULL constraint) and domain validation.

**Example**:
- ✅ Book "1984" → Genre "Science Fiction"
- ❌ Book "1984" → Genres ["Science Fiction", "Dystopian"] (not allowed in current system)

---

#### Rule 2: Genre Name Uniqueness
**Rule**: Genre names MUST be unique (case-sensitive).

**Rationale**:
- Prevents duplicate classifications
- Ensures consistent categorization
- Simplifies genre selection in UI

**Implementation**: Unique constraint on `genre` column.

**Example**:
- ✅ "Science Fiction" and "Historical Fiction" (different genres)
- ❌ "Science Fiction" twice (duplicate not allowed)
- ✅ "Science Fiction" and "science fiction" (different - case sensitive)

---

#### Rule 3: Genre Name Validation
**Rule**: Genre name MUST be:
- Not null
- Not blank (empty or whitespace only)
- Between 1 and 100 characters

**Rationale**:
- Prevents meaningless categories
- Ensures displayable names
- Limits database storage

**Implementation**: Bean validation and domain logic in Genre constructor.

**Examples**:
- ✅ "Science Fiction" (valid)
- ✅ "Mystery" (valid)
- ❌ "" (blank, invalid)
- ❌ null (null, invalid)
- ❌ "A".repeat(101) (too long, invalid)

---

#### Rule 4: Genre Required for Books
**Rule**: Books MUST have a genre assigned before persistence.

**Rationale**:
- Ensures all books are categorized
- Supports genre-based search and filtering
- Enables genre-based analytics

**Implementation**: NOT NULL constraint, validation in Book constructor.

**Example**: [Book.java:70-72](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L70-L72)

```java
if(genre==null)
    throw new IllegalArgumentException("Genre cannot be null");
setGenre(genre);
```

---

#### Rule 5: Genre Immutability
**Rule**: Genre names cannot be changed after creation (in current implementation).

**Rationale**:
- Simplifies system (no update cascade required)
- Genres are reference data
- Changes would require migration

**Implementation**: No update method in GenreService for genre names.

**Note**: This is a limitation that could be addressed in enhancements.

---

### Reader Interest Rules

#### Rule 6: Multiple Genre Interests
**Rule**: Readers CAN have zero or more genre interests.

**Rationale**:
- Enables personalized recommendations
- Reflects real-world reader preferences
- Optional feature (readers can opt out)

**Implementation**: Many-to-many relationship via join table.

**Examples**:
- ✅ Reader with no interests (valid)
- ✅ Reader interested in ["Science Fiction", "Mystery"]
- ✅ Reader interested in all genres (valid)

---

#### Rule 7: Interest-Based Suggestions
**Rule**: Book suggestions SHALL be based on reader's genre interests, with a configurable limit per genre.

**Configuration**: [library.properties:9](Original/src/main/resources/config/library.properties#L9)
```properties
suggestionsLimitPerGenre=2
```

**Rationale**:
- Personalizes user experience
- Limits result set size
- Configurable for different business strategies

**Implementation**: BookService.getSuggestions() uses reader interests.

---

## API Operations

### Genre Operations

#### 1. List All Genres (Implicit)

**Endpoint**: Not explicitly exposed, but available via service

**Service Method**: `GenreService.findAll()`

**Implementation**: [GenreServiceImpl.java:30-32](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L30-L32)

**Returns**: Iterable of all genres in the system

**Use Case**: Genre selection dropdown in book creation/editing form

---

#### 2. Get Genre by Name (Implicit)

**Endpoint**: Not explicitly exposed

**Service Method**: `GenreService.findByString(String name)`

**Implementation**: [GenreServiceImpl.java:25-27](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L25-L27)

**Returns**: Optional<Genre>

**Use Case**: Internal use for genre lookup when creating/updating books

---

#### 3. Create/Save Genre (Implicit)

**Endpoint**: Not explicitly exposed (internal use)

**Service Method**: `GenreService.save(Genre genre)`

**Implementation**: [GenreServiceImpl.java:41-43](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L41-L43)

**Use Case**: Bootstrapping initial genres, or creating new genres programmatically

**Note**: No public API for genre creation - genres typically bootstrapped or added through data migration.

---

### Genre Analytics Operations

#### 4. Get Top 5 Genres by Book Count

**Endpoint**: `GET /api/genres/top5`

**Access**: Librarian only

**Service Method**: `GenreService.findTopGenreByBooks()`

**Implementation**: [GenreServiceImpl.java:35-38](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L35-L38)

**Returns**: List of top 5 genres with book count

**Response Example**:
```json
{
  "items": [
    {"genre": "Science Fiction", "bookCount": 45},
    {"genre": "Mystery", "bookCount": 38},
    {"genre": "Historical Fiction", "bookCount": 32},
    {"genre": "Romance", "bookCount": 28},
    {"genre": "Thriller", "bookCount": 25}
  ]
}
```

**Use Case**: B10 - Library analytics dashboard

**Security**: [SecurityConfig.java:155](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L155)

---

#### 5. Get Average Lendings Per Genre

**Endpoint**: `POST /api/genres/avgLendingsPerGenre`

**Access**: Librarian only

**Service Method**: `GenreService.getAverageLendings(GetAverageLendingsQuery query, Page page)`

**Parameters**:
- `year`: Year for analysis
- `month`: Month for analysis (1-12)
- `page`: Pagination (optional, default: page 1, size 10)

**Implementation**: [GenreServiceImpl.java:51-58](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L51-L58)

**Returns**: List of genres with average lendings in specified month

**Response Example**:
```json
{
  "items": [
    {"genre": "Science Fiction", "averageLendings": 12.5},
    {"genre": "Mystery", "averageLendings": 10.3},
    {"genre": "Romance", "averageLendings": 8.7}
  ]
}
```

**Use Case**: B14 - Lending trends analysis by genre

**Security**: [SecurityConfig.java:157](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L157)

---

#### 6. Get Lendings Per Month (Last 12 Months)

**Endpoint**: `GET /api/genres/lendingsPerMonthLastTwelveMonths`

**Access**: Librarian only

**Service Method**: `GenreService.getLendingsPerMonthLastYearByGenre()`

**Implementation**: [GenreServiceImpl.java:46-48](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L46-L48)

**Returns**: List of genres with lending counts per month for last 12 months

**Response Example**:
```json
{
  "items": [
    {
      "genre": "Science Fiction",
      "month": "2024-01",
      "lendingCount": 45
    },
    {
      "genre": "Science Fiction",
      "month": "2024-02",
      "lendingCount": 52
    }
  ]
}
```

**Use Case**: B16 - Lending count trends over time by genre

**Security**: [SecurityConfig.java:158](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L158)

---

#### 7. Get Average Duration Per Month Per Genre

**Endpoint**: `GET /api/genres/lendingsAverageDurationPerMonth?startDate={date}&endDate={date}`

**Access**: Librarian only

**Service Method**: `GenreService.getLendingsAverageDurationPerMonth(String start, String end)`

**Parameters**:
- `startDate`: Start date (format: YYYY-MM-DD)
- `endDate`: End date (format: YYYY-MM-DD)

**Validation**:
- Dates must be in YYYY-MM-DD format
- Start date must be before or equal to end date
- Throws IllegalArgumentException if invalid

**Implementation**: [GenreServiceImpl.java:61-81](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java#L61-L81)

**Returns**: List of genres with average lending duration per month

**Response Example**:
```json
{
  "items": [
    {
      "genre": "Science Fiction",
      "month": "2024-01",
      "averageDuration": 12.5
    },
    {
      "genre": "Mystery",
      "month": "2024-01",
      "averageDuration": 10.2
    }
  ]
}
```

**Use Case**: B19 - Average lending duration analysis by genre and month

**Security**: [SecurityConfig.java:159](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L159)

---

## Reporting and Analytics

### Analytics Capabilities

The Genre Classification System supports comprehensive analytics for library management:

#### 1. Collection Management Analytics

**Purpose**: Understand genre distribution in library collection

**Metrics**:
- Total books per genre
- Top N genres by book count
- Genre diversity index (number of distinct genres)

**Business Value**:
- Identify collection gaps (underrepresented genres)
- Guide acquisition strategy
- Balance collection across genres

**Implementation**: `findTopGenreByBooks()` provides top 5 genres

---

#### 2. Lending Behavior Analytics

**Purpose**: Understand patron borrowing patterns by genre

**Metrics**:
- Average lendings per genre per month
- Total lendings per genre over time
- Genre popularity trends (month-over-month growth)

**Business Value**:
- Identify popular genres
- Predict future demand
- Adjust collection based on borrowing patterns

**Implementation**:
- `getAverageLendings()` - average per month
- `getLendingsPerMonthLastYearByGenre()` - trends over time

---

#### 3. Lending Duration Analytics

**Purpose**: Understand how long books of different genres are borrowed

**Metrics**:
- Average lending duration per genre
- Duration trends over time
- Comparison across genres

**Business Value**:
- Identify genres that require longer lending periods
- Optimize lending policies per genre
- Understand reading complexity (longer duration = more complex?)

**Implementation**: `getLendingsAverageDurationPerMonth()`

---

#### 4. Reader Interest Analytics

**Purpose**: Understand reader preferences

**Metrics**:
- Most popular genre interests (not implemented)
- Interest diversity per reader (not implemented)
- Interest-to-borrowing correlation (not implemented)

**Business Value**:
- Validate interest-based recommendations
- Identify opportunities for marketing
- Understand patron demographics

**Implementation Status**: Partial (interest storage exists, analytics not implemented)

---

### Analytics Data Model

```
┌─────────────────────────────────┐
│    GenreBookCountDTO            │
├─────────────────────────────────┤
│ - genre: String                 │
│ - bookCount: Long               │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│    GenreLendingsDTO             │
├─────────────────────────────────┤
│ - genre: String                 │
│ - averageLendings: Double       │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│  GenreLendingsPerMonthDTO       │
├─────────────────────────────────┤
│ - genre: String                 │
│ - month: String/Date            │
│ - lendingCount: Long            │
│ - averageDuration: Double       │
└─────────────────────────────────┘
```

---

## Data Model

### Database Schema

```sql
-- Genre Table
CREATE TABLE GENRE (
    PK BIGINT PRIMARY KEY AUTO_INCREMENT,
    GENRE VARCHAR(100) NOT NULL UNIQUE
);

-- Book-Genre Relationship (Many-to-One)
CREATE TABLE BOOK (
    PK BIGINT PRIMARY KEY AUTO_INCREMENT,
    ISBN VARCHAR(13) NOT NULL UNIQUE,
    TITLE VARCHAR(255) NOT NULL,
    GENRE_PK BIGINT NOT NULL,
    -- ... other columns
    FOREIGN KEY (GENRE_PK) REFERENCES GENRE(PK)
);

-- Reader-Genre Interest (Many-to-Many)
CREATE TABLE READER_DETAILS_INTEREST_LIST (
    READER_DETAILS_PK BIGINT NOT NULL,
    INTEREST_LIST_PK BIGINT NOT NULL,
    PRIMARY KEY (READER_DETAILS_PK, INTEREST_LIST_PK),
    FOREIGN KEY (READER_DETAILS_PK) REFERENCES READER_DETAILS(PK),
    FOREIGN KEY (INTEREST_LIST_PK) REFERENCES GENRE(PK)
);
```

### Entity-Relationship Diagram

```
┌──────────────┐
│    GENRE     │
│──────────────│
│ PK (PK)      │◄──────┐
│ GENRE (UK)   │       │ FK
└──────────────┘       │
                       │
                       │ 1
                       │
                       │
                       │ *
                ┌──────┴───────┐
                │     BOOK     │
                │──────────────│
                │ PK (PK)      │
                │ ISBN (UK)    │
                │ TITLE        │
                │ GENRE_PK (FK)│
                └──────────────┘

┌──────────────┐       *
│    GENRE     │◄───────────────────────┐
│──────────────│                        │
│ PK (PK)      │                        │
│ GENRE (UK)   │                        │
└──────────────┘                        │
                                        │
                                        │
                    ┌───────────────────┴────────────────┐
                    │ READER_DETAILS_INTEREST_LIST       │
                    │ (Join Table)                       │
                    │────────────────────────────────────│
                    │ READER_DETAILS_PK (FK, PK)         │
                    │ INTEREST_LIST_PK (FK, PK)          │
                    └───────────────────┬────────────────┘
                                        │
                                        │
                                        │
                                        ▼
                            ┌──────────────────┐
                            │  READER_DETAILS  │
                            │──────────────────│
                            │ PK (PK)          │
                            │ READER_NUMBER    │
                            │ ...              │
                            └──────────────────┘
```

### Indexes

**Automatically Created**:
- Primary key index on `GENRE.PK`
- Unique index on `GENRE.GENRE`
- Foreign key index on `BOOK.GENRE_PK`

**Recommended Additional Indexes** (for performance):
- Index on `BOOK.GENRE_PK` for genre-based searches
- Composite index on `READER_DETAILS_INTEREST_LIST (READER_DETAILS_PK, INTEREST_LIST_PK)` for interest queries

---

## Business Rules

### Validation Rules

| Rule ID | Rule Description | Enforcement | Error Message |
|---------|------------------|-------------|---------------|
| GEN-VAL-01 | Genre name cannot be null | Domain validation | "Genre cannot be null" |
| GEN-VAL-02 | Genre name cannot be blank | Domain validation | "Genre cannot be blank" |
| GEN-VAL-03 | Genre name max 100 characters | Bean validation + Domain | "Genre has a maximum of 100 characters" |
| GEN-VAL-04 | Genre name must be unique | Database constraint | Conflict exception |
| GEN-VAL-05 | Book must have genre | Domain validation | "Genre cannot be null" |

### Operational Rules

| Rule ID | Rule Description | Configuration | Rationale |
|---------|------------------|---------------|-----------|
| GEN-OPS-01 | Genres loaded via bootstrapping | Bootstrapper.java | Initial system setup |
| GEN-OPS-02 | No public API for genre creation | Security config | Controlled vocabulary |
| GEN-OPS-03 | Genre names are immutable | No update method | Simplifies system |
| GEN-OPS-04 | Suggestions limited per genre | suggestionsLimitPerGenre=2 | Performance, UX |
| GEN-OPS-05 | Top 5 genres pagination | PageRequest.of(0,5) | Report limit |

---

## Enhancement Recommendations

### Priority 1: High-Value Enhancements

#### Enhancement 1: Hierarchical Genre Classification

**Problem**: Current flat structure limits expressiveness

**Proposed Solution**: Implement hierarchical genres with parent-child relationships

**Benefits**:
- More accurate classification (e.g., "Science Fiction" → "Hard Sci-Fi", "Space Opera")
- Better browse navigation (drill-down)
- Improved search relevance

**Implementation Approach**:
```java
@Entity
public class Genre {
    @Id
    private Long pk;

    private String genre;

    @ManyToOne
    @JoinColumn(name = "parent_genre_pk")
    private Genre parentGenre;

    @OneToMany(mappedBy = "parentGenre")
    private List<Genre> subGenres;

    // Methods to navigate hierarchy
    public boolean isLeaf() { return subGenres.isEmpty(); }
    public boolean isRoot() { return parentGenre == null; }
    public List<Genre> getAncestors() { /* ... */ }
}
```

**Database Impact**:
```sql
ALTER TABLE GENRE ADD COLUMN PARENT_GENRE_PK BIGINT;
ALTER TABLE GENRE ADD FOREIGN KEY (PARENT_GENRE_PK) REFERENCES GENRE(PK);
```

**Effort**: Medium (2-3 days)
**Risk**: Low (additive change, backwards compatible)

---

#### Enhancement 2: Multiple Genres per Book

**Problem**: Books often span multiple genres

**Proposed Solution**: Change Book-Genre relationship from Many-to-One to Many-to-Many

**Benefits**:
- More accurate book representation
- Better search recall (find book in multiple genre searches)
- Industry standard (libraries typically use multiple subjects)

**Implementation Approach**:
```java
@Entity
public class Book extends EntityWithPhoto {
    @ManyToMany
    @JoinTable(
        name = "BOOK_GENRE",
        joinColumns = @JoinColumn(name = "BOOK_PK"),
        inverseJoinColumns = @JoinColumn(name = "GENRE_PK")
    )
    private List<Genre> genres = new ArrayList<>();

    // Require at least one genre
    public void addGenre(Genre genre) {
        if (!genres.contains(genre)) {
            genres.add(genre);
        }
    }

    public void removeGenre(Genre genre) {
        if (genres.size() > 1) {  // Keep at least one
            genres.remove(genre);
        } else {
            throw new IllegalStateException("Book must have at least one genre");
        }
    }
}
```

**Database Impact**:
```sql
-- New join table
CREATE TABLE BOOK_GENRE (
    BOOK_PK BIGINT NOT NULL,
    GENRE_PK BIGINT NOT NULL,
    PRIMARY KEY (BOOK_PK, GENRE_PK),
    FOREIGN KEY (BOOK_PK) REFERENCES BOOK(PK),
    FOREIGN KEY (GENRE_PK) REFERENCES GENRE(PK)
);

-- Migrate existing data
INSERT INTO BOOK_GENRE (BOOK_PK, GENRE_PK)
SELECT PK, GENRE_PK FROM BOOK;

-- Remove old column (after validation)
ALTER TABLE BOOK DROP COLUMN GENRE_PK;
```

**Effort**: Medium (3-4 days including data migration)
**Risk**: Medium (breaking change, requires migration)

---

#### Enhancement 3: Genre Management API

**Problem**: No public API to manage genres (create, update, delete)

**Proposed Solution**: Expose RESTful API for genre management

**Benefits**:
- Librarians can add new genres without code changes
- Support genre renaming and merging
- Enable genre lifecycle management

**Implementation Approach**:

**New Endpoints**:
```
POST   /api/genres              - Create new genre (Librarian)
GET    /api/genres              - List all genres (Librarian, Reader)
GET    /api/genres/{id}         - Get genre details (Librarian, Reader)
PATCH  /api/genres/{id}         - Update genre name (Librarian)
DELETE /api/genres/{id}         - Delete genre (Librarian, with constraints)
```

**GenreController Enhancement**:
```java
@RestController
@RequestMapping("/api/genres")
public class GenreController {

    @PostMapping
    @PreAuthorize("hasRole('LIBRARIAN')")
    public GenreView createGenre(@Valid @RequestBody CreateGenreRequest request) {
        final var genre = genreService.create(request.getGenre());
        return genreViewMapper.toGenreView(genre);
    }

    @GetMapping
    @PreAuthorize("hasAnyRole('LIBRARIAN', 'READER')")
    public ListResponse<GenreView> getAllGenres() {
        final var genres = genreService.findAll();
        return new ListResponse<>(genreViewMapper.toGenreView(genres));
    }

    @PatchMapping("/{id}")
    @PreAuthorize("hasRole('LIBRARIAN')")
    public GenreView updateGenre(@PathVariable Long id,
                                  @Valid @RequestBody UpdateGenreRequest request) {
        final var genre = genreService.update(id, request);
        return genreViewMapper.toGenreView(genre);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('LIBRARIAN')")
    public ResponseEntity<Void> deleteGenre(@PathVariable Long id) {
        genreService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Business Rules for Delete**:
- Cannot delete genre if books exist with that genre
- Option 1: Prevent deletion (throw exception)
- Option 2: Reassign books to different genre (require target genre)
- Option 3: Soft delete (mark as inactive, hide from UI)

**Effort**: Low-Medium (2-3 days)
**Risk**: Low (additive change)

---

### Priority 2: Analytics Enhancements

#### Enhancement 4: Genre Recommendation Engine

**Problem**: Current suggestions are simple (based on interests only)

**Proposed Solution**: Implement sophisticated recommendation algorithm

**Recommendation Strategies**:

1. **Collaborative Filtering**:
   - "Readers who borrowed books in genre X also borrowed books in genre Y"
   - Requires tracking reader borrowing history

2. **Content-Based Filtering**:
   - If reader likes "Science Fiction", suggest "Space Opera", "Cyberpunk" (sub-genres)
   - Requires hierarchical genre structure (Enhancement 1)

3. **Popularity-Based**:
   - Recommend trending genres (most borrowed this month)
   - Boost newly added genres

4. **Hybrid Approach**:
   - Combine multiple strategies with configurable weights

**Implementation Outline**:
```java
public interface RecommendationStrategy {
    List<Genre> recommend(ReaderDetails reader, int limit);
}

@Service
public class GenreRecommendationService {
    private final List<RecommendationStrategy> strategies;

    public List<Genre> recommendGenres(ReaderDetails reader, int limit) {
        // Combine recommendations from multiple strategies
        // Apply diversity (don't recommend only similar genres)
        // Filter out genres reader already borrowed extensively
    }
}
```

**Effort**: High (1-2 weeks)
**Risk**: Medium (complex algorithm, requires testing)

---

#### Enhancement 5: Advanced Genre Analytics

**Problem**: Limited analytics capabilities

**Proposed Enhancements**:

1. **Genre Diversity Metrics**:
   - Shannon entropy of genre distribution
   - Herfindahl-Hirschman Index (concentration)
   - Visualization: genre distribution pie chart

2. **Genre Trends**:
   - Month-over-month growth rate per genre
   - Seasonal patterns (e.g., romance in February, horror in October)
   - Predictive analytics (forecast next month's demand)

3. **Cross-Genre Analysis**:
   - Common author genres (authors who write multiple genres)
   - Genre transitions (readers who switch genres)
   - Genre co-borrowing patterns

4. **Reader Segmentation**:
   - Cluster readers by genre preferences
   - Identify "niche" readers vs. "diverse" readers
   - Target marketing by segment

**Implementation**: New AnalyticsService with custom repository queries

**Effort**: Medium-High (1 week)
**Risk**: Low (additive feature)

---

### Priority 3: Quality of Life Improvements

#### Enhancement 6: Genre Aliases and Synonyms

**Problem**: Same genre may have multiple names (e.g., "Sci-Fi" vs. "Science Fiction")

**Proposed Solution**: Support aliases for genres

**Implementation**:
```java
@Entity
public class Genre {
    @Id
    private Long pk;

    @Column(unique=true, nullable=false)
    private String canonicalName;  // Official name

    @ElementCollection
    @CollectionTable(name = "GENRE_ALIASES")
    private Set<String> aliases = new HashSet<>();  // Alternative names

    public boolean matches(String searchTerm) {
        return canonicalName.equalsIgnoreCase(searchTerm)
            || aliases.stream().anyMatch(a -> a.equalsIgnoreCase(searchTerm));
    }
}
```

**Use Cases**:
- Search returns results for both "Sci-Fi" and "Science Fiction"
- Import from external sources with different naming conventions
- Support multilingual genre names (future)

**Effort**: Low (1-2 days)
**Risk**: Low (additive change)

---

#### Enhancement 7: Genre Descriptions and Metadata

**Problem**: Genre name alone doesn't provide enough context

**Proposed Solution**: Add rich metadata to genres

**Additional Fields**:
```java
@Entity
public class Genre {
    private Long pk;
    private String genre;

    // New fields
    @Column(length = 1000)
    private String description;  // What defines this genre

    @Column(length = 500)
    private String examples;  // Example books/authors

    private String iconUrl;  // Visual icon for UI

    private LocalDate createdDate;
    private LocalDate lastModifiedDate;

    @Embedded
    private GenreStatistics statistics;  // Cached stats
}

@Embeddable
class GenreStatistics {
    private int totalBooks;
    private int totalLendings;
    private double averageLendingDuration;
    private LocalDate lastUpdated;
}
```

**Benefits**:
- Better genre selection UI (show description, examples)
- Visual representation (icons/colors)
- Cached statistics for performance

**Effort**: Low-Medium (2-3 days)
**Risk**: Low (additive change)

---

#### Enhancement 8: Genre Tagging (Folksonomy)

**Problem**: Rigid classification doesn't capture nuances

**Proposed Solution**: Allow free-form tags in addition to genres

**Concept**:
- Genres = formal classification (librarian-managed)
- Tags = informal classification (user-contributed)

**Implementation**:
```java
@Entity
public class Tag {
    @Id
    private Long pk;

    @Column(unique = true)
    private String tag;

    private int usageCount;  // How many books have this tag
}

@Entity
public class Book {
    @ManyToMany
    private List<Genre> genres;  // Formal classification

    @ManyToMany
    private List<Tag> tags;  // Informal tags (user-contributed)
}
```

**Example**:
- Genre: "Science Fiction"
- Tags: ["space-opera", "dystopian", "first-contact", "hard-sci-fi"]

**Benefits**:
- Richer discoverability
- Community-driven classification
- Capture attributes not well represented by genres (mood, pacing, themes)

**Effort**: Medium (3-4 days)
**Risk**: Medium (requires moderation to prevent tag spam)

---

## Integration with Other Modules

### Book Management

**Integration Points**:
1. **Book Creation**: Genre required when creating book
   - Service: `BookService.create(CreateBookRequest)`
   - Validates genre exists before creating book

2. **Book Update**: Genre can be changed
   - Service: `BookService.update(String isbn, UpdateBookRequest)`
   - Validates new genre exists

3. **Book Search**: Filter by genre
   - Service: `BookService.findByGenre(String genre, Page page)`
   - Returns all books in specified genre

**Code References**:
- [Book.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java)
- [BookService.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/)

---

### Reader Management

**Integration Points**:
1. **Reader Registration**: Optional genre interests
   - Service: `ReaderService.create(CreateReaderRequest)`
   - Reader can specify zero or more genres of interest

2. **Reader Update**: Modify genre interests
   - Service: `ReaderService.update(UpdateReaderRequest)`
   - Add/remove genres from interest list

3. **Book Suggestions**: Based on genre interests
   - Service: `BookService.getSuggestions(Reader reader)`
   - Uses reader's genre interests to suggest books
   - Configuration: `suggestionsLimitPerGenre=2`

**Code References**:
- [ReaderDetails.java:56-59](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L56-L59)
- [ReaderService.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/)

---

### Lending Management

**Integration Points**:
1. **Lending Analytics**: Genre-based statistics
   - Average lending duration per genre
   - Lending counts per genre per month
   - Top genres by lending volume

2. **Overdue Analysis**: Genre patterns in overdues
   - Potential: Do certain genres have higher overdue rates?
   - Not currently implemented

**Code References**:
- [LendingRepository.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/repositories/)
- [GenreRepository.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/repositories/GenreRepository.java)

---

### Reporting

**Integration Points**:
1. **Top 5 Reports**: Genre appears in multiple top N reports
   - Top 5 genres by book count
   - Top 5 readers per genre (lending count)

2. **Lending Analytics**: Multiple genre-based reports
   - Average lendings per genre per month
   - Lending counts per month last 12 months
   - Average duration per genre per month

**Code References**:
- [GenreController.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java)
- [GenreRepository.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/repositories/GenreRepository.java)

---

## Appendix

### Common Genre Examples

Based on typical library classifications:

**Fiction**:
- Science Fiction
- Fantasy
- Mystery
- Thriller
- Romance
- Historical Fiction
- Literary Fiction
- Horror
- Adventure

**Non-Fiction**:
- Biography
- History
- Science
- Technology
- Self-Help
- Business
- Cookbooks
- Travel
- Art
- Philosophy

**Specialized**:
- Young Adult
- Children's
- Graphic Novels
- Poetry
- Drama
- Essays

### References

- **Domain Documentation**: [Genre.md](Original/Docs/GlobalArtifacts/Entities/Genre.md)
- **Implementation**: [Genre.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/model/Genre.java)
- **Service Layer**: [GenreServiceImpl.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- **REST API**: [GenreController.java](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java)
- **Dewey Decimal Classification**: https://www.oclc.org/en/dewey.html
- **Library of Congress Classification**: https://www.loc.gov/catdir/cpso/lcc.html

---

**Document Control**
Version: 1.0
Last Updated: 2025-10-29
Status: Complete
Next Review: Upon genre system enhancements
