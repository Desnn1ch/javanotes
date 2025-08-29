### **Установка и настройка Hibernate в проекте**

### 1 Установка зависимостей
#### Maven
хз
#### Gradle

```java
plugins {  
    id 'java'  
    id 'application'  
}  
  
application {  
    mainClass = 'com.example.Main'  
}  
  
group = 'com.example'  
version = '1.0-SNAPSHOT'  
  
repositories {  
    mavenCentral()  
}  
  
dependencies {  
    // Tests  
    testImplementation platform('org.junit:junit-bom:5.10.0')  
    testImplementation 'org.junit.jupiter:junit-jupiter'  
  
    // ORM  
    implementation 'org.hibernate:hibernate-core:6.4.0.Final'  
    implementation 'org.postgresql:postgresql:42.7.2'  
    implementation 'jakarta.persistence:jakarta.persistence-api:3.1.0'  
  
    // Recommended connection pool  
    implementation 'com.zaxxer:HikariCP:5.0.0'  
    implementation 'org.hibernate:hibernate-hikaricp:6.4.0.Final'  
  
    // Logback for logging  
    implementation 'org.slf4j:slf4j-api:2.0.0-alpha1'  
    implementation 'ch.qos.logback:logback-classic:1.4.12'  
}  
  
test {  
    useJUnitPlatform()  
}
```

### 2 Конфигурация `hibernate.cfg.xml`

Создать файл `hibernate.cfg.xml` в `src/main/resources`:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE hibernate-configuration PUBLIC  
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"  
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">  
  
<hibernate-configuration>  
    <session-factory>  
        <!-- JDBC настройки -->  
        <property name="hibernate.connection.driver_class">org.postgresql.Driver</property>  
        <property name="hibernate.connection.url">jdbc:postgresql://localhost:5432/mydatabase</property>  
        <property name="hibernate.connection.username">myuser</property>  
        <property name="hibernate.connection.password">mypassword</property>  
  
        <!-- Настройка пула HikariCP -->  
        <property name="hibernate.hikari.minimumIdle">5</property>  
        <property name="hibernate.hikari.maximumPoolSize">20</property>  
        <property name="hibernate.hikari.idleTimeout">300000</property>  
        <property name="hibernate.hikari.maxLifetime">600000</property>  
  
        <!-- Управление схемой -->  
        <property name="hibernate.hbm2ddl.auto">create</property>  
  
        <!-- Логирование SQL-запросов -->  
        <property name="hibernate.show_sql">false</property>  
        <property name="hibernate.format_sql">false</property>  
        <property name="hibernate.use_sql_comments">false</property>  
        <property name="hibernate.generate_statistics">false</property>  
  
        <!-- Аннотированные классы -->  
        <mapping class="com.exmaple.User"/>  
    </session-factory>  
</hibernate-configuration>
```

Для отключения логов Logback создать `logback.xml` в ``src/main/resources`:

```xml
<configuration>  
    <!-- Отключаем Hibernate логи -->  
    <logger name="org.hibernate" level="OFF"/>  
  
    <!-- Пример конфигурации для вывода в консоль -->  
    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">  
        <encoder>  
            <pattern>%d{yyyy-MM-dd HH:mm:ss} - %msg%n</pattern>  
        </encoder>  
    </appender>  
  
    <!-- Настроим корневой логгер, чтобы выводить логи только на консоль -->  
    <root level="INFO">  
        <appender-ref ref="Console"/>  
    </root>  
</configuration>
```

### 3 Аннотированные классы (JPA + Hibernate)

Добавить класс `User`:

```java
package com.example;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true, length = 100)
    private String email;

    public User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Геттеры и сеттеры
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

### 4 Подключение к базе данных через Hibernate (SessionFactory)

Создать `HibernateUtil` для управления сессиями:

```java
package com.example.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            return new Configuration().configure().buildSessionFactory();
        } catch (Throwable ex) {
            System.err.println("Ошибка инициализации SessionFactory: " + ex);
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

### 5 Использование Hibernate для работы с базой

Создать `UserDAO` для работы с базой:

```java
package com.example;

import com.example.User;
import com.example.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.Transaction;

public class UserDAO {

    public void saveUser(User user) {
        Transaction transaction = null;
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            transaction = session.beginTransaction();
            session.persist(user);
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) transaction.rollback();
            e.printStackTrace();
        }
    }

    public User getUser(Long id) {
        try (Session session = HibernateUtil.getSessionFactory().openSession()) {
            return session.get(User.class, id);
        }
    }
}
```

## 6 Тестирование работы с Hibernate

Создадим `Main`:

```java
package com.example;

import com.example.UserDAO;
import com.example.User;

public class Main {
    public static void main(String[] args) {
        UserDAO userDAO = new UserDAO();

        // Добавление пользователя
        User newUser = new User("Иван", "ivan@example.com");
        userDAO.saveUser(newUser);

        // Получение пользователя
        User retrievedUser = userDAO.getUser(1L);
        System.out.println("Пользователь: " + retrievedUser.getName() + " - " + retrievedUser.getEmail());
    }
}
```

### 7 Поднятие БД

Создать `docker-compose.yml`:

```yml
version: '3.8'  
  
services:  
  postgres:  
    image: postgres:16  
    container_name: my_postgres  
    restart: always  
    environment:  
      POSTGRES_DB: mydatabase  
      POSTGRES_USER: myuser  
      POSTGRES_PASSWORD: mypassword  
    ports:  
      - "5432:5432"  
    volumes:  
      - postgres_data:/var/lib/postgresql/data  
    healthcheck:  
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydatabase"]  
      interval: 10s  
      retries: 5  
      start_period: 5s  
  
volumes:  
  postgres_data:
```


