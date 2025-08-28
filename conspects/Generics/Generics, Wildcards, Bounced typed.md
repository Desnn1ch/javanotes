### Основная концепция

**Generics** позволяют создавать классы, интерфейсы и методы с параметрами типов, что обеспечивает типобезопасность и исключает необходимость приведения типов.
#### Проблема до Generic
```
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

#### Решение с Generic
```
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

##### Преимущества

- **Type safety** - проверка типов на этапе компиляции
- **Elimination of casts** - не нужно явное приведение типов
- **Code reusability** - один алгоритм для разных типов


### Несовместимость generic-типов

Для того чтобы сохранить целостности и независимости друг от друга Коллекции, у Generics существует так называемая "Несовместимость generic-типов".

**Суть такова:**
Если `Foo` является подтипом `Bar`, то `G<Foo>` НЕ является подтипом `G<Bar>`

**Пример проблемы**
```
// Integer является подтипом Object
Integer extends Object ✓

// НО List<Integer> НЕ является подтипом List<Object>
List<Integer> extends List<Object> ✗
```

**Пример с ошибкой**
```
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
```
 void dump(Collection<Object> c) {
   for (Iterator<Object> i = c.iterator(); i.hasNext(); ) {
       Object o = i.next();
       System.out.println(o);
   }
 }
```

```
 List<Object> l; dump(l);
 List<Integer> l; dump(l); // **Ошибка**
```

В этом примере `List<Integer>` не может использовать метод dump, так как он не является подтипом `List<Object>`.

Проблема в том что эта реализация кода не эффективна, так как `Collection<Object>` не является полностью родительской коллекцией всех остальных коллекции, грубо говоря `Collection<Object>` имеет ограничения.

Для решения этой проблемы используется Wildcard ("?"). Он не имеет ограничения в использовании(то есть имеет соответствие с любым типом) и в этом его плюсы. И теперь, мы можем вызвать dump с любым типом коллекции.
```
 void dump(Collection<?> c) {
   for (Iterator<?> i = c.iterator(); i.hasNext(); ) {
       Object o = i.next();
       System.out.println(o);
   }
 }
```

##### Решение 2 - Bounded Wildcard
Пусть мы захотели написать метод, который рисует `List<Shape>`. И у Shape есть наследник Circle. И мы хотим вызвать draw для Circle.
```
 void draw(List<Shape> c) {
   for (Iterator<Shape> i = c.iterator(); i.hasNext(); ) {
       Shape s = i.next();
       s.draw();
   }
 }
```

```
List<Shape> l; draw(l);
List<Circle> l; draw(l); // Ошибка
```

Проблема в том, что у нас не получится из-за несовместимости типов. Предложенное решение используется, если метод который нужно реализовать использовал бы определенный тип и его подтипов. Так называемое "Ограничение сверху". Для этого нужно вместо `<Shape>` прописать `<? extends Shape>`.

```
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
```
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

```
 <T> void addAll(T[] a, Collection<T> c) {
   for (int i = 0; i < a.length; i++) {
       c.add(a[i]);
   }
 }
```

Но все равно после выполнение останется ошибка в третьей строчке:
```
 addAll(new Object[10], new ArrayList<String>()); // **Ошибка**
```

##### Решение 4 – Bounded type argument

Реализуем метод копирование из одной коллекции в другую
```
<M> void addAll(Collection<M> c, Collection<M> c2) {
 for (Iterator<M> i = c.iterator(); i.hasNext(); ) {
  M o = i.next();
  c2.add(o);
 }
}
```

```
addAll(new AL<Integer>(), new AL<Integer>());
addAll(new AL<Integer>(), new AL<Object>()); //Ошибка
```

Проблема в том что две Коллекции могут быть разных типов (несовместимость generic-типов). Для таких случаев было придуман Bounded type argument. Он нужен если метод ,который мы пишем использовал бы определенный тип данных. Для этого нужно ввести `<N extends M>` (N принимает только значения M). Также можно корректно писать` <T extends A & B & C>`. (Принимает значения нескольких переменных)

```
<M, N extends M> void addAll(Collection<N> c, Collection<M> c2) {
 for (Iterator<N> i = c.iterator(); i.hasNext(); ) {
  N o = i.next();
  c2.add(o);
 }
}
```

##### Решение 5 – **Lower bounded wcard**

Реализуем метод нахождение максимума в коллекции.
```
<T extends Comparable<T>>
  T max(Collection<T> c) {
   …
  }
```

```
List<Integer> il; Integer I = max(il);
class Test implements Comparable<Object> {…} 
List<Test> tl; Test t = max(tl); // Ошибка
```

` <T extends Comparable<T>> `обозначает что `Т` обязан реализовывать интерфейс `Comparable<T>`

Ошибка возникает из за того что Test реализует интерфейс `Comparable<Object>`. Решение этой проблемы - Lower bounded wcard("Ограничение снизу"). Суть в том что мы будет реализовывать метод не только для Т, но и для его Супер-типов(Родительских типов). Например: Если мы напишем
```
List<T super Integer> list;
```

то мы можем заполнить его `List<Integer>`, `List<Number>` или `List<Object>`.
```
<T extends Comparable<? super T>>
 T max(Collection<T> c) {
     …
 }
```

##### Решение 6 – Wildcard Capture

Реализуем метод Swap в `List<?>`
```
void swap(List<?> list, int i, int j) {
    list.set(i, list.get(j)); // Ошибка
}
```

Проблема в том, что метод List.set() не может работать с` List<?>`, так как ему не известно какой он List. Для решение этой проблемы используют "Wildcard Capture" (или "Capture helpers"). Суть заключается в том, чтобы обмануть компилятор. Напишем еще один метод с параметризованной переменной и будем его использовать внутри нашего метода.

```
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

```
Collection<T> c;
T[] ta;
new T[10]; // Ошибка !!
```

- Невозможно создать массив Generic-классов

```
new ArrayList<List<Integer>>();
List<?>[] la = new List<?>[10]; // Ошибка !!
```

#### Преобразование типов

В Generics также можно манипулировать с информацией, хранящийся в переменных.

- Уничтожение информации о типе

```
List l = new ArrayList<String>();
```

- Добавление информации о типе

```
List<String> l = (List<String>) new ArrayList();
List<String> l1 = new ArrayList();
```

