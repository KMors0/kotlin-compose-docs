## Трансформации

Последнее обновление: 04.03.2024

### map

Для трансформации одной коллекции/последовательности применяется функция map()

```

map(transform: (T) -> R): List<R>/Sequence<R>
```

В качестве параметра она принимает функцию пребразования. Функция пребразования получает текущий элемент коллекции/последовательности и
возвращает результат преобразования. Причем типа входных и выходных данных могут совпадать, а могут и отличаться. Например:

```

fun main(){

    val people = listOf(Person("Tom"), Person("Sam"), Person("Bob"))

    val names = people.map { it.name } // возвращаем имя каждого пользователя
    println(names)  // [Tom, Sam, Bob]
}
class Person(val name: String)
```

Здесь из списка List<Person> получаем список List<String>, который содержит имена пользователей.

Другой пример - трансформируем последовательность чисел в последовательность квадратов этих чисел:

```

val numbers = listOf(1, 2, 3, 4, 5)
val squares = numbers.map { it * it }
println(squares)    // [1, 4, 9, 16, 25]
```

Еще одна функция - mapIndexed() также передает в функцию преобразования индекс текущего элемента:

```

mapIndexed(transform: (index: Int, T) -> R): List<R>
```

Применение функции:

```

fun main(){

    val people = listOf(Person("Tom"), Person("Sam"), Person("Bob"))

    val names = people.mapIndexed{ index, p-> "${index+1}.${p.name}"}
    println(names)  // [1.Tom, 2.Sam, 3.Bob]
}
class Person(val name: String)
```

Если необходимо отсеять значения null, которые могут возникать при преобразовании, то можно применять аналоги выше упомянутых функций -
mapNotNull() и mapIndexedNotNull():

```

fun main(){

    val people = listOf(
        Person("Tom"), Person("Sam"),
        Person("Bob"), Person("Alice")
    )
    // элементы длиной имени не равной 3 преобразуем в null
    val names1 = people.mapNotNull{ if(it.name.length!=3) null else it.name }

    // элементы на четных позициях преобразуем в null
    val names2 = people.mapIndexedNotNull{ index, p -> if(index%2==0) null else p.name }

    println(names1)  // [Tom, Sam, Bob]
    println(names2)  // [Sam, Alice]
}
class Person(val name: String)
```

### flatten

Функция flatten() позволяет преобразовать коллекцию/последовательность, которая содержит вложенные коллекции/последовательности:

```

flatten(): List<T>/Sequence<T>
```

Эта функция помещает элементы всех вложенных коллекций в одну:

```

val personal = listOf(listOf("Tom", "Bob"), listOf("Sam", "Mike", "Kate"), listOf("Tom", "Bill"))
val people = personal.flatten()
println(people)    // [Tom, Bob, Sam, Mike, Kate, Tom, Bill]
```

[Назад](./7.8.php)[Содержание](./)[Вперед](./7.10.php)