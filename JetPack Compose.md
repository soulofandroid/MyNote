# JetPack Compose 基础核心模块

## 1. 核心概念

### @Composable 注解

```kotlin
// @Composable：标记函数为可组合函数，可生成 UI
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}

// 可组合函数可调用其他可组合函数
@Composable
fun UserProfile(user: User) {
    Text(text = user.name)
    Text(text = user.email)
}

// 可组合函数规则
// ✅ 允许：
// - 调用其他 @Composable 函数
// - 使用 Compose UI 组件（Text、Button 等）
// - 使用 State 读取状态

// ❌ 禁止：
// - 在非 @Composable 函数中调用 @Composable 函数
// - 直接修改 State（应通过事件回调）
// - 在 @Composable 中执行副作用（需用 LaunchedEffect 等）
```

### 重组（Recomposition）原理

```kotlin
// 重组：当 State 变化时，Compose 重新执行受影响的 @Composable 函数
// 触发重组的条件：
// 1. State 值变化（remember { mutableStateOf() }）
// 2. 参数变化
// 3. 强制重组（如 key 变化）

@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }  // State 变化触发重组

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

// 智能重组：Compose 跳过未变化的组件
@Composable
fun SmartList(items: List<String>) {
    items.forEach { item ->
        // 只有 item 变化时才重组该 Item
        ListItem(item = item)
    }
}

// 优化重组：使用 derivedStateOf 减少不必要的重组
@Composable
fun OptimizedCounter() {
    var count by remember { mutableStateOf(0) }
    // 只有当 count > 10 时才触发重组
    val isLarge by remember { derivedStateOf { count > 10 } }

    Text(if (isLarge) "Large" else "Normal")
}
```

### 函数式 UI vs 传统 XML

| 特性 | Compose（函数式） | XML（命令式） |
|------|------------------|---------------|
| **代码位置** | Kotlin 代码 | XML 布局文件 |
| **UI 更新** | 自动重组 | 手动 findViewById + setText |
| **状态管理** | State 驱动 UI | 手动同步状态与 UI |
| **代码量** | 少（约 1/3） | 多（重复代码） |
| **预览** | @Preview 实时预览 | 需运行 App 查看 |
| **学习曲线** | 需理解函数式思想 | 熟悉但繁琐 |

```kotlin
// Compose：声明式
@Composable
fun Greeting(name: String) {
    Text("Hello, $name!")  // UI 直接由数据决定
}

// XML：命令式
// Java/Kotlin 代码
textView.text = "Hello, $name!"  // 手动更新 UI
```

## 2. 基础常用布局

### Column / Row / Box

```kotlin
// Column：垂直排列
@Composable
fun ColumnExample() {
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp),  // 间距
        horizontalAlignment = Alignment.CenterHorizontally   // 水平对齐
    ) {
        Text("Item 1")
        Text("Item 2")
        Text("Item 3")
    }
}

// Row：水平排列
@Composable
fun RowExample() {
    Row(
        modifier = Modifier.padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,  // 两端对齐
        verticalAlignment = Alignment.CenterVertically       // 垂直居中
    ) {
        Text("Left")
        Text("Center")
        Text("Right")
    }
}

// Box：堆叠布局（类似 FrameLayout）
@Composable
fun BoxExample() {
    Box(
        modifier = Modifier.size(200.dp),
        contentAlignment = Alignment.Center  // 内容居中
    ) {
        // 底层
        Box(
            modifier = Modifier
                .size(150.dp)
                .background(Color.Blue)
        )
        // 中层
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(Color.Green)
        )
        // 顶层
        Text("Overlay", color = Color.White)
    }
}

// Arrangement 常用值
// - Start / End / Top / Bottom
// - Center
// - SpaceBetween（两端对齐）
// - SpaceAround（周围间距）
// - SpaceEvenly（均匀间距）
// - spacedBy(8.dp)（固定间距）

// Alignment 常用值
// - Start / End / Top / Bottom
// - Center
// - TopStart / TopEnd / BottomStart / BottomEnd
```

### ConstraintLayout

```kotlin
// 添加依赖
// implementation "androidx.constraintlayout:constraintlayout-compose:1.0.1"

@Composable
fun ConstraintLayoutExample() {
    ConstraintLayout(
        modifier = Modifier.fillMaxSize()
    ) {
        // 创建引用
        val (title, subtitle, button) = createRefs()

        // 标题：顶部居中
        Text(
            text = "Title",
            modifier = Modifier.constrainAs(title) {
                top.linkTo(parent.top, margin = 16.dp)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
            }
        )

        // 副标题：标题下方
        Text(
            text = "Subtitle",
            modifier = Modifier.constrainAs(subtitle) {
                top.linkTo(title.bottom, margin = 8.dp)
                start.linkTo(title.start)
                end.linkTo(title.end)
            }
        )

        // 按钮：底部居中
        Button(
            onClick = { },
            modifier = Modifier.constrainAs(button) {
                bottom.linkTo(parent.bottom, margin = 32.dp)
                start.linkTo(parent.start)
                end.linkTo(parent.end)
            }
        )
    }
}

// 链式约束
@Composable
fun ChainExample() {
    ConstraintLayout {
        val (text1, text2, text3) = createRefs()

        // 创建水平链
        createHorizontalChain(text1, text2, text3)

        Text("1", Modifier.constrainAs(text1) {
            top.linkTo(parent.top)
        })
        Text("2", Modifier.constrainAs(text2) {
            top.linkTo(parent.top)
        })
        Text("3", Modifier.constrainAs(text3) {
            top.linkTo(parent.top)
        })
    }
}
```

### Spacer / Weight

```kotlin
// Spacer：占位空白
@Composable
fun SpacerExample() {
    Column {
        Text("Top")
        Spacer(modifier = Modifier.height(16.dp))  // 垂直间距
        Text("Bottom")
    }
}

// Weight：权重分配（类似 LinearLayout weight）
@Composable
fun WeightExample() {
    Row(modifier = Modifier.fillMaxWidth()) {
        // 占 30% 宽度
        Text(
            text = "30%",
            modifier = Modifier
                .weight(0.3f)
                .background(Color.Red)
        )
        // 占 70% 宽度
        Text(
            text = "70%",
            modifier = Modifier
                .weight(0.7f)
                .background(Color.Blue)
        )
    }
}

// Weight + Spacer 组合
@Composable
fun CenteredContent() {
    Row(modifier = Modifier.fillMaxWidth()) {
        // 左侧弹性空白
        Spacer(modifier = Modifier.weight(1f))

        // 居中内容
        Text("Centered")

        // 右侧弹性空白
        Spacer(modifier = Modifier.weight(1f))
    }
}
```

### 滚动布局

```kotlin
// Column + verticalScroll
@Composable
fun ScrollableColumn() {
    Column(
        modifier = Modifier
            .verticalScroll(rememberScrollState())
            .padding(16.dp)
    ) {
        repeat(50) { i ->
            Text("Item $i", modifier = Modifier.padding(vertical = 4.dp))
        }
    }
}

// Row + horizontalScroll
@Composable
fun ScrollableRow() {
    Row(
        modifier = Modifier
            .horizontalScroll(rememberScrollState())
            .padding(16.dp)
    ) {
        repeat(20) { i ->
            Button(
                onClick = { },
                modifier = Modifier.padding(end = 8.dp)
            ) {
                Text("Btn $i")
            }
        }
    }
}

// LazyColumn：高性能列表（类似 RecyclerView）
@Composable
fun LazyListExample(items: List<String>) {
    LazyColumn(
        contentPadding = PaddingValues(16.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            Text(item)
        }

        // 固定项
        item {
            Text("Footer", modifier = Modifier.padding(top = 16.dp))
        }

        // 批量项
        items(100) { i ->
            Text("Generated $i")
        }
    }
}

// LazyRow：水平列表
@Composable
fun LazyRowExample() {
    LazyRow(
        contentPadding = PaddingValues(horizontal = 16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(50) { i ->
            Chip("Item $i")
        }
    }
}

// LazyVerticalGrid：网格
@Composable
fun GridExample(items: List<String>) {
    LazyVerticalGrid(
        columns = GridCells.Fixed(2),  // 2 列
        contentPadding = PaddingValues(16.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(items) { item ->
            Card {
                Text(item, modifier = Modifier.padding(16.dp))
            }
        }
    }
}
```

## 3. 基础常用控件

### Text

```kotlin
@Composable
fun TextExamples() {
    // 基础文本
    Text(text = "Hello Compose")

    // 样式
    Text(
        text = "Styled Text",
        fontSize = 20.sp,
        fontWeight = FontWeight.Bold,
        fontStyle = FontStyle.Italic,
        color = Color.Blue,
        textAlign = TextAlign.Center,
        lineHeight = 28.sp,  // 行高
        letterSpacing = 2.sp  // 字间距
    )

    // 最大行数 + 省略号
    Text(
        text = "Long text that should be truncated after three lines...",
        maxLines = 3,
        overflow = TextOverflow.Ellipsis
    )

    // 自定义字体
    Text(
        text = "Custom Font",
        fontFamily = FontFamily(
            Font(R.font.roboto_regular),
            Font(R.font.roboto_bold, FontWeight.Bold)
        )
    )

    // 带样式的部分文本
    Text(
        buildAnnotatedString {
            append("Hello ")
            withStyle(style = SpanStyle(color = Color.Red)) {
                append("Red")
            }
            append(" World")
        }
    )
}
```

### Button

```kotlin
@Composable
fun ButtonExamples() {
    // 普通按钮
    Button(
        onClick = { /* 点击事件 */ },
        modifier = Modifier
            .fillMaxWidth()
            .height(48.dp),
        enabled = true,  // 是否可用
        elevation = ButtonDefaults.buttonElevation(defaultElevation = 4.dp)
    ) {
        Icon(Icons.Default.Add, contentDescription = null)
        Spacer(Modifier.width(8.dp))
        Text("Button")
    }

    // 边框按钮
    OutlinedButton(
        onClick = { },
        modifier = Modifier.fillMaxWidth()
    ) {
        Text("Outlined Button")
    }

    // 纯文本按钮
    TextButton(
        onClick = { },
        modifier = Modifier.fillMaxWidth()
    ) {
        Text("Text Button")
    }

    // 自定义颜色
    Button(
        onClick = { },
        colors = ButtonDefaults.buttonColors(
            containerColor = Color.Green,
            contentColor = Color.White,
            disabledContainerColor = Color.Gray,
            disabledContentColor = Color.DarkGray
        )
    ) {
        Text("Custom Color")
    }

    // 无涟漪效果
    Button(
        onClick = { },
        interactionSource = remember { MutableInteractionSource() }
    ) {
        Text("No Ripple")
    }
}
```

### Image

```kotlin
@Composable
fun ImageExamples() {
    // 本地资源图片
    Image(
        painter = painterResource(id = R.drawable.logo),
        contentDescription = "Logo",
        modifier = Modifier.size(100.dp)
    )

    // 缩放类型
    Image(
        painter = painterResource(id = R.drawable.photo),
        contentDescription = "Photo",
        modifier = Modifier.size(200.dp),
        contentScale = ContentScale.Crop  // 裁剪填充
        // ContentScale.Fit       // 适应（保持比例）
        // ContentScale.FillWidth // 填充宽度
        // ContentScale.FillHeight // 填充高度
        // ContentScale.Inside    // 内部适应
        // ContentScale.None      // 不缩放
    )

    // 圆形图片
    Image(
        painter = painterResource(id = R.drawable.avatar),
        contentDescription = "Avatar",
        modifier = Modifier
            .size(100.dp)
            .clip(CircleShape)  // 圆形裁剪
    )

    // 颜色滤镜
    Image(
        painter = painterResource(id = R.drawable.icon),
        contentDescription = "Icon",
        colorFilter = ColorFilter.tint(Color.Blue)
    )

    // 透明度
    Image(
        painter = painterResource(id = R.drawable.bg),
        contentDescription = "Background",
        alpha = 0.5f  // 50% 透明度
    )
}

// 网络图片（Coil）
// implementation "io.coil-kt:coil-compose:2.5.0"

@Composable
fun NetworkImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .crossfade(true)
            .build(),
        contentDescription = "Network Image",
        modifier = Modifier.size(200.dp),
        placeholder = painterResource(R.drawable.placeholder),
        error = painterResource(R.drawable.error),
        contentScale = ContentScale.Crop
    )
}
```

### Icon / IconButton

```kotlin
@Composable
fun IconExamples() {
    // 内置图标
    Icon(
        imageVector = Icons.Default.Home,
        contentDescription = "Home",
        tint = Color.Blue,
        modifier = Modifier.size(24.dp)
    )

    // 自定义图标
    Icon(
        painter = painterResource(id = R.drawable.ic_custom),
        contentDescription = "Custom Icon",
        tint = Color.Unspecified  // 保持原色
    )

    // 图标按钮
    IconButton(
        onClick = { },
        modifier = Modifier.size(48.dp)
    ) {
        Icon(
            imageVector = Icons.Default.Favorite,
            contentDescription = "Favorite"
        )
    }

    // 切换图标按钮
    var isFavorite by remember { mutableStateOf(false) }
    IconButton(onClick = { isFavorite = !isFavorite }) {
        Icon(
            imageVector = if (isFavorite) Icons.Filled.Favorite else Icons.Outlined.Favorite,
            contentDescription = "Favorite",
            tint = if (isFavorite) Color.Red else Color.Gray
        )
    }

    // 带背景的图标按钮
    FilledIconButton(onClick = { }) {
        Icon(Icons.Default.Add, contentDescription = "Add")
    }

    OutlinedIconButton(onClick = { }) {
        Icon(Icons.Default.Search, contentDescription = "Search")
    }
}
```

### Divider

```kotlin
@Composable
fun DividerExamples() {
    Column {
        // 默认分割线
        Divider()

        // 自定义颜色
        Divider(color = Color.Red)

        // 自定义厚度
        Divider(thickness = 2.dp)

        // 垂直分割线
        Row(verticalAlignment = Alignment.CenterVertically) {
            Text("Item 1")
            Divider(
                modifier = Modifier
                    .height(24.dp)
                    .padding(horizontal = 8.dp),
                thickness = 1.dp
            )
            Text("Item 2")
        }

        // 带缩进的分割线
        Divider(
            modifier = Modifier.padding(start = 16.dp),
            thickness = 1.dp
        )
    }
}
```

## 4. 修饰符 Modifier（重中之重）

### Modifier 链式调用顺序

```kotlin
// ⚠️ 顺序很重要！Modifier 从上到下依次应用

// ✅ 正确：先 padding 后 background
Text(
    text = "Correct",
    modifier = Modifier
        .padding(16.dp)      // 1. 内边距
        .background(Color.Blue) // 2. 背景（包含 padding 区域）
)

// ❌ 错误：先 background 后 padding
Text(
    text = "Wrong",
    modifier = Modifier
        .background(Color.Blue) // 1. 背景
        .padding(16.dp)         // 2. 内边距（背景被裁掉）
)

// ✅ 推荐顺序：
// 1. size / width / height
// 2. padding
// 3. background / border
// 4. clip
// 5. clickable / interaction
// 6. align / offset

@Composable
fun CorrectModifierOrder() {
    Box(
        modifier = Modifier
            .size(200.dp)           // 尺寸
            .padding(16.dp)         // 内边距
            .background(Color.Blue) // 背景
            .clip(RoundedCornerShape(8.dp)) // 圆角
            .clickable { }          // 点击
    ) {
        Text("Content")
    }
}
```

### 常用修饰符

```kotlin
// size / 尺寸
Modifier.size(100.dp)              // 宽高
Modifier.width(200.dp)             // 宽度
Modifier.height(100.dp)            // 高度
Modifier.fillMaxSize()             // 填满父容器
Modifier.fillMaxWidth()            // 填满宽度
Modifier.fillMaxHeight()           // 填满高度
Modifier.sizeIn(minWidth = 50.dp, maxWidth = 200.dp)  // 尺寸范围

// padding / margin
Modifier.padding(16.dp)            // 内边距
Modifier.padding(horizontal = 16.dp, vertical = 8.dp)
Modifier.padding(start = 16.dp, end = 8.dp, top = 4.dp, bottom = 4.dp)
// ⚠️ Compose 没有 margin，用 padding 实现

// background / 背景
Modifier.background(Color.Blue)
Modifier.background(Color.Blue, RoundedCornerShape(8.dp))  // 圆角背景
Modifier.background(Brush.horizontalGradient(listOf(Color.Red, Color.Blue)))

// clickable / 点击
Modifier.clickable { /* 点击处理 */ }
Modifier.clickable(
    enabled = true,
    onClickLabel = "Click me",
    role = Role.Button,
    onClick = { }
)
Modifier.combinedClickable(
    onClick = { },
    onLongClick = { },
    onDoubleClick = { }
)

// border / 边框
Modifier.border(1.dp, Color.Gray)
Modifier.border(2.dp, Color.Blue, RoundedCornerShape(8.dp))

// clip / 裁剪
Modifier.clip(RoundedCornerShape(8.dp))
Modifier.clip(CircleShape)
Modifier.clip(RoundedCornerShape(topStart = 16.dp, topEnd = 16.dp))

// align / 对齐（在 Box/Row/Column 中）
Modifier.align(Alignment.Center)
Modifier.align(Alignment.TopStart)
Modifier.wrapContentSize(Alignment.Center)

// offset / 偏移
Modifier.offset(x = 10.dp, y = 10.dp)
Modifier.offset { IntOffset(10, 10) }

// rotate / 旋转
Modifier.rotate(45.degrees)
Modifier.rotate(90.degrees)

// alpha / 透明度
Modifier.alpha(0.5f)  // 50% 透明

// drawBehind / 自定义绘制
Modifier.drawBehind {
    drawCircle(Color.Red, radius = size.minDimension / 2)
}
```

### Modifier 实战组合

```kotlin
// 卡片样式
@Composable
fun CardItem(text: String) {
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
            .background(
                color = Color.White,
                shape = RoundedCornerShape(12.dp)
            )
            .border(
                width = 1.dp,
                color = Color.LightGray,
                shape = RoundedCornerShape(12.dp)
            )
            .clip(RoundedCornerShape(12.dp))
            .clickable { }
            .padding(16.dp)
    ) {
        Text(text)
    }
}

// 圆形头像
@Composable
fun Avatar(url: String, modifier: Modifier = Modifier) {
    AsyncImage(
        model = url,
        contentDescription = "Avatar",
        modifier = modifier
            .size(64.dp)
            .clip(CircleShape)
            .border(2.dp, Color.White, CircleShape)
    )
}

// 带角标的图标
@Composable
fun BadgeIcon(count: Int) {
    Box {
        Icon(
            imageVector = Icons.Default.ShoppingCart,
            contentDescription = "Cart"
        )
        if (count > 0) {
            Text(
                text = count.toString(),
                fontSize = 10.sp,
                color = Color.White,
                modifier = Modifier
                    .align(Alignment.TopEnd)
                    .background(Color.Red, CircleShape)
                    .size(16.dp)
                    .wrapContentSize(Alignment.Center)
            )
        }
    }
}

// 渐变背景按钮
@Composable
fun GradientButton(text: String, onClick: () -> Unit) {
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .height(48.dp)
            .background(
                brush = Brush.horizontalGradient(
                    colors = listOf(Color.Blue, Color.Purple)
                ),
                shape = RoundedCornerShape(24.dp)
            )
            .clip(RoundedCornerShape(24.dp))
            .clickable(onClick = onClick)
            .wrapContentSize(Alignment.Center)
    ) {
        Text(
            text = text,
            color = Color.White,
            fontWeight = FontWeight.Bold
        )
    }
}
```

### 自定义 Modifier

```kotlin
// 自定义 Modifier 扩展函数
fun Modifier.shimmerEffect(
    durationMillis: Int = 1000,
    color: Color = Color.LightGray
): Modifier = this.then(
    Modifier.drawWithContent {
        // 自定义绘制逻辑
        drawContent()
    }
)

// 使用 Compose 提供的 Modifier 组合
fun Modifier.cardStyle(): Modifier = this
    .background(Color.White, RoundedCornerShape(12.dp))
    .border(1.dp, Color.LightGray, RoundedCornerShape(12.dp))
    .clip(RoundedCornerShape(12.dp))
    .padding(16.dp)

// 使用 Modifier.Node 创建复杂自定义 Modifier
class CustomBorderModifier(
    private val width: Dp,
    private val color: Color,
    private val cornerRadius: Dp
) : Modifier.Node() {
    override fun ContentDrawScope.draw() {
        // 先绘制内容
        drawContent()
        // 再绘制边框
        drawRoundRect(
            color = color,
            style = Stroke(width.toPx()),
            cornerRadius = CornerRadius(cornerRadius.toPx())
        )
    }
}

fun Modifier.customBorder(
    width: Dp = 1.dp,
    color: Color = Color.Black,
    cornerRadius: Dp = 0.dp
): Modifier = this.then(
    Modifier.NodeModifierFactory {
        CustomBorderModifier(width, color, cornerRadius)
    }
)

// 使用
@Composable
fun CustomCard() {
    Box(
        modifier = Modifier
            .size(200.dp)
            .customBorder(2.dp, Color.Blue, 16.dp)
    ) {
        Text("Custom Border Card")
    }
}
```

### Modifier 原理

```kotlin
// Modifier 是一个接口
interface Modifier {
    fun <R> foldIn(initial: R, operation: (R, Modifier.Element) -> R): R
    fun <R> foldOut(initial: R, operation: (Modifier.Element, R) -> R): R
    fun all(predicate: (Modifier.Element) -> Boolean): Boolean
    fun any(predicate: (Modifier.Element) -> Boolean): Boolean

    // 链式调用
    infix fun then(other: Modifier): Modifier
}

// Modifier 链本质是链表结构
// Modifier.padding(16.dp).background(Color.Blue)
// 等价于：
// CombinedModifier(
//     PaddingModifier(16.dp),
//     BackgroundModifier(Color.Blue)
// )

// 每个修饰符是链表的一个节点
// 绘制时从尾到头依次应用

// 检查 Modifier 是否包含某个元素
fun Modifier.hasClickable(): Boolean {
    return any { it is ClickableElement }
}

// 提取 Modifier 中的某个元素
fun Modifier.extractPadding(): PaddingValues? {
    return foldIn(null) { acc, element ->
        if (element is PaddingElement) element.padding else acc
    }
}
```

## 快速对照表

| 布局 | 用途 | 类似 XML |
|------|------|----------|
| `Column` | 垂直排列 | `LinearLayout vertical` |
| `Row` | 水平排列 | `LinearLayout horizontal` |
| `Box` | 堆叠布局 | `FrameLayout` |
| `ConstraintLayout` | 约束布局 | `ConstraintLayout` |
| `LazyColumn` | 垂直列表 | `RecyclerView` |
| `LazyRow` | 水平列表 | `RecyclerView horizontal` |
| `LazyVerticalGrid` | 网格列表 | `RecyclerView GridLayoutManager` |

| 控件 | 用途 | 类似 XML |
|------|------|----------|
| `Text` | 文本 | `TextView` |
| `Button` | 按钮 | `Button` |
| `Image` | 图片 | `ImageView` |
| `Icon` | 图标 | `ImageView + srcCompat` |
| `IconButton` | 图标按钮 | `ImageButton` |
| `Divider` | 分割线 | `View + background` |

| Modifier | 用途 | 类似 XML |
|----------|------|----------|
| `padding` | 内边距 | `padding` |
| `size` | 尺寸 | `layout_width/height` |
| `background` | 背景 | `background` |
| `clip` | 裁剪 | `clipToOutline` |
| `clickable` | 点击 | `onClick` |
| `border` | 边框 | `stroke` |
| `align` | 对齐 | `layout_gravity` |

**记忆口诀**：
- **Column = 垂直，Row = 水平，Box = 堆叠**
- **Lazy 开头 = 高性能列表（RecyclerView）**
- **Modifier 顺序 = size → padding → background → clip → clickable**
- **Text 样式 = fontSize / fontWeight / color / textAlign**
- **Button 三兄弟 = Button / OutlinedButton / TextButton**
