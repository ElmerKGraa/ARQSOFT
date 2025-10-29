# Context Views

**View Type**: Context Views (System Context)
**Purpose**: Show the system in its environment - boundaries, external actors, and external systems
**Primary Stakeholders**: All stakeholders, especially business stakeholders and project managers

---

## Overview

Context views show the **system as a black box** within its environment. They answer questions like:
- What are the system boundaries?
- Who are the users (actors)?
- What external systems does it interact with?
- What are the external dependencies?

### Quality Attributes Addressed
- **Interoperability**: How system integrates with external systems
- **Usability**: Who the users are and how they interact
- **Security**: External threat surfaces
- **Deployability**: External dependencies that must be available

---

## View Catalog

| Diagram | Status | Description | File |
|---------|--------|-------------|------|
| System Context (C4 Level 1) | ✅ Complete | Overall system context | SystemContext-C4Level1.puml |
| Actor Diagram | ✅ Complete | User types and their roles | ActorDiagram.puml |
| External Integration Map | ✅ Complete | External systems and APIs | ExternalIntegrationMap.puml |

---

## Primary Presentation

See: [SystemContext-C4Level1.puml](SystemContext-C4Level1.puml)

This diagram shows:
- **Library Management System** (the system being documented)
- **Users**: Librarian, Reader, Anonymous User
- **External Systems**: API Ninjas (historical events service)
- **Interfaces**: REST API, HTTPS

---

## Element Catalog

### System Under Documentation

| Element | Type | Description | Responsibilities |
|---------|------|-------------|------------------|
| **Library Management System** | Software System | Complete library management solution | Manage authors, books, readers, lendings, and reporting |

### External Actors (Human)

| Actor | Type | Description | Interactions |
|-------|------|-------------|--------------|
| **Librarian** | User | Library staff member | Manages books, authors, readers, lendings; generates reports |
| **Reader** | User | Library patron | Borrows books, views catalog, manages own profile |
| **Anonymous User** | User | Unregistered visitor | Can register as new reader |

### External Systems

| System | Type | Protocol | Purpose |
|--------|------|----------|---------|
| **API Ninjas** | External API | HTTPS/REST | Provides historical events data for enrichment |
| **Email System** (Future) | External Service | SMTP | Send notifications for overdue books |

---

## Context Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     SYSTEM CONTEXT                              │
│                                                                 │
│   ┌──────────┐                                  ┌────────────┐ │
│   │Librarian │────┐                         ┌───│API Ninjas  │ │
│   └──────────┘    │                         │   └────────────┘ │
│                   │  HTTPS/REST             │   (Historical   │
│   ┌──────────┐    │                         │    Events)      │
│   │ Reader   │────┤      ┌──────────────────┴──┐              │
│   └──────────┘    ├─────▶│ Library Management │              │
│                   │      │      System        │              │
│   ┌──────────┐    │      └────────────────────┘              │
│   │Anonymous │────┘                                          │
│   └──────────┘                                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Variability Guide

### Variation Points

1. **Authentication Method**
   - Current: JWT with RSA
   - Alternative: OAuth2 with external provider (Google, Microsoft)
   - Alternative: SAML for enterprise integration

2. **External API Integration**
   - Current: API Ninjas (historical events)
   - Extensible: Can add more external services (book metadata, author info)

3. **Client Applications**
   - Current: Generic REST API clients
   - Future: Specific web app, mobile app, admin console

4. **Notification System**
   - Current: None
   - Planned: Email service for overdue notifications

---

## Rationale

### Why These Boundaries?

**System Boundary Decision**:
- **Include**: All library management functionality (authors, books, readers, lendings)
- **Exclude**: User authentication providers (use standard JWT), email service (external)

**Rationale**:
- Focus on core library domain
- Leverage existing external services where appropriate
- Clear separation of concerns
- Enables independent deployment and scaling

### Why These Actors?

**Three Actor Types**:
1. **Librarian**: Privileged user with management capabilities
2. **Reader**: Regular user with borrowing capabilities
3. **Anonymous**: Unprivileged visitor (registration only)

**Rationale**:
- Matches real-world library organization
- Clear separation of responsibilities
- Security model based on roles
- Supports different access levels

### Why API Ninjas?

**External Service Choice**:
- Demonstrates external integration capability
- Provides value-added feature (historical context)
- Low coupling (non-critical service)

**Alternative Considered**: Build own historical events database
**Rejected Because**: Out of scope, maintenance burden, not core value

---

## Related Views

### Links to Other Views

| Related View | Relationship | Description |
|--------------|--------------|-------------|
| [Component & Connector Views](../03-ComponentAndConnectorViews/) | Refinement | C&C views show internal structure of the system |
| [Module Views](../02-ModuleViews/) | Refinement | Module views show code organization inside system |
| [Allocation Views](../04-AllocationViews/) | Mapping | Shows where system components are deployed |

### Traceability

| Element in This View | Maps To (Other Views) |
|----------------------|-----------------------|
| Library Management System | Application Server (Allocation), All Modules (Module View) |
| Librarian | LIBRARIAN role (Security view), Admin operations (Behavior) |
| Reader | READER role (Security view), User operations (Behavior) |
| API Ninjas | External Service component (C&C View) |

---

## Interface Specifications

### REST API Interface

**Protocol**: HTTPS (HTTP in development)
**Format**: JSON
**Authentication**: JWT Bearer Token
**Base URL**: `https://api.library.com/api/` (production)

**Endpoint Categories**:
- `/api/authors` - Author management
- `/api/books` - Book catalog
- `/api/readers` - Reader management
- `/api/lendings` - Lending operations
- `/api/genres` - Genre management
- `/api/public` - Public endpoints (login, registration)

**Documentation**: OpenAPI 3.0 at `/swagger-ui`

### API Ninjas Interface

**Protocol**: HTTPS
**Format**: JSON
**Authentication**: API Key
**Endpoint**: `https://api-ninjas.com/api/historicalevents`

**Purpose**: Fetch historical events for given dates
**Usage**: Enrich author birthday information

---

## Security Considerations

### Trust Boundaries

```
┌─────────────────────────────────────────┐
│         UNTRUSTED ZONE                  │
│  - Anonymous Users                      │
│  - External API Ninjas                  │
└───────────────┬─────────────────────────┘
                │ JWT Authentication
                ▼
┌─────────────────────────────────────────┐
│         TRUSTED ZONE                    │
│  - Authenticated Users (Librarian,      │
│    Reader)                              │
│  - Library Management System            │
│  - Database                             │
└─────────────────────────────────────────┘
```

**Security Measures at Boundary**:
- JWT token validation
- Role-based access control (RBAC)
- Input validation
- HTTPS encryption (production)
- API rate limiting (future)

---

## Constraints

### External Dependencies

| Dependency | Type | Impact if Unavailable |
|------------|------|----------------------|
| **Internet Connection** | Network | Cannot access external APIs, clients cannot connect |
| **API Ninjas Service** | External Service | Historical events feature unavailable (non-critical) |
| **DNS** | Infrastructure | Cannot resolve domain names |

### Integration Constraints

1. **API Ninjas**: Rate limited to 10,000 requests/month on free tier
2. **Client Applications**: Must support JWT bearer token authentication
3. **Network**: Requires HTTPS in production (security requirement)

---

## Assumptions

1. **Network Availability**: System assumes reliable internet connectivity
2. **Client Capabilities**: Clients can handle JSON and REST APIs
3. **External Service Availability**: API Ninjas availability is best-effort (non-critical)
4. **User Authentication**: Users will use username/password (no external auth providers)

---

## Future Evolution

### Planned Additions

1. **Mobile Application** (Priority: High)
   - Native iOS and Android apps
   - Same REST API, different UI

2. **Admin Console** (Priority: Medium)
   - Separate web application for librarians
   - Enhanced reporting and management features

3. **Email Service Integration** (Priority: High)
   - SMTP server for notifications
   - Overdue book reminders
   - New book announcements

4. **External Authentication** (Priority: Low)
   - OAuth2 with Google/Microsoft
   - SAML for enterprise customers
   - Social login

5. **Additional External Services** (Priority: Low)
   - Google Books API (book metadata)
   - Open Library API (cover images)
   - GoodReads API (book ratings)

---

## Document Control

| Attribute | Value |
|-----------|-------|
| **View Type** | Context View |
| **Primary Presentation** | SystemContext-C4Level1.puml |
| **Status** | Complete |
| **Version** | 1.0 |
| **Last Updated** | 2025-10-29 |
| **Owner** | Architecture Team |
| **Reviewers** | Project Manager, Business Stakeholders |
| **Next Review** | 2025-12-29 (Quarterly) |

---

## References

- [C4 Model - System Context](https://c4model.com/#SystemContextDiagram)
- [SEI Views and Beyond - Context Views](https://wiki.sei.cmu.edu/sad/)
- [Architecture Documentation](../../README.md)
- [ASR Document](../../ASR-ArchitecturallySignificantRequirements.md)
