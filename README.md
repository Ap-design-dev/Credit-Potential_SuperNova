# Credit Potential SuperNova

Нативный компонент кредитного потенциала для **SwiftUI** и **Jetpack Compose**: сумма в рублях по центру, многострочный текст и кнопка дополнительной информации.

[Состояния Figma](https://www.figma.com/design/l1Mc2dfm0aVfw93T8GmmuM/?node-id=15-1940) · [Описание поведения](https://www.figma.com/design/l1Mc2dfm0aVfw93T8GmmuM/?node-id=18-2406) · [Сборки и тесты](https://github.com/Ap-design-dev/Credit-Potential_SuperNova/actions)

## Посмотреть

Скачайте **Code → Download ZIP**, распакуйте архив и откройте **[preview/index.html](preview/index.html)** в браузере. Зависимости и сервер для готового предпросмотра не нужны. HTML на странице GitHub отображается как исходный код — файл нужно скачать и открыть локально.

Предпросмотр позволяет вводить сумму, менять текст и название кнопки, включать Warning, Read Only и Skeleton, скрывать элементы и выбирать открытие информации в пуллере или на отдельной странице. Это браузерная модель; она не исполняет Swift/Kotlin.

## Поведение

- Только целые неотрицательные рубли. Модель — строка цифр; ноль отображается как `0 ₽`.
- Каждый тап ставит каретку в конец перед ₽. Толщина каретки — 2 логические единицы.
- Крестик очищает сумму и сохраняет клавиатуру. При фокусе он заменяет шеврон; оба можно скрыть независимо.
- Шрифт суммы уменьшается с 42 до 21, затем число прокручивается горизонтально.
- Текст и кнопка скрываются независимо. При скрытии обоих под суммой остаётся ровно 32; высота компонента — 82 при обычном размере шрифта.
- Кнопка работает в Read Only и вызывает обработчик приложения для открытия пуллера или страницы. Приложение управляет клавиатурой при переходе.
- Warning принимает многострочный текст и сохраняется до явного снятия приложением.

Все состояния, размеры и настройки — в [docs/credit-potential.md](docs/credit-potential.md).

## iOS — SwiftUI

Требуются iOS 16+, Xcode с поддержкой Swift 5.9+. Подключите Swift Package из папки `ios`. Библиотечный модуль сохраняет имя `TextFieldSuperNova`: новый компонент использует включённый в этот репозиторий общий механизм ввода, без зависимости от другого репозитория.

```swift
import SwiftUI
import TextFieldSuperNova

struct CreditScreen: View {
    @State private var amount = "1500000"
    @State private var editing = false
    @State private var showsInformation = false

    var body: some View {
        SuperNovaCreditPotential(
            amount: $amount, isEditing: $editing,
            text: "Кредитный потенциал", buttonTitle: "Подробнее"
        ) {
            editing = false
            showsInformation = true
        }
        .sheet(isPresented: $showsInformation) {
            Text("Дополнительная информация")
        }
    }
}
```

Режимы `SuperNovaCreditMode`: `.editable`, `.readOnly`, `.skeleton`. `warning: String?` передаёт предупреждение; `text: nil` и `buttonTitle: nil` скрывают блоки. Шрифты регистрируются автоматически. Пример с пуллером — [ios/Examples/DemoScreen.swift](ios/Examples/DemoScreen.swift); добавьте его в запускаемое iOS-приложение, импортирующее пакет.

```sh
cd ios
swift test
xcodebuild -scheme TextFieldSuperNova -destination 'generic/platform=iOS Simulator' CODE_SIGNING_ALLOWED=NO build
```

## Android — Jetpack Compose

Откройте папку `android` в Android Studio. В ней библиотека `library` и запускаемое приложение `demo`, показывающее Credit Potential с открытием пуллера.

Требуются JDK 17, Android SDK 35, Gradle 8.11.1. Зафиксированы AGP 8.9.2, Kotlin/Compose Compiler 2.1.20 и Compose BOM 2025.04.01. Gradle Wrapper не включён; используйте Gradle указанной версии или создайте wrapper.

```kotlin
import androidx.compose.runtime.*
import dev.supernova.textfield.SuperNovaCreditPotential
import dev.supernova.textfield.rememberSuperNovaAmountState

@Composable
fun CreditScreen(onInformation: () -> Unit) {
    val amount = rememberSuperNovaAmountState("1500000")
    SuperNovaCreditPotential(
        state = amount,
        text = "Кредитный потенциал",
        buttonTitle = "Подробнее",
        onInformation = onInformation,
    )
}
```

Режимы `SuperNovaCreditMode`: `Editable`, `ReadOnly`, `Skeleton`. `warning: String?` задаёт сообщение, `text = null` / `buttonTitle = null` скрывают блоки. Пакет `dev.supernova.textfield` сохраняет совместимость с общим кодом ввода.

```sh
cd android
gradle :library:testDebugUnitTest :library:lintDebug :demo:assembleDebug :library:compileDebugAndroidTestKotlin
```

Инструментальные тесты запускаются отдельно на эмуляторе/устройстве командой `gradle :library:connectedDebugAndroidTest`.

## Структура и материалы

- `ios`, `android` — компонент, общий механизм ввода, примеры и тесты.
- `assets` — оригинальные SVG, Skeleton и три используемых шрифта: SB Sans Display Semibold, SB Sans Text Regular и Semibold.
- `preview/index.html` — готовый автономный предпросмотр; `preview/fragment.html` — его исходный фрагмент.
- `preview/credit-potential.template.html` и `scripts/build-credit-preview.cjs` — воспроизводимая сборка фрагмента: `npm run preview`.
- `scripts/export-icons.cjs` — экспорт оригинальных SVG в нативные ресурсы: `npm install`, затем `npm run assets`.
- `.github/workflows/native-checks.yml` — автоматические сборки и проверки платформ.

Проект выделен из [TextFieldSuperNova](https://github.com/Ap-design-dev/TextFieldSuperNova/commit/e516b09fc53c2c72b5c867334b46e08f8ca1dc61). Общий код и ресурсы включены для самостоятельной сборки. Статус проверок и ограничения — в [docs/verification.md](docs/verification.md).

Предоставленные шрифты и иконки принадлежат их правообладателям. TTF содержат ссылку на [лицензию ParaType](https://www.paratype.com/eula); проект не изменяет условия использования материалов.
