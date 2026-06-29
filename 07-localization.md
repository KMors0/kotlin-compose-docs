# Локализация и интернационализация

> Локализация (l10n) и интернационализация (i18n) — важные аспекты приложений, ориентированных на международную аудиторию. В этой части рассматриваются подходы к организации строковых ресурсов и поддержке нескольких языков в кроссплатформенном приложении.

---

## 7.1. Организация строк

- **Android:** файлы `strings.xml`.
- **iOS:** файлы `.strings`.
- **Web/Shared:** JSON-файлы.

**Пример JSON:**
```json
{
  "hello": "Hello",
  "greet": "Hello, {name}!"
}
```

Для кроссплатформенного проекта рекомендуется использовать единый формат строковых ресурсов в общем модуле (например, JSON или `.properties`), а затем конвертировать их в платформенные форматы при сборке. Это позволяет поддерживать все переводы в одном месте и избежать рассинхронизации между платформами.

---

## 7.2. Библиотека `kotlin-i18n`

Упрощает подстановку строк:

**Инициализация:**
```kotlin
val locale = Locale("ru")
val resource = ResourceBundle.getBundle("messages", locale)
val i18n = I18n(resource)
```

**Использование:**
```kotlin
i18n.get("greet", name = "Alice") // "Hello, Alice!"
```

Библиотека `kotlin-i18n` предоставляет удобный API для подстановки параметров в локализованные строки. Она поддерживает плюрализацию (формы множественного числа), форматирование дат и чисел с учётом локали, а также fallback на язык по умолчанию, если перевод для текущей локали отсутствует.

---

## 7.3. Пример мультиязычного приложения

1. Создайте каталог `resources/i18n` с файлами `messages_en.properties`, `messages_ru.properties`.
2. Определите текущую локаль через `Locale.getDefault()` или пользовательский выбор.
3. Используйте `I18n` для подстановки строк в UI.

**В Compose:**
```kotlin
Text(i18n.get("app_name"))
```

**Пример файлов локализации:**

`messages_en.properties`:
```properties
app_name=Task Manager
add_task=Add Task
delete_task=Delete Task
task_count=You have {count} tasks
```

`messages_ru.properties`:
```properties
app_name=Менеджер задач
add_task=Добавить задачу
delete_task=Удалить задачу
task_count=У вас {count} задач
```

Для динамической смены языка в приложении создайте `@Composable`-обёртку, которая предоставляется через `CompositionLocal`. Это позволит перерисовать весь UI при смене языка без необходимости перезапуска приложения.

```kotlin
val LocalI18n = compositionLocalOf { I18n(ResourceBundle.getBundle("messages", Locale("en"))) }

@Composable
fun AppWithLocalization(locale: Locale, content: @Composable () -> Unit) {
    val i18n = remember(locale) {
        I18n(ResourceBundle.getBundle("messages", locale))
    }
    CompositionLocalProvider(LocalI18n provides i18n) {
        content()
    }
}
```

---

⬅️ [← Сетевые запросы](06-networking-api.md) | [К содержанию](README.md) | [Навигация →](08-navigation.md) ➡️
