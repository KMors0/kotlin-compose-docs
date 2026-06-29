# DevOps и CI/CD

> Автоматизация процессов сборки, тестирования и развёртывания — залог быстрой и надёжной доставки продукта. В этой части рассматриваются настройка CI/CD для кроссплатформенных проектов на Kotlin Multiplatform.

---

## 13.1. Сборка и деплой

- Настройте Gradle для мультиплатформы:
  ```bash
  ./gradlew assembleAndroidApp
  ./gradlew composeWebBrowser
  ```
- Используйте Docker для изоляции окружения.

Сборка кроссплатформенного приложения требует разных инструментов для каждой платформы. Android-сборка использует стандартный Gradle-плагин и генерирует APK/AAB. iOS-сборка требует Xcode и генерирует IPA. Web-сборка компилирует Kotlin/JS и генерирует статические файлы. Docker помогает унифицировать окружение сборки — вы можете создать образ с предустановленными JDK, Gradle и другими инструментами, чтобы гарантировать воспроизводимость сборки на любом CI-сервере.

---

## 13.2. GitHub Actions / GitLab CI

**Пример workflow для Android:**
```yaml
name: Android CI

on: push

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Build with Gradle
        run: ./gradlew assemble
      - name: Upload APK
        uses: actions/upload-artifact@v2
        with:
          name: android-app
          path: androidApp/build/outputs/apk/
```

GitHub Actions — один из самых популярных CI/CD-инструментов для open-source проектов. Для кроссплатформенного проекта вам понадобится несколько job'ов: один для Android (Ubuntu + JDK), один для iOS (macOS + Xcode) и один для Web (Ubuntu + Node.js). Используйте matrix builds для параллельного запуска тестов на разных платформах.

**Пример мультиплатформенного CI:**
```yaml
name: Multiplatform CI

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        platform: [android, ios, web]
    runs-on: ${{ matrix.platform == 'ios' && 'macos-latest' || 'ubuntu-latest' }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Run tests
        run: ./gradlew :shared:${{ matrix.platform }}Test
```

---

## 13.3. Автоматизация релизов

- Используйте SemVer для версий.
- Подписывайте APK/IPA через CI:
  - Android: `signingConfigs` в `build.gradle`.
  - iOS: настройте сертификаты в Xcode.

**Пример конфигурации подписи Android:**
```kotlin
android {
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("KEYSTORE_PATH"))
            storePassword = System.getenv("KEYSTORE_PASSWORD")
            keyAlias = System.getenv("KEY_ALIAS")
            keyPassword = System.getenv("KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}
```

Автоматизация релизов включает несколько шагов: увеличение версии (по SemVer), сборка release-артефактов, подписание, создание git-тега, публикация в магазине (Google Play, App Store) и генерация changelog. Используйте Fastlane для автоматизации публикации в App Store и Google Play — он интегрируется с CI и значительно упрощает процесс доставки.

---

⬅️ [← Безопасность](12-security.md) | [К содержанию](README.md) | [Архитектурные паттерны →](14-architecture.md) ➡️
