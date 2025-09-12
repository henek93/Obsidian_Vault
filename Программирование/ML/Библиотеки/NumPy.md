# NumPy - Основа научных вычислений в Python
#ml
## Что такое NumPy?

**NumPy** (Numerical Python) — фундаментальная библиотека для научных вычислений в Python, предоставляющая поддержку больших многомерных массивов и матриц, а также обширную коллекцию математических функций для работы с ними.

*Источник: [Официальная документация NumPy](https://numpy.org/doc/stable/)*

## Зачем нужен NumPy?

1. **Высокая производительность**: массивы NumPy хранятся в непрерывных блоках памяти и написаны на C/Fortran
2. **Основа для других библиотек**: pandas, scikit-learn, matplotlib построены поверх NumPy
3. **Математические операции**: векторизованные вычисления намного быстрее обычных циклов Python
4. **Работа с многомерными данными**: эффективная обработка матриц и тензоров

*Источник: [NumPy User Guide - What is NumPy?](https://numpy.org/doc/stable/user/whatisnumpy.html)*

## Установка

```bash
pip install numpy
```

## Основные концепции

### np.array - основная структура данных

```python
import numpy as np

# Создание массива
arr = np.array([1, 2, 3, 4, 5])
print(arr.dtype)  # int64 (зависит от системы)
print(arr.shape)  # (5,)
print(arr.ndim)   # 1 (одномерный)
```

*Источник: [Array creation routines](https://numpy.org/doc/stable/reference/routines.array-creation.html)*

## Создание массивов

### Основные способы создания

```python
# Из списка Python
arr1 = np.array([1, 2, 3])

# Многомерный массив
arr2 = np.array([[1, 2], [3, 4]])

# Массив нулей
zeros = np.zeros((3, 4))  # 3x4 массив нулей

# Массив единиц
ones = np.ones((2, 3))

# Массив с заданным значением
full = np.full((2, 2), 7)

# Единичная матрица
identity = np.eye(3)

# Массив с равномерно распределенными значениями
range_arr = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]

# Линейно распределенные значения
linspace = np.linspace(0, 1, 5)  # [0. 0.25 0.5 0.75 1.]

# Случайные числа
random_arr = np.random.random((2, 3))
```

*Источник: [Array creation routines](https://numpy.org/doc/stable/reference/routines.array-creation.html)*

## Основные операции с массивами

### Арифметические операции

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Поэлементные операции
print(a + b)    # [5 7 9]
print(a * b)    # [4 10 18]
print(a ** 2)   # [1 4 9]

# Операции с скаляром
print(a + 10)   # [11 12 13]
print(a * 2)    # [2 4 6]
```

### Математические функции

```python
arr = np.array([1, 4, 9, 16])

# Основные функции
print(np.sqrt(arr))     # [1. 2. 3. 4.]
print(np.exp(arr))      # экспонента
print(np.log(arr))      # натуральный логарифм
print(np.sin(arr))      # синус
print(np.cos(arr))      # косинус

# Агрегирующие функции
print(np.sum(arr))      # 30
print(np.mean(arr))     # 7.5
print(np.std(arr))      # стандартное отклонение
print(np.min(arr))      # 1
print(np.max(arr))      # 16
```

*Источник: [Mathematical functions](https://numpy.org/doc/stable/reference/routines.math.html)*

## Индексация и срезы

### Одномерные массивы

```python
arr = np.array([0, 1, 2, 3, 4, 5])

print(arr[0])       # 0
print(arr[-1])      # 5
print(arr[1:4])     # [1 2 3]
print(arr[::2])     # [0 2 4]
```

### Многомерные массивы

```python
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

print(arr2d[0, 1])      # 2
print(arr2d[1, :])      # [4 5 6]
print(arr2d[:, 1])      # [2 5 8]
print(arr2d[1:, 1:])    # [[5 6] [8 9]]
```

### Булевая индексация

```python
arr = np.array([1, 2, 3, 4, 5])
mask = arr > 3
print(arr[mask])        # [4 5]
print(arr[arr % 2 == 0])  # [2 4]
```

*Источник: [Indexing routines](https://numpy.org/doc/stable/reference/routines.indexing.html)*

## Изменение формы массивов

```python
arr = np.array([1, 2, 3, 4, 5, 6])

# Изменение формы
reshaped = arr.reshape(2, 3)    # [[1 2 3] [4 5 6]]
reshaped = arr.reshape(3, 2)    # [[1 2] [3 4] [5 6]]

# Сглаживание
flattened = reshaped.flatten()  # [1 2 3 4 5 6]

# Транспонирование
matrix = np.array([[1, 2], [3, 4]])
transposed = matrix.T           # [[1 3] [2 4]]

# Добавление измерений
expanded = np.expand_dims(arr, axis=0)  # [[1 2 3 4 5 6]]
```

*Источник: [Array manipulation routines](https://numpy.org/doc/stable/reference/routines.array-manipulation.html)*

## Объединение и разделение массивов

```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

# Объединение
concatenated = np.concatenate([a, b])       # [1 2 3 4 5 6]
stacked_v = np.vstack([a, b])              # [[1 2 3] [4 5 6]]
stacked_h = np.hstack([a, b])              # [1 2 3 4 5 6]

# Разделение
arr = np.array([1, 2, 3, 4, 5, 6])
split_result = np.split(arr, 3)            # [array([1, 2]), array([3, 4]), array([5, 6])]
```

## Линейная алгебра

```python
# Матричное умножение
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

product = np.dot(A, B)          # [[19 22] [43 50]]
# или с использованием @ (Python 3.5+)
product = A @ B

# Определитель
det = np.linalg.det(A)          # -2.0

# Обратная матрица
inv = np.linalg.inv(A)

# Собственные значения и векторы
eigenvals, eigenvecs = np.linalg.eig(A)

# Решение системы линейных уравнений Ax = b
b = np.array([1, 2])
x = np.linalg.solve(A, b)
```

*Источник: [Linear algebra (numpy.linalg)](https://numpy.org/doc/stable/reference/routines.linalg.html)*

## Статистические функции

```python
data = np.array([[1, 2, 3], [4, 5, 6]])

# Основная статистика
print(np.mean(data))            # 3.5
print(np.median(data))          # 3.5
print(np.std(data))             # стандартное отклонение
print(np.var(data))             # дисперсия

# По осям
print(np.mean(data, axis=0))    # [2.5 3.5 4.5]
print(np.mean(data, axis=1))    # [2. 5.]

# Квантили
print(np.percentile(data, 50))  # медиана
print(np.quantile(data, 0.25))  # первый квартиль
```

*Источник: [Statistics](https://numpy.org/doc/stable/reference/routines.statistics.html)*

## Работа с файлами

```python
# Сохранение в бинарный формат
arr = np.array([1, 2, 3, 4, 5])
np.save('array.npy', arr)

# Загрузка из бинарного формата
loaded = np.load('array.npy')

# Сохранение в текстовый формат
np.savetxt('array.txt', arr)

# Загрузка из текстового формата
loaded_txt = np.loadtxt('array.txt')

# Работа с CSV
data = np.array([[1, 2], [3, 4]])
np.savetxt('data.csv', data, delimiter=',')
loaded_csv = np.loadtxt('data.csv', delimiter=',')
```

*Источник: [Input and output](https://numpy.org/doc/stable/reference/routines.io.html)*

## Важные атрибуты массивов

```python
arr = np.array([[1, 2, 3], [4, 5, 6]])

print(arr.shape)        # (2, 3) - форма массива
print(arr.size)         # 6 - общее количество элементов
print(arr.ndim)         # 2 - количество измерений
print(arr.dtype)        # int64 - тип данных
print(arr.itemsize)     # 8 - размер элемента в байтах
print(arr.nbytes)       # 48 - общий размер в байтах
```

## Полезные трюки и советы

### Векторизация

```python
# Вместо циклов используйте векторизованные операции
# Медленно:
result = []
for x in range(1000000):
    result.append(x ** 2)

# Быстро:
arr = np.arange(1000000)
result = arr ** 2
```

### Broadcasting

```python
# NumPy автоматически расширяет массивы для операций
a = np.array([[1], [2], [3]])       # (3, 1)
b = np.array([10, 20, 30])          # (3,)
result = a + b                       # (3, 3)
```

*Источник: [Broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html)*

## Типы данных

```python
# Основные типы данных
int_arr = np.array([1, 2, 3], dtype=np.int32)
float_arr = np.array([1.0, 2.0, 3.0], dtype=np.float64)
bool_arr = np.array([True, False, True], dtype=np.bool_)

# Преобразование типов
arr = np.array([1.7, 2.3, 3.9])
int_converted = arr.astype(np.int32)   # [1 2 3]
```

*Источник: [Data type objects (dtype)](https://numpy.org/doc/stable/reference/arrays.dtypes.html)*

## Заключение

NumPy является основой для научных вычислений в Python. Его эффективные многомерные массивы и богатая библиотека функций делают его незаменимым инструментом для:

- Численных вычислений
- Линейной алгебры
- Обработки данных
- Машинного обучения
- Научного анализа

**Основные источники:**
- [Официальная документация NumPy](https://numpy.org/doc/stable/)
- [NumPy User Guide](https://numpy.org/doc/stable/user/index.html)
- [NumPy Reference](https://numpy.org/doc/stable/reference/index.html)