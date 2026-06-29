## Проверка элементов

Последнее обновление: 04.03.2024

### Проверка соответствия элементов условию

Отдельный вид операций позволяет проверить наличие элементов.

Функция all проверяет, все ли элементы коллекции/последовательности соответствуют условию предиката:

```

all(predicate: (T) -> Boolean): Boolean
```

Функция предиката получает каждый элемент и возвращает true, если элемент соответствует условию.

Функция any проверяет, соответствует хотя бы один элемент коллекции/последовательности условию предиката:

```

any(predicate: (T) -> Boolean): Boolean
```

Еще одна функция - none возвращает true, если ни один из элементов НЕ соответствует условию предиката:

```

none(predicate: (T) -> Boolean): Boolean
```

Проверка элементов на соответствие условию:

```

val people = listOf("Tom", "Kate", "Sam", "Alice", "Bob")
// all
println(people.all{it.length == 3})     // false
println(people.all{it.length != 10})     // true

// none
println(people.none{it.length == 3})     // false
println(people.none{it.length == 2})     // true

// any
println(people.any{it.length == 3})     // true
println(people.any{it.length == 10})    // false
```

Также функции any() и none() имеют версии без параметров. В этом случае функция
any() возвращает true, если коллекция/последовательность содержит хотя бы один элемент, а функция
none() возвращает true, если коллекция/последовательность пуста:

```

val people = listOf("Tom", "Kate", "Sam", "Alice", "Bob")
val empty: List<String> = listOf()
// any
println(people.any())     // true
println(empty.any())     // false

// none
println(people.none())     // false
println(empty.none())     // true
```

Функция contains() возвращает true, если в коллекции/последовательности есть определенный элемент:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
println(people.contains("Sam"))     // true
println(people.contains("Bill"))     // false
```

Еще одна функция - containsAll() возвращает true, если коллекция содержит все элементы другой коллекции:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
println(people.containsAll(listOf("Tom", "Sam")) ) // true
println(people.containsAll(listOf("Tom", "Bill")) ) // false
```

[Назад](./7.7.php)[Содержание](./)[Вперед](./7.9.php)