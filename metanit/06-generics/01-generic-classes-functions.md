## Ограничения обобщений

Последнее обновление: 04.03.2024

Ограничения обобщений (generic constraints) ограничивают набор типов, которые могут передаваться вместо параметра в обобщениях.

Например, мы хотим определить универсальную функцию для сравнения двух объектов и возвращать из функции наибольший объект. На первый взгляд мы можем просто определить обобщенную функцию:

```

fun <T> getBiggest(a: T, b: T): T{
    if(a > b) return a	// ! Ошибка
    return b
}
```

Но компилятор не скомпилирует эту функцию, потому что вместо параметра типа T могут передаваться самые различные типы, в том числе такие, которые не поддерживают операцию сравнения.

Однако все типы, которые по умолчанию поддерживают эту операцию сравнения, применяют интерфейс `Comparable`. То есть нам надо, чтобы два параметра представляли один и тот же тип, который реализует тип
`Comparable`. И в этом случае можно определить одну обобщенную функцию, которая будет ограниченна типом `Comparable`:

```

fun main() {

    val result1 = getBiggest(1, 2)
    println(result1)
    val result2 = getBiggest("Tom", "Sam")
    println(result2)

}
fun <T: Comparable<T>> getBiggest(a: T, b: T): T{
    return if(a > b) a
    else b
}
```

Ограничение указывается после названия параметра через двоеточие: `<T: Comparable<T>>` - то есть в данном случае тип T ограничен типом `Comparable<T>`,
иначе говоря должен представлять тип `Comparable<T>`. Причем тип Comparable сам является обобщенным.

Стоит отметить, что по умолчанию ко всем параметрам типа также применяется ограничение в виде типа `Any?`. То есть определение параметра типа
`<T>` фактически аналогично определению `<T: Any?>`

Подобным образом мы можем использовать в качестве ограничений собственные типы. Например, нам надо определить функцию для условной отправки сообщения:

```

fun<T:Message> send(message: T){
    println(message.text)
}

interface Message{
    val text: String
}
class EmailMessage(override val text: String): Message
class SmsMessage(override val text: String): Message
```

Здесь определен интерфейс Message, который имеет одно свойство - `text` и представляет условное сообщение. И также есть два класса, которые реализуют этот интерфейс:
`EmailMessage` и `SmsMessage`.

Функция `send()` использует ограничение `<T:Message>`, то есть она принимает объект некоторого типа, который должен
реализовать интерфейс Message.

Далее мы можем вызвать эту функцию, передав ей соответствующий объект:

```

fun main() {
    val email1 = EmailMessage("Hello METANIT.COM")
    send(email1)
    val sms1 = SmsMessage("Привет, ты спишь?")
    send(sms1)
}
```

Здесь в обоих вызовах функция `send()` ожидает объект Message. Однако мы можем указать точный тип, используемый функцией:

```

fun main() {
    val email1 = EmailMessage("Hello METANIT.COM")
    send<EmailMessage>(email1)
    val sms1 = SmsMessage("Привет, ты спишь?")
    send<SmsMessage>(sms1)
}
```

### Установка нескольких ограничений

В примере выше мы могли передавать в функцию `getBiggest()` любой объект, который реализует интерфейс Comparable. Но что, если мы хотим, чтобы функция могла сравнивать только числа?
Все числовые типы данных наследуются от базового класса Number. И мы можем задать еще одно ограничение - чтобы сравниваемый объект
представлял тип Number:

```

fun <T> getBiggest(a: T, b: T): T where T: Comparable<T>, 
                                        T: Number {
    return if(a > b) a
    else b
}
```

Если параметра типа надо установить несколько ограничений, то все они указываются после возвращаемого типа функции после слова where через запятую в форме:

```
параметр_типа: ограничений
```

И в этом случае мы сможем передать в функцию объекты, которые одновременно реализуют интерфейс Comparable и являются наследниками класса Number:

```

fun main() {

    val result1 = getBiggest(1, 2)
    println(result1)    // 2

    val result2 = getBiggest(1.6, -2.8)
    println(result2)    // 1.6
	
    // val result3 = getBiggest("Tom", "Sam")  // ! Ошибка - String не является производным от класса Number
    // println(result3)
}
```

Подобным образом мы можем использовать собственные типы в качестве ограничений:

```

fun main() {
    val email1 = EmailMessage("Hello METANIT.COM")
    send(email1)
    val sms1 = SmsMessage("Привет, ты спишь?")
    send(sms1)
}
fun<T> send(message: T) where T: Message, T: Logger{
    message.log()
}

interface Message{ val text: String }
interface Logger{ fun log() }

class EmailMessage(override val text: String): Message, Logger{
    override fun log() = println("Email: $text")
}
class SmsMessage(override val text: String): Message, Logger{
    override fun log() = println("SMS: $text")
}
```

Здесь для функции `send()` установлено два ограничения: используемый параметр типа T должен представлять одновременно оба интерфейса - Message и Logger.

### Ограничения в классах

Классы, как и функции, могут принимать ограничения обощений. Например, установка одного ограничения:

```

class Messenger<T:Message>(){
    fun send(mes: T){
        println(mes.text)
    }
}
```

Установка нескольких ограничений:

```

fun main() {
    val email1 = EmailMessage("Hello METANIT.COM")
    val outlook = Messenger<EmailMessage>()
    outlook.send(email1)

    val skype = Messenger<SmsMessage>()
    val sms1 = SmsMessage("Привет, ты спишь?")
    skype.send(sms1)
}
class Messenger<T>() where T: Message, T: Logger{
    fun send(mes: T){
        mes.log()
    }
}
interface Message{ val text: String }
interface Logger{ fun log() }

class EmailMessage(override val text: String): Message, Logger{
    override fun log() = println("Email: $text")
}
class SmsMessage(override val text: String): Message, Logger{
    override fun log() = println("SMS: $text")
}
```

Здесь стоит обратить внимание, что поскольку конструктор класса Messenger не принимает параметров типа T, то нам надо явным образом указать, какой именно тип будет использоваться:

```
val outlook = Messenger<EmailMessage>()
```

В качестве альтернативы можно было бы явным образом указать тип переменной:

```
val outlook: Messenger<EmailMessage> = Messenger()
```

[Назад](./6.1.php)[Содержание](./)[Вперед](./6.3.php)