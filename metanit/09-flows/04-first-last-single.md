## Функции first, last, single

Последнее обновление: 23.06.2021

### first/firstOrNull

Метод `first()` получает первый объект списка:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    val firstUser = userFlow.first()
    println("First User: $firstUser")       // First User: Tom
}
```

Также метод `first()` может в качестве параметра принимать функцию-условие, которая возвращает объект Boolean. Тогда `first()`
возвращает первый элемент потока, который соответствует этому условию:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    val firstUser = userFlow.first{ name-> name.length > 3}
    println("First User: $firstUser")       // First User: Kate
}
```

Здесь мы получаем первый элемент, длина которого больше 3 символов. В потоке это строка "Kate".

Однако может сложиться ситуация, когда условию не соответствует ни один из элементов потока:

```

val userFlow = listOf("Tom", "Bob", "Sam").asFlow()
val firstUser = userFlow.first{ name-> name.length > 3}
```

Или поток банально пуст:

```

val userFlow = listOf<String>().asFlow()
val firstUser = userFlow.first()
```

В обоих случаях при работе программы вылетит исключение. В этом случае мы, конечно, может обрабатывать исключение с помощью `try..catch`. Однако, в качестве
альтернативы Kotlin предоставляет метод `firstOrNull()`, который возвращает `null`, если поток пуст или ни один из его элементов не соответствуют условию:

```

val userFlow = listOf<String>().asFlow()
val firstUser1 = userFlow.firstOrNull()
val firstUser2 = userFlow.firstOrNull{ name-> name.length > 3}
```

В этом случае мы можем проверить полученное значение на `null`:

```

if(firstUser1 == null) 
	println("User not found")
else 
	println("First User: $firstUser1")
```

### last / lastOrNull

Функция last() вытягивает из потока последний элемент, только в отличие от функции `first()` (по крайней мере для версии Kotlin 1.5.1) она
не принимает условие: также может принимать условие:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Alice", "Sam").asFlow()
    val lastUser = userFlow.last()
	println("Last User: $lastUser")       // Last User: Sam
}
```

Если есть вероятность, что поток будет пуст, то можно использовать функцию lastOrNull, которая возвращает `null` в случае
отсутствия элементов:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Alice", "Sam").asFlow()
    val lastUser = userFlow.lastOrNull()
    if(lastUser!=null) println("Last User: $lastUser")
}
```

### single / singleOrNull

Функция single() возвращает единственный элемент потока, если поток содержит только один элемент. Если
поток не содержит элементов генерируется исключение `NoSuchElementException`, а если в потоке больше одного элемента - исключение `IllegalStateException`.

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom").asFlow()
    try {
        val singleUser = userFlow.single()
        println("Single User: $singleUser")
    }
    catch(e:Exception) { println(e.message)  }
}
```

В качестве альтернативы можно использовать метод singleOrNull(), который возвращает `null`, если поток пуст или если в потоке больше одного элемента.

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob").asFlow()
    val singleUser = userFlow.singleOrNull()
    if(singleUser!=null)
        println("Single User: $singleUser")
    else
        println("Not found")
}
```

[Назад](./9.4.php)[Содержание](./)[Вперед](./9.6.php)