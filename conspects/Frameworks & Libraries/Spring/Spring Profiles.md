## 1. Зачем нужны профили

В разных окружениях мы используем разные ресурсы и настройки:
- **База данных**: H2/SQLite в тестах, PostgreSQL/MySQL в проде.
- **Кэш**: локальный `ConcurrentMapCache` в dev, Redis в prod.
- **Интеграции**: тестовые/mock API в dev, реальные внешние API в prod.
- **Логирование**: `DEBUG` для dev, `INFO/WARN` для prod.

Профили позволяют переключать всё это без переписывания кода.

---

## 2. Как включить профиль

### Через `application.yml`

```yaml
spring:
  profiles:
    active: dev
```

### Через аргумент JVM

`java -jar app.jar --spring.profiles.active=prod`
или
`-Dspring.profiles.active=prod`

### Через переменные окружения (часто в Docker/Kubernetes)

`SPRING_PROFILES_ACTIVE=prod`

---

## 3. Аннотация `@Profile`

Можно создавать бины, которые загружаются только в определённом профиле:
```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Profile("dev")
    public DataSource h2DataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .build();
    }

    @Bean
    @Profile("prod")
    public DataSource postgresDataSource() {
        return DataSourceBuilder.create()
                .url("jdbc:postgresql://prod-db:5432/app")
                .username("user")
                .password("secret")
                .build();
    }
}
```

📌 В dev поднимется H2, в prod — PostgreSQL.

---

## 4. `application-{profile}.yml`

Spring автоматически подхватывает файлы с именем:

- `application-dev.yml`
- `application-prod.yml`
- `application-test.yml`

Например:

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
```
```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/app
    username: prod_user
    password: secret

```

Когда активен `dev`, Spring объединяет `application.yml` + `application-dev.yml`.

---

## 5. `spring.profiles.include`

Позволяет включить сразу несколько профилей:

```yaml
spring:
  profiles:
    active: dev
    include: common
```

В этом случае:
- грузятся настройки из `application-dev.yml`,
- плюс из `application-common.yml`.

Это удобно, если есть общий набор для всех окружений (например, `common` = настройки логирования).

---

## 6. Несколько профилей одновременно

Можно активировать несколько профилей через запятую:

`-Dspring.profiles.active=dev,local`

Тогда Spring соберёт конфигурацию из `application.yml` + `application-dev.yml` + `application-local.yml`.

---

## 7. Как проверять текущий профиль

В коде:
```java
@Autowired
private Environment env;

public void checkProfile() {
    System.out.println(Arrays.toString(env.getActiveProfiles()));
}
```
Или условно создавать бины:
```java
@Bean
@ConditionalOnProperty(name = "feature.enabled", havingValue = "true")
public SomeService featureService() {
    return new SomeService();
}
```
(это уже не `@Profile`, но часто используют вместе).

---

## 8. Использование в тестах

Для тестов можно указывать профиль через `@ActiveProfiles`:
```java
@SpringBootTest
@ActiveProfiles("test")
class UserServiceTest {
    ...
}
```
В тестах поднимется `application-test.yml`.

---

## 9. Практические кейсы

1. **База данных:**
    - `test` → H2 с автогенерацией схемы.
    - `dev` → PostgreSQL в Docker.
    - `prod` → PostgreSQL на реальном сервере.
    
2. **Интеграции:**
    - `dev` → моковые сервисы (WireMock).
    - `prod` → реальный API.
    
3. **Кэш:**
    - `dev` → no-op cache (для отладки).
    - `prod` → Redis.
    
4. **Безопасность:**
    - `dev` → отключённый OAuth2 или дефолтный юзер.
    - `prod` → полноценная авторизация.

---

## 10. Типичные вопросы на собесе

### 1. Что такое профили в Spring и зачем они нужны?

**Ответ:** Профили позволяют разделять конфигурацию для разных окружений (dev, test, prod). Это удобно для смены БД, логирования, интеграций, кэша и т.д.

---
### 2. Как активировать профиль?

**Ответ:**
- В `application.yml` → `spring.profiles.active=dev`.
- Через JVM аргумент → `-Dspring.profiles.active=dev`.
- Через переменную окружения → `SPRING_PROFILES_ACTIVE=dev`.

---

### 3. В чём разница между `spring.profiles.active` и `spring.profiles.include`?

**Ответ:**
- `spring.profiles.active` → задаёт основной активный профиль.
- `spring.profiles.include` → добавляет дополнительные профили поверх активного.

---

### 4. Можно ли активировать несколько профилей одновременно?

**Ответ:** Да, через запятую: `-Dspring.profiles.active=dev,local`

---

### 5. Как протестировать код с разными профилями?

**Ответ:** С помощью `@ActiveProfiles("test")` в тесте или запуском с разными профилями.

---

### 6. Что будет, если бин подходит под несколько профилей?

**Ответ:** Если на бин не указан `@Profile`, он доступен всегда. Если указано несколько, то бин поднимется только когда активен хотя бы один из этих профилей.

---

### 7. Что приоритетнее: `application.yml` или `application-{profile}.yml`?

**Ответ:** Сначала читается `application.yml`, потом поверх него накладывается `application-{profile}.yml`.

---

### 8. Как создать бин, который работает только в dev и test одновременно?

**Ответ:** 
```java
@Bean
@Profile({"dev", "test"})
public MyService devTestService() { ... }
```

---

### 9. Что будет, если в `application.yml` активировать несуществующий профиль?

**Ответ:** Spring загрузит `application.yml`, но `application-{profile}.yml` не найдёт. Ошибки не будет, просто профиль "пустой".

---

### 10. Можно ли активировать профиль программно?

**Ответ:** Да, через `SpringApplication.setAdditionalProfiles("dev")` или `ConfigurableEnvironment.addActiveProfile("dev")`.

---

### 11. Можно ли использовать разные файлы `application.properties` и `application.yml`вместе?

**Ответ:** Да, Spring загрузит оба, приоритет — `application.yml`.

---

### 12. В каком порядке Spring загружает настройки из профилей?

**Ответ:**
1. `application.yml`
2. `application-{profile}.yml`
3. Параметры командной строки (`--spring.profiles.active`)
4. Переменные окружения
5. JVM системные свойства (`-D...`)

Приоритет у командной строки > ENV > application.yml.

---

### 13. Как в тестах поднять профиль `test`, но подменить один бин из `prod`?

**Ответ:** Использовать `@ActiveProfiles("test")` + `@MockBean` или `@TestConfiguration` с `@Profile("test")`.

---

### 14. Что будет, если не указать `spring.profiles.active`?

**Ответ:** Используется только дефолтный `application.yml`, профили не активируются.

---

### 15. Можно ли задать разные профили для разных модулей в одном монолите?

**Ответ:** Да, у каждого модуля может быть свой `@Profile` на бинах. Spring выберет, какие поднять.

---

### 16. Можно ли активировать профиль во время работы приложения?

**Ответ:** Нет, профили выбираются при старте. После поднятия ApplicationContext переключить их нельзя.

---

### 17. Как отключить бин, если не активен ни один профиль?

**Ответ:** Указать `@Profile("!prod")` — бин поднимется во всех окружениях, кроме prod.

---

### 18. Можно ли задать несколько профилей для тестов с разными конфигами?

**Ответ:** Да:

`@ActiveProfiles({"test", "mock-api"})`

---

### 19. Что будет, если у тебя есть `application-dev.yml` и `application-dev.properties`?

**Ответ:** Оба файла загрузятся, Spring объединит их. Если одно и то же свойство есть в обоих → приоритет у `.yml`.

---

### 20. Как проверить активные профили во время работы приложения?

**Ответ:**
```java
@Autowired 
private Environment env; 
 
public void printProfiles() {     
	System.out.println(Arrays.toString(env.getActiveProfiles())); 
}
```