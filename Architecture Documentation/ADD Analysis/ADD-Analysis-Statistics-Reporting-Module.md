# ADD Analysis: Unified Statistics & Reporting Module

**Project:** Library Management System (psoft-g1)
**Purpose:** Architectural variation analysis for centralizing statistics/reporting functionality
**Date:** November 2025
**Version:** 1.0

---

## Executive Summary

**Current Problem:** Statistics views are **scattered across modules** with **duplicated patterns**:
- AuthorLendingView (author + count)
- BookCountView (book + count)
- GenreLendingsView (genre + average)
- GenreLendingsCountPerMonthView (genre + count + month)
- GenreLendingsAvgPerMonthView (genre + average + month)

**Proposed Solution:** Create a **unified Statistics/Reporting module** with **parameterized queries** supporting:
- Operation type (Top N, Average, Stats per period)
- Entity type (Author, Book, Genre, Reader)
- Aggregation type (Count, Average, Sum)
- Time grouping (Day, Month, Quarter, Year)
- Limit (Top 5, Top 10, Top 20, etc.)

---

## 1. Current State Analysis

### 1.1 Existing Statistical Views

| Module | View Class | Purpose | Pattern |
|--------|-----------|---------|---------|
| **AuthorManagement** | AuthorLendingView | Author name + lending count | Top N ranking |
| **BookManagement** | BookCountView | Book + lending count | Top N ranking |
| **BookManagement** | BookAverageLendingDurationView | Book + average duration | Average calculation |
| **GenreManagement** | GenreLendingsView | Genre + average lendings | Average calculation |
| **GenreManagement** | GenreLendingsCountPerMonthView | Genre + month + count | Time-series count |
| **GenreManagement** | GenreLendingsAvgPerMonthView | Genre + month + average duration | Time-series average |
| **GenreManagement** | GenreBookCountView | Genre + book count | Top N ranking |
| **ReaderManagement** | (Implied) ReaderLendingView | Reader + lending count | Top N ranking |

### 1.2 Repeated Patterns Identified

**Pattern 1: Top N Ranking**
```java
// Repeated in Author, Book, Genre, Reader modules
public class XxxLendingView {
    private String name;      // Entity identifier
    private Long count;       // Aggregation result
}

// Endpoint pattern
GET /api/xxx/top5
```

**Pattern 2: Average Calculation**
```java
// Repeated in Book, Genre, Lending modules
public class XxxAverageView {
    private String identifier;
    private Double average;
}

// Endpoint pattern
GET /api/xxx/average
```

**Pattern 3: Time-Series Statistics**
```java
// Repeated in Genre, potentially needed in others
public class XxxPerMonthView {
    private String identifier;
    private Integer year;
    private Integer month;
    private Long/Double value;
}

// Endpoint pattern
GET /api/xxx/stats?period=month&start=2024-01&end=2024-12
```

### 1.3 Code Duplication Evidence

**Example: Top 5 Queries**

```java
// AuthorManagement - SpringDataAuthorRepository
@Query("SELECT new ...AuthorLendingView(a.name.name, COUNT(l.pk)) " +
       "FROM Book b JOIN b.authors a JOIN Lending l ON l.book.pk = b.pk " +
       "GROUP BY a.name ORDER BY COUNT(l) DESC")
Page<AuthorLendingView> findTopAuthorByLendings(Pageable pageable);

// BookManagement - SpringDataBookRepository (implied)
@Query("SELECT new ...BookCountView(b.isbn, b.title, COUNT(l.pk)) " +
       "FROM Book b JOIN Lending l ON l.book.pk = b.pk " +
       "GROUP BY b.isbn ORDER BY COUNT(l) DESC")
Page<BookCountView> findTop5Books(Pageable pageable);

// GenreManagement - SpringDataGenreRepository (implied)
@Query("SELECT new ...GenreBookCountView(g.genre, COUNT(b.pk)) " +
       "FROM Genre g JOIN Book b ON b.genre = g " +
       "GROUP BY g.genre ORDER BY COUNT(b) DESC")
Page<GenreBookCountView> findTop5GenresByBookCount(Pageable pageable);
```

**Pattern:** Same structure, different entities
- Same aggregation: `COUNT()`
- Same grouping: `GROUP BY entity`
- Same sorting: `ORDER BY COUNT() DESC`
- Same limit: Top N via `Pageable`

---

## 2. Architectural Drivers (ASRs)

### ASR-1: Unified Statistics API
- **Stimulus:** Business analyst wants Top 10 authors by lending count
- **Current Response:** Developer creates new view, query, endpoint (30-60 minutes)
- **Desired Response:** Configuration-based query, reuses existing infrastructure (2 minutes)
- **Quality Attributes:** Modifiability, Reusability, Development Speed

### ASR-2: Flexible Reporting Requirements
- **Stimulus:** Stakeholder requests "Top 20 genres by book count for Q4 2024"
- **Current Response:** Not supported, requires code changes
- **Desired Response:** Parameterized API call
- **Quality Attributes:** Flexibility, Configurability

### ASR-3: Performance Optimization
- **Stimulus:** Statistical queries slow down main application
- **Current Response:** Each module executes expensive aggregations independently
- **Desired Response:** Dedicated statistics module with optimized queries, caching, possibly separate read model (CQRS)
- **Quality Attributes:** Performance, Scalability

### ASR-4: Cross-Module Analytics
- **Stimulus:** Request for "Top 5 authors with most readers in Science Fiction genre"
- **Current Response:** Requires cross-module joins, complex implementation
- **Desired Response:** Statistics module orchestrates cross-module queries
- **Quality Attributes:** Composability, Maintainability

---

## 3. Proposed Architecture

### 3.1 Unified Statistics Module Structure

```
pt.psoft.g1.psoftg1.statistics/
├── api/
│   ├── StatisticsController.java
│   ├── StatisticsRequest.java         # Parameterized query request
│   └── StatisticsResponse.java        # Generic response wrapper
│
├── model/
│   ├── StatisticType.java             # Enum: TOP_N, AVERAGE, SUM, COUNT
│   ├── EntityType.java                # Enum: AUTHOR, BOOK, GENRE, READER, LENDING
│   ├── AggregationType.java           # Enum: COUNT, AVG, SUM, MIN, MAX
│   ├── TimePeriod.java                # Enum: DAY, MONTH, QUARTER, YEAR
│   └── StatisticMetric.java           # Generic metric result
│
├── services/
│   ├── StatisticsService.java
│   ├── StatisticsServiceImpl.java
│   ├── QueryBuilder.java              # Dynamic query generation
│   └── strategies/
│       ├── TopNStatisticsStrategy.java
│       ├── AverageStatisticsStrategy.java
│       ├── TimeSeriesStatisticsStrategy.java
│       └── CrossModuleStatisticsStrategy.java
│
└── repositories/
    ├── StatisticsRepository.java
    └── impl/
        └── SpringDataStatisticsRepository.java  # Dynamic queries
```

### 3.2 API Design

#### **Endpoint 1: Generic Statistics Query**

```
POST /api/statistics/query
Content-Type: application/json
{
  "operation": "TOP_N",
  "entity": "AUTHOR",
  "metric": "LENDING_COUNT",
  "aggregation": "COUNT",
  "limit": 5,
  "filters": {
    "startDate": "2024-01-01",
    "endDate": "2024-12-31"
  },
  "groupBy": null
}

Response:
{
  "operation": "TOP_N",
  "entity": "AUTHOR",
  "results": [
    {
      "identifier": "Joshua Bloch",
      "identifierType": "NAME",
      "value": 42,
      "metadata": {
        "authorNumber": 1
      }
    },
    {
      "identifier": "Robert Martin",
      "identifierType": "NAME",
      "value": 38,
      "metadata": {
        "authorNumber": 2
      }
    }
  ],
  "totalResults": 2,
  "executionTime": 123
}
```

#### **Endpoint 2: Time-Series Statistics**

```
POST /api/statistics/query
{
  "operation": "TIME_SERIES",
  "entity": "GENRE",
  "metric": "LENDING_COUNT",
  "aggregation": "COUNT",
  "timePeriod": "MONTH",
  "filters": {
    "startDate": "2024-01-01",
    "endDate": "2024-12-31",
    "genreName": "Science Fiction"
  }
}

Response:
{
  "operation": "TIME_SERIES",
  "entity": "GENRE",
  "timePeriod": "MONTH",
  "results": [
    {
      "identifier": "Science Fiction",
      "period": "2024-01",
      "value": 15
    },
    {
      "identifier": "Science Fiction",
      "period": "2024-02",
      "value": 18
    },
    ...
  ]
}
```

#### **Endpoint 3: Cross-Entity Statistics**

```
POST /api/statistics/query
{
  "operation": "TOP_N",
  "entity": "AUTHOR",
  "metric": "LENDING_COUNT",
  "aggregation": "COUNT",
  "limit": 5,
  "filters": {
    "genre": "Programming",
    "hasPhoto": true
  }
}

Response: Top 5 authors in Programming genre with photos
```

#### **Endpoint 4: Simplified Convenience Endpoints**

```
# Backward-compatible shortcuts
GET /api/statistics/top/authors?limit=5
GET /api/statistics/top/books?limit=10
GET /api/statistics/top/genres?limit=5&by=book_count

GET /api/statistics/average/lending-duration?entity=BOOK
GET /api/statistics/average/lending-duration?entity=GENRE

GET /api/statistics/time-series/lendings?period=month&start=2024-01&end=2024-12
```

---

## 4. Implementation Design

### 4.1 Core Classes

#### **StatisticsRequest.java**
```java
@Data
@Builder
public class StatisticsRequest {
    @NotNull
    private StatisticOperation operation;  // TOP_N, AVERAGE, TIME_SERIES

    @NotNull
    private EntityType entity;             // AUTHOR, BOOK, GENRE, READER

    @NotNull
    private MetricType metric;             // LENDING_COUNT, BOOK_COUNT, AVG_DURATION

    @NotNull
    private AggregationType aggregation;   // COUNT, AVG, SUM

    private Integer limit;                 // For TOP_N operations

    private TimePeriod timePeriod;         // For TIME_SERIES (DAY, MONTH, QUARTER, YEAR)

    @Builder.Default
    private Map<String, Object> filters = new HashMap<>();

    private List<String> groupBy;          // Additional grouping fields
}
```

#### **StatisticsResponse.java**
```java
@Data
@Builder
public class StatisticsResponse<T> {
    private StatisticOperation operation;
    private EntityType entity;
    private TimePeriod timePeriod;

    private List<StatisticResult> results;

    private int totalResults;
    private long executionTimeMs;

    private Map<String, Object> metadata;
}

@Data
@Builder
public class StatisticResult {
    private String identifier;        // Entity name/ID
    private String identifierType;    // NAME, ISBN, READER_NUMBER, etc.

    private Object value;             // Numeric result (Long, Double)

    private LocalDate period;         // For time-series
    private Integer year;
    private Integer month;

    private Map<String, Object> metadata;  // Additional info
}
```

### 4.2 Strategy Pattern for Operations

```java
public interface StatisticsStrategy {
    List<StatisticResult> execute(StatisticsRequest request);
}

@Component
public class TopNStatisticsStrategy implements StatisticsStrategy {
    @Override
    public List<StatisticResult> execute(StatisticsRequest request) {
        // Build dynamic query for TOP N
        String query = buildTopNQuery(request);
        // Execute query
        // Map results to StatisticResult
        return results;
    }

    private String buildTopNQuery(StatisticsRequest request) {
        // Dynamic JPQL generation based on entity and metric
        switch (request.getEntity()) {
            case AUTHOR:
                return buildAuthorTopNQuery(request);
            case BOOK:
                return buildBookTopNQuery(request);
            // ...
        }
    }
}

@Component
public class AverageStatisticsStrategy implements StatisticsStrategy {
    // Similar implementation for averages
}

@Component
public class TimeSeriesStatisticsStrategy implements StatisticsStrategy {
    // Implementation for time-series data
}
```

### 4.3 Query Builder (Dynamic JPQL/Criteria API)

```java
@Component
public class StatisticsQueryBuilder {

    public String buildQuery(StatisticsRequest request) {
        StringBuilder jpql = new StringBuilder();

        // SELECT clause
        jpql.append("SELECT new ").append(getResultClass(request));
        jpql.append("(");
        jpql.append(getSelectFields(request));
        jpql.append(") ");

        // FROM clause
        jpql.append("FROM ").append(getFromClause(request));

        // JOIN clauses
        jpql.append(getJoinClauses(request));

        // WHERE clause
        if (!request.getFilters().isEmpty()) {
            jpql.append(" WHERE ").append(buildWhereClause(request));
        }

        // GROUP BY clause
        jpql.append(getGroupByClause(request));

        // ORDER BY clause
        jpql.append(getOrderByClause(request));

        return jpql.toString();
    }

    private String getSelectFields(StatisticsRequest request) {
        switch (request.getOperation()) {
            case TOP_N:
                return getEntityIdentifier(request) + ", " +
                       getAggregationFunction(request);
            case AVERAGE:
                return getEntityIdentifier(request) + ", AVG(" +
                       getMetricField(request) + ")";
            case TIME_SERIES:
                return getEntityIdentifier(request) + ", " +
                       getTimePeriodFields(request) + ", " +
                       getAggregationFunction(request);
            default:
                throw new IllegalArgumentException("Unsupported operation");
        }
    }

    private String getAggregationFunction(StatisticsRequest request) {
        switch (request.getAggregation()) {
            case COUNT:
                return "COUNT(" + getCountTarget(request) + ")";
            case AVG:
                return "AVG(" + getMetricField(request) + ")";
            case SUM:
                return "SUM(" + getMetricField(request) + ")";
            default:
                throw new IllegalArgumentException("Unsupported aggregation");
        }
    }

    // Example for AUTHOR TOP_N by LENDING_COUNT
    // Generates:
    // SELECT new StatisticResult(a.name.name, COUNT(l.pk))
    // FROM Book b
    // JOIN b.authors a
    // JOIN Lending l ON l.book.pk = b.pk
    // GROUP BY a.name
    // ORDER BY COUNT(l) DESC
}
```

### 4.4 Service Layer

```java
@Service
@RequiredArgsConstructor
public class StatisticsServiceImpl implements StatisticsService {
    private final Map<StatisticOperation, StatisticsStrategy> strategies;
    private final StatisticsRepository statisticsRepository;
    private final CacheManager cacheManager;

    @Override
    public StatisticsResponse<?> executeQuery(StatisticsRequest request) {
        // Validate request
        validateRequest(request);

        // Check cache
        Optional<StatisticsResponse<?>> cached = getCachedResult(request);
        if (cached.isPresent()) {
            return cached.get();
        }

        // Select strategy
        StatisticsStrategy strategy = strategies.get(request.getOperation());
        if (strategy == null) {
            throw new IllegalArgumentException(
                "No strategy for operation: " + request.getOperation()
            );
        }

        // Execute query
        long startTime = System.currentTimeMillis();
        List<StatisticResult> results = strategy.execute(request);
        long executionTime = System.currentTimeMillis() - startTime;

        // Build response
        StatisticsResponse<?> response = StatisticsResponse.builder()
            .operation(request.getOperation())
            .entity(request.getEntity())
            .results(results)
            .totalResults(results.size())
            .executionTimeMs(executionTime)
            .build();

        // Cache result
        cacheResult(request, response);

        return response;
    }
}
```

---

## 5. Benefits Analysis

### 5.1 Advantages

| Benefit | Impact | Evidence |
|---------|--------|----------|
| **Reduced Code Duplication** | High | Eliminate 8+ view classes, consolidate into 1 generic result |
| **Faster Feature Development** | High | New statistic = configuration, not coding (hours → minutes) |
| **Consistent API** | Medium | All statistics follow same request/response pattern |
| **Cross-Module Analytics** | Medium | Easy to combine data from multiple entities |
| **Performance Optimization** | High | Centralized caching, query optimization, potential CQRS |
| **Easier Testing** | Medium | Test one module vs. statistics scattered across 6 modules |
| **API Evolution** | Low | Can version statistics API independently |

### 5.2 Disadvantages

| Challenge | Impact | Mitigation |
|-----------|--------|------------|
| **Complexity** | High | Introduce QueryBuilder, Strategy pattern, dynamic queries |
| **Learning Curve** | Medium | Team must understand parameterized API, not simple endpoints |
| **Type Safety Loss** | Medium | Generic results vs. typed views (AuthorLendingView) |
| **SQL Injection Risk** | Low | Use Criteria API or parameterized queries, not string concatenation |
| **Backward Compatibility** | High | Existing endpoints must continue working |
| **Query Optimization** | Medium | Dynamic queries harder to optimize than static JPQL |

---

## 6. Migration Strategy

### Phase 1: Create Statistics Module (No Breaking Changes)
```
1. Create new statistics/ package
2. Implement generic StatisticsService
3. Add POST /api/statistics/query endpoint
4. Keep existing endpoints unchanged
5. No changes to other modules
```

### Phase 2: Migrate Existing Endpoints
```
1. Update GenreController.getTop5Genres()
   Old: Return GenreBookCountView
   New: Delegate to StatisticsService, map to GenreBookCountView
   Result: Same API, new implementation

2. Update AuthorController.getTop5()
   Old: Custom JPQL query
   New: StatisticsService.executeQuery(
          operation=TOP_N, entity=AUTHOR, limit=5
        )
```

### Phase 3: Deprecate Redundant Views
```
1. Mark old view classes as @Deprecated
2. Add @Deprecated to old endpoints
3. Add API documentation pointing to new statistics API
4. Keep for 2-3 releases, then remove
```

### Phase 4: Advanced Features
```
1. Add CQRS read model for statistics
2. Implement event-driven statistics updates
3. Add GraphQL endpoint for flexible queries
4. Add statistics dashboard (frontend)
```

---

## 7. Implementation Example

### Before (Current):

```java
// GenreController
@GetMapping("/top5")
public ListResponse<GenreBookCountView> getTop5() {
    return new ListResponse<>(genreService.findTop5GenresByBookCount());
}

// GenreServiceImpl
public List<GenreBookCountView> findTop5GenresByBookCount() {
    Pageable pageable = PageRequest.of(0, 5);
    return genreRepository.findTop5GenresByBookCount(pageable).getContent();
}

// SpringDataGenreRepository
@Query("SELECT new ...GenreBookCountView(g.genre, COUNT(b.pk)) " +
       "FROM Genre g JOIN Book b ON b.genre = g " +
       "GROUP BY g.genre ORDER BY COUNT(b) DESC")
Page<GenreBookCountView> findTop5GenresByBookCount(Pageable pageable);

// GenreBookCountView
public class GenreBookCountView {
    private String genre;
    private Long bookCount;
}
```

### After (Proposed):

```java
// GenreController (backward compatible)
@GetMapping("/top5")
public ListResponse<GenreBookCountView> getTop5() {
    // Delegate to statistics module
    StatisticsRequest request = StatisticsRequest.builder()
        .operation(StatisticOperation.TOP_N)
        .entity(EntityType.GENRE)
        .metric(MetricType.BOOK_COUNT)
        .aggregation(AggregationType.COUNT)
        .limit(5)
        .build();

    StatisticsResponse<?> response = statisticsService.executeQuery(request);

    // Map to legacy view
    List<GenreBookCountView> views = response.getResults().stream()
        .map(r -> new GenreBookCountView(
            r.getIdentifier(),
            (Long) r.getValue()
        ))
        .collect(Collectors.toList());

    return new ListResponse<>(views);
}

// OR use new statistics endpoint directly
POST /api/statistics/query
{
  "operation": "TOP_N",
  "entity": "GENRE",
  "metric": "BOOK_COUNT",
  "aggregation": "COUNT",
  "limit": 5
}
```

---

## 8. Comparison with Current Architecture

### Current (Scattered Statistics)

**Pros:**
- ✅ Simple, straightforward implementation
- ✅ Type-safe (each view is a specific class)
- ✅ Easy to understand (one endpoint = one query)
- ✅ No additional complexity

**Cons:**
- ❌ Code duplication (8+ similar view classes)
- ❌ Rigid (new statistic = code changes)
- ❌ Hard to maintain (changes in 6 modules)
- ❌ No cross-module analytics
- ❌ Performance (no centralized caching)

### Proposed (Unified Statistics Module)

**Pros:**
- ✅ No code duplication (1 generic result class)
- ✅ Flexible (new statistics via configuration)
- ✅ Easy to maintain (changes in 1 module)
- ✅ Supports cross-module analytics
- ✅ Centralized caching and optimization
- ✅ API evolution independent of domain modules

**Cons:**
- ❌ Higher initial complexity
- ❌ Dynamic queries harder to optimize
- ❌ Loss of type safety (generic results)
- ❌ Learning curve for team
- ❌ More abstraction layers

---

## 9. Recommendation

### **Yes, this is a valuable ADD variation point** ✅

**Reasons:**

1. **Clear Cross-Cutting Concern:** Statistics is orthogonal to domain logic (Author, Book, Genre)

2. **High ROI:** Eliminates duplication in 8+ classes, simplifies future development

3. **Supports Growth:** Easy to add new statistics without modifying domain modules

4. **Performance Path:** Enables CQRS, caching, read models in future

5. **Industry Best Practice:** Aligns with patterns like:
   - **Query Object Pattern** (Martin Fowler)
   - **Specification Pattern** (Eric Evans)
   - **CQRS** (Greg Young) - for advanced implementation

### **When to Implement:**

**Now (High Priority):**
- If you have 10+ statistical endpoints
- If stakeholders frequently request new statistics
- If performance is an issue

**Later (Lower Priority):**
- If current implementation is working
- If team is small and prefers simplicity
- If project is in maintenance mode

### **Hybrid Approach (Recommended):**

1. **Keep existing endpoints** for now (no breaking changes)
2. **Create StatisticsService** as new module
3. **Gradually migrate** high-value queries (Top 5, Averages)
4. **Add new statistics** using unified API only
5. **Evaluate** after 3-6 months, decide on full migration

---

## 10. Related Patterns and Alternatives

### Alternative 1: GraphQL
```
query {
  statistics(
    operation: TOP_N,
    entity: AUTHOR,
    limit: 5,
    metric: LENDING_COUNT
  ) {
    identifier
    value
  }
}
```
**Pros:** Even more flexible, client-driven queries
**Cons:** Requires GraphQL infrastructure

### Alternative 2: Reporting Framework (JasperReports, BIRT)
**Pros:** Visual reports, PDF/Excel export
**Cons:** Heavier infrastructure, more complex

### Alternative 3: Analytics Database (Elasticsearch, ClickHouse)
**Pros:** Optimized for analytics, blazing fast
**Cons:** Separate infrastructure, data sync complexity

### Alternative 4: Business Intelligence Tool (Tableau, Power BI)
**Pros:** Rich visualizations, no custom coding
**Cons:** External dependency, licensing costs

---

## Conclusion

**Your observation is spot-on!** The proliferation of specialized views (AuthorLendingView, GenreBookCountView, etc.) is a **code smell** indicating:
- **Repeated patterns** that could be abstracted
- **Cross-cutting concern** (statistics/reporting)
- **High potential for refactoring**

Creating a **unified Statistics/Reporting module** is:
- ✅ **Architecturally sound** (follows DDD, SoC principles)
- ✅ **Practical** (solves real problems)
- ✅ **Scalable** (supports growth)
- ⚠️ **Complex** (requires careful design)

**Recommendation:** Add this to your ADD analysis as **"Architectural Variation #5: Unified Statistics Module"** alongside the persistence, ISBN lookup, and ID generation variations.

Would you like me to create the complete implementation design with code examples for Phase 1?

---

**END OF STATISTICS MODULE ADD ANALYSIS**
