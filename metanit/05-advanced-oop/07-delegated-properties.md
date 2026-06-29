## Перегрузка операторов

Последнее обновление: 03.03.2024

Kotlin позволяет определить для типов ряд встроенных операторов. Для определения оператора
для типа определяется функция с ключевым словом operator:

```
operator fun название_оператора([параметры_оператора]) : возвращаемый_тип{
	// действия функции оператора
}
```

После ключевого слова fun идет название оператора и далее скобки. Если оператор бинарный, то в скобках указывается параметр оператора.
После скобок через двоеточие указывается возвращаемый тип.

Рассмотрим простейший пример. Допустим, у нас есть класс Counter, который представляет некоторый счетчик:

```
class Counter(var value: Int)
```

Свойство `value` собственно хранит значение счетчика.

И допустим, у нас есть два объекта класса Counter - два счетчика, которые мы хотим сравнивать или складывать на основании их свойства value, используя стандартные операции сравнения и сложения:

```

val counter1 = Counter(5)
val counter2 = Counter(7)

val result = counter1 > counter2;
val counter3: Counter = counter1 + counter2;
```

Но на данный момент ни операция сравнения, ни операция сложения для объектов Counter не доступны. Эти операции могут использоваться для ряда
встроенных типов. Например, по умолчанию мы можем складывать числовые значения, но как складывать объекты классов, которые создаются непосредственно разработчиком,
компилятор не знает. И для этого нам надо выполнить перегрузку нужных нам операторов:

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(7)

    val counter1IsGreater = counter1 > counter2
    val counter3: Counter = counter1 + counter2
	
    println(counter1IsGreater)  // false
    println(counter3.value)     // 12
}

class Counter(var value: Int){

    operator fun compareTo(counter: Counter) : Int{
        return this.value - counter.value
    }
    operator fun plus(counter: Counter): Counter {
        return Counter(this.value + counter.value)
    }
}
```

Переопределение операторов предполагает переопределение соответствующих этим операторам функций. Например, операция сравнения

```
counter1 > counter2
```

транслируется в функцию

```
counter1.compareTo(counter2) > 0
```

То есть, если левый операнд (counter1) операции больше чем правый операнд (counter2), то функция оператора должна возвращать число больше 0.
И в данном случае мы можем просто вычесть из `counter1.value` значение `counter2.value`, чтобы определить, больше ли counter1 чем
counter2:

```

operator fun compareTo(counter: Counter) : Int{
	return this.value - counter.value
}
```

Оператор сложения + транслируется в
функцию plus(). Параметр этой функции представляет правый операнд операции. Левый операнд доступен через ключевое слово this:

```

operator fun plus(counter: Counter): Counter {
    return Counter(this.value + counter.value)
}
```

Возвращаемое значение операции сложения может быть любым, но в данном случае мы предполагаем, что это будет также объект Counter.

### Операторы в виде функций расширений

Операторы могут быть определены как в виде функций класса, так и в виде функций расширений. А это значит, что мы можем переопределить операторы
даже для встроенных типов:

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(3)

    val counter3: Counter = counter1 + counter2

    val counter4: Counter = 33 + counter1
    
    println(counter3.value)     // 8
    println(counter4.value)     // 38
}

class Counter(val value: Int)

operator fun Counter.plus(counter: Counter): Counter {
    return Counter(this.value + counter.value)
}

operator fun Int.plus(counter: Counter): Counter {
    return Counter(this + counter.value)
}
```

Здесь для класса Counter определена опрерация сложения с помощью функции расширения. Но кроме того, здесь также определен оператор сложения и для встроенного
типа Int - в данном случае в качестве правого операнда будет передаваться объект Counter и результатом операции также будет объект Counter:

```

operator fun Int.plus(counter: Counter): Counter {
    return Counter(this + counter.value)
}
```

Благодаря этому мы также сможем складывать объекты Int и Counter:

```

val counter4: Counter = 33 + counter1
```

Рассмотрим, какие операторы мы можем переопределить

### Унарные операторы

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

Например, переопределение операции унарного минуса

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = -counter1

    println(counter2.value)     // -5
}

class Counter(var value: Int){

    operator fun unaryMinus(): Counter{
        return Counter(-value)
    }
}
```

### Инкремент/декремент

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `++a / a++` | `a.inc()` |
| `--a / a--` | `a.dec()` |

Следует отметить, что эти операторы не должны именять текущий объект, к которому применяется операция инкремента/декремента, а должны
возвращать новый объект этого типа. Например:

```

fun main(){
    var counter1 = Counter(5)
    var counter2 = counter1++

    println(counter1.value)     // 6
    println(counter2.value)     // 5

    var counter3 = ++counter1
    println(counter1.value)     // 7
    println(counter3.value)     // 7
}

class Counter(var value: Int){

    operator fun inc(): Counter{
        return Counter(value + 1)
    }
    operator fun dec(): Counter{
        return Counter(value - 1)
    }
}
```

При операции постфиксного инкремента (`counter1++`) компилятор сначала создает временную переменную, в которую сохраняет текущий объект. Затем текущий объект замещает значением, полученным
из функции `inc()`. В качестве результата операции возвращается значение временной переменной:

```

var counter1 = Counter(5)
var counter2 = counter1++	// counter2 получает старое значение объекта counter1 из временной переменной

println(counter1.value)     // 6
println(counter2.value)     // 5
```

При префиксном инкременте (`++counter1`) компилятор возвращает новое значение, полученное из функции `inc()`:

```

println(counter1.value)     // 6

var counter3 = ++counter1
println(counter1.value)     // 7
println(counter3.value)     // 7
```

Те же правила касаются и префиксного/постфиксного декремента.

### Бинарные арифметические операторы

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)` |
| `a..b` | `a.rangeTo(b)` |

Пример операции сложения:

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(25)
    val counter3: Counter = counter1 + counter2
    println(counter3.value)     // 30

    val counter4: Counter = counter1 + 4
    println(counter4.value)     // 9
}

class Counter(var value: Int){

    operator fun plus(counter: Counter): Counter{
        return Counter(value + counter.value)
    }
    operator fun plus(number: Int): Counter{
        return Counter(value + number)
    }
}
```

Здесь реализовано две версии оператора. Первая складывает объект Counter с другим объектом Counter. Вторая версия складывает объект Counter с целым числом.

### Операторы сравнения

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |
| `a > b` | `a.compareTo(b) > 0` |
| `a < b` | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

Для первых двух операторов (== и !=) необходимо переопределить функцию equals(value: Any?),
которая должна иметь один параметр типа Any? и возвращать значение Boolean

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(5)
    val counter3 = Counter(7)

    println(counter1 == counter2)   // true
    println(counter1 == counter3)   // false
}
class Counter(var value: Int){

    override operator fun equals(counter: Any?): Boolean{
        if(counter is Counter) return this.value == counter.value
        return false
    }
}
```

В данном случае два объекта Counter равны, если равны значения их свойств value.

Для остальных операций сравнения реализуется функция compareTo(), которая должна возвращать число - значение типа Int. Если левый операнд больше правого,
то возвращает число больше 0, если меньше, то возвращается число меньше 0. Если операнды равны, возвращается 0.

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(4)
    val counter3 = Counter(7)

    println(counter1 > counter2)   // true
    println(counter1 > counter3)   // false
    println(counter1 > 1)   // true
    println(counter1 > 56)   // false
}
class Counter(var value: Int){

    operator fun compareTo(counter: Counter): Int{
        return this.value - counter.value
    }
    operator fun compareTo(number: Int): Int{
        return this.value - number
    }
}
```

### Операторы присвоения

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.remAssign(b)` |

Все функции операторов присвоения должны возвращать значение типа Unit:

```

fun main(){
    val counter1 = Counter(5)
    val counter2 = Counter(4)
    val counter3 = Counter(7)

    counter1 += counter2
    println(counter1.value)   // 9

    counter3 += 3
    println(counter3.value)   // 10
}
class Counter(var value: Int){

    operator fun plusAssign(counter: Counter){
        this.value += counter.value
    }
    operator fun plusAssign(number: Int){
        this.value += number
    }
}
```

### Оператор in

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

Как правило, под операндом `b` подразумевается некоторая коллекция, которая в качестве элемента может иметь операнд `a`:

```

fun main(){
    val tom = Person("Tom")
    val mike = Person("Mike")
    val bob = "Bob"
    val alice = "Alice"
    val jetBrains = Company(arrayOf(Person("Tom"), Person("Bob"), Person("Sam")))

    val tomInJetBrains = tom in jetBrains
    println(tomInJetBrains) // true

    val mikeInJetBrains = mike in jetBrains
    println(mikeInJetBrains) // false

    val bobInJetBrains = bob in jetBrains
    println(bobInJetBrains) // true

    val aliceInJetBrains = alice in jetBrains
    println(aliceInJetBrains) // false
}
class Person(val name:String)
class Company(private val personal: Array<Person>){
    operator fun contains(person: Person): Boolean{
        for (employee in personal) {
            if(employee.name == person.name) return true
        }
        return false
    }
    operator fun contains(personName: String): Boolean{
        for (employee in personal) {
            if(employee.name == personName) return true
        }
        return false
    }
}
```

В данном случае класс компании Company хранит в свойстве personal штат сотрудников в виде массива объекта Person. И класс Company реализует две версии
оператора in: одна версия для проверки наличия объекта Person в массиве, другая для проверки наличия объект Person, имя которого соответствует строке -
условному имени сотрудника.

### Операторы доступа по индексу

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a[i]` | `a.get(i)` |
| `a[i, j]` | `a.get(i, j)` |
| `a[i_1, ..., i_n]` | `a.get(i_1, ..., i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ..., i_n] = b` | `a.set(i_1, ..., i_n, b)` |

Применим оператор доступа по индексу

```

fun main(){
    val jetBrains = Company(arrayOf(Person("Tom"), Person("Bob"), Person("Sam")))

    // получаем пользователей
    val firstPerson: Person? = jetBrains[0]
    println(firstPerson?.name) // Tom

    val fifthPerson: Person? = jetBrains[5]
    println(fifthPerson?.name) // null

    // устанавливаем пользователей
    jetBrains[0] = Person("Mike")
    println(jetBrains[0]?.name) // Mike
}
class Person(val name:String)
class Company(private val personal: Array<Person>){
    operator fun set(index: Int, person: Person){
        // если индекс есть в массиве personal
        if(index in personal.indices)
            personal[index] = person    // то переустанавливаем значение в массиве
    }
    operator fun get(index: Int): Person?{
        // если индекс есть в массиве personal
        if(index in personal.indices)
            return personal[index] // то возвращаем значение из массива
        return null // иначе возвращаем null
    }
}
```

В данном случае с помощью функций `get/set` опосредуется доступ к массиву personal в рамках объекта People. А благодаря операторам-индексаторам
мы сможем использовать объект People как массив.

### Операторы вызова

Операторы вызова в виде реализации функции invoke() применяются для выполнения объекта на манер функции:

|  |  |
| --- | --- |
| Операция | Транслируется в |
| `a()` | `a.invoke()` |
| `a(i)` | `a.invoke(i)` |
| `a(i, j)` | `a.invoke(i, j)` |
| `a(i_1, ..., i_n)` | `a.invoke(i_1, ..., i_n)` |

Применение:

```

fun main(){
    val message=Message()
    message("Hello Kotlin")		// Message text: Hello Kotlin
}
class Message(){
    operator fun invoke(text: String) {
        println("Message text: $text")
    }
}
```

Подобный оператор удобно использовать в качестве фабрики объекта:

```

fun main(){
    val message1=Message("Hello")
    val message2 = message1("World")
    val message3 = message2("!!!")
    println(message3.text)  // Hello World !!!
}
class Message(val text: String){
    operator fun invoke(addition: String) : Message {
        return Message("$text $addition")
    }
}
```

[Назад](./5.9.php)[Содержание](./)[Вперед](./5.6.php)