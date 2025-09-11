> 🗓️ 2025-06-27 16:26 | 🏷️ #flutter 

---
## 📝 Описание  
Реализую поддержку светлой и тёмной темы в Flutter-приложении LinguaLink.  
Цель: использовать системную тему устройства по умолчанию, но иметь возможность определить собственные стили (цвета, шрифты). Реализация с учётом лучших практик — разделение на light/dark темы и подключение кастомных цветов/шрифтов через `ThemeData`.

---

## ✅ Решение  

### 📁 Структура
```

lib/

├── theme/

│   ├── app_theme.dart          # Точка входа для темы

│   ├── color_schemes.dart      # Светлая и тёмная цветовые схемы

│   ├── text_styles.dart        # Кастомные текстовые стили

│   └── theme_extensions.dart   # Расширения для ThemeData

└── main.dart

````
---

### 🎨 Настройка темы

#### `color_schemes.dart`

```dart
import 'package:flutter/material.dart';

final lightColorScheme = ColorScheme.fromSeed(
  seedColor: Colors.indigo,
  brightness: Brightness.light,
);

final darkColorScheme = ColorScheme.fromSeed(
  seedColor: Colors.indigo,
  brightness: Brightness.dark,
);
````

#### **text_styles.dart**

```
import 'package:flutter/material.dart';

TextTheme customTextTheme(TextTheme base) => base.copyWith(
  titleLarge: base.titleLarge?.copyWith(
    fontFamily: 'Lato',
    fontWeight: FontWeight.bold,
  ),
  bodyMedium: base.bodyMedium?.copyWith(
    fontFamily: 'Lato',
  ),
);
```

#### **app_theme.dart**

```
import 'package:flutter/material.dart';
import 'color_schemes.dart';
import 'text_styles.dart';

class AppTheme {
  static final light = ThemeData(
    colorScheme: lightColorScheme,
    useMaterial3: true,
    textTheme: customTextTheme(Typography.blackMountainView),
  );

  static final dark = ThemeData(
    colorScheme: darkColorScheme,
    useMaterial3: true,
    textTheme: customTextTheme(Typography.whiteMountainView),
  );
}
```

---

### **🧠 Подключение в** 

### **main.dart**

```
return MaterialApp(
  title: 'LinguaLink',
  theme: AppTheme.light,
  darkTheme: AppTheme.dark,
  themeMode: ThemeMode.system, // Используется тема системы
  home: const HelloScreen(),
);
```

---

### **🧬 Подключение кастомных шрифтов**

  

#### **pubspec.yaml**

```
flutter:
  fonts:
    - family: Lato
      fonts:
        - asset: assets/fonts/Lato-Regular.ttf
        - asset: assets/fonts/Lato-Bold.ttf
          weight: 700
        - asset: assets/fonts/Lato-Light.ttf
          weight: 300
```

Шрифты должны быть в assets/fonts/.

---

## **🔍 Источник**

- [Flutter Themes Overview](https://docs.flutter.dev/cookbook/design/themes)
    
- [Material 3 Color Scheme Generator](https://m3.material.io/theme-builder)
    
- [Объяснение и помощь от ChatGPT]
    

---

## **💬 Дополнительно**

- Можно добавить ThemeCubit для ручного переключения темы.
    
- Использовать SharedPreferences, чтобы сохранять выбор темы.
    
- Для расширения ThemeData (например, добавление padding, radius и т.п.) — использовать ThemeExtension.
    
- Позже можно использовать адаптивные размеры (MediaQuery) или библиотеку adaptive_theme.