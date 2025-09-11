> 🗓️ 2025-06-29 17:22 | 🏷️ #flutter #theory #widget

---
## 📝 Описание теории
`InheritedWidget` — это виджет во Flutter, который позволяет передавать данные вниз по дереву виджетов без необходимости явной передачи через конструкторы. Он используется для предоставления глобальных данных, таких как настройки, темы, язык интерфейса или данные пользователя, всем дочерним виджетам в иерархии.

**Основные принципы**:
- Хранит данные, доступные через `BuildContext` с помощью метода `dependOnInheritedWidgetOfExactType`.
- Зависимые виджеты автоматически перестраиваются, если данные изменяются и метод `updateShouldNotify` возвращает `true`.
- Метод `of` (обычно статический) упрощает доступ к данным и часто включает `assert` для проверки наличия виджета в debug-режиме.
- Эффективен, так как перестраивает только зависимые виджеты, минимизируя затраты на рендеринг.

**Область применения**:
- Передача глобальных настроек (например, язык приложения, тема).
- Совместное использование данных между несколькими экранами или виджетами.
- Основа для библиотек управления состоянием, таких как `Provider` и `Riverpod`.

**Ключевые методы**:
- `updateShouldNotify`: определяет, нужно ли перестраивать зависимые виджеты при изменении данных.
- `dependOnInheritedWidgetOfExactType`: регистрирует виджет как зависимый от `InheritedWidget`.

---

## 💻 Пример кода
Пример приложения, которое переключает язык интерфейса (английский/русский) с использованием `InheritedWidget`. Текст и кнопка обновляются с анимацией при смене языка.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const LocaleApp();
  }
}

class LocaleInheritedWidget extends InheritedWidget {
  final String language;
  final Map<String, Map<String, String>> translations;

  const LocaleInheritedWidget({
    super.key,
    required this.language,
    required this.translations,
    required super.child,
  });

  static LocaleInheritedWidget of(BuildContext context) {
    final result = maybeOf(context);
    assert(result != null, 'LocaleInheritedWidget not found in context');
    return result!;
  }

  static LocaleInheritedWidget? maybeOf(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<LocaleInheritedWidget>();
  }

  @override
  bool updateShouldNotify(LocaleInheritedWidget oldWidget) {
    return language != oldWidget.language;
  }
}

class LocaleApp extends StatefulWidget {
  const LocaleApp({super.key});

  @override
  State<LocaleApp> createState() => _LocaleAppState();
}

class _LocaleAppState extends State<LocaleApp> {
  String _language = 'en';
  final translations = {
    'en': {'greeting': 'Hello world!', 'button': 'Switch to Russian'},
    'ru': {'greeting': 'Привет, мир!', 'button': 'Переключить на английский'},
  };

  void toggleLanguage() {
    setState(() {
      _language = _language == 'en' ? 'ru' : 'en';
    });
  }

  @override
  Widget build(BuildContext context) {
    return LocaleInheritedWidget(
      language: _language,
      translations: translations,
      child: MaterialApp(
        home: Scaffold(
          appBar: AppBar(title: const Text('Language Switcher')),
          body: Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const GreetingWidget(),
                Builder(
                  builder: (context) => ElevatedButton(
                    onPressed: toggleLanguage,
                    child: Text(
                      LocaleInheritedWidget.of(context).translations[_language]?['button'] ?? 'Switch Language',
                    ),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class GreetingWidget extends StatelessWidget {
  const GreetingWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final locale = LocaleInheritedWidget.of(context);
    return AnimatedSwitcher(
      duration: const Duration(milliseconds: 300),
      child: Text(
        locale.translations[locale.language]?['greeting'] ?? 'Hello',
        key: ValueKey(locale.language),
        style: const TextStyle(fontSize: 24),
      ),
    );
  }
}
````