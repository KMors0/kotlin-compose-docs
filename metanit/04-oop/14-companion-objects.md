## Companion-объекты

Последнее обновление: 28.03.2024

Класс в языке Kotlin может содержать так называемые companion-объекты. companion-объект определяется внутри некоторого класса и
позволяет определить свойства и методы, которые будут общими для всех объектов этого класса. В ряде языков программирования есть похожая концепция - статические поля/свойства и методы.
То есть companion-объекты определяют свойства и методы класса в целом, а не объекта.

Общий синтаксис опредедения companion-объекта:

```

class ClassName{

    // свойства и методы класса
    companion object {

        // свойства и методы companion-объекта
    }

}
```

Например, нам надо подсчитать, сколько было создано объектов определенного класса. Для этого определим следующую программу:

```

class Person(val name: String){
  
    init{
        counter++
    }
    companion object {
        var counter = 0
    }
}
fun main (){
    println(Person.counter) // 0
    Person("Tom")
    println(Person.counter) // 1
    Person("Bob")
    Person("Sam")
    println(Person.counter) // 3

}
```

Здесь в классе Person определен следующий companion-объект:

```

companion object {
    var counter = 0
}
```

Для его определения применяется ключевое слово companion. Внутри этого объекта определено свойство counter - счетчик, который по умолчанию равен 0.

В теле класса Person мы можем напрямую обращаться к свойствам и методам companion-объекта. В данном случае при создании объекта в инициализаторе мы увеличиваем этот счетчик на 1:

```

init{
    counter++
}
```

То есть при создании каждого объекта счетчик увеличивается на единицу. И поскольку companion-объекты определяют свойства и методы, общие для всех объектов класса,
то этот счетчик будет один общий для всех объектов класса Person. И для обращения к нему вне класса применяется имя класса, а не объекта:

```

println(Person.counter) // 0
Person("Tom")
println(Person.counter) // 1
Person("Bob")
Person("Sam")
println(Person.counter) // 3
```

И здесь мы видим, что изначально счетчик равен 0. После создания одного объекта он равен 1, а после создания еще двух объектов - 3.

Если же мы попробуем обратиться к переменной counter через имя объекта, то мы получим ошибку:

```

val tom = Person("Tom")
println(tom.counter) // Ошибка
```

Теоретически мы, конечно, можем определить и для объектов одноименное свойство. Например:

```

class Person(val name: String){
  
    var counter = 22
    init{
        Person.counter++
    }
    companion object {
        var counter = 0
    }
}
fun main (){
    println(Person.counter) // 0
    val tom = Person("Tom")
    println(tom.counter) // 22
    Person("Bob")
    Person("Sam")
    println(Person.counter) // 3
}
```

Здесь в классе Person определено свойство counter, которое по умолчанию равно 22:

```
var counter = 22
```

И для обращения к этому свойству мы можем использовать имя объекта:

```

val tom = Person("Tom")
println(tom.counter) // 22
```

В этом случае, если внутри класса мы хотим обратиться не к этому свойству, а к свойству companion-объекта, то мы также должны указывать имя класса:

```

init{
    Person.counter++
}
```

### Статические методы

Companion-объекты также могут определять методы, общие для всего класса и для обращения к которым также используется имя класса, а не объекта. Например, в примерах выше у нас
определенно есть проблема с целостностью данных, так как мы можем из внешнего кода присвоить счетчику любое значение:

```
Person.counter = -100500
```

Естественно это нежелательная ситуация. Поэтому опосредуем доступ к свойству counter:

```

class Person(val name: String){
  
    init{
        counter++
    }
    companion object {
        private var counter = 0
        fun printCounter() = println(counter)
    }
}
fun main (){
    Person.printCounter() // 0
    Person("Tom")
    Person.printCounter() // 22
    Person("Bob")
    Person("Sam")
    Person.printCounter() // 3
}
```

Итак, переменная counter теперь приватная, она недоступна вне своего класса. Однако внутри класса мы по прежнему можем к ней обращаться.

А для того, чтобы внешний код также мог получить ее значение, в companion-объекте определен метод printCounter:

```
fun printCounter() = println(counter)
```

Для доступа к этому методу применяется имя класса:

```
Person.printCounter()
```

### Наследование

Методы и свойства companion-объекта не наследуются, поэтому для обращения к ним применяется имя базового класса, в котором определен companion-объект:

```

open class Person(val name: String){
  
    init{
        counter++
    }
    companion object {
        private var counter = 0
        fun printCounter() = println(counter)
    }
}
class Employee(name:String):Person(name)

fun main (){
    Person.printCounter() // 0
    Employee("Tom")
    Person.printCounter() // 1
    // Employee.printCounter() ! Так нельзя - error: unresolved reference: printCounter
}
```

[Назад](./4.14.php)[Содержание](./)[Вперед](./6.1.php)