## Создание асинхронного потока

Последнее обновление: 23.06.2021

Для создания асинхронного потока можно применять различные способы. Рассмотрим три основных способа.

### Функция flow

Функция-построитель потока flow() позволяет задать логику передачи объектов в поток. Она может применяться как к отдельной функции, так и сама по себе.
Например, создание потока на базе функции:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val numberFlow = getNumbers()
    numberFlow.collect{n -> println(n)}
}

fun getNumbers() = flow{
    for(item in 1..5){
        emit(item * item)
    }
}
```

Здесь построитель `flow` создает поток на базе функции `getNumbers()`, передавая в поток квадраты значений от 1 до 5

Консольный вывод:

```

1
4
9
16
25
```

Определять поток в виде отдельной функции, как в примере выше, необязательно. Например:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = flow {
        val usersList = listOf("Tom", "Bob", "Sam")
        for (item in usersList){
            emit(item)
        }
    }
    userFlow.collect({user -> println(user)})
}
```

Здесь определена переменная `userFlow`, которая имеет тип `Flow<String>` и которая представляет поток, создаваемый построителем `flow`. В
данном случае flow передает в поток объекты из списка строк. Консольный вывод программы:

```

Tom
Bob
Sam
```

### Функция flowOf

Специальная функция-строитель flowOf() создает поток из набора переданных в функцию значений.

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val numberFlow : Flow<Int> = flowOf(1, 2, 3, 5, 8)
    numberFlow.collect{n -> println(n)}
}
```

В данном случае в функцию построителя асинхронного потока flowOf() передается 5 значений типа Int, поэтому создаваемый поток будет имет тип
`Flow<Int>`. Все переданные значения будут автоматически эмитироваться в поток. А получить их можно также как и в общем случае, например, через функцию `collect()`.

Консольный вывод программы:

```

1
2
3
5
8
```

Аналогичный пример со строками:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = flowOf("Tom", "Sam", "Bob")
    userFlow.collect({user -> println(user)})
}
```

### Метод asFlow

Стандартные коллекции и последовательности в Kotlin имеют метод расширения asFlow(), который позволяет преобразовать коллекцию или последовательность
в поток:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    // преобразование последовательности в поток
    val numberFlow : Flow<Int> = (1..5).asFlow()
    numberFlow.collect{n -> println(n)}

    // преобразование коллекции List<String> в поток
    val userFlow = listOf("Tom", "Sam", "Bob").asFlow()
    userFlow.collect({user -> println(user)})
}
```

[Назад](./9.1.php)[Содержание](./)[Вперед](./9.3.php)