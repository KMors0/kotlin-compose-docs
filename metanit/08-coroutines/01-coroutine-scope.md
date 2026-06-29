## Область корутины

Последнее обновление: 22.01.2022

Корутина может выполняться только в определенной области корутины (`coroutine scope`). Область корутин представляет пространство, в рамках которого
действуют корутины, она имеет определенный жизненный цикл и сама управляет жизненным циклом создаваемых внутри нее корутин.

И для создания области корутин в Kotlin может использоваться ряд функций, которые создают объект интерфейса CoroutineScope.
Одной из функций является coroutineScope. Она может применяться к любой функции, например:

```

import kotlinx.coroutines.*

suspend fun main(){

    doWork()

    println("Hello Coroutines")
}
suspend fun doWork()= coroutineScope{
    launch{
        for(i in 0..5){
            println(i)
            delay(400L)
        }
    }
}
```

### Запуск нескольких корутин

Подобным образом можно запускать в одной функции сразу несколько корутин. И они будут выполняться одновременно. Например:

```

import kotlinx.coroutines.*

suspend fun main()= coroutineScope{

    launch{
        for(i in 0..5){
            delay(400L)
            println(i)
        }
    }
    launch{
        for(i in 6..10){
            delay(400L)
            println(i)
        }
    }

    println("Hello Coroutines")
}
```

Функция coroutineScope(), которая создает область корутин, будет ожидать завершения всех определенных в этой области корутин.
То есть функция main завршит выполнение, когда будут завершены обе корутины.

И в моем случае я получу следующий консольный вывод (данный вывод строго не детерминирован):

```

Hello Coroutines
6
0
7
1
8
2
9
3
10
4
5
```

### runBlocking

Кроме функции `coroutineScope` для создания контекста корутины может применяться функция runBlocking.

```

import kotlinx.coroutines.*

fun main() = runBlocking{
    launch{
        for(i in 0..5){
            delay(400L)
            println(i)
        }
    }

    println("Hello Coroutines")
}
```

Функция runBlocking блокирует вызывающий поток, пока все корутины внутри вызова runBlocking { ... } не завершат свое выполнение.
В этом собственно основное отличие runBlocking от coroutineScope: coroutineScope не блокирует вызывающий поток, а просто приостанавливает выполнение, освобождания
поток для использования другими ресурсами.

### Вложенные корутины

Одна корутина может содержать другие корутины. Например:

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    launch{
        println("Outer coroutine")
        launch{
            println("Inner coroutine")
            delay(400L)
        }
    }

    println("End of Main")
}
```

И подобным образом внешние корутины определяют область для вложенных корутин и управляют их жизненным циклом.

[Назад](./8.1.php)[Содержание](./)[Вперед](./8.3.php)