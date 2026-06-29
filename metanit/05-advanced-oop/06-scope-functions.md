## Scope-функции

Последнее обновление: 03.03.2024

Scope-функции (можно перевести как "функции контекста" или "функции области видимости") позволяют
выполнить для некоторого объекта некоторый код в виде лямбда-выражение. При вызове подобной функции, создается временная область видимости.
В этой области видимости можно обращаться к объекту без использования его имени.

В Kotlin есть пять подобных функций: let, run, with, apply и also.
Эти функции похожи по своему действию и различаются прежде всего по параметрам и возвращаемым результатам

### let

Лямбда-выражение в функции let в качестве параметра it получает объект, для которого вызывается функция.
Возвращаемый результат функции let представляет результат данного лямбда-выражения.

```
inline fun <T, R> T.let(block: (T) -> R): R
```

Распространенным сценарием, где применяется данная функция, представляет проверка на null:

```

fun main() {

    val sam = Person("Sam", "sam@gmail.com")
    sam.email?.let{ println("Email: $it") }		// Email: sam@gmail.com
    // аналог без функции let
    //if(sam.email!=null) println("Email:${sam.email}")

    val tom = Person("Tom", null)
    tom.email?.let{ println("Email: $it") }	// функция let не будет выполняться

}
data class Person(val name: String, val email: String?)
```

Допустим, мы хотим вывести значение свойства Email объекта Person. Но это свойство может хранить значение null (например, если электронный адрес у пользователя не установлен).
С помощью выражения `sam.email?.` проверяем значение свойства `email` на `null`. Если email не равно null, то для строки в свойстве email вызывается
функция `let`, которая создает временную область видимости и передает в нее значение свойства email через параметр it.
Если же свойство email равно null, то функция let не вызывается.

Если лямбда-выражение вызывает лишь одну функцию, в которую передается параметр it, то можно сократить вызов - указать после оператора :: название вызываемой функции:

```

fun main() {

    val sam = Person("Sam", "sam@gmail.com")
    sam.email?.let(::println)   // sam@gmail.com

    val tom = Person("Tom", "tom@gmail.com")
    tom.email?.let(::printEmail)    // Email: tom@gmail.com

}

fun printEmail(email: String){
    println("Email: $email")
}
data class Person(val name: String, var email: String?)
```

### with

Лямбда-выражение в функции with в качестве параметра this получает объект, для которого вызывается функция.
Возвращаемый результат функции with представляет результат данного лямбда-выражения.

```
inline fun <T, R> with(receiver: T, block: T.() -> R): R
```

Функция with принимает объект, для которого надо выполнить блок кода, в качестве параметра. Далее в самом блоке кода мы можем
обращаться к общедоступным свойствам и методам объекта без его имени.

Обычно функция with() применяется, когда надо выполнить над объектом набор операций как единое целое:

```

fun main() {

    val tom = Person("Tom", null)
    val emailOfTom = with(tom) {
        if(email==null)
            email = "${name.lowercase()}@gmail.com"
        email
    }
    println(emailOfTom) // tom@gmail.com
}
data class Person(val name: String, var email: String?)
```

В данном случае функция with получает объект tom, поверяет его свойство email - если оно равно null, то устанавливает его на основе его имени. В качестве результата функции
возвращается значение свойства email.

Часто with применяется для установки свойств объектов. Например:

```

fun main() {

    val tom = Person("Tom", -19,null)
    with(tom) {
        if(email==null) email = "${name.lowercase()}@gmail.com"
        if(age !in 1..110) age = 18
    }
    println("${tom.name} (${tom.age}) - ${tom.email}") // Tom (18) - tom@gmail.com
}
data class Person(val name: String, var age:Int, var email: String?)
```

В данном случае функция with устанавливает свойства для объекта tom, если через конструктор им были переданы некорректные значения.

### run

Лямбда-выражение в функции run в качестве параметра this получает объект, для которого вызывается функция.
Возвращаемый результат функции run представляет результат данного лямбда-выражения.

```
inline fun <T, R> T.run(block: T.() -> R): R
```

Функция run похожа на функцию with за тем исключением, что run реализована как функция расширения.
Функция run также принимает объект, для которого надо выполнить блок кода, в качестве параметра. Далее в самом блоке кода мы можем
обращаться к общедоступным свойствам и методам объекта без его имени.

```

fun main() {

    val tom = Person("Tom", null)
    val emailOfTom = tom.run {
        if(email==null)
            email = "${name.lowercase()}@gmail.com"
        email
    }
    println(emailOfTom) // tom@gmail.com
}
data class Person(val name: String, var email: String?)
```

В данном случае функция `run` выполняет действия, аналогичные примеру с функцией `with`.

Реализация `run` как функции расширения упрощает проверку на null:

```

fun main() {

    val tom = Person("Tom", null)
    val validationResult = tom.email?.run {"valid"} ?: "undefined"
    println(validationResult) // undefined
}
data class Person(val name: String, var email: String?)
```

Также есть другая разновидность функции run(), которая просто позволяет выполнить некоторое лямбда-выражение:

```

fun main() {

    val randomText = run{ "hello world"}
    println(randomText)  // hello world

    run{ println("run function")}    // run function
}
```

### apply

Лямбда-выражение в функции apply в качестве параметра this получает объект, для которого вызывается функция.
Возвращаемым результатом функции фактически является передаваемый в функцию объект для которого выполняется функция.

```
inline fun T.apply(block: T.() -> Unit): T
```

Например:

```

fun main() {

    val tom = Person("Tom", null)
    tom.apply {
        if(email==null) email = "${name.lowercase()}@gmail.com"
    }
    println(tom.email) // tom@gmail.com
}
data class Person(val name: String, var email: String?)
```

В данном случае, как и в примерах с функциями `with` и `run`, проверяем значение свойства email, и если оно равно null, устанавливаем его,
используя свойство name.

Распространенным сценарием применения функции apply() является построение объекта в виде реализации вариации паттерна "Строитель":

```

fun main() {

    val bob = Employee()
    bob.name("Bob")
    bob.age(26)
    bob.company("JetBrains")
    println("${bob.name} (${bob.age}) - ${bob.company}") // Bob (26) - JetBrains
}

data class Employee(var name: String = "", var age: Int = 0, var company: String = "") {
    fun age(_age: Int): Employee = apply { age = _age }
    fun name(_name: String): Employee = apply { name = _name }
    fun company(_company: String): Employee = apply { company = _company }
}
```

В данном случае класс Employee содержит три метода, которые устанавливают каждое из свойств класса. Причем каждый подобный метод вызывает
функцию apply(), которое передает значение соответствующему свойству и возвращает текущий объект Employee.

### also

Лямбда-выражение в функции also в качестве параметра it получает объект, для которого вызывается функция.
Возвращаемым результатом функции фактически является передаваемый в функцию объект для которого выполняется функция.

```
inline fun T.also(block: (T) -> Unit): T
```

Эта функция аналогична функции `apply` за тем исключением, что внутри `also` объект, над которым выполняется блок кода, доступен
через параметр it:

```

fun main() {

    val tom = Person("Tom", null)
    tom.also {
        if(it.email==null)
            it.email = "${it.name.lowercase()}@gmail.com"
    }
    println(tom.email) // tom@gmail.com
}
data class Person(val name: String, var email: String?)
```

[Назад](./5.6.php)[Содержание](./)[Вперед](./5.4.php)