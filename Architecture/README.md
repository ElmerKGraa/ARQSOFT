# Architecture Documentation

**Project**: Library Management System (PSOFT-G1)
**Date**: 2025-10-29
**Version**: 1.0
**Status**: System As-Is Documentation (Reverse Engineered)

---

## Overview

This directory contains comprehensive architecture documentation for the Library Management System, extracted through reverse engineering of the implemented codebase. The documentation provides a complete view of the system's architecture, requirements, and design decisions.

### Documentation Goals

1. **System-as-is**: Document the current architecture as implemented
2. **Requirements**: Formalize functional and non-functional requirements from code
3. **ASRs**: Identify architecturally significant requirements and design decisions
4. **Classification**: Document the genre classification system with enhancement recommendations

---

## Document Index

### Core Architecture Documents

#### 1. [Architecturally Significant Requirements (ASRs)](ASR-ArchitecturallySignificantRequirements.md)
**Purpose**: Documents the key quality attributes, architectural decisions, and requirements that shaped the system architecture.

**Contents**:
- Quality attributes (Security, Performance, Maintainability, Usability, etc.)
- 18 detailed ASRs with scenarios and evidence
- 5 major architectural decisions (AD-01 through AD-05)
- Traceability matrix linking ASRs to implementation
- Validation scenarios

**Key Sections**:
- Security ASRs (JWT, RBAC, password security)
- Performance ASRs (optimistic locking, pagination)
- Maintainability ASRs (layered architecture, DDD)
- Integration ASRs (external APIs, file storage)

**Target Audience**: Architects, technical leads, developers

---

#### 2. [System Architecture Diagrams](SystemArchitecture.puml)
**Purpose**: Visual representation of the system's component architecture.

**Diagram Type**: PlantUML Component Diagram

**Shows**:
- Three-layer architecture (Presentation → Business → Data)
- 7 domain modules (Author, Book, Reader, Lending, Genre, User, Auth)
- Security infrastructure (JWT, authentication, authorization)
- Shared infrastructure (file storage, exception handling)
- Cross-cutting concerns
- Component dependencies

**How to Use**:
1. Open in PlantUML viewer or IDE plugin
2. Generate diagram: [SystemArchitecture.svg](SystemArchitecture.svg) (if rendered)
3. Use for onboarding, architecture reviews, documentation

**Target Audience**: All stakeholders

---

#### 3. [Deployment Architecture](DeploymentArchitecture.puml)
**Purpose**: Documents how the system is deployed and its runtime environment.

**Diagram Type**: PlantUML Deployment Diagram

**Shows**:
- Client environments (web browser, mobile app, API testing tools)
- Application server (Spring Boot with embedded Tomcat)
- H2 Database (TCP server, file-based, in-memory options)
- File system (configuration, uploads, keys)
- External services (API Ninjas)
- Network protocols (HTTP/HTTPS, JDBC)

**Deployment Options**:
- Standalone JAR execution
- IDE run configuration
- Maven spring-boot:run

**Target Audience**: DevOps, system administrators, architects

---

#### 4. [Container Architecture (C4 Model)](ContainerArchitecture.puml)
**Purpose**: High-level container diagram following the C4 model.

**Diagram Type**: PlantUML C4 Container Diagram

**Shows**:
- System boundary
- Containers: API Gateway, Business Logic, Data Access, Authentication, File Storage
- Database container (H2)
- External systems (API Ninjas, future email system)
- User personas (Librarian, Reader, Anonymous)
- Container interactions and protocols

**Target Audience**: Architects, business stakeholders, new developers

---

#### 5. [Layered Architecture (Detailed)](LayeredArchitecture.puml)
**Purpose**: Detailed view of the three-layer architecture with all classes.

**Diagram Type**: PlantUML Class Diagram (architectural view)

**Shows**:
- All controllers, services, repositories, and domain entities
- Layer boundaries and dependencies
- Design patterns (DTO, Repository, Service, Value Object)
- Package organization
- Domain relationships

**Target Audience**: Developers, technical leads

---

### Requirements Documents

#### 6. [Functional Requirements](FunctionalRequirements.md)
**Purpose**: Complete catalog of functional requirements extracted from implementation.

**Contents**:
- 44 functional requirements organized by module
- Each requirement includes: ID, description, actor, priority, implementation evidence, REST API endpoint
- Requirements traceability matrix (Phase 1 and Phase 2 use cases)
- Summary statistics

**Modules Covered**:
- Author Management (FR-AUT-01 through FR-AUT-07)
- Book Management (FR-BOOK-01 through FR-BOOK-07)
- Reader Management (FR-READ-01 through FR-READ-06)
- Lending Management (FR-LEND-01 through FR-LEND-05)
- Genre Management (FR-GENRE-01 through FR-GENRE-02)
- User Management (FR-USER-01 through FR-USER-02)
- Reporting (FR-REPORT-01 through FR-REPORT-10)
- Authentication & Authorization (FR-AUTH-01 through FR-AUTH-02)

**Implementation Status**: 100% Complete (all requirements implemented)

**Target Audience**: Business analysts, product owners, QA engineers, developers

---

#### 7. [Non-Functional Requirements (NFR)](NonFunctionalRequirements.md)
**Purpose**: Documents quality attributes and non-functional requirements.

**Contents**:
- 30 non-functional requirements organized by ISO/IEC 25010 quality model
- Each NFR includes: ID, description, priority, implementation evidence, measurement criteria, status
- Traceability matrix linking NFRs to code
- Quality attribute scenarios
- Recommendations for improvements

**Categories (ISO/IEC 25010)**:
- Security (7 NFRs) - Authentication, authorization, password security, input validation
- Performance (5 NFRs) - Response time, concurrency, pagination, file uploads
- Reliability (4 NFRs) - Data integrity, transactions, error handling, backup
- Maintainability (5 NFRs) - Code organization, documentation, testability, configuration
- Usability (4 NFRs) - API documentation, RESTful design, error messages
- Scalability (2 NFRs) - Horizontal scalability, database scalability
- Portability (2 NFRs) - Platform independence, containerization
- Compliance (3 NFRs) - GDPR, data retention, audit trail
- Interoperability (3 NFRs) - REST API, external integrations, client libraries

**Implementation Status**:
- ✅ Fully Implemented: 20 (67%)
- ⚠️ Partially Implemented: 8 (27%)
- ❌ Not Implemented: 2 (6%)

**Target Audience**: Architects, quality assurance, compliance officers, technical leads

---

### Domain-Specific Documents

#### 8. [Genre Classification System](GenreClassificationSystem.md)
**Purpose**: Comprehensive documentation of the genre classification system with enhancement recommendations.

**Contents**:
- Current system overview (as-is)
- Domain model (Genre entity, relationships)
- Classification rules and business logic
- API operations (7 genre-related endpoints)
- Reporting and analytics capabilities
- Data model and database schema
- 8 enhancement recommendations (prioritized)
- Integration points with other modules

**Key Features**:
- Single genre per book (current)
- Genre-based book search
- Genre-based analytics (top 5, trends, duration)
- Reader genre interests for personalization

**Enhancement Roadmap**:
1. **Priority 1** (High-Value):
   - Hierarchical genre classification
   - Multiple genres per book
   - Genre management API

2. **Priority 2** (Analytics):
   - Genre recommendation engine
   - Advanced analytics

3. **Priority 3** (QoL):
   - Genre aliases/synonyms
   - Genre descriptions/metadata
   - Genre tagging (folksonomy)

**Target Audience**: Product managers, librarians, developers working on classification features

---

## Architecture Summary

### Architecture Type: **Layered Architecture (3-Tier)**

```
┌────────────────────────────────────┐
│   Presentation Layer (API)         │
│   - REST Controllers               │
│   - DTOs, View Mappers             │
│   - Input Validation               │
└────────────┬───────────────────────┘
             │
┌────────────▼───────────────────────┐
│   Business Layer (Services)        │
│   - Business Logic                 │
│   - Domain Operations              │
│   - Transaction Management         │
└────────────┬───────────────────────┘
             │
┌────────────▼───────────────────────┐
│   Data Layer (Repositories)        │
│   - Data Access                    │
│   - JPA Repositories               │
│   - Database Queries               │
└────────────┬───────────────────────┘
             │
┌────────────▼───────────────────────┐
│   H2 Database                      │
└────────────────────────────────────┘
```

### Technology Stack

**Core**:
- Java 17+
- Spring Boot 3.x
- Spring Framework (MVC, Data JPA, Security)
- Jakarta EE (Persistence, Validation)

**Database**:
- H2 Database (embedded)
- Hibernate/JPA ORM

**Security**:
- Spring Security
- JWT (JSON Web Tokens)
- RSA asymmetric encryption
- BCrypt password hashing

**API**:
- RESTful architecture
- OpenAPI/Swagger documentation
- JSON serialization (Jackson)

### Domain Modules (7)

1. **Author Management** - Authors, biographies, photos, co-authors
2. **Book Management** - Books, ISBNs, genres, descriptions
3. **Reader Management** - Readers, profiles, interests, photos
4. **Lending Management** - Lendings, returns, fines, overdue tracking
5. **Genre Management** - Genres, classification, analytics
6. **User Management** - Users, authentication, roles (ADMIN, LIBRARIAN, READER)
7. **Reporting** - Top 5 reports, analytics, trends

### Key Design Patterns

- **Layered Architecture** - Separation of concerns across three layers
- **Domain-Driven Design** - Aggregates, value objects, repositories
- **Repository Pattern** - Data access abstraction
- **Service Pattern** - Business logic encapsulation
- **DTO Pattern** - Data transfer between layers
- **Mapper Pattern** - Entity ↔ DTO conversion
- **Dependency Injection** - Spring IoC container
- **Optimistic Locking** - Concurrency control with @Version

---

## How to Use This Documentation

### For New Developers

**Recommended Reading Order**:
1. Start with [README.md](README.md) (this document) for overview
2. Review [SystemArchitecture.puml](SystemArchitecture.puml) for visual understanding
3. Read [Functional Requirements](FunctionalRequirements.md) to understand what the system does
4. Study [Layered Architecture](LayeredArchitecture.puml) for detailed class-level view
5. Explore domain-specific documentation as needed (e.g., [Genre Classification](GenreClassificationSystem.md))

**Hands-On**:
1. Clone repository
2. Read [Deployment Architecture](DeploymentArchitecture.puml) for setup instructions
3. Run application: `mvn spring-boot:run`
4. Access Swagger UI: `http://localhost:8080/swagger-ui`
5. Test endpoints with Postman collections: [Docs/Psoft-G1.postman_collection.json](../Psoft-G1.postman_collection.json)

---

### For Architects

**Focus Areas**:
1. [ASR Document](ASR-ArchitecturallySignificantRequirements.md) - Understand key architectural decisions
2. [Non-Functional Requirements](NonFunctionalRequirements.md) - Assess quality attributes
3. [Architecture Diagrams](SystemArchitecture.puml) - Visualize structure
4. Enhancement recommendations in domain documents

**Key Questions Answered**:
- Why layered architecture? → [AD-01](ASR-ArchitecturallySignificantRequirements.md#ad-01-selection-of-layered-architecture)
- Why JWT with RSA? → [AD-02](ASR-ArchitecturallySignificantRequirements.md#ad-02-jwt-with-rsa-keys-for-authentication)
- Why H2 database? → [AD-03](ASR-ArchitecturallySignificantRequirements.md#ad-03-h2-embedded-database)
- Why DDD patterns? → [AD-04](ASR-ArchitecturallySignificantRequirements.md#ad-04-domain-driven-design-patterns)
- Why optimistic locking? → [AD-05](ASR-ArchitecturallySignificantRequirements.md#ad-05-optimistic-locking-strategy)

---

### For Product Owners

**Focus Areas**:
1. [Functional Requirements](FunctionalRequirements.md) - Understand implemented features
2. [Genre Classification System](GenreClassificationSystem.md) - Explore enhancement opportunities
3. Requirements traceability matrices - Map features to use cases

**Key Metrics**:
- **44 functional requirements** implemented across 7 modules
- **100% completion** of Phase 1 and Phase 2 use cases
- **30 non-functional requirements** documented (67% fully implemented)

---

### For QA Engineers

**Focus Areas**:
1. [Functional Requirements](FunctionalRequirements.md) - Test case derivation
2. [Non-Functional Requirements](NonFunctionalRequirements.md) - Quality attribute testing
3. [ASR Scenarios](ASR-ArchitecturallySignificantRequirements.md) - Architecture validation

**Testing Artifacts**:
- Postman collections: [Docs/Psoft-G1.postman_collection.json](../Psoft-G1.postman_collection.json)
- Test files: 23 unit/integration tests in codebase
- API documentation: Swagger UI at `/swagger-ui`

---

### For DevOps

**Focus Areas**:
1. [Deployment Architecture](DeploymentArchitecture.puml) - Deployment topology
2. [Non-Functional Requirements](NonFunctionalRequirements.md) - Infrastructure requirements
3. Configuration management - [application.properties](../../src/main/resources/application.properties)

**Deployment Checklist**:
- [ ] Java 17+ installed
- [ ] H2 database configured (TCP/file/in-memory)
- [ ] RSA keys generated and placed in classpath
- [ ] Configuration files customized (application.properties, library.properties)
- [ ] File upload directory created (uploads-psoft-g1/)
- [ ] HTTPS configured (production)
- [ ] CORS configured for allowed origins
- [ ] API Ninjas key configured (optional)

---

## Documentation Standards

### Document Structure

All architecture documents follow this structure:
1. **Header** - Title, project, date, version, status
2. **Table of Contents** - Navigable section links
3. **Introduction** - Purpose, scope, audience
4. **Main Content** - Organized by logical sections
5. **Traceability** - Links to code, other documents
6. **References** - External resources
7. **Document Control** - Version, status, review date

### Traceability

All requirements and ASRs include:
- **Implementation Evidence** - Links to source code (file:line)
- **Test Evidence** - Links to test files (where applicable)
- **Configuration** - Links to config files
- **Diagrams** - References to visual representations

### Code References

Format: `[FileName.java:line](path/to/file#Lline)`

Example: [SecurityConfig.java:179](../../src/main/java/pt/psoft/g1/psoftg1/configuration/SecurityConfig.java#L179)

### Diagrams

- **Format**: PlantUML (.puml) for version control and easy updates
- **Rendering**: Use PlantUML plugin in IDE or online renderer
- **Maintenance**: Update diagrams when architecture changes

---

## Related Documentation

### Existing Project Documentation

- **Domain Model**: [../GlobalArtifacts/DomainModel.puml](../GlobalArtifacts/DomainModel.puml)
- **Glossary**: [../GlobalArtifacts/Glossary.md](../GlobalArtifacts/Glossary.md)
- **Domain Artifacts**: [../GlobalArtifacts/](../GlobalArtifacts/)
  - Aggregates: Author, Book, Lending, Reader
  - Entities: Fine, Genre, Librarian, Photo, User
  - Value Objects: ISBN, Title, Name, Email, Password, etc.
- **Use Cases - Phase 1**: [../Phase1/](../Phase1/)
- **Use Cases - Phase 2**: [../Phase2/](../Phase2/)
- **REST API Mapping**: [../Phase2/RestMapping.md](../Phase2/RestMapping.md)
- **Postman Collections**: [../Psoft-G1.postman_collection.json](../Psoft-G1.postman_collection.json)

### External References

- **Spring Boot Documentation**: https://docs.spring.io/spring-boot/docs/current/reference/html/
- **Spring Security**: https://docs.spring.io/spring-security/reference/
- **Domain-Driven Design**: https://www.domainlanguage.com/ddd/
- **C4 Model**: https://c4model.com/
- **ISO/IEC 25010 Quality Model**: https://iso25000.com/index.php/en/iso-25000-standards/iso-25010

---

## Maintenance and Updates

### Document Ownership

**Architecture Team** is responsible for maintaining architecture documentation.

### Review Schedule

- **Minor Reviews**: After each feature addition or bug fix affecting architecture
- **Major Reviews**: Quarterly or after significant architectural changes
- **Annual Review**: Full documentation review and update

### Change Process

1. **Code Change** → Update implementation
2. **Test Change** → Update test evidence
3. **Document Change** → Update architecture documentation
4. **Review** → Architecture team reviews consistency
5. **Approve** → Merge documentation changes

### Version Control

- All documents versioned in Git
- Document version in header
- Change history in commit messages
- Architecture Decision Records (ADRs) for major decisions

---

## Contributing

### How to Contribute to Architecture Documentation

1. **Identify Gap**: Found missing or outdated documentation?
2. **Create Issue**: Document the gap in issue tracker
3. **Update Documentation**: Create branch, update documents
4. **Validate**: Ensure traceability to code
5. **Review**: Submit for architecture team review
6. **Merge**: After approval, merge to main

### Documentation Tools

- **Markdown Editor**: VS Code, IntelliJ IDEA, any text editor
- **PlantUML**: IDE plugin or online renderer (https://www.plantuml.com/plantuml/)
- **Diagrams**: PlantUML preferred for version control
- **Screenshots**: Only when diagram is not feasible

---

## FAQ

### Q: Why was this documentation created?

**A**: This documentation was reverse-engineered from the implemented codebase to provide a comprehensive "as-is" view of the system architecture. It serves as a knowledge base for new developers, supports architecture reviews, and provides a foundation for future enhancements.

### Q: Is this documentation complete?

**A**: This documentation covers the core architecture, requirements, and design decisions. Domain-specific documentation (like Genre Classification System) is comprehensive for key areas. Additional domain-specific documentation can be created as needed.

### Q: How do I know if documentation is up-to-date?

**A**: Check the "Last Updated" date in the document header and the "Status" field. Documents marked "Complete" are current as of the date shown. If code has changed since the date, documentation may need updating.

### Q: What if I find discrepancies between code and documentation?

**A**: Create an issue documenting the discrepancy. The code is the source of truth. Documentation should be updated to match the code, unless the code has a bug that needs fixing.

### Q: How do I generate diagrams from PlantUML files?

**A**:
- **IDE Plugin**: Install PlantUML plugin for VS Code, IntelliJ IDEA, or Eclipse
- **Online**: Paste .puml content into https://www.plantuml.com/plantuml/
- **Command Line**: Use PlantUML JAR with `java -jar plantuml.jar file.puml`

### Q: Who should I contact for architecture questions?

**A**: Contact the Architecture Team or create an issue in the project repository with the "architecture" tag.

---

## Document Control

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0 |
| **Status** | Complete |
| **Last Updated** | 2025-10-29 |
| **Next Review** | Upon major architectural changes |
| **Owner** | Architecture Team |
| **Approvers** | Technical Lead, Project Manager |

---

## Quick Links

### Documentation Files
- [ASR - Architecturally Significant Requirements](ASR-ArchitecturallySignificantRequirements.md)
- [Functional Requirements](FunctionalRequirements.md)
- [Non-Functional Requirements](NonFunctionalRequirements.md)
- [Genre Classification System](GenreClassificationSystem.md)

### Diagrams
- [System Architecture (Component)](SystemArchitecture.puml)
- [Deployment Architecture](DeploymentArchitecture.puml)
- [Container Architecture (C4)](ContainerArchitecture.puml)
- [Layered Architecture (Detailed)](LayeredArchitecture.puml)

### Project Resources
- [Domain Model](../GlobalArtifacts/DomainModel.puml)
- [Glossary](../GlobalArtifacts/Glossary.md)
- [REST API Mapping](../Phase2/RestMapping.md)
- [Postman Collection](../Psoft-G1.postman_collection.json)

---

**Generated**: 2025-10-29
**Method**: Reverse Engineering from Implementation
**Codebase**: 144 Java files, 75+ documentation files analyzed
