# Functional Requirements

**Project**: Library Management System (PSOFT-G1)
**Date**: 2025-10-29
**Version**: 1.0
**Status**: Extracted from Implementation (Reverse Engineered)

---

## Table of Contents
1. [Introduction](#introduction)
2. [Actor Definitions](#actor-definitions)
3. [Author Management (FR-AUT)](#author-management-fr-aut)
4. [Book Management (FR-BOOK)](#book-management-fr-book)
5. [Reader Management (FR-READ)](#reader-management-fr-read)
6. [Lending Management (FR-LEND)](#lending-management-fr-lend)
7. [Genre Management (FR-GENRE)](#genre-management-fr-genre)
8. [User Management (FR-USER)](#user-management-fr-user)
9. [Reporting (FR-REPORT)](#reporting-fr-report)
10. [Authentication & Authorization (FR-AUTH)](#authentication--authorization-fr-auth)
11. [Requirements Traceability](#requirements-traceability)

---

## Introduction

This document specifies the functional requirements for the Library Management System, extracted from the implemented codebase through reverse engineering. Each requirement is traced to its implementation in the source code.

### Requirement Format

Each requirement follows this structure:
- **ID**: Unique identifier (e.g., FR-AUT-01)
- **Name**: Short descriptive name
- **Description**: What the system shall do
- **Actor**: Who can perform this function
- **Priority**: High/Medium/Low (based on implementation completeness)
- **Implementation**: Source code references
- **REST API**: HTTP method and endpoint
- **Use Case**: Reference to use case documentation (if exists)

---

## Actor Definitions

| Actor | Description | Role in System |
|-------|-------------|----------------|
| **Anonymous** | Unauthenticated user | Can only register as a new reader |
| **Reader** | Registered library patron | Can borrow books, view catalog, manage own profile |
| **Librarian** | Library staff member | Can manage books, authors, readers, lendings, and generate reports |
| **Admin** | System administrator | Has full access to all operations (superuser) |

---

## Author Management (FR-AUT)

### FR-AUT-01: Create Author
**Name**: Create Author
**Description**: The system shall allow librarians to create a new author with name, biography, and optional photo.
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- Author name is mandatory and cannot be forbidden
- Author name must be validated against forbidden names list
- Biography is optional but has length constraints
- Photo upload is optional (max 20KB)

**Implementation**:
- API: `POST /api/authors`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.create()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Domain: [Author.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java)
- Security: [SecurityConfig.java:119](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L119)

**Use Case**: A3/B3 (Phase 2)

---

### FR-AUT-02: Update Author
**Name**: Update Author
**Description**: The system shall allow librarians to update an existing author's name, biography, and/or photo.
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- Optimistic locking enforced (version check)
- Partial updates supported (PATCH semantics)
- Photo can be updated separately

**Implementation**:
- API: `PATCH /api/authors/{authorNumber}`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.update()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Domain: [Author.applyPatch():55-64](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java#L55-L64)
- Security: [SecurityConfig.java:120](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L120)

**Use Case**: A4 (Phase 1)

---

### FR-AUT-03: Get Author Details
**Name**: Get Author by ID
**Description**: The system shall allow readers and librarians to retrieve detailed information about a specific author.
**Actor**: Reader, Librarian
**Priority**: High

**Business Rules**:
- Returns author number, name, biography, and photo URI
- Public information (no sensitive data)

**Implementation**:
- API: `GET /api/authors/{authorNumber}`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.findById()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Security: [SecurityConfig.java:121](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L121)

**Use Case**: A5 (Phase 1)

---

### FR-AUT-04: Search Authors by Name
**Name**: Search Authors by Name
**Description**: The system shall allow users to search for authors by name (partial match).
**Actor**: Reader, Librarian
**Priority**: Medium

**Business Rules**:
- Case-insensitive search
- Partial name matching supported
- Returns paginated results

**Implementation**:
- API: `GET /api/authors?name={name}`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.searchByName()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Security: [SecurityConfig.java:122](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L122)

**Use Case**: A6 (Phase 1)

---

### FR-AUT-05: Get Books by Author
**Name**: Get Books by Author
**Description**: The system shall allow readers to retrieve all books written by a specific author.
**Actor**: Reader
**Priority**: Medium

**Business Rules**:
- Returns books where author is main author or co-author
- Includes book title, ISBN, genre

**Implementation**:
- API: `GET /api/authors/{authorNumber}/books`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.getAuthorBooks()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Security: [SecurityConfig.java:123](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L123)

**Use Case**: B4 (Phase 2)

---

### FR-AUT-06: Get Author Co-authors
**Name**: Get Author Co-authors
**Description**: The system shall allow readers to retrieve all co-authors of a specific author.
**Actor**: Reader
**Priority**: Low

**Business Rules**:
- Co-authors are authors who have written books together
- Returns list of co-authors with shared book count

**Implementation**:
- API: `GET /api/authors/{authorNumber}/coauthors`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.getCoAuthors()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Security: [SecurityConfig.java:127](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L127)

**Use Case**: B5 (Phase 2)

---

### FR-AUT-07: Manage Author Photo
**Name**: Upload/Delete Author Photo
**Description**: The system shall allow librarians to upload or delete an author's photo.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Photo size limit: 20KB
- Supported formats: standard image formats
- Photo stored in file system
- Delete requires version check (optimistic locking)

**Implementation**:
- API: `POST /api/authors/{authorNumber}/photo` (upload)
- API: `DELETE /api/authors/{authorNumber}/photo` (delete)
- API: `GET /api/authors/{authorNumber}/photo` (retrieve)
- Service: [FileStorageService](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/FileStorageService.java)
- Domain: [Author.removePhoto():66-72](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/model/Author.java#L66-L72)
- Security: [SecurityConfig.java:125-126](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L125-L126)

---

## Book Management (FR-BOOK)

### FR-BOOK-01: Create Book
**Name**: Create Book
**Description**: The system shall allow librarians to create a new book with ISBN, title, genre, description, authors, and optional photo.
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- ISBN is unique and mandatory (validated format)
- Book must have at least one author
- Genre is mandatory
- Title is mandatory
- Description is optional
- Photo upload is optional (max 20KB)

**Implementation**:
- API: `PUT /api/books/{isbn}`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.create()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Domain: [Book.java:65-80](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L65-L80)
- Security: [SecurityConfig.java:130](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L130)

**Use Case**: A7/B7 (Phase 2)

---

### FR-BOOK-02: Update Book
**Name**: Update Book
**Description**: The system shall allow librarians to update an existing book's details.
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- Optimistic locking enforced (version check)
- Partial updates supported (PATCH semantics)
- ISBN cannot be changed
- Authors can be updated

**Implementation**:
- API: `PATCH /api/books/{isbn}`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.update()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Domain: [Book.applyPatch():94-122](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L94-L122)
- Security: [SecurityConfig.java:131](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L131)

**Use Case**: A8 (Phase 1)

---

### FR-BOOK-03: Get Book Details
**Name**: Get Book by ISBN
**Description**: The system shall allow users to retrieve detailed information about a specific book.
**Actor**: Reader, Librarian
**Priority**: High

**Business Rules**:
- Returns ISBN, title, genre, authors, description, photo URI
- Public information (no sensitive data)

**Implementation**:
- API: `GET /api/books/{isbn}`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.findByIsbn()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Security: [SecurityConfig.java:134](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L134)

**Use Case**: A9 (Phase 1)

---

### FR-BOOK-04: Search Books by Genre
**Name**: Search Books by Genre
**Description**: The system shall allow users to search for books by genre.
**Actor**: Reader, Librarian
**Priority**: High

**Business Rules**:
- Returns all books matching the specified genre
- Paginated results

**Implementation**:
- API: `GET /api/books?genre={genre}`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.findByGenre()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Security: [SecurityConfig.java:133](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L133)

**Use Case**: A10 (Phase 1)

---

### FR-BOOK-05: Search Books by Title
**Name**: Search Books by Title
**Description**: The system shall allow users to search for books by title (partial match).
**Actor**: Reader, Librarian
**Priority**: Medium

**Business Rules**:
- Case-insensitive search
- Partial title matching supported
- Returns paginated results

**Implementation**:
- API: `GET /api/books?title={title}` or `POST /api/books/search`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.searchByTitle()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Security: [SecurityConfig.java:139](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L139)

**Use Case**: B8 (Phase 2)

---

### FR-BOOK-06: Get Book Suggestions
**Name**: Get Personalized Book Suggestions
**Description**: The system shall provide personalized book suggestions to readers based on their interests.
**Actor**: Reader
**Priority**: Low

**Business Rules**:
- Based on reader's genre interests
- Maximum 2 suggestions per genre (configurable)
- Excludes books already borrowed by reader

**Implementation**:
- API: `GET /api/books/suggestions`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.getSuggestions()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Configuration: [library.properties:9](Original/src/main/resources/config/library.properties#L9)
- Security: [SecurityConfig.java:138](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L138)

**Use Case**: B13 (Phase 2)

---

### FR-BOOK-07: Manage Book Photo
**Name**: Upload/Delete Book Photo
**Description**: The system shall allow librarians to upload or delete a book's cover photo.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Photo size limit: 20KB
- Supported formats: standard image formats
- Photo stored in file system
- Delete requires version check (optimistic locking)

**Implementation**:
- API: `POST /api/books/{isbn}/photo` (upload)
- API: `DELETE /api/books/{isbn}/photo` (delete)
- API: `GET /api/books/{isbn}/photo` (retrieve)
- Service: [FileStorageService](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/FileStorageService.java)
- Domain: [Book.removePhoto():86-92](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/model/Book.java#L86-L92)
- Security: [SecurityConfig.java:136-137](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L136-L137)

---

## Reader Management (FR-READ)

### FR-READ-01: Register Reader
**Name**: Self-Registration
**Description**: The system shall allow anonymous users to register as a new reader.
**Actor**: Anonymous
**Priority**: High

**Business Rules**:
- Minimum age: 12 years (configurable)
- GDPR consent is mandatory
- Marketing and third-party consents are optional
- Username must be unique
- Password must meet strength requirements
- Email and phone number validation
- Photo upload is optional (max 20KB)

**Implementation**:
- API: `POST /api/readers`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.create()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Domain: [ReaderDetails.java:61-81](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L61-L81)
- Configuration: [library.properties:6](Original/src/main/resources/config/library.properties#L6)
- Security: [SecurityConfig.java:116](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L116) (permitAll)

**Use Case**: A11/B12 (Phase 2)

---

### FR-READ-02: Update Reader Profile
**Name**: Update Own Profile
**Description**: The system shall allow readers to update their own profile information.
**Actor**: Reader (self only)
**Priority**: High

**Business Rules**:
- Can update: name, birth date, phone number, consents, interests, photo
- Cannot update: reader number, registration date
- Optimistic locking enforced (version check)
- Reader can only update own profile

**Implementation**:
- API: `PATCH /api/readers/{year}/{seq}` or `PATCH /api/readers`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.update()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Domain: [ReaderDetails.applyPatch():101-151](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L101-L151)
- Security: [SecurityConfig.java:142](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L142)

**Use Case**: A12 (Phase 1)

---

### FR-READ-03: Get Reader Details
**Name**: Get Reader by Number
**Description**: The system shall allow librarians to retrieve detailed information about a specific reader.
**Actor**: Librarian, Reader (self only)
**Priority**: High

**Business Rules**:
- Reader number format: YYYY/SEQ
- Readers can only view own profile
- Librarians can view any reader profile

**Implementation**:
- API: `GET /api/readers/{year}/{seq}`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.findByNumber()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Security: [SecurityConfig.java:152](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L152)

**Use Case**: A13 (Phase 1)

---

### FR-READ-04: Search Readers by Name
**Name**: Search Readers by Name
**Description**: The system shall allow librarians to search for readers by name.
**Actor**: Librarian
**Priority**: Medium

**Business Rules**:
- Partial name matching supported
- Case-insensitive search
- Returns paginated results

**Implementation**:
- API: `GET /api/readers?name={name}` or `POST /api/readers/search`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.searchByName()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Security: [SecurityConfig.java:144](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L144)

**Use Case**: A14 (Phase 1)

---

### FR-READ-05: Get Reader Lendings
**Name**: Get Own Lending History
**Description**: The system shall allow readers to retrieve their own lending history, optionally filtered by book ISBN.
**Actor**: Reader (self only)
**Priority**: High

**Business Rules**:
- Reader can only view own lendings
- Optional filter by ISBN
- Includes current and past lendings

**Implementation**:
- API: `GET /api/readers/{year}/{seq}/lendings?isbn={isbn}`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.getLendings()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Security: [SecurityConfig.java:150](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L150)

**Use Case**: A16 (Phase 1)

---

### FR-READ-06: Manage Reader Photo
**Name**: Upload/Delete Own Photo
**Description**: The system shall allow readers to upload or delete their own photo.
**Actor**: Reader (self only)
**Priority**: Low

**Business Rules**:
- Photo size limit: 20KB
- Supported formats: standard image formats
- Photo stored in file system
- Delete requires version check (optimistic locking)

**Implementation**:
- API: `POST /api/readers/photo` (upload)
- API: `DELETE /api/readers/photo` (delete)
- API: `GET /api/readers/photo` or `GET /api/readers/{year}/{seq}/photo` (retrieve)
- Service: [FileStorageService](Original/src/main/java/pt/psoft/g1/psoftg1/shared/services/FileStorageService.java)
- Domain: [ReaderDetails.removePhoto():153-159](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/model/ReaderDetails.java#L153-L159)
- Security: [SecurityConfig.java:147-148,151](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L147-L148)

---

## Lending Management (FR-LEND)

### FR-LEND-01: Create Lending
**Name**: Create New Lending
**Description**: The system shall allow librarians to create a new book lending for a reader.
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- Book must exist and be available (not currently lent)
- Reader must exist and be active
- Lending duration: 15 days (configurable)
- Start date: current date
- Limit date: start date + lending duration
- Lending number generated automatically (YYYY/SEQ format)

**Implementation**:
- API: `POST /api/lendings`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.create()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Domain: [Lending.java:132-146](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L132-L146)
- Configuration: [library.properties:2](Original/src/main/resources/config/library.properties#L2)
- Security: [SecurityConfig.java:164](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L164)

**Use Case**: A15 (Phase 1)

---

### FR-LEND-02: Return Book
**Name**: Set Lending as Returned
**Description**: The system shall allow readers to mark a lending as returned and optionally provide commentary.
**Actor**: Reader (self only)
**Priority**: High

**Business Rules**:
- Can only return own lendings
- Returned date: current date
- Commentary is optional (max 1024 characters)
- If overdue, fine is calculated automatically
- Fine calculation: (days overdue) × (fine value per day)
- Fine value per day: 200 cents (configurable)
- Optimistic locking enforced (version check)

**Implementation**:
- API: `PATCH /api/lendings/{year}/{seq}`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.setReturned()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Domain: [Lending.setReturned():157-170](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L157-L170)
- Domain: [Lending.getFineValueInCents():215-222](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L215-L222)
- Configuration: [library.properties:3](Original/src/main/resources/config/library.properties#L3)
- Security: [SecurityConfig.java:167](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L167)

**Use Case**: A16 (Phase 1)

---

### FR-LEND-03: Get Lending Details
**Name**: Get Lending by Number
**Description**: The system shall allow users to retrieve detailed information about a specific lending.
**Actor**: Reader (self only), Librarian
**Priority**: High

**Business Rules**:
- Lending number format: YYYY/SEQ
- Readers can only view own lendings
- Librarians can view any lending
- Includes book, reader, dates, commentary, fine (if applicable)

**Implementation**:
- API: `GET /api/lendings/{year}/{seq}`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.findByNumber()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Security: [SecurityConfig.java:163](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L163)

**Use Case**: A17 (Phase 1)

---

### FR-LEND-04: Get Overdue Lendings
**Name**: List Overdue Lendings
**Description**: The system shall allow librarians to retrieve all lendings that are overdue (past limit date and not returned).
**Actor**: Librarian
**Priority**: High

**Business Rules**:
- Overdue = current date > limit date AND returned date is null
- Includes days overdue and calculated fine
- Sorted by days overdue (most overdue first)

**Implementation**:
- API: `GET /api/lendings/overdue`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.getOverdue()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Domain: [Lending.getDaysDelayed():179-185](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/model/Lending.java#L179-L185)
- Security: [SecurityConfig.java:162,166](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L162)

**Use Case**: B23 (Phase 2)

---

### FR-LEND-05: Search Lendings
**Name**: Search Lendings with Filters
**Description**: The system shall allow librarians to search lendings with various filters.
**Actor**: Librarian
**Priority**: Medium

**Business Rules**:
- Filters: book ISBN, reader number, date range, returned status
- Paginated results

**Implementation**:
- API: `POST /api/lendings/search`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.search()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Security: [SecurityConfig.java:168](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L168)

---

## Genre Management (FR-GENRE)

### FR-GENRE-01: Create Genre
**Name**: Create Genre (Implicit)
**Description**: The system shall automatically create genres when referenced by books.
**Actor**: System
**Priority**: Low

**Business Rules**:
- Genre name must be unique
- Genre name: 1-100 characters
- Genre name cannot be blank

**Implementation**:
- Service: [GenreServiceImpl.save()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Domain: [Genre.java:23-25](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/model/Genre.java#L23-L25)

---

### FR-GENRE-02: List All Genres
**Name**: List All Genres
**Description**: The system shall provide a list of all genres in the system.
**Actor**: Librarian, Reader
**Priority**: Medium

**Implementation**:
- Service: [GenreService.findAll()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreService.java)

---

## User Management (FR-USER)

### FR-USER-01: User Authentication
**Name**: Login
**Description**: The system shall allow users to authenticate with username and password.
**Actor**: Reader, Librarian, Admin
**Priority**: High

**Business Rules**:
- Username and password are mandatory
- Password validated against stored BCrypt hash
- Upon success, JWT token generated and returned
- Token contains user roles and claims
- Token valid for configured duration

**Implementation**:
- API: `POST /api/public/login`
- Controller: [AuthApi.java](Original/src/main/java/pt/psoft/g1/psoftg1/auth/api/AuthApi.java)
- Service: [UserAuthService.authenticate()](Original/src/main/java/pt/psoft/g1/psoftg1/auth/services/)
- Security: [SecurityConfig.java:115](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L115)

---

### FR-USER-02: User Authorization
**Name**: Access Control
**Description**: The system shall enforce role-based access control on all protected endpoints.
**Actor**: System
**Priority**: High

**Business Rules**:
- Three roles: ADMIN, LIBRARIAN, READER
- Each endpoint has defined role requirements
- JWT token must contain required role
- Self-access restrictions for readers (can only access own data)

**Implementation**:
- Security: [SecurityConfig.java:110-172](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L110-L172)
- Roles: [Role.java](Original/src/main/java/pt/psoft/g1/psoftg1/usermanagement/model/Role.java)

---

## Reporting (FR-REPORT)

### FR-REPORT-01: Top 5 Authors
**Name**: Get Top 5 Authors by Lending Count
**Description**: The system shall allow readers to retrieve the top 5 most lent authors.
**Actor**: Reader
**Priority**: Low

**Business Rules**:
- Ranked by total lending count
- Includes lending count

**Implementation**:
- API: `GET /api/authors/top5`
- Controller: [AuthorController.java](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/api/AuthorController.java)
- Service: [AuthorServiceImpl.getTop5()](Original/src/main/java/pt/psoft/g1/psoftg1/authormanagement/services/AuthorServiceImpl.java)
- Security: [SecurityConfig.java:124](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L124)

**Use Case**: B6 (Phase 2)

---

### FR-REPORT-02: Top 5 Books
**Name**: Get Top 5 Books by Lending Count
**Description**: The system shall allow librarians to retrieve the top 5 most lent books.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Ranked by total lending count
- Includes lending count

**Implementation**:
- API: `GET /api/books/top5`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.getTop5()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Security: [SecurityConfig.java:135](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L135)

**Use Case**: B9 (Phase 2)

---

### FR-REPORT-03: Top 5 Genres
**Name**: Get Top 5 Genres by Book Count
**Description**: The system shall allow librarians to retrieve the top 5 genres with most books.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Ranked by total book count
- Includes book count

**Implementation**:
- API: `GET /api/genres/top5`
- Controller: [GenreController.java:28-36](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L28-L36)
- Service: [GenreServiceImpl.findTopGenreByBooks()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Security: [SecurityConfig.java:155](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L155)

**Use Case**: B10 (Phase 2)

---

### FR-REPORT-04: Top 5 Readers
**Name**: Get Top 5 Readers by Lending Count
**Description**: The system shall allow librarians to retrieve the top 5 most active readers.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Ranked by total lending count
- Includes lending count

**Implementation**:
- API: `GET /api/readers/top5`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.getTop5()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Security: [SecurityConfig.java:146](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L146)

**Use Case**: B11 (Phase 2)

---

### FR-REPORT-05: Average Lending Duration
**Name**: Get Average Lending Duration
**Description**: The system shall allow librarians to calculate average lending duration across all lendings.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Calculated from returned lendings only
- Duration = returned date - start date

**Implementation**:
- API: `GET /api/lendings/avgDuration`
- Controller: [LendingController.java](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/api/LendingController.java)
- Service: [LendingServiceImpl.getAvgDuration()](Original/src/main/java/pt/psoft/g1/psoftg1/lendingmanagement/services/LendingServiceImpl.java)
- Security: [SecurityConfig.java:165](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L165)

**Use Case**: B15 (Phase 2)

---

### FR-REPORT-06: Average Lendings Per Genre
**Name**: Get Average Lendings Per Genre by Period
**Description**: The system shall allow librarians to calculate average lendings per genre for a specified time period.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Parameterized by: period (day/month), start date, end date
- Grouped by genre

**Implementation**:
- API: `GET /api/genres/avgLendings?period={day}&start={date}&end={date}` or `POST /api/genres/avgLendingsPerGenre`
- Controller: [GenreController.java:21-26](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L21-L26)
- Service: [GenreServiceImpl.getAverageLendings()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Security: [SecurityConfig.java:156-157](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L156-L157)

**Use Case**: B14 (Phase 2)

---

### FR-REPORT-07: Lendings Per Month (Last 12 Months)
**Name**: Get Lending Count Per Genre Per Month
**Description**: The system shall allow librarians to retrieve lending counts per genre for the last 12 months.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Fixed period: last 12 months
- Grouped by: genre, month

**Implementation**:
- API: `GET /api/genres/lendingsPerMonthLastTwelveMonths`
- Controller: [GenreController.java:38-48](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L38-L48)
- Service: [GenreServiceImpl.getLendingsPerMonthLastYearByGenre()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Security: [SecurityConfig.java:158](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L158)

**Use Case**: B16 (Phase 2)

---

### FR-REPORT-08: Average Duration Per Genre Per Month
**Name**: Get Average Lending Duration Per Genre Per Month
**Description**: The system shall allow librarians to calculate average lending duration grouped by genre and month.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Parameterized by: start date, end date
- Grouped by: genre, month

**Implementation**:
- API: `GET /api/genres/lendingsAverageDurationPerMonth?startDate={date}&endDate={date}`
- Controller: [GenreController.java:50-62](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/api/GenreController.java#L50-L62)
- Service: [GenreServiceImpl.getLendingsAverageDurationPerMonth()](Original/src/main/java/pt/psoft/g1/psoftg1/genremanagement/services/GenreServiceImpl.java)
- Security: [SecurityConfig.java:159](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L159)

**Use Case**: B19 (Phase 2)

---

### FR-REPORT-09: Top 5 Readers Per Genre
**Name**: Get Top 5 Readers Per Genre
**Description**: The system shall allow librarians to retrieve the top 5 readers per genre by lending count.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Ranked by lending count per genre
- Grouped by genre

**Implementation**:
- API: `GET /api/readers/top5ByGenre`
- Controller: [ReaderController.java](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/api/ReaderController.java)
- Service: [ReaderServiceImpl.getTop5ByGenre()](Original/src/main/java/pt/psoft/g1/psoftg1/readermanagement/services/ReaderServiceImpl.java)
- Security: [SecurityConfig.java:145,149](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L145)

**Use Case**: B17 (Phase 2)

---

### FR-REPORT-10: Average Book Duration by ISBN
**Name**: Get Average Lending Duration for a Specific Book
**Description**: The system shall allow librarians to calculate the average lending duration for a specific book.
**Actor**: Librarian
**Priority**: Low

**Business Rules**:
- Calculated from all returned lendings of the book
- Duration = returned date - start date

**Implementation**:
- API: `GET /api/books/{isbn}/avgDuration`
- Controller: [BookController.java](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/api/BookController.java)
- Service: [BookServiceImpl.getAvgDuration()](Original/src/main/java/pt/psoft/g1/psoftg1/bookmanagement/services/BookServiceImpl.java)
- Security: [SecurityConfig.java:132](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L132)

---

## Authentication & Authorization (FR-AUTH)

### FR-AUTH-01: JWT Token Generation
**Name**: Generate JWT Token
**Description**: The system shall generate a JWT token upon successful authentication.
**Actor**: System
**Priority**: High

**Business Rules**:
- Token signed with RSA private key
- Token contains: username, roles, expiration
- Token returned in response body

**Implementation**:
- Security: [SecurityConfig.jwtEncoder():179-185](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L179-L185)

---

### FR-AUTH-02: JWT Token Validation
**Name**: Validate JWT Token
**Description**: The system shall validate JWT tokens on every request to protected endpoints.
**Actor**: System
**Priority**: High

**Business Rules**:
- Token decoded with RSA public key
- Token expiration checked
- Token signature verified
- User roles extracted

**Implementation**:
- Security: [SecurityConfig.jwtDecoder():187-191](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L187-L191)
- Security: [SecurityConfig.jwtAuthenticationConverter():193-203](Original/src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L193-L203)

---

## Requirements Traceability

### Phase 1 Use Cases

| Use Case | FR ID | Status | Implementation |
|----------|-------|--------|----------------|
| A3 | FR-AUT-01 | ✅ Complete | POST /api/authors |
| A4 | FR-AUT-02 | ✅ Complete | PATCH /api/authors/{id} |
| A5 | FR-AUT-03 | ✅ Complete | GET /api/authors/{id} |
| A6 | FR-AUT-04 | ✅ Complete | GET /api/authors?name={name} |
| A7 | FR-BOOK-01 | ✅ Complete | PUT /api/books/{isbn} |
| A8 | FR-BOOK-02 | ✅ Complete | PATCH /api/books/{isbn} |
| A9 | FR-BOOK-03 | ✅ Complete | GET /api/books/{isbn} |
| A10 | FR-BOOK-04 | ✅ Complete | GET /api/books?genre={genre} |
| A11 | FR-READ-01 | ✅ Complete | POST /api/readers |
| A12 | FR-READ-02 | ✅ Complete | PATCH /api/readers/{year}/{seq} |
| A13 | FR-READ-03 | ✅ Complete | GET /api/readers/{year}/{seq} |
| A14 | FR-READ-04 | ✅ Complete | GET /api/readers?name={name} |
| A15 | FR-LEND-01 | ✅ Complete | POST /api/lendings |
| A16 | FR-LEND-02, FR-READ-05 | ✅ Complete | PATCH /api/lendings, GET /api/readers/.../lendings |
| A17 | FR-LEND-03 | ✅ Complete | GET /api/lendings/{year}/{seq} |

### Phase 2 Use Cases

| Use Case | FR ID | Status | Implementation |
|----------|-------|--------|----------------|
| B3 | FR-AUT-01 | ✅ Complete | Enhanced with photo support |
| B4 | FR-AUT-05 | ✅ Complete | GET /api/authors/{id}/books |
| B5 | FR-AUT-06 | ✅ Complete | GET /api/authors/{id}/coauthors |
| B6 | FR-REPORT-01 | ✅ Complete | GET /api/authors/top5 |
| B7 | FR-BOOK-01 | ✅ Complete | Enhanced with photo support |
| B8 | FR-BOOK-05 | ✅ Complete | POST /api/books/search |
| B9 | FR-REPORT-02 | ✅ Complete | GET /api/books/top5 |
| B10 | FR-REPORT-03 | ✅ Complete | GET /api/genres/top5 |
| B11 | FR-REPORT-04 | ✅ Complete | GET /api/readers/top5 |
| B12 | FR-READ-01 | ✅ Complete | Enhanced with interests, photo |
| B13 | FR-BOOK-06 | ✅ Complete | GET /api/books/suggestions |
| B14 | FR-REPORT-06 | ✅ Complete | POST /api/genres/avgLendingsPerGenre |
| B15 | FR-REPORT-05 | ✅ Complete | GET /api/lendings/avgDuration |
| B16 | FR-REPORT-07 | ✅ Complete | GET /api/genres/lendingsPerMonthLastTwelveMonths |
| B17 | FR-REPORT-09 | ✅ Complete | GET /api/readers/top5ByGenre |
| B19 | FR-REPORT-08 | ✅ Complete | GET /api/genres/lendingsAverageDurationPerMonth |
| B23 | FR-LEND-04 | ✅ Complete | GET /api/lendings/overdue |

---

## Summary Statistics

**Total Functional Requirements**: 44

**By Module**:
- Author Management: 7
- Book Management: 7
- Reader Management: 6
- Lending Management: 5
- Genre Management: 2
- User Management: 2
- Reporting: 10
- Authentication & Authorization: 2
- Shared Infrastructure: 3

**By Priority**:
- High: 20 (45%)
- Medium: 7 (16%)
- Low: 17 (39%)

**By Actor**:
- Librarian: 20
- Reader: 15
- Anonymous: 1
- System: 8

**Implementation Status**: 100% Complete (all requirements implemented)

---

## References

- [REST API Mapping - Phase 2](../Phase2/RestMapping.md)
- [Use Case Documentation - Phase 1](../Phase1/)
- [Use Case Documentation - Phase 2](../Phase2/)
- [Domain Model](../GlobalArtifacts/DomainModel.puml)
- [Glossary](../GlobalArtifacts/Glossary.md)

---

**Document Control**
Version: 1.0
Last Updated: 2025-10-29
Status: Complete
Next Review: Upon requirement changes or feature additions
