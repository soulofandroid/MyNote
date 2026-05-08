# JetPack Compose 页面架构与导航模块

## 1. Scaffold 脚手架

### Scaffold 基础结构

```kotlin
// Scaffold：Material Design 页面骨架
// 提供标准布局结构：TopBar、BottomBar、FAB、Drawer、Content

@Composable
fun ScaffoldExample() {
    Scaffold(
        // 顶部栏
        topBar = {
            TopAppBar(
                title = { Text("My App") }
            )
        },
        // 底部栏
        bottomBar = {
            BottomAppBar {
                Text("Bottom Bar")
            }
        },
        // 悬浮按钮
        floatingActionButton = {
            FloatingActionButton(onClick = { }) {
                Icon(Icons.Default.Add, contentDescription = "Add")
            }
        },
        // 内容区域
        content = { paddingValues ->
            // paddingValues 包含底部栏等占用的内边距
            Column(modifier = Modifier.padding(paddingValues)) {
                Text("Content")
            }
        }
    )
}

// Scaffold 完整参数
@Composable
fun FullScaffold() {
    val scaffoldState = rememberScaffoldState()

    Scaffold(
        scaffoldState = scaffoldState,
        topBar = { TopAppBarExample() },
        bottomBar = { BottomAppBarExample() },
        floatingActionButton = { FABExample() },
        floatingActionButtonPosition = FabPosition.End,  // FAB 位置
        snackbarHost = {
            SnackbarHost(hostState = scaffoldState.snackbarHostState)
        },
        drawerContent = { DrawerContent() },
        content = { padding ->
            // 内容
        }
    )
}
```

### TopAppBar 顶部栏

```kotlin
// 基础顶部栏
@Composable
fun BasicTopAppBar() {
    TopAppBar(
        title = { Text("Title") },
        colors = TopAppBarDefaults.topAppBarColors(
            containerColor = Color.Blue,
            titleContentColor = Color.White,
            navigationIconContentColor = Color.White,
            actionIconContentColor = Color.White
        )
    )
}

// 带导航图标
@Composable
fun TopAppBarWithNavigation(onNavigateClick: () -> Unit) {
    TopAppBar(
        title = { Text("Title") },
        navigationIcon = {
            IconButton(onClick = onNavigateClick) {
                Icon(Icons.Default.ArrowBack, contentDescription = "Back")
            }
        },
        actions = {
            IconButton(onClick = { }) {
                Icon(Icons.Default.Search, contentDescription = "Search")
            }
            IconButton(onClick = { }) {
                Icon(Icons.Default.MoreVert, contentDescription = "More")
            }
        }
    )
}

// 带进度条
@Composable
fun TopAppBarWithProgress(progress: Float) {
    TopAppBar(
        title = { Text("Title") },
        progressIndicator = {
            LinearProgressIndicator(
                progress = { progress },
                modifier = Modifier.fillMaxWidth()
            )
        }
    )
}

// Medium TopAppBar（Material 3）
@Composable
fun MediumTopAppBar() {
    MediumTopAppBar(
        title = { Text("Medium App Bar") },
        scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior()
    )
}

// Large TopAppBar（Material 3）
@Composable
fun LargeTopAppBar() {
    LargeTopAppBar(
        title = { Text("Large App Bar") },
        scrollBehavior = TopAppBarDefaults.exitUntilCollapsedScrollBehavior()
    )
}

// 带滚动行为
@Composable
fun ScrollableTopAppBar() {
    val scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Scrollable") },
                scrollBehavior = scrollBehavior
            )
        }
    ) { padding ->
        LazyColumn(
            contentPadding = padding,
            modifier = Modifier.nestedScroll(scrollBehavior.nestedScrollConnection)
        ) {
            items(100) { i ->
                Text("Item $i", modifier = Modifier.padding(16.dp))
            }
        }
    }
}
```

### BottomAppBar 底部栏

```kotlin
// 基础底部栏
@Composable
fun BasicBottomAppBar() {
    BottomAppBar(
        containerColor = Color.White,
        contentColor = Color.Black,
        tonalElevation = 8.dp
    ) {
        // 内容
        Text("Bottom App Bar", modifier = Modifier.padding(16.dp))
    }
}

// 带导航和内容
@Composable
fun BottomAppBarWithContent() {
    BottomAppBar(
        actions = {
            IconButton(onClick = { }) {
                Icon(Icons.Default.Search, contentDescription = "Search")
            }
            IconButton(onClick = { }) {
                Icon(Icons.Default.Favorite, contentDescription = "Favorite")
            }
            IconButton(onClick = { }) {
                Icon(Icons.Default.MoreVert, contentDescription = "More")
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = { },
                containerColor = Color.Blue
            ) {
                Icon(Icons.Default.Add, contentDescription = "Add")
            }
        }
    )
}

// 底部栏 + FAB 组合
@Composable
fun BottomBarWithFAB() {
    Scaffold(
        bottomBar = {
            BottomAppBar(
                actions = {
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.Home, contentDescription = "Home")
                    }
                    IconButton(onClick = { }) {
                        Icon(Icons.Default.Star, contentDescription = "Star")
                    }
                },
                floatingActionButton = {
                    FloatingActionButton(onClick = { }) {
                        Icon(Icons.Default.Add, contentDescription = "Add")
                    }
                }
            )
        }
    ) { padding ->
        // 内容
    }
}
```

### FloatingActionButton 悬浮按钮

```kotlin
// 基础 FAB
@Composable
fun BasicFAB(onClick: () -> Unit) {
    FloatingActionButton(
        onClick = onClick,
        containerColor = Color.Blue,
        contentColor = Color.White
    ) {
        Icon(Icons.Default.Add, contentDescription = "Add")
    }
}

// 小尺寸 FAB
@Composable
fun SmallFAB(onClick: () -> Unit) {
    SmallFloatingActionButton(
        onClick = onClick,
        containerColor = Color.Green
    ) {
        Icon(Icons.Default.Edit, contentDescription = "Edit")
    }
}

// 大尺寸 FAB
@Composable
fun LargeFAB(onClick: () -> Unit) {
    LargeFloatingActionButton(
        onClick = onClick,
        containerColor = Color.Purple
    ) {
        Icon(Icons.Default.Add, contentDescription = "Add")
    }
}

// 带图标的 FAB
@Composable
fun ExtendedFAB(onClick: () -> Unit) {
    ExtendedFloatingActionButton(
        onClick = onClick,
        icon = {
            Icon(Icons.Default.Add, contentDescription = null)
        },
        text = {
            Text("Create")
        },
        containerColor = Color.Blue
    )
}

// FAB 滚动行为
@Composable
fun ScrollableFAB() {
    val scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior()

    Scaffold(
        floatingActionButton = {
            FloatingActionButton(
                onClick = { },
                modifier = Modifier.hideOnScroll(scrollBehavior)
            ) {
                Icon(Icons.Default.Add, contentDescription = "Add")
            }
        }
    ) { padding ->
        // 滚动时 FAB 隐藏
    }
}

// 自定义 FAB 形状
@Composable
fun CustomShapeFAB() {
    FloatingActionButton(
        onClick = { },
        shape = RoundedCornerShape(16.dp)
    ) {
        Icon(Icons.Default.Star, contentDescription = "Star")
    }
}
```

### ModalNavigationDrawer 侧边栏

```kotlin
// 添加依赖
// implementation "androidx.compose.material:material-icons-extended"

@Composable
fun DrawerExample() {
    val drawerState = rememberDrawerState(initialValue = DrawerValue.Closed)
    val scope = rememberCoroutineScope()

    ModalNavigationDrawer(
        drawerState = drawerState,
        drawerContent = {
            ModalDrawerSheet {
                DrawerHeader()
                DrawerItem(
                    icon = Icons.Default.Home,
                    label = "Home",
                    selected = false,
                    onClick = {
                        scope.launch { drawerState.close() }
                    }
                )
                DrawerItem(
                    icon = Icons.Default.Settings,
                    label = "Settings",
                    selected = false,
                    onClick = {
                        scope.launch { drawerState.close() }
                    }
                )
                HorizontalDivider()
                DrawerItem(
                    icon = Icons.Default.Logout,
                    label = "Logout",
                    selected = false,
                    onClick = {
                        scope.launch { drawerState.close() }
                    }
                )
            }
        }
    ) {
        Scaffold(
            topBar = {
                TopAppBar(
                    title = { Text("Drawer App") },
                    navigationIcon = {
                        IconButton(onClick = {
                            scope.launch { drawerState.open() }
                        }) {
                            Icon(Icons.Default.Menu, contentDescription = "Menu")
                        }
                    }
                )
            }
        ) { padding ->
            // 主内容
        }
    }
}

// Drawer Header
@Composable
fun DrawerHeader() {
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .height(180.dp)
            .background(Color.Blue)
            .padding(16.dp),
        contentAlignment = Alignment.BottomStart
    ) {
        Column {
            Icon(
                Icons.Default.Person,
                contentDescription = null,
                modifier = Modifier.size(64.dp),
                tint = Color.White
            )
            Spacer(modifier = Modifier.height(8.dp))
            Text("windCloud", color = Color.White, fontWeight = FontWeight.Bold)
            Text("wind@example.com", color = Color.White.copy(0.7f))
        }
    }
}

// Drawer Item
@Composable
fun DrawerItem(
    icon: ImageVector,
    label: String,
    selected: Boolean,
    onClick: () -> Unit
) {
    NavigationDrawerItem(
        icon = { Icon(icon, contentDescription = null) },
        label = { Text(label) },
        selected = selected,
        onClick = onClick,
        modifier = Modifier.padding(horizontal = 8.dp, vertical = 4.dp)
    )
}

// Permanent Drawer（永久显示）
@Composable
fun PermanentDrawer() {
    Row(modifier = Modifier.fillMaxSize()) {
        NavigationDrawer(
            drawerState = rememberDrawerState(DrawerValue.Open),
            gesturesEnabled = false,
            drawerContent = {
                ModalDrawerSheet(modifier = Modifier.width(256.dp)) {
                    // Drawer 内容
                }
            }
        ) {
            // 主内容
        }
    }
}
```

### Scaffold 实战模板

```kotlin
// 完整页面模板
@Composable
fun MainScreen(
    onNavigateToSettings: () -> Unit,
    onFabClick: () -> Unit
) {
    val scaffoldState = rememberScaffoldState()
    val scope = rememberCoroutineScope()

    Scaffold(
        scaffoldState = scaffoldState,
        topBar = {
            TopAppBar(
                title = { Text("Main Screen") },
                actions = {
                    IconButton(onClick = onNavigateToSettings) {
                        Icon(Icons.Default.Settings, contentDescription = "Settings")
                    }
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(onClick = onFabClick) {
                Icon(Icons.Default.Add, contentDescription = "Add")
            }
        },
        snackbarHost = {
            SnackbarHost(hostState = scaffoldState.snackbarHostState)
        }
    ) { padding ->
        LazyColumn(contentPadding = padding) {
            items(50) { i ->
                Text("Item $i", modifier = Modifier.padding(16.dp))
            }
        }
    }
}

// 带底部导航的 Scaffold
@Composable
fun BottomNavScreen() {
    Scaffold(
        bottomBar = {
            BottomNavigationBar(
                items = listOf("Home", "Search", "Profile"),
                selectedIndex = 0,
                onItemSelected = { }
            )
        }
    ) { padding ->
        // 内容
    }
}

@Composable
fun BottomNavigationBar(
    items: List<String>,
    selectedIndex: Int,
    onItemSelected: (Int) -> Unit
) {
    NavigationBar {
        items.forEachIndexed { index, item ->
            NavigationBarItem(
                icon = {
                    Icon(
                        imageVector = when (index) {
                            0 -> Icons.Default.Home
                            1 -> Icons.Default.Search
                            2 -> Icons.Default.Person
                            else -> Icons.Default.Star
                        },
                        contentDescription = item
                    )
                },
                label = { Text(item) },
                selected = selectedIndex == index,
                onClick = { onItemSelected(index) }
            )
        }
    }
}
```

## 2. Navigation 导航

### NavHost / NavController 基础

```kotlin
// 添加依赖
// implementation "androidx.navigation:navigation-compose:2.7.0"

@Composable
fun NavigationApp() {
    // 创建 NavController
    val navController = rememberNavController()

    // NavHost 定义导航图
    NavHost(
        navController = navController,
        startDestination = "home"  // 起始页面
    ) {
        // 定义路由
        composable("home") {
            HomeScreen(onNavigateToDetail = { id ->
                navController.navigate("detail/$id")
            })
        }

        composable("detail/{id}") { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id")
            DetailScreen(id = id, onBack = {
                navController.popBackStack()
            })
        }

        composable("settings") {
            SettingsScreen(onBack = {
                navController.popBackStack()
            })
        }
    }
}

// 多模块导航图
@Composable
fun MultiModuleNavGraph() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "main") {
        // 主模块
        navigation(
            route = "main",
            startDestination = "home"
        ) {
            composable("home") { HomeScreen() }
            composable("profile") { ProfileScreen() }
        }

        // 登录模块
        navigation(
            route = "auth",
            startDestination = "login"
        ) {
            composable("login") { LoginScreen() }
            composable("register") { RegisterScreen() }
        }
    }
}
```

### 页面跳转与传参

```kotlin
// 方式一：路径参数
@Composable
fun NavHostWithParams() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "home") {
        composable("home") {
            HomeScreen(onNavigate = { id ->
                navController.navigate("detail/$id")
            })
        }

        composable(
            route = "detail/{userId}",
            arguments = listOf(
                navArgument("userId") {
                    type = NavType.StringType
                    nullable = false
                }
            )
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getString("userId")
            DetailScreen(userId = userId)
        }
    }
}

// 方式二：查询参数
@Composable
fun NavHostWithQueryParams() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "search") {
        composable(
            route = "search?query={query}&page={page}",
            arguments = listOf(
                navArgument("query") { defaultValue = "" },
                navArgument("page") { defaultValue = "1" }
            )
        ) { backStackEntry ->
            val query = backStackEntry.arguments?.getString("query") ?: ""
            val page = backStackEntry.arguments?.getString("page")?.toIntOrNull() ?: 1
            SearchScreen(query = query, page = page)
        }
    }
}

// 方式三：对象传参（序列化）
// 添加依赖
// implementation "androidx.navigation:navigation-compose:2.7.0"
// implementation "com.google.code.gson:gson:2.10.1"

data class User(val id: String, val name: String, val email: String)

// 创建 TypeSaver
val UserTypeSaver = Saver<User, String>(
    save = { user -> Gson().toJson(user) },
    restore = { json -> Gson().fromJson(json, User::class.java) }
)

@Composable
fun NavHostWithObject() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "userList") {
        composable("userList") {
            UserListScreen(onUserSelected = { user ->
                val userJson = Gson().toJson(user)
                navController.navigate("userDetail/$userJson")
            })
        }

        composable(
            route = "userDetail/{user}",
            arguments = listOf(
                navArgument("user") {
                    type = NavType.StringType
                }
            )
        ) { backStackEntry ->
            val userJson = backStackEntry.arguments?.getString("user")
            val user = Gson().fromJson(userJson, User::class.java)
            UserDetailScreen(user = user)
        }
    }
}

// 方式四：ViewModel 共享（推荐）
@HiltViewModel
class SharedUserViewModel @Inject constructor() : ViewModel() {
    private val _selectedUser = MutableStateFlow<User?>(null)
    val selectedUser: StateFlow<User?> = _selectedUser.asStateFlow()

    fun selectUser(user: User) {
        _selectedUser.value = user
    }
}

@Composable
fun NavHostWithViewModel() {
    val navController = rememberNavController()
    val viewModel: SharedUserViewModel = viewModel()

    NavHost(navController = navController, startDestination = "userList") {
        composable("userList") {
            UserListScreen(onUserSelected = { user ->
                viewModel.selectUser(user)
                navController.navigate("userDetail")
            })
        }

        composable("userDetail") {
            val user by viewModel.selectedUser.collectAsState()
            UserDetailScreen(user = user)
        }
    }
}
```

### 路由管理

```kotlin
// 路由常量管理（推荐）
object Routes {
    const val HOME = "home"
    const val DETAIL = "detail"
    const val SETTINGS = "settings"
    const val PROFILE = "profile"
    const val LOGIN = "login"
    const val REGISTER = "register"

    // 带参数的路由
    fun detail(userId: String) = "detail/$userId"
    fun profile(userId: String) = "profile/$userId"
    fun search(query: String, page: Int) = "search?query=$query&page=$page"
}

// 使用路由常量
@Composable
fun ManagedNavGraph() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = Routes.HOME) {
        composable(Routes.HOME) {
            HomeScreen(onNavigate = { id ->
                navController.navigate(Routes.detail(id))
            })
        }

        composable(
            route = "detail/{userId}",
            arguments = listOf(navArgument("userId") { type = NavType.StringType })
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getString("userId")
            DetailScreen(userId = userId)
        }
    }
}

// sealed class 路由（类型安全）
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Detail : Screen("detail/{userId}")
    object Settings : Screen("settings")
    object Profile : Screen("profile/{userId}")

    companion object {
        fun detail(userId: String) = "detail/$userId"
        fun profile(userId: String) = "profile/$userId"
    }
}

// 导航选项配置
@Composable
fun NavWithOptions() {
    val navController = rememberNavController()

    // 带选项的跳转
    navController.navigate("detail/123") {
        // 弹出到起始页面
        popUpTo("home") { inclusive = false }
        // 避免重复创建
        launchSingleTop = true
        // 恢复状态
        restoreState = true
    }

    // 清空返回栈
    navController.navigate("home") {
        popUpTo(0) { inclusive = true }
    }
}

// 导航监听
@Composable
fun NavWithListener() {
    val navController = rememberNavController()

    // 监听当前路由
    val currentRoute = navController.currentBackStackEntryFlow
        .collectAsState(initial = null)
        .value?.destination?.route

    LaunchedEffect(currentRoute) {
        println("Current route: $currentRoute")
    }

    // 监听返回栈
    val backStackEntry by navController.currentBackStackEntryFlow.collectAsState()
    val canGoBack = navController.previousBackStackEntry != null
}
```

### 底部导航联动

```kotlin
// 底部导航 + NavHost
@Composable
fun BottomNavWithNavigation() {
    val navController = rememberNavController()

    // 底部导航项
    val bottomNavItems = listOf(
        BottomNavItem("home", "首页", Icons.Default.Home),
        BottomNavItem("search", "搜索", Icons.Default.Search),
        BottomNavItem("profile", "我的", Icons.Default.Person)
    )

    // 当前选中项
    val navBackStackEntry by navController.currentBackStackEntryFlow.collectAsState()
    val currentRoute = navBackStackEntry?.destination?.route

    Scaffold(
        bottomBar = {
            NavigationBar {
                bottomNavItems.forEach { item ->
                    NavigationBarItem(
                        icon = { Icon(item.icon, contentDescription = item.title) },
                        label = { Text(item.title) },
                        selected = currentRoute == item.route,
                        onClick = {
                            navController.navigate(item.route) {
                                // 清除其他路由，避免返回栈堆积
                                popUpTo(navController.graph.startDestinationId) {
                                    saveState = true
                                }
                                launchSingleTop = true
                                restoreState = true
                            }
                        }
                    )
                }
            }
        }
    ) { padding ->
        NavHost(
            navController = navController,
            startDestination = "home",
            modifier = Modifier.padding(padding)
        ) {
            composable("home") { HomeScreen() }
            composable("search") { SearchScreen() }
            composable("profile") { ProfileScreen() }
        }
    }
}

data class BottomNavItem(
    val route: String,
    val title: String,
    val icon: ImageVector
)

// 带子路由的底部导航
@Composable
fun NestedBottomNav() {
    val navController = rememberNavController()

    Scaffold(
        bottomBar = {
            NavigationBar {
                // 首页（有子页面）
                NavigationBarItem(
                    icon = { Icon(Icons.Default.Home, contentDescription = "Home") },
                    label = { Text("首页") },
                    selected = false,
                    onClick = {
                        navController.navigate("home") {
                            popUpTo("home") { inclusive = true }
                        }
                    }
                )
                // 我的（有子页面）
                NavigationBarItem(
                    icon = { Icon(Icons.Default.Person, contentDescription = "Profile") },
                    label = { Text("我的") },
                    selected = false,
                    onClick = {
                        navController.navigate("profile") {
                            popUpTo("profile") { inclusive = true }
                        }
                    }
                )
            }
        }
    ) { padding ->
        NavHost(
            navController = navController,
            startDestination = "home"
        ) {
            // 首页模块
            navigation(
                route = "home",
                startDestination = "home_main"
            ) {
                composable("home_main") { HomeMainScreen() }
                composable("home_detail/{id}") { backStackEntry ->
                    val id = backStackEntry.arguments?.getString("id")
                    HomeDetailScreen(id = id)
                }
            }

            // 我的模块
            navigation(
                route = "profile",
                startDestination = "profile_main"
            ) {
                composable("profile_main") { ProfileMainScreen() }
                composable("profile_settings") { ProfileSettingsScreen() }
            }
        }
    }
}
```

### 深层链接 Deep Link

```kotlin
// 配置深层链接（AndroidManifest.xml）
/*
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- 自定义 scheme -->
        <data android:scheme="myapp" android:host="detail" />
        <!-- HTTP 链接 -->
        <data android:scheme="https" android:host="www.example.com" android:pathPrefix="/detail" />
    </intent-filter>
</activity>
*/

// Compose 中配置深层链接
@Composable
fun NavHostWithDeepLink() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = "home",
        deepLinks = listOf(
            navDeepLink {
                uriPattern = "myapp://detail/{userId}"
                action = Intent.ACTION_VIEW
            }
        )
    ) {
        composable(
            route = "home",
            deepLinks = listOf(
                navDeepLink {
                    uriPattern = "myapp://home"
                }
            )
        ) {
            HomeScreen()
        }

        composable(
            route = "detail/{userId}",
            arguments = listOf(
                navArgument("userId") { type = NavType.StringType }
            ),
            deepLinks = listOf(
                navDeepLink {
                    uriPattern = "myapp://detail/{userId}"
                    action = Intent.ACTION_VIEW
                    mimeType = "text/plain"
                },
                navDeepLink {
                    uriPattern = "https://www.example.com/detail/{userId}"
                }
            )
        ) { backStackEntry ->
            val userId = backStackEntry.arguments?.getString("userId")
            DetailScreen(userId = userId)
        }
    }
}

// 处理深层链接参数
@Composable
fun DeepLinkScreen() {
    val navController = rememberNavController()

    // 获取深层链接数据
    LaunchedEffect(Unit) {
        val intent = LocalContext.current.activity?.intent
        val data = intent?.data
        if (data != null) {
            val userId = data.lastPathSegment
            println("Deep link opened with userId: $userId")
        }
    }

    // 内容
}

// 导航图深层链接配置
@Composable
fun NavGraphDeepLink() {
    val navController = rememberNavController()

    NavHost(
        navController = navController,
        startDestination = "home"
    ) {
        composable(
            route = "home",
            deepLinks = listOf(
                navDeepLink {
                    uriPattern = "myapp://home"
                }
            )
        ) { HomeScreen() }

        composable(
            route = "detail/{userId}",
            arguments = listOf(navArgument("userId") { type = NavType.StringType }),
            deepLinks = listOf(
                navDeepLink {
                    uriPattern = "myapp://detail/{userId}"
                }
            )
        ) { backStackEntry ->
            DetailScreen(backStackEntry.arguments?.getString("userId"))
        }
    }
}
```

### 嵌套导航

```kotlin
// 嵌套导航图（子导航图）
@Composable
fun NestedNavGraph() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "main") {
        // 主导航图
        navigation(
            route = "main",
            startDestination = "home"
        ) {
            composable("home") { HomeScreen() }
            composable("detail/{id}") { backStackEntry ->
                val id = backStackEntry.arguments?.getString("id")
                DetailScreen(id = id)
            }
        }

        // 设置导航图（嵌套）
        navigation(
            route = "settings",
            startDestination = "settings_main"
        ) {
            composable("settings_main") { SettingsMainScreen() }
            composable("settings_account") { AccountSettingsScreen() }
            composable("settings_privacy") { PrivacySettingsScreen() }
        }

        // 登录导航图（嵌套）
        navigation(
            route = "auth",
            startDestination = "login"
        ) {
            composable("login") { LoginScreen() }
            composable("register") { RegisterScreen() }
            composable("forgot_password") { ForgotPasswordScreen() }
        }
    }
}

// 嵌套导航跳转
@Composable
fun NestedNavExample() {
    val navController = rememberNavController()

    // 跳转到嵌套导航图
    Button(onClick = {
        // 跳转到设置模块
        navController.navigate("settings/settings_main")
    }) {
        Text("Go to Settings")
    }

    // 从嵌套导航图返回
    Button(onClick = {
        // 返回上一级
        navController.popBackStack()
        // 或返回到指定路由
        navController.popBackStack("main", inclusive = false)
    }) {
        Text("Back to Main")
    }
}

// 条件导航（根据登录状态）
@Composable
fun ConditionalNavGraph() {
    val navController = rememberNavController()
    val isLoggedIn by remember { mutableStateOf(false) }  // 实际应从 ViewModel 获取

    NavHost(
        navController = navController,
        startDestination = if (isLoggedIn) "main" else "auth"
    ) {
        // 认证模块
        navigation(route = "auth", startDestination = "login") {
            composable("login") { LoginScreen(onLoginSuccess = {
                navController.navigate("main") {
                    popUpTo("auth") { inclusive = true }
                }
            }) }
            composable("register") { RegisterScreen() }
        }

        // 主模块
        navigation(route = "main", startDestination = "home") {
            composable("home") { HomeScreen() }
            composable("profile") { ProfileScreen() }
        }
    }
}
```

### 导航动画

```kotlin
// 添加依赖
// implementation "androidx.navigation:navigation-compose:2.7.0"
// implementation "com.google.accompanist:accompanist-navigation-animation:0.32.0"

// 使用 Accompanist 导航动画
@Composable
fun AnimatedNavGraph() {
    val navController = rememberNavController()

    AnimatedNavHost(
        navController = navController,
        startDestination = "home",
        enterTransition = {
            slideInHorizontally(initialOffsetX = { it }) + fadeIn()
        },
        exitTransition = {
            slideOutHorizontally(targetOffsetX = { -it }) + fadeOut()
        },
        popEnterTransition = {
            slideInHorizontally(initialOffsetX = { -it }) + fadeIn()
        },
        popExitTransition = {
            slideOutHorizontally(targetOffsetX = { it }) + fadeOut()
        }
    ) {
        composable("home") { HomeScreen() }
        composable("detail") { DetailScreen() }
    }
}

// 自定义动画
@Composable
fun CustomAnimatedNav() {
    val navController = rememberNavController()

    AnimatedNavHost(
        navController = navController,
        startDestination = "home",
        enterTransition = {
            when (initialState.destination.route) {
                "detail" -> slideInVertically(initialOffsetY = { it }) + fadeIn()
                else -> fadeIn()
            }
        },
        exitTransition = {
            fadeOut()
        }
    ) {
        composable("home") { HomeScreen() }
        composable("detail") { DetailScreen() }
    }
}
```

## 快速对照表

| 组件 | 用途 | 关键属性 |
|------|------|----------|
| `Scaffold` | 页面骨架 | topBar, bottomBar, FAB, content |
| `TopAppBar` | 顶部栏 | title, navigationIcon, actions |
| `BottomAppBar` | 底部栏 | actions, FAB |
| `FloatingActionButton` | 悬浮按钮 | onClick, containerColor |
| `ModalNavigationDrawer` | 侧边栏 | drawerState, drawerContent |
| `NavHost` | 导航容器 | navController, startDestination |
| `NavController` | 导航控制器 | navigate, popBackStack |
| `composable` | 定义路由 | route, arguments, deepLinks |

| 导航方法 | 用途 |
|----------|------|
| `navigate(route)` | 跳转到新页面 |
| `popBackStack()` | 返回上一页 |
| `popBackStack(route, inclusive)` | 返回到指定页面 |
| `navigate(route) { popUpTo(...) }` | 跳转并清理返回栈 |

| 传参方式 | 适用场景 |
|----------|----------|
| 路径参数 `{id}` | 简单 ID 传递 |
| 查询参数 `?key=value` | 可选参数、筛选条件 |
| ViewModel 共享 | 复杂对象、跨页面状态 |
| 序列化对象 | 需要 URL 分享的场景 |

**记忆口诀**：
- **Scaffold = 页面骨架，TopBar + BottomBar + FAB + Content**
- **NavHost = 导航容器，NavController = 导航控制器**
- **composable = 定义路由，navigate = 跳转页面**
- **路径参数用 {}，查询参数用 ?**
- **底部导航 + NavHost = popUpTo 清理返回栈**
- **深层链接 = uriPattern + AndroidManifest 配置**
- **嵌套导航 = navigation 包裹多个 composable**
