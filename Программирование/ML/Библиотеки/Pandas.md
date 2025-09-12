# Pandas - Библиотека анализа и обработки данных
#ml
## Что такое Pandas?

**Pandas** — мощная библиотека Python для анализа и обработки данных, предоставляющая высокоуровневые структуры данных и инструменты для работы с табличными и временными рядами данных.

*Источник: [Официальная документация Pandas](https://pandas.pydata.org/docs/)*

## Зачем нужен Pandas?

1. **Работа с табличными данными**: удобная обработка CSV, Excel, SQL данных
2. **Очистка данных**: обработка пропущенных значений, дубликатов, выбросов
3. **Агрегация и группировка**: мощные инструменты для анализа данных
4. **Временные ряды**: специализированные функции для работы с датами
5. **Интеграция**: легко интегрируется с NumPy, Matplotlib, Scikit-learn

*Источник: [Getting started tutorials](https://pandas.pydata.org/docs/getting_started/intro_tutorials/)*

## Установка

```bash
pip install pandas
```

## Основные структуры данных

### Series - одномерный массив с индексом

```python
import pandas as pd
import numpy as np

# Создание Series
s = pd.Series([1, 3, 5, np.nan, 6, 8])
print(s)

# Series с пользовательским индексом
s_indexed = pd.Series([10, 20, 30], index=['a', 'b', 'c'])
print(s_indexed['b'])  # 20

# Из словаря
s_dict = pd.Series({'a': 1, 'b': 2, 'c': 3})
```

*Источник: [Series documentation](https://pandas.pydata.org/docs/reference/api/pandas.Series.html)*

### DataFrame - двумерная таблица данных

```python
# Создание DataFrame из словаря
data = {
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'City': ['New York', 'London', 'Tokyo']
}
df = pd.DataFrame(data)

# Из списка списков
df_list = pd.DataFrame([
    ['Alice', 25, 'New York'],
    ['Bob', 30, 'London']
], columns=['Name', 'Age', 'City'])

# Из NumPy массива
dates = pd.date_range('20230101', periods=6)
df_numpy = pd.DataFrame(np.random.randn(6, 4), 
                       index=dates, 
                       columns=['A', 'B', 'C', 'D'])
```

*Источник: [DataFrame documentation](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html)*

## Чтение и запись данных

### CSV файлы

```python
# Чтение CSV
df = pd.read_csv('data.csv')
df = pd.read_csv('data.csv', sep=';', encoding='utf-8')

# Запись CSV
df.to_csv('output.csv', index=False)
df.to_csv('output.csv', sep=';', encoding='utf-8')
```

### Excel файлы

```python
# Чтение Excel
df = pd.read_excel('data.xlsx')
df = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# Запись Excel
df.to_excel('output.xlsx', index=False)
```

### JSON

```python
# Чтение JSON
df = pd.read_json('data.json')

# Запись JSON
df.to_json('output.json', orient='records')
```

### Базы данных

```python
import sqlite3

# Чтение из SQL
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table_name', conn)

# Запись в SQL
df.to_sql('table_name', conn, if_exists='replace')
```

*Источник: [IO tools (text, CSV, HDF5, ...)](https://pandas.pydata.org/docs/user_guide/io.html)*

## Базовые операции просмотра данных

```python
# Основная информация о DataFrame
print(df.head())        # первые 5 строк
print(df.tail())        # последние 5 строк
print(df.info())        # информация о колонках
print(df.describe())    # статистическое описание
print(df.shape)         # размерность (строки, столбцы)
print(df.columns)       # названия колонок
print(df.index)         # индекс
print(df.dtypes)        # типы данных колонок

# Быстрый просмотр
print(df.sample(5))     # 5 случайных строк
```

*Источник: [Essential basic functionality](https://pandas.pydata.org/docs/user_guide/basics.html)*

## Индексация и выборка данных

### Выбор колонок

```python
# Одна колонка
age_series = df['Age']
age_series = df.Age  # альтернативный способ

# Несколько колонок
subset = df[['Name', 'Age']]

# Все колонки кроме указанных
subset = df.drop(['City'], axis=1)
```

### Выбор строк

```python
# По индексу
first_row = df.iloc[0]          # первая строка
first_three = df.iloc[0:3]      # первые три строки
last_row = df.iloc[-1]          # последняя строка

# По меткам индекса
row_by_label = df.loc[0]        # строка с индексом 0
rows_slice = df.loc[0:2]        # строки с индексами 0, 1, 2
```

### Условная выборка

```python
# Булевая индексация
adults = df[df['Age'] > 30]
young_in_ny = df[(df['Age'] < 30) & (df['City'] == 'New York')]

# Метод query
result = df.query('Age > 30 and City == "London"')

# Выбор по значениям
cities_filter = df[df['City'].isin(['London', 'Tokyo'])]
```

*Источник: [Indexing and selecting data](https://pandas.pydata.org/docs/user_guide/indexing.html)*

## Обработка пропущенных данных

```python
# Проверка на пропущенные значения
print(df.isnull().sum())        # количество NaN в каждой колонке
print(df.isna().any())          # есть ли NaN в колонках

# Удаление строк с NaN
df_cleaned = df.dropna()        # удалить все строки с NaN
df_cleaned = df.dropna(subset=['Age'])  # удалить строки с NaN в Age

# Заполнение пропущенных значений
df_filled = df.fillna(0)        # заполнить нулями
df_filled = df.fillna(method='forward')  # заполнить предыдущим значением
df_filled = df.fillna(df.mean()) # заполнить средним (для числовых)

# Интерполяция
df_interpolated = df.interpolate()
```

*Источник: [Working with missing data](https://pandas.pydata.org/docs/user_guide/missing_data.html)*

## Преобразование данных

### Добавление и изменение колонок

```python
# Новая колонка
df['Age_in_months'] = df['Age'] * 12

# Условное создание колонки
df['Category'] = df['Age'].apply(lambda x: 'Adult' if x >= 18 else 'Minor')
df['Category'] = np.where(df['Age'] >= 18, 'Adult', 'Minor')

# Изменение существующей колонки
df['Age'] = df['Age'] + 1
```

### Применение функций

```python
# apply() для колонок
df['Name_length'] = df['Name'].apply(len)
df['Name_upper'] = df['Name'].apply(str.upper)

# apply() для строк
df['Row_sum'] = df[['Age']].apply(sum, axis=1)

# map() для замены значений
city_mapping = {'New York': 'NY', 'London': 'LN'}
df['City_code'] = df['City'].map(city_mapping)
```

### Преобразование типов данных

```python
# Изменение типов
df['Age'] = df['Age'].astype('float64')
df['Name'] = df['Name'].astype('string')

# Преобразование в категорию
df['City'] = df['City'].astype('category')

# Преобразование в datetime
df['Date'] = pd.to_datetime(df['Date'])
```

*Источник: [Data transformations](https://pandas.pydata.org/docs/user_guide/basics.html#basics-apply)*

## Группировка и агрегация

```python
# Группировка по одной колонке
grouped = df.groupby('City')
print(grouped['Age'].mean())    # средний возраст по городам

# Группировка по нескольким колонкам
grouped_multi = df.groupby(['City', 'Category'])

# Различные агрегации
agg_result = df.groupby('City').agg({
    'Age': ['mean', 'min', 'max'],
    'Name': 'count'
})

# Множественные операции
summary = df.groupby('City').agg(
    avg_age=('Age', 'mean'),
    total_count=('Name', 'count'),
    max_age=('Age', 'max')
)

# Применение пользовательских функций
custom_agg = df.groupby('City')['Age'].agg(lambda x: x.max() - x.min())
```

*Источник: [Group by: split-apply-combine](https://pandas.pydata.org/docs/user_guide/groupby.html)*

## Сортировка данных

```python
# Сортировка по значениям
df_sorted = df.sort_values('Age')                    # по возрасту
df_sorted = df.sort_values('Age', ascending=False)   # по убыванию
df_sorted = df.sort_values(['City', 'Age'])          # по нескольким колонкам

# Сортировка по индексу
df_sorted = df.sort_index()

# Сортировка с обработкой NaN
df_sorted = df.sort_values('Age', na_position='last')
```

## Объединение данных

### Concatenation (объединение)

```python
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})

# Вертикальное объединение
result = pd.concat([df1, df2])

# Горизонтальное объединение
result = pd.concat([df1, df2], axis=1)
```

### Merge (слияние)

```python
left = pd.DataFrame({'key': ['A', 'B', 'C'], 'left_val': [1, 2, 3]})
right = pd.DataFrame({'key': ['A', 'B', 'D'], 'right_val': [4, 5, 6]})

# Inner join
inner_result = pd.merge(left, right, on='key')

# Left join
left_result = pd.merge(left, right, on='key', how='left')

# Outer join
outer_result = pd.merge(left, right, on='key', how='outer')
```

### Join

```python
# Объединение по индексу
df1 = pd.DataFrame({'A': [1, 2]}, index=['X', 'Y'])
df2 = pd.DataFrame({'B': [3, 4]}, index=['X', 'Y'])
result = df1.join(df2)
```

*Источник: [Merge, join, concatenate and compare](https://pandas.pydata.org/docs/user_guide/merging.html)*

## Работа с временными рядами

```python
# Создание временных индексов
dates = pd.date_range('2023-01-01', periods=100, freq='D')
ts = pd.Series(np.random.randn(100), index=dates)

# Парсинг дат
df['Date'] = pd.to_datetime(df['Date'])

# Извлечение компонентов даты
df['Year'] = df['Date'].dt.year
df['Month'] = df['Date'].dt.month
df['Weekday'] = df['Date'].dt.day_name()

# Ресэмплинг временных рядов
monthly_mean = ts.resample('M').mean()  # среднее по месяцам
weekly_sum = ts.resample('W').sum()     # сумма по неделям

# Скользящие окна
rolling_mean = ts.rolling(window=7).mean()  # скользящее среднее за 7 дней
```

*Источник: [Time series / date functionality](https://pandas.pydata.org/docs/user_guide/timeseries.html)*

## Строковые операции

```python
# Основные строковые методы
df['Name_lower'] = df['Name'].str.lower()
df['Name_length'] = df['Name'].str.len()
df['Contains_A'] = df['Name'].str.contains('A')

# Замена подстрок
df['Name_replaced'] = df['Name'].str.replace('Alice', 'Alicia')

# Разделение строк
df[['First', 'Last']] = df['Full_Name'].str.split(' ', expand=True)

# Извлечение подстрок
df['Name_first_3'] = df['Name'].str[:3]

# Регулярные выражения
df['Emails'] = df['Text'].str.extract(r'([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})')
```

*Источник: [Working with text data](https://pandas.pydata.org/docs/user_guide/text.html)*

## Описательная статистика

```python
# Базовая статистика
print(df.describe())                    # статистика для числовых колонок
print(df.describe(include='all'))       # для всех типов данных

# Отдельные показатели
print(df['Age'].mean())                 # среднее
print(df['Age'].median())               # медиана
print(df['Age'].std())                  # стандартное отклонение
print(df['Age'].var())                  # дисперсия
print(df['Age'].min())                  # минимум
print(df['Age'].max())                  # максимум

# Квантили
print(df['Age'].quantile(0.25))         # первый квартиль
print(df['Age'].quantile([0.25, 0.5, 0.75]))  # несколько квантилей

# Подсчет уникальных значений
print(df['City'].value_counts())        # частоты значений
print(df['City'].nunique())             # количество уникальных

# Корреляция
correlation_matrix = df.corr()          # корреляционная матрица
correlation_with_age = df.corrwith(df['Age'])  # корреляция с одной колонкой
```

## Обнаружение и обработка дубликатов

```python
# Поиск дубликатов
duplicates = df.duplicated()            # булевая маска дубликатов
duplicate_rows = df[df.duplicated()]    # строки-дубликаты

# Удаление дубликатов
df_no_duplicates = df.drop_duplicates()

# Удаление дубликатов по определенным колонкам
df_unique = df.drop_duplicates(subset=['Name', 'City'])

# Оставить последний дубликат
df_last = df.drop_duplicates(keep='last')
```

## Работа с категориальными данными

```python
# Создание категориальной колонки
df['Grade'] = pd.Categorical(['A', 'B', 'C', 'A', 'B'], 
                           categories=['A', 'B', 'C'], 
                           ordered=True)

# Преобразование в категорию
df['City'] = df['City'].astype('category')

# One-hot encoding
dummy_vars = pd.get_dummies(df['City'])
df_with_dummies = pd.concat([df, dummy_vars], axis=1)

# Label encoding для упорядоченных категорий
df['Grade_numeric'] = df['Grade'].cat.codes
```

## Pivot tables и Cross-tabulation

```python
# Pivot table
pivot = df.pivot_table(values='Age', 
                      index='City', 
                      columns='Category', 
                      aggfunc='mean')

# Cross-tabulation
crosstab = pd.crosstab(df['City'], df['Category'])

# Многоуровневый pivot
multi_pivot = df.pivot_table(values=['Age'], 
                           index=['City'], 
                           columns=['Category'],
                           aggfunc=['mean', 'count'])
```

*Источник: [Reshaping and pivot tables](https://pandas.pydata.org/docs/user_guide/reshaping.html)*

## Полезные методы и трюки

### Работа с большими данными

```python
# Чтение по частям
chunk_size = 1000
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # обработка каждого chunk
    process_chunk(chunk)

# Оптимизация типов данных
df['int_col'] = df['int_col'].astype('int32')  # вместо int64
df['category_col'] = df['category_col'].astype('category')
```

### Профилирование производительности

```python
# Измерение времени выполнения
%timeit df.groupby('City')['Age'].mean()  # в Jupyter

# Информация о памяти
print(df.memory_usage(deep=True))
print(df.info(memory_usage='deep'))
```

### Настройка отображения

```python
# Настройки отображения pandas
pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 10)
pd.set_option('display.width', 1000)
pd.set_option('display.precision', 2)

# Сброс настроек
pd.reset_option('all')
```

## Экспорт и визуализация

```python
# Быстрая визуализация (требует matplotlib)
df['Age'].hist()                        # гистограмма
df['Age'].plot(kind='box')              # box plot
df.plot.scatter(x='Age', y='Salary')    # scatter plot

# Экспорт в различные форматы
df.to_csv('output.csv')
df.to_excel('output.xlsx')
df.to_json('output.json')
df.to_html('output.html')
df.to_latex('output.tex')
```

## Заключение

Pandas является незаменимым инструментом для анализа данных в Python, предоставляя:

- **Мощные структуры данных**: Series и DataFrame
- **Широкие возможности ввода/вывода**: CSV, Excel, JSON, SQL
- **Гибкую обработку данных**: фильтрация, группировка, агрегация
- **Временные ряды**: специализированные функции для работы с датами
- **Интеграцию с экосистемой**: NumPy, Matplotlib, Scikit-learn

**Основные источники:**
- [Официальная документация Pandas](https://pandas.pydata.org/docs/)
- [User Guide](https://pandas.pydata.org/docs/user_guide/index.html)
- [API Reference](https://pandas.pydata.org/docs/reference/index.html)
- [Getting started tutorials](https://pandas.pydata.org/docs/getting_started/intro_tutorials/)