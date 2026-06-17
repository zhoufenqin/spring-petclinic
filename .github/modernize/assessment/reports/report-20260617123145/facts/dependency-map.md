# Dependency Map

Spring PetClinic (v2.7.0-SNAPSHOT) declares 15 runtime/compile-scope dependencies managed under the Spring Boot 2.7.1 parent BOM, plus a test bundle and a set of build plugins.

## Dependencies

```mermaid
flowchart LR
    App["spring-petclinic v2.7.0"]

    subgraph BOM["Parent BOM"]
        BootBOM["spring-boot-starter-parent v2.7.1"]
    end

    subgraph Web["Web Frameworks"]
        SpringWeb["Spring Boot Starter Web 2.7.1"]
        Thymeleaf["Spring Boot Starter Thymeleaf 2.7.1"]
        Bootstrap["Bootstrap WebJar 5.1.3"]
        FontAwesome["Font-Awesome WebJar 4.7.0"]
        WebjarsLocator["Webjars Locator Core 0.50"]
    end

    subgraph DB["Database / ORM"]
        DataJPA["Spring Boot Starter Data JPA 2.7.1"]
        H2["H2 Database 2.1.x (runtime)"]
        MySQLDriver["MySQL Connector/J 8.0.x (runtime)"]
        PGDriver["PostgreSQL JDBC 42.x (runtime)"]
    end

    subgraph Cache["Caching"]
        BootCache["Spring Boot Starter Cache 2.7.1"]
        JCacheAPI["JCache API 1.1.1"]
        Ehcache["Ehcache 3.10.x"]
    end

    subgraph Observability["Observability"]
        Actuator["Spring Boot Starter Actuator 2.7.1"]
    end

    subgraph Validation["Validation"]
        Validation["Spring Boot Starter Validation 2.7.1"]
    end

    subgraph DevTools["Developer Tools"]
        DevToolsLib["Spring Boot DevTools 2.7.1 (optional)"]
    end

    App -.->|"managed by"| BootBOM
    App -->|"web"| Web
    App -->|"persistence"| DB
    App -->|"caching"| Cache
    App -->|"observability"| Observability
    App -->|"validation"| Validation
    App -->|"dev"| DevTools
    BootBOM -.->|"governs versions"| Web
    BootBOM -.->|"governs versions"| DB
    BootBOM -.->|"governs versions"| Cache
    BootBOM -.->|"governs versions"| Observability
    BootBOM -.->|"governs versions"| Validation
```

### Dependency Summary

| Category | Count | Key Libraries | Notes |
|----------|-------|--------------|-------|
| Web Frameworks | 5 | Spring Boot Starter Web 2.7.1, Thymeleaf 2.7.1, Bootstrap 5.1.3 | Legacy Spring Boot 2.x; Thymeleaf 3.0 under the hood |
| Database / ORM | 4 | Spring Data JPA 2.7.1, H2 2.1.x, MySQL Connector 8.0.x, PostgreSQL 42.x | Three database drivers bundled; only one active per profile |
| Caching | 3 | Spring Boot Starter Cache, JCache API 1.1.1, Ehcache 3.10.x | JSR-107 caching with Ehcache 3 as implementation |
| Observability | 1 | Spring Boot Actuator 2.7.1 | All endpoints exposed (`management.endpoints.web.exposure.include=*`) |
| Validation | 1 | Spring Boot Starter Validation 2.7.1 | Bean Validation 2.0 (Hibernate Validator under the hood) |
| Developer Tools | 1 | Spring Boot DevTools 2.7.1 | Optional dependency; automatic restarts in development |

### Version & Compatibility Risks

Spring Boot 2.7.1 is based on Java 8 (as declared in `pom.xml`), which reached end-of-life in 2019 for Oracle JDK and requires a paid support subscription. Spring Boot 2.7.x itself reached end-of-life in November 2023, meaning no further community security patches are available. The `mysql-connector-java` group ID changed to `com.mysql` in 8.0.31; the project still uses the deprecated `mysql:mysql-connector-java` artifact. All core Spring dependency versions are managed by the Spring Boot parent BOM, so they are internally consistent, but the entire stack should be upgraded to Spring Boot 3.x (Java 17+) to receive continued support.

### Notable Observations

- **All three database drivers are bundled at runtime**: H2, MySQL Connector/J, and the PostgreSQL driver are all on the classpath simultaneously. Only one is active per Spring profile, but shipping unused drivers increases image size and attack surface.
- **Actuator fully open**: `management.endpoints.web.exposure.include=*` exposes every Actuator endpoint (including `/actuator/env` and `/actuator/heapdump`) without authentication, which is a security concern in non-development environments.
- **Java source compatibility set to 1.8**: `<java.version>1.8</java.version>` targets Java 8, blocking use of modern language features (records, sealed classes, text blocks) available in Java 17+.
- **No security framework declared**: The application has no Spring Security or equivalent dependency, meaning all HTTP endpoints (including Actuator) are unauthenticated.

## Test Dependencies

| Framework | Version | Notes |
|-----------|---------|-------|
| Spring Boot Starter Test | 2.7.1 | Meta-dependency pulling in JUnit 5, Mockito, AssertJ, Spring Test, Hamcrest, JSONassert, JsonPath |
| JUnit 5 (Jupiter) | 5.8.x (via Boot BOM) | Primary test runner |
| Mockito | 4.5.x (via Boot BOM) | Mocking framework |
| AssertJ | 3.22.x (via Boot BOM) | Fluent assertion library |
| Spring Test | 5.3.x (via Boot BOM) | Integration test support (`@SpringBootTest`, `MockMvc`) |
| Hamcrest | 2.2 (via Boot BOM) | Matcher-based assertions |

Total test-scope dependencies: 6 (declared as 1 starter, resolving to ~6 frameworks via transitive BOM management)

The test suite uses `@SpringBootTest` for integration tests and `@WebMvcTest` for slice tests, giving good coverage of controller and JPA layers. No Testcontainers dependency is present, so MySQL and PostgreSQL database profiles are not exercised in automated tests — only H2 is tested via the default profile.
