## Set

Последнее обновление: 04.03.2024

Интерфейс Set представляет неупорядоченный набор объектов, который хранит только уникальные объекты.
Интерфейс Set представляет неизменяемый (immutable) набор. Set расширяет интерфейс `Collection` и соответственно все его методы.

Для создания неизменяемого (immutable) набора используется функция setOf().

```

val numbers = setOf(5, 6, 7)            // объект Set<Int>
val people = setOf("Tom", "Sam", "Bob") // объект Set<String>
```

Для перебора Set можно применять цикл for:

```

 val people = setOf("Tom", "Sam", "Bob")

for(person in people) println(person)
println(people) // [Tom, Sam, Bob]
```

Set реализует метод toString таким образом, что возвращает в удобочитабельном виде все элементы в виде строки

Причем, поскольку Set представляет набор уникальных объектов, то даже если мы передадим через функцию `setOf()` повторяющиеся значения,
то в наборе все равно будут только уникальные значения:

```

val numbers = setOf(5, 6, 7, 5, 6)
val people = setOf("Tom", "Sam", "Bob", "Tom")

println(numbers) // [5, 6, 7]
println(people) // [Tom, Sam, Bob]
```

Для преобразования других коллекций к типу множества Kotlin предоставляет функцию toSet(). Например, у нас есть список, из которого мы хотим удалить повторяющиеся значения.
И самым простым способом сделать это будет преобразование списка к множеству:

```

fun main() {

    val people = listOf("Tom", "Bob", "Sam", "Tom", "Bob", "Alex")
    val uniquePeople = people.toSet()
    println(uniquePeople)  // [Tom, Bob, Sam, Alex]
}
```

### Методы Set

Рассмотрим некоторые специфичные операции Set. Прежде всего это методы для операций с множествами:

* union: объединение множеств
* intersect: пересечение множеств (возвращает элементы, которые есть в обоих множествах)
* subtract: вычитание множеств (возвращает элементы, которые есть в первом множестве, но отсутствуют во втором)

Хотя эти операции могут применяться и к спискам List, но возвращают они всегда объект Set и более уместны для множеств Set.

```

val people = setOf("Tom", "Sam", "Bob", "Mike")
val employees = setOf("Tom", "Sam", "Kate", "Alice")

//  объединение множеств
val all = people.union(employees)
// пересечение множеств
val common = people.intersect(employees)
// вычитание множеств
val different = people.subtract(employees)

println(all)        // [Tom, Sam, Bob, Mike, Kate, Alice]
println(common)     // [Tom, Sam]
println(different)  // [Bob, Mike]
```

Причем данные методы можно применять как обычные операции:

```

//  объединение множеств
val all = people union employees
// пересечение множеств
val common = people intersect employees
// вычитание множеств
val different = people subtract employees
```

### Изменяемые коллекции

Изменяемые (mutable) наборы представлены интерфейсом MutableSet, который расширяет интерфейсы `Set` и
`MutableCollection`, соответственно поддерживает методы по изменению коллекции.

Для создания изменяемых (mutable) наборов применяется функция mutableSetOf().

```

val numbers: MutableSet<Int> = mutableSetOf(35, 36, 37)
```

Интерфейс MutableSet реализуется следующими типами изменяемых наборов:

* LinkedHashSet: объединяет возможности хеш-таблицы и связанного списка. Создается с помощью функции
  linkedSetOf().
* HashSet: представляет хеш-таблицу. Создается с помощью функции
  hashSetOf().

```

val numbers1: HashSet<Int> = hashSetOf(5, 6, 7)
val numbers2: LinkedHashSet<Int> = linkedSetOf(25, 26, 27)
val numbers3: MutableSet<Int> = mutableSetOf(35, 36, 37)
```

Изменение набора с помощью `MutableSet`:

```

val numbers: MutableSet<Int> = mutableSetOf(35, 36, 37)
println(numbers.add(2))
println(numbers.addAll(setOf(4, 5, 6)))
println(numbers.remove(36))
    
for (n in numbers){ println(n) }	// 35 37 2 4 5 6
numbers.clear()
```

[Назад](./7.2.php)[Содержание](./)[Вперед](./7.4.php)