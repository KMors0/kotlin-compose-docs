## Каналы

Последнее обновление: 25.06.2021

Каналы позволяют передавать потоки данных. В Kotlin каналы представлены интерфейсом Channel, у которого следует выделить два основных метода:

* `abstract suspend fun send(element: E): Unit`

  Отправляет объект `element` в канал
* `abstract suspend fun receive(): E`

  Получает данные из канала

Определим простейший канал, через который будем передавать числа типа Int:

```

import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

suspend fun main() = coroutineScope{

    val channel = Channel<Int>()
    launch {
        for (n in 1..5) {
            // отправляем данные через канал
            channel.send(n)
        }
    }
	
    // получаем данные из канала
    repeat(5) {
        val number = channel.receive()
        println(number)
    }
    println("End")
}
```

Основная функциональность каналов сосредочена в пакете kotlinx.coroutines.channels, соответственно импортируем из него тип Channel:

```
import kotlinx.coroutines.channels.Channel
```

В программе определяем переменную, которая будет представлять канал:

```
val channel = Channel<Int>()
```

Поскольку через канал будут передаваться значения типа `Int`, то соответственно объект Channel типизирован типом `Int`.

Затем с помощью функции `launch` создаем корутину, в которой в цикле передаем в канал числа от 1 до 5:

```

launch {
	for (n in 1..5) {
		// отправляем данные через канал
		channel.send(n)
	}
}
```

В метод send() собственно передается то значение, которое мы хотим отправить через канал. Особенностью этого метода является то, что мы
можем его запустить только в корутине.

Для получения данных из канала с помощью функции `repeat()` определяем функцию, которая будет выполнятся 5 раз - так как мы передаем в канал пять чисел:

```

repeat(5) {
	val number = channel.receive()	// получаем значения из канала
	println(number)
}
```

Метод receive() возвращает извлекаемый из канала объект.

Консольный вывод программы:

```

1
2
3
4
5
End
```

Другой пример - отправка через канал строк:

```

import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

suspend fun main() = coroutineScope{

    val channel = Channel<String>()
    launch {
        val users = listOf("Tom", "Bob", "Sam")
        for (user in users) {
            println("Sending $user")
            channel.send(user)
        }
    }

    repeat(3) {
        val user = channel.receive()
        println("Received: $user")
    }
    println("End")
}
```

В даном случае в канал передаем три строки, соответственно функция `repeat()` три раза запускает получение данных из канала. Консольный вывод программы:

```

Sending Tom
Received: Tom
Sending Bob
Received: Bob
Sending Sam
Received: Sam
End
```

### Закрытие канала

Чтобы указать, что в канале больше нет данных, его можно закрыть с помощью метода close().
Если для получения данных из канала применяется цикл for, то, получив сигнал о закрытии канала,
данный цикл получит все ранее посланные объекты до закрытия и завершит выполнение:

```

import kotlinx.coroutines.*
import kotlinx.coroutines.channels.Channel

suspend fun main() = coroutineScope{

    val channel = Channel<String>()
    launch {
        val users = listOf("Tom", "Bob", "Sam")
        for (user in users) {
            channel.send(user)	// Отправляем данные в канал
        }
        channel.close()  // Закрытие канала
    }

    for(user in channel) {	// Получаем данные из канала
        println(user)
    }
    println("End")
}
```

### Паттерн producer-consumer

Рассмотренный выше пример по сути является распростаненным способом передачи данных от одной корутины к другой. И чтобы упростить написание подобного кода,
Kotlin предоставляет ряд дополнительных функций. Так, функция produce() представляет построитель корутины, который создает корутину, в которой передаются данные в канал.
Например, с помощью функции `produce()` мы можем определить новую функцию-корутину, которая будет отправлять определенные данные:

```

fun CoroutineScope.getUsers(): ReceiveChannel<String> = produce{
    val users = listOf("Tom", "Bob", "Sam")
    for (user in users) {
        send(user)
    }
}
```

Здесь определяется функция `getUsers()`. Причем она определяется как функция интерфейса `CoroutineScope`. Функция должна возвращать
объект ReceiveChannel, типизированный типом передаваемых данных (в данном случае передаем значения типа String).

Функция `getUsers()` представляет корутину, создаваемую построителем корутин `produce`. В корутине опять же проходим по списку строк и
с помощью функции send передаем в канал данные.

Для потребления данных из канала может применяться метод consumeEach() объекта `ReceiveChannel`, который по сути заменяет цикл `for`. Он принимает функцию, в
которую в качестве параметра передается получаемый из канала объект:

```

val users = getUsers()
users.consumeEach { user -> println(user) }
```

Полный код программы:

```

import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

suspend fun main() = coroutineScope{

    val users = getUsers()
    users.consumeEach { user -> println(user) }
    println("End")
}

fun CoroutineScope.getUsers(): ReceiveChannel<String> = produce{
    val users = listOf("Tom", "Bob", "Sam")
    for (user in users) {
        send(user)
    }
}
```

Консольный вывод программы:

```

Tom
Bob
Sam
End
```

[Назад](./8.5.php)[Содержание](./)[Вперед](./9.1.php)