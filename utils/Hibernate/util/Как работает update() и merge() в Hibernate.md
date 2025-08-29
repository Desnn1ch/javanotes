## **1️⃣ Как `update()` отслеживает изменения?**

Метод `update()` переводит **отсоединённый (detached) объект в управляемое состояние (persistent)**.

📌 **Пример**:
```java
Session session1 = HibernateUtil.getSessionFactory().openSession();
Transaction transaction1 = session1.beginTransaction();

User user = session1.get(User.class, 1); // Загружаем объект
session1.close(); // Закрываем сессию, объект становится "detached"

// Изменяем объект в памяти
user.setName("Новое имя");

Session session2 = HibernateUtil.getSessionFactory().openSession();
Transaction transaction2 = session2.beginTransaction();

session2.update(user); // Переводим объект в managed-состояние
transaction2.commit(); // Hibernate отправит UPDATE в БД
session2.close();
```

📌 **Что происходит под капотом?**

1. Мы **загрузили объект** (он был управляемым).
2. Закрыли сессию → **объект стал "отсоединённым" (detached)**.
3. Изменили поле `name`, но Hibernate **пока не знает об этом**.
4. Открыли новую сессию, вызвали `session.update(user);` → Hibernate **снова начал управлять объектом**.
5. При коммите Hibernate **сравнил текущее состояние объекта с БД** и отправил `UPDATE users SET name = 'Новое имя' WHERE id = 1`.

---

## **2️⃣ Когда `update()` НЕ работает?**

### ❌ **Ошибка: обновление объекта, которого нет в БД**

Если объекта **вообще нет в БД**, Hibernate **выбросит исключение**:

```java
User user = new User();
user.setId(100); // Объекта с таким ID в БД нет
user.setName("Новый пользователь");

session.update(user); // Hibernate попытается сделать UPDATE, но объекта нет!
```

**Ошибка:**

```bash
org.hibernate.ObjectNotFoundException: No row with the given identifier exists
```

💡 **Решение:** перед `update()` **проверять, есть ли объект в БД** через `session.get()`.

---

### ❌ **Ошибка: два управляемых объекта с одним ID**

Если в одной сессии уже есть объект с тем же ID, что и у переданного в `update()`, Hibernate выбросит ошибку:

```java
User user1 = session.get(User.class, 1); // Первый объект (управляемый)
User user2 = new User(); // Второй объект (detached)
user2.setId(1);
user2.setName("Другое имя");

session.update(user2); // Ошибка! Два объекта с одним ID
```

**Ошибка:**
```bash
org.hibernate.NonUniqueObjectException: a different object with the same identifier value was already associated with the session
```

💡 **Решение:**

- Использовать `merge()` вместо `update()`
- Или сначала удалить `user1` из сессии:

```java
session.evict(user1); // evict - удаление объекта из сессии
session.update(user2);
```

---

## **3️⃣ `update()` vs `merge()`**

|Метод|Когда использовать?|Что делает?|
|---|---|---|
|`update(entity)`|Если **уверен, что объект есть в БД**|**Переводит detached-объект в managed-состояние** и применяет `UPDATE`|
|`merge(entity)`|Если **не уверен, есть ли объект в БД**|Если объект есть → `UPDATE`, если нет → `INSERT`|

- Если объект **есть в БД** → `UPDATE`.
- Если объекта **нет в БД** → `INSERT`.

---

## **Вывод**

✅ `update()` **переводит "отсоединённый" объект обратно в управляемое состояние** и выполняет `UPDATE` в БД.  
✅ Используй `update()`, если **точно знаешь, что объект уже есть в БД**.  
✅ Если объект может отсутствовать в БД, лучше **использовать `merge()`**.