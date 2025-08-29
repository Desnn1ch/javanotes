Любой объект в Hibernate может находиться в одном из **четырёх состояний**:
### 1️ **Transient (Временный, неуправляемый)**

Объект создан в коде, но еще **не добавлен в сессию** и **не существует в БД**.

```java
User user = new User(); // Это transient-объект
user.setName("Никита");
```

Hibernate его **не отслеживает** и не знает о нем.

---

### 2️ **Persistent (Управляемый, сохраненный в сессии)**

Когда объект **связан с сессией**, он становится **управляемым (managed)**.

```java
Session session = HibernateUtil.getSessionFactory().openSession();
Transaction transaction = session.beginTransaction();

User user = new User();
user.setName("Никита");

session.save(user); // Теперь объект управляемый
transaction.commit();
session.close();
```

Теперь Hibernate **отслеживает изменения** в `user` и при коммите автоматически применит `INSERT INTO users ...`.

---

### 3️ **Detached (Отсоединенный, неуправляемый)**

Когда **сессия закрывается**, объект остается в памяти, но **Hibernate больше за ним не следит**.

```java
session.close(); // Теперь объект detached
user.setName("Новое имя"); // Hibernate уже не видит это изменение!
```

Теперь, если изменить `user`, Hibernate **не отправит `UPDATE`**, потому что объект уже **не привязан к сессии**.

Чтобы снова сделать объект **управляемым**, нужно использовать `merge()` или `update()`:

```java
Session newSession = HibernateUtil.getSessionFactory().openSession();
Transaction newTransaction = newSession.beginTransaction();

newSession.update(user); // Теперь Hibernate снова следит за объектом
newTransaction.commit();
newSession.close();
// Если объекта в БД уже нет по какой-либо причине - ObjectNotFoundException
```

---

### 4️ **Removed (Помеченный на удаление)**

Когда вызывается `session.delete(user);`, объект становится **помеченным на удаление** и после `commit()` из БД **будет удалён**.

```java
session.delete(user); // Hibernate удалит запись при коммите
```

После `commit()` объект **по-прежнему существует в памяти**, но в БД его уже нет.

---

## **Вывод**

- **"Управляемый"** (Persistent) = Hibernate **следит** за объектом и его изменениями.  
- После `session.close()` объект становится **Detached**, и Hibernate его **больше не отслеживает**.  
- Чтобы снова сделать его **управляемым**, нужно `update()` или `merge()`.