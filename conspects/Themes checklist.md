# База

- [ ] **Core Java**
    - [ ] ⚪ Синтаксис, ООП – классы, наследование, интерфейсы, абстракция, инкапсуляция
    - [x] 🟡 Collections Framework – List, Set, Map, очереди, их внутренняя реализация и сложность операций
    - [x] 🟡 Generics, wildcard’ы, bounded types
    - [x] 🟡 Java Memory Model – heap, stack, GC (алгоритмы G1, ZGC, Shenandoah)
    - [x] 🟡 Многопоточность и конкурентность – Thread, Runnable, Executor, CompletableFuture, ForkJoinPool, volatile, synchronized, Lock, атомарные операции
    - [x] 🟡 Streams API & Lambda expressions
    - [ ] ⚪ Java 17–21+ features – records, sealed classes, pattern matching, switch expressions, virtual threads (Project Loom)
    - [x] 🟡 JVM internals – class loading, JIT, bytecode basics

- [ ] **Frameworks & Libraries**
    - [x] 🟡 Spring Core, Spring Boot – DI, IoC, AOP, профили
    - [ ] 🟡 Spring Data JPA – работа с БД, кастомные репозитории, JPQL/Criteria API
    - [x] 🟡 Spring Security – JWT, OAuth2, role-based access
    - [ ] 🟡 Spring MVC / WebFlux – REST API, валидация, фильтры, interceptors
    - [ ] 🟢 Spring Cloud – Config, Gateway, Eureka/Consul, Feign, Resilience4j
    - [ ] 🟡 Hibernate – lazy/eager loading, кэш, N+1, оптимизация запросов
    - [ ] ⚪ Popular libraries – MapStruct, Lombok, Jackson, Mockito, Testcontainers

- [ ] **Databases & Persistence**
    - [ ] 🟡 SQL – сложные SELECT, JOIN, GROUP BY, оконные функции
    - [ ] 🟡 PostgreSQL / MySQL – индексы, транзакции, уровни изоляции, explain/analyze
    - [ ] ⚪ NoSQL – MongoDB, Redis (для кэша, очередей)
    - [ ] ⚪ Flyway/Liquibase – миграции
    - [ ] 🟡 Performance tuning – индексы, батчи, connection pool (HikariCP)

- [ ] **Testing & QA**
    - [ ] 🟡 JUnit 5, Mockito – юнит-тесты, мокирование, параметризованные тесты
    - [ ] ⚪ Testcontainers – интеграционные тесты с Postgres, Kafka и т.д.
    - [ ] 🟡 Spring Boot Test – MockMvc, WebTestClient
    - [ ] ⚪ Contract testing – WireMock, Pact
    - [ ] 🟡 CI/CD тестирование – coverage, quality gates (JaCoCo, SonarQube)

- [ ] **Architecture & Design**
    - [x] 🟡 SOLID, DRY, KISS, YAGNI – уметь применять
    - [ ] 🟡 Design Patterns – Singleton, Factory, Builder, Strategy, Observer, Proxy
    - [ ] ⚪ Enterprise patterns – Repository, Unit of Work, CQRS, Event Sourcing
    - [ ] 🟡 REST API design – best practices, idempotency, статус-коды
    - [ ] ⚪ GraphQL/gRPC – базовое знание
    - [x] ⚪ DDD, Hexagonal architecture – базовый уровень

- [ ] **DevOps & Infrastructure**
    - [x] 🟡 Docker, Docker Compose – контейнеризация сервисов
    - [ ] ⚪ Monitoring – Prometheus + Grafana, ELK stack
    - [ ] ⚪ Cloud basics – AWS/GCP/Azure (S3, EC2, IAM, RDS)

- [ ] **Messaging & Async**
    - [ ] 🟡 Kafka / RabbitMQ – очереди, топики, consumer groups
    - [ ] ⚪ Event-driven architecture – pub/sub, at-least-once vs exactly-once
    - [ ] ⚪ Reactive programming – Project Reactor, WebFlux (понимать, когда применять)

- [ ] **Tools & Workflow**
    - [ ] 🟡 Git – rebase, cherry-pick, merge vs squash
    - [ ] ⚪ Maven/Gradle – депенденси-менеджмент, multi-module
    - [ ] ⚪ Agile/Scrum – понимание процессов

# Финтех


- [ ] **Интеграции / Архитектура**
	- [ ] REST vs gRPC
	- [ ] Circuit Breaker, Retry, Timeout
	- [ ] API Gateway
	- [ ] Event-driven архитектура
	- [ ] Микросервисы vs монолит

- [ ] **Безопасность**
	- [ ] Симметричное и асимметричное шифрование (AES, RSA)
	- [ ] Подписи (HMAC, JWT)
	- [ ] OWASP Top 10 
	    - [ ] SQL Injection
	    - [ ] XSS
	    - [ ] CSRF
	- [ ] Хранение паролей (bcrypt, Argon2)

- [ ] **DevOps / Инфраструктура**
	- [ ] Docker 
	    - [ ] Образы
	    - [ ] Контейнеры
	    - [ ] Volume
	    - [ ] Network
	- [ ] CI/CD
	    - [ ] Build
	    - [ ] Test
	    - [ ] Deploy
	- [ ] Kubernetes
	    - [ ] Pod
	    - [ ] Service
	    - [ ] Deployment (базовое понимание)
	- [ ] Мониторинг
	    - [ ] Prometheus
	    - [ ] Grafana

- [ ] **Финтех-специфика**
	- [ ] Идемпотентность (requestId, retry)
	- [ ] Двойное списание и защита
	- [ ] Workflow транзакций
	    - [ ] Авторизация
	    - [ ] Клиринг
	    - [ ] Сеттлмент
	- [ ] Сага-паттерн
	- [ ] Outbox-паттерн
	- [ ] Очереди
	    - [ ] Kafka
	    - [ ] RabbitMQ
	- [ ] Eventual consistency
	- [ ] 2-phase commit