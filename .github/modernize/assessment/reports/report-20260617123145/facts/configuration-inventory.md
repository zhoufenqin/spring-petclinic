# Configuration & Externalized Settings Inventory

Spring PetClinic uses 4 `application*.properties` files as its configuration sources, 2 Maven build profiles, and 2 Spring runtime profiles (`mysql`, `postgres`), with secrets supplied exclusively via environment variables and no external config server or secret store.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|--------|------|--------------|-------|
| Default application properties | Spring `application.properties` | `src/main/resources/application.properties` | Active for all profiles; sets database, JPA, Thymeleaf, Actuator, and logging defaults |
| MySQL profile properties | Spring `application-mysql.properties` | `src/main/resources/application-mysql.properties` | Activated by `spring.profiles.active=mysql`; overrides datasource URL and init mode |
| PostgreSQL profile properties | Spring `application-postgres.properties` | `src/main/resources/application-postgres.properties` | Activated by `spring.profiles.active=postgres`; overrides datasource URL and init mode |
| H2 SQL init scripts | SQL scripts | `src/main/resources/db/h2/schema.sql`, `db/h2/data.sql` | Applied by default profile; H2-specific DDL |
| MySQL SQL init scripts | SQL scripts | `src/main/resources/db/mysql/schema.sql`, `db/mysql/data.sql` | Applied by mysql profile |
| PostgreSQL SQL init scripts | SQL scripts | `src/main/resources/db/postgres/schema.sql`, `db/postgres/data.sql` | Applied by postgres profile |
| Docker Compose environment | `docker-compose.yml` env section | `docker-compose.yml` | Injects `SERVER_PORT`, `MYSQL_URL` into `petclinic` service; `MYSQL_ROOT_PASSWORD`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_DATABASE` into `mysqlserver` |
| i18n messages | Properties bundles | `src/main/resources/messages/messages*.properties` | UI label translations (EN, DE, ES) |

No Spring Cloud Config Server, HashiCorp Vault, AWS AppConfig, Consul KV, or Azure App Configuration is used. All externalized settings come from OS environment variables or Docker Compose environment sections.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies / Plugins Added |
|---------|-----------|---------|----------------------------------|
| `css` (default inactive) | Manual: `-P css` | Recompiles SCSS source in `src/main/scss/` into `src/main/resources/static/resources/css/` | Adds `maven-dependency-plugin` (Bootstrap SCSS unpack) and `libsass-maven-plugin 0.2.26` (SCSS compilation) |
| `m2e` | Auto: activated when Eclipse m2e property `m2e.version` is set | Suppresses Eclipse m2e lifecycle-mapping warnings for `maven-checkstyle-plugin`, `spring-boot-maven-plugin`, and `spring-javaformat-maven-plugin` | Adds `lifecycle-mapping` plugin management (no additional dependencies; configuration only) |

## Runtime Profiles

| Profile | Activation Method | Config Files Loaded | Key Overrides |
|---------|------------------|---------------------|---------------|
| default (no profile) | No `spring.profiles.active` set | `application.properties` + `db/h2/schema.sql` + `db/h2/data.sql` | Uses H2 in-memory database; `spring.sql.init.schema-locations=classpath*:db/h2/schema.sql` |
| `mysql` | `spring.profiles.active=mysql` (or `-Dspring-boot.run.profiles=mysql`) | `application.properties` + `application-mysql.properties` + `db/mysql/schema.sql` + `db/mysql/data.sql` | Overrides `database=mysql`, datasource URL/credentials, `spring.sql.init.mode=always` |
| `postgres` | `spring.profiles.active=postgres` | `application.properties` + `application-postgres.properties` + `db/postgres/schema.sql` + `db/postgres/data.sql` | Overrides `database=postgres`, datasource URL/credentials, `spring.sql.init.mode=always` |

Multiple profiles can be combined (e.g., `mysql,key-vault`), but no additional profile combinations are defined in the codebase.

## Properties Inventory

### Default (`application.properties`)

| Property Key | Default Value | Profiles | Source |
|-------------|--------------|---------|--------|
| `database` | `h2` | default | `application.properties` |
| `spring.sql.init.schema-locations` | `classpath*:db/${database}/schema.sql` | default | `application.properties` |
| `spring.sql.init.data-locations` | `classpath*:db/${database}/data.sql` | default | `application.properties` |
| `spring.thymeleaf.mode` | `HTML` | all | `application.properties` |
| `spring.jpa.hibernate.ddl-auto` | `none` | all | `application.properties` |
| `spring.jpa.open-in-view` | `true` | all | `application.properties` |
| `spring.messages.basename` | `messages/messages` | all | `application.properties` |
| `management.endpoints.web.exposure.include` | `*` | all | `application.properties` |
| `logging.level.org.springframework` | `INFO` | all | `application.properties` |
| `spring.web.resources.cache.cachecontrol.max-age` | `12h` | all | `application.properties` |

### MySQL Profile (`application-mysql.properties`)

| Property Key | Value | Profile | Source |
|-------------|-------|---------|--------|
| `database` | `mysql` | `mysql` | `application-mysql.properties` |
| `spring.datasource.url` | `${MYSQL_URL:jdbc:mysql://localhost/petclinic}` | `mysql` | Env var `MYSQL_URL` with fallback |
| `spring.datasource.username` | `${MYSQL_USER:petclinic}` | `mysql` | Env var `MYSQL_USER` with fallback |
| `spring.datasource.password` | `${MYSQL_PASS:petclinic}` | `mysql` | Env var `MYSQL_PASS` with fallback |
| `spring.sql.init.mode` | `always` | `mysql` | `application-mysql.properties` |

### PostgreSQL Profile (`application-postgres.properties`)

| Property Key | Value | Profile | Source |
|-------------|-------|---------|--------|
| `database` | `postgres` | `postgres` | `application-postgres.properties` |
| `spring.datasource.url` | `${POSTGRES_URL:jdbc:postgresql://localhost/petclinic}` | `postgres` | Env var `POSTGRES_URL` with fallback |
| `spring.datasource.username` | `${POSTGRES_USER:petclinic}` | `postgres` | Env var `POSTGRES_USER` with fallback |
| `spring.datasource.password` | `${POSTGRES_PASS:petclinic}` | `postgres` | Env var `POSTGRES_PASS` with fallback |
| `spring.sql.init.mode` | `always` | `postgres` | `application-postgres.properties` |

### Docker Compose Environment Variables

| Variable | Service | Value in Compose | Notes |
|----------|---------|-----------------|-------|
| `SERVER_PORT` | `petclinic` | `8080` | Maps Spring `server.port` |
| `MYSQL_URL` | `petclinic` | `jdbc:mysql://mysqlserver/petclinic` | Overrides default localhost URL |
| `MYSQL_ROOT_PASSWORD` | `mysqlserver` | (empty) | Allows empty root password for local dev |
| `MYSQL_ALLOW_EMPTY_PASSWORD` | `mysqlserver` | `true` | Dev-only |
| `MYSQL_USER` | `mysqlserver` | `petclinic` | DB user created at startup |
| `MYSQL_PASSWORD` | `mysqlserver` | `petclinic` | Password for above user |
| `MYSQL_DATABASE` | `mysqlserver` | `petclinic` | Schema name created at startup |

## Startup Parameters & Resource Requirements

| Service | JVM / Runtime Options | Memory Limits | Instance Count | Notes |
|---------|----------------------|--------------|----------------|-------|
| `petclinic` (development, Compose) | `-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000` (JDWP debug agent) | Not configured | 1 | `Dockerfile.multi` `development` stage runs `mvnw spring-boot:run -Dspring-boot.run.profiles=mysql` |
| `petclinic` (production, `Dockerfile`) | `-Djava.security.egd=file:/dev/./urandom` | Not configured | 1 | Uses `eclipse-temurin:17-jre-jammy`; fat JAR exec |
| `mysqlserver` (Compose) | N/A (external image) | Not configured | 1 | `mysql:8` official image |

No `-Xms`/`-Xmx` heap settings, Kubernetes resource requests/limits, or cloud deployment size configurations are present.

## Startup Dependency Chain

```
mysqlserver (MySQL 8)
    ↑ depends_on (Docker Compose)
petclinic
```

The `docker-compose.yml` specifies `depends_on: mysqlserver` for the `petclinic` service, meaning Compose starts MySQL before the application container. However, no `healthcheck` + `condition: service_healthy` is configured, so the application may attempt to connect before MySQL is fully ready. No `dockerize`, Kubernetes readiness probes, or Spring Cloud Config retry mechanisms are present.

Spring Boot Actuator is available at `/actuator/health` as a readiness indicator once the application starts, but no external orchestration uses it.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|-----------------|------|---------|
| `${MYSQL_URL:jdbc:mysql://localhost/petclinic}` | Database JDBC URL | Environment variable `MYSQL_URL`; fallback to plaintext default in properties file |
| `${MYSQL_USER:petclinic}` | Database username | Environment variable `MYSQL_USER`; fallback to plaintext `petclinic` in properties file |
| `${MYSQL_PASS:petclinic}` | Database password | Environment variable `MYSQL_PASS`; fallback to plaintext `petclinic` in properties file |
| `${POSTGRES_URL:jdbc:postgresql://localhost/petclinic}` | Database JDBC URL | Environment variable `POSTGRES_URL`; fallback to plaintext default |
| `${POSTGRES_USER:petclinic}` | Database username | Environment variable `POSTGRES_USER`; fallback to plaintext `petclinic` |
| `${POSTGRES_PASS:petclinic}` | Database password | Environment variable `POSTGRES_PASS`; fallback to plaintext `petclinic` |
| `MYSQL_ROOT_PASSWORD` (Compose) | MySQL root password | Docker Compose env; set to empty string (dev only) |

No encryption (Jasypt, DPAPI, sealed secrets), no external secret stores (Vault, KeyVault, AWS Secrets Manager), and no Kubernetes Secrets are used.

### Secrets Provisioning Workflow

Secrets are passed as OS-level environment variables at process startup. In Docker Compose (local development), credentials are defined in plain text in `docker-compose.yml` with default values acceptable for local dev. In non-Compose deployments, the operator must inject `MYSQL_URL`, `MYSQL_USER`, `MYSQL_PASS` (or Postgres equivalents) into the container runtime environment before startup.

There is no managed-identity, service-principal, or RBAC-based secrets model. The plaintext fallback values in `application-mysql.properties` (`petclinic`/`petclinic`) mean the application starts with default credentials if environment variables are not set, which is a security risk in non-development environments.

## Feature Flags

| Flag Name / Conditional Bean | Default | Controlled By | Notes |
|------------------------------|---------|--------------|-------|
| `@EnableCaching` in `CacheConfiguration` | Enabled (always) | Hardcoded in `@Configuration` class | Activates Spring Cache abstraction; no conditional property toggle |
| Spring Boot DevTools auto-restart | Enabled in dev classpath | Presence of `spring-boot-devtools` on classpath | `optional=true` in `pom.xml`; excluded from fat-JAR packaging automatically |

No LaunchDarkly, Unleash, Spring Feature Flags, or `@ConditionalOnProperty` feature toggles are present. Caching and DevTools are the only conditionally-active capabilities.

## Framework & Runtime Versions

| Component | Version | Source |
|-----------|---------|--------|
| Spring Boot | 2.7.1 | `pom.xml` parent |
| Spring Framework | 5.3.x (managed by Boot BOM) | Spring Boot BOM |
| Java (source/target compatibility) | 1.8 | `pom.xml` `<java.version>1.8</java.version>` |
| Docker base image (dev) | `eclipse-temurin:17-jdk-jammy` | `Dockerfile`, `Dockerfile.multi` |
| Docker base image (production) | `eclipse-temurin:17-jre-jammy` | `Dockerfile.multi` production stage |
| Hibernate ORM | 5.6.x (managed by Boot BOM) | Spring Data JPA starter |
| Thymeleaf | 3.0.x (managed by Boot BOM) | Spring Boot Starter Thymeleaf |
| Ehcache | 3.10.x (managed by Boot BOM) | `org.ehcache:ehcache` |
| JCache API (JSR-107) | 1.1.1 (managed by Boot BOM) | `javax.cache:cache-api` |
| Micrometer | 1.9.x (managed by Boot BOM) | Spring Boot Actuator |
| H2 | 2.1.x (managed by Boot BOM) | `com.h2database:h2` |
| MySQL Connector/J | 8.0.x (managed by Boot BOM) | `mysql:mysql-connector-java` |
| PostgreSQL JDBC | 42.x (managed by Boot BOM) | `org.postgresql:postgresql` |
| Bootstrap | 5.1.3 | `pom.xml` `<webjars-bootstrap.version>` |
| Font Awesome | 4.7.0 | `pom.xml` `<webjars-font-awesome.version>` |
| Maven | 3.x (wrapper) | `mvnw` / `.mvn/wrapper/` |
| JaCoCo | 0.8.7 | `pom.xml` `<jacoco.version>` |
| Spring Java Format | 0.0.31 | `pom.xml` `<spring-format.version>` |
| nohttp-checkstyle | 0.0.10 | `pom.xml` `<nohttp-checkstyle.version>` |
| MySQL (Docker) | 8 | `docker-compose.yml` `image: mysql:8` |

> Note: There is a version mismatch between the source-compatibility target (`java.version=1.8`) and the Docker base image (`eclipse-temurin:17`). The application compiles and runs on Java 17 in containers but is restricted to Java 8 language features and APIs by the Maven compiler settings.
