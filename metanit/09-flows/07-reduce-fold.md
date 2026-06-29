## Сведение данных. Функции reduce и fold

Последнее обновление: 28.06.2021

### Функция reduce

Оператор reduce сводит все значения потока к одному значению:

```

suspend fun <S, T : S> Flow<T>.reduce(
    operation: suspend (accumulator: S, value: T) -> S
): S (source)
```

`reduce` принимает функцию, которая имеет два параметра. Первый параметр при первом запуске представляет первый объект потока, а при последующих запусках -
результат функции над предыдущими объектами. А второй параметр функции - следующий объект.

Например, у нас есть поток чисел, найдем сумму всех чисел в потоке:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val numberFlow = listOf(1, 2, 3, 4, 5).asFlow()
    val reducedValue = numberFlow.reduce{ a, b -> a + b }
    println(reducedValue)   // 15
}
```

Здесь при первом запуске в функции в `reduce` параметр a равен 1, а параметр b равен 2.

При втором запуске параметр a содержит результат предыдущего выполнения функции, то если число 1 + 2 = 3, а параметр `b` равен 3 - следующее число в потоке.

Или другой пример со строками:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    val reducedValue = userFlow.reduce{ a, b -> "$a $b" }
    println(reducedValue)   // Tom Bob Kate Sam Alice
}
```

Здесь `reduce` соединяет все строки в одну.

### Функция fold

Функция fold также сводит все элементы потока в один. Но в отличие от оператора `reduce` оператор `fold`
в качестве первого параметра принимает начальное значение:

```

inline suspend fun <T, R> Flow<T>.fold(
    initial: R,
    crossinline operation: suspend (acc: R, value: T) -> R
): R (source)
```

Например:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    val foldedValue = userFlow.fold("Users:", { a, b -> "$a $b" })
    println(foldedValue)   // Users: Tom Bob Kate Sam Alice
}
```

В данном случае начальным значением является строка "Users:", к которой затем добавляются остальные элементы потока данных.

[Назад](./9.7.php)[Содержание](./)[Вперед](./9.9.php)