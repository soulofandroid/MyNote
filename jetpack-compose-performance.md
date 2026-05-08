# JetPack Compose 性能优化专项

## 1. 重组优化

### 只读状态与稳定性

```kotlin
// ❌ 错误：可变对象导致不必要的重组
@Composable
fun BadExample() {
    var user by remember { mutableStateOf(User("windCloud", 25)) }

    Button(onClick = {
        user.age++  // 修改属性，但引用不变，可能不触发重组
    }) {
        Text("Age: ${user.age}")
    }
}

// ✅ 正确：不可变对象，创建新实例
@Composable
fun GoodExample() {
    var user by remember { mutableStateOf(User("windCloud", 25)) }

    Button(onClick = {
        user = user.copy(age = user.age + 1)  // 创建新实例，触发重组
    }) {
        Text("Age: ${user.age}")
    }
}

// 使用 @Stable 注解标记稳定类
@Stable
class User(val name: String, val age: Int)

// 使用 @Immutable 注解标记不可变类
@Immutable
data class Product(val id: String, val name: String, val price: Double)

// 状态类使用 data class（自动 @Immutable）
data class UiState(
    val isLoading: Boolean = false,
    val data: List<String> = emptyList(),
    val error: String? = null
)
```

### 合理抽取组件

```kotlin
// ❌ 错误：大组件，任何状态变化都导致整个组件重组
@Composable
fun BigComponent(title: String, items: List<String>, count: Int) {
    Text(title)  // title 变化时，下面的 items 也会重组

    LazyColumn {
        items(items) { item ->
            Text(item)  // 不必要重组
        }
    }

    Text("Count: $count")  // count 变化时，上面的内容也会重组
}

// ✅ 正确：拆分成小组件，只有相关部分重组
@Composable
fun BigComponent(title: String, items: List<String>, count: Int) {
    TitleSection(title = title)  // 只随 title 重组
    ItemList(items = items)       // 只随 items 重组
    CountSection(count = count)   // 只随 count 重组
}

@Composable
fun TitleSection(title: String) {
    Text(title)
}

@Composable
fun ItemList(items: List<String>) {
    LazyColumn {
        items(items) { item ->
            Text(item)
        }
    }
}

@Composable
fun CountSection(count: Int) {
    Text("Count: $count")
}

// 使用 lambda 延迟执行
@Composable
fun OptimizedComponent(
    title: String,
    expensiveContent: @Composable () -> Unit
) {
    Text(title)  // 不受 expensiveContent 影响
    expensiveContent()  // 只有 lambda 内部状态变化时才重组
}

// 使用
@Composable
fun Parent() {
    var count by remember { mutableStateOf(0) }
    var title by remember { mutableStateOf("Title") }

    OptimizedComponent(
        title = title,
        expensiveContent = {
            // 这个内容只随 count 变化重组
            Text("Count: $count")
        }
    )
}
```

### 避免无用重组

```kotlin
// ❌ 错误：每次重组都创建新对象
@Composable
fun BadModifier() {
    Text(
        text = "Hello",
        modifier = Modifier
            .padding(16.dp)
            .background(Color.White)
            .clickable { }
    )
    // 每次重组都创建新的 Modifier 对象
}

// ✅ 正确：remember 缓存 Modifier
@Composable
fun GoodModifier() {
    val modifier = remember {
        Modifier
            .padding(16.dp)
            .background(Color.White)
            .clickable { }
    }

    Text(
        text = "Hello",
        modifier = modifier
    )
}

// ❌ 错误：每次重组都创建新 lambda
@Composable
fun BadLambda(onClick: () -> Unit) {
    Button(onClick = {
        onClick()  // 每次创建新 lambda
    }) {
        Text("Click")
    }
}

// ✅ 正确：使用 remember 或传递稳定 lambda
@Composable
fun GoodLambda(onClick: () -> Unit) {
    val stableOnClick = remember(onClick) {
        { onClick() }
    }

    Button(onClick = stableOnClick) {
        Text("Click")
    }
}

// ❌ 错误：在 Composable 中进行耗时计算
@Composable
fun BadCalculation(items: List<String>) {
    // 每次重组都重新排序
    val sortedItems = items.sortedBy { it.lowercase() }

    LazyColumn {
        items(sortedItems) { item ->
            Text(item)
        }
    }
}

// ✅ 正确：使用 remember 缓存计算结果
@Composable
fun GoodCalculation(items: List<String>) {
    // 只有 items 变化时才重新计算
    val sortedItems = remember(items) {
        items.sortedBy { it.lowercase() }
    }

    LazyColumn {
        items(sortedItems) { item ->
            Text(item)
        }
    }
}

// 使用 derivedStateOf 减少重组
@Composable
fun DerivedStateExample() {
    var inputValue by remember { mutableStateOf("") }
    val listState = rememberLazyListState()

    // ❌ 错误：每次 inputValue 变化都重组
    // val firstVisibleItem = listState.firstVisibleItemIndex

    // ✅ 正确：只有 firstVisibleItemIndex 变化时才重组
    val firstVisibleItem by remember {
        derivedStateOf { listState.firstVisibleItemIndex }
    }

    Column {
        TextField(
            value = inputValue,
            onValueChange = { inputValue = it }
        )
        Text("First visible: $firstVisibleItem")
    }
}
```

### key 的正确使用

```kotlin
// ❌ 错误：没有 key，Item 状态可能错乱
@Composable
fun BadList(items: List<Item>) {
    LazyColumn {
        items(items) { item ->
            ItemRow(item = item)
        }
    }
}

// ✅ 正确：使用唯一 key
@Composable
fun GoodList(items: List<Item>) {
    LazyColumn {
        items(
            items = items,
            key = { item -> item.id }  // 唯一 ID
        ) { item ->
            ItemRow(item = item)
        }
    }
}

// key 的好处：
// 1. Item 删除/移动时动画正确
// 2. Item 内部状态（如输入框内容）保持正确
// 3. 避免不必要的重组

// 复杂场景的 key
@Composable
fun ComplexKeyList(items: List<Any>) {
    LazyColumn {
        items(
            items = items,
            key = { item ->
                when (item) {
                    is Header -> "header-${item.id}"
                    is Content -> "content-${item.id}"
                    else -> item.hashCode().toString()
                }
            },
            contentType = { item ->
                when (item) {
                    is Header -> "header"
                    is Content -> "content"
                    else -> "unknown"
                }
            }
        ) { item ->
            when (item) {
                is Header -> HeaderItem(item)
                is Content -> ContentItem(item)
            }
        }
    }
}

// 可组合函数中的 key
@Composable
fun ItemRow(item: Item) {
    // 使用 key 确保内部状态正确
    var isChecked by remember(item.id) {
        mutableStateOf(false)
    }

    Row {
        Checkbox(
            checked = isChecked,
            onCheckedChange = { isChecked = it }
        )
        Text(item.name)
    }
}
```

## 2. 图片内存优化

### Coil 图片加载优化

```kotlin
// 基础优化
@Composable
fun OptimizedImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .crossfade(true)
            .size(200)  // 限制加载尺寸，减少内存
            .memoryCachePolicy(CachePolicy.ENABLED)
            .diskCachePolicy(CachePolicy.ENABLED)
            .build(),
        contentDescription = null,
        modifier = Modifier.size(200.dp),
        contentScale = ContentScale.Crop,
        placeholder = ColorPainter(Color.LightGray),
        error = ColorPainter(Color.Red)
    )
}

// 列表中的图片优化
@Composable
fun ImageList(images: List<String>) {
    LazyVerticalGrid(columns = GridCells.Fixed(3)) {
        items(images) { url ->
            AsyncImage(
                model = ImageRequest.Builder(LocalContext.current)
                    .data(url)
                    .size(150)  // 网格图片更小
                    .crossfade(true)
                    .build(),
                contentDescription = null,
                modifier = Modifier
                    .aspectRatio(1f)
                    .clip(RoundedCornerShape(8.dp)),
                contentScale = ContentScale.Crop
            )
        }
    }
}

// 预加载图片
@Composable
fun PrefetchImageList(images: List<String>) {
    val listState = rememberLazyListState()
    val imageLoader = remember {
        ImageLoader.Builder(LocalContext.current)
            .memoryCache {
                MemoryCache.Builder(LocalContext.current)
                    .maxSizePercent(0.25)  // 使用 25% 内存
                    .build()
            }
            .diskCache {
                DiskCache.Builder()
                    .directory(LocalContext.current.cacheDir.resolve("image_cache"))
                    .maxSizePercent(0.02)  // 使用 2% 存储
                    .build()
            }
            .build()
    }

    // 监听滚动，预加载即将可见的图片
    LaunchedEffect(listState.isScrollInProgress) {
        snapshotFlow { listState.firstVisibleItemIndex }
            .collect { firstVisible ->
                // 预加载后面 5 张图片
                val prefetchRange = firstVisible until (firstVisible + 5).coerceAtMost(images.size)
                prefetchRange.forEach { index ->
                    imageLoader.enqueue(
                        ImageRequest.Builder(LocalContext.current)
                            .data(images[index])
                            .build()
                    )
                }
            }
    }

    LazyColumn(state = listState) {
        items(images) { url ->
            AsyncImage(
                model = url,
                contentDescription = null,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(200.dp),
                imageLoader = imageLoader
            )
        }
    }
}

// 大图加载优化
@Composable
fun LargeImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .decoderFactory { result, options, _ ->
                // 使用 subsampling 加载大图
                SubsamplingDecoder(result, options)
            }
            .build(),
        contentDescription = null,
        modifier = Modifier.fillMaxSize(),
        contentScale = ContentScale.Fit
    )
}
```

### 图片缓存配置

```kotlin
// 全局 ImageLoader 配置
@Singleton
class ImageLoaderProvider @Inject constructor(
    @ApplicationContext private val context: Context
) {
    fun provideImageLoader(): ImageLoader {
        return ImageLoader.Builder(context)
            .memoryCache {
                MemoryCache.Builder(context)
                    .maxSizePercent(0.25)  // 25% 可用内存
                    .strongReferencesEnabled(true)
                    .build()
            }
            .diskCache {
                DiskCache.Builder()
                    .directory(context.cacheDir.resolve("image_cache"))
                    .maxSizeBytes(100 * 1024 * 1024)  // 100MB
                    .build()
            }
            .respectCacheHeaders(false)
            .availableMemoryMultiplier(0.5)
            .build()
    }
}

// 在 Application 中设置
class MyApplication : Application() {
    @Inject lateinit var imageLoaderProvider: ImageLoaderProvider

    override fun onCreate() {
        super.onCreate()
        Coil.setImageLoader(imageLoaderProvider.provideImageLoader())
    }
}
```

## 3. Lazy 列表优化

### 基础优化

```kotlin
// ✅ 使用 key 避免不必要的重组
@Composable
fun OptimizedLazyList(items: List<Item>) {
    LazyColumn {
        items(
            items = items,
            key = { item -> item.id }  // 唯一 key
        ) { item ->
            ItemRow(item = item)
        }
    }
}

// ✅ 使用 contentType 优化不同类型的 Item
@Composable
fun TypedLazyList(items: List<Any>) {
    LazyColumn {
        items(
            items = items,
            key = { item ->
                when (item) {
                    is Header -> item.id
                    is Content -> item.id
                }
            },
            contentType = { item ->
                when (item) {
                    is Header -> "header"
                    is Content -> "content"
                }
            }
        ) { item ->
            when (item) {
                is Header -> HeaderItem(item)
                is Content -> ContentItem(item)
            }
        }
    }
}

// ✅ 使用 item 和 items 的正确组合
@Composable
fun MixedLazyList(header: String, items: List<String>, footer: String) {
    LazyColumn {
        // 固定项使用 item
        item {
            HeaderView(header)
        }

        // 列表项使用 items
        items(items) { item ->
            ItemView(item)
        }

        // 固定项使用 item
        item {
            FooterView(footer)
        }
    }
}
```

### 高级优化

```kotlin
// 懒加载 Item 内容
@Composable
fun LazyLoadItem(item: Item) {
    var isLoaded by remember { mutableStateOf(false) }

    Box {
        if (isLoaded) {
            ItemContent(item = item)
        } else {
            // 占位内容
            Box(
                modifier = Modifier
                    .fillMaxWidth()
                    .height(100.dp)
                    .background(Color.LightGray)
            )
        }
    }

    // 当 Item 可见时加载内容
    LaunchedEffect(Unit) {
        delay(100)  // 模拟加载
        isLoaded = true
    }
}

// 分页加载优化
@Composable
fun PagedLazyList(viewModel: ArticleViewModel) {
    val listState = rememberLazyListState()
    val articles by viewModel.articles.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()

    LazyColumn(state = listState) {
        items(
            items = articles,
            key = { it.id }
        ) { article ->
            ArticleItem(article = article)
        }

        // 底部加载指示器
        if (isLoading) {
            item {
                LoadingIndicator()
            }
        }

        // 监听滚动到底部
        item {
            LaunchedEffect(listState.isScrollInProgress) {
                snapshotFlow {
                    val layoutInfo = listState.layoutInfo
                    val visibleItems = layoutInfo.visibleItemsInfo
                    val lastVisibleItem = visibleItems.lastOrNull()
                    lastVisibleItem?.index == layoutInfo.totalItemsCount - 1
                }
                .filter { it }
                .collect {
                    viewModel.loadMore()
                }
            }
        }
    }
}

// 避免在 LazyColumn 中嵌套 LazyColumn
@Composable
fun BadNestedLazy() {
    LazyColumn {
        items(categories) { category ->
            Text(category.name)
            // ❌ 错误：嵌套 LazyColumn
            LazyColumn {
                items(category.items) { item ->
                    Text(item.name)
                }
            }
        }
    }
}

// ✅ 正确：使用单个 LazyColumn
@Composable
fun GoodFlatLazy() {
    LazyColumn {
        categories.forEach { category ->
            item {
                Text(category.name)
            }
            items(category.items) { item ->
                Text(item.name)
            }
        }
    }
}
```

## 4. 内存泄漏排查

### 常见泄漏场景

```kotlin
// ❌ 错误：CoroutineScope 泄漏
@Composable
fun LeakyComposable() {
    // 使用 CoroutineScope 而不是 rememberCoroutineScope
    val scope = CoroutineScope(Dispatchers.Main)

    Button(onClick = {
        scope.launch {
            // 长时间运行的任务
            delay(10000)
            // Composable 已经销毁，但协程还在运行
        }
    }) {
        Text("Click")
    }
}

// ✅ 正确：使用 rememberCoroutineScope
@Composable
fun SafeComposable() {
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch {
            delay(10000)
            // Composable 销毁时，scope 自动取消
        }
    }) {
        Text("Click")
    }
}

// ❌ 错误：LaunchedEffect 没有正确 key
@Composable
fun LeakyEffect(userId: String) {
    // key 是 Unit，不会随 userId 变化重新执行
    LaunchedEffect(Unit) {
        // 加载用户数据
        loadUser(userId)
    }
}

// ✅ 正确：使用正确的 key
@Composable
fun SafeEffect(userId: String) {
    // userId 变化时，重新执行
    LaunchedEffect(userId) {
        loadUser(userId)
    }
}

// ❌ 错误：DisposableEffect 没有清理
@Composable
fun LeakyDisposable() {
    val listener = remember { LocationListener { } }

    DisposableEffect(Unit) {
        // 注册监听器
        locationManager.requestLocationUpdates(listener)
        // ❌ 没有 onDispose 清理
    }
}

// ✅ 正确：DisposableEffect 清理资源
@Composable
fun SafeDisposable() {
    val listener = remember { LocationListener { } }

    DisposableEffect(Unit) {
        locationManager.requestLocationUpdates(listener)

        // 离开时清理
        onDispose {
            locationManager.removeUpdates(listener)
        }
    }
}

// ❌ 错误：Flow 收集没有生命周期感知
@Composable
fun LeakyFlow(viewModel: MyViewModel) {
    // 没有使用 collectAsState
    LaunchedEffect(Unit) {
        viewModel.dataFlow.collect { data ->
            // Composable 销毁后仍在收集
        }
    }
}

// ✅ 正确：使用 collectAsState
@Composable
fun SafeFlow(viewModel: MyViewModel) {
    val data by viewModel.dataFlow.collectAsState()
    // 自动在 Composable 离开时取消
}
```

### 泄漏检测工具

```kotlin
// 使用 LeakCanary 检测泄漏
// 添加依赖
// debugImplementation "com.squareup.leakcanary:leakcanary-android:2.12"

// Compose 重组检测
@Composable
fun RecompositionCounter() {
    var recompositionCount by remember { mutableStateOf(0) }
    var contentCount by remember { mutableStateOf(0) }

    // 每次重组时增加计数
    recompositionCount++
    contentCount++

    Column {
        Text("Recompositions: $recompositionCount")
        Text("Content executions: $contentCount")
    }
}

// 使用 LayoutInspector 检测重组
// Android Studio → Layout Inspector → 选择 Compose 应用

// 使用 RecompositionLogger
object RecompositionLogger {
    private val recompositionCounts = mutableMapOf<String, Int>()

    fun log(composableName: String) {
        val count = recompositionCounts.getOrPut(composableName) { 0 } + 1
        recompositionCounts[composableName] = count
        Log.d("Recomposition", "$composableName: $count")
    }
}

// 在 Composable 中使用
@Composable
fun MyComposable() {
    RecompositionLogger.log("MyComposable")
    // 内容
}
```

## 5. Compose 最佳实践

### 性能检查清单

```kotlin
// ✅ 稳定性检查
// 1. 使用 @Stable 或 @Immutable 标记类
@Stable
class StableClass(val value: Int)

@Immutable
data class ImmutableData(val id: String, val name: String)

// 2. 避免在 Composable 中创建对象
@Composable
fun GoodComposable() {
    val modifier = remember { Modifier.padding(16.dp) }
    val color = remember { Color.Blue }

    Text("Hello", modifier = modifier, color = color)
}

// 3. 使用 derivedStateOf 减少重组
@Composable
fun DerivedExample() {
    var inputValue by remember { mutableStateOf("") }
    val listState = rememberLazyListState()

    val firstVisibleIndex by remember {
        derivedStateOf { listState.firstVisibleItemIndex }
    }

    Text("First visible: $firstVisibleIndex")
}

// 4. 合理抽取组件
@Composable
fun Parent(title: String, count: Int) {
    Title(title = title)  // 只随 title 重组
    Count(count = count)  // 只随 count 重组
}

// 5. 使用 key 保持状态
@Composable
fun ItemList(items: List<Item>) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemRow(item)
        }
    }
}
```

### 性能监控

```kotlin
// 重组监控
@Composable
fun MonitorRecomposition(name: String) {
    var executions by remember { mutableStateOf(0) }
    var disposals by remember { mutableStateOf(0) }

    executions++

    DisposableEffect(Unit) {
        onDispose {
            disposals++
            Log.d("Monitor", "$name - Executions: $executions, Disposals: $disposals")
        }
    }

    // 内容
}

// 性能统计
object PerformanceStats {
    private val recompositionStats = mutableMapOf<String, Int>()

    fun recordRecomposition(composableName: String) {
        recompositionStats[composableName] =
            (recompositionStats[composableName] ?: 0) + 1
    }

    fun printStats() {
        recompositionStats.forEach { (name, count) ->
            Log.d("Performance", "$name: $count recompositions")
        }
    }
}

// 在 Composable 中使用
@Composable
fun TrackedComposable() {
    PerformanceStats.recordRecomposition("TrackedComposable")
    // 内容
}
```

### 最佳实践总结

```kotlin
/*
JetPack Compose 性能优化最佳实践

1. 状态管理
   - 使用不可变对象（data class）
   - 使用 @Stable/@Immutable 注解
   - 状态提升，子组件无状态
   - 使用 derivedStateOf 减少重组

2. 组件设计
   - 小组件优于大组件
   - 合理抽取，按状态依赖拆分
   - 使用 lambda 延迟执行

3. 对象创建
   - remember 缓存 Modifier、颜色、lambda
   - 避免在 Composable 中创建对象
   - 使用 remember { } 缓存计算结果

4. 列表优化
   - 使用 key 保持状态
   - 使用 contentType 优化不同类型
   - 避免嵌套 LazyColumn
   - 图片限制尺寸

5. 协程与 Flow
   - 使用 rememberCoroutineScope
   - LaunchedEffect 使用正确 key
   - DisposableEffect 清理资源
   - 使用 collectAsState 收集 Flow

6. 图片加载
   - 限制图片尺寸
   - 启用缓存
   - 预加载可见图片
   - 使用合适的 ImageLoader 配置

7. 内存管理
   - 避免长生命周期引用短生命周期
   - 及时清理监听器
   - 使用 LeakCanary 检测泄漏
   - 监控重组次数
*/
```

## 快速对照表

| 优化项 | 技术 | 效果 |
|--------|------|------|
| 状态稳定 | `@Stable` / `@Immutable` | 减少不必要重组 |
| 对象缓存 | `remember { }` | 避免重复创建 |
| 派生状态 | `derivedStateOf { }` | 按需重组 |
| 列表 key | `key = { it.id }` | 状态保持正确 |
| 列表类型 | `contentType` | 复用不同类型的 Item |
| 图片尺寸 | `.size(200)` | 减少内存占用 |
| 图片缓存 | `MemoryCache` / `DiskCache` | 减少网络请求 |
| 协程作用域 | `rememberCoroutineScope` | 自动取消 |
| 资源清理 | `DisposableEffect` + `onDispose` | 避免泄漏 |
| Flow 收集 | `collectAsState` | 生命周期感知 |

| 检测工具 | 用途 |
|----------|------|
| Layout Inspector | 查看重组 |
| LeakCanary | 内存泄漏检测 |
| RecompositionLogger | 重组计数 |
| Android Profiler | 内存/CPU 监控 |

**记忆口诀**：
- **状态稳定 = @Stable / @Immutable**
- **对象缓存 = remember { }**
- **派生状态 = derivedStateOf**
- **列表优化 = key + contentType**
- **图片优化 = 限制尺寸 + 缓存**
- **协程安全 = rememberCoroutineScope**
- **资源清理 = DisposableEffect + onDispose**
- **Flow 收集 = collectAsState**
- **泄漏检测 = LeakCanary**
