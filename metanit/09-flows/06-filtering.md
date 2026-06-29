## Фильтрация данных

Последнее обновление: 28.06.2021

### Функция filter

Оператор filter выполняет фильтрацию объектов в потоке. В качестве параметра он принимает функцию-условие, которая получает объект потока и возвращает true (если объект
подходит фильтрацию) и false (если не проходит):

```

inline fun <T> Flow<T>.filter(
    crossinline predicate: suspend (T) -> Boolean
): Flow<T> (source)
```

Рассмотрим на примере:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Bill", 5),
        Person("Sam", 14),
        Person("Bob", 21),
    ).asFlow()

    peopleFlow.filter{ person -> person.age > 17}
        .collect { person -> println("name: ${person.name}   age:  ${person.age} ")}
}

data class Person(val name: String, val age: Int)
```

Здесь для фильтрации применяется условие `person.age > 17`. Если возраст пользователя больше 17, то он оказывается в выходном потоке. Консольный вывод программы:

```

name: Tom   age:  37 
name: Bob   age:  21 
```

### takeWhile

Кроме того, Kotlin предоставляет еще ряд операций фильтрации для различных ситуаций. Например, оператор takeWhile выбирает из потока элементы,
пока будет истино некоторое условие:

```

fun <T> Flow<T>.takeWhile(
    predicate: suspend (T) -> Boolean
): Flow<T>
```

Рассмотрим на примере:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Alice", 32),
        Person("Bill", 5),
        Person("Sam", 14),
        Person("Bob", 25),
    ).asFlow()

    peopleFlow.takeWhile{ person -> person.age > 17}
        .collect { person -> println("name: ${person.name}   age:  ${person.age} ")}
}

data class Person(val name: String, val age: Int)
```

В данном случае оператор `takeWhile` будет извлекать в возвращаемый поток все объекты Person, у которых значение `age` больше 17. Консольный вывод программы:

```

name: Tom   age:  37 
name: Alice   age:  32 
```

Из консольного вывода мы видим, что несмотря на то, что в потоке еще есть объекты Person, которые соответствуют условию, но столкнувшись с первым объектом, который не соответствует условию,
`takeWhile` перестает выбирать объекты в возвращаемый поток.

### dropWhile

Оператор dropWhile противоположен по своему действию оператору `takeWhile`. dropWhile
удаляет из потока элементы, пока они не начнут соответствовать некоторому условию:

```

fun <T> Flow<T>.dropWhile(
    predicate: suspend (T) -> Boolean
): Flow<T>
```

Так, возьмем предыдущий пример, только вместо `takeWhile` используем dropWhile:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Alice", 32),
        Person("Bill", 5),
        Person("Sam", 14),
        Person("Bob", 25),
    ).asFlow()

    peopleFlow.dropWhile{ person -> person.age > 17}
        .collect { person -> println("name: ${person.name}   age:  ${person.age} ")}
}

data class Person(val name: String, val age: Int)
```

В данном случае оператор `dropWhile` пропустит первые два объекта Person, которые соответствовуют условию `person.age > 17`.
Третий элемент потока НЕ соответствовует этому условию, поэтому начиная с третьего элемента все остальные элементы попадут в возвращаемый поток (даже если они опять же соответствуют данному условию).
Консольный вывод программы:

```

name: Bill   age:  5 
name: Sam   age:  14 
name: Bob   age:  25 
```

[Назад](./9.6.php)[Содержание](./)[Вперед](./9.8.php)