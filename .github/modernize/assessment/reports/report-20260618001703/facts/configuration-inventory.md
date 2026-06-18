# Configuration & Externalized Settings Inventory

The repository uses a small set of Spring property files, SQL seed scripts, and optional container manifests to externalize runtime behavior. Profile-driven datasource settings are the primary variation point, and secrets are supplied through environment variables or sample inline compose values rather than a managed secret store.

## Configuration Sources

| Source | Type | Path/Location | Notes |
| --- | --- | --- | --- |
| Spring application defaults | Properties file | `src/main/resources/application.properties` | Base datasource selector, JPA, Thymeleaf, logging, actuator, and static-cache settings |
| Spring profile overrides | Properties file | `src/main/resources/application-mysql.properties` | Activates MySQL datasource URL, credentials, and SQL init mode |
| Spring profile overrides | Properties file | `src/main/resources/application-postgres.properties` | Activates PostgreSQL datasource URL, credentials, and SQL init mode |
| SQL schema scripts | SQL files | `src/main/resources/db/*/schema.sql` | Defines schema per supported database flavor |
| SQL seed scripts | SQL files | `src/main/resources/db/*/data.sql` | Seeds reference and sample domain data |
| Docker Compose | YAML | `docker-compose.yml` | Starts the app with a MySQL sidecar and environment overrides |
| Alternate Compose for k8s image | YAML | `k8s/compose/docker-compose-k8s.yml` | Uses a prebuilt image but the same environment contract |
| Container build files | Dockerfile | `Dockerfile`, `Dockerfile.multi` | Define development and production runtime commands |
| Maven wrapper | Properties file | `.mvn/wrapper/maven-wrapper.properties` | Pins Maven distribution for repeatable builds |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
| --- | --- | --- | --- |
| default Maven build | Automatic | Runs validation, tests, packaging, and build metadata generation | spring-javaformat, nohttp checkstyle, Spring Boot, JaCoCo, git-commit-id plugins |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
| --- | --- | --- | --- |
| default | No explicit Spring profile | `application.properties` | Uses embedded H2 schema/data scripts and base web/JPA/actuator settings |
| mysql | `spring.profiles.active=mysql` or Docker development command | `application.properties`, `application-mysql.properties` | Overrides datasource URL, username, password, database selector, and forces SQL init |
| postgres | `spring.profiles.active=postgres` | `application.properties`, `application-postgres.properties` | Overrides datasource URL, username, password, database selector, and forces SQL init |

## Properties Inventory

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `database` | `h2` | default; overridden by `mysql` and `postgres` | `application.properties`, profile files |
| `spring.sql.init.schema-locations` | `classpath*:db/${database}/schema.sql` | all | `application.properties` |
| `spring.sql.init.data-locations` | `classpath*:db/${database}/data.sql` | all | `application.properties` |
| `spring.thymeleaf.mode` | `HTML` | all | `application.properties` |
| `spring.jpa.hibernate.ddl-auto` | `none` | all | `application.properties` |
| `spring.jpa.open-in-view` | `true` | all | `application.properties` |
| `spring.messages.basename` | `messages/messages` | all | `application.properties` |
| `management.endpoints.web.exposure.include` | `*` | all | `application.properties` |
| `logging.level.org.springframework` | `INFO` | all | `application.properties` |
| `spring.web.resources.cache.cachecontrol.max-age` | `12h` | all | `application.properties` |
| `spring.datasource.url` | environment-backed | mysql, postgres | `application-mysql.properties`, `application-postgres.properties` |
| `spring.datasource.username` | environment-backed | mysql, postgres | `application-mysql.properties`, `application-postgres.properties` |
| `spring.datasource.password` | environment-backed | mysql, postgres | `application-mysql.properties`, `application-postgres.properties` |
| `spring.sql.init.mode` | `always` | mysql, postgres | profile files |
| `SERVER_PORT` | `8080` in compose | compose deployment | `docker-compose.yml` |
| `MYSQL_URL` | `jdbc:mysql://mysqlserver/petclinic` in compose | compose deployment | `docker-compose.yml` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
| --- | --- | --- | --- |
| spring-petclinic local | none committed in source | Not specified | 1 process expected |
| spring-petclinic Docker development | `-Dspring-boot.run.profiles=mysql` and remote debug `-agentlib:jdwp=...address=*:8000` | Not specified | 1 container expected |
| spring-petclinic Docker production | `-Djava.security.egd=file:/dev/./urandom -jar /spring-petclinic.jar` | Not specified | 1 container expected |
| mysqlserver compose service | MySQL image defaults | Not specified | 1 container expected |

## Startup Dependency Chain

1. `mysqlserver` starts before `petclinic` in `docker-compose.yml` via `depends_on`, but no explicit health check or wait-for-ready mechanism is configured.
2. `petclinic` starts independently in local default mode because the embedded H2 datasource is auto-configured from the classpath.
3. In Docker development mode, the app process runs with the `mysql` Spring profile and relies on the MySQL hostname provided by Compose networking.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
| --- | --- | --- |
| `spring.datasource.password` | Database password | `${MYSQL_PASS:[MASKED]}` or `${POSTGRES_PASS:[MASKED]}` in profile files |
| `spring.datasource.username` | Database username | `${MYSQL_USER:[MASKED]}` or `${POSTGRES_USER:[MASKED]}` in profile files |
| `MYSQL_PASSWORD` | Compose database password | Inline sample value `[MASKED]` |
| `MYSQL_ROOT_PASSWORD` | Compose root password | Empty sample value `[MASKED]` |

### Secrets Provisioning Workflow

Secrets are supplied directly through environment variables or inline Docker Compose variables. There is no Key Vault, Vault, encrypted property source, managed identity, or RBAC-based secret retrieval flow in the repository. In practice, the application expects database credentials to be injected into the process environment before startup, and the sample Compose setup wires those values directly into the MySQL container and Spring datasource properties.

## Feature Flags

| Flag Name | Default | Controlled By |
| --- | --- | --- |
| None detected | n/a | n/a |

## Framework & Runtime Versions

| Component | Version | Source |
| --- | --- | --- |
| Spring Boot parent | 2.7.1 | `pom.xml` |
| Java target | 1.8 | `pom.xml` |
| Maven wrapper | 3.8.2 | `.mvn/wrapper/maven-wrapper.properties` |
| Bootstrap WebJar | 5.1.3 | `pom.xml` |
| Font Awesome WebJar | 4.7.0 | `pom.xml` |
| JaCoCo plugin | 0.8.7 | `pom.xml` |
| Spring Java Format plugin | 0.0.31 | `pom.xml` |
| Runtime container base image | eclipse-temurin 17 jdk and jre jammy | `Dockerfile`, `Dockerfile.multi` |
| Compose database image | mysql 8 | `docker-compose.yml` |
