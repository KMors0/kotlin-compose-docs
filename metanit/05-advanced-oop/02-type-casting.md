## Преобразование типов

Последнее обновление: 04.03.2024

Нередко может возникать задача по преобразованию типов, например, чтобы использовать данные одного типа в констексте, где требуются данные другого типа.
В этом случае Kotlin представляет ряд возможностей по преобразованию типов.

### Встроенные методы преобразования типов

Для преобразования данных одного типа в другой можно использовать встроенные следующие функции, которые есть у базовых типов (Int, Long, Double и т.д.) (Конкретный набор функций для разных базовых типов может отличаться. ):

* toByte
* toShort
* toInt
* toLong
* toFloat
* toDouble
* toChar

Все эти функции преобразуют данные в тот тип, которые идет после префикса to: toByte.

```

val s: String = "12"
val d: Int = s.toInt()
println(d)
```

В данном случае строка s преобразуется в число d. Просто так передать строку переменной типа Int, мы не можем, несмотря на то, что вроде бы строка
и содержит число 12:

```
val d: Int = "12"	// ! Ошибка
```

Однако надо учитывать, что значение не всегда может быть преобразовано к определенному типу. И в этом случае генерируется исключение. Соответственно
в таких случаях желательно отлавливать исключение:

```

val s: String = "tom"
try {
	val d: Int = s.toInt()
	println(d)
}
catch(e: Exception){
	println(e.message)
}
```

### Smart cast и оператор is

Оператор is позволяет проверить выражение на принадлежность определенному типу данных:

```
значение is тип_данных
```

Этот оператор возвращает `true`, если значение слева от оператора принадлежит типу, указанному справа от оператора. Если локальная переменная или свойство успешно пройдет проверку
на принадлежность определенному типу, то далее нет нужды дополнительно приводить значение к этому типу.
Данные преобразования еще называются smart casts или "умные преобразования".

Данный оператор можно применять как к базовым типам, но к собственным классам и интерфейсам, которые находятся в одной и той же иерархии наследования.

```

fun main() {

    val tom = Person("Tom")
    val bob = Employee("Bob", "JetBrains")

    checkEmployment(tom)    // Tom does not have a job
    checkEmployment(bob)    // Bob works in JetBrains
}

fun checkEmployment(person: Person){
	// println("${person.name} works in ${person.company}")    // Ошибка - у Person нет свойства company
    if(person is Employee){
        println("${person.name} works in ${person.company}")
    }
    else{
        println("${person.name} does not have a job")
    }
}
open class Person(val name: String)
class Employee(name: String, val company: String): Person(name)
```

Здесь класс Employee наследуется от класса Person. В функции `checkEmployment()` получаем объект Person. С помощью оператора `is` проверяем,
представляет ли он тип Employee (так как не каждый объект Person может представлять тип Employee).
Если он представляет тип Employee, то выводим название его компании, если он не представляет тип Employee, то выводим, сообщение, что он безработный.

Причем даже если значение представляет тип Employee, то до применения оператора is оно тем не менее принадлежит типу Person. И только
применение оператора `is` преобразует значение из типа Person в тип Employee.

Также можно применять другую форму оператора - !is. Она возвращает `true`, если значение НЕ представляет указанный тип данных:

```

fun main() {

    val tom = Person("Tom")
    val bob = Employee("Bob", "JetBrains")

    checkEmployment(tom)    // Tom does not have a job
    checkEmployment(bob)    // Bob works in JetBrains
}

fun checkEmployment(person: Person){
	// println("${person.name} works in ${person.company}")    // Ошибка - у Person нет свойства company
    if(person !is Employee){
        println("${person.name} does not have a job")
    }
    else{
		println("${person.name} works in ${person.company}")
    }
}
open class Person(val name: String)
class Employee(name: String, val company: String): Person(name)
```

Однако, что, если свойство `company` имеет пустую строку, например, `val bob = Employee("Bob", "")`, то есть фактически компания
не указана. А мы хотим выводить компанию, если это свойство имеет какое-нибудь содержимое. В этом случае мы можем выполнить проверку на длину строку сразу же после применения оператора `is`:

```

fun checkEmployment(person: Person){
    if(person is Employee && person.company.length > 0){
        println("${person.name} works in ${person.company}")
    }
    else{
        println("${person.name} does not have a job")
    }
}
```

в выражении

```
person.company.length > 0){
```

компилятор уже видит, что person - это объект типа `Employee`, поэтому позволяет обращаться к его свойствам и функциям.

Если необходимо определить различные действия в зависимости от типа объекта, то удобно использовать конструкцию when:

```

fun identifyPerson(person: Person){
    when(person){
        is Manager -> println("${person.name} is a manager")
        is Employee -&t println("${person.name} is an employee")
        is Person -> println("${person.name} is just a person")
    }
}

open class Person(val name: String)
open class Employee(name: String, val company: String): Person(name)
class Manager(name: String, company: String): Employee(name, company)
```

#### Ограничения умных преобразований

Подобные smart-преобразования тем не менее имеют ограничения. Они могут применяться, только если
компилятор может гарантировать, что переменная не изменила своего значения в промежутке между проверкой и использованием.
Для smart-преобразований действуют следующие правила:

1. smart-преобразования применяются к локальным val-переменным (за исключением делегированных свойств)
2. smart-преобразования применяются к val-свойствам, за исключением свойств с модификатором `open`
   (то есть открытых к переопределению в производных классах) или свойств, для которых явным образом определен геттер
3. smart-преобразования применяются к локальным var-переменным (то есть к переменным, определенным в функциях), если
   переменная не изменяет своего значения в промежутке между проверкой и использованием и не используется в лямбда-выражении, которое изменяет ее, а также не является локальным делегированным свойством
4. к var-свойствам smart-преобразования не применяются

Рассмотрим некоторые случаи. Возьмем последнее правило про var-свойства:

```

fun main() {

    val tom = Person("Tom")

    if(tom.phone is SmartPhone){

        println("SmartPhone: ${tom.phone.name}, OS: ${tom.phone.os}")	// ! Ошибка
    }
    else{
        println("Phone: ${tom.phone.name}")
    }
}
open class Phone(val name: String)
class SmartPhone(name: String, val os: String) : Phone(name)

open class Person(val name: String){
    var phone: Phone = SmartPhone("Pixel 5", "Android")
}
```

Здесь класс Person хранит var-свойство класса Phone, которому присваивается объект класса SmartPhone. Соответственно в выражении:

```
if(tom.phone is SmartPhone){
```

очевидно, что свойство `tom.phone` представляет класс SmartPhone, однако поскольку это свойство определено с помощью var, то к нему
не применяются smart-преобразования. То есть если в данном случае мы заменим `var` на val, то у нас проблем не возникнет.

Или второе правило - изменим класс Person, определив для свойства геттер:

```

open class Person(val name: String){
    val phone: Phone
        get()  = SmartPhone("Pixel 5", "Android")
}
```

В соответствии с вторым правилом опять же к такому свойству не применяются smart-преобразования, так как оно имеет геттер.

### Явные преобразования и оператор as

С помощью оператора as мы можем приводить значения одного типа к другому типу:

```
значение as тип_данных
```

Слева от оператора указывается значение, а справа - тип данных, в которых надо преобразовать значение. Например, преобразуем значение типа `String?` в тип
`String`:

```

fun main() {

    val hello: String? = "Hello Kotlin"
    val message: String = hello as String
    println(message)
}
```

Здесь переменная `hello` хранит строку и может быть удачно преобразована в значение типа `String`. Однако если переменная hello равна `null`:

```

val hello: String? = null
val message: String = hello as String
println(message)
```

В этом случае преобразование завершится неудачно - ведь значение null нельзя преобразовать в значение типа `String`. Поэтому будет
сгенерировано исключение NullPointerException.

Чтобы избежать генерации исключения мы можем применять более безопасную версию оператора as?, которая в случае неудачи
преобразования возвращает null.

```


val hello: String? = null
    val message: String? = hello as? String
    println(message)
```

В данном случае оператор `as?` возвратит null, так как null нельзя преобразовать в строку.

Также можно применять данный оператор к преобразованиям своих типов:

```

fun main() {

    val tom = Person("Tom")
    val bob = Employee("Bob", "JetBrains")
    checkCompany(tom)
    checkCompany(bob)
}
fun checkCompany(person: Person){
    val employee = person as? Employee
    if (employee!=null){
        println("${employee.name} works in ${employee.company}")
    }
    else{
        println("${person.name} is not an employee")
    }
}
open class Person(val name: String)
open class Employee(name: String, var company: String): Person(name)
```

Здесь функция `checkCompany()` принимает объект класса Person и пытается преобразовать его в объект типа Employee, который унаследован от Employee.
Но если каждый объект Employee представляет также объект Person (каждый работник является человеком), то не каждый объект Person представляет объект Employee
(не каждый человек является работником). И в этом случае чтобы получить значение типа Employee, применяется оператор as?. Если объект
person представляет тип Employee, то возвращается объект этого типа, иначе возвращается `null`. И далее мы можем проверить на значение null и выполнить те или иные действия.

Консольный вывод программы:

```

Tom is not an employee
Bob works in JetBrains
```

[Назад](./5.1.php)[Содержание](./)[Вперед](./5.5.php)