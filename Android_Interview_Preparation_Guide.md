# 📱 Подготовка к техническому интервью Android Junior Developer

## 📋 Содержание
1. [Общий план подготовки](#общий-план-подготовки)
2. [Основы Kotlin/Java](#основы-kotlinjava)
3. [Android основы](#android-основы)
4. [Компоненты Android](#компоненты-android)
5. [UI/UX и Jetpack Compose](#uiux-и-jetpack-compose)
6. [Работа с данными](#работа-с-данными)
7. [Сетевое взаимодействие](#сетевое-взаимодействие)
8. [Архитектура и паттерны](#архитектура-и-паттерны)
9. [Практические задачи](#практические-задачи)
10. [Вопросы для собеседования](#вопросы-для-собеседования)
11. [Полезные ресурсы](#полезные-ресурсы)

---

## 🎯 Общий план подготовки

## ⚡ ЭКСТРЕННАЯ ПОДГОТОВКА ЗА 2 ДНЯ

### День 1 (8 часов): Основы современного Android стека
**Утро (4 часа):**
- [ ] **Kotlin продвинутые возможности** (2 часа): coroutines, flows, sealed classes
- [ ] **Android жизненный цикл** (1 час): Activity, Fragment, ViewModel lifecycle
- [ ] **MVVM архитектура** (1 час): ViewModel, LiveData, State management

**Вечер (4 часа):**
- [ ] **Jetpack Compose** (3 часа): State, Composables, Modifiers, Effects
- [ ] **Navigation Compose** (1 час): навигация между экранами

### День 2 (8 часов): Архитектура + Практика
**Утро (4 часа):**
- [ ] **Hilt Dependency Injection** (1.5 часа): @Inject, @Module, @Provides
- [ ] **Room Database** (1.5 часа): Entity, Dao, Database с Compose
- [ ] **Retrofit + Coroutines** (1 час): API calls с современным стеком

**Вечер (4 часа):**
- [ ] **Создание простого приложения** (2 часа): ToDo app с Compose + Room + Hilt
- [ ] **Повторение вопросов** (1 час): типичные вопросы интервью
- [ ] **Live coding практика** (1 час): решение задач на время

### 🚨 КРИТИЧЕСКИ ВАЖНЫЕ ТЕМЫ ДЛЯ JUNIOR
1. **Kotlin Coroutines** - обязательно знать
2. **Jetpack Compose основы** - современный UI
3. **MVVM + ViewModel** - архитектурный паттерн
4. **Hilt DI** - внедрение зависимостей
5. **Room Database** - локальное хранение
6. **Retrofit** - сетевые запросы

---

## 💻 Основы Kotlin (Современный стек)

### 🚀 Kotlin Coroutines - КЛЮЧЕВАЯ ТЕМА!

📝 **ЧТО ТАКОЕ COROUTINES:**
Coroutines - это легковесные потоки в Kotlin, которые позволяют писать асинхронный код так, как будто он синхронный. Они не блокируют поток выполнения и могут быть приостановлены и возобновлены позже.

🔑 **КЛЮЧЕВЫЕ ПРЕИМУЩЕСТВА:**
- **Легковесность**: миллионы coroutines могут работать одновременно
- **Отсутствие блокировки**: основной поток остается свободным
- **Структурированность**: автоматическая отмена при завершении родительского scope
- **Простота**: код выглядит как обычный последовательный код

🔄 **ОСНОВНЫЕ КОНЦЕПЦИИ:**
- **suspend** - ключевое слово для функций, которые могут быть приостановлены
- **launch** - запускает новую coroutine, не возвращает результат (fire-and-forget)
- **async** - запускает coroutine и возвращает Deferred для получения результата
- **Flow** - асинхронные потоки данных, могут эмитировать множество значений
- **StateFlow** - горячий поток для хранения состояния

```kotlin
// Основы Coroutines
class UserRepository {
    private val api = ApiService.create()
    
    suspend fun getUser(id: Int): User {
        return withContext(Dispatchers.IO) {
            api.getUser(id)
        }
    }
    
    // Flow - реактивные потоки
    fun getUsersFlow(): Flow<List<User>> = flow {
        while (true) {
            emit(api.getUsers())
            delay(30000) // Обновление каждые 30 сек
        }
    }.flowOn(Dispatchers.IO)
}

// ViewModel с Coroutines
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    val users: StateFlow<List<User>> = _users.asStateFlow()
    
    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                repository.getUsersFlow()
                    .catch { exception -> 
                        // Обработка ошибок
                    }
                    .collect { userList ->
                        _users.value = userList
                    }
            } finally {
                _isLoading.value = false
            }
        }
    }
}

// Параллельное выполнение
suspend fun loadUserData(userId: Int) {
    coroutineScope {
        val userDeferred = async { userRepository.getUser(userId) }
        val postsDeferred = async { postRepository.getUserPosts(userId) }
        
        val user = userDeferred.await()
        val posts = postsDeferred.await()
        
        // Оба запроса выполняются параллельно!
    }
}
```

### 💫 StateFlow vs LiveData vs Flow

📝 **ПОНИМАНИЕ РАЗЛИЧИЙ:**

**StateFlow** - это **горячий поток** (сразу предоставляет текущее значение) для хранения состояния UI. Всегда имеет значение.

**Flow** - это **холодный поток** (начинает работу только при подписке) для потоков данных. Может эмитировать множество значений со временем.

**LiveData** - это **Android-специфичный** класс для наблюдения за данными. Автоматически учитывает lifecycle (приостанавливается при onStop).

🔄 **КОГДА ЧТО ИСПОЛЬЗОВАТЬ:**
- **StateFlow** → для состояния UI (список пользователей, состояние загрузки)
- **Flow** → для потоков данных (поиск, фильтрация, сетевые запросы)
- **LiveData** → когда нужна тесная связь с lifecycle (Fragment/Activity)

```kotlin
// StateFlow - для state management
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()
    
    fun increment() {
        _count.value += 1
    }
}

// Flow - для потоков данных
fun searchUsers(query: String): Flow<List<User>> = flow {
    emit(repository.searchUsers(query))
}.flowOn(Dispatchers.IO)

// LiveData - традиционный подход (знать нужно!)
class OldViewModel : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}
```

### 🗂️ Sealed Classes для State Management

📝 **ПОНИМАНИЕ SEALED CLASSES:**
Sealed classes - это **ограниченная иерархия классов**. Они позволяют определить все возможные варианты состояния в одном месте, что делает код более безопасным и понятным.

🔑 **ПРЕИМУЩЕСТВА:**
- **Полнота when**: компилятор проверяет, что обработаны все случаи
- **Типобезопасность**: невозможно создать невалидное состояние
- **Рефакторинг**: легко добавлять новые состояния
- **Отладка**: легко отслеживать переходы между состояниями

🎁 **ПОЛЬЗА В ANDROID:**
В Android разработке sealed classes часто используют для:
- **UI состояний**: загрузка, успех, ошибка, пусто
- **Навигации**: экраны и маршруты
- **Сетевых результатов**: успех, ошибка, загрузка

```kotlin
// Мощный паттерн для обработки состояний
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val exception: Throwable) : UiState<Nothing>()
    object Empty : UiState<Nothing>()
}

class ModernViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState<List<User>>>(UiState.Empty)
    val uiState: StateFlow<UiState<List<User>>> = _uiState.asStateFlow()
    
    fun loadUsers() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val users = repository.getUsers()
                _uiState.value = if (users.isNotEmpty()) {
                    UiState.Success(users)
                } else {
                    UiState.Empty
                }
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e)
            }
        }
    }
}
```

### 🔧 Extension Functions (часто спрашивают!)

📝 **ПОНИМАНИЕ EXTENSION FUNCTIONS:**
Extension functions - это способ **добавить новую функциональность к существующим классам** без их изменения. Они позволяют писать более читаемый и выразительный код.

🔑 **ПРЕИМУЩЕСТВА:**
- **Читаемость**: код становится ближе к естественному языку
- **Повторное использование**: однажды написанное расширение можно использовать всюду
- **Инкапсуляция**: логика собрана в одном месте
- **Нет наследования**: можно расширять даже final классы

🎁 **ПОПУЛЯРНЫЕ ПРИМеРЫ В ANDROID:**

```kotlin
// Полезные расширения для Android
fun Context.showToast(message: String, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}

fun View.visible() {
    visibility = View.VISIBLE
}

fun View.gone() {
    visibility = View.GONE
}

fun String.isValidEmail(): Boolean {
    return android.util.Patterns.EMAIL_ADDRESS.matcher(this).matches()
}

// Использование:
context.showToast("Привет!")
progressBar.gone()
email.isValidEmail()
```

### 📊 Scope Functions (ОЧЕНЬ ВАЖНО!)

📝 **ПОНИМАНИЕ SCOPE FUNCTIONS:**
Scope functions - это функции, которые **выполняют блок кода в контексте объекта**. Они сокращают количество кода и делают его более выразительным.

🔑 **ОСНОВНЫЕ SCOPE FUNCTIONS:**

**let** - для **null-safety проверок**. Выполняет блок только если объект не null

**apply** - для **настройки объектов**. Возвращает сам объект, позволяет строить цепочки

**run** - для **выполнения вычислений**. Возвращает результат последней строки

**with** - для **группировки операций** над объектом. Не нужно повторять имя объекта

**also** - для **дополнительных действий** (логирование, валидация)

🎁 **КОГДА КАКУЮ ИСПОЛЬЗОВАТЬ:**

```kotlin
// let - для null-safety
user?.let { user ->
    binding.userName.text = user.name
    binding.userEmail.text = user.email
}

// apply - для настройки объектов
val textView = TextView(context).apply {
    text = "Привет"
    textSize = 16f
    setTextColor(Color.BLACK)
}

// run - выполнение блока кода
val result = run {
    val a = 10
    val b = 20
    a + b // возвращаемое значение
}

// with - работа с объектом
with(binding) {
    userName.text = user.name
    userEmail.text = user.email
    userAge.text = user.age.toString()
}
```

### 🚀 КЛЮЧЕВЫЕ ТЕМЫ ДЛЯ ИНТЕРВЬЮ:
- **Coroutines**: suspend, launch, async, withContext
- **Flow vs StateFlow**: когда что использовать
- **Null Safety**: `?`, `!!`, `?.`, `?:`, `let`
- **Collections**: `map`, `filter`, `forEach`, `groupBy`
- **Data classes**: автогенерация `equals`, `hashCode`, `toString`
- **Sealed classes**: альтернатива enum для сложных состояний

### Важные темы для изучения:
- **Null Safety**: `?`, `!!`, `?.`, `?:`
- **Collections**: List, Set, Map и их операции
- **Lambda expressions**: `{ }`, `it`, higher-order functions
- **Coroutines**: suspend, launch, async, withContext
- **Scope functions**: let, run, with, apply, also

### Полезные ресурсы:
- [Kotlin Koans](https://kotlinlang.org/docs/koans.html) - интерактивные упражнения
- [Kotlin by JetBrains](https://kotlinlang.org/docs/home.html) - официальная документация
- [Kotlin for Android Developers](https://developer.android.com/kotlin) - Android-специфичный Kotlin

---

## 🤖 Android основы

📝 **ПОНИМАНИЕ ANDROID LIFECYCLE:**
Жизненный цикл - это **последовательность состояний**, через которые проходит компонент Android (от создания до уничтожения). Понимание lifecycle критически важно для:
- **Оптимизации производительности**: остановка ненужных операций
- **Предотвращения крашей**: обращение к UI только в правильном состоянии
- **Управления ресурсами**: освобождение памяти и сетевых соединений

### Жизненный цикл Activity

🔄 **ОСНОВНЫЕ СОСТОЯНИЯ:**
- **onCreate()** - Activity создана, но еще не видна. Инициализация UI.
- **onStart()** - Activity становится видимой, но не в фокусе
- **onResume()** - Activity активна и в фокусе. Пользователь может взаимодействовать
- **onPause()** - Activity теряет фокус (например, открылось диалоговое окно)
- **onStop()** - Activity не видна (перекрыта другой Activity)
- **onDestroy()** - Activity уничтожается

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Инициализация
        Log.d("Lifecycle", "onCreate")
    }
    
    override fun onStart() {
        super.onStart()
        Log.d("Lifecycle", "onStart")
    }
    
    override fun onResume() {
        super.onResume()
        Log.d("Lifecycle", "onResume")
    }
    
    override fun onPause() {
        super.onPause()
        Log.d("Lifecycle", "onPause")
    }
    
    override fun onStop() {
        super.onStop()
        Log.d("Lifecycle", "onStop")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        Log.d("Lifecycle", "onDestroy")
    }
}
```

### Жизненный цикл Fragment

📝 **ОСОБЕННОСТИ FRAGMENT LIFECYCLE:**
Fragment имеет **более сложный** жизненный цикл, чем Activity, потому что он зависит от жизненного цикла **хост-Activity** и своего собственного состояния.

🔑 **КЛЮЧЕВЫЕ МОМЕНТЫ:**
- **onCreateView()** - создание UI иерархии Fragment
- **onViewCreated()** - UI создано, можно настраивать View
- **onDestroyView()** - UI уничтожается (например, при навигации)

⚠️ **ВАЖНО**: Fragment может пережить свою View! Поэтому настройку UI делаем в onViewCreated(), а очистку - в onDestroyView().

```kotlin
class MyFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        return inflater.inflate(R.layout.fragment_my, container, false)
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // Настройка UI
    }
}
```

### Ключевые концепции:
- **Manifest**: разрешения, компоненты, intent-filters
- **Resources**: strings, colors, dimensions, styles
- **Layouts**: ConstraintLayout, LinearLayout, RelativeLayout
- **View Binding**: безопасный доступ к View элементам
- **Intent**: явные и неявные, передача данных

### Ресурсы для изучения:
- [Android Developer Guides](https://developer.android.com/guide) - официальная документация
- [Android Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle) - жизненный цикл компонентов
- [Android Codelab](https://developer.android.com/codelabs) - практические задания

---

## 🧩 Компоненты Android

### Activity
```kotlin
// Запуск новой Activity
val intent = Intent(this, SecondActivity::class.java)
intent.putExtra("key", "value")
startActivity(intent)

// Получение данных
val data = intent.getStringExtra("key")
```

### Fragment
```kotlin
// Добавление Fragment
supportFragmentManager.beginTransaction()
    .replace(R.id.container, MyFragment())
    .addToBackStack(null)
    .commit()
```

### Service
```kotlin
class MyService : Service() {
    override fun onBind(intent: Intent?): IBinder? = null
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Выполнение фоновой работы
        return START_STICKY
    }
}
```

### BroadcastReceiver
```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        when (intent?.action) {
            Intent.ACTION_BATTERY_LOW -> {
                // Обработка события
            }
        }
    }
}
```

### Content Provider
```kotlin
class MyContentProvider : ContentProvider() {
    override fun query(/* parameters */): Cursor? {
        // Возврат данных
        return null
    }
    // Другие методы CRUD
}
```

### Важные темы:
- **Intent и Intent Filters**
- **Permissions**: runtime и manifest permissions
- **AsyncTask vs Coroutines**
- **Handler и Looper**
- **WorkManager** для фоновых задач

---

## 🎨 Jetpack Compose - Современный UI

📝 **ФИЛОСОФИЯ JETPACK COMPOSE:**
Jetpack Compose представляет собой **декларативный подход** к созданию UI, что кардинально отличается от традиционного императивного подхода с XML. Вместо описания "КАК" создать UI (findViewById, setText и т.д.), мы описываем "ЧТО" мы хотим видеть на экране.

🔑 **КЛЮЧЕВЫЕ ПРИНЦИПЫ:**
- **Декларативность**: описываем состояние UI, а не шаги для его создания
- **Композиция**: маленькие функции объединяются в сложные UI
- **Реактивность**: UI автоматически обновляется при изменении данных
- **Unidirectional Data Flow**: данные текут в одном направлении (сверху вниз)
- **State Hoisting**: состояние поднимается на уровень выше для переиспользования

💡 **ПРЕИМУЩЕСТВА COMPOSE:**
- **Меньше кода**: нет XML, ViewBinding, findViewById
- **Типобезопасность**: все проверяется на этапе компиляции
- **Интероперабельность**: можно использовать с существующими View
- **Производительность**: умные recomposition и пропуск неизменившихся частей
- **Современность**: использует последние возможности Kotlin

🔄 **ЧТО ТАКОЕ RECOMPOSITION:**
Recomposition - это процесс **повторного выполнения @Composable функций** при изменении их входных параметров или наблюдаемого состояния. Это основа реактивности Compose.

**Важные особенности recomposition:**
- **Автоматичность**: происходит без вашего участия
- **Оптимизированность**: перевычисляются только изменившиеся части
- **Умность**: Compose запоминает, какие части зависят от каких данных
- **Идемпотентность**: повторный вызов с теми же параметрами должен давать тот же результат

📊 **STATE MANAGEMENT В COMPOSE:**

**remember** - сохраняет значение между recomposition
```kotlin
// Локальное состояние компонента
var count by remember { mutableStateOf(0) }
```

**rememberSaveable** - сохраняет состояние даже при пересоздании Activity
```kotlin
// Переживает поворот экрана
var text by rememberSaveable { mutableStateOf("") }
```

**derivedStateOf** - вычисляемое состояние, которое обновляется только при изменении зависимостей
```kotlin
val isScrolled by remember {
    derivedStateOf { scrollState.value > 0 }
}
```

🏗️ **STATE HOISTING PATTERN:**
State hoisting - это паттерн **поднятия состояния** на уровень выше, чтобы сделать компоненты переиспользуемыми и тестируемыми.

**Принципы State Hoisting:**
1. **Stateless компоненты**: не содержат собственного состояния
2. **События вверх**: передаем callbacks для обработки событий
3. **Данные вниз**: передаем текущее состояние как параметры
4. **Single Source of Truth**: состояние хранится в одном месте

⚡ **SIDE EFFECTS В COMPOSE:**
Side effects - это операции, которые **выходят за рамки рендеринга UI**: сетевые запросы, работа с базой данных, навигация и т.д.

**LaunchedEffect** - запускается при входе в composition или изменении ключей
```kotlin
LaunchedEffect(userId) {
    // Выполнится при изменении userId
    viewModel.loadUser(userId)
}
```

**DisposableEffect** - для ресурсов, которые нужно очистить
```kotlin
DisposableEffect(Unit) {
    val listener = LocationListener()
    locationManager.addListener(listener)
    
    onDispose {
        locationManager.removeListener(listener)
    }
}
```

**SideEffect** - выполняется при каждой успешной recomposition
```kotlin
SideEffect {
    // Синхронизация с внешними системами
    analytics.trackScreenView("UserProfile")
}
```

🎯 **ПРОИЗВОДИТЕЛЬНОСТЬ В COMPOSE:**

**Оптимизации recomposition:**
- Используйте **stable types** (примитивы, data classes с неизменяемыми полями)
- Применяйте **key()** для списков с изменяющимися элементами
- **Избегайте lambda в Composable параметрах** если они меняются
- Используйте **remember** для дорогих вычислений

### 🚀 Основы Compose (ОБЯЗАТЕЛЬНО ЗНАТЬ!)
```kotlin
@Composable
fun UserCard(user: User, onUserClick: (User) -> Unit) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp)
            .clickable { onUserClick(user) },
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = user.name,
                style = MaterialTheme.typography.headlineSmall,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = user.email,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}

// Лист пользователей (LazyColumn)
@Composable
fun UserList(
    users: List<User>,
    onUserClick: (User) -> Unit,
    modifier: Modifier = Modifier
) {
    LazyColumn(
        modifier = modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(users) { user ->
            UserCard(
                user = user,
                onUserClick = onUserClick
            )
        }
    }
}
```

### 📋 State Management в Compose
```kotlin
// remember - для локального состояния
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    
    Column(
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = "Count: $count",
            style = MaterialTheme.typography.headlineMedium
        )
        
        Row {
            Button(onClick = { count-- }) {
                Text("-")
            }
            Spacer(modifier = Modifier.width(16.dp))
            Button(onClick = { count++ }) {
                Text("+")
            }
        }
    }
}

// Состояние из ViewModel
@Composable
fun UserScreen(
    viewModel: UserViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    
    when (uiState) {
        is UiState.Loading -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
        is UiState.Success -> {
            UserList(
                users = uiState.data,
                onUserClick = { user ->
                    // Навигация к деталям пользователя
                }
            )
        }
        is UiState.Error -> {
            ErrorScreen(
                message = uiState.exception.message ?: "Неизвестная ошибка",
                onRetry = { viewModel.loadUsers() }
            )
        }
        is UiState.Empty -> {
            EmptyScreen()
        }
    }
}
```

### 🛣️ Navigation Compose
```kotlin
// Настройка навигации
@Composable
fun AppNavigation() {
    val navController = rememberNavController()
    
    NavHost(
        navController = navController,
        startDestination = "users"
    ) {
        composable("users") {
            UserScreen(
                onUserClick = { user ->
                    navController.navigate("user/${user.id}")
                }
            )
        }
        
        composable(
            route = "user/{userId}",
            arguments = listOf(navArgument("userId") { type = NavType.IntType })
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getInt("userId") ?: 0
            UserDetailScreen(
                userId = userId,
                onBackClick = { navController.popBackStack() }
            )
        }
    }
}

// Bottom Navigation
@Composable
fun MainScreen() {
    val navController = rememberNavController()
    
    Scaffold(
        bottomBar = {
            NavigationBar {
                NavigationBarItem(
                    icon = { Icon(Icons.Default.Person, contentDescription = null) },
                    label = { Text("Пользователи") },
                    selected = true,
                    onClick = { /* Навигация */ }
                )
                NavigationBarItem(
                    icon = { Icon(Icons.Default.Settings, contentDescription = null) },
                    label = { Text("Настройки") },
                    selected = false,
                    onClick = { /* Навигация */ }
                )
            }
        }
    ) { paddingValues ->
        NavHost(
            navController = navController,
            startDestination = "users",
            modifier = Modifier.padding(paddingValues)
        ) {
            composable("users") { UserScreen() }
            composable("settings") { SettingsScreen() }
        }
    }
}
```

### 🏠 Compose Effects (ВАЖНО!)
```kotlin
// LaunchedEffect - для side effects
@Composable
fun UserScreen(userId: Int, viewModel: UserViewModel = hiltViewModel()) {
    LaunchedEffect(userId) {
        viewModel.loadUser(userId) // Вызов при смене userId
    }
    
    // UI код...
}

// DisposableEffect - для очистки ресурсов
@Composable
fun LocationScreen() {
    DisposableEffect(Unit) {
        val locationManager = LocationManager()
        locationManager.startTracking()
        
        onDispose {
            locationManager.stopTracking()
        }
    }
}

// rememberCoroutineScope - для manual coroutines
@Composable
fun ScrollableScreen() {
    val listState = rememberLazyListState()
    val coroutineScope = rememberCoroutineScope()
    
    Column {
        Button(
            onClick = {
                coroutineScope.launch {
                    listState.animateScrollToItem(0)
                }
            }
        ) {
            Text("К началу")
        }
        
        LazyColumn(state = listState) {
            // Контент...
        }
    }
}
```

### 📱 Material Design 3 темы
```kotlin
// Кастомная тема
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        darkTheme -> darkColorScheme(
            primary = Color(0xFF90CAF9),
            onPrimary = Color(0xFF003258),
            primaryContainer = Color(0xFF004881)
        )
        else -> lightColorScheme(
            primary = Color(0xFF1976D2),
            onPrimary = Color.White,
            primaryContainer = Color(0xFFBBDEFB)
        )
    }
    
    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}

// Типография
val Typography = Typography(
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Bold,
        fontSize = 32.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp
    )
)
```

### 🚀 КЛЮЧЕВЫЕ КОНЦЕПЦИИ COMPOSE:
- **Recomposition**: когда и почему происходит
- **State hoisting**: поднятие состояния вверх
- **Side effects**: LaunchedEffect, DisposableEffect, rememberCoroutineScope
- **Performance**: remember, derivedStateOf, key()
- **Modifiers**: порядок важен!
- **Composition Local**: передача данных через дерево

### RecyclerView
```kotlin
class MyAdapter(private val items: List<String>) : 
    RecyclerView.Adapter<MyAdapter.ViewHolder>() {
    
    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val textView: TextView = view.findViewById(R.id.textView)
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_layout, parent, false)
        return ViewHolder(view)
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.textView.text = items[position]
    }
    
    override fun getItemCount() = items.size
}
```

### Ключевые темы:
- **ConstraintLayout**: создание адаптивных макетов
- **RecyclerView**: эффективное отображение списков
- **ViewPager2**: свайпы между экранами
- **Navigation Component**: навигация между экранами
- **Material Design**: следование гайдлайнам Google

---

## 🏗️ Архитектура и паттерны (Современный стек)

### 🔧 Hilt Dependency Injection (ОБЯЗАТЕЛЬНО!)

📝 **ФИЛОСОФИЯ DEPENDENCY INJECTION:**
Dependency Injection (DI) - это паттерн проектирования, при котором **зависимости объекта предоставляются извне**, а не создаются внутри самого объекта. Это основа современной архитектуры приложений.

🔑 **ПРОБЛЕМЫ БЕЗ DI:**
```kotlin
// Плохой пример - жесткая связь
class UserViewModel {
    private val repository = UserRepository() // Мы не можем заменить!
    private val analytics = FirebaseAnalytics() // Тяжело тестировать!
}

class UserRepository {
    private val apiService = ApiService() // Нельзя мокировать!
}
```

🌟 **ПРЕИМУЩЕСТВА DI:**
- **Развязка**: классы не зависят от конкретных реализаций
- **Тестируемость**: легко мокировать зависимости
- **Гибкость**: можно легко менять реализации
- **Масштабируемость**: легко добавлять новые части
- **Однообразность**: синглтоны создаются автоматически

🌯 **ПОЧЕМУ HILT, А НЕ DAGGER 2:**
Hilt - это **обертка над Dagger 2**, специально созданная для Android:
- **Меньше boilerplate кода**: нет нужды в @Component
- **Автоматическая интеграция** с Android компонентами
- **Предопределенные scopes**: SingletonComponent, ActivityComponent, и т.д.
- **Лучшая обработка ошибок**: понятные сообщения о сборке

📊 **ОСНОВНЫЕ КОМПОНЕНТЫ HILT:**

**@HiltAndroidApp** - помечает Application класс:
```kotlin
@HiltAndroidApp
class MyApplication : Application() {
    // Hilt автоматически создает ApplicationComponent
}
```

**@AndroidEntryPoint** - помечает Android компоненты:
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    // Теперь можно использовать @Inject
}

@AndroidEntryPoint
class UserFragment : Fragment() {
    // Тоже можно внедрять зависимости
}
```

**@Inject** - помечает места для внедрения:
```kotlin
class UserRepository @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) {
    // Hilt автоматически предоставит зависимости
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var repository: UserRepository
    
    @Inject
    lateinit var analytics: Analytics
}
```

📦 **HILT MODULES:**

**@Module** - определяет как создавать объекты:
```kotlin
@Module
@InstallIn(SingletonComponent::class) // Указываем scope
object NetworkModule {
    
    @Provides
    @Singleton // Синглтон на все приложение
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder().build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(okHttpClient: OkHttpClient): ApiService {
        return Retrofit.Builder()
            .client(okHttpClient)
            .build()
            .create(ApiService::class.java)
    }
}
```

**@Provides** vs **@Binds**:
```kotlin
// @Provides - для сложной логики создания
@Provides
fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
    return Room.databaseBuilder(
        context,
        AppDatabase::class.java,
        "app_database"
    ).build()
}

// @Binds - для простой привязки интерфейса к реализации
@Module
@InstallIn(ViewModelComponent::class)
abstract class RepositoryModule {
    
    @Binds
    abstract fun bindUserRepository(
        userRepositoryImpl: UserRepositoryImpl
    ): UserRepository
}
```

🎯 **HILT SCOPES (ОЧЕНЬ ВАЖНО!):**

**SingletonComponent** - живет все время жизни приложения:
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    @Provides
    @Singleton // Один экземпляр на все приложение
    fun provideDatabase(): AppDatabase = ...
}
```

**ViewModelComponent** - живет время жизни ViewModel:
```kotlin
@Module
@InstallIn(ViewModelComponent::class)
object ViewModelModule {
    @Provides
    @ViewModelScoped // Новый экземпляр для каждой ViewModel
    fun provideUserUseCase(): UserUseCase = ...
}
```

**ActivityComponent** - живет время жизни Activity:
```kotlin
@Module
@InstallIn(ActivityComponent::class)
object ActivityModule {
    @Provides
    @ActivityScoped // Один экземпляр на Activity
    fun provideImageLoader(@ActivityContext context: Context): ImageLoader = ...
}
```

🎁 **QUALIFIERS (КВАЛИФИКАТОРЫ):**
Используются когда нужно различать объекты одного типа:
```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class DatabaseName

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class ApiBaseUrl

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @DatabaseName
    fun provideDatabaseName(): String = "app_database"
    
    @Provides
    @ApiBaseUrl
    fun provideApiBaseUrl(): String = "https://api.example.com/"
}

class Repository @Inject constructor(
    @DatabaseName private val dbName: String,
    @ApiBaseUrl private val baseUrl: String
) {
    // Теперь Hilt знает, какую строку куда передать
}
```

```kotlin
// Application class
@HiltAndroidApp
class MyApplication : Application()

// Modules
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }
    
    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        ).build()
    }
    
    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}

@Module
@InstallIn(ViewModelComponent::class)
object RepositoryModule {
    
    @Provides
    fun provideUserRepository(
        apiService: ApiService,
        userDao: UserDao
    ): UserRepository {
        return UserRepositoryImpl(apiService, userDao)
    }
}
```

### 📁 Repository Pattern (Современный подход)
```kotlin
// Repository Interface
interface UserRepository {
    fun getUsers(): Flow<List<User>>
    suspend fun getUserById(id: Int): User?
    suspend fun refreshUsers(): Result<Unit>
    suspend fun createUser(user: User): Result<User>
    suspend fun deleteUser(userId: Int): Result<Unit>
}

// Repository Implementation
@Singleton
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao,
    private val ioDispatcher: CoroutineDispatcher = Dispatchers.IO
) : UserRepository {
    
    override fun getUsers(): Flow<List<User>> = flow {
        // Сначала показываем кэшированные данные
        emitAll(userDao.getAllUsersFlow())
        
        // Затем обновляем с сервера
        try {
            refreshUsers()
        } catch (e: Exception) {
            // Логирование ошибки, но не прерываем Flow
            Log.e("UserRepository", "Failed to refresh users", e)
        }
    }.flowOn(ioDispatcher)
    
    override suspend fun refreshUsers(): Result<Unit> = withContext(ioDispatcher) {
        try {
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                response.body()?.let { users ->
                    userDao.deleteAllUsers()
                    userDao.insertUsers(users)
                }
                Result.success(Unit)
            } else {
                Result.failure(HttpException(response))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun getUserById(id: Int): User? = withContext(ioDispatcher) {
        userDao.getUserById(id)
    }
    
    override suspend fun createUser(user: User): Result<User> = withContext(ioDispatcher) {
        try {
            val response = apiService.createUser(user)
            if (response.isSuccessful) {
                response.body()?.let { createdUser ->
                    userDao.insertUser(createdUser)
                    Result.success(createdUser)
                } ?: Result.failure(Exception("Empty response body"))
            } else {
                Result.failure(HttpException(response))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun deleteUser(userId: Int): Result<Unit> = withContext(ioDispatcher) {
        try {
            val response = apiService.deleteUser(userId)
            if (response.isSuccessful) {
                userDao.deleteUserById(userId)
                Result.success(Unit)
            } else {
                Result.failure(HttpException(response))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 📊 ViewModel с современным стеком

📝 **ПОНИМАНИЕ MVVM АРХИТЕКТУРЫ:**
MVVM (Model-View-ViewModel) - это архитектурный паттерн, который **разделяет ответственность** между бизнес-логикой, представлением данных и UI. Это делает код более тестируемым, масштабируемым и поддерживаемым.

🔑 **КОМПОНЕНТЫ MVVM:**

**Model** - это **слой данных**:
- Repository, Data Sources, API
- Бизнес-логика работы с данными
- Не знает о View и ViewModel

**View** - это **слой представления**:
- Activity, Fragment, Composable функции
- Отвечает за отображение UI
- Наблюдает за ViewModel и реагирует на изменения
- Передает пользовательские события в ViewModel

**ViewModel** - это **мост между Model и View**:
- Хранит состояние UI (StateFlow/LiveData)
- Обрабатывает пользовательские действия
- Взаимодействует с Model для получения/обновления данных
- Переживает повороты экрана

💡 **ПОЧЕМУ ViewModel ПЕРЕЖИВАЕТ ПОВОРОТЫ:**
ViewModel создается **ViewModelProvider** и привязывается к **ViewModelStore**, который хранится в **ViewModelStoreOwner** (обычно Activity). При повороте экрана:

1. **Activity пересоздается** (вызывается onDestroy → onCreate)
2. **ViewModelStore остается в памяти** благодаря **retained fragments** или **onRetainCustomNonConfigurationInstance**
3. **Новая Activity получает тот же ViewModelStore** и восстанавливает ViewModel
4. **ViewModel.onCleared()** вызывается только при окончательном уничтожении Activity

🌊 **РАЗЛИЧИЯ StateFlow, LiveData И Flow:**

**StateFlow** (современный подход):
- Часть Kotlin Coroutines
- **Горячий поток** - всегда имеет текущее значение
- Можно использовать вне Android (мультиплатформенность)
- Работает с suspend функциями
- Нужно вручную управлять lifecycle

**LiveData** (традиционный):
- Android-специфичный
- **Lifecycle-aware** - автоматически приостанавливается при onStop
- Не работает с coroutines напрямую
- Легко использовать в Activity/Fragment

**Flow** (для потоков данных):
- **Холодный поток** - начинает работу при подписке
- Может эмитировать множество значений
- Поддерживает операторы (map, filter, combine)
- Идеально для асинхронных операций и трансформаций

💵 **КОГДА ЧТО ИСПОЛЬЗОВАТЬ:**
- **StateFlow** → для UI состояния, которое всегда имеет значение
- **Flow** → для потоков данных (поиск, фильтрация, сетевые запросы)
- **LiveData** → когда нужна автоматическая связь с lifecycle

🛡️ **VIEWMODEL SCOPES И ОЧИСТКА:**

**viewModelScope** - это **CoroutineScope**, который:
- Автоматически отменяет все coroutines при onCleared()
- Использует **Dispatchers.Main.immediate** по умолчанию
- Предотвращает **memory leaks** и **IllegalStateException**

**Правильная очистка ресурсов:**
```kotlin
class UserViewModel : ViewModel() {
    private val _users = MutableStateFlow<List<User>>(emptyList())
    
    override fun onCleared() {
        super.onCleared()
        // Очистка ресурсов (если нужна)
        // viewModelScope автоматически отменяет coroutines
    }
}
```

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val userRepository: UserRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<UiState<List<User>>>(UiState.Loading)
    val uiState: StateFlow<UiState<List<User>>> = _uiState.asStateFlow()
    
    private val _isRefreshing = MutableStateFlow(false)
    val isRefreshing: StateFlow<Boolean> = _isRefreshing.asStateFlow()
    
    init {
        loadUsers()
    }
    
    fun loadUsers() {
        viewModelScope.launch {
            userRepository.getUsers()
                .catch { exception ->
                    _uiState.value = UiState.Error(exception)
                }
                .collect { users ->
                    _uiState.value = if (users.isNotEmpty()) {
                        UiState.Success(users)
                    } else {
                        UiState.Empty
                    }
                }
        }
    }
    
    fun refreshUsers() {
        viewModelScope.launch {
            _isRefreshing.value = true
            try {
                userRepository.refreshUsers().getOrThrow()
            } catch (e: Exception) {
                // Показываем ошибку
                _uiState.value = UiState.Error(e)
            } finally {
                _isRefreshing.value = false
            }
        }
    }
    
    fun createUser(name: String, email: String) {
        viewModelScope.launch {
            val newUser = User(id = 0, name = name, email = email)
            userRepository.createUser(newUser)
                .onSuccess {
                    // Пользователь создан, Flow автоматически обновится
                }
                .onFailure { exception ->
                    _uiState.value = UiState.Error(exception)
                }
        }
    }
    
    fun deleteUser(userId: Int) {
        viewModelScope.launch {
            userRepository.deleteUser(userId)
                .onFailure { exception ->
                    _uiState.value = UiState.Error(exception)
                }
        }
    }
}
```

### 📦 Room Database с современным стеком
```kotlin
// Entity
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "name") val name: String,
    @ColumnInfo(name = "email") val email: String,
    @ColumnInfo(name = "created_at") val createdAt: Long = System.currentTimeMillis()
)

// DAO с Flow
@Dao
interface UserDao {
    @Query("SELECT * FROM users ORDER BY name ASC")
    fun getAllUsersFlow(): Flow<List<User>>
    
    @Query("SELECT * FROM users WHERE id = :id")
    suspend fun getUserById(id: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<User>)
    
    @Delete
    suspend fun deleteUser(user: User)
    
    @Query("DELETE FROM users WHERE id = :userId")
    suspend fun deleteUserById(userId: Int)
    
    @Query("DELETE FROM users")
    suspend fun deleteAllUsers()
    
    @Query("SELECT COUNT(*) FROM users")
    suspend fun getUserCount(): Int
}

// Database
@Database(
    entities = [User::class],
    version = 1,
    exportSchema = false
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

// Type Converters
class Converters {
    @TypeConverter
    fun fromTimestamp(value: Long?): Date? {
        return value?.let { Date(it) }
    }
    
    @TypeConverter
    fun dateToTimestamp(date: Date?): Long? {
        return date?.time
    }
}
```

### 🌐 Retrofit с современным стеком
```kotlin
// API Interface
interface ApiService {
    @GET("users")
    suspend fun getUsers(): Response<List<User>>
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): Response<User>
    
    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
    
    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: Int,
        @Body user: User
    ): Response<User>
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int): Response<Unit>
    
    @GET("users/search")
    suspend fun searchUsers(@Query("q") query: String): Response<List<User>>
}

// Error Handling
class HttpException(private val response: Response<*>) : Exception() {
    override val message: String
        get() = "HTTP ${response.code()}: ${response.message()}"
}

// Network Response Wrapper
sealed class NetworkResult<T> {
    data class Success<T>(val data: T) : NetworkResult<T>()
    data class Error<T>(val message: String, val code: Int? = null) : NetworkResult<T>()
    class Loading<T> : NetworkResult<T>()
}

// Safe API Call
suspend fun <T> safeApiCall(
    dispatcher: CoroutineDispatcher = Dispatchers.IO,
    apiCall: suspend () -> Response<T>
): NetworkResult<T> {
    return withContext(dispatcher) {
        try {
            val response = apiCall()
            if (response.isSuccessful) {
                response.body()?.let { data ->
                    NetworkResult.Success(data)
                } ?: NetworkResult.Error("Пустой ответ")
            } else {
                NetworkResult.Error(
                    message = response.message(),
                    code = response.code()
                )
            }
        } catch (e: Exception) {
            NetworkResult.Error(e.message ?: "Неизвестная ошибка")
        }
    }
}
```

### 🚀 КЛЮЧЕВЫЕ ПАТТЕРНЫ:
- **MVVM**: Model-View-ViewModel с современным UI
- **Repository Pattern**: единый источник данных
- **Dependency Injection**: внедрение зависимостей через Hilt
- **Observer Pattern**: StateFlow, Flow для реактивности
- **Single Source of Truth**: Room как единый источник
- **Offline-First**: сначала кэш, потом сеть

---

## ⚡ ПРОДВИНУТЫЕ ТЕМЫ ANDROID РАЗРАБОТКИ

### 🚀 Performance Optimization (КРИТИЧНО ДЛЯ SENIOR ПОЗИЦИЙ)

📝 **ПОНИМАНИЕ ПРОИЗВОДИТЕЛЬНОСТИ:**
Производительность в Android - это не только скорость работы приложения, но и **эффективное использование ресурсов** (CPU, память, батарея, сеть). Плохая производительность приводит к:
- **ANR (Application Not Responding)** - блокировка UI потока более 5 секунд
- **Jank** - пропуск кадров, когда UI обновляется медленнее 60 FPS
- **Memory Leaks** - утечки памяти, ведущие к OutOfMemoryError
- **Battery Drain** - быстрая разрядка батареи

🎯 **ОСНОВНЫЕ ПРИНЦИПЫ ОПТИМИЗАЦИИ:**

**1. ГЛАВНЫЙ ПОТОК (UI Thread) ДОЛЖЕН БЫТЬ СВОБОДЕН:**
```kotlin
// ❌ ПЛОХО - блокировка UI потока
class BadActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Сетевой запрос в UI потоке - ANR гарантирован!
        val data = apiService.getData() // НИКОГДА ТАК НЕ ДЕЛАЙТЕ!
        updateUI(data)
    }
}

// ✅ ХОРОШО - асинхронная работа
class GoodActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        lifecycleScope.launch {
            try {
                val data = withContext(Dispatchers.IO) {
                    apiService.getData() // Выполняется в IO потоке
                }
                updateUI(data) // Возврат в UI поток
            } catch (e: Exception) {
                handleError(e)
            }
        }
    }
}
```

**2. ОПТИМИЗАЦИЯ LAYOUT:**
```kotlin
// ❌ ПЛОХО - вложенные LinearLayout
<LinearLayout>
    <LinearLayout>
        <LinearLayout>
            <TextView .../>
        </LinearLayout>
    </LinearLayout>
</LinearLayout>

// ✅ ХОРОШО - плоская структура с ConstraintLayout
<androidx.constraintlayout.widget.ConstraintLayout>
    <TextView 
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

**3. ОПТИМИЗАЦИЯ СПИСКОВ:**
```kotlin
// RecyclerView оптимизации
class OptimizedAdapter : RecyclerView.Adapter<ViewHolder>() {
    init {
        // Уведомляем о фиксированном размере
        setHasStableIds(true)
    }
    
    override fun getItemId(position: Int): Long {
        return items[position].id.toLong() // Стабильные ID
    }
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bind(items[position])
    }
}

// DiffUtil для умного обновления
class UserDiffCallback : DiffUtil.ItemCallback<User>() {
    override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem.id == newItem.id
    }
    
    override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
        return oldItem == newItem
    }
}

class ModernAdapter : ListAdapter<User, ViewHolder>(UserDiffCallback()) {
    // Автоматическая анимация изменений!
}
```

**4. MEMORY MANAGEMENT:**
```kotlin
// ❌ ПЛОХО - утечка памяти через static ссылку
class LeakyActivity : AppCompatActivity() {
    companion object {
        var context: Context? = null // УТЕЧКА!
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        context = this // Activity не будет очищена из памяти
    }
}

// ✅ ХОРОШО - правильное управление ресурсами
class ProperActivity : AppCompatActivity() {
    private var networkCallback: ConnectivityManager.NetworkCallback? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        networkCallback = object : ConnectivityManager.NetworkCallback() {
            override fun onAvailable(network: Network) {
                // Обработка
            }
        }
        
        connectivityManager.registerNetworkCallback(request, networkCallback!!)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        networkCallback?.let {
            connectivityManager.unregisterNetworkCallback(it)
        }
        networkCallback = null
    }
}
```

🔧 **ИНСТРУМЕНТЫ ПРОФИЛИРОВАНИЯ:**

**Android Studio Profiler:**
- **CPU Profiler**: отслеживание использования CPU, поиск горячих мест
- **Memory Profiler**: анализ использования памяти, поиск утечек
- **Network Profiler**: мониторинг сетевых запросов
- **Energy Profiler**: анализ потребления батареи

**Layout Inspector:**
```kotlin
// Анализ иерархии View в реальном времени
// Tools → Layout Inspector в Android Studio

// Поиск overdraw (перерисовка)
// Developer Options → Debug GPU overdraw
```

### 🧠 Memory Management (ОБЯЗАТЕЛЬНО ЗНАТЬ!)

📝 **ПОНИМАНИЕ ПАМЯТИ В ANDROID:**
Android использует **Dalvik/ART runtime** с автоматическим управлением памяти через **Garbage Collector (GC)**. Однако разработчик должен понимать:
- **Heap Memory**: основная память для объектов Java/Kotlin
- **Stack Memory**: локальные переменные и вызовы методов
- **Method Area**: код классов, константы, статические переменные
- **Native Memory**: JNI объекты, Bitmap до API 11

🚨 **ОСНОВНЫЕ ПРИЧИНЫ УТЕЧЕК ПАМЯТИ:**

**1. СТАТИЧЕСКИЕ ССЫЛКИ НА CONTEXT:**
```kotlin
// ❌ ПРОБЛЕМА
object BadSingleton {
    private var context: Context? = null // Держит ссылку на Activity!
    
    fun init(context: Context) {
        this.context = context
    }
}

// ✅ РЕШЕНИЕ
object GoodSingleton {
    private var appContext: Context? = null
    
    fun init(context: Context) {
        this.appContext = context.applicationContext // Безопасно!
    }
}
```

**2. НЕОТМЕНЕННЫЕ COROUTINES:**
```kotlin
// ❌ ПРОБЛЕМА
class BadViewModel : ViewModel() {
    fun loadData() {
        GlobalScope.launch { // Не отменяется при уничтожении ViewModel!
            val data = repository.getData()
            // Если ViewModel уничтожена, но coroutine работает - утечка!
        }
    }
}

// ✅ РЕШЕНИЕ
class GoodViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch { // Автоматическая отмена при onCleared()
            val data = repository.getData()
            updateUi(data)
        }
    }
}
```

**3. LISTENERS БЕЗ ОТПИСКИ:**
```kotlin
// ❌ ПРОБЛЕМА
class BadActivity : AppCompatActivity() {
    private val locationListener = object : LocationListener {
        override fun onLocationChanged(location: Location) {
            // Обработка
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        locationManager.requestLocationUpdates(
            LocationManager.GPS_PROVIDER,
            1000L,
            1f,
            locationListener
        )
        // НЕТ ОТПИСКИ В onDestroy - утечка!
    }
}

// ✅ РЕШЕНИЕ
class GoodActivity : AppCompatActivity() {
    private val locationListener = object : LocationListener {
        override fun onLocationChanged(location: Location) {
            // Обработка
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) 
            == PackageManager.PERMISSION_GRANTED) {
            locationManager.requestLocationUpdates(
                LocationManager.GPS_PROVIDER,
                1000L,
                1f,
                locationListener
            )
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        locationManager.removeUpdates(locationListener) // Обязательная очистка!
    }
}
```

**4. INNER CLASSES С НЕЯВНЫМИ ССЫЛКАМИ:**
```kotlin
// ❌ ПРОБЛЕМА
class BadActivity : AppCompatActivity() {
    
    inner class AsyncTask { // Держит ссылку на Activity!
        fun doWork() {
            // Долгая работа - Activity не может быть очищена
        }
    }
}

// ✅ РЕШЕНИЕ
class GoodActivity : AppCompatActivity() {
    
    class AsyncTask(private val contextRef: WeakReference<Context>) { // Слабая ссылка
        fun doWork() {
            val context = contextRef.get()
            if (context != null) {
                // Работа с контекстом
            }
        }
    }
    
    private val asyncTask = AsyncTask(WeakReference(this))
}
```

💡 **ИНСТРУМЕНТЫ ДЛЯ ПОИСКА УТЕЧЕК:**

**LeakCanary** (обязательно использовать!):
```kotlin
// В build.gradle (:app)
dependencies {
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.10'
}

// Автоматически обнаруживает утечки в debug сборке
```

### 🧪 Testing (СОВРЕМЕННЫЙ ПОДХОД)

📝 **ПОНИМАНИЕ ТЕСТИРОВАНИЯ В ANDROID:**
Тестирование - это **проверка корректности работы** приложения на разных уровнях. В Android различают:
- **Unit Tests**: изолированное тестирование бизнес-логики
- **Integration Tests**: тестирование взаимодействия компонентов
- **UI Tests**: автоматизированное тестирование пользовательского интерфейса
- **Instrumented Tests**: тесты, которые выполняются на устройстве/эмуляторе

🎯 **ПИРАМИДА ТЕСТИРОВАНИЯ:**
```
      /\
     /UI\
    /____\
   /Integration\
  /______________\
 /   Unit Tests   \
/________________\
```

**Unit Tests (70%)** - быстрые, дешевые, много
**Integration Tests (20%)** - средние по скорости и стоимости
**UI Tests (10%)** - медленные, дорогие, мало

🔬 **UNIT TESTING С MODERN STACK:**

**Тестирование ViewModel:**
```kotlin
@ExperimentalCoroutinesinesApi
class UserViewModelTest {
    
    @get:Rule
    val mainDispatcherRule = MainDispatcherRule()
    
    @Mock
    private lateinit var userRepository: UserRepository
    
    private lateinit var viewModel: UserViewModel
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        viewModel = UserViewModel(userRepository)
    }
    
    @Test
    fun `loadUsers should update uiState with success when repository returns data`() = runTest {
        // Given
        val expectedUsers = listOf(
            User(1, "John", "john@example.com"),
            User(2, "Jane", "jane@example.com")
        )
        whenever(userRepository.getUsers()).thenReturn(flowOf(expectedUsers))
        
        // When
        viewModel.loadUsers()
        
        // Then
        val uiState = viewModel.uiState.value
        assertThat(uiState).isInstanceOf(UiState.Success::class.java)
        assertThat((uiState as UiState.Success).data).isEqualTo(expectedUsers)
    }
    
    @Test
    fun `loadUsers should update uiState with error when repository throws exception`() = runTest {
        // Given
        val expectedException = RuntimeException("Network error")
        whenever(userRepository.getUsers()).thenReturn(flow { throw expectedException })
        
        // When
        viewModel.loadUsers()
        
        // Then
        val uiState = viewModel.uiState.value
        assertThat(uiState).isInstanceOf(UiState.Error::class.java)
        assertThat((uiState as UiState.Error).exception).isEqualTo(expectedException)
    }
}

// MainDispatcherRule для корректной работы coroutines в тестах
@ExperimentalCoroutinesApi
class MainDispatcherRule(private val dispatcher: TestDispatcher = UnconfinedTestDispatcher()) : TestWatcher() {
    
    override fun starting(description: Description?) {
        Dispatchers.setMain(dispatcher)
    }
    
    override fun finished(description: Description?) {
        Dispatchers.resetMain()
    }
}
```

**Тестирование Repository:**
```kotlin
class UserRepositoryTest {
    
    @Mock
    private lateinit var apiService: ApiService
    
    @Mock
    private lateinit var userDao: UserDao
    
    private lateinit var repository: UserRepositoryImpl
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        repository = UserRepositoryImpl(apiService, userDao)
    }
    
    @Test
    fun `getUsers should return cached data first then network data`() = runTest {
        // Given
        val cachedUsers = listOf(User(1, "Cached", "cached@example.com"))
        val networkUsers = listOf(User(2, "Network", "network@example.com"))
        
        whenever(userDao.getAllUsersFlow()).thenReturn(flowOf(cachedUsers))
        whenever(apiService.getUsers()).thenReturn(
            Response.success(networkUsers)
        )
        
        // When
        val result = repository.getUsers().toList()
        
        // Then
        assertThat(result).hasSize(2)
        assertThat(result[0]).isEqualTo(cachedUsers) // Сначала кэш
        assertThat(result[1]).isEqualTo(networkUsers) // Потом сеть
        
        verify(userDao).insertUsers(networkUsers) // Проверяем сохранение в кэш
    }
}
```

🎨 **COMPOSE UI TESTING:**

```kotlin
@RunWith(AndroidJUnit4::class)
class TaskAppUITest {
    
    @get:Rule
    val composeTestRule = createComposeRule()
    
    @Test
    fun addTaskDialog_shouldAddTaskWhenValidInput() {
        // Given
        composeTestRule.setContent {
            AddTaskDialog(
                onDismiss = { },
                onAddTask = { title, description, priority ->
                    // Проверяем, что вызывается с правильными параметрами
                }
            )
        }
        
        // When
        composeTestRule
            .onNodeWithText("Название")
            .performTextInput("Test Task")
        
        composeTestRule
            .onNodeWithText("Описание")
            .performTextInput("Test Description")
        
        composeTestRule
            .onNodeWithText("Высокий")
            .performClick()
        
        composeTestRule
            .onNodeWithText("Добавить")
            .performClick()
        
        // Then - проверяем через callback или состояние
    }
    
    @Test
    fun taskList_shouldDisplayTasks() {
        // Given
        val testTasks = listOf(
            Task(1, "Task 1", "Description 1"),
            Task(2, "Task 2", "Description 2")
        )
        
        // When
        composeTestRule.setContent {
            TaskList(
                tasks = testTasks,
                onTaskClick = { },
                onDeleteTask = { }
            )
        }
        
        // Then
        composeTestRule
            .onNodeWithText("Task 1")
            .assertIsDisplayed()
        
        composeTestRule
            .onNodeWithText("Task 2")
            .assertIsDisplayed()
    }
}
```

🔧 **ЗАВИСИМОСТИ ДЛЯ ТЕСТИРОВАНИЯ:**
```kotlin
// build.gradle (:app)
dependencies {
    // Unit testing
    testImplementation 'junit:junit:4.13.2'
    testImplementation 'org.mockito:mockito-core:4.6.1'
    testImplementation 'org.mockito.kotlin:mockito-kotlin:4.0.0'
    testImplementation 'androidx.arch.core:core-testing:2.2.0'
    testImplementation 'org.jetbrains.kotlinx:kotlinx-coroutines-test:1.6.4'
    testImplementation 'app.cash.turbine:turbine:0.12.1' // Для тестирования Flow
    testImplementation 'com.google.truth:truth:1.1.3'
    
    // Android Instrumented tests
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
    androidTestImplementation 'androidx.compose.ui:ui-test-junit4:1.4.3'
    
    // For Hilt testing
    testImplementation 'com.google.dagger:hilt-android-testing:2.44'
    androidTestImplementation 'com.google.dagger:hilt-android-testing:2.44'
    kaptAndroidTest 'com.google.dagger:hilt-android-compiler:2.44'
}
```

### 🔐 Security (БЕЗОПАСНОСТЬ)

📝 **ПОНИМАНИЕ БЕЗОПАСНОСТИ В ANDROID:**
Безопасность Android приложения включает **защиту данных пользователя**, предотвращение несанкционированного доступа и защиту от вредоносных атак.

🛡️ **ОСНОВНЫЕ ПРИНЦИПЫ БЕЗОПАСНОСТИ:**

**1. NETWORK SECURITY:**
```kotlin
// ✅ HTTPS ТОЛЬКО
class SecureApiClient {
    companion object {
        private const val BASE_URL = "https://api.example.com/" // ТОЛЬКО HTTPS!
        
        fun create(): ApiService {
            val okHttpClient = OkHttpClient.Builder()
                .certificatePinner(
                    CertificatePinner.Builder()
                        .add("api.example.com", "sha256/XXXXXX") // Certificate pinning
                        .build()
                )
                .build()
            
            return Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(okHttpClient)
                .build()
                .create(ApiService::class.java)
        }
    }
}

// Network Security Config (res/xml/network_security_config.xml)
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2024-12-31">
            <pin digest="SHA-256">XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

**2. DATA STORAGE SECURITY:**
```kotlin
// ✅ ЗАШИФРОВАННОЕ ХРАНЕНИЕ
class SecureStorage(context: Context) {
    
    private val sharedPreferences = EncryptedSharedPreferences.create(
        "secret_shared_prefs",
        MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC),
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    fun saveToken(token: String) {
        sharedPreferences.edit()
            .putString("auth_token", token)
            .apply()
    }
    
    fun getToken(): String? {
        return sharedPreferences.getString("auth_token", null)
    }
}

// ✅ ЗАШИФРОВАННАЯ БАЗА ДАННЫХ
@Database(
    entities = [User::class],
    version = 1
)
abstract class EncryptedAppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        fun create(context: Context): EncryptedAppDatabase {
            val passphrase: ByteArray = SQLiteDatabase.getBytes("passphrase".toCharArray())
            val factory = SupportFactory(passphrase)
            
            return Room.databaseBuilder(
                context,
                EncryptedAppDatabase::class.java,
                "encrypted_database"
            )
                .openHelperFactory(factory)
                .build()
        }
    }
}
```

**3. AUTHENTICATION & AUTHORIZATION:**
```kotlin
// ✅ БЕЗОПАСНАЯ АУТЕНТИФИКАЦИЯ
class AuthManager @Inject constructor(
    private val secureStorage: SecureStorage,
    private val apiService: ApiService
) {
    
    suspend fun login(email: String, password: String): Result<AuthToken> {
        return try {
            // Хэширование пароля (никогда не отправляем plain text!)
            val hashedPassword = hashPassword(password)
            
            val response = apiService.login(LoginRequest(email, hashedPassword))
            
            if (response.isSuccessful && response.body() != null) {
                val authToken = response.body()!!
                
                // Сохраняем токен безопасно
                secureStorage.saveToken(authToken.accessToken)
                secureStorage.saveRefreshToken(authToken.refreshToken)
                
                Result.success(authToken)
            } else {
                Result.failure(AuthException("Login failed"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    private fun hashPassword(password: String): String {
        val digest = MessageDigest.getInstance("SHA-256")
        val hash = digest.digest(password.toByteArray())
        return Base64.encodeToString(hash, Base64.NO_WRAP)
    }
    
    fun isLoggedIn(): Boolean {
        val token = secureStorage.getToken()
        return token != null && !isTokenExpired(token)
    }
    
    fun logout() {
        secureStorage.clearTokens()
    }
}
```

**4. BIOMETRIC AUTHENTICATION:**
```kotlin
class BiometricAuthManager(private val context: Context) {
    
    fun authenticateWithBiometrics(
        onSuccess: () -> Unit,
        onError: (String) -> Unit
    ) {
        val biometricPrompt = BiometricPrompt(
            context as FragmentActivity,
            ContextCompat.getMainExecutor(context),
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(
                    result: BiometricPrompt.AuthenticationResult
                ) {
                    super.onAuthenticationSucceeded(result)
                    onSuccess()
                }
                
                override fun onAuthenticationError(
                    errorCode: Int,
                    errString: CharSequence
                ) {
                    super.onAuthenticationError(errorCode, errString)
                    onError(errString.toString())
                }
            }
        )
        
        val promptInfo = BiometricPrompt.PromptInfo.Builder()
            .setTitle("Биометрическая аутентификация")
            .setSubtitle("Используйте отпечаток пальца для входа")
            .setNegativeButtonText("Отмена")
            .build()
        
        biometricPrompt.authenticate(promptInfo)
    }
    
    fun isBiometricAvailable(): Boolean {
        return BiometricManager.from(context)
            .canAuthenticate(BiometricManager.Authenticators.BIOMETRIC_WEAK) == BiometricManager.BIOMETRIC_SUCCESS
    }
}
```

🔒 **PERMISSIONS BEST PRACTICES:**
```kotlin
// ✅ RUNTIME PERMISSIONS
class PermissionManager(private val activity: AppCompatActivity) {
    
    private val requestPermissionLauncher = activity.registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { isGranted: Boolean ->
        if (isGranted) {
            // Разрешение получено
        } else {
            // Разрешение отклонено - показываем объяснение
            showPermissionRationale()
        }
    }
    
    fun requestLocationPermission() {
        when {
            ContextCompat.checkSelfPermission(
                activity,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) == PackageManager.PERMISSION_GRANTED -> {
                // Разрешение уже есть
                proceedWithLocationFeature()
            }
            
            activity.shouldShowRequestPermissionRationale(
                Manifest.permission.ACCESS_FINE_LOCATION
            ) -> {
                // Показываем объяснение, почему нужно разрешение
                showPermissionRationale()
            }
            
            else -> {
                // Запрашиваем разрешение
                requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
            }
        }
    }
    
    private fun showPermissionRationale() {
        AlertDialog.Builder(activity)
            .setTitle("Требуется разрешение")
            .setMessage("Приложению нужен доступ к местоположению для корректной работы")
            .setPositiveButton("Разрешить") { _, _ ->
                requestPermissionLauncher.launch(Manifest.permission.ACCESS_FINE_LOCATION)
            }
            .setNegativeButton("Отказаться", null)
            .show()
    }
}
```

---

## 💾 Работа с данными (СОВРЕМЕННЫЙ ПОДХОД)

📝 **ПОНИМАНИЕ СОВРЕМЕННОГО DATA LAYER:**
Современная работа с данными в Android основывается на принципе **Single Source of Truth** - единого источника правды для каждого типа данных. Это означает, что данные должны иметь одно место хранения и все компоненты приложения должны обращаться к этому источнику.

🔑 **АРХИТЕКТУРНЫЕ ПРИНЦИПЫ DATA LAYER:**
- **Repository Pattern** - единая точка доступа к данным
- **Offline-First** - приложение работает без интернета
- **Reactive Streams** - данные обновляются автоматически через Flow
- **Layered Caching** - многоуровневое кэширование (Memory → Disk → Network)
- **Data Consistency** - синхронизация между источниками данных

### 🏗️ Room Database (ОБЯЗАТЕЛЬНО ЗНАТЬ!)

📝 **ПОНИМАНИЕ ROOM:**
Room - это **ORM (Object-Relational Mapping)** библиотека, которая предоставляет абстракцию над SQLite. Room автоматически генерирует код для работы с базой данных и проверяет SQL запросы на этапе компиляции.

🔑 **ОСНОВНЫЕ КОМПОНЕНТЫ ROOM:**
- **@Entity** - представляет таблицу в базе данных
- **@Dao** - интерфейс для доступа к данным (Data Access Object)
- **@Database** - основной класс базы данных
- **@TypeConverter** - преобразователи для сложных типов данных
- **@Relation** - связи между таблицами

```kotlin
// Продвинутая Entity с различными аннотациями
@Entity(
    tableName = "users",
    indices = [
        Index(value = ["email"], unique = true),
        Index(value = ["created_at"])
    ],
    foreignKeys = [
        ForeignKey(
            entity = Department::class,
            parentColumns = ["id"],
            childColumns = ["department_id"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class User(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    
    @ColumnInfo(name = "first_name")
    val firstName: String,
    
    @ColumnInfo(name = "email")
    val email: String,
    
    @ColumnInfo(name = "department_id")
    val departmentId: Int?,
    
    @ColumnInfo(name = "created_at")
    val createdAt: Date = Date(),
    
    @Embedded
    val address: Address,
    
    @Ignore
    val fullName: String = "$firstName $lastName"
)

data class Address(
    val street: String,
    val city: String,
    @ColumnInfo(name = "postal_code")
    val postalCode: String
)
```

📊 **ПРОДВИНУТЫЙ DAO С РЕАКТИВНОСТЬЮ:**
```kotlin
@Dao
interface UserDao {
    // Flow для реактивных обновлений
    @Query("SELECT * FROM users WHERE is_active = 1 ORDER BY created_at DESC")
    fun getActiveUsersFlow(): Flow<List<User>>
    
    // Сложные запросы с JOIN
    @Query("""
        SELECT u.*, d.name as department_name 
        FROM users u 
        LEFT JOIN departments d ON u.department_id = d.id 
        WHERE u.is_active = 1
    """)
    fun getUsersWithDepartments(): Flow<List<UserWithDepartment>>
    
    // Поиск
    @Query("""
        SELECT * FROM users 
        WHERE (first_name LIKE '%' || :query || '%' 
            OR email LIKE '%' || :query || '%') 
            AND is_active = 1
        ORDER BY first_name ASC
    """)
    fun searchUsers(query: String): Flow<List<User>>
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUsers(users: List<User>): List<Long>
    
    @Transaction
    suspend fun refreshUsers(networkUsers: List<User>) {
        deleteAllUsers()
        insertUsers(networkUsers)
    }
}
```

### 📱 DataStore (ЗАМЕНА SharedPreferences)

📝 **ПОНИМАНИЕ РАЗЛИЧИЙ:**
**SharedPreferences** - традиционный способ хранения простых настроек приложения. Работает синхронно, может блокировать UI поток.

**DataStore** - современная замена SharedPreferences. Работает асинхронно с Kotlin Coroutines, типобезопасен, поддерживает миграции.

```kotlin
// Современный подход - Preferences DataStore
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

class DataStoreManager(private val context: Context) {
    
    companion object {
        private val USER_NAME_KEY = stringPreferencesKey("user_name")
        private val IS_LOGGED_IN_KEY = booleanPreferencesKey("is_logged_in")
        private val THEME_KEY = stringPreferencesKey("theme")
    }
    
    suspend fun saveUserName(name: String) {
        context.dataStore.edit { preferences ->
            preferences[USER_NAME_KEY] = name
        }
    }
    
    fun getUserName(): Flow<String> {
        return context.dataStore.data.map { preferences ->
            preferences[USER_NAME_KEY] ?: ""
        }
    }
    
    // Комбинирование нескольких значений
    fun getUserProfile(): Flow<UserProfile> {
        return context.dataStore.data.map { preferences ->
            UserProfile(
                name = preferences[USER_NAME_KEY] ?: "",
                isLoggedIn = preferences[IS_LOGGED_IN_KEY] ?: false
            )
        }
    }
}
```

### 🔄 Repository Pattern (СОВРЕМЕННАЯ АРХИТЕКТУРА)

📝 **ПОНИМАНИЕ REPOSITORY PATTERN:**
Repository - это **слой абстракции** между бизнес-логикой (ViewModel) и источниками данных (API, Database, Cache). Он обеспечивает единую точку доступа к данным и скрывает детали их получения.

🎯 **ПРИНЦИПЫ СОВРЕМЕННОГО REPOSITORY:**
- **Single Source of Truth**: данные из одного авторитетного источника
- **Offline-First**: сначала показываем кэш, потом обновляем с сервера
- **Reactive**: данные обновляются автоматически через Flow
- **Error Handling**: централизованная обработка ошибок
- **Caching Strategy**: умное кэширование на разных уровнях

```kotlin
interface UserRepository {
    fun getUsers(): Flow<List<User>>
    fun getUserById(id: Int): Flow<User?>
    suspend fun refreshUsers(): Result<Unit>
    suspend fun createUser(user: User): Result<User>
    fun searchUsers(query: String): Flow<List<User>>
}

@Singleton
class UserRepositoryImpl @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao,
    private val dataStoreManager: DataStoreManager,
    @IoDispatcher private val ioDispatcher: CoroutineDispatcher,
    private val networkMonitor: NetworkMonitor
) : UserRepository {
    
    // Memory cache для быстрого доступа
    private val memoryCache = LruCache<Int, User>(100)
    
    override fun getUsers(): Flow<List<User>> = flow {
        // 1. Сначала эмитим данные из локальной БД (быстро)
        emitAll(userDao.getActiveUsersFlow())
        
        // 2. Параллельно пытаемся обновить с сервера
        if (networkMonitor.isConnected.value) {
            try {
                refreshUsers()
            } catch (e: Exception) {
                Log.w("UserRepository", "Failed to refresh users", e)
            }
        }
    }.flowOn(ioDispatcher)
    
    override fun getUserById(id: Int): Flow<User?> = flow {
        // 1. Проверяем memory cache
        memoryCache.get(id)?.let { emit(it) }
        
        // 2. Проверяем локальную БД
        val localUser = userDao.getUserById(id)
        if (localUser != null) {
            memoryCache.put(id, localUser)
            emit(localUser)
        }
        
        // 3. Если нет локально и есть сеть - загружаем с сервера
        if (localUser == null && networkMonitor.isConnected.value) {
            try {
                val response = apiService.getUserById(id)
                if (response.isSuccessful && response.body() != null) {
                    val networkUser = response.body()!!
                    userDao.insertUser(networkUser)
                    memoryCache.put(id, networkUser)
                    emit(networkUser)
                }
            } catch (e: Exception) {
                Log.w("UserRepository", "Failed to fetch user $id", e)
            }
        }
    }.flowOn(ioDispatcher)
    
    override suspend fun refreshUsers(): Result<Unit> = withContext(ioDispatcher) {
        try {
            val response = apiService.getUsers()
            
            if (response.isSuccessful && response.body() != null) {
                val networkUsers = response.body()!!
                
                // Атомарное обновление БД
                userDao.refreshUsers(networkUsers)
                
                // Обновляем memory cache
                networkUsers.forEach { user ->
                    memoryCache.put(user.id, user)
                }
                
                Result.success(Unit)
            } else {
                Result.failure(HttpException(response))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun createUser(user: User): Result<User> = withContext(ioDispatcher) {
        try {
            // Сначала сохраняем локально с pending статусом
            val localUser = user.copy(syncStatus = SyncStatus.PENDING_CREATE)
            val localId = userDao.insertUser(localUser)
            
            if (networkMonitor.isConnected.value) {
                val response = apiService.createUser(user)
                
                if (response.isSuccessful && response.body() != null) {
                    val serverUser = response.body()!!
                    userDao.updateUser(serverUser.copy(syncStatus = SyncStatus.SYNCED))
                    memoryCache.put(serverUser.id, serverUser)
                    Result.success(serverUser)
                } else {
                    Result.failure(HttpException(response))
                }
            } else {
                // Offline - возвращаем локальную версию
                Result.success(localUser.copy(id = localId.toInt()))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

enum class SyncStatus {
    SYNCED, PENDING_CREATE, PENDING_UPDATE, PENDING_DELETE
}
```

---

## 🌐 Сетевое взаимодействие (СОВРЕМЕННЫЙ ПОДХОД)

📝 **ПОНИМАНИЕ СЕТЕВОЙ АРХИТЕКТУРЫ:**
Современные Android приложения полагаются на **RESTful API** для обмена данными с сервером. Ключевые компоненты:
- **Retrofit** - типобезопасный HTTP клиент для Android
- **OkHttp** - низкоуровневый HTTP клиент с поддержкой interceptors
- **JSON парсинг** - преобразование данных между клиентом и сервером
- **Error Handling** - обработка ошибок сетевых запросов
- **Caching** - кэширование сетевых ответов

### 🚀 Retrofit (ОБЯЗАТЕЛЬНО ЗНАТЬ!)

📝 **ПОНИМАНИЕ RETROFIT:**
Retrofit - это **типобезопасный HTTP клиент** для Android, который преобразует **HTTP API в интерфейс Kotlin**. Он автоматически сериализует/десериализует JSON, обрабатывает параметры запросов и поддерживает асинхронное выполнение.

```kotlin
// API Interface
interface ApiService {
    @GET("users")
    suspend fun getUsers(): Response<List<User>>
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): Response<User>
    
    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
    
    @PUT("users/{id}")
    suspend fun updateUser(
        @Path("id") id: Int,
        @Body user: User
    ): Response<User>
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") id: Int): Response<Unit>
    
    @GET("users/search")
    suspend fun searchUsers(@Query("q") query: String): Response<List<User>>
}

// Retrofit setup
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(HttpLoggingInterceptor().apply {
                level = HttpLoggingInterceptor.Level.BODY
            })
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }
    
    @Provides
    @Singleton
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .client(okHttpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

### 🔍 OkHttp Interceptors

📝 **ПОНИМАНИЕ INTERCEPTORS:**
Interceptors - это **механизм перехвата** HTTP запросов и ответов в OkHttp. Они позволяют **модифицировать, логировать, кэшировать** запросы.

```kotlin
// Аутентификация Interceptor
class AuthInterceptor(private val token: String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $token")
            .build()
        return chain.proceed(authenticatedRequest)
    }
}

// Логирование Interceptor
val loggingInterceptor = HttpLoggingInterceptor().apply {
    level = if (BuildConfig.DEBUG) {
        HttpLoggingInterceptor.Level.BODY
    } else {
        HttpLoggingInterceptor.Level.NONE
    }
}

// Кэширование
val cacheSize = 10 * 1024 * 1024L // 10 MB
val cache = Cache(File(context.cacheDir, "http_cache"), cacheSize)

val okHttpClient = OkHttpClient.Builder()
    .cache(cache)
    .addInterceptor(AuthInterceptor("your_token"))
    .addInterceptor(loggingInterceptor)
    .build()
```

### 📦 JSON парсинг с Gson

```kotlin
data class ApiResponse<T>(
    val success: Boolean,
    val data: T?,
    val message: String?
)

// Кастомный десериализатор
class UserDeserializer : JsonDeserializer<User> {
    override fun deserialize(
        json: JsonElement,
        typeOfT: Type,
        context: JsonDeserializationContext
    ): User {
        val jsonObject = json.asJsonObject
        return User(
            id = jsonObject.get("id").asInt,
            name = jsonObject.get("full_name").asString // маппинг разных имен полей
        )
    }
}

// Gson конфигурация
val gson = GsonBuilder()
    .setDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'")
    .registerTypeAdapter(User::class.java, UserDeserializer())
    .create()
```

### ⚠️ Error Handling

```kotlin
// Безопасный API вызов
suspend fun <T> safeApiCall(
    dispatcher: CoroutineDispatcher = Dispatchers.IO,
    apiCall: suspend () -> Response<T>
): Result<T> {
    return withContext(dispatcher) {
        try {
            val response = apiCall()
            if (response.isSuccessful) {
                response.body()?.let { data ->
                    Result.success(data)
                } ?: Result.failure(Exception("Пустой ответ"))
            } else {
                Result.failure(
                    HttpException(
                        "HTTP ${response.code()}: ${response.message()}"
                    )
                )
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// Repository с обработкой ошибок
class UserRepository(private val apiService: ApiService) {
    suspend fun getUsers(): Result<List<User>> {
        return safeApiCall {
            apiService.getUsers()
        }
    }
}
```

### 🔑 Ключевые темы:
- **HTTP методы**: GET, POST, PUT, DELETE
- **JSON**: парсинг и сериализация
- **Error Handling**: обработка сетевых ошибок
- **Caching**: кэширование сетевых запросов
- **Security**: HTTPS, Certificate Pinning
- **Interceptors**: логирование, аутентификация, кэширование

---

### Room Database
```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "name") val name: String,
    @ColumnInfo(name = "email") val email: String
)

@Dao
interface UserDao {
    @Query("SELECT * FROM users")
    suspend fun getAllUsers(): List<User>
    
    @Insert
    suspend fun insertUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
}

@Database(
    entities = [User::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

### SharedPreferences
```kotlin
class PreferencesManager(context: Context) {
    private val sharedPreferences = context.getSharedPreferences(
        "app_preferences", 
        Context.MODE_PRIVATE
    )
    
    fun saveString(key: String, value: String) {
        sharedPreferences.edit().putString(key, value).apply()
    }
    
    fun getString(key: String, defaultValue: String = ""): String {
        return sharedPreferences.getString(key, defaultValue) ?: defaultValue
    }
}
```

### DataStore (современная альтернатива SharedPreferences)
```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

class DataStoreManager(private val context: Context) {
    private val userNameKey = stringPreferencesKey("user_name")
    
    suspend fun saveUserName(name: String) {
        context.dataStore.edit { preferences ->
            preferences[userNameKey] = name
        }
    }
    
    fun getUserName(): Flow<String> {
        return context.dataStore.data.map { preferences ->
            preferences[userNameKey] ?: ""
        }
    }
}
```

### Важные концепции:
- **SQLite**: основы работы с базой данных
- **Room**: ORM для Android
- **DataStore**: замена SharedPreferences
- **File Storage**: internal и external storage
- **Content Providers**: обмен данными между приложениями

---

## 🌐 Сетевое взаимодействие

### Retrofit
```kotlin
// API Interface
interface ApiService {
    @GET("users")
    suspend fun getUsers(): Response<List<User>>
    
    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
    
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): Response<User>
}

// Retrofit setup
object ApiClient {
    private const val BASE_URL = "https://api.example.com/"
    
    val apiService: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}

// Repository
class UserRepository(private val apiService: ApiService) {
    suspend fun getUsers(): Result<List<User>> {
        return try {
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                Result.success(response.body() ?: emptyList())
            } else {
                Result.failure(Exception("API Error: ${response.code()}"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### OkHttp Interceptors
```kotlin
class AuthInterceptor(private val token: String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $token")
            .build()
        return chain.proceed(authenticatedRequest)
    }
}

// Добавление в Retrofit
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor("your_token"))
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY
    })
    .build()
```

### JSON парсинг с Gson
```kotlin
data class ApiResponse<T>(
    val success: Boolean,
    val data: T?,
    val message: String?
)

// Кастомный десериализатор
class UserDeserializer : JsonDeserializer<User> {
    override fun deserialize(
        json: JsonElement,
        typeOfT: Type,
        context: JsonDeserializationContext
    ): User {
        val jsonObject = json.asJsonObject
        return User(
            id = jsonObject.get("id").asInt,
            name = jsonObject.get("full_name").asString // маппинг разных имен полей
        )
    }
}
```

### Ключевые темы:
- **HTTP методы**: GET, POST, PUT, DELETE
- **JSON**: парсинг и сериализация
- **Error Handling**: обработка сетевых ошибок
- **Caching**: кэширование сетевых запросов
- **Security**: HTTPS, Certificate Pinning

---

## 🏗️ Архитектура и паттерны

### MVVM с ViewModel и LiveData
```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading
    
    fun loadUsers() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                val result = repository.getUsers()
                _users.value = result.getOrNull() ?: emptyList()
            } catch (e: Exception) {
                // Handle error
            } finally {
                _isLoading.value = false
            }
        }
    }
}

// Activity/Fragment
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: UserViewModel
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        viewModel = ViewModelProvider(this)[UserViewModel::class.java]
        
        viewModel.users.observe(this) { users ->
            // Обновление UI
        }
        
        viewModel.isLoading.observe(this) { isLoading ->
            // Показ/скрытие прогресс бара
        }
        
        viewModel.loadUsers()
    }
}
```

### Dependency Injection с Hilt
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    @Provides
    @Singleton
    fun provideApiService(): ApiService {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
    
    @Provides
    @Singleton
    fun provideUserRepository(apiService: ApiService): UserRepository {
        return UserRepository(apiService)
    }
}

@HiltAndroidApp
class MyApplication : Application()

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var userRepository: UserRepository
}
```

### Repository Pattern
```kotlin
interface UserRepository {
    suspend fun getUsers(): Flow<List<User>>
    suspend fun getUserById(id: Int): User?
    suspend fun createUser(user: User): Result<User>
}

class UserRepositoryImpl(
    private val apiService: ApiService,
    private val userDao: UserDao
) : UserRepository {
    
    override suspend fun getUsers(): Flow<List<User>> = flow {
        // Сначала показываем данные из кэша
        emit(userDao.getAllUsers())
        
        // Затем загружаем с сервера
        try {
            val response = apiService.getUsers()
            if (response.isSuccessful) {
                response.body()?.let { users ->
                    userDao.insertUsers(users)
                    emit(users)
                }
            }
        } catch (e: Exception) {
            // Логирование ошибки
        }
    }
}
```

### Важные паттерны:
- **MVVM**: Model-View-ViewModel
- **Repository Pattern**: абстракция источников данных
- **Observer Pattern**: LiveData, StateFlow
- **Singleton**: единственный экземпляр класса
- **Factory Pattern**: создание объектов

---

## 🎤 Вопросы для собеседования (ОБЯЗАТЕЛЬНО ПОВТОРИТЬ!)

### 🔥 ОСНОВНЫЕ ВОПРОСЫ (JUNIOR УРОВЕНЬ)

**Q: Объясните жизненный цикл Activity**
📝 **Ответ**: Activity проходит через несколько состояний: onCreate() - создание и инициализация UI, onStart() - Activity становится видимой, onResume() - Activity активна и может взаимодействовать с пользователем, onPause() - Activity теряет фокус, onStop() - Activity не видна, onDestroy() - Activity уничтожается.

**Q: Что такое Intent и какие виды Intent вы знаете?**
📝 **Ответ**: Intent - это объект сообщения для запроса действия от другого компонента приложения. Два вида: Явные (Explicit) - указывают конкретный класс для запуска, Неявные (Implicit) - описывают действие, которое нужно выполнить, и система находит подходящий компонент.

**Q: Объясните разницу между launch и async в Kotlin Coroutines**
📝 **Ответ**: launch - это "fire-and-forget" операция, которая запускает coroutine и не возвращает результат. async - запускает coroutine и возвращает Deferred, чтобы получить результат через await(). Используется для параллельного выполнения нескольких операций.

**Q: Как работает RecyclerView?**
📝 **Ответ**: RecyclerView использует паттерн ViewHolder для эффективного отображения больших списков. Он переиспользует View элементы вместо создания новых для каждого элемента. Adapter связывает данные с View элементами, LayoutManager определяет как элементы расположены.

### 🔥 ПРОДВИНУТЫЕ ВОПРОСЫ (MIDDLE УРОВЕНЬ)

**Q: MVVM архитектура и почему ViewModel переживает поворот экрана?**
📝 **Ответ**: MVVM состоит из Model (данные), View (UI), ViewModel (бизнес-логика и состояние UI). ViewModel переживает поворот, потому что она создается ViewModelProvider и связывается с ViewModelStoreOwner, который сохраняется при пересоздании Activity.

**Q: Разница между StateFlow и LiveData?**
📝 **Ответ**: StateFlow - это часть Kotlin Coroutines, работает с suspend функциями, можно использовать вне Android. LiveData - Android-специфичный, lifecycle-aware, автоматически приостанавливается при onStop. StateFlow требует ручного управления lifecycle.

**Q: Что такое Dependency Injection и как работает Hilt?**
📝 **Ответ**: DI - паттерн, при котором зависимости объекта предоставляются извне, а не создаются внутри. Hilt - обёртка над Dagger 2 для Android. Он автоматически создаёт компоненты для Android классов, упрощает настройку и уменьшает boilerplate код.

**Q: Как работает recomposition в Jetpack Compose?**
📝 **Ответ**: Recomposition - процесс повторного выполнения @Composable функций при изменении состояния. Compose автоматически отслеживает, какие части UI зависят от конкретных данных и перевычисляет только необходимые части.

### 🚨 ВОПРОСЫ НА ЛОГИКУ И ПРАКТИКУ

**Q: Как избежать ANR (Application Not Responding)?**
📝 **Ответ**: Не блокировать главный поток операциями дольше 5 секунд. Использовать Kotlin Coroutines, AsyncTask (deprecated), или WorkManager для фоновых операций. Выносить тяжелые вычисления в background thread.

**Q: Что такое memory leaks и как их избежать?**
📝 **Ответ**: Утечки памяти - когда объекты не могут быть собраны Garbage Collector. Основные причины: static ссылки на Context, неотмененные coroutines, listeners без отписки, inner classes с ссылкой на Activity. Избегать через WeakReference, правильное управление lifecycle, отмену корутин.

**Q: Объясните разницу между apply, let, run, with, also**
📝 **Ответ**: 
- **let** - для null-safety проверок, возвращает результат блока
- **apply** - для настройки объектов, возвращает сам объект
- **run** - для выполнения вычислений, возвращает результат блока
- **with** - для группировки операций над объектом
- **also** - для дополнительных действий (логирование), возвращает объект

---

## 📚 Полезные ресурсы для изучения

### 📖 Официальная документация
- [Android Developer Guides](https://developer.android.com/guide) - официальная документация Android
- [Kotlin Documentation](https://kotlinlang.org/docs/home.html) - официальная документация Kotlin
- [Jetpack Compose](https://developer.android.com/jetpack/compose) - современный UI toolkit
- [Android Architecture Components](https://developer.android.com/topic/libraries/architecture) - компоненты архитектуры

### 🎥 Видео курсы и каналы
- [Android Developers YouTube](https://www.youtube.com/user/androiddevelopers) - официальный канал
- [Coding in Flow](https://www.youtube.com/c/CodinginFlow) - отличные туториалы по Android
- [Stevdza-San](https://www.youtube.com/c/StevdzaSan) - Kotlin и Android разработка
- [Philipp Lackner](https://www.youtube.com/c/PhilippLackner) - современная Android разработка

### 📱 Практические ресурсы
- [Android Codelabs](https://developer.android.com/codelabs) - практические задания от Google
- [Kotlin Koans](https://kotlinlang.org/docs/koans.html) - интерактивные упражнения Kotlin
- [LeetCode](https://leetcode.com/) - алгоритмические задачи
- [HackerRank](https://www.hackerrank.com/) - программирование и алгоритмы

### 📚 Книги
- "Kotlin in Action" - Manning Publications
- "Android Programming: The Big Nerd Ranch Guide" - Big Nerd Ranch
- "Effective Java" - Joshua Bloch (фундаментальные принципы)
- "Clean Code" - Robert Martin (принципы чистого кода)

### 🛠️ Инструменты разработки
- [Android Studio](https://developer.android.com/studio) - официальная IDE
- [Firebase](https://firebase.google.com/) - backend-as-a-service от Google
- [Postman](https://www.postman.com/) - тестирование API
- [Git](https://git-scm.com/) - система контроля версий

---

## 🎯 ФИНАЛЬНЫЕ РЕКОМЕНДАЦИИ

### 🔥 ОБЯЗАТЕЛЬНО К ИЗУЧЕНИЮ (Топ приоритет)
1. **Kotlin Coroutines** - основа асинхронного программирования
2. **Jetpack Compose** - современный подход к UI
3. **MVVM + ViewModel** - архитектурный паттерн
4. **Hilt Dependency Injection** - управление зависимостями
5. **Room Database** - локальное хранение данных
6. **Retrofit + OkHttp** - сетевые запросы

### 📝 ПЛАН ДЕЙСТВИЙ НА ПОСЛЕДНИЙ ДЕНЬ
1. **Повторить основы** (2 часа): Activity lifecycle, Intent, Fragment
2. **Kotlin Coroutines** (2 часа): launch vs async, suspend functions, Flow
3. **Compose основы** (2 часа): State, Composable, remember
4. **Архитектура** (1 час): MVVM, Repository, DI
5. **Практика** (1 час): написать простое приложение с изученными концепциями

### 💡 СОВЕТЫ ДЛЯ ИНТЕРВЬЮ
1. **Будьте честными** - лучше сказать "не знаю", чем соврать
2. **Объясняйте код** - рассказывайте что и почему делаете
3. **Задавайте вопросы** - покажите интерес к проекту и команде
4. **Подготовьте примеры** - из своих проектов или учебных задач
5. **Практикуйтесь** - решайте задачи на кодинг и объясняйте решения

### 🚀 ДЛЯ ДАЛЬНЕЙШЕГО РАЗВИТИЯ
- **Kotlin Multiplatform** - разработка под несколько платформ
- **Android Testing** - unit, integration, UI тесты
- **CI/CD** - автоматизация сборки и развертывания
- **Performance Optimization** - профилирование и оптимизация
- **Security** - безопасность Android приложений

---

# 🎉 ЗАКЛЮЧЕНИЕ

Вы изучили весь необходимый материал для успешного прохождения технического интервью на позицию Android Junior Developer! Помните:

✅ **Практика** - ключ к успеху. Создавайте проекты, экспериментируйте
✅ **Понимание** важнее заучивания. Объясняйте концепции своими словами
✅ **Современный стек** - фокусируйтесь на актуальных технологиях
✅ **Уверенность** - вы готовы к интервью!

**УДАЧИ НА СОБЕСЕДОВАНИИ! 🍀 ВЫ ОБЯЗАТЕЛЬНО СПРАВИТЕСЬ! 💪**

*P.S. Сохраните этот гайд - он пригодится вам и в будущем для изучения более продвинутых тем!*

### 🚀 ПРОЕКТ НА 2 ДНЯ: TODO APP С COMPOSE + HILT + ROOM

Постарайтесь создать простое TODO приложение за оставшиеся 2 дня:

```kotlin
// 1. Модель данных
@Entity(tableName = "tasks")
data class Task(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val title: String,
    val description: String = "",
    val isCompleted: Boolean = false,
    val createdAt: Long = System.currentTimeMillis(),
    val priority: TaskPriority = TaskPriority.MEDIUM
)

enum class TaskPriority {
    LOW, MEDIUM, HIGH
}

// 2. DAO
@Dao
interface TaskDao {
    @Query("SELECT * FROM tasks ORDER BY priority DESC, createdAt DESC")
    fun getAllTasksFlow(): Flow<List<Task>>
    
    @Query("SELECT * FROM tasks WHERE isCompleted = 0")
    fun getActiveTasks(): Flow<List<Task>>
    
    @Insert
    suspend fun insertTask(task: Task)
    
    @Update
    suspend fun updateTask(task: Task)
    
    @Delete
    suspend fun deleteTask(task: Task)
}

// 3. Repository
interface TaskRepository {
    fun getAllTasks(): Flow<List<Task>>
    fun getActiveTasks(): Flow<List<Task>>
    suspend fun addTask(task: Task)
    suspend fun updateTask(task: Task)
    suspend fun deleteTask(task: Task)
    suspend fun toggleTaskCompletion(taskId: Int)
}

@Singleton
class TaskRepositoryImpl @Inject constructor(
    private val taskDao: TaskDao
) : TaskRepository {
    override fun getAllTasks(): Flow<List<Task>> = taskDao.getAllTasksFlow()
    override fun getActiveTasks(): Flow<List<Task>> = taskDao.getActiveTasks()
    override suspend fun addTask(task: Task) = taskDao.insertTask(task)
    override suspend fun updateTask(task: Task) = taskDao.updateTask(task)
    override suspend fun deleteTask(task: Task) = taskDao.deleteTask(task)
    
    override suspend fun toggleTaskCompletion(taskId: Int) {
        val task = taskDao.getTaskById(taskId)
        task?.let {
            taskDao.updateTask(it.copy(isCompleted = !it.isCompleted))
        }
    }
}

// 4. ViewModel
@HiltViewModel
class TaskViewModel @Inject constructor(
    private val repository: TaskRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(TaskUiState())
    val uiState: StateFlow<TaskUiState> = _uiState.asStateFlow()
    
    private val _showCompleted = MutableStateFlow(true)
    val showCompleted: StateFlow<Boolean> = _showCompleted.asStateFlow()
    
    init {
        loadTasks()
    }
    
    private fun loadTasks() {
        viewModelScope.launch {
            combine(
                repository.getAllTasks(),
                _showCompleted
            ) { tasks, showCompleted ->
                if (showCompleted) {
                    tasks
                } else {
                    tasks.filter { !it.isCompleted }
                }
            }.collect { filteredTasks ->
                _uiState.value = _uiState.value.copy(
                    tasks = filteredTasks,
                    isLoading = false
                )
            }
        }
    }
    
    fun addTask(title: String, description: String, priority: TaskPriority) {
        if (title.isBlank()) return
        
        viewModelScope.launch {
            val task = Task(
                title = title.trim(),
                description = description.trim(),
                priority = priority
            )
            repository.addTask(task)
        }
    }
    
    fun toggleTaskCompletion(task: Task) {
        viewModelScope.launch {
            repository.updateTask(task.copy(isCompleted = !task.isCompleted))
        }
    }
    
    fun deleteTask(task: Task) {
        viewModelScope.launch {
            repository.deleteTask(task)
        }
    }
    
    fun toggleShowCompleted() {
        _showCompleted.value = !_showCompleted.value
    }
    
    fun updateShowAddTaskDialog(show: Boolean) {
        _uiState.value = _uiState.value.copy(showAddTaskDialog = show)
    }
}

data class TaskUiState(
    val tasks: List<Task> = emptyList(),
    val isLoading: Boolean = true,
    val showAddTaskDialog: Boolean = false,
    val error: String? = null
)

// 5. Compose UI
@Composable
fun TaskApp(
    viewModel: TaskViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    val showCompleted by viewModel.showCompleted.collectAsState()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("TODO App") },
                actions = {
                    IconButton(
                        onClick = { viewModel.toggleShowCompleted() }
                    ) {
                        Icon(
                            imageVector = if (showCompleted) {
                                Icons.Default.Visibility
                            } else {
                                Icons.Default.VisibilityOff
                            },
                            contentDescription = "Toggle completed tasks"
                        )
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = { viewModel.updateShowAddTaskDialog(true) }
            ) {
                Icon(Icons.Default.Add, contentDescription = "Add Task")
            }
        }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            if (uiState.isLoading) {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            } else if (uiState.tasks.isEmpty()) {
                EmptyTasksScreen(showCompleted = showCompleted)
            } else {
                TaskList(
                    tasks = uiState.tasks,
                    onTaskClick = { task ->
                        viewModel.toggleTaskCompletion(task)
                    },
                    onDeleteTask = { task ->
                        viewModel.deleteTask(task)
                    }
                )
            }
        }
    }
    
    // Add Task Dialog
    if (uiState.showAddTaskDialog) {
        AddTaskDialog(
            onDismiss = { viewModel.updateShowAddTaskDialog(false) },
            onAddTask = { title, description, priority ->
                viewModel.addTask(title, description, priority)
                viewModel.updateShowAddTaskDialog(false)
            }
        )
    }
}

@Composable
fun TaskList(
    tasks: List<Task>,
    onTaskClick: (Task) -> Unit,
    onDeleteTask: (Task) -> Unit
) {
    LazyColumn(
        modifier = Modifier.fillMaxSize(),
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(
            items = tasks,
            key = { task -> task.id }
        ) { task ->
            TaskItem(
                task = task,
                onClick = { onTaskClick(task) },
                onDelete = { onDeleteTask(task) }
            )
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun TaskItem(
    task: Task,
    onClick: () -> Unit,
    onDelete: () -> Unit
) {
    var showDeleteDialog by remember { mutableStateOf(false) }
    
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .alpha(if (task.isCompleted) 0.6f else 1f),
        onClick = onClick
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Icon(
                imageVector = if (task.isCompleted) {
                    Icons.Default.CheckCircle
                } else {
                    Icons.Default.RadioButtonUnchecked
                },
                contentDescription = null,
                tint = if (task.isCompleted) {
                    MaterialTheme.colorScheme.primary
                } else {
                    MaterialTheme.colorScheme.onSurfaceVariant
                },
                modifier = Modifier.size(24.dp)
            )
            
            Spacer(modifier = Modifier.width(16.dp))
            
            Column(
                modifier = Modifier.weight(1f)
            ) {
                Text(
                    text = task.title,
                    style = MaterialTheme.typography.titleMedium,
                    textDecoration = if (task.isCompleted) {
                        TextDecoration.LineThrough
                    } else {
                        TextDecoration.None
                    }
                )
                
                if (task.description.isNotBlank()) {
                    Text(
                        text = task.description,
                        style = MaterialTheme.typography.bodyMedium,
                        color = MaterialTheme.colorScheme.onSurfaceVariant,
                        maxLines = 2,
                        overflow = TextOverflow.Ellipsis
                    )
                }
                
                Row(
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    PriorityChip(priority = task.priority)
                    Spacer(modifier = Modifier.width(8.dp))
                    Text(
                        text = formatDate(task.createdAt),
                        style = MaterialTheme.typography.labelSmall,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
            
            IconButton(
                onClick = { showDeleteDialog = true }
            ) {
                Icon(
                    Icons.Default.Delete,
                    contentDescription = "Delete Task",
                    tint = MaterialTheme.colorScheme.error
                )
            }
        }
    }
    
    if (showDeleteDialog) {
        AlertDialog(
            onDismissRequest = { showDeleteDialog = false },
            title = { Text("Удалить задачу?") },
            text = { Text("Это действие нельзя отменить.") },
            confirmButton = {
                TextButton(
                    onClick = {
                        onDelete()
                        showDeleteDialog = false
                    }
                ) {
                    Text("Удалить")
                }
            },
            dismissButton = {
                TextButton(
                    onClick = { showDeleteDialog = false }
                ) {
                    Text("Отмена")
                }
            }
        )
    }
}

@Composable
fun PriorityChip(priority: TaskPriority) {
    val (color, text) = when (priority) {
        TaskPriority.HIGH -> Color.Red to "Высокий"
        TaskPriority.MEDIUM -> Color.Orange to "Средний"
        TaskPriority.LOW -> Color.Green to "Низкий"
    }
    
    Surface(
        shape = RoundedCornerShape(12.dp),
        color = color.copy(alpha = 0.1f),
        border = BorderStroke(1.dp, color.copy(alpha = 0.5f))
    ) {
        Text(
            text = text,
            modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp),
            style = MaterialTheme.typography.labelSmall,
            color = color
        )
    }
}

@Composable
fun AddTaskDialog(
    onDismiss: () -> Unit,
    onAddTask: (String, String, TaskPriority) -> Unit
) {
    var title by remember { mutableStateOf("") }
    var description by remember { mutableStateOf("") }
    var selectedPriority by remember { mutableStateOf(TaskPriority.MEDIUM) }
    
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Новая задача") },
        text = {
            Column {
                OutlinedTextField(
                    value = title,
                    onValueChange = { title = it },
                    label = { Text("Название") },
                    singleLine = true,
                    modifier = Modifier.fillMaxWidth()
                )
                
                Spacer(modifier = Modifier.height(8.dp))
                
                OutlinedTextField(
                    value = description,
                    onValueChange = { description = it },
                    label = { Text("Описание (необязательно)") },
                    maxLines = 3,
                    modifier = Modifier.fillMaxWidth()
                )
                
                Spacer(modifier = Modifier.height(16.dp))
                
                Text(
                    text = "Приоритет",
                    style = MaterialTheme.typography.labelMedium
                )
                
                Row(
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    TaskPriority.values().forEach { priority ->
                        FilterChip(
                            onClick = { selectedPriority = priority },
                            label = {
                                Text(
                                    when (priority) {
                                        TaskPriority.HIGH -> "Высокий"
                                        TaskPriority.MEDIUM -> "Средний"
                                        TaskPriority.LOW -> "Низкий"
                                    }
                                )
                            },
                            selected = selectedPriority == priority
                        )
                    }
                }
            }
        },
        confirmButton = {
            TextButton(
                onClick = {
                    if (title.isNotBlank()) {
                        onAddTask(title, description, selectedPriority)
                    }
                },
                enabled = title.isNotBlank()
            ) {
                Text("Добавить")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) {
                Text("Отмена")
            }
        }
    )
}

@Composable
fun EmptyTasksScreen(showCompleted: Boolean) {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Icon(
            imageVector = if (showCompleted) {
                Icons.Default.CheckCircle
            } else {
                Icons.Default.Assignment
            },
            contentDescription = null,
            modifier = Modifier.size(64.dp),
            tint = MaterialTheme.colorScheme.onSurfaceVariant
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        Text(
            text = if (showCompleted) {
                "Все задачи выполнены!\n🎉 Отличная работа!"
            } else {
                "Нет активных задач\nДобавьте новую задачу"
            },
            style = MaterialTheme.typography.titleMedium,
            textAlign = TextAlign.Center,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
    }
}

fun formatDate(timestamp: Long): String {
    val formatter = SimpleDateFormat("dd MMM", Locale.getDefault())
    return formatter.format(Date(timestamp))
}

// 6. Database Module
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideAppDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context.applicationContext,
            AppDatabase::class.java,
            "task_database"
        )
            .fallbackToDestructiveMigration()
            .build()
    }
    
    @Provides
    fun provideTaskDao(database: AppDatabase): TaskDao {
        return database.taskDao()
    }
}

@Module
@InstallIn(ViewModelComponent::class)
abstract class RepositoryModule {
    
    @Binds
    abstract fun bindTaskRepository(
        taskRepositoryImpl: TaskRepositoryImpl
    ): TaskRepository
}

@Database(
    entities = [Task::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun taskDao(): TaskDao
}eleteTask(task)
}

// 4. ViewModel
@HiltViewModel
class TaskViewModel @Inject constructor(
    private val repository: TaskRepository
) : ViewModel() {
    
    val allTasks = repository.getAllTasks()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    private val _showCompleted = MutableStateFlow(true)
    val showCompleted = _showCompleted.asStateFlow()
    
    val filteredTasks = combine(allTasks, showCompleted) { tasks, showCompleted ->
        if (showCompleted) tasks else tasks.filter { !it.isCompleted }
    }.stateIn(
        scope = viewModelScope,
        started = SharingStarted.WhileSubscribed(5000),
        initialValue = emptyList()
    )
    
    fun addTask(title: String, description: String, priority: TaskPriority) {
        viewModelScope.launch {
            val task = Task(
                title = title,
                description = description,
                priority = priority
            )
            repository.addTask(task)
        }
    }
    
    fun toggleTaskCompletion(task: Task) {
        viewModelScope.launch {
            repository.updateTask(task.copy(isCompleted = !task.isCompleted))
        }
    }
    
    fun deleteTask(task: Task) {
        viewModelScope.launch {
            repository.deleteTask(task)
        }
    }
    
    fun toggleShowCompleted() {
        _showCompleted.value = !_showCompleted.value
    }
}

// 5. Compose UI
@Composable
fun TaskScreen(
    viewModel: TaskViewModel = hiltViewModel()
) {
    val tasks by viewModel.filteredTasks.collectAsState()
    val showCompleted by viewModel.showCompleted.collectAsState()
    var showAddDialog by remember { mutableStateOf(false) }
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("TODO App") },
                actions = {
                    IconButton(onClick = { viewModel.toggleShowCompleted() }) {
                        Icon(
                            imageVector = if (showCompleted) Icons.Default.VisibilityOff else Icons.Default.Visibility,
                            contentDescription = "Показать выполненные"
                        )
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = { showAddDialog = true }
            ) {
                Icon(Icons.Default.Add, contentDescription = "Добавить")
            }
        }
    ) { paddingValues ->
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues),
            contentPadding = PaddingValues(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            items(tasks, key = { it.id }) { task ->
                TaskCard(
                    task = task,
                    onToggleComplete = { viewModel.toggleTaskCompletion(task) },
                    onDelete = { viewModel.deleteTask(task) }
                )
            }
        }
    }
    
    if (showAddDialog) {
        AddTaskDialog(
            onDismiss = { showAddDialog = false },
            onAddTask = { title, description, priority ->
                viewModel.addTask(title, description, priority)
                showAddDialog = false
            }
        )
    }
}

@Composable
fun TaskCard(
    task: Task,
    onToggleComplete: () -> Unit,
    onDelete: () -> Unit
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Checkbox(
                checked = task.isCompleted,
                onCheckedChange = { onToggleComplete() }
            )
            
            Column(
                modifier = Modifier
                    .weight(1f)
                    .padding(start = 8.dp)
            ) {
                Text(
                    text = task.title,
                    style = MaterialTheme.typography.titleMedium,
                    textDecoration = if (task.isCompleted) TextDecoration.LineThrough else null
                )
                if (task.description.isNotBlank()) {
                    Text(
                        text = task.description,
                        style = MaterialTheme.typography.bodyMedium,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
                
                Text(
                    text = task.priority.name,
                    style = MaterialTheme.typography.labelSmall,
                    color = when (task.priority) {
                        TaskPriority.HIGH -> Color.Red
                        TaskPriority.MEDIUM -> Color.Orange
                        TaskPriority.LOW -> Color.Green
                    }
                )
            }
            
            IconButton(onClick = onDelete) {
                Icon(Icons.Default.Delete, contentDescription = "Удалить")
            }
        }
    }
}
```

### 🎯 Критически важные задачи для Live Coding:

**Задача 1: Счетчик с сохранением состояния**
```kotlin
@HiltViewModel
class CounterViewModel @Inject constructor(
    private val dataStore: DataStore<Preferences>
) : ViewModel() {
    
    private val counterKey = intPreferencesKey("counter")
    
    val counter = dataStore.data
        .map { preferences -> preferences[counterKey] ?: 0 }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = 0
        )
    
    fun increment() {
        viewModelScope.launch {
            dataStore.edit { preferences ->
                val currentValue = preferences[counterKey] ?: 0
                preferences[counterKey] = currentValue + 1
            }
        }
    }
    
    fun decrement() {
        viewModelScope.launch {
            dataStore.edit { preferences ->
                val currentValue = preferences[counterKey] ?: 0
                preferences[counterKey] = (currentValue - 1).coerceAtLeast(0)
            }
        }
    }
}
```

**Задача 2: Поиск с debounce**
```kotlin
@Composable
fun SearchScreen() {
    var searchQuery by remember { mutableStateOf("") }
    var searchResults by remember { mutableStateOf<List<String>>(emptyList()) }
    
    LaunchedEffect(searchQuery) {
        delay(300) // Debounce
        if (searchQuery.isNotBlank()) {
            searchResults = performSearch(searchQuery)
        } else {
            searchResults = emptyList()
        }
    }
    
    Column {
        OutlinedTextField(
            value = searchQuery,
            onValueChange = { searchQuery = it },
            label = { Text("Поиск") },
            modifier = Modifier.fillMaxWidth()
        )
        
        LazyColumn {
            items(searchResults) { result ->
                Text(
                    text = result,
                    modifier = Modifier.padding(16.dp)
                )
            }
        }
    }
}
```

**Задача 3: Асинхронная загрузка с обработкой ошибок**
```kotlin
@Composable
fun DataScreen(
    viewModel: DataViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    
    when (uiState) {
        is UiState.Loading -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
        is UiState.Success -> {
            LazyColumn {
                items(uiState.data) { item ->
                    Text(
                        text = item.toString(),
                        modifier = Modifier.padding(16.dp)
                    )
                }
            }
        }
        is UiState.Error -> {
            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(16.dp),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.Center
            ) {
                Text(
                    text = "Ошибка: ${uiState.exception.message}",
                    color = MaterialTheme.colorScheme.error
                )
                Button(
                    onClick = { viewModel.retry() },
                    modifier = Modifier.padding(top = 16.dp)
                ) {
                    Text("Повторить")
                }
            }
        }
        is UiState.Empty -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text("Нет данных")
            }
        }
    }
    
    LaunchedEffect(Unit) {
        viewModel.loadData()
    }
}
```

---

## ❓ Вопросы для собеседования (Современный стек)

### 🚀 КРИТИЧЕСКИ ВАЖНЫЕ вопросы (учить ОБЯЗАТЕЛЬНО!):

**1. Kotlin Coroutines:**
- «Объясните разницу между launch и async»
- «Что такое suspend функция?»
- «Как обрабатывать ошибки в coroutines?»

**2. Jetpack Compose:**
- «Что такое recomposition?»
- «Когда использовать remember вс mutableStateOf?»
- «Объясните State hoisting»

**3. MVVM архитектура:**
- «Почему ViewModel переживает поворот экрана?»
- «Разница между StateFlow и LiveData?»

**4. Hilt DI:**
- «Как работает Dependency Injection?»
- «Разница между @Inject и @Provides?»

**5. Room Database:**
- «Как Flow работает с Room?»
- «Объясните @Entity, @Dao, @Database»

### 💬 Ответы на частые вопросы:

**Q: Объясните разницу между launch и async**
```kotlin
// launch - запускает coroutine и не возвращает результат
viewModelScope.launch {
    val data = fetchData() // Последовательное выполнение
    updateUI(data)
}

// async - возвращает Deferred для параллельного выполнения
val userDeferred = async { fetchUser() }
val postsDeferred = async { fetchPosts() }
val user = userDeferred.await() // Параллельно!
val posts = postsDeferred.await()
```

**Q: Что такое recomposition в Compose?**
“Recomposition - это процесс повторного выполнения @Composable функций при изменении состояния. Compose автоматически отслеживает, какие части UI нужно обновить.”

**Q: Почему ViewModel переживает поворот экрана?**
“ViewModel создается ViewModelProvider и связывается с жизненным циклом ViewModelStoreOwner (обычно Activity). При повороте Activity пересоздается, но ViewModel остается в памяти.”

**Q: Разница между StateFlow и LiveData?**
- **StateFlow**: часть Kotlin Coroutines, работает с suspend функциями, можно использовать вне Android
- **LiveData**: Android-специфичный, lifecycle-aware, автоматически приостанавливается

### 💻 Код-вопросы (ПРАКТИКА!):

**Вопрос 1: Напишите Composable для списка с pull-to-refresh**
```kotlin
@Composable
fun RefreshableList(
    items: List<String>,
    isRefreshing: Boolean,
    onRefresh: () -> Unit
) {
    val pullRefreshState = rememberPullRefreshState(
        refreshing = isRefreshing,
        onRefresh = onRefresh
    )
    
    Box(
        modifier = Modifier.pullRefresh(pullRefreshState)
    ) {
        LazyColumn {
            items(items) { item ->
                Text(
                    text = item,
                    modifier = Modifier.padding(16.dp)
                )
            }
        }
        
        PullRefreshIndicator(
            refreshing = isRefreshing,
            state = pullRefreshState,
            modifier = Modifier.align(Alignment.TopCenter)
        )
    }
}
```

**Вопрос 2: Как сделать безопасный API вызов?**
```kotlin
suspend fun <T> safeApiCall(
    apiCall: suspend () -> Response<T>
): Result<T> {
    return try {
        val response = apiCall()
        if (response.isSuccessful) {
            response.body()?.let { data ->
                Result.success(data)
            } ?: Result.failure(Exception("Пустой ответ"))
        } else {
            Result.failure(HttpException(response))
        }
    } catch (e: Exception) {
        Result.failure(e)
    }
}

// Использование:
suspend fun getUsers(): Result<List<User>> {
    return safeApiCall { apiService.getUsers() }
}
```

**Вопрос 3: Как обработать состояние загрузки в Compose?**
```kotlin
@Composable
fun LoadingStateExample(
    uiState: UiState<List<String>>
) {
    when (uiState) {
        is UiState.Loading -> {
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
        is UiState.Success -> {
            LazyColumn {
                items(uiState.data) { item ->
                    Text(item)
                }
            }
        }
        is UiState.Error -> {
            ErrorMessage(
                message = uiState.exception.message ?: "Ошибка",
                onRetry = { /* Повтор */ }
            )
        }
        is UiState.Empty -> {
            EmptyState()
        }
    }
}
```

### 💡 Частые ошибки Junior разработчиков:

1. **Не использовать viewModelScope** для coroutines в ViewModel
2. **Мутировать state напрямую** в Compose
3. **Не обрабатывать ошибки** в coroutines
4. **Использовать GlobalScope** вместо правильных scope
5. **Забывать про remember** в Compose

---

## 📚 Полезные ресурсы

### Официальная документация:
- [Android Developers](https://developer.android.com/) - основной ресурс
- [Kotlin Documentation](https://kotlinlang.org/docs/home.html) - документация Kotlin
- [Material Design](https://material.io/) - принципы дизайна

### Обучающие платформы:
- [Udacity Android Courses](https://www.udacity.com/course/android-kotlin-developer-nanodegree--nd940)
- [Coursera Android Development](https://www.coursera.org/specializations/android-app-development)
- [Pluralsight Android Path](https://www.pluralsight.com/paths/android)

### Практика и вызовы:
- [LeetCode](https://leetcode.com/) - алгоритмические задачи
- [HackerRank](https://www.hackerrank.com/) - задачи по программированию
- [Codewars](https://www.codewars.com/) - практика Kotlin

### YouTube каналы:
- [Android Developers](https://www.youtube.com/user/androiddevelopers)
- [Coding in Flow](https://www.youtube.com/channel/UC_Fh8kvtkVPkeihBs42jGcA)
- [Philipp Lackner](https://www.youtube.com/c/PhilippLackner)

### Блоги и статьи:
- [Medium Android Publications](https://medium.com/tag/android-development)
- [Vogella Android Tutorials](https://www.vogella.com/tutorials/android.html)
- [Baeldung Android](https://www.baeldung.com/category/android/)

### Инструменты разработки:
- [Android Studio](https://developer.android.com/studio) - IDE
- [Genymotion](https://www.genymotion.com/) - эмулятор
- [Postman](https://www.postman.com/) - тестирование API

---

## 🚀 Финальные советы (ЭКСПРЕСС ПОДГОТОВКА!)

### 🔥 КРИТИЧЕСКИ важно за 2 дня:

**Первый день:**
- Kotlin Coroutines (launch, async, suspend, Flow)
- Jetpack Compose основы (@Composable, remember, State)
- MVVM с ViewModel (StateFlow vs LiveData)

**Второй день:**
- Hilt DI (@Inject, @Module, @Provides)
- Room Database (@Entity, @Dao, Flow)
- Retrofit (suspend functions)
- Практика кодинга

### 📝 ШПАРГАЛКИ перед интервью:

**Kotlin:**
- `suspend fun` - может быть приостановлена
- `launch` vs `async` - fire-and-forget vs возвращает результат
- `Flow` vs `StateFlow` - потоки vs state

**Compose:**
- `@Composable` - функция UI
- `remember` - сохраняет состояние
- `LaunchedEffect` - для side effects

### 🎭 Как вести себя на интервью:

- **Думайте вслух** - объясняйте ход мыслей
- **Задавайте вопросы** - уточняйте требования
- **Признавайте незнание** - лучше "не знаю, но могу подумать"

### 📦 Минимальное портфолио:

1. **TODO приложение** с Compose + Room + Hilt
2. **Приложение с API** (Retrofit + Coroutines)

### 🚑 Полезные ресурсы:

- [Android Codelabs](https://codelabs.developers.google.com/?cat=android)
- [Compose Tutorial](https://developer.android.com/jetpack/compose/tutorial)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Philipp Lackner YouTube](https://www.youtube.com/c/PhilippLackner)
- [LeetCode Easy](https://leetcode.com/problemset/all/?difficulty=Easy)

---

## 🎆 ЗАКЛЮЧЕНИЕ

**2 дня - это мало, но достаточно!** Сосредоточьтесь на:

✅ Kotlin Coroutines
✅ Jetpack Compose 
✅ MVVM + ViewModel
✅ Hilt DI
✅ Практика!

**Удачи на собеседовании! 🚀**