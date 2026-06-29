## Функции count, take и drop. Количество элементов в потоке

Последнее обновление: 28.06.2021

### Функция count

Оператор `count` получает количество объектов в потоке:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Sam").asFlow()
    println("Count: ${userFlow.count()}")       // Count: 3
}
```

Также мы можем передать в функцию `count()` условие в виде функции, которая возвращает объект Boolean, то есть `true` или `false`. Тогда функция
`count()` возвратит количество элементов, которые соответствуют этому условию:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    val count = userFlow.count{ username -> username.length > 3 }
    println("Count: $count")       // Count: 2
}
```

В данном случае в качестве условия передается функция `username -> username.length > 3` . Ее параметр - это объект потока. Здесь мы говорим учитывать
строку, если ее длина больше 3 символов. В итоге в приведеном списке только два объекта соответствуют этому условию, поэтому переменная `count` будет равна 2.

### Функция take

Оператор take ограничивает количество элементов в потоке. В качестве параметра она принимает количество элементов с начала потока,
которые надо оставить в потоке:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    userFlow.take(3).collect{user -> println(user)}
}
```

В примере выше оставляем первые три элемента:

```

Tom
Bob
Kate
```

### Функция drop

Оператор drop удаляет из потока определенное количество элементов. В качестве параметра она принимает количество элементов с начала потока,
которые надо убрать из потока:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val userFlow = listOf("Tom", "Bob", "Kate", "Sam", "Alice").asFlow()
    userFlow.drop(3).collect{user -> println(user)}
}
```

В примере выше убираем первые три элемента, в итоге в потоке останется два последних элемента:

```

Sam
Alice
```

[Назад](./9.3.php)[Содержание](./)[Вперед](./9.5.php)