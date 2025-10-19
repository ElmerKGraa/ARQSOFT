# Architecturally Significant Requirements (ASR)

## 1. Introduction

This document identifies and documents the Architecturally Significant Requirements (ASR) for the Library Management System. ASRs are requirements that have a measurable effect on the system's architecture and must be considered during architectural design decisions.

## 2. ASR Identification Criteria

A requirement is considered architecturally significant if it:
- Affects the system structure or key components
- Impacts quality attributes (performance, security, scalability, etc.)
- Involves technical or business risk
- Requires significant infrastructure or technology decisions
- Influences multiple system components

## 3. Functional ASRs

### ASR-F1: Multi-Role User Management
**Category**: Functional Requirement  
**Priority**: High  
**Architectural Impact**: High

**Description**: The system must support different user roles (Librarian, Reader, Administrator) with role-specific capabilities and access controls.

**Rationale**: Different user types require different levels of access and functionality, impacting authentication, authorization, and user interface design.

**Architectural Decisions**:
- Role-Based Access Control (RBAC) implementation
- JWT-based authentication with role claims
- Separate user management module
- Spring Security integration

**Affected Components**:
- Authentication & Authorization
- User Management
- All REST API controllers
- Security configuration

---

### ASR-F2: Book Lending Workflow
**Category**: Functional Requirement  
**Priority**: High  
**Architectural Impact**: High

**Description**: The system must manage the complete book lending lifecycle including borrowing, return tracking, and fine calculation.

**Rationale**: Core business functionality that requires coordination between multiple domain entities and business rules.

**Architectural Decisions**:
- Lending aggregate as separate bounded context
- State machine for lending status
- Fine calculation service
- Date/time handling for due dates

**Affected Components**:
- Lending Management
- Book Management
- Reader Management
- Fine calculation logic

---

### ASR-F3: Multi-Entity Search Capabilities
**Category**: Functional Requirement  
**Priority**: Medium  
**Architectural Impact**: Medium

**Description**: The system must provide search functionality across Books, Authors, and Readers with various filter criteria.

**Rationale**: Users need to quickly find information; poor search performance impacts user experience.

**Architectural Decisions**:
- Repository pattern with query methods
- Spring Data JPA query derivation
- Database indexing strategy
- Pagination support

**Affected Components**:
- All repository interfaces
- Search service methods
- Database schema

---

### ASR-F4: Photo Management
**Category**: Functional Requirement  
**Priority**: Medium  
**Architectural Impact**: Medium

**Description**: The system must support photo uploads for Authors, Books, and Readers.

**Rationale**: File upload and storage requires infrastructure decisions and security considerations.

**Architectural Decisions**:
- File system storage (local directory)
- Photo URI reference in entities
- File upload validation
- Storage path configuration

**Affected Components**:
- File storage service
- Author, Book, and Reader entities
- API controllers for uploads

---

### ASR-F5: External API Integration
**Category**: Functional Requirement  
**Priority**: Low  
**Architectural Impact**: Medium

**Description**: The system integrates with external APIs (API Ninjas) for additional data enrichment.

**Rationale**: External dependencies require resilience patterns and error handling.

**Architectural Decisions**:
- Separate external services component
- Spring WebClient for non-blocking calls
- Timeout and retry configuration
- Graceful degradation

**Affected Components**:
- External services module
- Author management (uses external data)
- Configuration management

## 4. Quality Attribute ASRs

### ASR-Q1: Security and Authentication
**Quality Attribute**: Security  
**Priority**: Critical  
**Architectural Impact**: Very High

**Scenario**:
- **Source**: Unauthorized user
- **Stimulus**: Attempts to access protected resources
- **Response**: System denies access and logs attempt
- **Measure**: 100% of unauthorized access attempts blocked

**Rationale**: Library system handles personal data (GDPR) and must prevent unauthorized access.

**Architectural Decisions**:
- JWT-based authentication (OAuth2 Resource Server)
- Role-based authorization on all endpoints
- Password encryption (BCrypt)
- HTTPS for all communications
- Input sanitization (OWASP sanitizer)
- CORS configuration

**Affected Components**:
- Spring Security configuration
- All REST controllers
- User management
- Password handling

**Verification**:
- Security testing
- Penetration testing
- Authentication/authorization tests

---

### ASR-Q2: Modifiability and Maintainability
**Quality Attribute**: Modifiability  
**Priority**: High  
**Architectural Impact**: Very High

**Scenario**:
- **Source**: Developer
- **Stimulus**: Needs to add new lending rule
- **Response**: Change isolated to lending service
- **Measure**: < 2 hours development time, < 5 files modified

**Rationale**: Business rules change frequently; system must accommodate changes easily.

**Architectural Decisions**:
- Layered architecture with clear separation
- Domain-Driven Design bounded contexts
- Repository pattern for data access abstraction
- DTO pattern for API decoupling
- Dependency injection

**Affected Components**:
- Overall system structure
- All modules follow same pattern
- Interface-based design

**Verification**:
- Code review
- Change impact analysis
- Metrics: coupling, cohesion

---

### ASR-Q3: Testability
**Quality Attribute**: Testability  
**Priority**: High  
**Architectural Impact**: High

**Scenario**:
- **Source**: Developer/QA
- **Stimulus**: Needs to test lending business logic
- **Response**: Can test in isolation without database
- **Measure**: > 80% code coverage, < 5 minutes test execution

**Rationale**: Automated testing ensures quality and enables continuous integration.

**Architectural Decisions**:
- Dependency injection (Spring IoC)
- Interface-based repository pattern
- Mock-friendly design
- H2 in-memory database for integration tests
- Service layer separation

**Affected Components**:
- All service classes
- Repository interfaces
- Test infrastructure

**Verification**:
- Unit test coverage reports
- Integration test execution
- Test execution time

---

### ASR-Q4: API Usability
**Quality Attribute**: Usability  
**Priority**: High  
**Architectural Impact**: Medium

**Scenario**:
- **Source**: API Consumer (Frontend developer)
- **Stimulus**: Needs to understand API endpoints
- **Response**: Complete documentation available
- **Measure**: < 30 minutes to understand and use new endpoint

**Rationale**: Multiple clients may consume the API; clear documentation improves adoption.

**Architectural Decisions**:
- RESTful API design principles
- OpenAPI/Swagger documentation
- Consistent error responses
- Request/response validation
- Clear naming conventions

**Affected Components**:
- All REST controllers
- API documentation configuration
- DTO design

**Verification**:
- API documentation review
- Developer feedback
- API testing

---

### ASR-Q5: Performance
**Quality Attribute**: Performance  
**Priority**: Medium  
**Architectural Impact**: Medium

**Scenario**:
- **Source**: User
- **Stimulus**: Searches for books by title
- **Response**: Results returned
- **Measure**: < 2 seconds response time for 95% of requests

**Rationale**: Users expect responsive system; poor performance impacts user satisfaction.

**Architectural Decisions**:
- Database indexing on frequently queried fields
- Pagination for large result sets
- Connection pooling (HikariCP)
- Lazy loading for relationships
- Efficient query design

**Affected Components**:
- Repository queries
- Database schema
- Service methods
- API pagination

**Verification**:
- Performance testing
- Database query analysis
- Response time monitoring

---

### ASR-Q6: Data Integrity
**Quality Attribute**: Reliability  
**Priority**: High  
**Architectural Impact**: High

**Scenario**:
- **Source**: System
- **Stimulus**: Book borrowed simultaneously by two readers
- **Response**: Only one lending succeeds
- **Measure**: 0 data inconsistencies

**Rationale**: Library operations must maintain data consistency to prevent conflicts.

**Architectural Decisions**:
- Database transactions
- JPA entity management
- Optimistic locking (versioning)
- Validation at multiple layers
- Value objects for domain validation

**Affected Components**:
- All service methods
- JPA entities
- Transaction management
- Value object validation

**Verification**:
- Concurrency testing
- Data consistency tests
- Transaction rollback tests

---

### ASR-Q7: GDPR Compliance
**Quality Attribute**: Legal Compliance  
**Priority**: Critical  
**Architectural Impact**: Medium

**Scenario**:
- **Source**: Reader
- **Stimulus**: Requests data deletion
- **Response**: All personal data removed within 30 days
- **Measure**: 100% compliance with data protection regulations

**Rationale**: System handles personal data and must comply with GDPR.

**Architectural Decisions**:
- Explicit consent management fields
- Data anonymization capabilities
- Audit trail for data access
- Privacy by design

**Affected Components**:
- Reader management
- User management
- Data model (consent fields)
- Data deletion services

**Verification**:
- Compliance audit
- Data protection testing
- Privacy policy review

---

### ASR-Q8: Interoperability
**Quality Attribute**: Interoperability  
**Priority**: Medium  
**Architectural Impact**: Medium

**Scenario**:
- **Source**: External system
- **Stimulus**: Requests book catalog data
- **Response**: Data provided in standard format
- **Measure**: REST API with JSON, OpenAPI specification

**Rationale**: System may integrate with other library systems or applications.

**Architectural Decisions**:
- RESTful API with standard HTTP methods
- JSON for data exchange
- OpenAPI specification
- Standard date/time formats (ISO 8601)
- Standard error responses

**Affected Components**:
- All REST APIs
- DTO serialization
- API documentation

**Verification**:
- API conformance testing
- Integration testing
- Standards compliance check

---

### ASR-Q9: Deployability
**Quality Attribute**: Deployability  
**Priority**: Medium  
**Architectural Impact**: Medium

**Scenario**:
- **Source**: DevOps
- **Stimulus**: Deploys new version
- **Response**: Application starts successfully
- **Measure**: < 5 minutes deployment time, zero-downtime possible

**Rationale**: System must be easy to deploy and update.

**Architectural Decisions**:
- Spring Boot executable JAR
- Embedded server (Tomcat)
- Externalized configuration
- Profile-based configuration
- Database migration support

**Affected Components**:
- Build configuration (Maven)
- Application packaging
- Configuration management

**Verification**:
- Deployment testing
- Configuration testing
- Startup time measurement

---

### ASR-Q10: Availability
**Quality Attribute**: Availability  
**Priority**: Medium  
**Architectural Impact**: Low

**Scenario**:
- **Source**: External API failure
- **Stimulus**: API Ninjas service unavailable
- **Response**: System continues functioning without external data
- **Measure**: 99% uptime for core functionality

**Rationale**: System should remain operational even with external service failures.

**Architectural Decisions**:
- Graceful degradation for external services
- Timeout configuration
- Error handling
- Optional features fail independently

**Affected Components**:
- External service integration
- Error handling
- Service resilience

**Verification**:
- Failure injection testing
- Availability monitoring
- Error recovery testing

## 5. Constraints

### ASR-C1: Technology Stack
**Type**: Technical Constraint  
**Priority**: Critical

**Description**: System must use Spring Boot 3.x with Java 17.

**Rationale**: Organizational standard, team expertise, long-term support.

**Impact**: Determines framework, libraries, and development practices.

---

### ASR-C2: Database
**Type**: Technical Constraint  
**Priority**: High

**Description**: H2 database for development; must support migration to PostgreSQL/MySQL.

**Rationale**: Development convenience with production-ready option.

**Impact**: Database abstraction through JPA, standard SQL.

---

### ASR-C3: REST API
**Type**: Interface Constraint  
**Priority**: High

**Description**: All external interfaces must be RESTful APIs.

**Rationale**: Industry standard, client compatibility.

**Impact**: API design, HTTP methods, status codes.

## 6. ASR Priority Matrix

| ASR ID | Description | Business Impact | Technical Risk | Priority |
|--------|-------------|-----------------|----------------|----------|
| ASR-Q1 | Security | Critical | Medium | Critical |
| ASR-Q7 | GDPR Compliance | Critical | Medium | Critical |
| ASR-F1 | Multi-Role User Management | High | Low | High |
| ASR-F2 | Book Lending Workflow | High | Medium | High |
| ASR-Q2 | Modifiability | High | Low | High |
| ASR-Q3 | Testability | High | Low | High |
| ASR-Q4 | API Usability | High | Low | High |
| ASR-Q6 | Data Integrity | High | Medium | High |
| ASR-F3 | Search Capabilities | Medium | Low | Medium |
| ASR-F4 | Photo Management | Medium | Low | Medium |
| ASR-Q5 | Performance | Medium | Low | Medium |
| ASR-Q8 | Interoperability | Medium | Low | Medium |
| ASR-Q9 | Deployability | Medium | Low | Medium |
| ASR-Q10 | Availability | Medium | Low | Medium |
| ASR-F5 | External API Integration | Low | Medium | Low |

## 7. Traceability Matrix

| ASR | Architectural Decision | Component | Verification Method |
|-----|------------------------|-----------|---------------------|
| ASR-Q1 | Spring Security + JWT | Security Config | Security Testing |
| ASR-Q2 | Layered Architecture | All Modules | Code Review |
| ASR-Q3 | Repository Pattern | Repositories | Test Coverage |
| ASR-F1 | RBAC Implementation | User Management | Authorization Tests |
| ASR-F2 | Domain Model | Lending Management | Integration Tests |
| ASR-Q6 | JPA Transactions | All Services | Concurrency Tests |
| ASR-Q4 | OpenAPI Docs | All Controllers | API Documentation |

## 8. References
- [Architecture Overview](./01-Architecture-Overview.md)
- [Quality Attributes](./05-Quality-Attributes.md)
- [Architectural Tactics](./07-Architectural-Tactics.md)
- [Design Alternatives](./10-Design-Alternatives.md)
