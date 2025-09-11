# 🚀 ШПАРГАЛКА - Android Interview (Современный стек)

## ⚡ КРИТИЧЕСКИ ВАЖНЫЕ ВОПРОСЫ (ЗНАТЬ ОБЯЗАТЕЛЬНО!)

### 🔵 KOTLIN COROUTINES

**Q: Объясните разницу между launch и async**
```kotlin
// launch - fire-and-forget, не возвращает результат
viewModelScope.launch {
    val data = fetchData() // Последовательно
    updateUI(data)
}

// async - возвращает Deferred для параллельного выполнения
val userDeferred = async { fetchUser() }
val postsDeferred = async { fetchPosts() }
val user = userDeferred.await() // Параллельно!
val posts = postsDeferred.await()
```

**Q: Что такое suspend функция?**
"Suspend функция - это функция, которая может быть приостановлена и возобновлена позже без блокировки потока. Может вызываться только из другой suspend функции или coroutine."

**Q: Разница между Flow и StateFlow?**
- **Flow** - холодный поток данных, начинает работу при подписке
- **StateFlow** - горячий поток с текущим состоянием, всегда имеет значение

```kotlin
// Flow - для потоков данных
fun searchUsers(query: String): Flow<List<User>> = flow {
    emit(repository.search(query))
}

// StateFlow - для состояния UI
private val _users = MutableStateFlow<List<User>>(emptyList())
val users: StateFlow<List<User>> = _users.asStateFlow()
```

**Q: Как обрабатывать ошибки в coroutines?**
```kotlin
viewModelScope.launch {
    try {
        val data = repository.getData()
        _uiState.value = UiState.Success(data)
    } catch (e: Exception) {
        _uiState.value = UiState.Error(e)
    }
}

// Или с runCatching
viewModelScope.launch {
    repository.getData()
        .onSuccess { data -> _uiState.value = UiState.Success(data) }
        .onFailure { error -> _uiState.value = UiState.Error(error) }
}
```

---

### 🎨 JETPACK COMPOSE

**Q: Что такое recomposition?**
"Recomposition - процесс повторного выполнения @Composable функций при изменении состояния. Compose автоматически отслеживает, какие части UI нужно обновить."

**Q: Когда использовать remember vs mutableStateOf?**
```kotlin
// remember - сохраняет значение между recomposition
val expensiveObject = remember { createExpensiveObject() }

// mutableStateOf - создает наблюдаемое состояние
var count by remember { mutableStateOf(0) }

// Вместе для локального состояния компонента
var text by remember { mutableStateOf("") }
```

**Q: Объясните State hoisting**
"State hoisting - поднятие состояния на уровень выше к общему родителю. Позволяет компонентам быть stateless и переиспользуемыми."

```kotlin
// ❌ Плохо - состояние внутри компонента
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) { Text("$count") }
}

// ✅ Хорошо - состояние поднято вверх
@Composable
fun Counter(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) { Text("$count") }
}
```

**Q: Когда использовать LaunchedEffect?**
```kotlin
// Для side effects, которые должны выполниться один раз или при изменении ключа
@Composable
fun UserProfile(userId: Int) {
    LaunchedEffect(userId) { // Выполнится при смене userId
        viewModel.loadUser(userId)
    }
}
```

---

### 🏗️ MVVM АРХИТЕКТУРА

**Q: Почему ViewModel переживает поворот экрана?**
"ViewModel создается ViewModelProvider и связывается с жизненным циклом ViewModelStoreOwner. При повороте Activity пересоздается, но ViewModel остается в памяти до полного уничтожения Activity."

**Q: Разница между StateFlow и LiveData?**
- **StateFlow**: Kotlin Coroutines, работает с suspend функциями, можно использовать вне Android
- **LiveData**: Android-специфичный, lifecycle-aware, автоматически приостанавливается

```kotlin
// StateFlow (современный подход)
class UserViewModel : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
}

// LiveData (старый подход)
class UserViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
}
```

**Q: Как правильно использовать viewModelScope?**
```kotlin
// ✅ Правильно
class UserViewModel : ViewModel() {
    fun loadUsers() {
        viewModelScope.launch { // Автоматическая отмена при уничтожении ViewModel
            val users = repository.getUsers()
            _users.value = users
        }
    }
}

// ❌ Неправильно - использование GlobalScope
GlobalScope.launch { // Не отменяется, может привести к утечкам памяти
    // ...
}
```

---

### 🔧 HILT DEPENDENCY INJECTION

**Q: Как работает Dependency Injection?**
"DI - паттерн, при котором зависимости объекта предоставляются извне, а не создаются внутри самого объекта. Упрощает тестирование и делает код более гибким."

**Q: Разница между @Inject и @Provides?**
```kotlin
// @Inject - для классов, которые мы можем изменить
class UserRepository @Inject constructor(
    private val apiService: ApiService
) {
    // ...
}

// @Provides - для внешних библиотек или сложной логики создания
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .build()
    }
}
```

**Q: Что такое Scopes в Hilt?**
- `SingletonComponent` - живет всё время жизни приложения
- `ViewModelComponent` - живет время жизни ViewModel
- `ActivityComponent` - живет время жизни Activity

---

### 💾 ROOM DATABASE

**Q: Объясните @Entity, @Dao, @Database**
```kotlin
// @Entity - таблица в базе данных
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    val name: String
)

// @Dao - интерфейс для работы с данными
@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    fun getAllUsers(): Flow<List<User>>
    
    @Insert
    suspend fun insertUser(user: User)
}

// @Database - основной класс базы данных
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

**Q: Как Flow работает с Room?**
"Room автоматически обновляет Flow при изменении данных в таблице. Это позволяет UI реактивно обновляться при изменениях в базе данных."

---

## 🔴 СЛОЖНЫЕ ВОПРОСЫ (SENIOR УРОВЕНЬ)

### Q: Что такое ANR и как его избежать?
"ANR (Application Not Responding) - когда главный поток заблокирован более 5 секунд. Избегать: использовать coroutines, не делать тяжелые операции на главном потоке."

### Q: Memory Leaks в Android
"Утечки памяти чаще всего из-за:
- Static ссылок на Context
- Неотмененных coroutines
- Listeners без отписки
- Inner classes с ссылкой на Activity"

### Q: Как оптимизировать RecyclerView?
- `setHasFixedSize(true)` если размер не меняется
- ViewHolder pattern (автоматически)
- DiffUtil для умного обновления
- Пагинация для больших списков

### Q: Жизненный цикл Fragment
```
onAttach() → onCreate() → onCreateView() → onViewCreated() → onStart() → onResume()
onPause() → onStop() → onDestroyView() → onDestroy() → onDetach()
```

---

## 💡 ЧАСТЫЕ ОШИБКИ JUNIOR РАЗРАБОТЧИКОВ

1. **Не использовать viewModelScope** для coroutines в ViewModel
2. **Мутировать state напрямую** в Compose вместо hoisting
3. **Не обрабатывать ошибки** в coroutines
4. **Использовать GlobalScope** вместо правильных scope
5. **Забывать про remember** в Compose
6. **Блокировать главный поток** тяжелыми операциями
7. **Не использовать lifecycle-aware компоненты**

---

## 🎯 ПРАКТИЧЕСКИЕ ЗАДАЧИ (ВОЗМОЖНЫЕ НА ИНТЕРВЬЮ)

### Задача 1: Напишите безопасный API вызов
```kotlin
suspend fun <T> safeApiCall(
    apiCall: suspend () -> Response<T>
): Result<T> {
    return try {
        val response = apiCall()
        if (response.isSuccessful) {
            response.body()?.let { Result.success(it) }
                ?: Result.failure(Exception("Empty body"))
        } else {
            Result.failure(HttpException(response))
        }
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### Задача 2: Composable с состоянием загрузки
```kotlin
@Composable
fun LoadingContent(
    isLoading: Boolean,
    content: @Composable () -> Unit
) {
    if (isLoading) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            CircularProgressIndicator()
        }
    } else {
        content()
    }
}
```

### Задача 3: ViewModel с обработкой ошибок
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState<List<User>>>(UiState.Loading)
    val uiState = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            repository.getUsers()
                .onSuccess { users -> _uiState.value = UiState.Success(users) }
                .onFailure { error -> _uiState.value = UiState.Error(error) }
        }
    }
}
```

---

## 🗣️ ГОТОВЫЕ ОТВЕТЫ НА ПОПУЛЯРНЫЕ ВОПРОСЫ

**Q: Расскажите о себе**
"Я Android разработчик с опытом работы с современным стеком: Kotlin, Jetpack Compose, Coroutines, MVVM, Hilt. Увлечен созданием качественных пользовательских интерфейсов и изучением новых технологий."

**Q: Почему Android разработка?**
"Мне нравится создавать приложения, которыми пользуются миллионы людей. Android предоставляет большие возможности для творчества и постоянно развивается."

**Q: Какие у вас слабые стороны?**
"Иногда увлекаюсь перфекционизмом в коде, но учусь находить баланс между качеством и скоростью разработки."

**Q: Ваши планы на будущее?**
"Хочу углубить знания в архитектуре приложений, изучить Kotlin Multiplatform и развиваться в направлении Senior разработчика."

---

## 🚨 ПОСЛЕДНЯЯ ПРОВЕРКА ПЕРЕД ИНТЕРВЬЮ

### Обязательно знать:
- [ ] Kotlin Coroutines (launch, async, suspend)
- [ ] Jetpack Compose основы
- [ ] MVVM архитектура
- [ ] Hilt Dependency Injection
- [ ] Room Database
- [ ] Retrofit для API
- [ ] Жизненный цикл Activity/Fragment

### Подготовить примеры:
- [ ] Свой проект на GitHub
- [ ] Объяснение архитектуры проекта
- [ ] Пример сложной задачи и как её решали

**УДАЧИ! 🍀 ВЫ СПРАВИТЕСЬ! 💪**