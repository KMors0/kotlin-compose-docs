## Map

Последнее обновление: 04.03.2024

Коллекция Map представляет коллекцию объектов, где каждый элемент имеет ключ и сопоставляемое с ним значение.
При этом все ключи в коллекции являются уникальными. В отличие от List и Set интерфейс Map не расширяет интерфейс Collection.

Map представляет неизменяемую коллекцию, для создания которой применяется метод mapOf().

```

val people = mapOf(1 to "Tom", 5 to "Sam", 8 to "Bob")
```

Функция mapOf принимает набор элементов, каждый из которых с помощью оператора to сопоставляет ключ со значением, например,
`1 to "Tom"` (с условным идентификатором пользователя сопоставляется его имя). В данном случае переменная `people` представляет
объект `Map<Int, String>`, где первый тип - Int представляет тип ключей (идентификатор пользователя), а второй тип - String
представляет тип значений.

Для перебора Map можно использовать цикл for, при этом каждый элемент будет представлять объект `Map.Entry<K, V>`. Через его
свойство key можно получить ключ, а с помощью value - значение:

```

val people = mapOf(1 to "Tom", 5 to "Sam", 8 to "Bob")
for(person in people){
    println("${person.key} - ${person.value}")
}
println(people)		// {1=Tom, 5=Sam, 8=Bob}
```

Консольный вывод:

```

1 - Tom
5 - Sam
8 - Bob
{1=Tom, 5=Sam, 8=Bob}
```

### Обращение к элементам Map

Для получения элементов по ключу может применяться метод get(), в который передается ключ элемента:

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
val blue = dictionary.get("blue")
println(blue)   // синий
```

В данном случае переменная `dictionary` представляет объект `Map<String, String>`, где ключи представляют строки, а значения -
то же строки (условный перевод слова). Для получения значения по ключу "blue", применяется выражение `dictionary.get("blue")`

Также можно сократить получение элемента с помощью квадратных скобок:

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
val blue = dictionary["blue"]
println(blue)   // синий
```

Если в Map нет элемента с указанным ключом, то возвращается null:

```

val yellow = dictionary.get("yellow")
// либо так 
// val yellow = dictionary["yellow"]
println(yellow)   // null
```

Такое поведение не всегда может быть предпочтительно. И в этом случае можно использовать пару других методов для получения элементов. Так, метод
getOrDefault() позволяет задать значение по умолчанию, которое будет возврашаться, если по указанному ключу нет элементов:

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
val yellow = dictionary.getOrDefault("yellow", "Undefined")
println(yellow)   // Undefined
val blue = dictionary.getOrDefault("blue", "Undefined")
println(blue)   // синий
```

Еще один метод - getOrElse() в качестве второго параметра принимает функцию, которая задает значение на случай,
если по указанному ключу нет элементов:

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
val yellow = dictionary.getOrElse("yellow"){"Not found"}
println(yellow)   // Not found
val blue = dictionary.getOrElse("blue"){"Not found"}
println(blue)   // синий
```

Кроме получения отдельных элементов Map позволяет получить отдельно ключи и значения с помощью свойств :

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
println(dictionary.values)  // [красный, синий, зеленый]
println(dictionary.keys)   // [red, blue, green]
```

Также с помощью методов containsKey() и containsValue() можно проверить наличие в Map определенного ключа и
значения соответственно:

```

val dictionary = mapOf("red" to "красный", "blue" to "синий", "green" to "зеленый")
println(dictionary.containsKey("blue"))     // true
println(dictionary.containsKey("yellow"))  // false

println(dictionary.containsValue("желтый"))    // false
println(dictionary.containsValue("зеленый"))  // true
```

### MutableMap

Изменяемые коллекции представлены интерфейсом MutableMap, который расширяет интерфейс Map. Для создания объекта
MutableMap применяется функция mutableMapOf().

```

val people = mutableMapOf(220 to "Tom", 225 to "Sam", 228 to "Bob") // MutableMap<Int, String>
```

Интерфейс MutableMap реализуется рядом коллекций:

* HashMap: простейшая реализация интерфейса MutableMap, не гарантирует порядок элементов в коллекции. Создается функцией hashMapOf()
* LinkedHashMap: представляет комбинацию HashMap и связанного списка, создается функцией linkedMapOf()

```

val linkedMap = linkedMapOf(220 to "Tom", 225 to "Sam", 228 to "Bob")  // объект типа LinkedHashMap<Int, String>
val hashMap = hashMapOf(220 to "Tom", 225 to "Sam", 228 to "Bob")   // объект типа HashMap<Int, String>
```

Для изменения Map можно применять ряд методов:

* put(key: K, value: V): добавляет элемент с ключом key и значением value
* putAll(): добавляет набор объектов типа `Pair<K, V>`. В качестве такого набора могут выступать объекты
  Iterable, Sequence и Array
* set(key: K, value: V): устанавливает для элемента с ключом key значение value
* remove(key: K): V?: удаляет элемент с ключом key. Если удаление успешно произведено, то возвращается значение удаленного элемента, иначе возвращается null.

  Дополнительная версия метода

  ```
  remove(key: K, value: V): Boolean
  ```

  удаляет элемент с ключом key, если он имеет значение value, и возвращает true, если элемент был успешно удален

Добавление данных:

```

val people = mutableMapOf(1 to "Tom", 2 to "Sam", 3 to "Bob")

// добавляем один элемент с ключом 229 и значением Mike
people.put(229, "Mike")
println(people)		// {1=Tom, 2=Sam, 3=Bob, 229=Mike}

// добавляем другую коллекцию
val employees = mapOf(301 to "Kate", 302 to "Bill")
people.putAll(employees)
println(people)		// {1=Tom, 2=Sam, 3=Bob, 229=Mike, 301=Kate, 302=Bill}
```

Изменение данных:

```

val people = mutableMapOf(1 to "Tom", 2 to "Sam", 3 to "Bob")

// изменяем элемент с ключом 1
people.set(1, "Tomas")
println(people) // {1=Tomas, 2=Sam, 3=Bob}
```

Вместо метода set для установки значения могут применяться квадратные скобки, в которые передается ключ элемента:

```

val people = mutableMapOf(1 to "Tom", 2 to "Sam", 3 to "Bob")

// изменяем элемент с ключом 1
people[1] = "Tomas"
println(people) // {1=Tomas, 2=Sam, 3=Bob}
// изменяем элемент с ключом 5
people[5] = "Adam"
println(people) // {1=Tomas, 2=Sam, 3=Bob, 5=Adam}
```

Причем если с указанным ключом нет элемента, то он добавляется, как в примере выше в случае с ключом 5.

Удаление данных:

```

val people = mutableMapOf(1 to "Tom", 2 to "Sam", 3 to "Bob")

// удаляем элемент с ключом 1
people.remove(1)
println(people) // {2=Sam, 3=Bob}
    
// удаляем элемент с ключом 3, если его значение - "Alice"
people.remove(3, "Alice")
println(people) // {2=Sam, 3=Bob}

// удаляем элемент с ключом 3, если его значение - "Bob"
people.remove(3, "Bob")
println(people) // {2=Sam}
```

[Назад](./7.3.php)[Содержание](./)[Вперед](./7.5.php)