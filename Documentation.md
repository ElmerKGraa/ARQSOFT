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