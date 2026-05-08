# JetPack Compose 列表与高性能模块

## 1. Lazy 懒加载列表

### LazyColumn 基础

```kotlin
// LazyColumn：垂直懒加载列表（类似 RecyclerView）
// 只渲染可见项，不可见项自动回收

@Composable
fun BasicLazyColumn(items: List<String>) {
    LazyColumn {
        // 单个 item
        items(items) { item ->
            Text(
                text = item,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
            )
        }
    }
}

// LazyColumn 完整参数
@Composable
fun FullLazyColumn(items: List<String>) {
    LazyColumn(
        // 内容间距
        contentPadding = PaddingValues(16.dp),
        // 垂直间距
        verticalArrangement = Arrangement.spacedBy(8.dp),
        // 水平对齐
        horizontalAlignment = Alignment.CenterHorizontally,
        // 修饰符
        modifier = Modifier.fillMaxSize(),
        // 状态（滚动位置）
        state = rememberLazyListState()
    ) {
        items(items) { item ->
            ListItem(item = item)
        }
    }
}

// 带滚动状态的 LazyColumn
@Composable
fun LazyColumnWithState(items: List<String>) {
    val listState = rememberLazyListState()

    LazyColumn(state = listState) {
        items(items) { item ->
            Text(item, modifier = Modifier.padding(16.dp))
        }
    }

    // 滚动到顶部按钮
    FloatingActionButton(
        onClick = {
            coroutineScope.launch {
                listState.animateScrollToItem(0)
            }
        }
    ) {
        Icon(Icons.Default.ArrowUpward, contentDescription = "Scroll to top")
    }
}

// 滚动控制
@Composable
fun ScrollControl(items: List<String>) {
    val listState = rememberLazyListState()
    val coroutineScope = rememberCoroutineScope()

    LazyColumn(state = listState) {
        items(items) { item ->
            Text(item, modifier = Modifier.padding(16.dp))
        }
    }

    Row {
        // 滚动到指定位置
        Button(onClick = {
            coroutineScope.launch {
                listState.scrollToItem(50)  // 立即滚动
                // listState.animateScrollToItem(50)  // 动画滚动
            }
        }) {
            Text("Go to 50")
        }

        // 滚动到底部
        Button(onClick = {
            coroutineScope.launch {
                listState.animateScrollToItem(items.size - 1)
            }
        }) {
            Text("Go to Bottom")
        }

        // 获取当前可见项
        Text("First visible: ${listState.firstVisibleItemIndex}")
    }
}
```

### LazyRow 水平列表

```kotlin
// LazyRow：水平懒加载列表
@Composable
fun BasicLazyRow(items: List<String>) {
    LazyRow(
        contentPadding = PaddingValues(horizontal = 16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            Chip(text = item)
        }
    }
}

// 带状态的 LazyRow
@Composable
fun LazyRowWithState(items: List<String>) {
    val listState = rememberLazyListState()

    LazyRow(state = listState) {
        items(items) { item ->
            Card(
                modifier = Modifier
                    .width(120.dp)
                    .height(80.dp)
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Text(item)
                }
            }
        }
    }
}

// 水平列表 + 指示器
@Composable
fun LazyRowWithIndicator(items: List<String>) {
    val listState = rememberLazyListState()

    Box {
        LazyRow(
            state = listState,
            contentPadding = PaddingValues(16.dp),
            horizontalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            items(items) { item ->
                Card(modifier = Modifier.width(150.dp)) {
                    Text(item, modifier = Modifier.padding(16.dp))
                }
            }
        }

        // 滚动指示器
        LinearProgressIndicator(
            progress = {
                val layoutInfo = listState.layoutInfo
                val totalItems = layoutInfo.totalItemsCount
                val firstVisible = layoutInfo.visibleItemsInfo.firstOrNull()?.index ?: 0
                firstVisible.toFloat() / totalItems
            },
            modifier = Modifier
                .align(Alignment.BottomCenter)
                .fillMaxWidth()
                .height(4.dp)
        )
    }
}
```

### 条目 Item 与复用

```kotlin
// 基础 Item
@Composable
fun ListItem(item: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp)
    ) {
        Text(
            text = item,
            modifier = Modifier.padding(16.dp)
        )
    }
}

// 带点击的 Item
@Composable
fun ClickableListItem(
    item: String,
    onClick: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp)
            .clickable(onClick = onClick)
    ) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text(item)
            Icon(Icons.Default.ChevronRight, contentDescription = "Navigate")
        }
    }
}

// 复杂 Item（避免过度重组）
@Composable
fun ComplexListItem(
    item: DataItem,
    onFavoriteToggle: (String) -> Unit
) {
    // 使用 remember 缓存计算结果
    val formattedDate by remember(item.timestamp) {
        derivedStateOf {
            formatDate(item.timestamp)
        }
    }

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp)
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = item.title,
                fontWeight = FontWeight.Bold
            )
            Text(
                text = item.description,
                style = MaterialTheme.typography.bodyMedium,
                color = Color.Gray
            )
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 8.dp),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Text(
                    text = formattedDate,
                    style = MaterialTheme.typography.labelSmall
                )
                IconButton(onClick = { onFavoriteToggle(item.id) }) {
                    Icon(
                        imageVector = if (item.isFavorite) Icons.Filled.Favorite else Icons.Outlined.Favorite,
                        contentDescription = "Favorite",
                        tint = if (item.isFavorite) Color.Red else Color.Gray
                    )
                }
            }
        }
    }
}

// Item 复用优化
@Composable
fun OptimizedListItem(
    item: DataItem,
    modifier: Modifier = Modifier
) {
    // 使用 key 确保状态正确复用
    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 4.dp)
    ) {
        // 稳定参数避免不必要的重组
        Text(
            text = item.title,
            modifier = Modifier.padding(16.dp)
        )
    }
}
```

## 2. 分组与粘性头部

### 分组列表

```kotlin
// 按类别分组的数据
data class GroupedItem(
    val category: String,
    val items: List<String>
)

@Composable
fun GroupedLazyColumn(groups: List<GroupedItem>) {
    LazyColumn {
        groups.forEach { group ->
            // 分组头部
            item {
                Text(
                    text = group.category,
                    style = MaterialTheme.typography.titleMedium,
                    fontWeight = FontWeight.Bold,
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp)
                        .background(Color.LightGray)
                )
            }

            // 分组内容
            items(group.items) { item ->
                Text(
                    text = item,
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp)
                )
            }

            // 分组分隔线
            item {
                Divider()
            }
        }
    }
}

// 带索引的分组
@Composable
fun IndexedGroupedList(items: List<Contact>) {
    // 按字母分组
    val grouped = items.groupBy { it.name.first().uppercase() }
    val sortedKeys = grouped.keys.sorted()

    LazyColumn {
        sortedKeys.forEach { key ->
            item {
                StickyHeader {
                    Text(
                        text = key,
                        modifier = Modifier
                            .fillMaxWidth()
                            .background(Color.White)
                            .padding(8.dp)
                    )
                }
            }

            items(grouped[key]!!) { contact ->
                ContactItem(contact = contact)
            }
        }
    }
}
```

### StickyHeader 粘性头部

```kotlin
// 粘性头部：滚动时固定在顶部
@Composable
fun StickyHeaderList(items: List<Contact>) {
    val grouped = items.groupBy { it.letter }

    LazyColumn {
        grouped.forEach { (letter, contacts) ->
            // stickyHeader：粘性头部
            stickyHeader {
                Box(
                    modifier = Modifier
                        .fillMaxWidth()
                        .background(Color.LightGray)
                        .padding(16.dp)
                ) {
                    Text(
                        text = letter,
                        fontWeight = FontWeight.Bold,
                        fontSize = 18.sp
                    )
                }
            }

            items(contacts) { contact ->
                ContactItem(contact = contact)
            }
        }
    }
}

// 自定义粘性头部样式
@Composable
fun CustomStickyHeader(groups: List<CategoryGroup>) {
    LazyColumn {
        groups.forEach { group ->
            stickyHeader(
                key = group.id,  // key 用于状态保持
                contentType = "header"  // 内容类型用于优化
            ) {
                CategoryHeader(
                    title = group.name,
                    count = group.items.size
                )
            }

            items(
                items = group.items,
                key = { it.id },  // 唯一 key
                contentType = { "item" }
            ) { item ->
                GroupedItemRow(item = item)
            }
        }
    }
}

// 粘性头部 + 滚动指示
@Composable
fun StickyHeaderWithIndicator(items: List<Contact>) {
    val listState = rememberLazyListState()
    val grouped = items.groupBy { it.letter }

    Box {
        LazyColumn(state = listState) {
            grouped.forEach { (letter, contacts) ->
                stickyHeader {
                    CurrentLetterHeader(letter = letter)
                }
                items(contacts) { contact ->
                    ContactItem(contact = contact)
                }
            }
        }

        // 侧边字母索引
        LetterIndex(
            letters = grouped.keys.sorted(),
            onLetterClick = { letter ->
                // 滚动到指定字母
                val index = grouped
                    .flatMap { it.value }
                    .indexOfFirst { it.letter == letter }
                if (index >= 0) {
                    coroutineScope.launch {
                        listState.scrollToItem(index)
                    }
                }
            }
        )
    }
}
```

## 3. 网格布局

### LazyVerticalGrid

```kotlin
// 固定列数网格
@Composable
fun FixedGrid(items: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(2),  // 2 列
        contentPadding = PaddingValues(8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            GridItem(text = item)
        }
    }
}

// 自适应列数（根据宽度自动调整）
@Composable
fun AdaptiveGrid(items: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Adaptive(minSize = 150.dp),  // 每列最小 150dp
        contentPadding = PaddingValues(8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            GridItem(text = item)
        }
    }
}

// 网格 Item
@Composable
fun GridItem(text: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .aspectRatio(1f),  // 正方形
        elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
    ) {
        Box(contentAlignment = Alignment.Center) {
            Text(
                text = text,
                fontSize = 18.sp,
                fontWeight = FontWeight.Bold
            )
        }
    }
}

// 带图片的网格
@Composable
fun ImageGrid(images: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(3),
        contentPadding = PaddingValues(4.dp),
        horizontalArrangement = Arrangement.spacedBy(4.dp),
        verticalArrangement = Arrangement.spacedBy(4.dp)
    ) {
        items(images) { imageUrl ->
            AsyncImage(
                model = imageUrl,
                contentDescription = null,
                modifier = Modifier
                    .fillMaxWidth()
                    .aspectRatio(1f)
                    .clip(RoundedCornerShape(8.dp)),
                contentScale = ContentScale.Crop
            )
        }
    }
}
```

### LazyHorizontalGrid

```kotlin
// 水平网格
@Composable
fun HorizontalGrid(items: List<String>) {
    LazyHorizontalGrid(
        rows = GridCells.Fixed(2),  // 2 行
        contentPadding = PaddingValues(8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            Card(
                modifier = Modifier
                    .width(100.dp)
                    .aspectRatio(1f)
            ) {
                Box(contentAlignment = Alignment.Center) {
                    Text(item)
                }
            }
        }
    }
}
```

### 瀑布流效果

```kotlin
// 瀑布流（不同高度 Item）
@Composable
fun WaterfallGrid(items: List<DataItem>) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(2),
        contentPadding = PaddingValues(8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .wrapContentHeight()
            ) {
                Column {
                    AsyncImage(
                        model = item.imageUrl,
                        contentDescription = null,
                        modifier = Modifier
                            .fillMaxWidth()
                            .height(IntrinsicSize.Min),
                        contentScale = ContentScale.Crop
                    )
                    Text(
                        text = item.title,
                        modifier = Modifier.padding(8.dp)
                    )
                }
            }
        }
    }
}

// 使用 Accompanist 实现真正瀑布流
// implementation "com.google.accompanist:accompanist-flowlayout:0.32.0"

@Composable
fun AccompanistWaterfall(items: List<DataItem>) {
    val columns = 2
    val columnItems = List(columns) { mutableListOf<DataItem>() }

    // 分配 Item 到各列
    items.forEachIndexed { index, item ->
        columnItems[index % columns].add(item)
    }

    Row {
        columnItems.forEach { column ->
            Column(
                modifier = Modifier
                    .weight(1f)
                    .padding(4.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp)
            ) {
                column.forEach { item ->
                    WaterfallItem(item = item)
                }
            }
        }
    }
}
```

## 4. 列表性能优化

### key 绑定

```kotlin
// ❌ 错误：没有 key，状态可能错乱
@Composable
fun BadList(items: List<User>) {
    LazyColumn {
        items(items) { user ->
            UserItem(user = user)
        }
    }
}

// ✅ 正确：使用唯一 key
@Composable
fun GoodList(items: List<User>) {
    LazyColumn {
        items(
            items = items,
            key = { user -> user.id }  // 唯一 ID
        ) { user ->
            UserItem(user = user)
        }
    }
}

// key 的好处：
// 1. Item 状态正确保持（如输入框内容、开关状态）
// 2. 删除/移动 Item 时动画正确
// 3. 避免不必要的重组

// 复杂 key 场景
@Composable
fun ComplexKeyList(items: List<DataItem>) {
    LazyColumn {
        items(
            items = items,
            key = { item -> "${item.type}-${item.id}" }  // 组合 key
        ) { item ->
            DataItemRow(item = item)
        }
    }
}

// 带 contentType 优化
@Composable
fun ContentTypeList(items: List<Any>) {
    LazyColumn {
        items(
            items = items,
            key = { item ->
                when (item) {
                    is Header -> "header-${item.id}"
                    is Content -> "content-${item.id}"
                    else -> item.hashCode()
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
                is Header -> HeaderItem(header = item)
                is Content -> ContentItem(content = item)
            }
        }
    }
}
```

### 避免重组

```kotlin
// 使用 derivedStateOf 减少重组
@Composable
fun OptimizedList(items: List<DataItem>) {
    val listState = rememberLazyListState()

    // 只有当 firstVisibleItemIndex 变化时才重组
    val firstVisibleItemIndex by remember {
        derivedStateOf { listState.firstVisibleItemIndex }
    }

    LazyColumn(state = listState) {
        items(items, key = { it.id }) { item ->
            ListItem(item = item)
        }
    }

    // 显示当前滚动位置
    Text("First visible: $firstVisibleItemIndex")
}

// 使用 remember 缓存计算
@Composable
fun CachedList(items: List<DataItem>) {
    // 缓存格式化结果
    val formattedItems = remember(items) {
        items.map { item ->
            item.copy(formattedDate = formatDate(item.timestamp))
        }
    }

    LazyColumn {
        items(formattedItems, key = { it.id }) { item ->
            Text("${item.title} - ${item.formattedDate}")
        }
    }
}

// 稳定参数避免重组
@Composable
fun StableParamList(
    items: List<DataItem>,
    onItemClick: (DataItem) -> Unit,
    onFavoriteToggle: (String) -> Unit
) {
    LazyColumn {
        items(items, key = { it.id }) { item ->
            // 传递稳定参数
            ListItem(
                item = item,
                onClick = { onItemClick(item) },
                onFavoriteToggle = { onFavoriteToggle(item.id) }
            )
        }
    }
}

// Item 使用稳定参数
@Composable
fun ListItem(
    item: DataItem,
    onClick: () -> Unit,
    onFavoriteToggle: () -> Unit,
    modifier: Modifier = Modifier
) {
    // 使用 modifier 参数而不是创建新 Modifier
    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp)
            .clickable(onClick = onClick)
    ) {
        // 内容
    }
}
```

### remember 缓存

```kotlin
// 缓存 expensive 计算
@Composable
fun ExpensiveList(items: List<String>) {
    // ❌ 错误：每次重组都重新计算
    val sortedItems = items.sortedByDescending { it.length }

    // ✅ 正确：使用 remember 缓存
    val cachedSortedItems = remember(items) {
        items.sortedByDescending { it.length }
    }

    LazyColumn {
        items(cachedSortedItems) { item ->
            Text(item)
        }
    }
}

// 缓存复杂对象创建
@Composable
fun CachedObjectList(items: List<DataItem>) {
    // 缓存颜色映射
    val colorMap = remember {
        mutableMapOf<String, Color>()
    }

    LazyColumn {
        items(items, key = { it.id }) { item ->
            val color = remember(item.category) {
                colorMap.getOrPut(item.category) {
                    getCategoryColor(item.category)
                }
            }

            Text(
                text = item.title,
                color = color
            )
        }
    }
}

// 缓存 Modifier
@Composable
fun CachedModifierList(items: List<String>) {
    // ❌ 错误：每次创建新 Modifier
    LazyColumn {
        items(items) { item ->
            Text(
                item,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
                    .background(Color.White)
            )
        }
    }

    // ✅ 正确：缓存 Modifier
    val itemModifier = remember {
        Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .background(Color.White)
    }

    LazyColumn {
        items(items) { item ->
            Text(item, modifier = itemModifier)
        }
    }
}

// 缓存回调函数
@Composable
fun CachedCallbackList(
    items: List<String>,
    onItemClick: (String) -> Unit
) {
    // ❌ 错误：每次创建新 lambda
    LazyColumn {
        items(items) { item ->
            Text(
                item,
                modifier = Modifier.clickable {
                    onItemClick(item)
                }
            )
        }
    }

    // ✅ 正确：使用 remember 缓存 lambda
    val cachedItemClick = remember(onItemClick) {
        { item: String -> onItemClick(item) }
    }

    LazyColumn {
        items(items) { item ->
            val itemClick = remember(item, cachedItemClick) {
                { cachedItemClick(item) }
            }
            Text(
                item,
                modifier = Modifier.clickable(onClick = itemClick)
            )
        }
    }
}
```

### 图片加载优化

```kotlin
// Coil 图片加载优化
@Composable
fun OptimizedImageList(images: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(3),
        contentPadding = PaddingValues(4.dp),
        horizontalArrangement = Arrangement.spacedBy(4.dp)
    ) {
        items(images) { url ->
            AsyncImage(
                model = ImageRequest.Builder(LocalContext.current)
                    .data(url)
                    .crossfade(true)
                    .size(200)  // 限制加载尺寸
                    .memoryCachePolicy(CachePolicy.ENABLED)
                    .diskCachePolicy(CachePolicy.ENABLED)
                    .build(),
                contentDescription = null,
                modifier = Modifier
                    .fillMaxWidth()
                    .aspectRatio(1f)
                    .clip(RoundedCornerShape(8.dp)),
                contentScale = ContentScale.Crop,
                placeholder = ColorPainter(Color.LightGray),
                error = ColorPainter(Color.Red)
            )
        }
    }
}

// 预加载图片
@Composable
fun PrefetchImageList(images: List<String>) {
    val listState = rememberLazyListState()
    val imageLoader = remember { ImageLoader.Builder(LocalContext.current).build() }

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
                            .memoryCachePolicy(CachePolicy.ENABLED)
                            .diskCachePolicy(CachePolicy.ENABLED)
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
```

### 分页加载

```kotlin
// 分页加载（Paging 3 + Compose）
// 添加依赖
// implementation "androidx.paging:paging-compose:3.2.0"

@Composable
fun PagingList(viewModel: MyViewModel) {
    val pagingItems = viewModel.items.collectAsLazyPagingItems()

    LazyColumn {
        items(
            count = pagingItems.itemCount,
            key = pagingItems::peekAt,
            contentType = { position ->
                when {
                    pagingItems[position] == null -> "loading"
                    else -> "item"
                }
            }
        ) { index ->
            val item = pagingItems[index]
            when {
                item == null -> {
                    // 加载中
                    LoadingItem()
                }
                else -> {
                    DataItemRow(item = item)
                }
            }
        }

        // 底部加载指示器
        pagingItems.apply {
            when {
                loadState.refresh is LoadState.Loading -> {
                    item { LoadingIndicator() }
                }
                loadState.append is LoadState.Loading -> {
                    item { LoadingIndicator() }
                }
                loadState.prepend is LoadState.Error -> {
                    item { ErrorItem(error = loadState.prepend as LoadState.Error) }
                }
                loadState.append is LoadState.Error -> {
                    item { ErrorItem(error = loadState.append as LoadState.Error) }
                }
            }
        }
    }
}

// 手动分页加载
@Composable
fun ManualPagingList(viewModel: MyViewModel) {
    val listState = rememberLazyListState()
    val items by viewModel.items.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()

    LazyColumn(state = listState) {
        items(items, key = { it.id }) { item ->
            DataItemRow(item = item)
        }

        // 底部加载指示器
        if (isLoading) {
            item {
                LoadingIndicator()
            }
        }

        // 监听滚动到底部，加载更多
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
```

## 快速对照表

| 组件 | 用途 | 关键属性 |
|------|------|----------|
| `LazyColumn` | 垂直列表 | state, contentPadding, verticalArrangement |
| `LazyRow` | 水平列表 | state, contentPadding, horizontalArrangement |
| `LazyVerticalGrid` | 垂直网格 | columns, horizontal/verticalArrangement |
| `LazyHorizontalGrid` | 水平网格 | rows, horizontal/verticalArrangement |
| `stickyHeader` | 粘性头部 | key, contentType |

| 优化技术 | 用途 |
|----------|------|
| `key = { it.id }` | 唯一 key 保持状态 |
| `contentType` | 区分 Item 类型减少重组 |
| `remember { }` | 缓存计算结果 |
| `derivedStateOf { }` | 派生状态减少重组 |
| `modifier = Modifier` | 默认参数避免创建新对象 |

| 性能指标 | 目标值 |
|----------|--------|
| 帧率 | ≥ 60fps |
| 重组次数 | 最小化 |
| 图片加载 | 限制尺寸 + 缓存 |
| 内存占用 | 懒加载 + 回收 |

**记忆口诀**：
- **LazyColumn = 垂直列表，LazyRow = 水平列表**
- **LazyVerticalGrid = 网格，columns = 列数**
- **stickyHeader = 粘性头部，滚动时固定**
- **key 绑定 = 状态正确，避免错乱**
- **remember = 缓存计算，减少重组**
- **derivedStateOf = 派生状态，按需更新**
- **contentType = 区分类型，优化复用**
- **分页加载 = Paging 3 / 手动监听滚动**
