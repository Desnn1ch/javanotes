## 1. Создание и использование `SessionFactory` и `Session`

### SessionFactory

`SessionFactory` - фабрика для создания объектов `Session`
- Создается при запуске приложения 1 раз
- Потокобезопасен
- Требует закрытия при завершении работы приложения

Пример создания `SessionFactory`:
```java
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            return new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
        } catch (Throwable ex) {
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        getSessionFactory().close();
    }
}
```

### Session

`Session` - это объект для соединения с БД
- Не является потокобезопасным
- Используется для выполнения SQL-запросов через HQL, Criteria API или SQL
- Управляет состоянием объектов в контексте транзакций

Получение `Session`:
```java
Session session = HibernateUtil.getSessionFactory().openSession();
```

## 2. Важность и работа с транзакциями

Hibernate использует транзакции (`Transaction`) для обеспечения целостности данных
- Операции CRUD должны выполняться внутри транзакций
- Транзакции обеспечивают атомарность: либо изменения сохранятся ('commit'), либо все ролбэкается (`rollback`)

Пример работы с транзакцией:
```java
Session session = HibernateUtil.getSessionFactory().openSession();
Transaction transaction = null;

try {
    transaction = session.beginTransaction();
    
    // Код работы с БД

    transaction.commit();
} catch (Exception e) {
    if (transaction != null) {
        transaction.rollback();
    }
    e.printStackTrace();
} finally {
    session.close();
}
```

## 3. CRUD операции (Create, Read, Update, Delete)

### Create
```java
Session session = HibernateUtil.getSessionFactory().openSession();
Transaction transaction = session.beginTransaction();

User user = new User();
user.setName("Никита");
user.setEmail("nikita@example.com");

session.save(user);  // Добавляет объект в БД
transaction.commit();
session.close();
```

**Как это работает?**
- `session.save(user);` — Hibernate **назначает объекту id** и отправляет 
  `INSERT INTO users (...) VALUES (...)`
- После коммита объект становится управляемым (находится под наблюдением сессии, и любые изменения в нем автоматически отслеживаются и могут быть сохранены в БД)

### Read
```java
Session session = HibernateUtil.getSessionFactory().openSession();

User user = session.get(User.class, 1);  // Получение объекта по ID
System.out.println(user.getName());

session.close();
```

### Update
```java
Session session = HibernateUtil.getSessionFactory().openSession();
Transaction transaction = session.beginTransaction();

User user = session.get(User.class, 1);
user.setEmail("newemail@example.com");

session.update(user);
transaction.commit();
session.close();
```
### Delete
```java
Session session = HibernateUtil.getSessionFactory().openSession();
Transaction transaction = session.beginTransaction();

User user = session.get(User.class, 1);
session.delete(user);

transaction.commit();
session.close();
```

## 4. **Обработка сессий и транзакций**

### Открытие и закрытие сессий

Открывать `Session` нужно только при необходимости и закрывать после работы:
```java
Session session = HibernateUtil.getSessionFactory().openSession();
session.close();
```

### Откат транзакции при ошибках

Если произошла ошибка, можно сделать `rollback()`, чтобы отменить изменения:
```java
try {
    transaction.commit();
} catch (Exception e) {
    transaction.rollback();
}
```

### Использование try-with-resources (Hibernate 5.2+)

Можно автоматизировать закрытие сессии:
```java
try (Session session = HibernateUtil.getSessionFactory().openSession()) {
    Transaction transaction = session.beginTransaction();
    
    // Работа с БД

    transaction.commit();
}
```