## Вариантность, ковариантность и контравариантность

Последнее обновление: 04.03.2024

Вариантность описывает, как обобщенные типы, типизированные классами из одной иерархии наследования, соотносятся друг с другом.

### Инвариантность

Инвариантность предполагает, что, если у нас есть классы Base и Derived, где Derived - производный класс от Base, то класс C<Base> не
является ни базовым классом для С<Derived>, ни производным. Например, у нас есть следующие типы:

```

interface Messenger<T: Message>()

open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```

В данном случае мы не можем присвоить объект `Messenger<EmailMessage>` переменной типа `Messenger<Message>` и наоборот, они никак между собой
не соотносятся, несмотря на то, что EmailMessage наследуется от Message:

```

fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<Message> = obj	// ! Ошибка
}
fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<EmailMessage> = obj		// ! Ошибка
}
```

Мы можем присвоить переменным по умолчанию только объекты их типов:

```

fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<Message> = obj
}
fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<EmailMessage> = obj
}
```

### Ковариантость

Ковариантость предполагает, что, если у нас есть классы Base и Derived, где Base - базовый класс для Derived, то класс SomeClass<Base>
является базовым классом для SomeClass<Derived>

Для определения обобщенного типа как ковариантного параметр обощения определяется с ключевым словом out:

```

interface Messenger<out T: Message>
open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```

В данном случае интерфейс Messenger является ковариантным, так как его параметр определен со словом out:

```
interface Messenger<out T>
```

И теперь переменной типа `Messenger<Message>` мы можем присвоить значение типа `Messenger<EmailMessage>`

```

fun changeMessengerToEmail(obj: Messenger<EmailMessage>){
    val messenger: Messenger<Message> = obj
}
```

Вообще не случайно используется именно слово out. Оно указывает, что обобщенный тип может возвращать из функции значение типа T:

```

fun main() {

    val messenger: Messenger<Message> = EmailMessenger()
    val message = messenger.writeMessage("Hello Kotlin")
    println(message.text)    // Email: Hello Kotlin
}
open class Message(val text: String)
class EmailMessage(text: String): Message(text)

interface Messenger<out T: Message>{
    fun writeMessage(text: String): T
}
class EmailMessenger(): Messenger<EmailMessage>{
    override fun writeMessage(text: String): EmailMessage {
        return EmailMessage("Email: $text")
    }
}
```

В данном случае обобщенный интерфейс Messenger определяет функцию `writeMessage()` для генерации объекта Message. Класс EmailMessenger применяет интерфейс Messenger и реализует
эту функцию. То есть в данном случае тип EmailMessenger по сути представляет тип `Messenger<EmailMessage>`.

Поскольку в Messenger параметр T определен с аннотацией out, то мы можем присвоить переменной типа `Messenger<Message>`
значение типа `EmailMessenger` (а по сути значение типа `Messenger<EmailMessage>`)

```
val messenger: Messenger<Message> = EmailMessenger()
```

В то же время тип T нельзя использовать в качестве типа входных параметров функции. Например, в следующем случае компилятор известит нас об ошибке:

```

interface Messenger<out T: Message>{
    fun writeMessage(text: String): T
    fun sendMessage(message: T)		// Ошибка - тип T может представлять только возвращемый тип
}
```

### Контравариантность

Контравариантость предполагает в какой-то степени обратную ситуацию. Контравариантость предполагает, что, если у нас есть классы Base и Derived, где
Base - базовый класс для Derived, то объекту SomeClass<Derived> мы можем присвоить значение SomeClass<Base> (при ковариантности, наоборот, - объекту
SomeClass<Base> можно присвоить значение SomeClass<Derived>)

Для определения обобщенного типа как контравариантного параметр обобщения определяется с ключевым словом in:

```

interface Messenger<in T: Message>
open class Message(val text: String)
class EmailMessage(text: String): Message(text)
```

В данном случае интерфейс Messenger является контравариантным, так как его параметр определен со словом in:

```
interface Messenger<in T>
```

И теперь переменной типа `Messenger<EmailMessage>` мы можем присвоить значение типа `Messenger<Message>`

```

fun changeMessengerToDefault(obj: Messenger<Message>){
    val messenger: Messenger<EmailMessage> = obj
}
```

Применение аннотации in означает, что обобщенный тип может получать значение типа T через параметр функции:

```

fun main() {

    val messenger: Messenger<EmailMessage> = InstantMessenger() // InstantMessenger - это Messenger<Message>

    val message = EmailMessage("Hi Kotlin")
    messenger.sendMessage(message)
}
open class Message(val text: String)
class EmailMessage(text: String): Message(text)

interface Messenger<in T: Message>{
    //fun writeMessage(text: String): T
    fun sendMessage(message: T)
}

class InstantMessenger(): Messenger<Message>{
    override fun sendMessage(message: Message){
        println("Send message: ${message.text}")
    }
}
```

В данном случае обобщенный интерфейс Messenger определяет функцию `sendMessage()`, которая принимает объект Message в качестве параметра.
Класс InstantMessenger применяет интерфейс Messenger и реализует эту функцию. То есть в данном случае тип InstantMessenger по сути представляет тип
`Messenger<Message>`.

Поскольку в интерфейсе Messenger параметр T определен с аннотацией in, то мы можем присвоить переменной типа
`Messenger<EmailMessage>`
значение типа `InstantMessenger` (то есть значение типа `Messenger<Message>`)

```
val messenger: Messenger<EmailMessage> = InstantMessenger()
```

В то же время тип T нельзя использовать в качестве типа результата функции. Например, в следующем случае компилятор известит нас об ошибке:

```

interface Messenger<in T: Message>{
    fun writeMessage(text: String): T	// Ошибка - тип T может представлять только параметр функции
    fun sendMessage(message: T)	
}
```

[Назад](./6.2.php)[Содержание](./)[Вперед](./5.2.php)