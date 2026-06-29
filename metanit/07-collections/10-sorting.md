## Сортировка

Последнее обновление: 11.12.2024

Для сортировки коллекции/последовательности применяются функции `sorted()` (сортировка по возрастанию) и
`sortedDescending()` (сортировка по убыванию).

Что в данном случае значит сортировка по возрастанию или убыванию? По умолчанию процесс сортировки опирается на реализацию интерфейса Comparable,
которая определяет, какой объект будет больше, а какой меньше. Так, для встроенных базовых типов действует следующая логика:

* Числа сравниваются как в математике исходя из их значения
* Символы (Char) и строки (String) сравниваются исходя из лексикографического порядка, то есть "Tom" больше, чем "Alice", потому что первый символ - T располагается в алфавите после символа A.

Сортировка чисел и строк:

```

val people = listOf("Tom", "Mike", "Bob", "Sam", "Alice")
val numbers = listOf(3, 5, 2, -4, -6, 9, 1)

// сортировка по возрастанию
val sortedPeople = people.sorted()
val sortedNumbers = numbers.sorted()
println(sortedPeople)   // [Alice, Bob, Mike, Sam, Tom]
println(sortedNumbers)  // [-6, -4, 1, 2, 3, 5, 9]

// сортировка по убыванию
println(people.sortedDescending())  // [Tom, Sam, Mike, Bob, Alice]
println(numbers.sortedDescending()) // [9, 5, 3, 2, 1, -4, -6]
```

### Реализация интерфейса Comparable

Если мы определяем свои типы и хотим, чтобы их объекты также можно было отсортировать, то в этом случае следует реализовать
интерфейс Comparable:

```

ublic interface Comparable<in T> {
    public operator fun compareTo(other: T): Int
}
```

Его функция `compareTo()` должна определять логику сравнения. В качестве параметра она принимает объект, который сравнивается с текущим.

В качестве результата возвращается число. Если текущий объект равен объекту other, то функция должна возвратить 0. Если текущий объект
больше объекта other, то возвращается положительное число, если меньше - то отрицательное.

Посмотрим на примере реализацию интерфейса:

```

fun main(){

    val people = listOf(
        Person("Tom", 37),
        Person("Bob",41),
        Person("Sam", 25)
    )
    // сортировка по возрастанию
    val sortedPeople = people.sorted()

    // сортировка по  возрастанию
    println(sortedPeople)   // [Bob (41), Sam (25), Tom (37)]

    // сортировка по убыванию
    println(people.sortedDescending())  // [Tom (37), Sam (25), Bob (41)]
}
class Person(val name: String, val age: Int): Comparable<Person> {
    override fun compareTo(other: Person): Int = name.compareTo(other.name)
    override fun toString(): String = "$name ($age)"
}
```

В данном случае класс Person реализует интерфейс Comparable, однако в самом методе compareTo фактически мы полагаемся на реализацию этого
интерфейса для строк. То есть мы фактически возвращаем результат сравнения двух строк - имен пользователей:

```
name.compareTo(other.name)
```

В итоге объекты Person будут сортироваться в соответствии с расположением первых букв их имен в лексикографического порядке -
стандартная сортировка для строк.

Теперь изменим принцип сортировки и сравним два объекта по их возрасту - значению свойства age:

```

fun main(){

    val people = listOf(
        Person("Tom", 37),
        Person("Bob",41),
        Person("Sam", 25)
    )
    // сортировка по возрастанию
    val sortedPeople = people.sorted()

    // сортировка по  возрастанию
    println(sortedPeople)   // [Sam (25), Tom (37), Bob (41)]

    // сортировка по убыванию
    println(people.sortedDescending())  // [Bob (41), Tom (37), Sam (25)]
}
class Person(val name: String, val age: Int): Comparable<Person> {
    override fun compareTo(other: Person): Int = age - other.age
    override fun toString(): String = "$name ($age)"
}
```

Тепрь объект Person считается "больше", если у него больше значение свойства age.

### sortedWith и интерфейс Comparator

Интерфейс Comparable позволяет легко определить логику сортировки, однако бывает, что нам недоступен код классов, объекты которых мы хотим отсортировать. Либо мы хотим задать
для уже существующих типов, которые уже реализуют Comparable, другой способ сортировки. В этом случае мы можем
использовать интерфейс Comparator (грубо говоря компаратор):

```

public expect fun interface Comparator<T> {
    public fun compare(a: T, b: T): Int
}
```

Его функция compare() принимает два параметра - два сравниваемых объекта и также возращает целое число. Если первый параметр
больше второго, то возвращается положительное число, если меньше - то отрицательное. Если объекты равно, возвращается 0.

Kotlin имеет встроенную функцию sortedWith(), которая в качестве параметра принимает компаратор и на его основе
сортирует коллекцию/последовательность:

```

sortedWith(comparator: Comparator<in T>): List<T>
```

Пример реализации:

```

fun main(){

    val people = listOf(
        Person("Tom", 37),
        Person("Bob",41),
        Person("Sam", 25)
    )
    val personComparator = Comparator{ p1: Person, p2: Person -> p1.age - p2.age }

    val sortedPeople = people.sortedWith(personComparator)
    println(sortedPeople)   // [Sam (25), Tom (37), Bob (41)]
}
class Person(val name: String, val age: Int){
    override fun toString(): String = "$name ($age)"
}
```

В данном случае компаратор определен в виде переменной personComparator. В реализации интерфейса как и в предыдущем примере сравниваем
пользователей на основе возраста: если значение свойства `age` больше, то и объект Person условно считается "больше".

В принципе в данном случае мы можем сократить вызов функции сортировки:

```

val sortedPeople = people.sortedWith{ p1: Person, p2: Person -> p1.age - p2.age }
```

или так:

```

val sortedPeople = people.sortedWith(compareBy{ it.age })
```

Благодаря компаратору можно задать свою логику сортировки к уже имеющимся типам. Например, отсортируем строки по длине:

```

fun main(){

    val people = listOf("Tom", "Alice", "Kate", "Sam", "Bob")
    // отсортируем по длине строки
    val sorted  = people.sortedWith(compareBy{ it.length })
    println(sorted)   // [Tom, Sam, Bob, Alice]
}
```

### Сортировка на основе критерия

Еще две функцию позволяют отсортировать объекты по определенному критерию:

```

sortedBy(crossinline selector: (T) -> R?): List<T> / Sequence<T>
sortedByDescending(crossinline selector: (T) -> R?): List<T> / Sequence<T>
```

Функция sortedBy() сортирует по возрастанию, а sortedByDescending() - по убыванию. В качестве параметра
они принимают функцию, которая получает элемент и возвращает значение, применяемое для сортировки.

```

fun main(){

    val people = listOf( Person("Tom", 37), Person("Bob",41), Person("Sam", 25))

    // сортировка по свойству name по возрастанию 
    val sortedByName = people.sortedBy { it.name }
    println(sortedByName)   // [Bob (41), Sam (25), Tom (37)]

    // сортировка по свойству age по возрастанию 
    val sortedByAge = people.sortedBy { it.age }
    println(sortedByAge)   // [Sam (25), Tom (37), Bob (41)]

    // сортировка по свойству age по убыванию 
    val sortedByAgeDesc = people.sortedByDescending { it.age }
    println(sortedByAgeDesc)   // [Bob (41), Tom (37), Sam (25)]
}
class Person(val name: String, val age: Int){
    override fun toString(): String = "$name ($age)"
}
```

### reverse и shaffle

Стоит отметить еще две функции, которые не сортируют, а просто меняют порядок элементов.
Функция reversed() изменяет порядок элементов на обратный, а функция shuffle() перемешивает элементы случайным
образом:

```

val numbers = listOf(1, 2, 3, 4, 5, 6)
val reversed = numbers.reversed()
println(reversed)   // [6, 5, 4, 3, 2, 1]

val shuffled = numbers.shuffled()
println(shuffled)   // [4, 6, 3, 5, 1, 2]
```

[Назад](./7.10.php)[Содержание](./)[Вперед](./7.12.php)