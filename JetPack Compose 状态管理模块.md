
# JetPack Compose 状态管理模块

## 1. 基础状态

### mutableStateOf

```kotlin
// mutableStateOf：Compose 最基本的状态容器
// 当值变化时，自动触发重组（Recomposition）

@Composable
fun Counter() {
    // 方式一：直接创建
    var count by mutableStateOf(0)

    // 方式二：配合 remember 使用（推荐）
    var count2 by remember { mutableStateOf(0) }

    // 方式三：使用 delegated 属性
    val countState = remember { mutableStateOf(0) }
    val count3 = countState.value

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

// mutableStateOf 可观察任何类型的变化
@Composable
fun UserForm() {
    var name by remember { mutableStateOf("") }
    var age by remember { mutableStateOf(0) }
    var isLogin by remember { mutableStateOf(false) }

    Column {
        TextField(
            value = name,
            onValueChange = { name = it }
        )
        Text("Age: $age")
        Text("Status: ${if (isLogin) "Logged in" else "Guest"}")
    }
}

// 自定义类作为状态
data class User(val name: String, val age: Int)

@Composable
fun UserProfile() {
    var user by remember { mutableStateOf(User("windCloud", 25)) }

    Column {
        Text("Name: ${user.name}")
        Text("Age: ${user.age}")
        Button(onClick = {
            user = user.copy(age = user.age + 1)  // 创建新对象触发重组
        }) {
            Text("Add Age")
        }
    }
}
```

### remember

```kotlin
// remember：记住状态，重组时不丢失
// ⚠️ 不加 remember，每次重组都会重置状态

@Composable
fun WithoutRemember() {
    // ❌ 错误：每次重组 count 都重置为 0
    var count by mutableStateOf(0)
    Button(onClick = { count++ }) {
        Text("Count: $count")  // 永远显示 0
    }
}

@Composable
fun WithRemember() {
    // ✅ 正确：remember 保持状态
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")  // 正常累加
    }
}

// remember 可记住多个值
@Composable
fun RememberMultiple() {
    val state = remember {
        // 返回多个值
        mutableStateOf(Pair(0, 0))
    }
    var (x, y) by state

    // 或者分别记住
    var count1 by remember { mutableStateOf(0) }
    var count2 by remember { mutableStateOf(0) }
}

// remember 带 key：key 变化时重新创建
@Composable
fun RememberWithKey(userId: String) {
    // userId 变化时，重新创建状态
    var userData by remember(userId) {
        mutableStateOf(fetchUser(userId))
    }
}

// remember 计算初始值
@Composable
fun RememberCalculation(items: List<String>) {
    // 只计算一次，后续重组不重复计算
    val sortedItems by remember(items) {
        mutableStateOf(items.sorted())
    }
}
```

### rememberSaveable

```kotlin
// rememberSaveable：配置变更（如旋转屏幕）后保持状态
// 数据保存到 Bundle，进程杀死后无法恢复

@Composable
fun SaveableExample() {
    // ✅ 屏幕旋转后 count 保持不变
    var count by rememberSaveable { mutableStateOf(0) }

    // ✅ 保存自定义类型（需使用状态保存策略）
    var text by rememberSaveable(stateSaver = TextFieldValue.Saver) {
        mutableStateOf(TextFieldValue(""))
    }

    // ✅ 保存复杂对象（需实现 Parcelable 或使用 Saver）
    var user by rememberSaveable(saver = UserSaver) {
        mutableStateOf(User("windCloud", 25))
    }
}

// 自定义 Saver
data class User(val name: String, val age: Int)

val UserSaver = Saver<User, Pair<String, Int>>(
    save = { user -> Pair(user.name, user.age) },
    restore = { pair -> User(pair.first, pair.second) }
)

// 使用 Saver
@Composable
fun SaveableUser() {
    var user by rememberSaveable(saver = UserSaver) {
        mutableStateOf(User("windCloud", 25))
    }
}

// 列表状态保存
@Composable
fun SaveableList() {
    var items by rememberSaveable {
        mutableStateOf(mutableListOf<String>())
    }
    // ⚠️ mutableList 修改不会触发重组，需用 snapshotFlow 或 toList()
}

// 正确保存列表
@Composable
fun SaveableListCorrect() {
    var items by rememberSaveable {
        mutableStateOf(listOf<String>())
    }

    Button(onClick = {
        items = items + "New Item"  // 创建新列表触发重组
    }) {
        Text("Add Item")
    }

    items.forEach { item ->
        Text(item)
    }
}

// remember vs rememberSaveable
@Composable
fun Comparison() {
    // remember：重组保持，配置变更丢失
    var count1 by remember { mutableStateOf(0) }

    // rememberSaveable：重组保持，配置变更也保持
    var count2 by rememberSaveable { mutableStateOf(0) }

    // 使用场景：
    // - 临时 UI 状态（动画进度等）→ remember
    // - 用户输入、计数器等 → rememberSaveable
}
```

## 2. 状态提升（State Hoisting）

### 状态提升模式

```kotlin
// 状态提升：将状态移到父组件，子组件通过参数接收状态和事件
// 原则：子组件不持有可变状态

// ❌ 错误：子组件持有状态
@Composable
fun BadTextField() {
    var text by remember { mutableStateOf("") }
    TextField(
        value = text,
        onValueChange = { text = it }
    )
}

// ✅ 正确：状态提升到父组件
@Composable
fun GoodTextField(text: String, onTextChange: (String) -> Unit) {
    TextField(
        value = text,
        onValueChange = onTextChange
    )
}

@Composable
fun ParentComponent() {
    var text by remember { mutableStateOf("") }
    GoodTextField(
        text = text,
        onTextChange = { text = it }
    )
}

// 完整示例：带状态的按钮
@Composable
fun LikeButton(isLiked: Boolean, onLikeToggle: () -> Unit) {
    IconButton(onClick = onLikeToggle) {
        Icon(
            imageVector = if (isLiked) Icons.Filled.Favorite else Icons.Outlined.Favorite,
            contentDescription = "Like",
            tint = if (isLiked) Color.Red else Color.Gray
        )
    }
}

@Composable
fun FeedItem() {
    var isLiked by remember { mutableStateOf(false) }
    LikeButton(
        isLiked = isLiked,
        onLikeToggle = { isLiked = !isLiked }
    )
}
```

### 单向数据流（UDF）

```kotlin
// 单向数据流：数据从上到下流动，事件从下到上流动
// State ↓ UI → Event ↑

// 状态类
data class CounterState(
    val count: Int = 0,
    val isLoading: Boolean = false,
    val error: String? = null
)

// 事件类
sealed class CounterEvent {
    object Increment : CounterEvent()
    object Decrement : CounterEvent()
    data class SetCount(val newCount: Int) : CounterEvent()
}

// 无状态组件
@Composable
fun CounterScreen(
    state: CounterState,
    onEvent: (CounterEvent) -> Unit
) {
    Column {
        Text("Count: ${state.count}")

        if (state.isLoading) {
            CircularProgressIndicator()
        }

        state.error?.let { error ->
            Text("Error: $error", color = Color.Red)
        }

        Row {
            Button(onClick = { onEvent(CounterEvent.Decrement) }) {
                Text("-")
            }
            Button(onClick = { onEvent(CounterEvent.Increment) }) {
                Text("+")
            }
        }
    }
}

// 有状态组件（持有状态）
@Composable
fun CounterContainer() {
    var state by remember { mutableStateOf(CounterState()) }

    fun handleEvent(event: CounterEvent) {
        when (event) {
            is CounterEvent.Increment -> {
                state = state.copy(count = state.count + 1)
            }
            is CounterEvent.Decrement -> {
                state = state.copy(count = state.count - 1)
            }
            is CounterEvent.SetCount -> {
                state = state.copy(count = event.newCount)
            }
        }
    }

    CounterScreen(
        state = state,
        onEvent = ::handleEvent
    )
}
```

### 状态提升实战

```kotlin
// 场景：表单输入
data class FormState(
    val name: String = "",
    val email: String = "",
    val isValid: Boolean = false
)

@Composable
fun FormScreen(
    state: FormState,
    onNameChange: (String) -> Unit,
    onEmailChange: (String) -> Unit,
    onSubmit: () -> Unit
) {
    Column {
        TextField(
            value = state.name,
            onValueChange = onNameChange,
            label = { Text("Name") }
        )

        TextField(
            value = state.email,
            onValueChange = onEmailChange,
            label = { Text("Email") }
        )

        Button(
            onClick = onSubmit,
            enabled = state.isValid
        ) {
            Text("Submit")
        }
    }
}

@Composable
fun FormContainer() {
    var state by remember { mutableStateOf(FormState()) }

    // 验证逻辑
    fun validate() {
        state = state.copy(
            isValid = state.name.isNotBlank() && state.email.contains("@")
        )
    }

    FormScreen(
        state = state,
        onNameChange = { state = state.copy(name = it); validate() },
        onEmailChange = { state = state.copy(email = it); validate() },
        onSubmit = { /* 提交逻辑 */ }
    )
}

// 场景：列表项选择
@Composable
fun SelectableList(
    items: List<String>,
    selectedItem: String?,
    onItemSelected: (String) -> Unit
) {
    Column {
        items.forEach { item ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .background(
                        if (item == selectedItem) Color.Blue.copy(0.1f)
                        else Color.Transparent
                    )
                    .clickable { onItemSelected(item) }
                    .padding(16.dp)
            ) {
                Text(
                    text = item,
                    color = if (item == selectedItem) Color.Blue else Color.Black
                )
            }
        }
    }
}

@Composable
fun SelectableListContainer() {
    val items = listOf("Apple", "Banana", "Cherry", "Date")
    var selectedItem by remember { mutableStateOf<String?>(null) }

    SelectableList(
        items = items,
        selectedItem = selectedItem,
        onItemSelected = { selectedItem = it }
    )
}
```

### 状态下放（State Delegation）

```kotlin
// 状态下放：父组件持有状态，但将部分状态管理委托给子组件

// 父组件持有完整状态
@Composable
fun ParentWithDelegation() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("Parent Count: $count")

        // 下放状态给子组件管理增减
        CounterChild(
            count = count,
            onCountChange = { count = it }
        )
    }
}

// 子组件负责内部逻辑
@Composable
fun CounterChild(
    count: Int,
    onCountChange: (Int) -> Unit
) {
    Column {
        Button(onClick = { onCountChange(count + 1) }) {
            Text("Increment")
        }
        Button(onClick = { onCountChange(count - 1) }) {
            Text("Decrement")
        }
        // 重置功能
        Button(onClick = { onCountChange(0) }) {
            Text("Reset")
        }
    }
}

// 使用 derivedStateOf 下放计算状态
@Composable
fun DerivedStateExample() {
    var input by remember { mutableStateOf("") }

    // 计算状态下放
    val isValid by remember { derivedStateOf { input.length >= 6 } }
    val errorText by remember {
        derivedStateOf {
            if (input.isEmpty()) ""
            else if (input.length < 6) "至少 6 个字符"
            else ""
        }
    }

    Column {
        TextField(
            value = input,
            onValueChange = { input = it },
            isError = !isValid && input.isNotEmpty()
        )
        if (errorText.isNotEmpty()) {
            Text(errorText, color = Color.Red)
        }
    }
}
```

## 3. 高级状态

### StateFlow 配合 Compose

```kotlin
// StateFlow：可观察的状态流，适合 ViewModel
// collectAsState：将 StateFlow 转为 Compose State

class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count.asStateFlow()

    fun increment() {
        _count.value++
    }

    fun decrement() {
        _count.value--
    }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    // collectAsState：自动收集 StateFlow 并转为 Compose State
    val count by viewModel.count.collectAsState()

    Column {
        Text("Count: $count")
        Row {
            Button(onClick = { viewModel.decrement() }) {
                Text("-")
            }
            Button(onClick = { viewModel.increment() }) {
                Text("+")
            }
        }
    }
}

// 带生命周期的收集
@Composable
fun LifecycleAwareScreen(viewModel: MyViewModel) {
    val lifecycleOwner = LocalLifecycleOwner.current

    val data by viewModel.dataFlow
        .collectAsState(
            initial = LoadingState,
            lifecycleOwner.lifecycle,
            Lifecycle.State.STARTED
        )

    // 只在 STARTED 状态收集
}

// 多个 StateFlow 组合
@Composable
fun CombinedScreen(viewModel: MyViewModel) {
    val uiState by viewModel.uiState.collectAsState()
    val userData by viewModel.userData.collectAsState()
    val settings by viewModel.settings.collectAsState()

    when (val state = uiState) {
        is UiState.Loading -> LoadingView()
        is UiState.Success -> SuccessView(userData, settings)
        is UiState.Error -> ErrorView(state.message)
    }
}
```

### SharedFlow 配合 Compose

```kotlin
// SharedFlow：事件流，无状态，适合一次性事件
// 如：导航、Toast、Snackbar

class EventViewModel : ViewModel() {
    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    fun onButtonClick() {
        viewModelScope.launch {
            _events.emit(UiEvent.ShowToast("操作成功"))
        }
    }

    fun onNavigate() {
        viewModelScope.launch {
            _events.emit(UiEvent.NavigateToDetail)
        }
    }
}

sealed class UiEvent {
    data class ShowToast(val message: String) : UiEvent()
    object NavigateToDetail : UiEvent()
    data class ShowError(val error: String) : UiEvent()
}

// 收集事件（使用 LaunchedEffect）
@Composable
fun EventScreen(viewModel: EventViewModel = viewModel()) {
    val snackbarHostState = remember { SnackbarHostState() }

    // 监听事件
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowToast -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is UiEvent.NavigateToDetail -> {
                    // 导航逻辑
                }
                is UiEvent.ShowError -> {
                    snackbarHostState.showSnackbar("错误：${event.error}")
                }
            }
        }
    }

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        // 内容
    }
}

// 使用 repeatOnLifecycle 收集（Fragment/Activity）
@Composable
fun LifecycleEventScreen(viewModel: EventViewModel) {
    val lifecycleOwner = LocalLifecycleOwner.current

    LaunchedEffect(Unit) {
        lifecycleOwner.lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.events.collect { event ->
                // 处理事件
            }
        }
    }
}
```

### collectAsState 详解

```kotlin
// collectAsState 基础用法
@Composable
fun BasicCollect(viewModel: MyViewModel) {
    val state by viewModel.stateFlow.collectAsState()
    Text("Value: $state")
}

// collectAsState 带初始值
@Composable
fun WithInitialValue(viewModel: MyViewModel) {
    val state by viewModel.stateFlow.collectAsState(initial = LoadingState)
    Text("Value: $state")
}

// collectAsState 带生命周期
@Composable
fun WithLifecycle(viewModel: MyViewModel) {
    val lifecycleOwner = LocalLifecycleOwner.current
    val state by viewModel.stateFlow.collectAsState(
        initial = LoadingState,
        lifecycle = lifecycleOwner.lifecycle,
        minActiveState = Lifecycle.State.STARTED
    )
}

// collectAsState 在 LazyColumn 中
@Composable
fun ListScreen(viewModel: ListViewModel) {
    val items by viewModel.items.collectAsState()

    LazyColumn {
        items(items) { item ->
            Text(item.name)
        }
    }
}

// collectAsState 配合 derivedStateOf
@Composable
fun DerivedScreen(viewModel: MyViewModel) {
    val rawState by viewModel.stateFlow.collectAsState()

    // 计算派生状态
    val displayState by remember(rawState) {
        derivedStateOf {
            rawState.copy(
                displayName = rawState.name.uppercase()
            )
        }
    }

    Text("Display: ${displayState.displayName}")
}

// 多个 Flow 组合
@Composable
fun CombinedFlowScreen(viewModel: MyViewModel) {
    val state1 by viewModel.flow1.collectAsState()
    val state2 by viewModel.flow2.collectAsState()

    // 组合两个状态
    val combined = remember(state1, state2) {
        combineStates(state1, state2)
    }

    Text("Combined: $combined")
}
```

### snapshotFlow：Compose State 转 Flow

```kotlin
// snapshotFlow：将 Compose State 转为 Flow
// 用于在 ViewModel 中观察 Compose 状态

@Composable
fun SearchScreen(viewModel: SearchViewModel) {
    var query by rememberSaveable { mutableStateOf("") }

    // 将 Compose State 转为 Flow 发送给 ViewModel
    LaunchedEffect(query) {
        snapshotFlow { query }
            .debounce(300)
            .collect { q ->
                viewModel.search(q)
            }
    }

    TextField(
        value = query,
        onValueChange = { query = it }
    )

    val results by viewModel.results.collectAsState()
    results.forEach { result ->
        Text(result)
    }
}

// ViewModel 中使用
class SearchViewModel : ViewModel() {
    private val _results = MutableStateFlow<List<String>>(emptyList())
    val results: StateFlow<List<String>> = _results.asStateFlow()

    fun search(query: String) {
        viewModelScope.launch {
            _results.value = repository.search(query)
        }
    }
}

// snapshotFlow 配合 StateFlow
@Composable
fun FormScreen(viewModel: FormViewModel) {
    var name by rememberSaveable { mutableStateOf("") }
    var email by rememberSaveable { mutableStateOf("") }

    // 监听表单变化
    LaunchedEffect(Unit) {
        snapshotFlow { Pair(name, email) }
            .collect { (n, e) ->
                viewModel.updateForm(n, e)
            }
    }

    // 监听验证状态
    val isValid by viewModel.isValid.collectAsState()

    Button(
        onClick = { viewModel.submit() },
        enabled = isValid
    ) {
        Text("Submit")
    }
}
```

## 4. 全局状态

### ViewModel + Compose

```kotlin
// ViewModel：生命周期感知的状态容器
// hiltViewModel() / viewModel() 获取 ViewModel

@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {

    // UI 状态
    private val _uiState = MutableStateFlow<UiState<User>>(UiState.Loading)
    val uiState: StateFlow<UiState<User>> = _uiState.asStateFlow()

    // 事件流
    private val _events = MutableSharedFlow<UiEvent>()
    val events: SharedFlow<UiEvent> = _events.asSharedFlow()

    init {
        loadUser()
    }

    fun loadUser() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            repository.getUser()
                .onSuccess { user ->
                    _uiState.value = UiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UiState.Error(error.message ?: "Unknown error")
                    _events.emit(UiEvent.ShowError(error.message ?: "Unknown error"))
                }
        }
    }

    fun updateUser(name: String) {
        viewModelScope.launch {
            repository.updateName(name)
                .onSuccess {
                    _events.emit(UiEvent.ShowToast("更新成功"))
                }
                .onFailure {
                    _events.emit(UiEvent.ShowError("更新失败"))
                }
        }
    }
}

// sealed class 状态定义
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}

// Compose 中使用
@Composable
fun UserScreen(viewModel: UserViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    val snackbarHostState = remember { SnackbarHostState() }

    // 监听事件
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowToast -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is UiEvent.ShowError -> {
                    snackbarHostState.showSnackbar("错误：${event.message}")
                }
            }
        }
    }

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        when (val state = uiState) {
            is UiState.Loading -> {
                Box(modifier = Modifier.fillMaxSize()) {
                    CircularProgressIndicator(modifier = Modifier.align(Alignment.Center))
                }
            }
            is UiState.Success -> {
                UserContent(
                    user = state.data,
                    onUpdate = { name -> viewModel.updateUser(name) }
                )
            }
            is UiState.Error -> {
                ErrorView(message = state.message)
            }
        }
    }
}
```

### 跨组件状态共享

```kotlin
// 方式一：ViewModel 共享（推荐）
// 同一 Activity 的 Fragment 共享 ViewModel

// Fragment A
@HiltViewModel
class SharedViewModel @Inject constructor() : ViewModel() {
    private val _selectedItem = MutableStateFlow<Item?>(null)
    val selectedItem: StateFlow<Item?> = _selectedItem.asStateFlow()

    fun selectItem(item: Item) {
        _selectedItem.value = item
    }
}

// Fragment A
@Composable
fun FragmentA(viewModel: SharedViewModel = hiltViewModel()) {
    val items = listOf(Item("1"), Item("2"), Item("3"))

    LazyColumn {
        items(items) { item ->
            ListItem(
                item = item,
                onClick = { viewModel.selectItem(item) }
            )
        }
    }
}

// Fragment B
@Composable
fun FragmentB(viewModel: SharedViewModel = hiltViewModel()) {
    val selectedItem by viewModel.selectedItem.collectAsState()

    selectedItem?.let { item ->
        Text("Selected: ${item.name}")
    }
}

// 方式二：NavGraph 共享
@Composable
fun NavGraph() {
    val navController = rememberNavController()
    val viewModel: SharedViewModel = viewModel()  // NavGraph 作用域

    NavHost(navController, startDestination = "list") {
        composable("list") {
            ListScreen(
                onItemSelected = { item ->
                    viewModel.selectItem(item)
                    navController.navigate("detail")
                }
            )
        }
        composable("detail") {
            DetailScreen(
                item = viewModel.selectedItem.collectAsState().value
            )
        }
    }
}

// 方式三：单例状态容器（简单场景）
object AppState {
    private val _theme = MutableStateFlow<Theme>(Theme.Light)
    val theme: StateFlow<Theme> = _theme.asStateFlow()

    fun setTheme(theme: Theme) {
        _theme.value = theme
    }
}

@Composable
fun ThemedApp() {
    val theme by AppState.theme.collectAsState()

    MaterialTheme(
        colorScheme = if (theme == Theme.Dark) darkColorScheme() else lightColorScheme()
    ) {
        AppContent()
    }
}
```

### 全局状态管理方案对比

| 方案 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| **ViewModel + StateFlow** | 推荐，标准方案 | 生命周期感知、测试友好 | 需要 ViewModel |
| **CompositionLocal** | 主题、配置等全局值 | 简洁、无需传递 | 难以追踪、测试困难 |
| **单例对象** | 简单全局状态 | 简单易用 | 难以测试、生命周期问题 |
| **依赖注入（Hilt）** | 大型项目 | 解耦、可测试 | 配置复杂 |

### CompositionLocal 全局状态

```kotlin
// CompositionLocal：隐式传递数据（慎用）
// 适合：主题、配置、导航等全局值

// 定义
val LocalUser = compositionLocalOf<User?> { null }
val LocalTheme = compositionLocalOf<Theme> { Theme.Light }

// 提供
@Composable
fun AppContent() {
    val user = remember { User("windCloud", 25) }

    CompositionLocalProvider(
        LocalUser provides user,
        LocalTheme provides Theme.Dark
    ) {
        MainScreen()
    }
}

// 使用
@Composable
fun MainScreen() {
    val user = LocalUser.current
    val theme = LocalTheme.current

    Text("Welcome, ${user?.name ?: "Guest"}")
}

// 带默认值的 CompositionLocal
val LocalSnackbarHost = compositionLocalOf<SnackbarHostState?> { null }

// 使用
@Composable
fun MyComposable() {
    val snackbarHost = LocalSnackbarHost.current
    Button(onClick = {
        snackbarHost?.showSnackbar("Hello")
    }) {
        Text("Show Snackbar")
    }
}
```

## 快速对照表

| 状态类型 | 用途 | 生命周期 |
|----------|------|----------|
| `mutableStateOf` | 基础状态 | 重组保持 |
| `remember { mutableStateOf }` | 推荐基础状态 | 重组保持 |
| `rememberSaveable` | 配置变更保持 | 进程杀死丢失 |
| `StateFlow` | ViewModel 状态 | ViewModel 生命周期 |
| `SharedFlow` | 一次性事件 | ViewModel 生命周期 |
| `ViewModel` | 全局/共享状态 | ViewModel 生命周期 |

| 转换函数 | 用途 |
|----------|------|
| `collectAsState()` | StateFlow → Compose State |
| `snapshotFlow { }` | Compose State → Flow |
| `derivedStateOf { }` | 派生状态（减少重组） |

| 状态管理方案 | 推荐度 | 场景 |
|--------------|--------|------|
| ViewModel + StateFlow | ⭐⭐⭐⭐⭐ | 标准方案 |
| rememberSaveable | ⭐⭐⭐⭐ | 简单 UI 状态 |
| CompositionLocal | ⭐⭐ | 主题/配置 |
| 单例对象 | ⭐ | 不推荐 |

**记忆口诀**：
- **基础状态 → remember + mutableStateOf**
- **配置变更 → rememberSaveable**
- **子组件 → 状态提升，不持状态**
- **ViewModel → StateFlow 发状态**
- **一次性事件 → SharedFlow**
- **Compose 转 Flow → snapshotFlow**
- **Flow 转 Compose → collectAsState**
