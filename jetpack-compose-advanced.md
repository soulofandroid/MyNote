# JetPack Compose 业务进阶 & 工程化

## 1. ViewModel 深度配合

### 基础 ViewModel 集成

```kotlin
// ViewModel 定义
class UserViewModel(
    private val userRepository: UserRepository
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
            userRepository.getUser()
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
            _events.emit(UiEvent.ShowMessage("更新中..."))
            userRepository.updateName(name)
                .onSuccess {
                    _events.emit(UiEvent.ShowMessage("更新成功"))
                    loadUser()  // 刷新数据
                }
                .onFailure { error ->
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

// sealed class 事件定义
sealed class UiEvent {
    data class ShowMessage(val message: String) : UiEvent()
    data class ShowError(val message: String) : UiEvent()
    object NavigateBack : UiEvent()
}

// Compose 中使用
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    val snackbarHostState = remember { SnackbarHostState() }

    // 监听事件
    LaunchedEffect(Unit) {
        viewModel.events.collect { event ->
            when (event) {
                is UiEvent.ShowMessage -> {
                    snackbarHostState.showSnackbar(event.message)
                }
                is UiEvent.ShowError -> {
                    snackbarHostState.showSnackbar("错误：${event.message}")
                }
                is UiEvent.NavigateBack -> {
                    // 导航返回
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
                    onUpdate = { name -> viewModel.updateUser(name) },
                    modifier = Modifier.padding(padding)
                )
            }
            is UiState.Error -> {
                ErrorView(
                    message = state.message,
                    onRetry = { viewModel.loadUser() },
                    modifier = Modifier.padding(padding)
                )
            }
        }
    }
}
```

### 业务逻辑解耦

```kotlin
// 方式一：Use Case 模式（推荐）
// 业务用例类
class GetUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
    private val dispatcherProvider: DispatcherProvider
) {
    suspend operator fun invoke(userId: String): Result<User> =
        withContext(dispatcherProvider.io) {
            try {
                val user = userRepository.getUserById(userId)
                if (user != null) {
                    Result.success(user)
                } else {
                    Result.failure(UserNotFoundException())
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
}

class UpdateUserUseCase @Inject constructor(
    private val userRepository: UserRepository,
    private val dispatcherProvider: DispatcherProvider
) {
    suspend operator fun invoke(user: User): Result<Unit> =
        withContext(dispatcherProvider.io) {
            try {
                userRepository.updateUser(user)
                Result.success(Unit)
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
}

// ViewModel 使用 Use Case
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase,
    private val updateUserUseCase: UpdateUserUseCase,
    savedStateHandle: SavedStateHandle
) : ViewModel() {

    private val userId: String = savedStateHandle["userId"] ?: ""

    private val _uiState = MutableStateFlow<UiState<User>>(UiState.Loading)
    val uiState: StateFlow<UiState<User>> = _uiState.asStateFlow()

    init {
        loadUser()
    }

    private fun loadUser() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            getUserUseCase(userId)
                .onSuccess { user ->
                    _uiState.value = UiState.Success(user)
                }
                .onFailure { error ->
                    _uiState.value = UiState.Error(error.message ?: "Unknown error")
                }
        }
    }

    fun updateUser(name: String) {
        viewModelScope.launch {
            val currentState = _uiState.value
            if (currentState is UiState.Success) {
                val updatedUser = currentState.data.copy(name = name)
                updateUserUseCase(updatedUser)
                    .onSuccess {
                        _uiState.value = UiState.Success(updatedUser)
                    }
                    .onFailure { error ->
                        _uiState.value = currentState.copy(error = error.message ?: "Unknown error")
                    }
            }
        }
    }
}

// 方式二：Repository 模式
interface UserRepository {
    suspend fun getUser(): Result<User>
    suspend fun updateUser(user: User): Result<Unit>
}

class UserRepositoryImpl @Inject constructor(
    private val api: UserApi,
    private val localDataSource: UserLocalDataSource
) : UserRepository {

    override suspend fun getUser(): Result<User> {
        return try {
            // 先尝试本地缓存
            val localUser = localDataSource.getUser()
            if (localUser != null) {
                Result.success(localUser)
            } else {
                // 从网络获取
                val networkUser = api.getUser()
                localDataSource.saveUser(networkUser)
                Result.success(networkUser)
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    override suspend fun updateUser(user: User): Result<Unit> {
        return try {
            api.updateUser(user)
            localDataSource.saveUser(user)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 生命周期感知

```kotlin
// 生命周期感知的数据收集
@Composable
fun LifecycleAwareScreen(viewModel: MyViewModel) {
    val lifecycleOwner = LocalLifecycleOwner.current

    // 只在 STARTED 状态收集
    val data by viewModel.dataFlow.collectAsState(
        initial = LoadingState,
        lifecycle = lifecycleOwner.lifecycle,
        minActiveState = Lifecycle.State.STARTED
    )

    // 或者使用 repeatOnLifecycle
    LaunchedEffect(Unit) {
        lifecycleOwner.lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
            viewModel.dataFlow.collect { data ->
                // 处理数据
            }
        }
    }

    // 内容
}

// Fragment 中的生命周期感知
class MyFragment : Fragment() {
    private val viewModel: MyViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { state ->
                    // 更新 UI
                }
            }
        }
    }
}

// 自动取消的协程
@Composable
fun AutoCancelEffect(viewModel: MyViewModel) {
    // LaunchedEffect 自动在 Composable 离开 composition 时取消
    LaunchedEffect(Unit) {
        viewModel.loadData()
    }

    // 带 key 的 LaunchedEffect，key 变化时重新执行
    LaunchedEffect(viewModel.userId) {
        viewModel.loadUser(viewModel.userId)
    }
}

// DisposableEffect：清理资源
@Composable
fun DisposableEffectExample() {
    var listener by remember { mutableStateOf<LocationListener?>(null) }

    DisposableEffect(Unit) {
        // 进入时执行
        listener = LocationListener { location ->
            // 处理位置更新
        }
        locationManager.requestLocationUpdates(listener)

        // 离开时清理
        onDispose {
            locationManager.removeUpdates(listener)
        }
    }
}
```

## 2. 权限封装

### 基础权限请求

```kotlin
// 使用 Accompanist Permissions
// 添加依赖
// implementation "com.google.accompanist:accompanist-permissions:0.32.0"

@Composable
fun PermissionExample() {
    val permissionState = rememberPermissionState(
        permission = Manifest.permission.CAMERA
    )

    Column {
        if (permissionState.status.isGranted) {
            Text("相机权限已授予")
            // 使用相机功能
        } else {
            Text("需要相机权限")
            Button(onClick = { permissionState.launchPermissionRequest() }) {
                Text("请求权限")
            }

            // 权限被拒绝
            if (permissionState.status.shouldShowRationale) {
                Text("需要相机权限来拍照")
            }

            // 权限被永久拒绝
            if (!permissionState.status.shouldShowRationale) {
                Text("请在设置中启用相机权限")
                Button(onClick = { openAppSettings() }) {
                    Text("打开设置")
                }
            }
        }
    }
}

// 多个权限请求
@Composable
fun MultiplePermissionsExample() {
    val permissions = listOf(
        Manifest.permission.CAMERA,
        Manifest.permission.RECORD_AUDIO
    )

    val multiplePermissionsState = rememberMultiplePermissionsState(
        permissions = permissions
    )

    Column {
        val allGranted = multiplePermissionsState.allPermissionsGranted

        if (allGranted) {
            Text("所有权限已授予")
        } else {
            Text("需要以下权限:")
            multiplePermissionsState.revokedPermissions.forEach { permission ->
                Text("- ${permission.permission}")
            }

            Button(
                onClick = { multiplePermissionsState.launchMultiplePermissionRequest() },
                enabled = !multiplePermissionsState.shouldShowRationale
            ) {
                Text("请求权限")
            }

            if (multiplePermissionsState.shouldShowRationale) {
                Text("需要这些权限来使用完整功能")
            }
        }
    }
}
```

### 权限封装工具类

```kotlin
// 权限管理器
class PermissionManager @Inject constructor() {

    sealed class PermissionResult {
        object Granted : PermissionResult()
        data class Denied(val shouldShowRationale: Boolean) : PermissionResult()
    }

    suspend fun requestPermission(
        context: Context,
        permission: String
    ): PermissionResult {
        return suspendCoroutine { continuation ->
            val activity = context as? Activity ?: run {
                continuation.resume(PermissionResult.Denied(false))
                return@suspendCoroutine
            }

            if (ContextCompat.checkSelfPermission(context, permission) ==
                PackageManager.PERMISSION_GRANTED
            ) {
                continuation.resume(PermissionResult.Granted)
            } else {
                ActivityCompat.requestPermissions(
                    activity,
                    arrayOf(permission),
                    REQUEST_CODE
                )

                // 实际项目中需要使用回调或 Flow 来接收结果
            }
        }
    }

    companion object {
        private const val REQUEST_CODE = 1001
    }
}

// Compose 权限封装
@Composable
fun rememberPermission(
    permission: String,
    onResult: (Boolean) -> Unit
): PermissionState {
    val context = LocalContext.current
    val permissionState = rememberPermissionState(permission = permission)

    LaunchedEffect(permissionState.status) {
        when {
            permissionState.status.isGranted -> {
                onResult(true)
            }
            !permissionState.status.shouldShowRationale -> {
                // 永久拒绝
                onResult(false)
            }
        }
    }

    return permissionState
}

// 使用示例
@Composable
fun CameraScreen() {
    val permissionState = rememberPermission(
        permission = Manifest.permission.CAMERA
    ) { granted ->
        if (granted) {
            // 显示相机预览
        }
    }

    if (permissionState.status.isGranted) {
        CameraPreview()
    } else {
        PermissionRequestView(
            onPermissionRequest = { permissionState.launchPermissionRequest() }
        )
    }
}
```

### 权限检查扩展

```kotlin
// Context 扩展
fun Context.hasPermission(permission: String): Boolean {
    return ContextCompat.checkSelfPermission(this, permission) ==
            PackageManager.PERMISSION_GRANTED
}

fun Context.shouldShowPermissionRationale(permission: String): Boolean {
    return when (this) {
        is Activity -> shouldShowRequestPermissionRationale(permission)
        is Fragment -> shouldShowRequestPermissionRationale(permission)
        else -> false
    }
}

// Flow 权限状态
fun Context.permissionFlow(permission: String): Flow<Boolean> = flow {
    emit(hasPermission(permission))

    // 监听权限变化（需要配合 ActivityResultRegistry）
}

// ViewModel 中使用
class PermissionViewModel(
    private val context: Context
) : ViewModel() {

    private val _cameraPermission = MutableStateFlow(false)
    val cameraPermission: StateFlow<Boolean> = _cameraPermission.asStateFlow()

    init {
        checkPermission()
    }

    private fun checkPermission() {
        _cameraPermission.value = context.hasPermission(Manifest.permission.CAMERA)
    }

    fun requestPermission(activity: Activity) {
        ActivityCompat.requestPermissions(
            activity,
            arrayOf(Manifest.permission.CAMERA),
            REQUEST_CODE
        )
    }

    fun onPermissionResult(granted: Boolean) {
        _cameraPermission.value = granted
    }

    companion object {
        private const val REQUEST_CODE = 1001
    }
}
```

## 3. 网络 + Compose

### 网络状态封装

```kotlin
// 网络状态管理
sealed class NetworkState<out T> {
    object Loading : NetworkState<Nothing>()
    data class Success<T>(val data: T) : NetworkState<T>()
    data class Error(val message: String, val code: Int? = null) : NetworkState<Nothing>()
    object Empty : NetworkState<Nothing>()
}

// 通用 Repository
abstract class BaseRepository {
    protected suspend fun <T> apiCall(
        call: suspend () -> T
    ): NetworkState<T> {
        return try {
            val result = call()
            if (result is Collection<*> && result.isEmpty()) {
                NetworkState.Empty
            } else {
                NetworkState.Success(result)
            }
        } catch (e: HttpException) {
            NetworkState.Error(e.message ?: "HTTP Error", e.code())
        } catch (e: Exception) {
            NetworkState.Error(e.message ?: "Unknown error")
        }
    }
}

// ViewModel 中使用
class ArticleViewModel(
    private val repository: ArticleRepository
) : ViewModel() {

    private val _articles = MutableStateFlow<NetworkState<List<Article>>>(NetworkState.Loading)
    val articles: StateFlow<NetworkState<List<Article>>> = _articles.asStateFlow()

    fun loadArticles() {
        viewModelScope.launch {
            _articles.value = repository.getArticles()
        }
    }
}

// Compose 中处理不同状态
@Composable
fun ArticleScreen(viewModel: ArticleViewModel = viewModel()) {
    val articlesState by viewModel.articles.collectAsState()

    when (val state = articlesState) {
        is NetworkState.Loading -> {
            LoadingView()
        }
        is NetworkState.Success -> {
            ArticleList(articles = state.data)
        }
        is NetworkState.Error -> {
            ErrorView(
                message = state.message,
                onRetry = { viewModel.loadArticles() }
            )
        }
        is NetworkState.Empty -> {
            EmptyView(onRefresh = { viewModel.loadArticles() })
        }
    }
}
```

### 加载/空布局/错误页统一处理

```kotlin
// 通用状态处理组件
@Composable
fun <T> StateHandler(
    state: NetworkState<T>,
    onRetry: () -> Unit,
    onRefresh: () -> Unit,
    loadingContent: @Composable () -> Unit = { DefaultLoadingView() },
    emptyContent: @Composable () -> Unit = { DefaultEmptyView(onRefresh) },
    errorContent: @Composable (String) -> Unit = { DefaultErrorView(it, onRetry) },
    successContent: @Composable (T) -> Unit
) {
    when (state) {
        is NetworkState.Loading -> {
            loadingContent()
        }
        is NetworkState.Success -> {
            successContent(state.data)
        }
        is NetworkState.Error -> {
            errorContent(state.message)
        }
        is NetworkState.Empty -> {
            emptyContent()
        }
    }
}

// 默认加载视图
@Composable
fun DefaultLoadingView() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            CircularProgressIndicator()
            Spacer(modifier = Modifier.height(16.dp))
            Text("加载中...")
        }
    }
}

// 默认空视图
@Composable
fun DefaultEmptyView(onRefresh: () -> Unit) {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Icon(
                imageVector = Icons.Default.Inbox,
                contentDescription = null,
                modifier = Modifier.size(64.dp),
                tint = Color.Gray
            )
            Spacer(modifier = Modifier.height(16.dp))
            Text("暂无数据", style = MaterialTheme.typography.titleMedium)
            Spacer(modifier = Modifier.height(8.dp))
            Text("下拉刷新获取更多", style = MaterialTheme.typography.bodyMedium)
            Spacer(modifier = Modifier.height(16.dp))
            Button(onClick = onRefresh) {
                Text("刷新")
            }
        }
    }
}

// 默认错误视图
@Composable
fun DefaultErrorView(message: String, onRetry: () -> Unit) {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Icon(
                imageVector = Icons.Default.Error,
                contentDescription = null,
                modifier = Modifier.size(64.dp),
                tint = Color.Red
            )
            Spacer(modifier = Modifier.height(16.dp))
            Text("加载失败", style = MaterialTheme.typography.titleMedium)
            Spacer(modifier = Modifier.height(8.dp))
            Text(message, style = MaterialTheme.typography.bodyMedium)
            Spacer(modifier = Modifier.height(16.dp))
            Button(onClick = onRetry) {
                Text("重试")
            }
        }
    }
}

// 使用示例
@Composable
fun UserListScreen(viewModel: UserViewModel = viewModel()) {
    val state by viewModel.users.collectAsState()

    StateHandler(
        state = state,
        onRetry = { viewModel.loadUsers() },
        onRefresh = { viewModel.loadUsers() },
        successContent = { users ->
            LazyColumn {
                items(users) { user ->
                    UserItem(user = user)
                }
            }
        }
    )
}
```

### 下拉刷新 + 分页

```kotlin
// 下拉刷新
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun RefreshableList(viewModel: ArticleViewModel = viewModel()) {
    val state by viewModel.articles.collectAsState()
    val pullRefreshState = rememberPullRefreshState(
        refreshing = state is NetworkState.Loading,
        onRefresh = { viewModel.loadArticles() }
    )

    Box(
        modifier = Modifier
            .fillMaxSize()
            .pullRefresh(pullRefreshState)
    ) {
        when (val networkState = state) {
            is NetworkState.Success -> {
                LazyColumn {
                    items(networkState.data) { article ->
                        ArticleItem(article = article)
                    }
                }
            }
            is NetworkState.Error -> {
                DefaultErrorView(networkState.message) {
                    viewModel.loadArticles()
                }
            }
            is NetworkState.Empty -> {
                DefaultEmptyView {
                    viewModel.loadArticles()
                }
            }
            is NetworkState.Loading -> {
                // 初始加载显示加载指示器
                if (networkState.data == null) {
                    DefaultLoadingView()
                }
            }
        }

        PullRefreshIndicator(
            refreshing = state is NetworkState.Loading,
            state = pullRefreshState,
            modifier = Modifier.align(Alignment.TopCenter)
        )
    }
}

// 分页加载（Paging 3）
@Composable
fun PagedArticleList(viewModel: ArticleViewModel = viewModel()) {
    val pagingItems = viewModel.articles.collectAsLazyPagingItems()

    LazyColumn {
        items(
            count = pagingItems.itemCount,
            key = pagingItems::peekAt,
            contentType = { position ->
                when {
                    pagingItems[position] == null -> "loading"
                    else -> "article"
                }
            }
        ) { index ->
            val article = pagingItems[index]
            when {
                article == null -> {
                    // 加载中
                    Box(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        contentAlignment = Alignment.Center
                    ) {
                        CircularProgressIndicator(modifier = Modifier.size(24.dp))
                    }
                }
                else -> {
                    ArticleItem(article = article)
                }
            }
        }

        // 处理加载状态
        pagingItems.apply {
            when {
                loadState.refresh is LoadState.Loading -> {
                    item {
                        Box(
                            modifier = Modifier.fillMaxWidth(),
                            contentAlignment = Alignment.Center
                        ) {
                            CircularProgressIndicator()
                        }
                    }
                }
                loadState.append is LoadState.Loading -> {
                    item {
                        Box(
                            modifier = Modifier.fillMaxWidth(),
                            contentAlignment = Alignment.Center
                        ) {
                            CircularProgressIndicator()
                        }
                    }
                }
                loadState.prepend is LoadState.Error -> {
                    item {
                        val error = loadState.prepend as LoadState.Error
                        ErrorItem(error = error)
                    }
                }
                loadState.append is LoadState.Error -> {
                    item {
                        val error = loadState.append as LoadState.Error
                        ErrorItem(error = error)
                    }
                }
            }
        }
    }
}
```

## 4. 自定义组件

### 通用标题栏

```kotlin
// 通用标题栏
@Composable
fun CommonAppBar(
    title: String,
    modifier: Modifier = Modifier,
    showBackButton: Boolean = true,
    onBackClick: () -> Unit = {},
    actions: @Composable RowScope.() -> Unit = {}
) {
    TopAppBar(
        title = { Text(title) },
        navigationIcon = {
            if (showBackButton) {
                IconButton(onClick = onBackClick) {
                    Icon(
                        Icons.Default.ArrowBack,
                        contentDescription = "返回"
                    )
                }
            }
        },
        actions = actions,
        modifier = modifier
    )
}

// 使用示例
@Composable
fun DetailScreen() {
    Scaffold(
        topBar = {
            CommonAppBar(
                title = "详情",
                actions = {
                    IconButton(onClick = { /* 分享 */ }) {
                        Icon(Icons.Default.Share, contentDescription = "分享")
                    }
                    IconButton(onClick = { /* 更多 */ }) {
                        Icon(Icons.Default.MoreVert, contentDescription = "更多")
                    }
                }
            )
        }
    ) { padding ->
        // 内容
    }
}

// 带搜索的标题栏
@Composable
fun SearchAppBar(
    query: String,
    onQueryChange: (String) -> Unit,
    onSearch: (String) -> Unit,
    onBackClick: () -> Unit
) {
    TopAppBar(
        title = {
            TextField(
                value = query,
                onValueChange = onQueryChange,
                placeholder = { Text("搜索") },
                singleLine = true,
                colors = TextFieldDefaults.colors(
                    focusedContainerColor = Color.Transparent,
                    unfocusedContainerColor = Color.Transparent,
                    focusedIndicatorColor = Color.Transparent,
                    unfocusedIndicatorColor = Color.Transparent
                ),
                modifier = Modifier.fillMaxWidth()
            )
        },
        navigationIcon = {
            IconButton(onClick = onBackClick) {
                Icon(Icons.Default.ArrowBack, contentDescription = "返回")
            }
        },
        actions = {
            if (query.isNotEmpty()) {
                IconButton(onClick = { onQueryChange("") }) {
                    Icon(Icons.Default.Clear, contentDescription = "清除")
                }
            }
        }
    )
}
```

### 通用列表

```kotlin
// 通用列表项
@Composable
fun CommonListItem(
    title: String,
    subtitle: String? = null,
    icon: ImageVector? = null,
    trailingContent: @Composable (() -> Unit)? = null,
    onClick: () -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(onClick = onClick)
            .padding(horizontal = 16.dp, vertical = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        if (icon != null) {
            Icon(
                imageVector = icon,
                contentDescription = null,
                modifier = Modifier.size(24.dp),
                tint = MaterialTheme.colorScheme.primary
            )
            Spacer(modifier = Modifier.width(16.dp))
        }

        Column(modifier = Modifier.weight(1f)) {
            Text(
                text = title,
                style = MaterialTheme.typography.bodyLarge
            )
            if (subtitle != null) {
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = subtitle,
                    style = MaterialTheme.typography.bodyMedium,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }

        if (trailingContent != null) {
            Spacer(modifier = Modifier.width(8.dp))
            trailingContent()
        } else {
            Icon(
                Icons.Default.ChevronRight,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}

// 通用列表容器
@Composable
fun <T> CommonList(
    items: List<T>,
    key: ((T) -> Any)? = null,
    emptyMessage: String = "暂无数据",
    itemContent: @Composable ColumnScope.(T) -> Unit
) {
    if (items.isEmpty()) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Text(emptyMessage)
        }
    } else {
        LazyColumn {
            if (key != null) {
                items(items = items, key = key) { item ->
                    itemContent(item)
                }
            } else {
                items(items) { item ->
                    itemContent(item)
            }
            }
        }
    }
}

// 使用示例
@Composable
fun SettingsScreen() {
    val settings = listOf(
        Setting("账号", "windCloud"),
        Setting("通知", "已开启"),
        Setting("隐私", null),
        Setting("关于", "v1.0.0")
    )

    CommonList(
        items = settings,
        key = { it.title }
    ) { setting ->
        CommonListItem(
            title = setting.title,
            subtitle = setting.subtitle,
            onClick = { /* 导航到设置详情页 */ }
        )
    }
}

data class Setting(val title: String, val subtitle: String?)
```

### 空状态页

```kotlin
// 通用空状态组件
@Composable
fun EmptyState(
    message: String = "暂无数据",
    actionText: String? = null,
    onAction: (() -> Unit)? = null,
    modifier: Modifier = Modifier,
    imageVector: ImageVector = Icons.Default.Inbox
) {
    Box(
        modifier = modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(
            horizontalAlignment = Alignment.CenterHorizontally,
            modifier = Modifier.padding(32.dp)
        ) {
            Icon(
                imageVector = imageVector,
                contentDescription = null,
                modifier = Modifier.size(80.dp),
                tint = MaterialTheme.colorScheme.onSurfaceVariant.copy(alpha = 0.5f)
            )

            Spacer(modifier = Modifier.height(24.dp))

            Text(
                text = message,
                style = MaterialTheme.typography.titleMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )

            if (actionText != null && onAction != null) {
                Spacer(modifier = Modifier.height(24.dp))

                Button(onClick = onAction) {
                    Text(actionText)
                }
            }
        }
    }
}

// 特定场景空状态
@Composable
fun EmptyCart(onShop: () -> Unit) {
    EmptyState(
        message = "购物车是空的",
        actionText = "去购物",
        onAction = onShop,
        imageVector = Icons.Default.ShoppingCart
    )
}

@Composable
fun EmptySearch(onClear: () -> Unit) {
    EmptyState(
        message = "没有找到相关内容",
        actionText = "清除搜索",
        onAction = onClear,
        imageVector = Icons.Default.Search
    )
}

@Composable
fun EmptyMessages(onCompose: () -> Unit) {
    EmptyState(
        message = "暂无消息",
        actionText = "写消息",
        onAction = onCompose,
        imageVector = Icons.Default.Mail
    )
}
```

## 5. 跨页面状态共享

### ViewModel 共享

```kotlin
// 同一 Activity 内共享 ViewModel
@HiltViewModel
class SharedViewModel @Inject constructor() : ViewModel() {
    private val _selectedItem = MutableStateFlow<Item?>(null)
    val selectedItem: StateFlow<Item?> = _selectedItem.asStateFlow()

    fun selectItem(item: Item) {
        _selectedItem.value = item
    }

    fun clearSelection() {
        _selectedItem.value = null
    }
}

// Fragment A - 列表页
@Composable
fun ListFragment(viewModel: SharedViewModel = hiltViewModel()) {
    val items = listOf(Item("1"), Item("2"), Item("3"))

    LazyColumn {
        items(items) { item ->
            ListItem(
                item = item,
                onClick = {
                    viewModel.selectItem(item)
                    // 导航到详情页
                }
            )
        }
    }
}

// Fragment B - 详情页
@Composable
fun DetailFragment(viewModel: SharedViewModel = hiltViewModel()) {
    val selectedItem by viewModel.selectedItem.collectAsState()

    selectedItem?.let { item ->
        Text("Selected: ${item.name}")
    } ?: run {
        Text("No item selected")
    }
}

// NavGraph 作用域共享
@Composable
fun AppNavGraph() {
    val navController = rememberNavController()
    // 在 NavGraph 级别创建 ViewModel
    val sharedViewModel: SharedViewModel = viewModel(navGraphViewModels = true)

    NavHost(navController = navController, startDestination = "list") {
        composable("list") {
            ListScreen(
                onItemSelected = { item ->
                    sharedViewModel.selectItem(item)
                    navController.navigate("detail")
                }
            )
        }
        composable("detail") {
            DetailScreen(
                item = sharedViewModel.selectedItem.collectAsState().value
            )
        }
    }
}
```

### 全局状态管理

```kotlin
// 方式一：单例状态容器（简单场景）
object AppState {
    private val _theme = MutableStateFlow<Theme>(Theme.Light)
    val theme: StateFlow<Theme> = _theme.asStateFlow()

    private val _user = MutableStateFlow<User?>(null)
    val user: StateFlow<User?> = _user.asStateFlow()

    fun setTheme(theme: Theme) {
        _theme.value = theme
    }

    fun setUser(user: User?) {
        _user.value = user
    }
}

// 在 Application 中使用
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    val theme by AppState.theme.collectAsState()

    MaterialTheme(
        colorScheme = if (theme == Theme.Dark) darkColorScheme() else lightColorScheme()
    ) {
        content()
    }
}

// 方式二：Hilt 依赖注入（推荐）
@Singleton
class GlobalStateManager @Inject constructor() {
    private val _theme = MutableStateFlow<Theme>(Theme.Light)
    val theme: StateFlow<Theme> = _theme.asStateFlow()

    private val _user = MutableStateFlow<User?>(null)
    val user: StateFlow<User?> = _user.asStateFlow()

    fun setTheme(theme: Theme) {
        _theme.value = theme
    }

    fun setUser(user: User?) {
        _user.value = user
    }
}

// 在 ViewModel 中注入
@HiltViewModel
class SettingsViewModel @Inject constructor(
    private val globalStateManager: GlobalStateManager
) : ViewModel() {

    val theme = globalStateManager.theme

    fun toggleTheme() {
        val currentTheme = theme.value
        globalStateManager.setTheme(
            if (currentTheme == Theme.Light) Theme.Dark else Theme.Light
        )
    }
}

// 方式三：CompositionLocal（主题等全局值）
val LocalUser = compositionLocalOf<User?> { null }
val LocalTheme = compositionLocalOf<Theme> { Theme.Light }

@Composable
fun AppContent(user: User?, theme: Theme) {
    CompositionLocalProvider(
        LocalUser provides user,
        LocalTheme provides theme
    ) {
        MainScreen()
    }
}

// 在任何 Composable 中使用
@Composable
fun UserProfile() {
    val user = LocalUser.current
    val theme = LocalTheme.current

    Text("Welcome, ${user?.name ?: "Guest"}")
}
```

### 跨进程状态同步

```kotlin
// 使用 DataStore 持久化全局状态
@Singleton
class SettingsRepository @Inject constructor(
    private val dataStore: DataStore<Preferences>
) {
    val theme: Flow<Theme> = dataStore.data.map { preferences ->
        val themeValue = preferences[THEME_KEY] ?: "light"
        Theme.valueOf(themeValue.uppercase())
    }

    suspend fun setTheme(theme: Theme) {
        dataStore.edit { preferences ->
            preferences[THEME_KEY] = theme.name.lowercase()
        }
    }

    companion object {
        private val THEME_KEY = stringPreferencesKey("theme")
    }
}

// ViewModel 中同步
@HiltViewModel
class ThemeViewModel @Inject constructor(
    private val settingsRepository: SettingsRepository
) : ViewModel() {

    val theme: StateFlow<Theme> = settingsRepository.theme
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = Theme.Light
        )

    fun setTheme(theme: Theme) {
        viewModelScope.launch {
            settingsRepository.setTheme(theme)
        }
    }
}
```

## 快速对照表

| 模块 | 组件/模式 | 用途 |
|------|----------|------|
| ViewModel | `StateFlow` + `collectAsState` | UI 状态管理 |
| ViewModel | `SharedFlow` + `LaunchedEffect` | 一次性事件 |
| 业务解耦 | Use Case / Repository | 分层架构 |
| 权限 | Accompanist Permissions | 权限请求 |
| 网络状态 | `NetworkState<T>` | 统一状态处理 |
| 状态处理 | `StateHandler` | 加载/空/错误统一处理 |
| 自定义组件 | `CommonAppBar` | 通用标题栏 |
| 自定义组件 | `CommonListItem` | 通用列表项 |
| 自定义组件 | `EmptyState` | 空状态页 |
| 跨页面共享 | ViewModel 共享 | 同 Activity 状态共享 |
| 全局状态 | 单例 / DI / CompositionLocal | 全局状态管理 |

**记忆口诀**：
- **ViewModel + StateFlow = UI 状态**
- **SharedFlow + LaunchedEffect = 一次性事件**
- **Use Case = 业务逻辑封装**
- **NetworkState = 加载/成功/错误/空**
- **StateHandler = 统一状态处理**
- **CommonAppBar/CommonListItem = 通用组件**
- **EmptyState = 空状态页**
- **ViewModel 共享 = 跨页面状态**
- **CompositionLocal = 全局隐式传值**
