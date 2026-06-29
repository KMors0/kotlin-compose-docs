## Группировка

Последнее обновление: 24.01.2022

Для группировки элементов коллекции/последовательности применяется функция groupBy():

```

groupBy(keySelector: (T) -> K): Map<K, List<T>>
groupBy(keySelector: (T) -> K,valueTransform: (T) -> V): Map<K, List<V>>
```

Обе версии в качестве параметра принимают функцию, которая определяет критерий группировки. Вторая версия в дополнение принимает функцию
преобразования. Результатом функции является объект Map, который хранит набор групп. Ключами являются критерии группировки - ключи групп,
а значениями - списки List, которые соответствуют этим критериям группировки и представляют группы.

Например, сгруппирует сотрудников по их компаниям:

```

fun main(){

    val employees = listOf(
        Employee("Tom", "Microsoft"),
        Employee("Bob", "JetBrains"),
        Employee("Sam", "Google"),
        Employee("Alice", "Microsoft"),
        Employee("Kate", "Google")
    )
    val companies = employees.groupBy { it.company }    // объект Map<String, List<Employee>>

    println(companies) // {Microsoft=[Tom, Alice], JetBrains=[Bob], Google=[Sam, Kate]}

    // перебор групп
    for (company in companies){
        println(company.key) // название компании
        // перебор списка сотрудников
        for (employee in company.value){
            println(employee.name)
        }
        println() // для отделения групп
    }
}
class Employee(val name: String, val company: String){
    override fun toString(): String = name
}
```

Здесь критерием группировки является свойство `company` объекта Employee. Соответственно ключом группы в результирующем объекте Map
будет значение этого свойства, а значением - список Employee. Затем можно получить все значения из каждой группы стандартным перебором объекта Map. Консольный вывод программы:

```

{Microsoft=[Tom, Alice], JetBrains=[Bob], Google=[Sam, Kate]}
Microsoft
Tom
Alice

JetBrains
Bob

Google
Sam
Kate
```

Теперь применим другую версию функции groupBy с использованием трансформации:

```

fun main(){

    val employees = listOf(
        Employee("Tom", "Microsoft"),
        Employee("Bob", "JetBrains"),
        Employee("Sam", "Google"),
        Employee("Alice", "Microsoft"),
        Employee("Kate", "Google")
    )
    val companies = employees.groupBy({it.company}) { it.name }  // объект Map<String, List<String>>

    println(companies) // {Microsoft=[Tom, Alice], JetBrains=[Bob], Google=[Sam, Kate]}
    // перебор групп
    for (company in companies){
        println(company.key) // название компании
        // перебор списка сотрудников
        for (employee in company.value){
            println(employee)
        }
        println() // для отделения групп
    }
}
class Employee(val name: String, val company: String){
    override fun toString(): String = name
}
```

В данном случае в качестве критерия группировки опять выступает свойство `company` объектов Employee (`{ it.company }`),
однако сама группа будет представлять список строк - объект `List<String>`, поскольку функция преобразования вытаскивает значение свойства
`name` объекта Employee: `{ it.name }`

[Назад](./7.9.php)[Содержание](./)[Вперед](./7.11.php)