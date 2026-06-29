# Интеграция Kotlin + Compose + M3: лучшие практики

> Практическое руководство по настройке проекта, кроссплатформенному темированию и лучшим практикам, основанным на опыте компаний, внедривших Compose Multiplatform в продакшен.

---

## 23.1. Настройка проекта

Для создания нового проекта Compose Multiplatform с Material 3 используется KMP Wizard (в IDE или на kmp.jetbrains.com) или ручная настройка через Gradle. Ключевые зависимости в `libs.versions.toml`:

```toml
[versions]
kotlin = "2.1.0"
compose-multiplatform = "1.8.0"
agp = "8.7.0"
material3 = "1.3.0"

[libraries]
material3 = { module = "androidx.compose.material3:material3", version.ref = "material3" }

[plugins]
kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
composeMultiplatform = { id = "org.jetbrains.compose", version.ref = "compose-multiplatform" }
composeCompiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
```

В модуле `shared` настраивается общая тема Material 3, которая автоматически применяется на всех платформах. Для платформоспецифичных настроек (например, динамического цвета на Android) используются `expect`/`actual` декларации:

```kotlin
// commonMain
@Composable
expect fun getAppColorScheme(isDark: Boolean): ColorScheme

// androidMain
@Composable
actual fun getAppColorScheme(isDark: Boolean): ColorScheme =
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S)
        if (isDark) dynamicDarkColorScheme(LocalContext.current)
        else dynamicLightColorScheme(LocalContext.current)
    else if (isDark) DarkColorScheme else LightColorScheme

// iosMain / desktopMain / wasmJsMain
@Composable
actual fun getAppColorScheme(isDark: Boolean): ColorScheme =
    if (isDark) DarkColorScheme else LightColorScheme
```

---

## 23.2. Кроссплатформенное темирование

Один из ключевых advantage'ов стека — единая тема для всех платформ. `MaterialTheme` из `androidx.compose.material3` работает одинаково на Android, iOS, Desktop и Web. Это означает, что цветовая палитра, типографика, форма компонентов и их поведение полностью согласованы на всех платформах без дополнительных усилий.

Для импорта темы из [Material Theme Builder](https://m3.material.io/theme-builder) в KMP-проект необходимо убедиться, что пакетный путь корректен для всех исходных наборов (source sets), а цвета определены в `commonMain`. Пользовательские шрифты требуют платформоспецифичной загрузки через `expect`/`actual` для каждого таргета.

---

## 23.3. Лучшие практики

На основе анализа опыта компаний, внедривших Compose Multiplatform в продакшен, и рекомендаций JetBrains, можно выделить следующие лучшие практики:

### 1. Инкрементальное внедрение

KMP и Compose Multiplatform поддерживают пошаговое внедрение в существующие проекты. Можно начать с вынесения общей бизнес-логики в `shared`-модуль, а затем постепенно переносить UI на Compose. Это критически важно для существующих проектов — не нужно переписывать всё сразу.

### 2. Архитектура Clean/Modular

Используйте много модульную архитектуру с чётким разделением слоёв: `data` (репозитории, API), `domain` (use cases), `feature-*` (UI + ViewModel). Это облегчает тестирование и параллельную разработку в команде.

### 3. Единая кодовая база UI

Максимально выносите UI в `commonMain`. Платформоспецифичный код сводите к минимуму — точкам входа, платформенным API (push-уведомления, биометрия) и ресурсам. Чем больше кода в `commonMain`, тем меньше дублирования и несогласованности между платформами.

### 4. Типобезопасная навигация

Используйте `@Serializable` маршруты с `composable<Type>()` для обеспечения безопасности типов и рефакторинга навигации. Это позволяет IDE отслеживать все переходы и предупреждать о нарушениях на этапе компиляции.

### 5. Тестирование общего кода

Пишите модульные тесты для бизнес-логики и ViewModel в `commonTest`, чтобы они выполнялись на всех платформах без дублирования. Один набор тестов покрывает все платформы.

### 6. Ресурсное управление

Используйте общие строковые ресурсы через `compose.resources` и платформоспецифичные ресурсы (изображения в разных разрешениях) через механизм `expect`/`actual`. Это обеспечивает единообразие текстов и локализации.

---

⬅️ [← M3: глубокое погружение](22-deep-dive-md3.md) | [К содержанию](README.md) | [Ограничения →](24-limitations.md) ➡️
