## Преобразование данных. Функции map и transform

Последнее обновление: 28.06.2021

### Функция map

Оператор map() преобразует данные потока. В качестве параметра он принимает функцию преобразования. Функция преобразования принимает
в качестве единственного параметра объект из потока и возвращает преобразованные данные.

```

inline fun <T, R> Flow<T>.map(
    crossinline transform: suspend (value: T) -> R
): Flow<R> (source)
```

Рассмотрим на примере:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Sam", 41),
        Person("Bob", 21)
    ).asFlow()

    peopleFlow.map{ person -> person.name }
        .collect { personName -> println(personName) }
}

data class Person(val name: String, val age: Int)
```

В данном случае определяем поток данных `peopleFlow`, который содержит объекты типа `Person`. Далее к этому потоку применяется функция
map(), в которую передается следующая функция преобразования:

```
person -> person.name
```

То есть функция преобразования принимает в качестве параметра `person` объект типа Person и возвращает значение его поля `name` - то есть строку.
Таким образом, функция `map()` из потока объектов Person создаст поток обектов String.

Далее к полученному потоку объектов String применяем функцию `collect()`, в которой получаем каждую строку из потока и выводим ее на консоль:

```
collect { personName -> println(personName) }
```

Консольный вывод программы:

```

Tom
Sam
Bob
```

Другой пример:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Bill", 5),
        Person("Sam", 14),
        Person("Bob", 21),
    ).asFlow()

    peopleFlow.map{ person -> object{
        val name = person.name
        val isAdult = person.age > 17
    }}.collect { user -> println("name: ${user.name}   adult:  ${user.isAdult} ")}
}

data class Person(val name: String, val age: Int)
```

В данном случае функция map преобразует поток данных типа Person в поток объектов анонимного типа, который определяет два поля: name (имя) и isAdult (больше ли пользователю 17 лет).
Соответственно в функции `collect` мы получаем объекты этого анонимного типа и выводим их данные на консоль. Консольный вывод:

```

name: Tom   adult:  true 
name: Bill   adult:  false 
name: Sam   adult:  false 
name: Bob   adult:  true 
```

### Функция transform

Оператор transform также позволяет выполнять преобразование объектов в потоке. В отличие от map она позволяет использовать
функцию `emit()`, чтобы передавать в поток произвольные объекты.

```

inline fun <T, R> Flow<T>.transform(
    crossinline transform: suspend FlowCollector<R>.(value: T) -> Unit
): Flow<R> (source)
```

Оператор `transform` принимает функцию, которая получает в качестве параметра объект потока. Например:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val peopleFlow = listOf(
        Person("Tom", 37),
        Person("Bill", 5),
        Person("Sam", 14),
        Person("Bob", 21),
    ).asFlow()

    peopleFlow.transform{ person ->
        if(person.age > 17){
            emit(person.name)
        }
    }.collect { personName -> println(personName)}
}

data class Person(val name: String, val age: Int)
```

В данном случае получаем в функции transform объект Person из потока. Если поле `age` этого объекта больше 17, то передаем его в новый поток
через функцию emit(). Консольный вывод программы:

```

Tom
Bob
```

Причем при обработке одного объекта мы можем несколько раз вызывать функцию `emit()`, передавая таким образом в потоке несколько объектов:

```

import kotlinx.coroutines.flow.*

suspend fun main(){

    val numbersFlow = listOf(2, 3, 4).asFlow()
    numbersFlow.transform{ n ->
        emit(n)
        emit(n * n)
    }.collect { n -> println(n)}
}
```

Например, здесь преобразуем поток чисел. Причем в выходной поток передается само число и его квадрат. Таким образом, на консоли мы увидим:

```

2
4
3
9
4
16
```

[Назад](./9.5.php)[Содержание](./)[Вперед](./9.7.php)