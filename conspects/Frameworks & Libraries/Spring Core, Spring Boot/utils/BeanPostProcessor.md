## Что такое `BeanPostProcessor`?

`BeanPostProcessor` — это **расширение Spring**, интерфейс, который позволяет "подхватывать" каждый бин **после того, как Spring его создал и внедрил зависимости**, но **до и/или после инициализации**.

Он выполняет _дополнительную логику над бином_, например:

- создание **прокси-обёрток**, ([[Как происходит создание прокси-оберток]])
- валидацию или настройку полей,
- регистрацию сторонних инструментов (логирование, мониторинг, метрики).

---

## Жизненный цикл и `BeanPostProcessor`

1. **Spring создаёт объект бина**.
2. **Внедряются зависимости** (`@Autowired`, конструкторы, сеттеры).
3. Запускается `postProcessBeforeInitialization(bean, beanName)`  
    → здесь можно _заменить бин_ или обернуть его в прокси.
4. Выполняется инициализация (`@PostConstruct`, `afterPropertiesSet`, custom init-метод).
5. Запускается `postProcessAfterInitialization(bean, beanName)`  
    → финальный шанс изменить бин (часто именно здесь делают AOP-прокси).

![[Screenshot 2025-08-31 at 01.17.26.png]]


---

## Пример

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("До инициализации: " + beanName);
        return bean; // можно вернуть другой объект
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("После инициализации: " + beanName);
        return bean; // часто возвращают прокси
    }
}
```

---

## Реальные применения

- **Spring AOP**: `AnnotationAwareAspectJAutoProxyCreator` — `BeanPostProcessor`, который ищет аннотации `@Transactional`, `@Async`, `@Cacheable` и оборачивает бин в прокси.
- **Validation**: Hibernate Validator подключается через `BeanPostProcessor`.
- **Lombok @Slf4j аналоги** — тоже можно реализовать на `BeanPostProcessor`, чтобы "вживлять" логгер.