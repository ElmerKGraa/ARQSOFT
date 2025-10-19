# Architecture Overview - Library Management System

## 1. System Context

The Library Management System is a web-based application that manages library operations including book cataloging, author management, reader registration, and book lending processes.

### 1.1 System Purpose
- Manage library book inventory
- Track book lending and returns
- Manage reader accounts and profiles
- Manage author information
- Handle fines for overdue books
- Provide search and reporting capabilities

### 1.2 Stakeholders
- **Librarians**: Primary users who manage books, authors, and lendings
- **Readers**: Users who can borrow books and manage their profile
- **System Administrators**: Manage user accounts and system configuration

## 2. System Architecture

### 2.1 Architectural Style
The system implements a **Layered Architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────┐
│         Presentation Layer              │
│         (REST API / Controllers)        │
├─────────────────────────────────────────┤
│         Application Layer               │
│         (Services / Use Cases)          │
├─────────────────────────────────────────┤
│         Domain Layer                    │
│         (Entities / Value Objects)      │
├─────────────────────────────────────────┤
│         Infrastructure Layer            │
│    (Repositories / External Services)   │
└─────────────────────────────────────────┘
```

### 2.2 Technology Stack
- **Backend Framework**: Spring Boot 3.2.5
- **Language**: Java 17
- **Database**: H2 (in-memory for development)
- **Security**: Spring Security with OAuth2 Resource Server (JWT)
- **API Documentation**: SpringDoc OpenAPI (Swagger)
- **Object Mapping**: MapStruct
- **Validation**: Bean Validation API
- **Testing**: JUnit, Spring Boot Test

### 2.3 Module Organization

The system is organized into bounded contexts following Domain-Driven Design principles:

1. **User Management** (`usermanagement`)
   - User authentication and authorization
   - User account management
   - Role-based access control

2. **Reader Management** (`readermanagement`)
   - Reader registration and profile management
   - Reader search and reporting
   - GDPR consent management

3. **Author Management** (`authormanagement`)
   - Author registration and profile management
   - Author search and reporting
   - Author biography management

4. **Book Management** (`bookmanagement`)
   - Book catalog management
   - Book search and filtering
   - ISBN management

5. **Genre Management** (`genremanagement`)
   - Genre classification
   - Genre-based book organization

6. **Lending Management** (`lendingmanagement`)
   - Lending process management
   - Return tracking
   - Fine calculation and management

7. **Shared** (`shared`)
   - Common domain models
   - Shared value objects
   - Cross-cutting concerns

8. **External Services** (`external`)
   - Integration with external APIs (e.g., API Ninjas for historical events)

## 3. Architectural Principles

### 3.1 Design Principles Applied
1. **Separation of Concerns**: Clear layer boundaries and module independence
2. **Single Responsibility**: Each module handles one business capability
3. **Dependency Inversion**: Abstractions over concrete implementations
4. **Interface Segregation**: Repository interfaces for data access
5. **Open/Closed Principle**: Extension through configuration and polymorphism

### 3.2 Package Structure Convention
Each module follows a consistent structure:
```
<module>
├── api/              # REST controllers and DTOs
├── model/            # Domain entities and value objects
├── repositories/     # Repository interfaces
├── infrastructure/   # Repository implementations
│   └── repositories/
│       └── impl/
└── services/         # Business logic and use cases
```

## 4. Key Architectural Decisions

### 4.1 Layered Architecture
**Decision**: Implement a traditional layered architecture
**Rationale**: 
- Provides clear separation of concerns
- Familiar to development team
- Suitable for the system's complexity level
- Easy to understand and maintain

### 4.2 Repository Pattern
**Decision**: Use Repository pattern for data access
**Rationale**:
- Abstracts data persistence details
- Enables easier testing through mocking
- Provides flexibility to change data sources
- Supports Spring Data JPA integration

### 4.3 DTO Pattern
**Decision**: Use Data Transfer Objects for API layer
**Rationale**:
- Decouples internal domain models from external API
- Provides version control for API contracts
- Enables field-level control over exposed data
- Supports API evolution

### 4.4 Value Objects
**Decision**: Use Value Objects for domain concepts
**Rationale**:
- Encapsulates validation logic
- Provides type safety
- Ensures data integrity
- Improves domain model expressiveness

## 5. System Interfaces

### 5.1 REST API
The system exposes RESTful APIs for all operations:
- Authentication endpoints (`/auth/**`)
- User management endpoints (`/api/admin/user/**`)
- Author management endpoints (`/api/authors/**`)
- Book management endpoints (`/api/books/**`)
- Reader management endpoints (`/api/readers/**`)
- Lending management endpoints (`/api/lendings/**`)
- Genre management endpoints (`/api/genre/**`)

### 5.2 External Integrations
- **API Ninjas**: Historical events service for enrichment features

## 6. Security Architecture

### 6.1 Authentication & Authorization
- JWT-based authentication using OAuth2 Resource Server
- Role-based access control (RBAC)
- Roles: ADMIN, LIBRARIAN, READER

### 6.2 Data Protection
- Password encryption using BCrypt
- GDPR compliance features
- User consent management
- HTML sanitization for user inputs

## 7. Data Architecture

### 7.1 Domain Model
The system implements the following core aggregates:
- **User Aggregate**: User, Librarian, Reader
- **Book Aggregate**: Book, ISBN, Title, Description
- **Author Aggregate**: Author, AuthorBio
- **Lending Aggregate**: Lending, Fine
- **Genre**: Genre classification

### 7.2 Persistence Strategy
- JPA/Hibernate for ORM
- Spring Data JPA for repository implementation
- H2 in-memory database for development
- Schema managed by JPA annotations

## 8. Cross-Cutting Concerns

### 8.1 Exception Handling
Centralized exception handling for:
- Business rule violations
- Validation errors
- Resource not found
- Authentication/authorization failures

### 8.2 Logging
- Structured logging for debugging
- Audit trail for critical operations

### 8.3 Configuration Management
- Externalized configuration through Spring Boot
- Profile-based configuration (dev, prod)

### 8.4 API Documentation
- Automatic API documentation with Swagger/OpenAPI
- Interactive API testing through Swagger UI

## 9. Development Practices

### 9.1 Code Generation
- MapStruct for DTO mapping
- Lombok for reducing boilerplate code
- Annotation processors for compile-time validation

### 9.2 Testing Strategy
- Unit tests for business logic
- Integration tests for API endpoints
- Repository tests with in-memory database

## 10. Quality Attributes Summary

The architecture supports the following quality attributes:
- **Modifiability**: Layered architecture and modular design
- **Testability**: Dependency injection and interface-based design
- **Security**: Authentication, authorization, and data protection
- **Usability**: RESTful API with comprehensive documentation
- **Maintainability**: Clear structure and separation of concerns
- **Interoperability**: Standard REST API and JSON format

## References
- [Component View](./02-Component-View.md)
- [Architecturally Significant Requirements](./04-ASR.md)
- [Quality Attributes](./05-Quality-Attributes.md)
