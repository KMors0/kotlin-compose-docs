## List

Последнее обновление: 04.03.2024

List представляет последовательный список элементов. При этом List представляет неизменяемую (immutable) коллекцию, которая в основном
только обеспечивает получение элементов по позиции.

Интерфейс List расширяет интерфейс Collection, поэтому перенимает его возможности.

Для создания объекта List применяется метод listOf():

```

var numbers = listOf(1, 2, 3, 4, 5)     // объект List<Int>
val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")   // объект List<String>
```

Списки поддерживают перебор с помощью цикла for, кроме для списка по умолчанию задача реализация toString, которая выводит все элементы списка в
удобочитаемом виде:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")

for(person in people) println(person)
println(people) // [Tom, Sam, Kate, Bob, Alice]
```

#### Методы списков

Кроме унаследованных методов класс List имеет ряд специфичных. Рассмотрим некоторые из них.

Для получения элемента по индексу можно применять метод get(index), который возвращает элемент по индексу

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
val first = people.get(0)
val second = people.get(1)
println(first)      // Tom
println(second)     // Sam
```

Вместо метода get для обращения по индексу можно использовать квадратные скобки []:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
val first = people[0]
val second = people[1]
println(first)      // Tom
println(second)     // Sam
```

Однако, если индекс выходит за границы списка, то при использовании метода `get()` и квадратных скобок генерируется исключение.
Чтобы избежать подобной ситуации, можно применять метод getOrNull(), который возвращает null, если индекс находится вне границ списка:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
val first = people.getOrNull(0)
val tenth = people.getOrNull(10)
println(first)      // Tom
println(tenth)     // null
```

Либо в качестве альтернативы можно применять метод getOrElse():

```

getOrElse(index: Int, defaultValue: (Int) -> T): T
```

Первый параметр представляет индекс, а второй параметр - функция, которая получает запрошенный индекс и возвращает значение, которое возвращается, если индекс выходит
за границы списка:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice")
val first = people.getOrElse(0){"Undefined"}
val seventh = people.getOrElse(7){"Invalid index $it"}
val tenth = people.getOrElse(10){"Undefined"}
    
println(first)      // Tom
println(seventh)    // Invalid index 7
println(tenth)     // Undefined
```

#### Получение части списка

Метод subList() возвращает часть списка и в качестве параметров принимает начальный и конечный индексы извлекаемых элементов:

```
subList(fromIndex: Int, toIndex: Int): List<E>
```

Например, получим подсписок с 1 по 4 индексы:

```

val people = listOf("Tom", "Sam", "Kate", "Bob", "Alice", "Mike")
val subPeople = people.subList(1, 4)
println(subPeople)      // [Sam, Kate, Bob]
```

### Изменяемые списки

Изменяемые списки представлены интерфейсом MutableList. Он расширяет интерфейс List и позволяют добавлять и удалять элементы.
Данный интерфейс реализуется классом ArrayList.

Для создания изменяемых списков можно использовать ряд методов:

* arrayListOf(): создает объект ArrayList
* mutableListOf(): создает объект MutableList

Создание изменяемых списков:

```

var numbers : ArrayList<Int> = arrayListOf(1, 2, 3, 4, 5)
var numbers2: MutableList<Int> = mutableListOf(5, 6, 7)
```

Если необходимо добавлять или удалять элементы, то надо использовать методы MutableList:

* add(index, element): добавлят элемент по индексу
* add(element): добавляет элемент
* addAll(collection): добавляет коллекцию элементов
* remove(element): удаляет элемент
* removeAt(index): удаляет элемент по индексу
* clear(): удаляет все элементы коллекции

```

fun main() {

    val numbers1 : ArrayList<Int> = arrayListOf(1, 2, 3, 4, 5)
    numbers1.add(4)
    numbers1.clear()
	
    val numbers2: MutableList<Int> = mutableListOf(5, 6, 7)

    numbers2.add(12)
    numbers2.add(0, 23)
    numbers2.addAll(0, listOf(-3, -2, -1))
    numbers2.removeAt(0)
    numbers2.remove(5)

    for (n in numbers2){ println(n) }
}
```

[Назад](./7.1.php)[Содержание](./)[Вперед](./7.3.php)