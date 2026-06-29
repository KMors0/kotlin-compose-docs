## Получение отдельных элементов

Последнее обновление: 04.03.2024

### Получение элемента по индексу

Для получения элемента по индексу применяется функция elementAt(), в которую передается индекс элемента:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
// получаем элемент по индексу 1
println(people.elementAt(1))     // Sam
```

Однако если переданный индекс выходит за границы коллекции/последовательности, то приложение сгенерирует ошибку. В этом случае мы можем использовать
функцию elementAtOrNull(), которая возвращает null, если индекс вне границ:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
println(people.elementAtOrNull(1))     // Sam
println(people.elementAtOrNull(6))     // null
```

Также в этом случае можно использовать функцию elementAtOrElse():

```

elementAtOrElse(index: Int, defaultValue: (Int) -> T): T
```

Первый параметр функции - индекс элемента, а второй - функция, которая получает индекс и возвращает значение по умолчанию, если индекс находится вне границ
коллекции/последовательности.

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")

println(people.elementAtOrElse(1){"Undefined"})     // Sam
println(people.elementAtOrElse(6){"Undefined"})     // Undefined
println(people.elementAtOrElse(8){"Index $it out of bounds"})  // Index 8 out of bounds
```

### Получение по условию

Если надо получить первый или последний элементы коллекции/последовательности, то можно использовать соответственно функции first()
и last():

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
println(people.first())     // Tom
println(people.last())     // Alice
```

В качестве параметра эти функции могут принимать функцию предиката

```

first(predicate: (T) -> Boolean): T
last(predicate: (T) -> Boolean): T
```

Эти функции возвращают первый и последний элементы, которые соответствуют условию предиката:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
// первый элемент с длиной в 4 символа
println(people.first{it.length==4})     // Kate
// последний элемент с длиной в 3 символа
println(people.last{it.length==3})     // Bob
```

Однако если коллеккция пуста, или в коллекции/последовательности нет элементов, которые соответствуют условию, то программа генерирует ошибку. В этом случае
можно применять функции firstOrNull() и lastOrNull(), которые возвращают null, если коллекция пуста или
в ней нет элементов, соответствующих условию:

```

firstOrNull(): T?
firstOrNull(predicate: (T) -> Boolean): T?

lastOrNull(): T?
lastOrNull(predicate: (T) -> Boolean): T?
```

Пример применения:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")

println(people.firstOrNull{it.length==3})     // Tom
println(people.firstOrNull{it.length==33})     // null

println(people.lastOrNull{it.length==3})     // Bob
println(people.lastOrNull{it.length==33})     // null
```

### Выбор случайного элемента

Для извлечения случайного элемента применяется функция random()

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
println(people.random())
```

Однако если коллекция/последовательность пуста, то эта функция генерирует ошибку. В этом случае можно использовать функцию randomOrNull(),
которая в этом случае возвращает null:

```

val people = listOf<String>()
println(people.randomOrNull())     // null
```

[Назад](./7.14.php)[Содержание](./)[Вперед](./8.1.php)