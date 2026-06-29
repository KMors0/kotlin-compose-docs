## Агрегатные операции

Последнее обновление: 24.01.2022

Коллекции/последовательности поддерживают ряд агрегатных операций, которые возвращают некоторое значение.

### Минимальное и максимальное значение

Функции minOrNull() и maxOrNull() возвращают соответственно минимальное и максимальное значение
(если коллекция/последовательность пута, то возвращается null).
Причем для работы этих функций элементы коллекции/последовательности должны реализовать интерфейс Comparable:

```

val numbers = listOf(4, 6, 3, 5, 1, 2)
val people = listOf("Alice", "Tom", "Sam", "Kate", "Bob")

println(numbers.minOrNull())   // 1
println(numbers.maxOrNull())   // 6

println(people.minOrNull())   // Alice
println(people.maxOrNull())   // Tom
```

Функции minByOrNull() и maxByOrNull() в качестве параметра принимают функцию селектора,
которая позволяет определить критерий сравнения объектов:

```

fun main(){

    val people = listOf( Person("Tom", 37), Person("Bob",41), Person("Sam", 25))
    // минимальный возраст
    val personWithMinAge = people.minByOrNull { it.age }
    println(personWithMinAge)     // Sam (25)
    // максимальный возраст
    val personWithMaxAge = people.maxByOrNull { it.age }
    println(personWithMaxAge)     // Bob (41)
}
class Person(val name: String, val age: Int){
    override fun toString(): String = "$name ($age)"
}
```

Выше функции находили элемент с наименьшим и наибольшим значением свойства age. Но что, если мы хотим получить не весь
объект, а сами значения минимального и максимального возраста? В этом случае мы можем использовать функции
minOfOrNull() и maxOfOrNull(), которые также принимают функцию селектора, но возвращает само значение,
на основе которого происходит сравнение:

```

fun main(){

    val people = listOf( Person("Tom", 37), Person("Bob",41), Person("Sam", 25))
    // минимальный возраст
    val minAge = people.minOfOrNull { it.age }
    println(minAge)     // 25
    // максимальный возраст
    val maxAge = people.maxOfOrNull { it.age }
    println(maxAge)     // 41
}
class Person(val name: String, val age: Int){
    override fun toString(): String = "$name ($age)"
}
```

Еще пара функций - minWithOrNull() и maxWithOrNull() в качестве параметра принимают компаратор -
реализацию интерфейса Comparator:

```

val colors = listOf("red", "green", "blue", "yellow")
val minColor = colors.minWithOrNull(compareBy {it.length})
val maxColor = colors.maxWithOrNull(compareBy {it.length})
println(minColor)   // red
println(maxColor)   // yellow
```

В качестве критерия сравнения здесь применяется свойство length строк, то есть строки сравниваются по длине.

И еще одна пара функций - minOfWithOrNull() и maxOfWithOrNull() принимают реализацию
интерфейса Comparator (первый параметр) и селектор критерия для сравнения (второй параметр):

```

maxOfWithOrNull(comparator: Comparator<in R>, selector: (T) -> R): R?
minOfWithOrNull(comparator: Comparator<in R>, selector: (T) -> R): R?
```

Применим данные функции:

```

fun main(){

    val people = listOf(
        Person("Tom", 37), Person("Kate",29),
        Person("Sam", 25), Person("Alice", 33)
    )
    // самое короткое имя
    val minName = people.minOfWithOrNull(compareBy{it.length}) { it.name }
    println(minName)     // Tom
    // самое длинное имя
    val maxName = people.maxOfWithOrNull(compareBy{it.length}) { it.name }
    println(maxName)     // Alice
}
class Person(val name: String, val age: Int){
    override fun toString(): String = "$name ($age)"
}
```

В качестве критерия сравнения здесь применяется свойство name объектов Person, так как функция селектора определена следующим образом: `{ it.name }` (здесь `it` - это текущий объект Person)

Поскольку в качестве критерия сравнения применяется свойство name, то в функцию компаратора передается значение этого свойства. И собственно
оно используется для сравнения объектов Person: `compareBy{it.length}` (здесь `it` - это значение свойства name)

### Среднее значение

Для получения среднего значения применяется функция average():

```

val numbers = listOf(4, 6, 3, 5, 1, 2)
val avg = numbers.average()
println(avg)    // 3.5
```

### Сумма значений

Для получения суммы числовых значений применяется функция sum():

```

val numbers = listOf(4, 6, 3, 5, 1, 2)
val sum = numbers.sum()
println(sum)    // 21
```

### Количество элементов

Для получения количества элементов в коллекции/последовательности применяется функция count(). Она имеет два варианта:

```

count(): Int
count(predicate: (T) -> Boolean): Int
```

Первая версия возвращает количество всех элементов. Вторая версия возвращает количество элементов, которые соответствуют условию предиката,
передаваемого в функцию в качестве параметра. Пример применения функций:

```

 val people = listOf("Tom", "Sam", "Bob", "Kate", "Alice")
// совокупное количество
val count1 = people.count()
println(count1)      // 5

// количество строк, у которых длина равна 3
val count2 = people.count{it.length == 3}
println(count2)      // 3
```

### Сведение элементов

Функция reduce() сводит все значения потока к одному значению:

```

educe(operation: (acc: S, T) -> S): S
```

reduce принимает функцию, которая имеет два параметра. Первый параметр при первом запуске представляет первый элемент, а при
последующих запусках - результат функции над предыдущими элементами. А второй параметр функции - следующий элемент.

Например, у нас есть список чисел, найдем сумму всех чисел:

```

val numbers = listOf(1, 2, 3, 4, 5)
val reducedValue = numbers.reduce{ a, b -> a + b }
println(reducedValue)   // 15
```

Здесь при первом запуске в функции в reduce параметр a равен 1, а параметр b равен 2.
При втором запуске параметр a содержит результат предыдущего выполнения функции, то если число 1 + 2 = 3, а параметр b равен 3 - следующее число в потоке.

Или другой пример со строками:

```

val people = listOf("Tom", "Bob", "Kate", "Sam", "Alice")
val reducedValue = people.reduce{ a, b -> "$a $b" }
println(reducedValue)   // Tom Bob Kate Sam Alice
```

Здесь `reduce` соединяет все строки в одну.

Функция fold также сводит все элементы потока в один. Но в отличие от `reduce` в качестве первого параметра принимает начальное значение:

```

fold(initial: R, operation: (acc: R, T) -> R): R
```

Например:

```

val people = listOf("Tom", "Bob", "Kate", "Sam", "Alice")
val foldedValue = people.fold("People:", { a, b -> "$a $b" })
println(foldedValue)   // People: Tom Bob Kate Sam Alice
```

В данном случае начальным значением является строка "People:", к которой затем добавляются остальные элементы списка.

[Назад](./7.11.php)[Содержание](./)[Вперед](./7.13.php)