# Software Architecture Visual Models

**Project**: Library Management System (PSOFT-G1)
**Based on**: SEI CMU Software Architecture Documentation Template
**Reference**: [SEI SAD Template](https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template)
**Approach**: "Views and Beyond" - Multiple architectural views with cross-view mappings

---

## Overview

This directory contains visual models documenting the software architecture following the SEI (Software Engineering Institute) template structure. The architecture is documented through multiple **views**, each addressing different stakeholder concerns and quality attributes.

### Documentation Philosophy

> "The software architecture of a system is the set of structures needed to reason about the system, which comprise software elements, relations among them, and properties of both."
> — SEI Software Architecture Definition

We document architecture through **multiple views** because:
- No single view can capture all architectural concerns
- Different stakeholders need different information
- Quality attributes drive the choice of views
- Multiple views enable complete understanding

---

## Directory Structure

```
VisualModels/
├── 01-ContextViews/              # System context and external interactions
├── 02-ModuleViews/                # Code organization and implementation structures
├── 03-ComponentAndConnectorViews/ # Runtime structures and interactions
├── 04-AllocationViews/            # Mapping to physical/organizational structures
├── 05-DomainModels/               # Business domain models (DDD)
├── 06-BehaviorModels/             # Dynamic behavior and interactions
└── README.md                      # This file
```

---

## View Types and Their Purpose

### 1. Context Views (01-ContextViews/)

**Purpose**: Show the system's place in its environment

**Stakeholders**: All stakeholders, especially business stakeholders, project managers

**Key Questions Answered**:
- What are the system boundaries?
- What external systems does it interact with?
- Who are the users/actors?
- What are the external dependencies?

**Diagrams**:
- System Context Diagram (C4 Level 1)
- Actor Diagram
- External Systems Integration Map
- Ecosystem View

**Quality Attributes**: Interoperability, Usability

---

### 2. Module Views (02-ModuleViews/)

**Purpose**: Show how the system is structured as a set of code units (modules, packages, classes)

**Stakeholders**: Developers, architects, maintainers

**Key Questions Answered**:
- How is the code organized?
- What are the implementation units?
- What are the dependencies between modules?
- How is functionality decomposed?

**Diagrams**:
- Layered Architecture Diagram
- Package Diagram
- Module Dependency Diagram
- Domain Module Organization

**Styles**:
- **Layered Style**: Presentation → Business → Data
- **Module Decomposition**: Breaking system into smaller units
- **Uses Style**: Module A uses services of Module B

**Quality Attributes**: Maintainability, Modifiability, Reusability

---

### 3. Component and Connector Views (03-ComponentAndConnectorViews/)

**Purpose**: Show runtime structures - components and how they interact

**Stakeholders**: Performance engineers, testers, integrators

**Key Questions Answered**:
- What are the runtime components?
- How do components communicate?
- What are the data flows?
- What protocols are used?

**Diagrams**:
- Component Diagram (C4 Level 2 - Container)
- Component Communication Diagram
- Data Flow Diagram
- Service Interaction Diagram

**Styles**:
- **Client-Server Style**: REST API architecture
- **Publish-Subscribe Style**: Event-driven interactions
- **Pipe-and-Filter Style**: Data processing pipelines

**Quality Attributes**: Performance, Scalability, Reliability, Security

---

### 4. Allocation Views (04-AllocationViews/)

**Purpose**: Show mapping to physical or organizational structures

**Stakeholders**: Deployers, system administrators, project managers

**Key Questions Answered**:
- How is the system deployed?
- What hardware is needed?
- How does code map to teams?
- What are the work assignments?

**Diagrams**:
- Deployment Diagram
- Infrastructure Diagram
- Work Assignment View
- Implementation View

**Styles**:
- **Deployment Style**: Software to hardware mapping
- **Implementation Style**: Modules to file system
- **Work Assignment Style**: Modules to teams

**Quality Attributes**: Deployability, Scalability, Performance, Availability

---

### 5. Domain Models (05-DomainModels/)

**Purpose**: Show business domain concepts and their relationships (DDD focus)

**Stakeholders**: Domain experts, business analysts, developers

**Key Questions Answered**:
- What are the key domain concepts?
- How are business entities related?
- What are the aggregates and boundaries?
- What is the ubiquitous language?

**Diagrams**:
- Domain Model Diagram
- Aggregate Diagram
- Entity-Relationship Diagram
- Bounded Context Map
- Value Object Catalog

**Approach**: Domain-Driven Design (DDD)

**Quality Attributes**: Understandability, Business Alignment, Maintainability

---

### 6. Behavior Models (06-BehaviorModels/)

**Purpose**: Show dynamic behavior - how the system behaves over time

**Stakeholders**: Developers, testers, performance engineers

**Key Questions Answered**:
- How do components interact over time?
- What is the sequence of operations?
- What are the state transitions?
- How are use cases realized?

**Diagrams**:
- Sequence Diagrams
- State Machine Diagrams
- Activity Diagrams
- Use Case Realizations
- Collaboration Diagrams

**Quality Attributes**: Correctness, Performance, Understandability

---

## View Selection Guide

### Which Views to Create?

Not all views are needed for every system. Select views based on:

1. **Stakeholder needs**: Who needs to understand what?
2. **Quality attributes**: Which QAs are most important?
3. **System complexity**: More complex = more views
4. **Project phase**: Different phases need different views

### Our System (Library Management)

For the Library Management System, we focus on:

| View Type | Priority | Rationale |
|-----------|----------|-----------|
| **Context Views** | High | Clear system boundaries needed |
| **Module Views** | High | Layered architecture is key structure |
| **Component & Connector** | High | REST API interactions are central |
| **Allocation Views** | Medium | Deployment is straightforward |
| **Domain Models** | High | DDD approach, rich domain |
| **Behavior Models** | Medium | Support key use case understanding |

---

## Cross-View Mappings

Understanding how views relate is crucial. Key mappings:

### Module View ↔ Component & Connector View
- **Question**: Which modules are deployed in which runtime components?
- **Example**: `AuthorService` module (code) → `Business Logic Layer` component (runtime)

### Component & Connector View ↔ Allocation View
- **Question**: Which runtime components run on which hardware?
- **Example**: `API Gateway` component → `Application Server` (Tomcat)

### Domain Model ↔ Module View
- **Question**: How are domain concepts implemented in code?
- **Example**: `Author` aggregate → `authormanagement` package

### Behavior Model ↔ Component & Connector View
- **Question**: How do runtime interactions realize behaviors?
- **Example**: Create Author sequence → AuthorController → AuthorService → AuthorRepository

---

## Diagram Standards and Conventions

### Notation

We use standard notations:
- **UML**: Unified Modeling Language for structure and behavior
- **C4 Model**: Context, Containers, Components, Code
- **ArchiMate**: For enterprise architecture (if needed)
- **Informal Box-and-Line**: For conceptual views

### Tooling

- **PlantUML**: Text-based diagrams (version controllable)
- **Draw.io/diagrams.net**: Complex diagrams
- **Lucidchart**: Collaborative diagramming (if available)

### File Naming Convention

```
[ViewType]-[DiagramType]-[SubjectArea].[ext]

Examples:
- Context-SystemContext-OverallSystem.puml
- Module-LayeredArchitecture-FullSystem.puml
- CC-ComponentDiagram-AuthorManagement.puml
- Allocation-Deployment-ProductionEnvironment.puml
- Domain-AggregateModel-CoreDomain.puml
- Behavior-Sequence-CreateAuthor.puml
```

### Diagram Elements

Each diagram should include:
- **Title**: Clear, descriptive
- **Legend**: Explain symbols and colors
- **Timestamp**: Last updated date
- **Version**: Diagram version number
- **Author**: Who created/maintains it
- **Description**: Brief explanation of what it shows

---

## View Templates

### Template Structure for Each View

Each view should have:

```
ViewName/
├── README.md                    # View overview and catalog
├── PrimaryPresentation.puml     # Main diagram
├── ElementCatalog.md            # List of elements and properties
├── ContextDiagram.puml          # Context for this view
├── VariabilityGuide.md          # Variation points (if applicable)
├── Rationale.md                 # Design decisions for this view
└── RelatedViews.md              # Cross-references to other views
```

### Minimal View Documentation

At minimum, each view must have:
1. **Primary Presentation**: The main diagram
2. **Element Catalog**: What elements mean
3. **Context Diagram**: How this view fits with others
4. **Rationale**: Why this structure was chosen

---

## Quality Attribute Mapping

Different views emphasize different quality attributes:

| View Type | Primary QAs | Secondary QAs |
|-----------|-------------|---------------|
| **Context** | Interoperability | Usability, Security |
| **Module** | Maintainability, Modifiability | Reusability, Testability |
| **C&C** | Performance, Reliability | Scalability, Security |
| **Allocation** | Deployability, Performance | Availability, Scalability |
| **Domain** | Understandability | Maintainability, Correctness |
| **Behavior** | Correctness | Performance, Understandability |

---

## Documentation Guidelines

### Creating a New View

1. **Identify Purpose**: Why is this view needed?
2. **Identify Stakeholders**: Who will use this view?
3. **Choose Style**: What architectural style best captures the concerns?
4. **Create Primary Presentation**: Draw the main diagram
5. **Document Elements**: Create element catalog
6. **Add Context**: Show how it relates to other views
7. **Provide Rationale**: Explain key decisions
8. **Review**: Get feedback from stakeholders

### Maintaining Views

- **Keep synchronized**: When architecture changes, update all affected views
- **Version control**: Use Git for all diagrams (prefer text-based formats)
- **Review regularly**: At least once per major release
- **Deprecate obsolete**: Mark old views as deprecated, don't delete immediately

### Consistency Rules

1. **Same element, same name**: If Author appears in multiple views, use same name
2. **Consistent styling**: Use same colors/shapes for same concepts across views
3. **Explicit relationships**: Show how elements in different views map to each other
4. **Clear boundaries**: Make system boundaries explicit in every view

---

## View Catalog

### Current Views

| View Name | Type | Status | Last Updated | Purpose |
|-----------|------|--------|--------------|---------|
| System Context | Context | ✅ Complete | 2025-10-29 | Show system boundaries |
| Layered Architecture | Module | ✅ Complete | 2025-10-29 | Show 3-tier structure |
| Component Architecture | C&C | ✅ Complete | 2025-10-29 | Show runtime components |
| Deployment Architecture | Allocation | ✅ Complete | 2025-10-29 | Show deployment topology |
| Domain Model | Domain | ✅ Complete | 2025-10-29 | Show business entities |
| Create Author Sequence | Behavior | 🔄 In Progress | - | Show author creation flow |

### Planned Views

| View Name | Type | Priority | Reason |
|-----------|------|----------|--------|
| Security Architecture | C&C | High | Document JWT flow, RBAC |
| Database Schema | Allocation | Medium | Show data persistence |
| Error Handling Flow | Behavior | Low | Document exception handling |
| Book Lending Sequence | Behavior | Medium | Show core use case |

---

## View Dependencies

Some views depend on others:

```
Context Views (Level 0 - System Boundary)
    ↓
Component & Connector Views (Level 1 - Containers)
    ↓
Component Detailed Views (Level 2 - Components)
    ↓
Module Views (Level 3 - Code Structure)
```

**Rule**: Higher-level views should be created before detailed views.

---

## Stakeholder-View Matrix

Which stakeholders need which views?

| Stakeholder | Context | Module | C&C | Allocation | Domain | Behavior |
|-------------|---------|--------|-----|------------|--------|----------|
| **Business Stakeholder** | ✅✅ | ❌ | ❌ | ❌ | ✅✅ | ✅ |
| **Project Manager** | ✅✅ | ✅ | ✅ | ✅✅ | ❌ | ❌ |
| **Architect** | ✅✅ | ✅✅ | ✅✅ | ✅✅ | ✅✅ | ✅ |
| **Developer** | ✅ | ✅✅ | ✅✅ | ✅ | ✅✅ | ✅✅ |
| **Tester** | ✅ | ✅ | ✅✅ | ❌ | ✅ | ✅✅ |
| **DevOps** | ✅ | ❌ | ✅ | ✅✅ | ❌ | ❌ |
| **Security Engineer** | ✅ | ✅ | ✅✅ | ✅ | ❌ | ✅ |
| **Performance Engineer** | ✅ | ❌ | ✅✅ | ✅✅ | ❌ | ✅ |

Legend: ✅✅ = Critical, ✅ = Important, ❌ = Not needed

---

## Integration with Existing Documentation

These visual models complement existing documentation:

### Relationship to ASR Document
- ASRs → Quality Attributes → Drive View Selection
- Example: ASR-SEC-01 (JWT) → Requires Security Architecture view (C&C)

### Relationship to Requirements
- Functional Requirements → Realized in Behavior Views (sequences, use cases)
- Non-Functional Requirements → Addressed in specific views based on QA

### Relationship to Domain Documentation
- Glossary → Element names in all views
- Domain Model → Basis for Domain Views
- Aggregates → Module boundaries in Module Views

---

## Best Practices

### Do's ✅

1. **Start with context**: Always begin with system context view
2. **Choose views purposefully**: Each view should answer specific questions
3. **Keep synchronized**: Update all affected views when architecture changes
4. **Use standard notations**: UML, C4, ArchiMate - don't invent your own
5. **Explain rationale**: Always document WHY, not just WHAT
6. **Cross-reference**: Link related views explicitly
7. **Consider audience**: Tailor detail level to stakeholders

### Don'ts ❌

1. **Don't create unnecessary views**: More views = more maintenance
2. **Don't mix concerns**: Keep views focused on one perspective
3. **Don't duplicate**: If information is in one view, reference it, don't copy
4. **Don't use proprietary formats**: Prefer text-based (PlantUML) for version control
5. **Don't ignore consistency**: Same names for same elements across views
6. **Don't forget context**: Every view needs context diagram
7. **Don't over-detail**: Use appropriate abstraction level

---

## References

### Standards and Templates
- [SEI Software Architecture Documentation Template](https://wiki.sei.cmu.edu/confluence/display/SAD/Software+Architecture+Documentation+Template)
- [Documenting Software Architectures: Views and Beyond, 2nd Edition](https://www.sei.cmu.edu/library/abstracts/books/9780321552686.cfm)
- [C4 Model for Visualizing Software Architecture](https://c4model.com/)
- [UML 2.5 Specification](https://www.omg.org/spec/UML/)

### Books
- **"Documenting Software Architectures: Views and Beyond"** - Paul Clements et al. (SEI Series)
- **"Software Architecture in Practice"** - Len Bass et al.
- **"Clean Architecture"** - Robert C. Martin
- **"Domain-Driven Design"** - Eric Evans

### Tools
- [PlantUML](https://plantuml.com/) - Text-based UML diagrams
- [Structurizr](https://structurizr.com/) - C4 Model tooling
- [ArchiMate](https://www.opengroup.org/archimate-forum) - Enterprise Architecture

---

## Getting Started

### For New Contributors

1. **Read this README** completely
2. **Review existing views** in each directory
3. **Understand the system** (read Architecture/ docs)
4. **Choose appropriate view type** for your need
5. **Follow naming conventions** and templates
6. **Create primary presentation** (diagram)
7. **Document elements** (element catalog)
8. **Get review** from architecture team

### For Viewers/Consumers

1. **Start with Context Views** (01-ContextViews/)
2. **Check stakeholder-view matrix** (which views do you need?)
3. **Read view README** for each view you explore
4. **Follow cross-references** to related views
5. **Consult glossary** for term definitions

---

## Maintenance

### Review Schedule
- **Minor review**: After each architectural change
- **Major review**: Quarterly or before major releases
- **Comprehensive review**: Annually

### Change Process
1. Architecture change identified
2. Determine affected views
3. Update views
4. Update cross-view mappings
5. Review with stakeholders
6. Merge to main branch

### Deprecation Process
1. Mark view as deprecated in README
2. Add deprecation date and reason
3. Point to replacement view (if any)
4. Keep for 6 months
5. Move to archive/ directory
6. Delete after 1 year if not referenced

---

## FAQ

**Q: Do I need all six view types?**
A: No. Create views based on stakeholder needs and quality attribute priorities. Our system focuses on Context, Module, C&C, and Domain views.

**Q: What's the difference between Component (C&C) and Module views?**
A: Module views show code organization (static). C&C views show runtime behavior (dynamic). A module might map to multiple runtime components.

**Q: Should I use UML or C4?**
A: Use both! C4 for high-level (context, container), UML for detailed (components, classes, sequences).

**Q: How detailed should diagrams be?**
A: Use appropriate abstraction. High-level views for executives (boxes and lines). Detailed views for developers (all attributes, methods).

**Q: Can I create custom view types?**
A: Yes, if standard views don't address your concerns. Document purpose clearly and follow same template structure.

**Q: How do I keep views synchronized with code?**
A: Review views regularly, especially after architectural changes. Use architecture decision records (ADRs) to track changes.

---

## Document Control

| Attribute | Value |
|-----------|-------|
| **Version** | 1.0 |
| **Status** | Complete |
| **Last Updated** | 2025-10-29 |
| **Owner** | Architecture Team |
| **Template Source** | SEI CMU SAD Template |
| **Next Review** | 2026-01-29 (Quarterly) |

---

## Quick Links

### Visual Models Directories
- [01-ContextViews/](01-ContextViews/)
- [02-ModuleViews/](02-ModuleViews/)
- [03-ComponentAndConnectorViews/](03-ComponentAndConnectorViews/)
- [04-AllocationViews/](04-AllocationViews/)
- [05-DomainModels/](05-DomainModels/)
- [06-BehaviorModels/](06-BehaviorModels/)

### Related Documentation
- [Architecture Documentation Index](../README.md)
- [ASR Document](../ASR-ArchitecturallySignificantRequirements.md)
- [Functional Requirements](../FunctionalRequirements.md)
- [Non-Functional Requirements](../NonFunctionalRequirements.md)

---

**Note**: This structure follows the SEI (Software Engineering Institute) approach to software architecture documentation, based on the principle that architecture should be documented through multiple, coordinated views, each addressing specific stakeholder concerns and quality attributes.
