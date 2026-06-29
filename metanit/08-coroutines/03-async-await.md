## Async, await и Deferred

Последнее обновление: 21.06.2021

Наряду с `launch` в пакете `kotlinx.coroutines` есть еще один построитель корутин - функция async.
Эта функция применяется, когда надо получить из корутины некоторый результат.

async запускает отдельную корутину, которая выполняется параллельно с остальными корутинами. Например:

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    async{ printHello()}
    println("Program has finished")
}
suspend fun printHello(){
    delay(500L)  // имитация продолжительной работы
    println("Hello work!")
}
```

Консольный вывод программы:

```

Program has finished
Hello work!
```

Кроме того, async-корутина возвращает объект
Deferred, который ожидает получения результата корутины. (Интерфейс Deferred унаследован от интерфейса
Job, поэтому для также доступны весь функционал, определенный для интефейса Job)

Для получения результата из объекта Deferred применяется функция await(). Рассмотрим на примере:

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    val message: Deferred<String> = async{ getMessage()}
    println("message: ${message.await()}")
    println("Program has finished")
}
suspend fun getMessage() : String{
    delay(500L)  // имитация продолжительной работы
    return "Hello"
}
```

В данном случае для имитации продолжительной работы определена функция `getMessage()`, которая возвращает строку.

С помощью функции async запускаем корутину, которая выполняет эту функцию.

```
async{ getMessage()}
```

Поскольку функция `getMessage()` возвращает объект типа `String`, то возвращаемый корутиной объект представляет тип
`Deferred<String>` (объект Deferred типизиуется возвращаемым типом функции, то есть в данном случае типом String).

```
val message: Deferred<String> = async{ getMessage()}
```

Далее у объекта Deferred для получения результата функции `getMessage()` вызываем метод await(). Он ожидает, пока не будет
получен результат. Консольный вывод программы:

```

message: Hello
Program has finished
```

Поскольку функция `getMessage()` возвращает объект типа String, то метод await() в данном случае также будет возвращать строку,
которую мы могли бы, например, присвоить переменной:

```
val text: String = message.await()
```

При этом мы можем с помощью `async` запустить несколько корутин, которые будут выполняться параллельно:

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    val numDeferred1 = async{ sum(1, 2)}
    val numDeferred2 = async{ sum(3, 4)}
    val numDeferred3 = async{ sum(5, 6)}
    val num1 = numDeferred1.await()
    val num2 = numDeferred2.await()
    val num3 = numDeferred3.await()

    println("number1: $num1  number2: $num2  number3: $num3")
}
suspend fun sum(a: Int, b: Int) : Int{
    delay(500L) // имитация продолжительной работы
    return a + b
}
```

Здесь запускается три корутины, каждая из которых выполняет функцию `sum()`. Эта функция складывает два числа и возвращает их сумму в виде объекта `Int`.
Поэтому корутины возвращают объект `Deferred<Int>`. Соответственно вызов метода `await()` у этого объекта возвратит объект `Int`, то есть
сумму двух чисел. При этом все три корутины будет запущены одновременно. Например, ниже на скриншоте отладчика корутин видно, что две корутины уже работают (или находятся в состоянии
`Running`), и еще одна корутина только создана и ожидает запуска (состояние `Created`)

![Coroutine async and await in Kotlin](./pics/coroutine5.png)

### Отложенный запуск

По умолчанию построитель корутин async создает и сразу же запускает корутину. Но как и при создании корутины с помощью
`launch` для async-корутин можно применять технику отложенного запуска. Только в данном случае корутина запускается не только
при вызове метода start объекта Deferred (который усналедован от интерфейса Job), но также и с помощью
метода await() при обращении к результу корутины. Например:

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    // корутина создана, но не запущена
    val sum = async(start = CoroutineStart.LAZY){ sum(1, 2)}

    delay(1000L)
    println("Actions after the coroutine creation")
    println("sum: ${sum.await()}")   // запуск и выполнение корутины
}
fun sum(a: Int, b: Int) : Int{
    println("Coroutine has started")
    return a + b
}
```

Консольный вывод программы:

```

Actions after the coroutine creation
Coroutine has started
sum: 3
```

Если необходимо, чтобы корутина еще до метода `await()` начала выполняться, то можно вызвать метод start():

```

import kotlinx.coroutines.*

suspend fun main() = coroutineScope{

    // корутина создана, но не запущена
    val sum = async(start = CoroutineStart.LAZY){ sum(1, 2)}

    delay(1000L)
    println("Actions after the coroutine creation")
    sum.start()     				 // запуск корутины
    println("sum: ${sum.await()}")   // получаем результат
}
fun sum(a: Int, b: Int) : Int{
    println("Coroutine has started")
    return a + b
}
```

[Назад](./8.3.php)[Содержание](./)[Вперед](./8.7.php)