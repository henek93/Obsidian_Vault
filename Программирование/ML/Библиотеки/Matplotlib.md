# Matplotlib - Библиотека визуализации данных

## Что такое Matplotlib?

**Matplotlib** — фундаментальная библиотека Python для создания статических, анимированных и интерактивных визуализаций. Она предоставляет объектно-ориентированный API для встраивания графиков в приложения.

*Источник: [Официальная документация Matplotlib](https://matplotlib.org/stable/)*

## Зачем нужен Matplotlib?

1. **Научная визуализация**: создание публикационного качества графиков
2. **Исследовательский анализ данных**: быстрое построение диаграмм для понимания данных
3. **Основа для других библиотек**: Seaborn, Pandas plotting построены поверх Matplotlib
4. **Гибкость**: полный контроль над каждым элементом графика
5. **Интеграция**: работает с NumPy, Pandas и Jupyter notebooks

*Источник: [Matplotlib Overview](https://matplotlib.org/stable/users/index.html)*

## Установка

```bash
pip install matplotlib
```

## Архитектура Matplotlib

Matplotlib имеет два основных интерфейса:
1. **pyplot** — процедурный интерфейс (MATLAB-стиль)
2. **Object-oriented API** — объектно-ориентированный интерфейс

*Источник: [Matplotlib API Overview](https://matplotlib.org/stable/api/index.html)*

## Pyplot - быстрое построение графиков

### Базовые примеры

```python
import matplotlib.pyplot as plt
import numpy as np

# Простой линейный график
x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y)
plt.title('Синусоида')
plt.xlabel('x')
plt.ylabel('sin(x)')
plt.grid(True)
plt.show()
```

### Основные типы графиков

```python
# Данные для примеров
x = np.linspace(0, 10, 20)
y1 = np.sin(x)
y2 = np.cos(x)

# Создание subplot'ов
plt.figure(figsize=(12, 8))

# Линейный график
plt.subplot(2, 2, 1)
plt.plot(x, y1, 'b-', label='sin(x)')
plt.plot(x, y2, 'r--', label='cos(x)')
plt.title('Линейные графики')
plt.legend()
plt.grid(True)

# Точечный график
plt.subplot(2, 2, 2)
plt.scatter(x, y1, c='blue', alpha=0.6)
plt.title('Точечный график')

# Столбчатая диаграмма
categories = ['A', 'B', 'C', 'D']
values = [23, 45, 56, 78]
plt.subplot(2, 2, 3)
plt.bar(categories, values, color=['red', 'green', 'blue', 'orange'])
plt.title('Столбчатая диаграмма')

# Гистограмма
data = np.random.normal(0, 1, 1000)
plt.subplot(2, 2, 4)
plt.hist(data, bins=30, alpha=0.7, color='purple')
plt.title('Гистограмма')

plt.tight_layout()
plt.show()
```

*Источник: [Pyplot tutorial](https://matplotlib.org/stable/tutorials/introductory/pyplot.html)*

## Object-Oriented API

### Основные концепции

```python
# Создание Figure и Axes
fig, ax = plt.subplots(figsize=(10, 6))

# Построение графика
ax.plot(x, y1, 'b-', linewidth=2, label='sin(x)')
ax.plot(x, y2, 'r--', linewidth=2, label='cos(x)')

# Настройка осей
ax.set_xlabel('X значения')
ax.set_ylabel('Y значения')
ax.set_title('Пример OO API')
ax.legend()
ax.grid(True, alpha=0.3)

# Настройка пределов осей
ax.set_xlim(0, 10)
ax.set_ylim(-1.5, 1.5)

plt.show()
```

*Источник: [Object-oriented interface](https://matplotlib.org/stable/tutorials/introductory/usage.html#object-oriented-interface)*

## Основные типы графиков

### Линейные графики

```python
fig, ax = plt.subplots(figsize=(10, 6))

# Различные стили линий
ax.plot(x, y1, 'b-', linewidth=2, label='Сплошная')
ax.plot(x, y2, 'r--', linewidth=2, label='Пунктирная')
ax.plot(x, y1 + 0.5, 'g:', linewidth=2, label='Точечная')

# Маркеры
ax.plot(x[::5], y1[::5], 'bo', markersize=8, label='Круги')

ax.legend()
ax.set_title('Различные стили линий и маркеров')
plt.show()
```

### Scatter plots

```python
# Генерация данных
n = 100
x_scatter = np.random.randn(n)
y_scatter = np.random.randn(n)
colors = np.random.rand(n)
sizes = 1000 * np.random.rand(n)

fig, ax = plt.subplots(figsize=(10, 6))

# Scatter plot с цветами и размерами
scatter = ax.scatter(x_scatter, y_scatter, c=colors, s=sizes, alpha=0.6, cmap='viridis')
plt.colorbar(scatter, ax=ax, label='Значения цвета')

ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_title('Scatter Plot с переменными цветами и размерами')
plt.show()
```

### Гистограммы и столбчатые диаграммы

```python
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Гистограмма
data1 = np.random.normal(0, 1, 1000)
axes[0].hist(data1, bins=30, alpha=0.7, color='blue', edgecolor='black')
axes[0].set_title('Гистограмма')
axes[0].set_xlabel('Значения')
axes[0].set_ylabel('Частота')

# Столбчатая диаграмма
categories = ['A', 'B', 'C', 'D']
values1 = [20, 35, 30, 25]
axes[1].bar(categories, values1, color='skyblue', edgecolor='navy')
axes[1].set_title('Столбчатая диаграмма')

# Круговая диаграмма
sizes = [30, 25, 20, 15, 10]
labels = ['A', 'B', 'C', 'D', 'E']
axes[2].pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90)
axes[2].set_title('Круговая диаграмма')

plt.tight_layout()
plt.show()
```

*Источник: [Plot types](https://matplotlib.org/stable/plot_types/index.html)*

## Настройка графиков

### Цвета, стили и текст

```python
fig, ax = plt.subplots(figsize=(10, 6))

ax.plot(x, y1, linewidth=2)

# Настройка шрифтов
ax.set_title('Заголовок графика', fontsize=16, fontweight='bold')
ax.set_xlabel('Ось X', fontsize=12, fontweight='bold')
ax.set_ylabel('Ось Y', fontsize=12, fontweight='bold')

# Добавление текста
ax.text(5, 0.5, 'Текст на графике', fontsize=12, 
        bbox=dict(boxstyle="round,pad=0.3", facecolor="yellow", alpha=0.5))

# Аннотация с стрелкой
max_idx = np.argmax(y1)
ax.annotate('Максимальное значение', 
           xy=(x[max_idx], y1[max_idx]), 
           xytext=(x[max_idx]+1, y1[max_idx]+0.3),
           arrowprops=dict(arrowstyle='->', color='red', lw=2),
           fontsize=12, color='red')

plt.show()
```

### Настройка осей

```python
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Логарифмические оси
x_log = np.logspace(0, 2, 50)
y_log = x_log ** 2
axes[0, 0].loglog(x_log, y_log)
axes[0, 0].set_title('Логарифмические оси')
axes[0, 0].grid(True)

# Пользовательские тики
axes[0, 1].plot(x, y1)
axes[0, 1].set_xticks(np.arange(0, 11, 2))
axes[0, 1].set_yticks(np.arange(-1, 1.5, 0.5))
axes[0, 1].set_title('Пользовательские тики')
axes[0, 1].grid(True)

# Поворот меток
categories = ['Длинная категория A', 'Категория B', 'Категория C']
values = [10, 15, 12]
axes[1, 0].bar(categories, values)
axes[1, 0].set_title('Повернутые метки')
axes[1, 0].tick_params(axis='x', rotation=45)

# Две оси Y
ax2 = axes[1, 1]
ax2_twin = ax2.twinx()

ax2.plot(x, y1, 'b-', label='sin(x)')
ax2_twin.plot(x, y1*100, 'r-', label='sin(x)*100')

ax2.set_ylabel('Левая ось', color='b')
ax2_twin.set_ylabel('Правая ось', color='r')
ax2.set_title('Две оси Y')

plt.tight_layout()
plt.show()
```

*Источник: [Customizing Matplotlib](https://matplotlib.org/stable/tutorials/introductory/customizing.html)*

## Сохранение графиков

```python
fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y1, linewidth=2)
ax.set_title('График для сохранения')

# Различные форматы сохранения
plt.savefig('график.png', dpi=300, bbox_inches='tight')
plt.savefig('график.pdf', bbox_inches='tight')
plt.savefig('график.svg', bbox_inches='tight')

# С прозрачным фоном
plt.savefig('график_прозрачный.png', transparent=True, dpi=300, bbox_inches='tight')

plt.show()
```

*Источник: [Saving figures](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.savefig.html)*

## 3D графики

```python
from mpl_toolkits.mplot3d import Axes3D

# Данные для 3D
x_3d = np.linspace(-5, 5, 30)
y_3d = np.linspace(-5, 5, 30)
X, Y = np.meshgrid(x_3d, y_3d)
Z = np.sin(np.sqrt(X**2 + Y**2))

fig = plt.figure(figsize=(12, 4))

# 3D поверхность
ax1 = fig.add_subplot(131, projection='3d')
surf = ax1.plot_surface(X, Y, Z, cmap='viridis')
ax1.set_title('3D поверхность')

# 3D точки
ax2 = fig.add_subplot(132, projection='3d')
ax2.scatter(X.ravel()[::10], Y.ravel()[::10], Z.ravel()[::10], c=Z.ravel()[::10])
ax2.set_title('3D точки')

# Контурные линии
ax3 = fig.add_subplot(133)
contour = ax3.contour(X, Y, Z, levels=15)
ax3.set_title('Контурные линии')

plt.tight_layout()
plt.show()
```

## Интеграция с Pandas

```python
import pandas as pd

# Создание данных
dates = pd.date_range('2023-01-01', periods=50)
data = pd.DataFrame({
    'date': dates,
    'value1': np.cumsum(np.random.randn(50)),
    'value2': np.cumsum(np.random.randn(50))
})

# Использование pandas plotting
fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# Временной ряд
data.set_index('date').plot(ax=axes[0, 0], title='Временные ряды')

# Гистограмма
data['value1'].hist(ax=axes[0, 1], bins=15, title='Гистограмма')

# Scatter plot
data.plot.scatter(x='value1', y='value2', ax=axes[1, 0], title='Scatter plot')

# Box plot
data[['value1', 'value2']].plot.box(ax=axes[1, 1], title='Box plot')

plt.tight_layout()
plt.show()
```

## Полезные советы

### Стили и темы

```python
# Просмотр доступных стилей
print(plt.style.available)

# Применение стиля
with plt.style.context('seaborn-v0_8'):
    fig, ax = plt.subplots(figsize=(10, 6))
    ax.plot(x, y1, linewidth=2)
    ax.set_title('График в стиле Seaborn')
    ax.grid(True)
    plt.show()
```

### Настройка глобальных параметров

```python
# Настройка rcParams
plt.rcParams.update({
    'font.size': 12,
    'axes.grid': True,
    'grid.alpha': 0.3,
    'lines.linewidth': 2,
    'figure.figsize': (10, 6)
})

# График с новыми настройками
fig, ax = plt.subplots()
ax.plot(x, y1, label='sin(x)')
ax.plot(x, y2, label='cos(x)')
ax.legend()
ax.set_title('График с глобальными настройками')
plt.show()

# Сброс настроек
plt.rcdefaults()
```

## Заключение

Matplotlib является фундаментальной библиотекой визуализации в Python, предоставляя:

- **Гибкость**: полный контроль над каждым элементом графика
- **Многообразие**: поддержка всех основных типов графиков
- **Качество**: создание графиков публикационного качества
- **Интеграцию**: основа для других библиотек визуализации
- **Интерактивность**: поддержка анимаций и интерактивных элементов

**Основные источники:**
- [Официальная документация Matplotlib](https://matplotlib.org/stable/)
- [Tutorials](https://matplotlib.org/stable/tutorials/index.html)
- [Examples Gallery](https://matplotlib.org/stable/gallery/index.html)
- [API Reference](https://matplotlib.org/stable/api/index.html)
#ml