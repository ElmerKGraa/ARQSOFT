# Quality Attributes - Library Management System

## 1. Introduction

This document details the quality attributes (non-functional requirements) for the Library Management System. Quality attributes define how well the system performs its functions and are critical drivers of architectural decisions.

## 2. Quality Attribute Scenarios

Quality attribute scenarios follow the format:
- **Source**: Origin of the stimulus
- **Stimulus**: Event or condition that affects the system
- **Artifact**: Part of the system affected
- **Environment**: System conditions when stimulus occurs
- **Response**: System reaction to the stimulus
- **Response Measure**: Quantifiable measure of the response

## 3. Security

### 3.1 Authentication

**QA-SEC-1: User Authentication**
- **Source**: User (Librarian, Reader, or Admin)
- **Stimulus**: Attempts to log in with credentials
- **Artifact**: Authentication service
- **Environment**: Normal operation
- **Response**: System validates credentials and issues JWT token
- **Response Measure**: 
  - Authentication completes in < 1 second
  - 100% of invalid credentials rejected
  - Token expires after configured time (e.g., 24 hours)

### 3.2 Authorization

**QA-SEC-2: Access Control**
- **Source**: Authenticated user
- **Stimulus**: Attempts to access protected resource
- **Artifact**: Authorization filter
- **Environment**: Normal operation
- **Response**: System checks role permissions and grants/denies access
- **Response Measure**: 
  - 100% of unauthorized access attempts denied
  - Authorization check completes in < 100ms
  - Audit log created for access attempts

### 3.3 Data Protection

**QA-SEC-3: Password Storage**
- **Source**: System administrator
- **Stimulus**: Reviews password storage
- **Artifact**: User database
- **Environment**: Any time
- **Response**: All passwords encrypted using BCrypt
- **Response Measure**: 
  - 0 plain-text passwords in database
  - BCrypt with work factor ≥ 10

**QA-SEC-4: Input Validation**
- **Source**: Malicious user
- **Stimulus**: Submits malicious input (SQL injection, XSS)
- **Artifact**: Input validation layer
- **Environment**: Normal operation
- **Response**: System rejects invalid input
- **Response Measure**: 
  - 100% of malicious inputs blocked
  - No successful injection attacks

### 3.4 Communication Security

**QA-SEC-5: Transport Security**
- **Source**: Client application
- **Stimulus**: Sends sensitive data
- **Artifact**: Network layer
- **Environment**: Production
- **Response**: All data transmitted over HTTPS/TLS
- **Response Measure**: 
  - 100% of traffic encrypted in production
  - TLS 1.2 or higher

## 4. Performance

### 4.1 Response Time

**QA-PERF-1: API Response Time**
- **Source**: API client
- **Stimulus**: Sends REST API request
- **Artifact**: REST controllers and services
- **Environment**: Normal load (< 100 concurrent users)
- **Response**: System processes request and returns response
- **Response Measure**: 
  - 95th percentile response time < 2 seconds
  - 99th percentile response time < 5 seconds
  - Average response time < 500ms

**QA-PERF-2: Search Performance**
- **Source**: User
- **Stimulus**: Searches for books/authors/readers
- **Artifact**: Search services and database
- **Environment**: Database with 10,000 books, 1,000 authors
- **Response**: Returns search results
- **Response Measure**: 
  - Search completes in < 1 second
  - Results paginated (max 100 per page)

### 4.2 Throughput

**QA-PERF-3: Request Throughput**
- **Source**: Multiple concurrent users
- **Stimulus**: Submit simultaneous requests
- **Artifact**: Web server and application
- **Environment**: Peak load
- **Response**: System handles all requests
- **Response Measure**: 
  - Support 500 requests per minute
  - Support 100 concurrent users
  - No request rejection under normal load

### 4.3 Resource Utilization

**QA-PERF-4: Memory Usage**
- **Source**: System monitor
- **Stimulus**: Monitors memory consumption
- **Artifact**: Application server
- **Environment**: Normal operation
- **Response**: Application runs within memory limits
- **Response Measure**: 
  - Maximum heap size < 2 GB
  - No memory leaks over 24 hours
  - Garbage collection < 5% of CPU time

## 5. Availability

### 5.1 Uptime

**QA-AVAIL-1: System Availability**
- **Source**: Users
- **Stimulus**: Access system during business hours
- **Artifact**: Entire system
- **Environment**: Normal operation
- **Response**: System available and responsive
- **Response Measure**: 
  - 99% uptime during business hours
  - < 4 hours planned downtime per month
  - < 1 hour unplanned downtime per month

### 5.2 Fault Tolerance

**QA-AVAIL-2: External Service Failure**
- **Source**: External API (API Ninjas)
- **Stimulus**: Service becomes unavailable
- **Artifact**: External service integration
- **Environment**: Runtime
- **Response**: System continues operating without external features
- **Response Measure**: 
  - Core features remain available
  - Graceful error messages shown
  - Automatic retry after 5 minutes

**QA-AVAIL-3: Database Connection Failure**
- **Source**: Database server
- **Stimulus**: Connection lost
- **Artifact**: Database connection pool
- **Environment**: Runtime
- **Response**: System attempts reconnection
- **Response Measure**: 
  - Automatic reconnection within 30 seconds
  - Pending requests queued
  - Users notified of temporary issue

## 6. Modifiability

### 6.1 Code Maintainability

**QA-MOD-1: Add New Entity**
- **Source**: Developer
- **Stimulus**: Needs to add new domain entity (e.g., Publisher)
- **Artifact**: Domain model and related components
- **Environment**: Development time
- **Response**: Add entity following existing patterns
- **Response Measure**: 
  - < 4 hours development time
  - < 10 files modified/created
  - No changes to other modules

**QA-MOD-2: Change Business Rule**
- **Source**: Business analyst
- **Stimulus**: Lending period changed from 14 to 21 days
- **Artifact**: Lending service
- **Environment**: Development time
- **Response**: Update business logic
- **Response Measure**: 
  - < 1 hour development time
  - < 3 files modified
  - Change isolated to lending module

### 6.2 Technology Changes

**QA-MOD-3: Database Migration**
- **Source**: System administrator
- **Stimulus**: Migrate from H2 to PostgreSQL
- **Artifact**: Data access layer
- **Environment**: Deployment
- **Response**: Update configuration and test
- **Response Measure**: 
  - < 4 hours migration time
  - 0 code changes (configuration only)
  - All tests pass after migration

**QA-MOD-4: Add Authentication Method**
- **Source**: Security team
- **Stimulus**: Add OAuth2 social login
- **Artifact**: Authentication module
- **Environment**: Development time
- **Response**: Integrate new authentication provider
- **Response Measure**: 
  - < 8 hours development time
  - No changes to business logic
  - Backward compatible

## 7. Testability

### 7.1 Unit Testing

**QA-TEST-1: Service Layer Testing**
- **Source**: Developer
- **Stimulus**: Writes unit test for service method
- **Artifact**: Service classes
- **Environment**: Development time
- **Response**: Can test in isolation with mocks
- **Response Measure**: 
  - 100% of services testable without database
  - Test execution < 10 seconds for unit tests
  - > 80% code coverage

### 7.2 Integration Testing

**QA-TEST-2: API Testing**
- **Source**: QA engineer
- **Stimulus**: Tests API endpoint
- **Artifact**: REST controllers
- **Environment**: Test environment
- **Response**: Can test with in-memory database
- **Response Measure**: 
  - All endpoints testable
  - Test suite runs in < 5 minutes
  - Automated test execution

### 7.3 Test Automation

**QA-TEST-3: Continuous Integration**
- **Source**: CI/CD pipeline
- **Stimulus**: Code commit triggers build
- **Artifact**: Entire system
- **Environment**: CI server
- **Response**: Automated build and test
- **Response Measure**: 
  - All tests automated
  - Build + test < 10 minutes
  - Test results reported automatically

## 8. Usability

### 8.1 API Usability

**QA-USE-1: API Documentation**
- **Source**: Frontend developer
- **Stimulus**: Needs to integrate with API
- **Artifact**: API documentation
- **Environment**: Development
- **Response**: Complete documentation available
- **Response Measure**: 
  - All endpoints documented
  - Request/response examples provided
  - Interactive testing available (Swagger UI)
  - < 30 minutes to understand new endpoint

**QA-USE-2: Error Messages**
- **Source**: API client
- **Stimulus**: Sends invalid request
- **Artifact**: Error handling
- **Environment**: Any time
- **Response**: Clear error message returned
- **Response Measure**: 
  - Error message describes problem
  - Includes error code
  - Suggests corrective action
  - Consistent format across all endpoints

### 8.2 API Consistency

**QA-USE-3: API Design Consistency**
- **Source**: API consumer
- **Stimulus**: Uses multiple API endpoints
- **Artifact**: All REST APIs
- **Environment**: Development/Runtime
- **Response**: Consistent patterns and conventions
- **Response Measure**: 
  - Consistent URL patterns
  - Consistent HTTP method usage
  - Consistent response formats
  - Consistent authentication across endpoints

## 9. Scalability

### 9.1 Data Scalability

**QA-SCALE-1: Database Growth**
- **Source**: System usage over time
- **Stimulus**: Database grows to 100,000 books
- **Artifact**: Database and queries
- **Environment**: Production
- **Response**: System maintains performance
- **Response Measure**: 
  - Query performance degradation < 20%
  - Database indexes effective
  - Storage growth manageable

### 9.2 User Scalability

**QA-SCALE-2: Concurrent Users**
- **Source**: Multiple users
- **Stimulus**: 200 concurrent users
- **Artifact**: Web server and connection pools
- **Environment**: Peak usage
- **Response**: System serves all users
- **Response Measure**: 
  - No request rejections
  - Response time increase < 50%
  - Resource utilization < 80%

### 9.3 Horizontal Scalability

**QA-SCALE-3: Load Distribution**
- **Source**: Load balancer
- **Stimulus**: Distributes requests across instances
- **Artifact**: Application instances
- **Environment**: Production with multiple instances
- **Response**: All instances handle requests
- **Response Measure**: 
  - Stateless application design
  - No session affinity required
  - Linear scalability up to 5 instances

## 10. Reliability

### 10.1 Data Integrity

**QA-REL-1: Concurrent Lending**
- **Source**: Two users
- **Stimulus**: Try to borrow same book simultaneously
- **Artifact**: Lending service and database
- **Environment**: Runtime
- **Response**: Only one lending succeeds
- **Response Measure**: 
  - 0 double bookings
  - Optimistic locking prevents conflicts
  - Second user receives clear error

**QA-REL-2: Transaction Management**
- **Source**: Service operation
- **Stimulus**: Error occurs during multi-step operation
- **Artifact**: Transaction management
- **Environment**: Runtime
- **Response**: All changes rolled back
- **Response Measure**: 
  - 100% ACID compliance
  - No partial updates
  - Database consistency maintained

### 10.2 Error Recovery

**QA-REL-3: Error Handling**
- **Source**: System component
- **Stimulus**: Unexpected error occurs
- **Artifact**: Exception handling
- **Environment**: Runtime
- **Response**: Error logged and user notified
- **Response Measure**: 
  - All errors logged with stack trace
  - User receives appropriate error message
  - System remains operational
  - No data corruption

## 11. Interoperability

### 11.1 API Standards

**QA-INTER-1: REST Compliance**
- **Source**: External system
- **Stimulus**: Integrates with library API
- **Artifact**: REST APIs
- **Environment**: Integration
- **Response**: Standard REST principles followed
- **Response Measure**: 
  - HTTP methods used correctly (GET, POST, PUT, DELETE)
  - Standard HTTP status codes
  - JSON format for data exchange
  - Content negotiation supported

**QA-INTER-2: Data Format Standards**
- **Source**: API consumer
- **Stimulus**: Parses API responses
- **Artifact**: Data serialization
- **Environment**: Integration
- **Response**: Standard data formats used
- **Response Measure**: 
  - ISO 8601 for dates
  - UTF-8 encoding
  - Standard JSON structure
  - Schema documented (OpenAPI)

## 12. Deployability

### 12.1 Deployment Process

**QA-DEPLOY-1: Application Deployment**
- **Source**: DevOps engineer
- **Stimulus**: Deploys new version
- **Artifact**: Application package
- **Environment**: Target server
- **Response**: Application starts successfully
- **Response Measure**: 
  - Single executable JAR
  - < 2 minutes startup time
  - Configuration externalized
  - Health check endpoint available

**QA-DEPLOY-2: Configuration Management**
- **Source**: System administrator
- **Stimulus**: Changes configuration
- **Artifact**: Configuration files
- **Environment**: Any environment
- **Response**: Configuration applied without code changes
- **Response Measure**: 
  - All environment-specific settings externalized
  - Profile-based configuration
  - Environment variables supported
  - No hard-coded values

## 13. Legal Compliance

### 13.1 Data Protection

**QA-LEGAL-1: GDPR Compliance**
- **Source**: Data protection officer
- **Stimulus**: Audits data handling
- **Artifact**: User data management
- **Environment**: Any time
- **Response**: System demonstrates compliance
- **Response Measure**: 
  - Explicit consent recorded
  - Data deletion capability
  - Data export capability
  - Audit trail maintained
  - Privacy by design

**QA-LEGAL-2: Data Retention**
- **Source**: Legal requirement
- **Stimulus**: Data retention policy
- **Artifact**: Data management
- **Environment**: Operational
- **Response**: Old data archived/deleted
- **Response Measure**: 
  - Configurable retention periods
  - Automatic cleanup process
  - Deleted data irrecoverable

## 14. Quality Attribute Utility Tree

```
Quality Attributes
├── Security (Critical)
│   ├── Authentication (H, H)
│   ├── Authorization (H, H)
│   └── Data Protection (H, M)
├── Performance (High)
│   ├── Response Time (H, M)
│   └── Throughput (M, M)
├── Modifiability (High)
│   ├── Code Maintainability (H, L)
│   └── Technology Changes (M, M)
├── Testability (High)
│   ├── Unit Testing (H, L)
│   └── Integration Testing (H, M)
├── Usability (Medium)
│   ├── API Usability (H, L)
│   └── Error Messages (M, L)
├── Availability (Medium)
│   ├── Uptime (M, M)
│   └── Fault Tolerance (M, M)
├── Reliability (Medium)
│   ├── Data Integrity (H, M)
│   └── Error Recovery (M, L)
└── Legal Compliance (Critical)
    └── GDPR (H, M)

Legend: (Business Value, Technical Risk)
H = High, M = Medium, L = Low
```

## 15. Trade-offs and Priorities

### 15.1 Primary Quality Attributes
1. **Security** - Non-negotiable, handles personal data
2. **Legal Compliance** - GDPR requirements mandatory
3. **Modifiability** - Business rules change frequently
4. **Testability** - Ensures quality and rapid changes

### 15.2 Secondary Quality Attributes
1. **Performance** - Adequate for user base
2. **Usability** - Good documentation essential
3. **Availability** - Important but not mission-critical
4. **Reliability** - Standard enterprise level

### 15.3 Trade-off Decisions
- **Security vs Performance**: Security prioritized, caching limited
- **Modifiability vs Performance**: Layered architecture chosen despite slight overhead
- **Testability vs Complexity**: Dependency injection adds complexity but improves testability
- **Usability vs Development Time**: Swagger documentation automated to balance both

## References
- [Architecture Overview](./01-Architecture-Overview.md)
- [ASR](./04-ASR.md)
- [Architectural Tactics](./07-Architectural-Tactics.md)
