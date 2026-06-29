# Функции расширения с получателем

Последнее обновление: 27.06.2025

В Kotlin кроме собственно функций расширения применяются такие конструкции как **extension function with receiver** или функции расширения с получателем. Приведу пример
объявления такой функции:

```
fun some_func(block: T.() -> Unit)
```

Разберем это определение. Эта функция принимает параметр block, который представляет тип `T.() -> Unit`. Это синтаксис для функции расширения с получателем
(extension function with receiver). Рассмотрим его по частям:

* `T`: тип получателя. Это означает, что функция `block` будет выполняться как если бы она была методом экземпляра типа `T`. Внутри лямбда-выражения `block` ключевое слово `this` будет ссылаться на экземпляр типа `T`.
* `()`: указывает, что функция `block` не принимает никаких аргументов
* `-> Unit`: указывает, что функция `block` не возвращает никакого значения.

Таким образом, `T.() -> Unit` описывает тип функции, которая:

* Выполняется в контексте объекта типа `T`
* Не принимает никаких аргументов
* Не возвращает никакого значения

Этот паттерн очень часто используется в Kotlin для создания DSL (предметно-ориентированных языков) или для реализации строителей (builders).
Он позволяет писать код, который выглядит очень читабельно и декларативно.

Например, у нас есть класс `Person`:

```

class Person{
    var name: String = ""
    var age: Int = 0
    var company: String =""

    fun display(){
        print("Name: $name  Age: $age  Company: $company")
    }
}
```

Мы могли бы определить функцию для конфигурации объекта:

```

class Person{
    var name: String = ""
    var age: Int = 0
    var company: String =""

    fun display(){
        print("Name: $name  Age: $age  Company: $company")
    }
}

// Здесь Person является получателем. Внутри блока конфигурации
// this будет ссылаться на экземпляр Person.
fun Person.configure(block: Person.() -> Unit): Person {
    // Выполняем переданный блок конфигурации на текущем экземпляре Person (this)
    block()
    // Возвращаем настроенный экземпляр Person для цепочки вызовов
    return this
}

fun main() {
    val person1 = Person().configure {
        // Внутри этого блока this ссылается на person1
        name = "Tom" // То же самое, что и this.name = "Tom"
        age = 41
        company = "MyCorp"
    }

    person1.display() // Name: Tom  Age: 41  Company: MyCorp
}
```

Разберем определение функции `Person.configure`:

* `fun Person.configure`: Это объявление функции расширения для класса Person. Это означает, что теперь у любого объекта Person будет доступен метод configure.
* `block: Person.() -> Unit`: Это тип параметра block. Он определяет лямбда-функцию
* `Person.()`: Указывает, что эта лямбда является функцией расширения с получателем Person. Это значит, что внутри тела этой лямбды ({ ... }) можно напрямую обращаться к свойствам и методам объекта Person (например, name, age, display()) без необходимости префикса this.
* `-> Unit`: Обозначает, что эта лямбда не возвращает никакого значения.

Функция configure возвращает экземпляр Person (сам себя), что позволяет создавать цепочки вызовов.
Внутри функции configure мы просто вызываем переданный блок block. Поскольку block — это функция расширения для Person, она будет выполнена в контексте текущего экземпляра Person (который является this в функции configure).

Можно также использовать для настройки уже существующего объекта:

```

val person2 = Person()
person2.name = "Alice"

person2.configure {
    // Здесь this ссылается на person2
    age = 25
    company = "GlobalContest"
}
```

При наличие аналогичных методов, которые возвращают объект того же типа, можно вызывать функции по цепочке:

```

fun main() {
    val person1 = Person()

    person1.configure {
        name = "Tom"
    }.configure {   // вызываем цепочку функций, где настраиваем объект
        age = 41
    }.configure {
        company = "MyCorp"
    }

    person1.display() // Name: Tom  Age: 41  Company: MyCorp
}
```

Рассмотрим на более показательном примере - построении HTML, который наглядно показывает, как лямбды с получателем позволяют создавать вложенные, декларативные структуры.
Допустим, что мы хотим создать простой HTML-документ. Без лямбд с получателем это было бы громоздко. Но с ними мы можем "строить" HTML, как будто пишем его непосредственно в коде.

Сначала, давайте создадим очень упрощенные классы для представления HTML-тегов:

```

// Базовый класс для всех HTML-элементов
abstract class HtmlElement(val tagName: String) {
    val children = mutableListOf<HtmlElement>()
    var text: String = ""

    // Метод для добавления дочерних элементов
    fun <T : HtmlElement> doInit(element: T, init: T.() -> Unit) {
        element.init() // Выполняем лямбду-получатель для настройки дочернего элемента
        children.add(element)
    }

    // Метод для рендеринга HTML
    open fun render(indent: String = "", level: Int = 0): String {
        val currentIndent = indent.repeat(level)
        val childIndent = indent.repeat(level + 1)
        val childrenHtml = children.joinToString("") { it.render(indent, level + 1) }

        return buildString {
            append("$currentIndent<$tagName>")
            if (text.isNotEmpty()) {
                append(text)
            }
            if (childrenHtml.isNotEmpty()) {
                append("\n")
                append(childrenHtml)
                append("\n$currentIndent")
            }
            append("</$tagName>\n")
        }
    }
}

// Классы для конкретных HTML-тегов
class Html : HtmlElement("html") {
    fun head(init: Head.() -> Unit) = doInit(Head(), init)
    fun body(init: Body.() -> Unit) = doInit(Body(), init)
}

class Head : HtmlElement("head") {
    fun title(init: Title.() -> Unit) = doInit(Title(), init)
}

class Title : HtmlElement("title")

class Body : HtmlElement("body") {
    fun h1(init: H1.() -> Unit) = doInit(H1(), init)
    fun p(init: P.() -> Unit) = doInit(P(), init)
}

class H1 : HtmlElement("h1")
class P : HtmlElement("p")

// **********************************************
// Функция расширения с получателем для создания HTML
fun html(init: Html.() -> Unit): Html {
    val html = Html()
    html.init() // Вызываем лямбду-получатель для настройки корневого элемента Html
    return html
}
```

А теперь самое интересное — как мы используем эти функции расширения с получателем для создания HTML:

```

fun main() {
    val myHtmlDocument = html { // html {} - это вызов функции расширения с получателем.
                               // Внутри этого блока this - это объект Html.
        head { // head {} - это вызов метода head на объекте Html.
               // head сам принимает лямбду-получатель, где this - это объект Head.
            title { // Здесь this - это объект Title.
                text = "Моя первая страница на Kotlin DSL"
            }
        }
        body { // body {} - вызов метода body на Html. this - это Body.
            h1 { // h1 {} - вызов метода h1 на Body. this - это H1.
                text = "Добро пожаловать в Ktor HTML DSL!"
            }
            p { // p {} - вызов метода p на Body. this - это P.
                text = "Это демонстрация функций расширения с получателем."
            }
            p {
                text = "Они позволяют создавать очень выразительные и читабельные DSL."
            }
        }
    }

    println(myHtmlDocument.render(indent = "  "))
}
```

Основные моменты примера:

* Декларативность: код выглядит как фактическая структура HTML. Нет явных вызовов `new Html()`, `html.add(head)`, `head.add(title)`. Мы просто "декларируем", какой должна быть структура.
* Вложенность и Контекст: каждая вложенная лямбда (`head { ... }`, `body { ... }` ) предоставляет свой собственный контекст (получатель). Внутри `head { ... }` мы работаем с объектом `Head`, поэтому можно вызвать `title { ... }`.
  Внутри `body { ... }` мы работаем с `Body`, поэтому можно вызвать `h1 { ... }` или `p { ... }`. Компилятор Kotlin следит за этим, предоставляя вам автодополнение и типобезопасность.
* Исключительная читабельность: по сравнению с традиционным подходом, где мы бы вручную создавали объекты и добавляли их в списки, этот подход значительно более читабелен и интуитивен.
* Устранение "boilerplate" кода: множество повторяющихся вызовов `.add()` или присвоений убираются, поскольку все действия происходят внутри контекста получателя.

Вывод программы:

```

<html>
  <head>
    <title>Моя первая страница на Kotlin DSL</title>
</head>
  <body>
    <h1>Добро пожаловать в Ktor HTML DSL!</h1>
    <p>Это демонстрация функций расширения с получателем.</p>
    <p>Они позволяют создавать очень выразительные и читабельные DSL.</p>
</body>
</html>
```

[Назад](./5.5.php)[Содержание](./)[Вперед](./5.8.php)