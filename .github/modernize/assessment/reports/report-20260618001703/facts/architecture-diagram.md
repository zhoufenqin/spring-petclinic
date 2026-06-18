# Architecture Diagram

This Spring PetClinic repository is a single deployable Spring Boot web application. Its runtime architecture centers on Spring MVC request handling, a JPA-backed persistence layer, and a small cache for veterinarian lookups.

## Application Architecture

```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        Browser["Web Browser"]
    end
    subgraph App["Application Layer - Spring Boot 2.7.1"]
        Web["Spring MVC Controllers"]
        Views["Thymeleaf Views"]
        Domain["Owner Pet Visit and Vet Domain Logic"]
    end
    subgraph Data["Data Layer"]
        Repo["Spring Data Repository Interfaces"]
        Cache["JCache vets cache"]
        DB[("Relational Database")]
    end
    subgraph Profiles["Profile Specific Datastores"]
        H2[("Embedded H2 default")]
        MySQL[("MySQL profile")]
        Postgres[("PostgreSQL profile")]
    end

    Browser -->|"HTTP requests"| Web
    Web -->|"render model"| Views
    Web -->|"load and mutate aggregates"| Domain
    Domain -->|"query and persist"| Repo
    Repo -->|"cache read heavy vet queries"| Cache
    Repo -->|"SQL via JPA"| DB
    DB -->|"default profile"| H2
    DB -->|"mysql profile"| MySQL
    DB -->|"postgres profile"| Postgres
```

### Technology Stack Summary

| Layer | Technology | Version | Purpose |
| --- | --- | --- | --- |
| Presentation | Spring MVC + Thymeleaf | Spring Boot 2.7.1 managed | Server-side HTML forms and page rendering |
| Application | Spring Boot | 2.7.1 | Bootstraps the monolithic application and autoconfiguration |
| Domain | Java domain entities and validators | Java 8 target | Encodes owner, pet, visit, and vet business rules |
| Data Access | Spring Data repository interfaces + Hibernate/JPA | Spring Boot managed | Reads and writes relational data |
| Caching | Spring Cache + JCache + Ehcache | cache-api managed, Ehcache managed | Caches veterinarian queries |
| Operations | Spring Boot Actuator | Spring Boot 2.7.1 managed | Exposes management and metrics endpoints |

### Data Storage & External Services

The application persists all operational data in a single relational database schema. It defaults to an embedded H2 database during local execution, while `mysql` and `postgres` profiles switch the same schema and seed scripts to MySQL or PostgreSQL. No outbound API, message broker, or third-party SaaS integration is implemented in the source tree.

### Key Architectural Decisions

- Uses a classic server-rendered monolith: controllers, views, repositories, and entities are packaged together in one Spring Boot deployment unit.
- Reuses one domain model across HTML form binding, JSON responses, and persistence instead of introducing a separate service or DTO layer.
- Applies caching only to veterinarian lookups, which are read frequently and change rarely.

## Component Relationships

```mermaid
flowchart LR
    subgraph Presentation["Presentation"]
        WelcomeCtrl["WelcomeController"]
        OwnerCtrl["OwnerController"]
        PetCtrl["PetController"]
        VisitCtrl["VisitController"]
        VetCtrl["VetController"]
    end
    subgraph Business["Business Logic"]
        OwnerAgg["Owner Pet Visit Aggregate"]
        VetAgg["Vet Specialty Aggregate"]
        PetVal["PetValidator"]
    end
    subgraph DataAccess["Data Access"]
        OwnerRepo["OwnerRepository"]
        VetRepo["VetRepository"]
    end
    subgraph Infra["Infrastructure"]
        CacheConfig["CacheConfiguration"]
        JpaInfra["Spring Data JPA"]
        SqlDb["Relational Database"]
    end

    WelcomeCtrl -->|"serves landing page"| OwnerCtrl
    OwnerCtrl -->|"queries owners"| OwnerRepo
    PetCtrl -->|"loads owner and pet"| OwnerRepo
    VisitCtrl -->|"loads owner and pet"| OwnerRepo
    VetCtrl -->|"queries vets"| VetRepo
    PetCtrl -->|"validates forms"| PetVal
    OwnerRepo -->|"materializes"| OwnerAgg
    VetRepo -->|"materializes"| VetAgg
    OwnerAgg -->|"owns pets and visits"| PetVal
    OwnerRepo -->|"JPA queries"| JpaInfra
    VetRepo -->|"JPA queries"| JpaInfra
    CacheConfig -.->|"enables cache"| VetRepo
    JpaInfra -->|"SQL"| SqlDb
```

### Component Inventory

| Component | Layer | Type | Responsibility |
| --- | --- | --- | --- |
| WelcomeController | Presentation | MVC Controller | Serves the landing page |
| OwnerController | Presentation | MVC Controller | Handles owner search, create, update, and detail flows |
| PetController | Presentation | MVC Controller | Creates and edits pets under an owner |
| VisitController | Presentation | MVC Controller | Records visits for an existing pet |
| VetController | Presentation | MVC Controller and JSON endpoint | Lists veterinarians in HTML and JSON |
| PetValidator | Business Logic | Validator | Enforces required pet name, type, and birth date rules |
| Owner aggregate | Business Logic | Domain aggregate | Owns owner, pet, and visit relationships plus duplicate-name checks |
| Vet aggregate | Business Logic | Domain aggregate | Represents veterinarians and specialties |
| OwnerRepository | Data Access | Spring Data repository | Searches owners, pet types, and owner detail graphs |
| VetRepository | Data Access | Spring Data repository | Loads veterinarian directories with caching |
| CacheConfiguration | Infrastructure | Spring configuration | Creates the `vets` cache region and enables statistics |
| Spring Data JPA | Infrastructure | Persistence framework | Maps entities to the relational schema |
