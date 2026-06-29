## Анонимные функции

Последнее обновление: 03.03.2024

Анонимные функции выглядят как обычные за тем исключением, что они не имеют имени. Анонимная функция может иметь одно выражение:

```

fun(x: Int, y: Int): Int = x + y
```

Либо может представлять блок кода:

```

fun(x: Int, y: Int): Int{ 
	return x + y
}
```

Анонимную функцию можно передавать в качестве значения переменной:

```

fun main() {

    val message = fun()=println("Hello")
    message()
}
```

Здесь переменной `message` передается анонимная функция `fun()=println("Hello")`. Эта анонимная функция не принимает параметров и просто выводит
на консоль строку "Hello". Таким образом, переменная `message` будет представлять тип `() -> Unit`.

Далее мы можем вызывать эту функцию через имя переменной как обычную функцию: `message()`.

Другой пример - анонимная функция с параметрами:

```

fun main() {

    val sum = fun(x: Int, y: Int): Int = x + y 
    val result = sum(5, 4)
    println(result)		// 9
}
```

В данном случае переменной `sum` присваивается анонимная функция, которая принимает два параметра - два целых числа типа Int и возвращает их сумму.

Также через имя переменной мы можем вызвать эту анонимную функцию, передав ей некоторые значения для параметров и получить ее результат: `val result = sum(5, 4)`

### Анонимная функция как аргумент функции

Анонимную функцию можно передавать в функцию, если параметр соответствует типу этой функции:

```

fun main() {

    doOperation(9,5, fun(x: Int, y: Int): Int = x + y )		// 14
    doOperation(9,5, fun(x: Int, y: Int): Int = x - y)		// 4

    val action = fun(x: Int, y: Int): Int = x * y
    doOperation(9, 5, action)		// 45
}
fun doOperation(x: Int, y: Int, op: (Int, Int) ->Int){

    val result = op(x, y)
    println(result)
}
```

### Возвращение анонимной функции из функции

И также фунция может возвращать анонимную функцию в качестве результата:

```

fun main() {

    val action1 = selectAction(1)
    val result1 = action1(4, 5)
    println(result1)        // 9

    val action2 = selectAction(3)
    val result2 = action2(4, 5)
    println(result2)        // 20

    val action3 = selectAction(9)
    val result3 = action3(4, 5)
    println(result3)        // 0
}

fun selectAction(key: Int): (Int, Int) -> Int{
    // определение возвращаемого результата
    when(key){
        1 -> return fun(x: Int, y: Int): Int = x + y
        2 -> return fun(x: Int, y: Int): Int = x - y
        3 -> return fun(x: Int, y: Int): Int = x * y
        else -> return fun(x: Int, y: Int): Int = 0
    }
}
```

Здесь функция `selectAction()` в зависимости от переданного значения возвращает одну из четырех анонимных функций. Последняя анонимная функция
`fun(x: Int, y: Int): Int = 0` просто возвращает число 0.

При обращении к `selectAction()` переменная получит определенную анонимную функцию:

```

val action1 = selectAction(1)
```

То есть в данном случае переменная `action1` хранит ссылку на функцию `fun(x: Int, y: Int): Int = x + y`

[Назад](./3.7.php)[Содержание](./)[Вперед](./3.6.php)