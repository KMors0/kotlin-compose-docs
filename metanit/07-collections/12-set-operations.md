## Сложение, вычитание и объединение коллекций

Последнее обновление: 24.01.2022

С помощью функции plus можно складывать коллекции/последовательности с другими элементами или коллекциями/последовательностями.
Данная функция принимает как одиночный элемент, так и коллекцию/последовательность:

```

fun main(){
    val people = listOf("Tom", "Bob", "Sam")
    // добавляем один объект 
    val result1 = people.plus("Alice")
    println(result1)  // [Tom, Bob, Sam, Alice]

    val employees = listOf("Mike", "Kate")
    // добавляем коллекцию объектов
    val result2 = people.plus(employees)
    println(result2)  // [Tom, Bob, Sam, Mike, Kate]
}
```

Обратите внимание, что начальная коллекция, у которой вызывается функция `minus()` (в примере выше коллекция people), не изменяется, результатом объединения является новая коллекция.

Вместо функции `plus()` можно использовать оператор +

```

fun main(){
    val people = listOf("Tom", "Bob", "Sam")
    val result1 = people + "Alice"
    println(result1)  // [Tom, Bob, Sam, Alice]

    val employees = listOf("Mike", "Kate")
    val result2 = people + employees
    println(result2)  // [Tom, Bob, Sam, Mike, Kate]
}
```

Аналогичным образом можно использовать функцию minus() для вычитания либо одиночного объекта, либо коллекции/последовательности:

```

fun main(){
    val people = listOf("Tom", "Bob", "Sam", "Kate")
    // вычитаем один объект
    val result1 = people.minus("Bob")
    println(result1)  // [Tom, Sam, "Kate"]

    val employees = listOf("Mike", "Kate")
    // вычитаем коллекцию
    val result2 = people.minus(employees)
    println(result2)  // [Tom, Bob, Sam]
}
```

Начальная коллекция, у которой вызывается функция `minus()`, также не изменяется, результатом вычитания является новая коллекция.

Вместо функции `minus()` можно использовать оператор -

```

val people = listOf("Tom", "Bob", "Sam", "Kate")
val result1 = people - "Bob"
println(result1)  // [Tom, Sam, "Kate"]

val employees = listOf("Mike", "Kate")
val result2 = people - employees
println(result2)  // [Tom, Bob, Sam]
```

### Объединение

Для объединения двух разных коллекций/последовательностей в одну применяется функция zip():

```

zip(other: Iterable<R>/Sequence<R>): List<Pair<T, R>>/Sequence<Pair<T, R>>
```

В качестве параметра она принимает другую коллекцию/последовательность и возвращает набор из объектов типа Pair

Например, объединим два списка в один:

```

val english = listOf("red", "blue", "green")
val russian = listOf("красный", "синий", "зеленый")
val words = english.zip(russian)

for(word in words)
    println("${word.first}: ${word.second}")
```

Здесь каждому элементу из списка english сопоставляется элемент на соответствующей позиции из списка russian. Результатом функции будет
список из объектов `Pair<String, String>`, где свойство `first` хранит значение из текущей коллекции, а свойство `second`
- значение из коллекции, переданной в качестве параметра:

```

red: красный
blue: синий
green: зеленый
```

Если в одной из коллекций меньше элементов, чем в другой, то сопоставляется столько элементов, сколько в наименьшей коллекции.

Функция zip() имеет еще одну версию, которая в качестве второго параметра получает функцию преобразования,
применяемую к элементам обоих коллекций/последовательностей:

```

zip(other: Iterable<R>,transform: (a: T, b: R) -> V): List<V>
zip(other: Sequence<R>,transform: (a: T, b: R) -> V): Sequence<V>
```

Пример использования:

```

val english = listOf("red", "blue", "green")
val russian = listOf("красный", "синий", "зеленый")
val words = english.zip(russian){en, ru -> "$en - $ru"}

for(word in words) println(word)
```

Консольный вывод:

```

red - красный
blue - синий
green - зеленый
```

Также в Kotlin есть обратная функция - unzip, которая позволяет обратно получить две коллекции

```
unzip(): Pair<List<T>, List<R>>
```

Пример использования:

```

val dictionary = listOf("red", "blue", "green")
        .zip(listOf("красный", "синий", "зеленый"))
val words = dictionary.unzip()
println(words.first)    // [red, blue, green]
println(words.second)   // [красный, синий, зеленый]
```

[Назад](./7.12.php)[Содержание](./)[Вперед](./7.14.php)