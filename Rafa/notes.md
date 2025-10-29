# Aplicação do Processo ADD 

## STEP 1: Review Inputs 
### Identificação dos drivers arquiteturais:
#### Primary Functional Requirements:

- FR1: Gerar IDs em formatos configuráveis (base65+timestamp+hex)
- FR2: Buscar ISBN por título em APIs externas (ISBNdb, Google Books, Open Library)
- FR3: Persistir dados em diferentes backends (SQL, MongoDB, ElasticSearch) com Redis

#### Quality Attribute Scenarios (ASR - Architecturally Significant Requirements):
| Quality Attribute | Cenário | Importância |
| Configurabilidade | Trocar estratégia de geração de ID via properties sem recompilação | CRÍTICA |
| Extensibilidade | Adicionar novo provider de ISBN sem modificar código existente | ALTA |
| Modifiabilidade | Substituir backend de persistência (SQL→MongoDB) via configuração | CRÍTICA |
| Testabilidade | Testar cada estratégia isoladamente com mocks | ALTA |
| Interoperabilidade | Integrar com múltiplas APIs externas (ISBNdb, Google Books, etc.) | MÉDIA |

##### Constraints:
- Uso obrigatório de Spring Boot/Spring Framework
- Estrutura de ID mandatória: [base65+timestamp+hexadecimal]
- Manter compatibilidade com código legado (LendingNumber, ReaderNumber)

##### Concerns (Cross-cutting):

- Configuration management
- Dependency injection
- Exception handling
- Logging


## STEP 2: Establish Goals and Select Inputs 🎯
### Iteration Goals:
#### Iteração 1 (FASE 1): Configurabilidade de Geração de IDs

- Driver Principal: Configurabilidade
- Elementos afetados: LendingNumber, ReaderNumber, outros IDs de entidades

#### Iteração 2 (FASE 2): Extensibilidade para ISBN Lookup

- Driver Principal: Extensibilidade + Interoperabilidade
- Elementos afetados: BookServiceImpl, integração com APIs externas

##### Iteração 3 (FASE 3): Modifiabilidade da Camada de Persistência

- Driver Principal: Modifiabilidade + Portabilidade
- Elementos afetados: Repositories, JpaConfig, persistência


## STEP 3: Choose Elements to Decompose 🏗️
### Elementos identificados para decomposição:

- ID Generation Subsystem: LendingNumber, ReaderNumber → IdGenerator abstraction
- ISBN Lookup Subsystem: BookServiceImpl (hardcoded logic) → IsbnLookupService abstraction
- Persistence Layer: SpringDataXxxRepository → Repository abstraction + implementations


## STEP 4: Choose Design Concepts (Tactics & Patterns) 🛠️
Aqui está o cerne da justificativa arquitetural baseada no documento ADD:

### 4.1 Tactics para CONFIGURABILIDADE (Flexibility)
Do documento ADD:

"Configuration-Driven Behavior: Allow behavior to be modified through configuration files instead of code changes."

Táticas Aplicadas:
| Tática | Implementação | Justificativa | 
| Configuration-Driven Behavior | `@Value`, `@ConfigurationProperties` para ler `application.properties` |Permite trocar estratégias em setup time sem recompilação | 
| Use of Abstractions | Interfaces `IdGenerator`, `IsbnLookupService`, `Repository<T>` | Permite futuras mudanças com impacto mínimo |
| Dynamic Binding | Spring DI injeta implementação baseada em properties |Binding da estratégia em runtime baseado em configuração |

### Setup time configuration
id.generation.strategy=configurable  # ou legacy
```
→ Spring automaticamente injeta `ConfigurableIdGenerator` ou `LegacyIdGenerator`

---

#### **4.2 Tactics para EXTENSIBILIDADE & MODIFIABILIDADE**

Do documento ADD:
> *"Use of Plugins or Extensions: Implement a plugin architecture to allow new features without modifying core components."*
> *"Encapsulation: Group related functionality and hide internal details."*
> *"Use of Interfaces: Define interfaces for components to allow easy replacement or modification."*

**Táticas Aplicadas:**

| Tática | Implementação | Justificativa |
|--------|--------------|---------------|
| **Encapsulation** | Factories (`IdGeneratorFactory`, `IsbnLookupFactory`) escondem lógica de criação | Isola complexidade de instanciação |
| **Use of Interfaces** | `IdGenerator`, `IsbnLookupService` como contratos | Permite **substituição fácil** de implementações |
| **Separation of Concerns** | Camadas: Model / Repository / Service / Config | **Responsabilidade única** por camada |
| **Abstract Common Services** | `IdGenerator` reutilizado por todas as entidades | Evita **duplicação** de lógica |
| **Reduce Coupling** | Services dependem de interfaces, não implementações | **Low coupling, high cohesion** |

---

#### **4.3 Tactics para TESTABILIDADE**

Do documento ADD:
> *"Dependency Injection: Facilitate testing by injecting dependencies."*
> *"Isolation: Design components to function independently, allowing targeted testing."*

**Táticas Aplicadas:**

| Tática | Implementação | Justificativa |
|--------|--------------|---------------|
| **Dependency Injection** | Constructor injection via Spring | Permite **injetar mocks** em testes |
| **Isolation** | Cada estratégia é **independente** | Testes unitários **isolados** por estratégia |
| **Mocking** | Interfaces facilitam uso de Mockito | Simular dependências externas (APIs, DB) |

---

#### **4.4 Architectural Patterns (Design Concepts)**

Do documento ADD:
> *"Design concepts are building blocks from which the design is created: Reference Architectures, Deployment patterns, Architecture Patterns, Tactics, and Externally developed components."*

**Padrões Arquiteturais Aplicados:**

| Padrão | Onde Aplicado | Motivo (Relação com Quality Attributes) |
|--------|--------------|----------------------------------------|
| **Strategy Pattern** | `IdGenerator`, `IsbnLookupService`, `Repository` | **Configurabilidade**: Trocar algoritmos/estratégias em runtime |
| **Factory Pattern** | `IdGeneratorFactory`, `IsbnLookupFactory` | **Extensibilidade**: Centralizar criação baseada em config |
| **Adapter Pattern** | `CompositeIsbnLookupService` | **Interoperabilidade**: Adaptar múltiplas APIs externas |
| **Decorator Pattern** | `RedisBookCache` (wrapper sobre Repository) | **Performance**: Adicionar caching transparentemente |
| **Template Method** | `AbstractRepository` (se necessário) | **Modifiabilidade**: Código comum reutilizável |
| **Dependency Injection** | Spring Framework | **Testabilidade + Modifiabilidade**: Injetar dependências |

---

#### **4.5 Externally Developed Components**

- **Spring Boot**: Framework base com DI container
- **Spring Data JPA**: Abstração para SQL
- **Spring Data MongoDB**: Suporte MongoDB
- **Spring Data Redis**: Cache layer
- **WebClient/RestTemplate**: HTTP clients para APIs externas

---

### **STEP 5: Instantiate Elements and Define Interfaces** 🏛️

**Artefatos Criados:**

#### **FASE 1 - ID Generation:**
```
Interfaces:
  - IdGenerator (contract)
Implementations:
  - ConfigurableIdGenerator (novo formato)
  - LegacyIdGenerator (formato antigo)
Factories:
  - IdGeneratorFactory
Configuration:
  - IdGeneratorProperties
  - IdGeneratorConfig
```

#### **FASE 2 - ISBN Lookup:**
```
Interfaces:
  - IsbnLookupService (contract)
Implementations:
  - IsbnDbLookupService
  - GoogleBooksLookupService
  - OpenLibraryLookupService
  - CompositeIsbnLookupService (combina 2)
Factories:
  - IsbnLookupFactory
Configuration:
  - IsbnLookupProperties
  - IsbnLookupConfig
```

#### **FASE 3 - Persistence:**
```
Interfaces:
  - BookRepository (abstração genérica)
Implementations:
  - JpaSqlBookRepository
  - MongoBookRepository
  - ElasticSearchBookRepository
Decorators:
  - RedisBookCache (caching layer)
Factories:
  - RepositoryFactory
Configuration:
  - PersistenceProperties
  - PersistenceConfig
```

---

### **STEP 6: Sketch Views and Record Decisions** 📐

**Principais Decisões Arquiteturais:**

| ID | Decisão | Rationale | Trade-offs |
|----|---------|-----------|----------|
| D1 | Usar Strategy Pattern para ID generation | Permite configurabilidade sem if/else hardcoded | Complexidade adicional inicial |
| D2 | Factory baseada em Properties | Centraliza lógica de criação, facilita testes | Mais classes criadas |
| D3 | Interfaces para todas as estratégias | Open/Closed Principle, extensibilidade | Overhead de abstrações |
| D4 | Redis como cache universal | Performance, reduz carga no DB primário | Dependência adicional |
| D5 | Composite para combinar ISBN providers | Fallback automático, resiliência | Latência aumentada |

---

### **STEP 7: Perform Analysis and Review** ✅

**Verificação dos Quality Attributes:**

| Quality Attribute | Como é atendido | Evidência |
|------------------|----------------|-----------|
| **Configurabilidade** | ✅ Trocar estratégia via properties | Nenhum código recompilado, apenas config alterado |
| **Extensibilidade** | ✅ Adicionar nova estratégia sem modificar existentes | Criar nova classe que implementa interface |
| **Modifiabilidade** | ✅ Substituir backend de persistência | Alterar `persistence.strategy` em properties |
| **Testabilidade** | ✅ Testar cada estratégia isoladamente | Mocks facilmente injetados via DI |
| **Interoperabilidade** | ✅ Integrar com múltiplas APIs | Adapter pattern, HTTP clients configuráveis |
| **Reliability** | ✅ Composite ISBN lookup com fallback | Se API 1 falha, tenta API 2 |

---

## 🔁 Iterative Design (Step 8: Refine as Necessary)

Processo iterativo conforme ADD:
1. **Iteração 1**: ID Generation (Configurabilidade) → **Validar** com testes
2. **Iteração 2**: ISBN Lookup (Extensibilidade) → **Refinar** baseado em feedback
3. **Iteração 3**: Persistence (Modifiabilidade) → **Revisar** trade-offs

---

## 📊 Mapeamento: Táticas ↔ Quality Attributes ↔ Implementação
```
CONFIGURABILIDADE (Critical)
├─ Configuration-Driven Behavior
│  └─ application.properties + @ConfigurationProperties
├─ Dynamic Binding
│  └─ Spring DI + Factories
└─ Use of Abstractions
   └─ IdGenerator, IsbnLookupService, Repository interfaces

EXTENSIBILIDADE (High)
├─ Use of Plugins/Extensions
│  └─ Strategy Pattern para cada subsistema
├─ Encapsulation
│  └─ Factories encapsulam lógica de criação
└─ Separation of Concerns
   └─ Model / Repository / Service / Config layers

MODIFIABILIDADE (Critical)
├─ Use of Interfaces
│  └─ Contratos bem definidos
├─ Reduce Coupling
│  └─ Depend on abstractions, not concretions
└─ Abstract Common Services
   └─ IdGenerator reusável

TESTABILIDADE (High)
├─ Dependency Injection
│  └─ Constructor injection
├─ Isolation
│  └─ Cada estratégia independente
└─ Mocking
   └─ Interfaces facilitam Mockito


## Step 1 files:

### Interface e Base:

- IdGeneratorProperties.java
- IdGeneratorConfig.java
- IdGeneratorInitializer.java

### Interface e Implementações:

- IdGenerator.java (interface)
- ConfigurableIdGenerator.java
- LegacyIdGenerator.java
- IdGeneratorFactory.java

### Modelos Refatorados:

- LendingNumber_Updated.java
- ReaderNumber_Updated.java

## Step 2 files:

### Interface e Base:

- IsbnLookupService.java (interface)
- AbstractIsbnLookupService.java
- IsbnLookupException.java

### Providers (APIs Externas):

- GoogleBooksLookupService.java
- OpenLibraryLookupService.java
- IsbnDbLookupService.java

### Composite (Fallback Automático):

- CompositeIsbnLookupService.java

### Configuration:

- IsbnLookupProperties.java
- IsbnLookupFactory.java
- IsbnLookupConfig.java

### Integração com BookService:

- BookServiceImpl_Updated.java

## Step 3 files:

- PersistenceProperties.java
- CacheService.java
- RedisCacheService.java
- CachedBookRepository.java
- MongoBookRepository.java
- RedisConfig.java
- PersistenceConfig.java