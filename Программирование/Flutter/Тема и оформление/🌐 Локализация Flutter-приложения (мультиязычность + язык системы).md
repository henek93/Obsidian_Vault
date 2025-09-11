> 🗓️ 2025-06-27 16:21 | 🏷️ #flutter #lang #interna
 
---
## 📝 Описание  
Реализую поддержку мультиязычности в Flutter-приложении LinguaLink.  
Цель: сделать приложение мультиязычным с автоматическим определением языка системы (iOS/Android) и возможностью локализации текстов через `.arb`-файлы. Используется официальный механизм Flutter (`flutter gen-l10n`), без сторонних библиотек.

---

## ✅ Решение  

### ⚙️ Конфигурация

Создан файл `l10n.yaml` в корне проекта:

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
output-dir: lib/generated
output-class: S
preferred-supported-locales:
  - en
  - ru
````

Добавлен параметр в pubspec.yaml:

```
flutter:
  generate: true
```

### **📁 Структура**

```
lib/
├── l10n/
│   ├── app_en.arb
│   └── app_ru.arb
├── generated/
│   └── app_localizations.dart  ← генерируется автоматически
├── extensions/
│   └── l10n_context.dart       ← расширение BuildContext
└── main.dart
```

### **🌍 Пример ARB-файлов**

  

lib/l10n/app_en.arb:

```
{
  "@@locale": "en",
  "login_button": "Log In",
  "welcome_title": "Welcome to LinguaLink"
}
```

lib/l10n/app_ru.arb:

```
{
  "@@locale": "ru",
  "login_button": "Войти",
  "welcome_title": "Добро пожаловать в LinguaLink"
}
```

### **📦 Расширение контекста (**

### **l10n_context.dart**

### **)**

```
import 'package:flutter/widgets.dart';
import 'package:generated/app_localizations.dart';

extension L10nX on BuildContext {
  S get l10n => S.of(this);
}
```

### **🧠** 

### **MaterialApp**

###  **с автоматической локалью**

```
return MaterialApp(
  locale: WidgetsBinding.instance.window.locale,
  supportedLocales: S.supportedLocales,
  localizationsDelegates: const [
    S.delegate,
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  theme: AppTheme.light,
  darkTheme: AppTheme.dark,
  themeMode: ThemeMode.system,
  home: const HelloScreen(),
);
```

---

## **🔍 Источник**

- [Flutter Internationalization](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization)
    
- [Breaking changes: gen-l10n](https://docs.flutter.dev/release/breaking-changes/flutter-generate-i10n-source)
    
- [Объяснение и помощь от ChatGPT]
    

---

## **💬 Дополнительно**

- Можно добавить LocaleCubit, чтобы управлять языком вручную.
    
- При необходимости сохранять выбор языка в SharedPreferences.
    
- Позже возможно расширение на другие языки (французский, испанский и т.п.).
    
- Обработка TextDirection.rtl (арабский, иврит) возможна автоматически через MaterialApp.