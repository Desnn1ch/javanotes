
Когда мы запускаем `SpringApplication.run(...)`, внутри под капотом Spring Boot шаг за шагом поднимает окружение, ApplicationContext, загружает бины, стартует встроенный сервер и т. д. 

В каждый из этих моментов Spring Boot выбрасывает **событие (`ApplicationEvent`)**, чтобы можно было "подписаться" и выполнить свой код. Это механизм **observer pattern** внутри Spring. Мы можем слушать определённый этап старта/жизни приложения и реагировать: логировать, настраивать что-то, запускать свои сервисы.

### 1 **Старт приложения**

Вызов `SpringApplication.run(MyApp.class, args);`:
- Создаётся `SpringApplication`.
- Регистрируются слушатели (в т.ч. для логирования и чтения конфигов).
- Инициализируется **логирование** (по умолчанию SLF4J + Logback) ещё до контекста, чтобы видеть ранние логи.
- Печатается **баннер** (можно отключить `spring.main.banner-mode=off` или заменить `banner.txt`).

Посылается событие `ApplicationStartingEvent` — самый старт, контекста ещё нет.

---

### 2 **Подготовка Environment**

Собирается `Environment`: читаются `application.yml/properties`, переменные окружения, аргументы командной строки. 

Определяются **профили** (`spring.profiles.active`), подхватываются профильные файлы (`application-dev.yml` и т.п.). 

Посылается событие `ApplicationEnvironmentPreparedEvent` — окружение собрано, контекста ещё нет.

---

### 3 **Создание ApplicationContext**

Выбирается тип контекста: веб-приложение → `AnnotationConfigServletWebServerApplicationContext`, не-веб → `AnnotationConfigApplicationContext`.

Применяются `ApplicationContextInitializer` (если заданы и подготавливается инфраструктура для регистрации бинов (но сами бины ещё не созданы).

**Как сюда встроен парсинг конфигурации (аннотации/XML):**

**Аннотации (JavaConfig)**: контекст использует
- `AnnotatedBeanDefinitionReader` — регистрирует `@Configuration` классы;
- `ClassPathBeanDefinitionScanner` — сканирует пакеты на `@Component/@Service/@Repository/@Controller`.

**XML (альтернатива)**:
- `DefaultListableBeanFactory` + `XmlBeanDefinitionReader` читают DOM XML и регистрируют определения бинов.

Результат этого шага — начальный **реестр определений** (`BeanDefinitionRegistry`), т.е. карта `Map<beanName, BeanDefinition>` (логически — «метаданные бинов»).

Посылается `ApplicationContextInitializedEvent` — контекст создан, но бины ещё не загружены (идёт подготовка/регистрация определений).

---

### 4 **Загрузка конфигураций и регистрация BeanDefinition**

Обрабатываются `@Configuration` (включая `@Bean`, `@Import`, `@ImportResource`). Идёт **component-scan** по пакетам.

Для XML — `XmlBeanDefinitionReader` регистрирует определения из `*.xml`.

Срабатывают механизмы разворота конфигурации: `ImportSelector`, `ImportBeanDefinitionRegistrar`.

Автоконфигурация Boot (через `@EnableAutoConfiguration`) подсовывает дефолтные бины по classpath.

На выходе мы получаем полный набор `BeanDefinition` в реестре.

Посылается событие `ApplicationPreparedEvent` — контекст «собран» (definitions готовы), но ещё не «освежён» (`refresh()` не завершён).

---

### 5 **Применение BeanFactoryPostProcessor (метаданные, до создания бинов)**

Выполняются все `BeanFactoryPostProcessor` над **реестром определений** (метаданные): можно менять scope, зависимости, свойства до инстанцирования. 

Примеры: `PropertySourcesPlaceholderConfigurer` (подставляет `${...}`), пользовательские процессоры.

```java
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory bf) {
    var def = bf.getBeanDefinition("myService");
    def.setScope(BeanDefinition.SCOPE_PROTOTYPE); // меняем scope ДО создания бина
  }
}
```

---

### 6 **Регистрация BeanPostProcessor (инстансы, вокруг инициализации)**

В контейнер регистрируются `BeanPostProcessor` — перехватчики **жизненного цикла объектов** (не метаданных). Например:

- `AutowiredAnnotationBeanPostProcessor` (вколачивает `@Autowired`),
- `CommonAnnotationBeanPostProcessor` (`@PostConstruct`, `@PreDestroy`),
- AOP-семейство (`AbstractAutoProxyCreator`) — делает прокси под `@Transactional`, `@Async`, `@Cacheable` и т.д.

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
  @Override
  public Object postProcessBeforeInitialization(Object bean, String name) { /* ... */ return bean; }
  @Override
  public Object postProcessAfterInitialization (Object bean, String name) { /* ... */ return bean; }
}
```

> Эти процессоры будут вызываться **для каждого** создаваемого бина на шаге 7.

---

### 7 **Создание и инициализация singleton-бинов**

**Что происходит для каждого бина:**

1. **Инстанцирование** (конструктор или фабричный метод `@Bean`).
2. Внедрение зависимостей (`@Autowired`, `@Value`, сеттеры/поля/конструктор).
3. `BeanPostProcessor#postProcessBeforeInitialization`.
4. Инициализация:
    `@PostConstruct` и/или `InitializingBean#afterPropertiesSet()`.
5. `BeanPostProcessor#postProcessAfterInitialization`:
    здесь часто **оборачивают в прокси** (AOP/транзакции/кеш/асинхронность).

**Итог:** к этому моменту все singleton-бины готовы, часть из них — проксированы.

---

### 8 **Поднятие встроенного веб-сервера и запуск веб-инфраструктуры (если web)**

В `ServletWebServerApplicationContext` в процессе `refresh()` создаётся `ServletWebServerFactory`(Tomcat/Jetty/Undertow) и стартует сервер.

Регистрируется `DispatcherServlet`, настраиваются `HandlerMapping`/`HandlerAdapter`, фильтры, конвертеры и т.п. Привязываются все `@Controller` и их маппинги.

Отсылаются события на шаге
- `WebServerInitializedEvent` — веб-сервер поднялся и держит порт.
- `ContextRefreshedEvent` — контекст «освежён» (refresh завершён).
- `ApplicationStartedEvent` — приложение запустилось (но ещё не выполнялись раннеры).
- Сразу после — `AvailabilityChangeEvent(LivenessState.CORRECT)` чтобы обозначить, что приложение считается работающим

---

### 9 **Финализация запуска: раннеры и готовность к трафику**

Выполняются `ApplicationRunner` и `CommandLineRunner` (если есть) — инициализация данных, фоновые задачи и т.п. Приложение объявляет «готовность».

Отсылаются события на шаге
- `ApplicationReadyEvent` — всё готово, раннеры отработали.
- Сразу после — `AvailabilityChangeEvent(ReadinessState.ACCEPTING_TRAFFIC)` — «принимаем трафик».

Если на любом этапе что-то упало → публикуется `ApplicationFailedEvent`, логируется причина 

---

## Итого

1. `run()` → логирование, баннер, `ApplicationStartingEvent`.
2. Environment → конфиги/профили, `ApplicationEnvironmentPreparedEvent`.
3. Создание контекста → выбор типа, init-ы, подготовка к регистрации, `ApplicationContextInitializedEvent`.
4. Регистрация `BeanDefinition` → аннотации/XML, автоконфигурация, `ApplicationPreparedEvent`.
5. `BeanFactoryPostProcessor` → правим **метаданные** до создания.
6. Регистрируем `BeanPostProcessor` → будем перехватывать **объекты**.
7. Создаём singleton-бины → DI → `@PostConstruct` → прокси (AOP).
8. Старт web-сервера (если есть) → `WebServerInitializedEvent` → `ContextRefreshedEvent` → `ApplicationStartedEvent` → `Liveness=CORRECT`.
9. Выполняем раннеры → `ApplicationReadyEvent` → `Readiness=ACCEPTING_TRAFFIC`.