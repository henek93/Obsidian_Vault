# Подготовка к техническому собеседованию Android Developer

## 📚 Содержание
1. [Сборщик мусора (Garbage Collector)](#сборщик-мусора-garbage-collector)
2. [Sealed Class vs Enum](#sealed-class-vs-enum)
3. [Dagger и Hilt](#dagger-и-hilt)

---

## 🗑️ Сборщик мусора (Garbage Collector)

### Что такое Garbage Collector?
Garbage Collector (GC) - это автоматический механизм управления памятью в JVM, который освобождает память от объектов, которые больше не используются приложением.

### Основные принципы работы GC

#### 1. **Достижимость объектов (Reachability)**
- **Reachable** - объект доступен через цепочку ссылок от корневых объектов (GC Roots)
- **Unreachable** - нет путей доступа к объекту, он становится кандидатом на удаление

#### 2. **GC Roots включают:**
- Локальные переменные в стеке потоков
- Статические переменные
- JNI references
- Объекты в synchronized блоках

### Поколения объектов в памяти

```
┌─────────────────────────────────────────────┐
│                 Heap Memory                 │
├─────────────────┬───────────────────────────┤
│   Young Gen     │        Old Gen            │
├─────────────────┼───────────────────────────┤
│ Eden │ S0 │ S1  │      Tenured Space        │
└─────────────────┴───────────────────────────┘
```

#### **Young Generation:**
- **Eden Space** - где создаются новые объекты
- **Survivor Spaces (S0, S1)** - объекты, пережившие несколько циклов GC

#### **Old Generation (Tenured):**
- Долгоживущие объекты, пережившие множество циклов GC

### Типы сборки мусора

#### 1. **Minor GC**
- Происходит в Young Generation
- Быстрая операция (обычно < 100ms)
- Запускается когда Eden Space заполняется

#### 2. **Major GC**
- Происходит в Old Generation
- Более медленная операция
- Может вызвать "stop-the-world" паузы

#### 3. **Full GC**
- Очищает всю память (Young + Old + Metaspace)
- Самая медленная операция
- Следует избегать частых Full GC

### Алгоритмы сборки мусора

#### **Serial GC**
- Однопоточный
- Подходит для небольших приложений
- Использует mark-and-compact алгоритм

#### **Parallel GC (по умолчанию в Android)**
- Многопоточный
- Хорошо подходит для многоядерных устройств
- Parallel Young + Parallel Old

#### **G1GC (доступен с Android 10+)**
- Низкие паузы
- Подходит для приложений с большой памятью
- Predictable pause times

### Оптимизация работы с GC в Android

#### ✅ **Лучшие практики:**

1. **Избегайте создания объектов в циклах**
```kotlin
// ❌ Плохо
for (i in 0..1000) {
    val temp = SomeObject()
    // использование temp
}

// ✅ Хорошо
val temp = SomeObject()
for (i in 0..1000) {
    temp.reset()
    // использование temp
}
```

2. **Используйте object pools для часто создаваемых объектов**
```kotlin
class BitmapPool {
    private val pool = mutableListOf<Bitmap>()
    
    fun acquire(): Bitmap = pool.removeFirstOrNull() ?: createNew()
    fun release(bitmap: Bitmap) { pool.add(bitmap) }
}
```

3. **Избегайте утечек памяти**
```kotlin
// ❌ Утечка через anonymous inner class
class MainActivity : AppCompatActivity() {
    private val handler = Handler() {
        // Держит ссылку на Activity
    }
}

// ✅ Статический класс + WeakReference
class MyHandler(activity: MainActivity) : Handler() {
    private val activityRef = WeakReference(activity)
}
```

#### ⚠️ **Частые проблемы:**

1. **Memory Leaks**
   - Слушатели событий не отписываются
   - Static references на Activity/Fragment
   - Неправильное использование Context

2. **Excessive object creation**
   - Создание объектов в onDraw()
   - Частые String concatenations
   - Boxing/unboxing в циклах

---

## 🎭 Sealed Class vs Enum

### Enum Classes

#### **Определение и особенности:**
```kotlin
enum class NetworkState {
    LOADING,
    SUCCESS,
    ERROR,
    IDLE
}
```

#### **Преимущества Enum:**
- **Компактность** - занимает меньше памяти
- **Производительность** - быстрое сравнение через ordinal
- **Встроенные методы** - values(), valueOf(), name, ordinal
- **Serialization** - автоматическая поддержка

#### **Ограничения Enum:**
- Все значения должны быть одного типа
- Невозможно добавить дополнительные данные к каждому значению
- Ограниченная расширяемость

### Sealed Classes

#### **Определение и особенности:**
```kotlin
sealed class NetworkResult {
    object Loading : NetworkResult()
    data class Success(val data: String) : NetworkResult()
    data class Error(val exception: Exception) : NetworkResult()
}
```

#### **Преимущества Sealed Classes:**
- **Типобезопасность** - каждый подкласс может иметь свои свойства
- **Exhaustive when** - компилятор проверяет все ветки
- **Гибкость** - различные типы данных в одной иерархии
- **Расширяемость** - легко добавлять новые состояния

```kotlin
fun handleResult(result: NetworkResult) = when (result) {
    is NetworkResult.Loading -> showProgressBar()
    is NetworkResult.Success -> showData(result.data)
    is NetworkResult.Error -> showError(result.exception.message)
    // Не нужен else - компилятор знает все варианты
}
```

### Когда использовать что?

#### 🔄 **Используйте Enum когда:**

1. **Простые состояния без данных**
```kotlin
enum class Theme { LIGHT, DARK, AUTO }
enum class Priority { LOW, MEDIUM, HIGH }
```

2. **Нужна совместимость с Java**
3. **Важна производительность и память**
4. **Состояния не будут расширяться**

#### 🎯 **Используйте Sealed Class когда:**

1. **Каждое состояние несет разные данные**
```kotlin
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val error: Throwable) : UiState<Nothing>()
}
```

2. **Моделирование Result/Optional типов**
```kotlin
sealed class Result<out T> {
    data class Success<T>(val value: T) : Result<T>()
    data class Failure(val error: Exception) : Result<Nothing>()
}
```

3. **Состояния экрана в MVVM**
```kotlin
sealed class LoginScreenState {
    object Idle : LoginScreenState()
    object Loading : LoginScreenState()
    data class Success(val user: User) : LoginScreenState()
    data class ValidationError(val errors: List<String>) : LoginScreenState()
}
```

### Сравнительная таблица

| Аспект | Enum | Sealed Class |
|--------|------|-------------|
| **Память** | Меньше | Больше |
| **Производительность** | Быстрее | Медленнее |
| **Типобезопасность** | Базовая | Высокая |
| **Данные в состояниях** | ❌ | ✅ |
| **Exhaustive when** | ✅ | ✅ |
| **Расширяемость** | Ограниченная | Высокая |
| **Совместимость с Java** | ✅ | Частичная |

---

## 🏗️ Dagger и Hilt

### Dependency Injection - зачем нужно?

#### **Проблемы без DI:**
```kotlin
// ❌ Тесная связанность
class UserRepository {
    private val api = ApiService() // Жестко привязано к реализации
    private val db = Database() // Сложно тестировать
}
```

#### **Решение с DI:**
```kotlin
// ✅ Слабая связанность
class UserRepository(
    private val api: ApiService,
    private val db: Database
) {
    // Зависимости внедряются извне
}
```

### Dagger 2

#### **Основные концепции:**

1. **@Inject** - помечает конструктор/поле для инъекции
2. **@Component** - интерфейс, описывающий граф зависимостей
3. **@Module** - класс, предоставляющий зависимости
4. **@Provides** - метод в модуле, создающий зависимость
5. **@Binds** - связывает интерфейс с реализацией

#### **Пример Dagger 2:**
```kotlin
// 1. Класс с инъекцией
class UserRepository @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
)

// 2. Модуль
@Module
abstract class RepositoryModule {
    @Binds
    abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}

@Module
class NetworkModule {
    @Provides
    @Singleton
    fun provideApiService(): ApiService = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .build()
        .create(ApiService::class.java)
}

// 3. Компонент
@Singleton
@Component(modules = [RepositoryModule::class, NetworkModule::class])
interface AppComponent {
    fun inject(activity: MainActivity)
    fun userRepository(): UserRepository
}
```

#### **Сложности Dagger 2:**
- Много boilerplate кода
- Сложная настройка для Android
- Необходимость создавать множество компонентов
- Сложная иерархия зависимостей

### Hilt - Dagger для Android

#### **Преимущества Hilt:**
- **Меньше boilerplate** - автоматическая генерация компонентов
- **Android-специфичный** - встроенная поддержка Activity, Fragment, Service
- **Предопределенные компоненты** - готовые scopes для Android
- **Простое тестирование** - встроенная поддержка

#### **Основные аннотации Hilt:**

```kotlin
// 1. Application class
@HiltAndroidApp
class MyApplication : Application()

// 2. Android компоненты
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var userRepository: UserRepository
}

// 3. ViewModel
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()
```

#### **Модули в Hilt:**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    
    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService = 
        retrofit.create(ApiService::class.java)
}

@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    
    @Binds
    abstract fun bindUserRepository(
        userRepositoryImpl: UserRepositoryImpl
    ): UserRepository
}
```

#### **Scopes в Hilt:**

| Компонент | Scope | Время жизни |
|-----------|--------|-------------|
| `SingletonComponent` | `@Singleton` | Весь жизненный цикл приложения |
| `ActivityRetainedComponent` | `@ActivityRetainedScoped` | От создания Activity до окончательного уничтожения |
| `ViewModelComponent` | `@ViewModelScoped` | Жизненный цикл ViewModel |
| `ActivityComponent` | `@ActivityScoped` | Жизненный цикл Activity |
| `FragmentComponent` | `@FragmentScoped` | Жизненный цикл Fragment |
| `ViewComponent` | `@ViewScoped` | Жизненный цикл View |

### Продвинутые возможности

#### **Qualifiers - различение одинаковых типов:**
```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ApiUrl

@Qualifier
@Retention(AnnotationRetention.BINARY)  
annotation class DatabaseUrl

@Module
@InstallIn(SingletonComponent::class)
object ConfigModule {
    
    @Provides
    @ApiUrl
    fun provideApiUrl(): String = "https://api.example.com/"
    
    @Provides  
    @DatabaseUrl
    fun provideDatabaseUrl(): String = "https://db.example.com/"
}

class NetworkManager @Inject constructor(
    @ApiUrl private val apiUrl: String,
    @DatabaseUrl private val dbUrl: String
)
```

#### **Multibindings - коллекции зависимостей:**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class ValidationModule {
    
    @Binds
    @IntoSet
    abstract fun bindEmailValidator(validator: EmailValidator): Validator
    
    @Binds
    @IntoSet  
    abstract fun bindPasswordValidator(validator: PasswordValidator): Validator
}

class FormValidator @Inject constructor(
    private val validators: Set<@JvmSuppressWildcards Validator>
)
```

#### **Lazy и Provider:**
```kotlin
class ExpensiveOperationManager @Inject constructor(
    private val lazyService: Lazy<ExpensiveService>, // Создается при первом обращении
    private val serviceProvider: Provider<Service>   // Новый экземпляр каждый раз
)
```

### Тестирование с Hilt

```kotlin
@HiltAndroidTest
class UserRepositoryTest {
    
    @get:Rule
    var hiltRule = HiltAndroidRule(this)
    
    @Inject
    lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        hiltRule.inject()
    }
    
    @Test
    fun testUserRepository() {
        // Тест с реальными зависимостями
    }
}

// Замена зависимостей для тестов
@Module
@InstallIn(SingletonComponent::class)
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [NetworkModule::class]
)
object FakeNetworkModule {
    
    @Provides
    @Singleton
    fun provideFakeApiService(): ApiService = FakeApiService()
}
```

### Миграция с Dagger на Hilt

#### **Поэтапная стратегия:**

1. **Добавить Hilt в проект**
```kotlin
// app/build.gradle
plugins {
    id 'dagger.hilt.android.plugin'
}

dependencies {
    implementation 'com.google.dagger:hilt-android:2.48.1'
    kapt 'com.google.dagger:hilt-compiler:2.48.1'
}
```

2. **Создать Application класс**
```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

3. **Миграция модулей**
```kotlin
// Было в Dagger
@Module
class NetworkModule {
    @Provides
    @Singleton
    fun provideApiService(): ApiService = ...
}

// Стало в Hilt
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideApiService(): ApiService = ...
}
```

4. **Миграция компонентов Android**
```kotlin
// Было
class MainActivity : AppCompatActivity() {
    @Inject lateinit var repository: UserRepository
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DaggerAppComponent.create().inject(this)
    }
}

// Стало
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var repository: UserRepository
    // Инъекция происходит автоматически
}
```

### Лучшие практики

#### ✅ **Рекомендации:**

1. **Используйте @Binds вместо @Provides для интерфейсов**
2. **Предпочитайте Constructor Injection**
3. **Избегайте циклических зависимостей**
4. **Используйте правильные scopes**
5. **Тестируйте с реальными зависимостями**

#### ⚠️ **Частые ошибки:**

1. **Неправильные scopes** - может привести к утечкам памяти
2. **Инъекция Context** - используйте @ApplicationContext/@ActivityContext
3. **Забытый @AndroidEntryPoint** - инъекция не работает
4. **Неправильная установка модулей** - неверный @InstallIn

---

## 🎯 Вопросы для самопроверки

### Garbage Collector:
1. Объясните разницу между Minor и Major GC
2. Как работает алгоритм Mark and Sweep?
3. Какие объекты считаются GC Roots?
4. Как избежать утечек памяти в Android?

### Sealed Class vs Enum:
1. В каких случаях лучше использовать sealed class?
2. Почему sealed class обеспечивает типобезопасность?
3. Как реализовать Result pattern с sealed class?

### Dagger/Hilt:
1. Объясните разницу между @Binds и @Provides
2. Какие scopes есть в Hilt и когда их использовать?
3. Как тестировать код с Hilt?
4. В чем преимущества Hilt перед Dagger 2?

---

## 📚 Полезные ресурсы

- [Android Memory Management](https://developer.android.com/topic/performance/memory)
- [Kotlin Sealed Classes](https://kotlinlang.org/docs/sealed-classes.html)
- [Dagger Hilt Documentation](https://dagger.dev/hilt/)
- [Android Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)

---

*Удачи на собеседовании! 🚀*