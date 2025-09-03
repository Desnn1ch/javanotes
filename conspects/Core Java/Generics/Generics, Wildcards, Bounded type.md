### Ключевые слова (ориентиры)

- **Object до Java 5**
- **Type safety**
- **Casting / ClassCastException**
- **Invariant types (`List<Integer>` ≠ `List<Object>`)**
- **Wildcard `?`**
- **Bounded wildcard (`extends`)**
- **Lower bounded wildcard (`super`)**
- **Generic method `<T>`**
- **Bounded type argument `<N extends M>`**
- **Wildcard capture**
- **Type erasure**
- **Ограничения массивов**

### Основная концепция

**Generics** позволяют создавать классы, интерфейсы и методы с параметрами типов, что обеспечивает типобезопасность и исключает необходимость приведения типов.
#### Проблема до Generic
```java
// До Java 5 - использование Object
class OldBox {
    private Object value;
    
    public void setValue(Object value) {
        this.value = value;
    }
    
    public Object getValue() {
        return value;
    }
}

// Проблемы:
OldBox box = new OldBox();
box.setValue("Hello");
String s = (String) box.getValue(); // Необходимость cast'а
Integer i = (Integer) box.getValue(); // ClassCastException в runtime!
```

## Generic

Generics — это механизм параметризации типов в Java. Они решают 2 задачи:
- **Типобезопасность на этапе компиляции** (ловим ошибки заранее).
- **Повторное использование кода** (один `List<T>` вместо `ListString`, `ListInteger`, …).

```java
class GenericBox<T> { // T - параметр типа (type parameter)
    private T value;
    
    public void setValue(T value) {
        this.value = value;
    }
    
    public T getValue() {
        return value;
    }
}

// Использование
GenericBox<String> stringBox = new GenericBox<>();
stringBox.setValue("Hello");
String s = stringBox.getValue(); // Без приведения типов!

GenericBox<Integer> intBox = new GenericBox<>();
intBox.setValue(123);
Integer i = intBox.getValue(); // Типобезопасно
```

### Type Erasure (стирание типов)

В **байткоде JVM нет информации о типах generics**. На этапе компиляции `<T>` стирается до верхней границы (`extends`) или до `Object`, если границы нет. Пример:
```java
class Box<T> {
    private T value;
    public T get() { return value; }
}

// After compilation
class Box {
    private Object value;
    public Object get() { return value; }
}

//Compiler is auto-inserting casts
Box<String> box = new Box<>();
String s = (String) box.get(); // inserting String
```

###### Если в рантайме нет вообще никакой информации о дженериках, тогда как операции вообще выполняются в рантайме? 

В рантайме действительно нет информации о дженериках, потому что при компиляции параметр типа стирается до `Object` или до верхней границы `extends`. JVM всегда работает с объектами через ссылки фиксированного размера, поэтому ей не нужно знать конкретный generic-тип. 

Проверку корректности типов выполняет компилятор: он гарантирует, что в `Box<String>` нельзя положить `Integer`, а при извлечении автоматически вставляет в байткод приведение типов (`checkcast`). Если в реальности окажется несоответствие, в рантайме будет выброшено `ClassCastException`.

###### Как выделяется нужное количество памяти в рантайме?

Что касается памяти, то JVM всегда хранит в полях и переменных generic-классов лишь ссылки на объекты, которые занимают одинаковый объём (обычно 4 или 8 байт). Конкретный объект в куче уже знает свой фактический тип и размер, и выделение памяти для него выполняется при `new`, независимо от того, в каком generic-контексте он используется. Таким образом, generics нужны только для безопасности типов на этапе компиляции, а в байткоде и при выполнении остаются обычные объекты и ссылки на них.

### Ограничения

#### 1 Нельзя создавать массив дженериков

```java
T[] arr = new T[10];
System.out.println(arr.getClass());
```

Почему?

- В рантайме generics стираются → JVM видит просто `Object[]`.
- У массивов есть _runtime type check_ (они хранят информацию о типе).
- У generics — только compile-time check.
- Получается что создается массив Object-ов и в compile-time-е у массива тип Object, и в рантайме тоже также.
- Поэтому массив дженериков нельзя создать — иначе можно положить не тот тип и поймать `ClassCastException`.

---

#### 2 Нельзя использовать примитивы

`List<int> list = new ArrayList<>(); // ошибка`

Generics требуют ссылочные типы, потому что примитивы не реализуют класс Object, соответственно невозможно сделать Type Erasure

---

#### 3 Перегрузка методов по generic-параметру невозможна

```java
void test(List<String> list) {}
void test(List<Integer> list) {} // ошибка компиляции
```

После type erasure обе сигнатуры будут `void test(List list)` → Компилятор не сможет различить методы.

---

#### 4 instanceof с generics

```java
if (list instanceof List<String>) {} // ошибка
```

В рантайме `List<String>` → `List`. Проверять можно только на `List<?>` или `List`.

---

#### 5 Нельзя создать `new T()`

```java
class Box<T> {
    T value;
    public Box() {
        value = new T(); // ошибка
    }
}
```

Тип `T` стирается до `Object` или верхней границы. В рантайме JVM не знает, какой именно конструктор вызвать. У него нету конструкторов типа T в рантайме.

Решение: передавать `Class<T>` и использовать рефлексию:

```java
class Box<T> {
    T value;
    Box(Class<T> clazz) throws Exception {
        value = clazz.getDeclaredConstructor().newInstance();
    }
}
```

---

#### 6 static-контекст

В дженерик-классе **нельзя использовать T в статике**.

```java
class Box<T> {
    static T value; // ошибка
}
```

`static` принадлежит классу, а не объекту. У разных экземпляров с разными T статика была бы одна → ломается типобезопасность.

---

#### 7 Обобщения и наследование

`List<Object>` ≠ `List<String>` (это не ковариантность).

```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings; // ошибка
```

Иначе можно было бы положить `Integer` в `List<Object>`, а фактически это был бы `List<String>`.

Для таких случаев нужны wildcard-и: `List<? extends Object>`.


## Несовместимость generic-типов

Для того чтобы сохранить целостности и независимости друг от друга Коллекции, у Generics существует так называемая "Несовместимость generic-типов".

**Суть такова:**
Если `Foo` является подтипом `Bar`, то `G<Foo>` НЕ является подтипом `G<Bar>`

**Пример проблемы**
```java
// Integer является подтипом Object
Integer extends Object ✓

// НО List<Integer> НЕ является подтипом List<Object>
List<Integer> extends List<Object> ✗
```

**Пример с ошибкой**
```java
List<Integer> integerList = new ArrayList<>();
integerList.add(1);
integerList.add(2);
integerList.add(3);

// Попытка присвоения - ОШИБКА КОМПИЛЯЦИИ!
List<Object> objectList = integerList; // Cannot convert List<Integer> to List<Object>
```

### Проблемы реализации Generics

##### Решение 1 - **Wildcard**

Пусть мы захотели написать метод, который берет `Collection<Object>` и выводит на экран. И мы захотели вызвать dump для Integer.
```java
 void dump(Collection<Object> c) {
   for (Iterator<Object> i = c.iterator(); i.hasNext(); ) {
       Object o = i.next();
       System.out.println(o);
   }
 }
```

```java
 List<Object> l; dump(l);
 List<Integer> l; dump(l); // **Ошибка**
```

В этом примере `List<Integer>` не может использовать метод dump, так как он не является подтипом `List<Object>`.

Проблема в том что эта реализация кода не эффективна, так как `Collection<Object>` не является полностью родительской коллекцией всех остальных коллекции, грубо говоря `Collection<Object>` имеет ограничения.

Для решения этой проблемы используется Wildcard ("?"). Он не имеет ограничения в использовании(то есть имеет соответствие с любым типом) и в этом его плюсы. И теперь, мы можем вызвать dump с любым типом коллекции.
```java
 void dump(Collection<?> c) {
   for (Iterator<?> i = c.iterator(); i.hasNext(); ) {
       Object o = i.next();
       System.out.println(o);
   }
 }
```

##### Решение 2 - Bounded Wildcard
Пусть мы захотели написать метод, который рисует `List<Shape>`. И у Shape есть наследник Circle. И мы хотим вызвать draw для Circle.
```java
 void draw(List<Shape> c) {
   for (Iterator<Shape> i = c.iterator(); i.hasNext(); ) {
       Shape s = i.next();
       s.draw();
   }
 }
```

```java
List<Shape> l; draw(l);
List<Circle> l; draw(l); // Ошибка
```

Проблема в том, что у нас не получится из-за несовместимости типов. Предложенное решение используется, если метод который нужно реализовать использовал бы определенный тип и его подтипов. Так называемое "Ограничение сверху". Для этого нужно вместо `<Shape>` прописать `<? extends Shape>`.

```java
 void draw(List<? extends Shape> c) {
   for (Iterator<? extends Shape> i = c.iterator();
           i.hasNext(); ) {
       Shape s = i.next();
       s.draw();
   }
 }
```

##### Решение 3 – **Generic-Метод**
Пусть вы захотели сделать метод, который берет массив Object и переносить их в коллекцию.
```java
 void addAll(Object[] a, Collection<?> c) {
   for (int i = 0; i < a.length; i++) {
       c.add(a[i]);
   }
 }
```

```
 addAll(new String[10], new ArrayList<String>());
 addAll(new Object[10], new ArrayList<Object>());
 addAll(new Object[10], new ArrayList<String>()); // **Ошибка**
 addAll(new String[10], new ArrayList<Object>()); // **Ошибка**
```
Напомним, что вы не можете просто засунуть Object в коллекции неизвестного типа. Способ решения этой проблемы является использование "Generic-Метод" Для этого перед методом нужно объявить `<T>` и использовать его.

```java
 <T> void addAll(T[] a, Collection<T> c) {
   for (int i = 0; i < a.length; i++) {
       c.add(a[i]);
   }
 }
```

Но все равно после выполнение останется ошибка в третьей строчке:
```java
 addAll(new Object[10], new ArrayList<String>()); // **Ошибка**
```

##### Решение 4 – Bounded type argument

Реализуем метод копирование из одной коллекции в другую
```java
<M> void addAll(Collection<M> c, Collection<M> c2) {
 for (Iterator<M> i = c.iterator(); i.hasNext(); ) {
  M o = i.next();
  c2.add(o);
 }
}
```

```java
addAll(new AL<Integer>(), new AL<Integer>());
addAll(new AL<Integer>(), new AL<Object>()); //Ошибка
```

Проблема в том что две Коллекции могут быть разных типов (несовместимость generic-типов). Для таких случаев было придуман Bounded type argument. Он нужен если метод ,который мы пишем использовал бы определенный тип данных. Для этого нужно ввести `<N extends M>` (N принимает только значения M). Также можно корректно писать` <T extends A & B & C>`. (Принимает значения нескольких переменных)

```java
<M, N extends M> void addAll(Collection<N> c, Collection<M> c2) {
 for (Iterator<N> i = c.iterator(); i.hasNext(); ) {
  N o = i.next();
  c2.add(o);
 }
}
```

##### Решение 5 – **Lower bounded wcard**

Реализуем метод нахождение максимума в коллекции.
```java
<T extends Comparable<T>>
  T max(Collection<T> c) {
   …
  }
```

```java
List<Integer> il; 
Integer I = max(il);
class Test implements Comparable<Object> {…} 
List<Test> tl; Test t = max(tl); // Ошибка
```

` <T extends Comparable<T>> `обозначает что `Т` обязан реализовывать интерфейс `Comparable<T>`

Ошибка возникает из за того что Test реализует интерфейс `Comparable<Object>`. Решение этой проблемы - Lower bounded wcard("Ограничение снизу"). Суть в том что мы будет реализовывать метод не только для Т, но и для его Супер-типов(Родительских типов). Например: Если мы напишем
```java
List<T super Integer> list;
```

то мы можем заполнить его `List<Integer>`, `List<Number>` или `List<Object>`.
```java
<T extends Comparable<? super T>>
 T max(Collection<T> c) {
     …
 }
```

##### Решение 6 – Wildcard Capture

Реализуем метод Swap в `List<?>`
```java
void swap(List<?> list, int i, int j) {
    list.set(i, list.get(j)); // Ошибка
}
```

Проблема в том, что метод List.set() не может работать с` List<?>`, так как ему не известно какой он List. Для решение этой проблемы используют "Wildcard Capture" (или "Capture helpers"). Суть заключается в том, чтобы обмануть компилятор. Напишем еще один метод с параметризованной переменной и будем его использовать внутри нашего метода.

```java
void swap(List<?> list, int i, int j) {
    swapImpl(list, i, j);
}
<T> void swapImpl(List<T> list, int i, int j) {
    T temp = list.get(i);
    list.set(i, list.get(j));
    list.set(j, temp);
}
```

#### Ограничения Generic

Также нужно запомнить простые правила для работы с Generics.

- Невозможно создать массив параметра типа

```java
Collection<T> c;
T[] ta;
new T[10]; // Ошибка !!
```

- Невозможно создать массив Generic-классов

```java
new ArrayList<List<Integer>>();
List<?>[] la = new List<?>[10]; // Ошибка !!
```

#### Преобразование типов

В Generics также можно манипулировать с информацией, хранящийся в переменных.

- Уничтожение информации о типе

```java
List l = new ArrayList<String>();
```

- Добавление информации о типе

```
List<String> l = (List<String>) new ArrayList();
List<String> l1 = new ArrayList();
```

