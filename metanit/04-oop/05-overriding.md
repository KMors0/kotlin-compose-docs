## Модификаторы видимости

Последнее обновление: 03.03.2024

Все используемые типы, а также компоненты типов (классы, объекты, интерфейсы, конструкторы, функции, свойства) имеют определеннй уровень видимости,
определяемый модификатором видимости (модификатором доступа). Модификатор видимости определяет, где те или иные типы и их компоненты доступны и где их можно использовать.
В Kotlin есть следующие модификаторы видимости:

* private: классы, объекты, интерфейсы, а также функции и свойства, определенные вне класса, с этим модификатором видны только в том файле, в котором они определены.
  Члены класса с этим модификатором видны только в рамках своего класса
* protected: члены класса с этим модификатором видны в классе, в котором они определены, и в классах-наследниках
* internal: классы, объекты, интерфейсы, функции, свойства, конструкторы с этим модификатором видны в любой части модуля, в котором они определены. Модуль представляет набор файлов Kotlin, скомпилированных вместе в одну структурную единицу.
  Это может быть модуль IntelliJ IDEA или проект Maven
* public: классы, функции, свойства, объекты, интерфейсы с этим модификатором видны в любой части программы. (При этом если функции или классы с этим модификатором
  определены в другом пакете их все равно нужно импортировать)

Для установки уровня видимости модификатор ставится перед ключевыми словами `var/val/fun` в самом начале определения свойства или функции.

Если модификатор видимости явным образом не указан, то применяется модификатор public. То есть следующий класс

```

class Person(){

    var name = "Undefined"
    var age = 18
    
    fun printPerson(){
        println("Name: $name  Age: $age")
    }
}
```

Будет эквивалентен следующему определению класса:

```

class Person(){

    public var name = "Undefined"
    public var age = 18

    public fun printPerson(){
        println("Name: $name  Age: $age")
    }
}
```

Если свойства объявляются через первичный конструктор и для них явным образом не указан модификатор видимости:

```
class Person(val name: String, val age: Int){
	public fun printPerson(){
        println("Name: $name  Age: $age")
    }
}
```

То также к таким свойствам автоматически применяется `public`:

```
class Person(public val name: String, public val age: Int){
	public fun printPerson(){
        println("Name: $name  Age: $age")
    }
}
```

Соответственно мы можем обращаться к подобным компонентам класса в любом месте программы:

```

fun main() {

    val tom = Person("Tom", 37)
    tom.printPerson()       // Name: Tom    Age: 37

    println(tom.name)
	println(tom.age)
}
```

### private

Если же к свойствам и методам применяется модификатор private, то к ним нельзя будет обратиться извне - вне данного класса.

```

class Person(private val name:String, _age: Int){

    private val age = _age

    fun printPerson(){
        printName()
        printAge()
    }
    private fun printName(){
        println("Name: $name")
    }
    private fun printAge(){
        println("Age: $age")
    }
}

fun main() {

    val tom = Person("Tom", 37)
    tom.printPerson()
    
    // println(tom.name)   // Ошибка! - свойство name - private
    // tom.printAge()  // Ошибка! - функция printAge - private
}
```

В данном случае класс Person определяет два свойства `name` (имя человека) и `age` (возраст человека).
Чтобы было более показательно, одно свойство определено через конструктор, а второе как переменная класса. И поскольку эти свойства
определены с модификатором `private`, то мы можем к ним обращаться только внутри этого класса. Вне класса обращаться к ним нельзя:

```
println(tom.name)   // Ошибка! - свойство name - private
```

Также в классе определены три функции `printPerson()`, `printAge()` и `printName()`. Последние две функции выводят значения свойств.
А функция printPerson выводит информацию об объекте, вызывая предыдущие две функции.

Однако функции `printAge()` и `printName()` определены как приватные, поэтому их можно использовать только внутри класса:

```
tom.printAge()  // Ошибка! - функция printAge - private
```

### protected

Модификатор protected определяет свойства и функции, которые из вне класса видны только в классах-наследниках:

```

fun main() {

    val tom = Employee("Tom", 37)
    tom.printEmployee()       // Name: Tom    Age: 37

    // println(tom.name)   // Ошибка! - свойство name - protected
    // tom.printPerson()  // Ошибка! - функция printPerson - protected
}
open class Person(protected val name:String, private val age: Int){

     protected fun printPerson(){
        printName()
        printAge()
    }
    private fun printName(){
        println("Name: $name")
    }
    private fun printAge(){
        println("Age: $age")
    }
}
class Employee(name:String, age: Int) : Person(name, age){

    fun printEmployee(){
        println("Employee $name. Full information:")
        printPerson()
        // printName() // нельзя - printName - private
        // println("Age: $age")    // нельзя age - private
    }
}
```

Здесь в классе Person свойство `name` определенно как `protected`, поэтому оно доступно в классе-наследнике Employee (однако вне базового и производного класса - например, в функции main оно
недоступно). А вот свойство `age` - приватное, поэтому оно доступно только внутри класса Person.

Также в классе Employee будет доступна функция `printPerson()`, так как она имеет модификатор protected, а функции `printAge()` и
`printName()` с модификатором `private` будут недоступны.

### Модификаторы конструкторов

Конструкторы как первичные, так и вторичные также могут иметь модификаторы. Модификатор указывается перед ключевым словом constructor.
По умолчанию они имеют модификатор `public`. Если для первичного конструктора необходимо явным образом установить
модификатор доступа, то конструктор определяется с помощью ключевого слова constructor:

```

fun main() {

    // val bob  = Person("Bob")    // Так нельзя - конструктор private
}
open class Person private constructor(val name:String){

     fun printPerson(){
        println("Name: $name")
    }
}
// class Employee(name:String) : Person(name)  // так нельзя - конструктор в Person private
```

Стоит отметить, что в данном случае, поскольку конструктор приватный мы не можем его использовать вне класса ни для создания объекта класса в функции main,
ни при наследовании. Но мы можем использовать такой конструктор в других конструкторах внутри класса:

```

fun main() {

    val tom = Employee("Tom", 37)
    tom.printPerson()
}
open class Person private constructor(val name:String){

    var age: Int = 0
    protected constructor(_name:String, _age: Int): this(_name){    // вызываем приватный конструктор
        age = _age
    }
     fun printPerson(){
        println("Name: $name Age: $age")
    }
}
class Employee(name:String, age: Int) : Person(name, age)
```

Здесь вторичный конструктор класса Person, который имеет модификатор `protected` (то есть доступен в текущем классе и классах-наследниках)
вызывает первичный конструктор класса Person, который имеет модификатор `private`.

### Модификаторы объектов и типов верхнего уровня

Классы, а также переменные и функции, которые определены вне других классов, также могут иметь модификаторы public, private и internal.

Допустим, у нас есть файл base.kt, который определяет одноименный пакет:

```

package base

private val privateVal = 3
internal val internalVal = 4
val publicVal = 5

private fun privateFun() = println("privateFn")
fun internalFun() = println("internalFn")
fun publicFun() = println("publicFn")

private class PrivateClass(val name: String)
internal class InternalClass(val name:String)
class PublicClass(val name:String)

fun printData(){
    // внутри модуля доступны приватные идентификаторы
    val privateClass= PrivateClass("Tom")
    println(privateVal)
    privateFun()

    // внутри модуля доступны internal-идентификаторы
    val internalClass= InternalClass("Tom")
    println(internalVal)
    internalFun()

    // внутри модуля доступны public-идентификаторы
    val publicClass= PublicClass("Tom")
    println(publicVal)
    publicFun()
}
```

Внутри данного файла мы можем использовать его приватные переменные, функции классы. Однако при подключении этого пакета в другие файлы, приватные переменные,
функции и классы будут недоступны:

```

import base.*

fun main() {

    publicFun()
    val publicClass= PublicClass("Tom")
    println(publicVal)


    // privateFun()                         // функция недоступна
    // val privateClass= PrivateClass("Tom")    // класс недоступен
    // println(privateVal)      // переменная недоступна

    // если в одном модуле, то internal-компоненты доступны
    internalFun()                         // функция доступна
    val internalClass= InternalClass("Tom")    // класс доступен
    println(internalVal)      // переменная доступна
}
```

Однако даже внутри одного файла есть ограничения на использование приватных классов:

```

package email

private class Message(val text: String)

fun send(message: Message, address : String){
    println("Message `${message.text}` has been sent to $address")
}
```

Здесь мы столкнемся с ошибкой, так как публичная функция не может принимать параметр приватного класса. И в данном случае нам надо либо сделать класс
Message публичным, либо функцию send приватной.

[Назад](./4.10.php)[Содержание](./)[Вперед](./4.2.php)