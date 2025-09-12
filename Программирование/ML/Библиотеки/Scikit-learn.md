# Scikit-learn - Библиотека машинного обучения
#ml
## Что такое Scikit-learn?

**Scikit-learn** — наиболее популярная библиотека машинного обучения для Python, предоставляющая простые и эффективные инструменты для анализа данных и построения моделей машинного обучения.

*Источник: [Официальная документация Scikit-learn](https://scikit-learn.org/stable/)*

## Зачем нужен Scikit-learn?

1. **Простота использования**: единообразный API для всех алгоритмов
2. **Широкий функционал**: классификация, регрессия, кластеризация, снижение размерности
3. **Качество**: хорошо протестированные и оптимизированные алгоритмы
4. **Интеграция**: отлично работает с NumPy, Pandas, Matplotlib
5. **Обучающие материалы**: отличная документация и примеры

*Источник: [Scikit-learn Overview](https://scikit-learn.org/stable/tutorial/basic/tutorial.html)*

## Установка

```bash
pip install scikit-learn
```

## Основные модули

- **sklearn.linear_model**: линейные модели
- **sklearn.tree**: деревья решений
- **sklearn.ensemble**: ансамбли алгоритмов
- **sklearn.cluster**: кластеризация
- **sklearn.preprocessing**: предобработка данных
- **sklearn.model_selection**: валидация и подбор параметров
- **sklearn.metrics**: метрики качества

*Источник: [API Reference](https://scikit-learn.org/stable/modules/classes.html)*

## Основной рабочий процесс

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report

# 1. Загрузка данных
from sklearn.datasets import load_iris
X, y = load_iris(return_X_y=True)

# 2. Разделение на обучающую и тестовую выборки
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 3. Предобработка данных
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 4. Обучение модели
model = LogisticRegression(random_state=42)
model.fit(X_train_scaled, y_train)

# 5. Предсказание
y_pred = model.predict(X_test_scaled)

# 6. Оценка качества
accuracy = accuracy_score(y_test, y_pred)
print(f"Точность: {accuracy:.3f}")
print(classification_report(y_test, y_pred))
```

## Предобработка данных

### Масштабирование признаков

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

# Стандартизация (среднее=0, стд=1)
scaler_std = StandardScaler()
X_standardized = scaler_std.fit_transform(X)

# Нормализация в диапазон [0, 1]
scaler_minmax = MinMaxScaler()
X_normalized = scaler_minmax.fit_transform(X)

# Робастное масштабирование (устойчиво к выбросам)
scaler_robust = RobustScaler()
X_robust = scaler_robust.fit_transform(X)
```

### Кодирование категориальных переменных

```python
from sklearn.preprocessing import LabelEncoder, OneHotEncoder
import pandas as pd

# Данные с категориальными переменными
data = pd.DataFrame({
    'city': ['Moscow', 'SPb', 'Moscow', 'Kazan', 'SPb'],
    'size': ['S', 'M', 'L', 'M', 'S'],
    'price': [100, 200, 150, 120, 90]
})

# Label Encoding (для порядковых переменных)
le = LabelEncoder()
data['size_encoded'] = le.fit_transform(data['size'])

# One-Hot Encoding (для номинальных переменных)
ohe = OneHotEncoder(sparse_output=False, drop='first')
city_encoded = ohe.fit_transform(data[['city']])
city_columns = ohe.get_feature_names_out(['city'])

# Альтернативно через pandas
city_dummies = pd.get_dummies(data['city'], prefix='city', drop_first=True)
```

### Обработка пропущенных значений

```python
from sklearn.impute import SimpleImputer, KNNImputer
import numpy as np

# Создание данных с пропусками
X_with_missing = np.array([[1, 2], [np.nan, 3], [7, 6], [4, np.nan]])

# Простая импутация (среднее, медиана, мода)
imputer_mean = SimpleImputer(strategy='mean')
X_imputed_mean = imputer_mean.fit_transform(X_with_missing)

imputer_median = SimpleImputer(strategy='median')
X_imputed_median = imputer_median.fit_transform(X_with_missing)

# KNN импутация
imputer_knn = KNNImputer(n_neighbors=2)
X_imputed_knn = imputer_knn.fit_transform(X_with_missing)
```

*Источник: [Preprocessing data](https://scikit-learn.org/stable/modules/preprocessing.html)*

## Классификация

### Логистическая регрессия

```python
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
from sklearn.metrics import accuracy_score, confusion_matrix, classification_report

# Создание данных
X, y = make_classification(n_samples=1000, n_features=4, n_classes=2, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Обучение модели
lr = LogisticRegression(random_state=42)
lr.fit(X_train, y_train)

# Предсказание
y_pred = lr.predict(X_test)
y_prob = lr.predict_proba(X_test)  # вероятности классов

# Оценка
print(f"Точность: {accuracy_score(y_test, y_pred):.3f}")
print("Матрица ошибок:")
print(confusion_matrix(y_test, y_pred))
```

### Случайный лес

```python
from sklearn.ensemble import RandomForestClassifier

# Обучение Random Forest
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

# Предсказание
y_pred_rf = rf.predict(X_test)

# Важность признаков
feature_importance = rf.feature_importances_
print("Важность признаков:", feature_importance)
```

### Машина опорных векторов (SVM)

```python
from sklearn.svm import SVC

# Обучение SVM
svm = SVC(kernel='rbf', random_state=42)
svm.fit(X_train, y_train)

# Предсказание
y_pred_svm = svm.predict(X_test)

print(f"SVM точность: {accuracy_score(y_test, y_pred_svm):.3f}")
```

### Метод k-ближайших соседей

```python
from sklearn.neighbors import KNeighborsClassifier

# Обучение KNN
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train, y_train)

# Предсказание
y_pred_knn = knn.predict(X_test)

print(f"KNN точность: {accuracy_score(y_test, y_pred_knn):.3f}")
```

*Источник: [Classification](https://scikit-learn.org/stable/supervised_learning.html#supervised-learning)*

## Регрессия

### Линейная регрессия

```python
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression
from sklearn.metrics import mean_squared_error, r2_score
import matplotlib.pyplot as plt

# Создание данных для регрессии
X_reg, y_reg = make_regression(n_samples=100, n_features=1, noise=10, random_state=42)
X_train_reg, X_test_reg, y_train_reg, y_test_reg = train_test_split(
    X_reg, y_reg, test_size=0.2, random_state=42)

# Обучение модели
lr_reg = LinearRegression()
lr_reg.fit(X_train_reg, y_train_reg)

# Предсказание
y_pred_reg = lr_reg.predict(X_test_reg)

# Оценка
mse = mean_squared_error(y_test_reg, y_pred_reg)
r2 = r2_score(y_test_reg, y_pred_reg)

print(f"MSE: {mse:.3f}")
print(f"R²: {r2:.3f}")
print(f"Коэффициенты: {lr_reg.coef_}")
print(f"Свободный член: {lr_reg.intercept_:.3f}")
```

### Полиномиальная регрессия

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import Pipeline

# Создание пайплайна с полиномиальными признаками
poly_reg = Pipeline([
    ('poly', PolynomialFeatures(degree=2)),
    ('linear', LinearRegression())
])

poly_reg.fit(X_train_reg, y_train_reg)
y_pred_poly = poly_reg.predict(X_test_reg)

print(f"Полиномиальная регрессия R²: {r2_score(y_test_reg, y_pred_poly):.3f}")
```

### Регуляризация (Ridge, Lasso)

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet

# Ridge регрессия (L2 регуляризация)
ridge = Ridge(alpha=1.0)
ridge.fit(X_train_reg, y_train_reg)
y_pred_ridge = ridge.predict(X_test_reg)

# Lasso регрессия (L1 регуляризация)
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_reg, y_train_reg)
y_pred_lasso = lasso.predict(X_test_reg)

# Elastic Net (L1 + L2)
elastic = ElasticNet(alpha=0.1, l1_ratio=0.5)
elastic.fit(X_train_reg, y_train_reg)
y_pred_elastic = elastic.predict(X_test_reg)

print(f"Ridge R²: {r2_score(y_test_reg, y_pred_ridge):.3f}")
print(f"Lasso R²: {r2_score(y_test_reg, y_pred_lasso):.3f}")
print(f"Elastic Net R²: {r2_score(y_test_reg, y_pred_elastic):.3f}")
```

*Источник: [Linear Models](https://scikit-learn.org/stable/modules/linear_model.html)*

## Кластеризация

### K-Means

```python
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
import matplotlib.pyplot as plt

# Создание данных для кластеризации
X_clusters, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.6, random_state=42)

# K-Means кластеризация
kmeans = KMeans(n_clusters=4, random_state=42, n_init=10)
cluster_labels = kmeans.fit_predict(X_clusters)

# Центроиды кластеров
centroids = kmeans.cluster_centers_

# Визуализация
plt.figure(figsize=(10, 6))
plt.scatter(X_clusters[:, 0], X_clusters[:, 1], c=cluster_labels, cmap='viridis')
plt.scatter(centroids[:, 0], centroids[:, 1], c='red', marker='x', s=200, linewidths=3)
plt.title('K-Means Кластеризация')
plt.show()
```

### DBSCAN

```python
from sklearn.cluster import DBSCAN

# DBSCAN кластеризация
dbscan = DBSCAN(eps=0.5, min_samples=5)
dbscan_labels = dbscan.fit_predict(X_clusters)

# Количество кластеров (исключая шум)
n_clusters = len(set(dbscan_labels)) - (1 if -1 in dbscan_labels else 0)
print(f"Количество кластеров: {n_clusters}")
print(f"Количество точек шума: {list(dbscan_labels).count(-1)}")
```

### Иерархическая кластеризация

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

# Агломеративная кластеризация
agg_clustering = AgglomerativeClustering(n_clusters=4)
agg_labels = agg_clustering.fit_predict(X_clusters)

# Дендрограмма
linkage_matrix = linkage(X_clusters, method='ward')
plt.figure(figsize=(10, 6))
dendrogram(linkage_matrix)
plt.title('Дендрограмма')
plt.show()
```

*Источник: [Clustering](https://scikit-learn.org/stable/modules/clustering.html)*

## Снижение размерности

### Анализ главных компонент (PCA)

```python
from sklearn.decomposition import PCA
from sklearn.datasets import load_digits
import matplotlib.pyplot as plt

# Загрузка данных с высокой размерностью
digits = load_digits()
X_digits = digits.data  # 64 признака (8x8 пикселей)
y_digits = digits.target

# PCA для снижения размерности
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_digits)

# Объясненная дисперсия
print(f"Объясненная дисперсия: {pca.explained_variance_ratio_}")
print(f"Суммарная объясненная дисперсия: {sum(pca.explained_variance_ratio_):.3f}")

# Визуализация
plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_pca[:, 0], X_pca[:, 1], c=y_digits, cmap='tab10')
plt.colorbar(scatter)
plt.title('PCA: снижение размерности с 64 до 2 измерений')
plt.xlabel('Первая главная компонента')
plt.ylabel('Вторая главная компонента')
plt.show()
```

### t-SNE

```python
from sklearn.manifold import TSNE

# t-SNE для нелинейного снижения размерности
tsne = TSNE(n_components=2, random_state=42, perplexity=30)
X_tsne = tsne.fit_transform(X_digits[:1000])  # берем меньше данных для скорости

plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y_digits[:1000], cmap='tab10')
plt.colorbar(scatter)
plt.title('t-SNE визуализация')
plt.show()
```

*Источник: [Dimensionality reduction](https://scikit-learn.org/stable/modules/decomposition.html)*

## Валидация модели и подбор параметров

### Кросс-валидация

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold

# Кросс-валидация
model = RandomForestClassifier(n_estimators=100, random_state=42)

# Простая k-fold кросс-валидация
cv_scores = cross_val_score(model, X, y, cv=5)
print(f"CV точность: {cv_scores.mean():.3f} (+/- {cv_scores.std() * 2:.3f})")

# Стратифицированная кросс-валидация
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
stratified_scores = cross_val_score(model, X, y, cv=skf)
print(f"Стратифицированная CV: {stratified_scores.mean():.3f}")
```

### Grid Search

```python
from sklearn.model_selection import GridSearchCV

# Определение сетки параметров
param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [None, 10, 20],
    'min_samples_split': [2, 5, 10]
}

# Grid Search с кросс-валидацией
grid_search = GridSearchCV(
    RandomForestClassifier(random_state=42),
    param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)

grid_search.fit(X_train, y_train)

print(f"Лучшие параметры: {grid_search.best_params_}")
print(f"Лучший результат: {grid_search.best_score_:.3f}")

# Использование лучшей модели
best_model = grid_search.best_estimator_
y_pred_best = best_model.predict(X_test)
```

### Random Search

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint

# Определение распределений параметров
param_dist = {
    'n_estimators': randint(10, 200),
    'max_depth': [None] + list(range(1, 21)),
    'min_samples_split': randint(2, 11)
}

# Random Search
random_search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42),
    param_dist,
    n_iter=50,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    random_state=42
)

random_search.fit(X_train, y_train)
print(f"Random Search лучшие параметры: {random_search.best_params_}")
```

*Источник: [Model selection](https://scikit-learn.org/stable/model_selection.html)*

## Метрики качества

### Метрики классификации

```python
from sklearn.metrics import (accuracy_score, precision_score, recall_score, 
                            f1_score, roc_auc_score, roc_curve, 
                            classification_report, confusion_matrix)

# Предсказания модели
model = LogisticRegression(random_state=42)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
y_prob = model.predict_proba(X_test)[:, 1]  # вероятности для положительного класса

# Основные метрики
accuracy = accuracy_score(y_test, y_pred)
precision = precision_score(y_test, y_pred, average='weighted')
recall = recall_score(y_test, y_pred, average='weighted')
f1 = f1_score(y_test, y_pred, average='weighted')

print(f"Точность (Accuracy): {accuracy:.3f}")
print(f"Полнота (Recall): {recall:.3f}")
print(f"Точность (Precision): {precision:.3f}")
print(f"F1-мера: {f1:.3f}")

# Подробный отчет
print("\nПодробный отчет:")
print(classification_report(y_test, y_pred))

# Матрица ошибок
print("\nМатрица ошибок:")
print(confusion_matrix(y_test, y_pred))

# ROC-AUC для бинарной классификации
if len(np.unique(y)) == 2:
    auc = roc_auc_score(y_test, y_prob)
    print(f"ROC-AUC: {auc:.3f}")
```

### Метрики регрессии

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

# Обучение модели регрессии
reg_model = LinearRegression()
reg_model.fit(X_train_reg, y_train_reg)
y_pred_reg = reg_model.predict(X_test_reg)

# Метрики регрессии
mse = mean_squared_error(y_test_reg, y_pred_reg)
rmse = np.sqrt(mse)
mae = mean_absolute_error(y_test_reg, y_pred_reg)
r2 = r2_score(y_test_reg, y_pred_reg)

print(f"MSE: {mse:.3f}")
print(f"RMSE: {rmse:.3f}")
print(f"MAE: {mae:.3f}")
print(f"R²: {r2:.3f}")
```

*Источник: [Model evaluation metrics](https://scikit-learn.org/stable/modules/model_evaluation.html)*

## Пайплайны

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer

# Создание пайплайна для предобработки и обучения
numeric_features = ['feature1', 'feature2']
categorical_features = ['category']

# Предобработчик
preprocessor = ColumnTransformer(
    transformers=[
        ('num', StandardScaler(), numeric_features),
        ('cat', OneHotEncoder(drop='first'), categorical_features)
    ]
)

# Полный пайплайн
pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(random_state=42))
])

# Обучение пайплайна
# pipeline.fit(X_train, y_train)
# y_pred = pipeline.predict(X_test)

print("Пайплайн создан успешно!")
```

*Источник: [Pipelines](https://scikit-learn.org/stable/modules/compose.html)*

## Работа с несбалансированными данными

```python
from sklearn.utils.class_weight import compute_class_weight
from imblearn.over_sampling import SMOTE  # требует установки imbalanced-learn

# Автоматический расчет весов классов
class_weights = compute_class_weight('balanced', classes=np.unique(y_train), y=y_train)
class_weight_dict = dict(zip(np.unique(y_train), class_weights))

# Модель с весами классов
weighted_model = RandomForestClassifier(
    class_weight='balanced',
    random_state=42
)
weighted_model.fit(X_train, y_train)

# SMOTE для увеличения размера меньшинства
# smote = SMOTE(random_state=42)
# X_train_smote, y_train_smote = smote.fit_resample(X_train, y_train)

print("Модель для несбалансированных данных создана!")
```

## Заключение

Scikit-learn предоставляет комплексную экосистему для машинного обучения:

- **Единообразный API**: все модели следуют принципу fit/predict
- **Широкий выбор алгоритмов**: от простых до сложных
- **Инструменты предобработки**: масштабирование, кодирование, импутация
- **Валидация моделей**: кросс-валидация, подбор параметров
- **Метрики качества**: полный набор для оценки моделей
- **Пайплайны**: автоматизация рабочих процессов

**Основные источники:**
- [Официальная документация Scikit-learn](https://scikit-learn.org/stable/)
- [User Guide](https://scikit-learn.org/stable/user_guide.html)
- [Examples Gallery](https://scikit-learn.org/stable/auto_examples/index.html)
- [API Reference](https://scikit-learn.org/stable/modules/classes.html)