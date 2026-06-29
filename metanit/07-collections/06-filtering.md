## Фильтрация

Последнее обновление: 04.03.2024

### Фильтрация по условию

Фильтрация является одной из распространенных операций. Для фильтрации по условию применяется функция filter(),
которая в качестве параметра принимает условие-предикат в виде функции `(T) -> Boolean`.

```
filter(predicate: (T) -> Boolean): List<T>/Map<K, V>/Sequence<T>
```

Функция предиката принимает в качестве параметра элемент набора. Если элемент соответствует условию, то возвращается true, а данный
элемент помещается в возвращаемый набор.

Для коллекций List и Set эта функция возвращает объект List, для Map - объект Map, для последовательностей Sequence - также объект Sequence:

```

fun main(){

    var people = sequenceOf("Tom", "Sam", "Mike", "Bob", "Alice")
    people = people.filter{it.length == 3}    // получаем значения с длиной в 3 символа
    println(people.joinToString())  // Tom, Sam, Bob

    var employees = listOf(
        Person("Tom", 37),
        Person("Bob", 41),
        Person("Sam", 25)
    )
    employees = employees.filter{it.age > 30} // получаем всех Person, у которых age > 30
    println(employees.joinToString())  // Person(name=Tom, age=37), Person(name=Bob, age=41)
}
data class Person(val name: String, val age: Int)
```

Если надо получить элементы, которые, наоборот, НЕ соответствует условию, то можно применить функцию filterNot(),
которая работает аналогично:

```

var people = sequenceOf("Tom", "Sam", "Mike", "Bob", "Alice")
people = people.filterNot{it.length == 3}    // получаем значения с длиной, не равной 3 символам
println(people.joinToString())  // Mike, Alice
```

### Фильтрация по индексу

Еще одна функция - filterIndexed() также получает индекс текущего элемента:

```
filterIndexed(predicate: (index: Int, T) -> Boolean): List<T>
```

Например, получим из коллекции строк элементы с четными индексами и длиной в 3 символа:

```

val people = listOf("Tom", "Mike", "Sam", "Bob", "Alice")
// получаем значения с длиной в 3 символа на четных индексах
val filtered = people.filterIndexed{ index, s -> (index % 2 == 0) && (s.length == 3)}
println(filtered)  // [Tom, Sam]
```

### Фильтрация по типу

Если коллекция/последовательность содержит элементы разных типов, то с помощью функции filterIsInstance()
можно извлечь элементы определенного типа. Например:

```

fun main(){

    val people = listOf(
        Person("Tom"), Employee("Bob"),
        Person("Sam"), Employee("Mike")
    )
    // получаем только элементы типа Employee
    val employees = people.filterIsInstance<Employee>();
    println(employees)  // [Bob, Mike]
}
open class Person(val name: String){
    override fun toString(): String  = name
}
class Employee(name: String): Person(name)
```

В данном случае получаем из коллекции people только те объекты, которые представляют тип Employee. Чтобы указать тип получаемых объектов,
при вызове функция типизируется соответствующим типом.

### Фильтрация по null

Функция filterNotNull() позволяет выфильтровать все значение, которые равны null:

```
filterNotNull(): List<T>/Sequence<T>
```

Например:

```

fun main(){

    val people = listOf(Person("Tom"), null, Person("Sam"), null)
    println(people)     // [Tom, null, Sam, null]
    val filtered = people.filterNotNull()
    println(filtered)  // [Tom, Sam]
}
open class Person(val name: String){
    override fun toString(): String  = name
}
```

[Назад](./7.6.php)[Содержание](./)[Вперед](./7.8.php)