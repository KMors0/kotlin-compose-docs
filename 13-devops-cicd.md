# DevOps и CI/CD

> Автоматизация процессов сборки, тестирования и развёртывания — залог быстрой и надёжной доставки продукта. В этой части рассматриваются настройка CI/CD для кроссплатформенных проектов на Kotlin Multiplatform.

---

## 13.1. Сборка и деплой

Сборка кроссплатформенного приложения требует разных инструментов для каждой платформы. Android использует стандартный Gradle-плагин и генерирует APK/AAB. iOS требует Xcode и генерирует IPA. Web компилирует Kotlin/Wasm в WebAssembly-бандл. Desktop компилируется в JVM-приложение с нативными установщиками (MSI, DMG, DEB). Docker помогает унифицировать окружение сборки, гарантируя воспроизводимость.

```bash
# Android
./gradlew :androidApp:assembleDebug
./gradlew :androidApp:bundleRelease         # AAB для Google Play

# iOS (требует macOS + Xcode)
./gradlew :composeApp:linkDebugFrameworkIosSimulatorArm64

# Web (Wasm)
./gradlew :composeApp:wasmJsBrowserProductionWebpack

# Desktop
./gradlew :composeApp:packageDmg
./gradlew :composeApp:packageMsi

# Все тесты
./gradlew :composeApp:allTests
```

---

## 13.2. GitHub Actions (актуальные версии)

> **Важно:** Используйте актуальные версии action. Плагины `@v2`, `@v3` устарели и могут перестать работать. Текущие стабильные версии: `@v4`.

**Пример Android CI (актуальный):**
```yaml
name: Android CI

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
      - name: Build with Gradle
        run: ./gradlew :androidApp:assembleDebug
      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: android-app
          path: androidApp/build/outputs/apk/
```

**Мультиплатформенный CI с матрицей:**
```yaml
name: Multiplatform CI

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        platform: [android, desktop, wasm-js]
        include:
          - platform: android
            runs-on: ubuntu-latest
            command: ./gradlew :composeApp:androidUnitTest
          - platform: desktop
            runs-on: ubuntu-latest
            command: ./gradlew :composeApp:desktopTest
          - platform: wasm-js
            runs-on: ubuntu-latest
            command: ./gradlew :composeApp:wasmJsBrowserTest
          - platform: ios
            runs-on: macos-latest
            command: ./gradlew :composeApp:iosSimulatorArm64Test
    runs-on: ${{ matrix.runs-on || 'ubuntu-latest' }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
      - uses: gradle/actions/setup-gradle@v4
      - name: Run tests
        run: ${{ matrix.command }}
```

---

## 13.3. Codemagic и Bitrise для iOS CI

GitHub Actions не подходит для iOS-сборок в больших объёмах из-за лимита 2000 минут на macOS-раннеры. Для production KMP-проектов используют специализированные сервисы:

**Codemagic** — CI/CD от Nevercode, оптимизированный для Flutter и KMP. Поддерживает одновременную сборку Android + iOS, встроенное кодподписание, публикацию в App Store и Google Play.

```yaml
# codemagic.yaml
workflows:
  build:
    name: KMP Build
    max_build_duration: 30
    environment:
      java: 17
      xcode: 16
    scripts:
      - name: Build Android
        script: ./gradlew :androidApp:assembleRelease
      - name: Build iOS
        script: |
          cd iosApp
          pod install
          xcodebuild -workspace iosApp.xcworkspace -scheme iosApp -sdk iphonesimulator -configuration Debug
    artifacts:
      - androidApp/build/outputs/apk/**/*.apk
      - iosApp/build/Build/Products/Debug-iphonesimulator/*.app
```

**Bitrise** — альтернативный CI/CD с хорошей поддержкой iOS. Предоставляет готовые шаги (steps) для установки CocoaPods, запуска Xcode-тестов и публикации.

Выбор между GitHub Actions и специализированными сервисами зависит от масштаба проекта. Для open-source и небольших команд GitHub Actions достаточно. Для enterprise-проектов с частыми iOS-сборками — Codemagic или Bitrise экономят время и деньги.

---

## 13.4. Автоматизация релизов

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
