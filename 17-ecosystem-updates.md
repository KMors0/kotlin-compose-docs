# Обновления экосистемы

> Экосистема Kotlin и Compose Multiplatform активно развивается. В этой части рассматривается дорожная карта Compose Multiplatform, сравнение Material Design 3 с предыдущей версией и ключевые сообщества.

---

## 17.1. Дорожная карта Compose Multiplatform

Следите за обновлениями:
- Новые компоненты.
- Улучшения производительности.
- Поддержка новых платформ (например, Linux).

Compose Multiplatform развивается очень быстро. JetBrains регулярно выпускает обновления с новыми возможностями и исправлениями. Среди ключевых направлений развития: расширение набора компонентов, приближение к parity с Jetpack Compose для Android, улучшение производительности рендеринга (особенно на iOS и Web), а также добавление поддержки новых платформ.

**Ключевые вехи:**
- Стабильная поддержка iOS — возможность создания production-ready iOS-приложений.
- Compose for Web (Wasm) — компиляция в WebAssembly для максимальной производительности в браузере.
- Улучшенная интеграция с Xcode и Android Studio.

---

## 17.2. Сравнение MD3 и MD2

- Упрощённая система цветов.
- Новые компоненты (`TopAppBar` с адаптивным поведением).
- Обновлённая типографика.

Material Design 3 (Material You) значительно отличается от Material Design 2. Ключевые изменения:

| Аспект | MD2 | MD3 |
|--------|-----|-----|
| Цветовая система | Фиксированные цвета | Динамические цвета (Dynamic Color) |
| Компоненты | Классический дизайн | Обновлённые формы и высоты |
| Типографика | 13 категорий | 15 категорий с адаптивными размерами |
| Формы | Скруглённые углы | Разнообразные формы (rounded, cut, fully rounded) |
| Топ-панель | `TopAppBar` | Адаптивная `TopAppBar` с `MediumTopAppBar` и `LargeTopAppBar` |

Динамические цвета — главное нововведение MD3. На Android 12+ приложение автоматически адаптирует свою палитру на основе обоев пользователя, создавая уникальный визуальный опыт. Для платформ без поддержки Dynamic Color используется явно заданная цветовая схема.

---

## 17.3. Тренды и сообщества

- **JetBrains Compose Discord** — обсуждения, вопросы.
- **GitHub** — популярные репозитории (теги `compose-multiplatform`).
- **Конференции:** Google I/O, JetBrains DevConf.

Сообщество вокруг Compose Multiplatform быстро растёт. На GitHub появляются десятки open-source библиотек и примеров приложений. JetBrains Compose Discord — основное место для обсуждения проблем и получения помощи от разработчиков фреймворка. Конференции Google I/O и KotlinConf регулярно включают доклады о Compose Multiplatform, где анонсируются новые возможности и демонстрируются реальные кейсы.

**Рекомендуемые ресурсы:**
- [Compose Multiplatform GitHub](https://github.com/JetBrains/compose-multiplatform)
- [Kotlin Slack #compose-multiplatform](https://slack.kotlinlang.org/)
- [Compose Multiplatform samples](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials)

---

⬅️ [← Примеры приложений](16-real-examples.md) | [К содержанию](README.md) | [Стиль кода →](18-code-style.md) ➡️
