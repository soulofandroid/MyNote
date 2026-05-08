# JetPack Compose 动画模块

## 1. 基础动画

### 透明度动画

```kotlin
// 基础透明度动画
@Composable
fun FadeAnimation() {
    var visible by remember { mutableStateOf(true) }
    // 动画状态
    val alpha by animateFloatAsState(
        targetValue = if (visible) 1f else 0f,
        animationSpec = tween(durationMillis = 300)
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .alpha(alpha)
                .background(Color.Blue)
        )

        Button(
            onClick = { visible = !visible },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text(if (visible) "隐藏" else "显示")
        }
    }
}

// 使用 AnimatedAlpha
@Composable
fun AnimatedAlphaExample() {
    var visible by remember { mutableStateOf(true) }

    AnimatedAlpha(
        alpha = if (visible) 1f else 0f
    ) {
        Text("淡入淡出文字")
    }

    Button(onClick = { visible = !visible }) {
        Text("切换")
    }
}
```

### 位移动画

```kotlin
// 偏移动画
@Composable
fun OffsetAnimation() {
    var moved by remember { mutableStateOf(false) }
    // 动画偏移
    val offsetX by animateIntAsState(
        targetValue = if (moved) 200 else 0,
        animationSpec = spring()
    )
    val offsetY by animateIntAsState(
        targetValue = if (moved) 100 else 0,
        animationSpec = spring()
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .offset { IntOffset(offsetX, offsetY) }
                .background(Color.Green)
        )

        Button(
            onClick = { moved = !moved },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("移动")
        }
    }
}

// 使用 animateDpAsState
@Composable
fun DpAnimation() {
    var expanded by remember { mutableStateOf(false) }
    val width by animateDpAsState(
        targetValue = if (expanded) 300.dp else 100.dp,
        animationSpec = tween(500)
    )

    Column {
        Box(
            modifier = Modifier
                .width(width)
                .height(50.dp)
                .background(Color.Red)
        )

        Button(onClick = { expanded = !expanded }) {
            Text("展开/收缩")
        }
    }
}
```

### 缩放动画

```kotlin
// 缩放动画
@Composable
fun ScaleAnimation() {
    var scaled by remember { mutableStateOf(false) }
    val scale by animateFloatAsState(
        targetValue = if (scaled) 1.5f else 1f,
        animationSpec = spring(
            stiffness = Spring.StiffnessLow
        )
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .scale(scale)
                .background(Color.Magenta)
        )

        Button(
            onClick = { scaled = !scaled },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("缩放")
        }
    }
}

// 组合动画（透明度 + 缩放）
@Composable
fun CombinedAnimation() {
    var visible by remember { mutableStateOf(true) }

    val alpha by animateFloatAsState(
        targetValue = if (visible) 1f else 0f,
        animationSpec = tween(300)
    )
    val scale by animateFloatAsState(
        targetValue = if (visible) 1f else 0.8f,
        animationSpec = tween(300)
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .alpha(alpha)
                .scale(scale)
                .background(Color.Cyan)
        )

        Button(
            onClick = { visible = !visible },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("组合动画")
        }
    }
}

// 使用 GraphicsLayer 进行高性能动画
@Composable
fun GraphicsLayerAnimation() {
    var rotated by remember { mutableStateOf(false) }
    val rotation by animateFloatAsState(
        targetValue = if (rotated) 360f else 0f,
        animationSpec = tween(1000)
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .graphicsLayer {
                    rotationZ = rotation
                }
                .background(Color.Yellow)
        )

        Button(
            onClick = { rotated = !rotated },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("旋转")
        }
    }
}
```

### 颜色动画

```kotlin
// 颜色渐变动画
@Composable
fun ColorAnimation() {
    var isBlue by remember { mutableStateOf(true) }
    val color by animateColorAsState(
        targetValue = if (isBlue) Color.Blue else Color.Red,
        animationSpec = tween(500)
    )

    Column {
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(color)
        )

        Button(
            onClick = { isBlue = !isBlue },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("切换颜色")
        }
    }
}

// 背景渐变动画
@Composable
fun GradientAnimation() {
    var isDark by remember { mutableStateOf(false) }
    val color1 by animateColorAsState(
        targetValue = if (isDark) Color.DarkGray else Color.LightGray,
        animationSpec = tween(1000)
    )
    val color2 by animateColorAsState(
        targetValue = if (isDark) Color.Black else Color.White,
        animationSpec = tween(1000)
    )

    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(
                brush = Brush.verticalGradient(
                    colors = listOf(color1, color2)
                )
            )
    ) {
        Button(
            onClick = { isDark = !isDark },
            modifier = Modifier.align(Alignment.Center)
        ) {
            Text("切换主题")
        }
    }
}
```

## 2. animateContentSize 内容大小动画

### 基础使用

```kotlin
// 内容大小变化动画
@Composable
fun AnimateContentSizeExample() {
    var expanded by remember { mutableStateOf(false) }

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .animateContentSize(
                animationSpec = spring(
                    stiffness = Spring.StiffnessMediumLow
                )
            )
            .clickable { expanded = !expanded }
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = "点击展开/收缩",
                style = MaterialTheme.typography.titleMedium
            )

            if (expanded) {
                Spacer(modifier = Modifier.height(8.dp))
                Text("这是展开后显示的内容")
                Text("可以有更多行")
            }
        }
    }
}

// 列表项展开动画
@Composable
fun ExpandableListItem() {
    var expandedIndex by remember { mutableStateOf<Int?>(null) }
    val items = listOf("项目 1", "项目 2", "项目 3", "项目 4")

    LazyColumn {
        items(items) { item ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 16.dp, vertical = 8.dp)
                    .animateContentSize()
            ) {
                Column(
                    modifier = Modifier
                        .fillMaxWidth()
                        .clickable {
                            expandedIndex = if (expandedIndex == items.indexOf(item)) {
                                null
                            } else {
                                items.indexOf(item)
                            }
                        }
                        .padding(16.dp)
                ) {
                    Row(
                        modifier = Modifier.fillMaxWidth(),
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        Text(item)
                        Icon(
                            imageVector = if (expandedIndex == items.indexOf(item)) {
                                Icons.Default.ExpandLess
                            } else {
                                Icons.Default.ExpandMore
                            },
                            contentDescription = null
                        )
                    }

                    if (expandedIndex == items.indexOf(item)) {
                        Spacer(modifier = Modifier.height(8.dp))
                        Text("详细内容：这是 $item 的详细信息")
                    }
                }
            }
        }
    }
}
```

### 自定义动画规格

```kotlin
// 带自定义动画规格
@Composable
fun CustomAnimateContentSize() {
    var expanded by remember { mutableStateOf(false) }

    Box(
        modifier = Modifier
            .fillMaxWidth()
            .animateContentSize(
                animationSpec = tween(
                    durationMillis = 500,
                    easing = FastOutSlowInEasing
                )
            )
            .clickable { expanded = !expanded }
    ) {
        Column {
            Text("标题")
            if (expanded) {
                Text("展开的内容 1")
                Text("展开的内容 2")
                Text("展开的内容 3")
            }
        }
    }
}

// 多个动画组合
@Composable
fun CombinedSizeAnimation() {
    var expanded by remember { mutableStateOf(false) }

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .animateContentSize(
                animationSpec = spring(
                    dampingRatio = Spring.DampingRatioMediumBouncy,
                    stiffness = Spring.StiffnessLow
                )
            )
            .clickable { expanded = !expanded }
    ) {
        Row(
            modifier = Modifier
                .padding(16.dp)
                .animateContentSize()
        ) {
            Icon(
                imageVector = Icons.Default.Info,
                contentDescription = null,
                modifier = Modifier.animateContentSize()
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text("信息标题")
                if (expanded) {
                    Spacer(modifier = Modifier.height(8.dp))
                    Text("详细信息内容...")
                }
            }
        }
    }
}
```

## 3. AnimatedVisibility 显示隐藏动画

### 基础使用

```kotlin
// 基础显示/隐藏动画
@Composable
fun BasicAnimatedVisibility() {
    var visible by remember { mutableStateOf(true) }

    Column {
        AnimatedVisibility(
            visible = visible,
            enter = fadeIn() + slideInVertically(),
            exit = fadeOut() + slideOutVertically()
        ) {
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(16.dp)
            ) {
                Text(
                    "这是可动画显示/隐藏的内容",
                    modifier = Modifier.padding(16.dp)
                )
            }
        }

        Button(onClick = { visible = !visible }) {
            Text(if (visible) "隐藏" else "显示")
        }
    }
}

// 不同进入/退出动画
@Composable
fun DifferentAnimations() {
    var visible by remember { mutableStateOf(true) }

    Column {
        // 滑入/滑出
        AnimatedVisibility(
            visible = visible,
            enter = slideInHorizontally(),
            exit = slideOutHorizontally()
        ) {
            Text("水平滑动", modifier = Modifier.padding(8.dp))
        }

        // 缩放进入/退出
        AnimatedVisibility(
            visible = visible,
            enter = scaleIn(),
            exit = scaleOut()
        ) {
            Text("缩放动画", modifier = Modifier.padding(8.dp))
        }

        // 淡入/淡出
        AnimatedVisibility(
            visible = visible,
            enter = fadeIn(),
            exit = fadeOut()
        ) {
            Text("淡入淡出", modifier = Modifier.padding(8.dp))
        }

        Button(onClick = { visible = !visible }) {
            Text("切换")
        }
    }
}
```

### 自定义动画规格

```kotlin
// 自定义进入/退出动画
@Composable
fun CustomAnimatedVisibility() {
    var visible by remember { mutableStateOf(true) }

    AnimatedVisibility(
        visible = visible,
        enter = fadeIn(animationSpec = tween(500)) +
                slideInVertically(
                    animationSpec = spring(),
                    initialOffsetY = { -100 }
                ),
        exit = fadeOut(animationSpec = tween(500)) +
               slideOutVertically(
                   animationSpec = spring(),
                   targetOffsetY = { 100 }
               )
    ) {
        Card(modifier = Modifier.padding(16.dp)) {
            Text("自定义动画", modifier = Modifier.padding(16.dp))
        }
    }
}

// 条件动画
@Composable
fun ConditionalAnimation() {
    var showContent by remember { mutableStateOf(false) }
    var animationType by remember { mutableStateOf(0) }

    Column {
        AnimatedVisibility(
            visible = showContent,
            enter = when (animationType) {
                0 -> fadeIn() + slideInVertically()
                1 -> scaleIn()
                2 -> slideInHorizontally()
                else -> fadeIn()
            },
            exit = when (animationType) {
                0 -> fadeOut() + slideOutVertically()
                1 -> scaleOut()
                2 -> slideOutHorizontally()
                else -> fadeOut()
            }
        ) {
            Text("条件动画内容")
        }

        Row {
            Button(onClick = { showContent = !showContent }) {
                Text("切换")
            }
            Button(onClick = { animationType = (animationType + 1) % 3 }) {
                Text("切换动画类型")
            }
        }
    }
}
```

### 嵌套 AnimatedVisibility

```kotlin
// 顺序动画
@Composable
fun SequentialAnimation() {
    var visible by remember { mutableStateOf(false) }

    Column {
        AnimatedVisibility(
            visible = visible,
            enter = fadeIn() + slideInVertically(),
            exit = fadeOut() + slideOutVertically()
        ) {
            Card(modifier = Modifier.padding(8.dp)) {
                Text("第一项", modifier = Modifier.padding(16.dp))
            }
        }

        AnimatedVisibility(
            visible = visible,
            enter = fadeIn(delay = 100) + slideInVertically(delay = 100),
            exit = fadeOut(delay = 100) + slideOutVertically(delay = 100)
        ) {
            Card(modifier = Modifier.padding(8.dp)) {
                Text("第二项", modifier = Modifier.padding(16.dp))
            }
        }

        AnimatedVisibility(
            visible = visible,
            enter = fadeIn(delay = 200) + slideInVertically(delay = 200),
            exit = fadeOut(delay = 200) + slideOutVertically(delay = 200)
        ) {
            Card(modifier = Modifier.padding(8.dp)) {
                Text("第三项", modifier = Modifier.padding(16.dp))
            }
        }

        Button(onClick = { visible = !visible }) {
            Text("切换全部")
        }
    }
}

// 列表项动画
@Composable
fun AnimatedListItems() {
    var items by remember { mutableStateOf(listOf<String>()) }

    Column {
        items.forEachIndexed { index, item ->
            AnimatedVisibility(
                visible = item in items,
                enter = slideInHorizontally(
                    initialOffsetX = { -100 },
                    animationSpec = tween(300, delay = index * 50)
                ) + fadeIn(animationSpec = tween(300, delay = index * 50)),
                exit = slideOutHorizontally(
                    targetOffsetX = { 100 },
                    animationSpec = tween(300)
                ) + fadeOut(animationSpec = tween(300))
            ) {
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(8.dp)
                ) {
                    Row(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp),
                        horizontalArrangement = Arrangement.SpaceBetween
                    ) {
                        Text(item)
                        IconButton(onClick = {
                            items = items - item
                        }) {
                            Icon(Icons.Default.Close, contentDescription = "删除")
                        }
                    }
                }
            }
        }

        Row {
            Button(
                onClick = {
                    items = items + "项目 ${items.size + 1}"
                }
            ) {
                Text("添加")
            }
        }
    }
}
```

## 4. 页面共享元素动画

```kotlin
// 使用 Accompanist 实现共享元素动画
// 添加依赖
// implementation "com.google.accompanist:accompanist-navigation-animation:0.32.0"

@Composable
fun SharedElementNavigation() {
    val navController = rememberNavController()

    AnimatedNavHost(
        navController = navController,
        startDestination = "list"
    ) {
        composable("list") {
            ImageList(
                onImageClick = { id ->
                    navController.navigate("detail/$id")
                }
            )
        }

        composable(
            route = "detail/{id}",
            arguments = listOf(navArgument("id") { type = NavType.StringType })
        ) { backStackEntry ->
            val id = backStackEntry.arguments?.getString("id")
            ImageDetail(
                id = id,
                onBack = { navController.popBackStack() }
            )
        }
    }
}

// 列表页面
@Composable
fun ImageList(onImageClick: (String) -> Unit) {
    val images = listOf("1", "2", "3", "4")

    LazyVerticalGrid(columns = GridCells.Fixed(2)) {
        items(images) { id ->
            Box(
                modifier = Modifier
                    .aspectRatio(1f)
                    .clickable { onImageClick(id) }
                    .sharedElement(
                        state = rememberSharedElementState(key = "image-$id"),
                        animatedVisibilityScope = this
                    )
            ) {
                Image(
                    painter = painterResource(id = R.drawable.sample),
                    contentDescription = null,
                    modifier = Modifier.fillMaxSize()
                )
            }
        }
    }
}

// 详情页面
@Composable
fun ImageDetail(id: String?, onBack: () -> Unit) {
    Box {
        Image(
            painter = painterResource(id = R.drawable.sample),
            contentDescription = null,
            modifier = Modifier
                .fillMaxSize()
                .sharedElement(
                    state = rememberSharedElementState(key = "image-$id"),
                    animatedVisibilityScope = this
                )
        )

        TopAppBar(
            title = { Text("详情") },
            navigationIcon = {
                IconButton(onClick = onBack) {
                    Icon(Icons.Default.ArrowBack, contentDescription = "返回")
                }
            }
        )
    }
}
```

## 5. 无限动画、插值器

### 无限循环动画

```kotlin
// 无限旋转
@Composable
fun InfiniteRotation() {
    val infiniteTransition = rememberInfiniteTransition()
    val rotation by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 360f,
        animationSpec = infiniteRepeatable(
            animation = tween(2000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        )
    )

    Box(
        modifier = Modifier
            .size(100.dp)
            .graphicsLayer { rotationZ = rotation }
            .background(Color.Blue, CircleShape)
    )
}

// 无限脉冲（缩放）
@Composable
fun InfinitePulse() {
    val infiniteTransition = rememberInfiniteTransition()
    val scale by infiniteTransition.animateFloat(
        initialValue = 1f,
        targetValue = 1.2f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000),
            repeatMode = RepeatMode.Reverse
        )
    )

    Box(
        modifier = Modifier
            .size(100.dp)
            .scale(scale)
            .background(Color.Red, CircleShape)
    )
}

// 无限呼吸灯（透明度）
@Composable
fun InfiniteBreath() {
    val infiniteTransition = rememberInfiniteTransition()
    val alpha by infiniteTransition.animateFloat(
        initialValue = 0.3f,
        targetValue = 1f,
        animationSpec = infiniteRepeatable(
            animation = tween(1500),
            repeatMode = RepeatMode.Reverse
        )
    )

    Box(
        modifier = Modifier
            .size(100.dp)
            .alpha(alpha)
            .background(Color.Green, CircleShape)
    )
}

// 无限左右移动
@Composable
fun InfiniteSlide() {
    val infiniteTransition = rememberInfiniteTransition()
    val offsetX by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 100f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000),
            repeatMode = RepeatMode.Reverse
        )
    )

    Box(
        modifier = Modifier
            .size(50.dp)
            .offset(x = offsetX.dp)
            .background(Color.Magenta)
    )
}

// 加载指示器（组合动画）
@Composable
fun CustomLoadingIndicator() {
    val infiniteTransition = rememberInfiniteTransition()

    val rotation by infiniteTransition.animateFloat(
        initialValue = 0f,
        targetValue = 360f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000, easing = LinearEasing),
            repeatMode = RepeatMode.Restart
        )
    )

    val scale by infiniteTransition.animateFloat(
        initialValue = 0.8f,
        targetValue = 1.2f,
        animationSpec = infiniteRepeatable(
            animation = tween(500),
            repeatMode = RepeatMode.Reverse
        )
    )

    Box(
        modifier = Modifier
            .size(50.dp)
            .graphicsLayer {
                rotationZ = rotation
                scaleX = scale
                scaleY = scale
            }
            .background(Color.Cyan, CircleShape)
    )
}
```

### 插值器（Easing）

```kotlin
// 常用插值器
@Composable
fun EasingDemo() {
    Column {
        EasingItem("Linear", LinearEasing)
        EasingItem("FastOutSlowIn", FastOutSlowInEasing)
        EasingItem("FastOutLinearIn", FastOutLinearInEasing)
        EasingItem("LinearOutSlowIn", LinearOutSlowInEasing)
        
        // 自定义插值器
        EasingItem("Custom", PathEasing(Path.cubicBezier(0.4f, 0f, 0.2f, 1f)))
    }
}

@Composable
fun EasingItem(name: String, easing: Easing) {
    var started by remember { mutableStateOf(false) }
    val progress by animateFloatAsState(
        targetValue = if (started) 1f else 0f,
        animationSpec = tween(1000, easing = easing)
    )

    Column(modifier = Modifier.padding(8.dp)) {
        Text(name)
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(4.dp)
                .background(Color.LightGray)
        ) {
            Box(
                modifier = Modifier
                    .fillMaxWidth(progress)
                    .height(4.dp)
                    .background(Color.Blue)
            )
        }
        Button(
            onClick = { started = !started },
            modifier = Modifier.padding(top = 4.dp)
        ) {
            Text("播放")
        }
    }
}

// 自定义贝塞尔曲线插值器
@Composable
fun CustomBezierAnimation() {
    var started by remember { mutableStateOf(false) }
    
    // 自定义贝塞尔曲线
    val customEasing = PathEasing(
        Path.cubicBezier(0.68f, -0.55f, 0.265f, 1.55f)  // 弹性效果
    )
    
    val progress by animateFloatAsState(
        targetValue = if (started) 1f else 0f,
        animationSpec = tween(1500, easing = customEasing)
    )

    Column {
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(50.dp)
                .background(Color.LightGray)
        ) {
            Box(
                modifier = Modifier
                    .fillMaxWidth(progress)
                    .height(50.dp)
                    .background(Color.Blue)
            )
        }
        
        Button(
            onClick = { started = !started },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("弹性动画")
        }
    }
}
```

### 动画规格（AnimationSpec）

```kotlin
// tween：基于时间的动画
@Composable
fun TweenAnimation() {
    var target by remember { mutableStateOf(0f) }
    val value by animateFloatAsState(
        targetValue = target,
        animationSpec = tween(
            durationMillis = 1000,
            delayMillis = 100,
            easing = FastOutSlowInEasing
        )
    )

    Slider(
        value = value,
        onValueChange = { target = it },
        valueRange = 0f..100f
    )
}

// spring：基于物理的弹簧动画
@Composable
fun SpringAnimation() {
    var target by remember { mutableStateOf(0f) }
    val value by animateFloatAsState(
        targetValue = target,
        animationSpec = spring(
            dampingRatio = Spring.DampingRatioMediumBouncy,
            stiffness = Spring.StiffnessLow
        )
    )

    Column {
        Text("位置：$value")
        Slider(
            value = value,
            onValueChange = { target = it }
        )
        
        // 阻尼比选项
        Text("阻尼比:")
        Row {
            Text("NoBouncy")
            Text("LowBouncy")
            Text("MediumBouncy")
            Text("HighBouncy")
        }
        
        // 刚度选项
        Text("刚度:")
        Row {
            Text("Low")
            Text("Medium")
            Text("High")
            Text("VeryHigh")
        }
    }
}

// keyframes：关键帧动画
@Composable
fun KeyframeAnimation() {
    var started by remember { mutableStateOf(false) }
    val value by animateFloatAsState(
        targetValue = if (started) 300f else 0f,
        animationSpec = keyframes {
            durationMillis = 2000
            0f at 0
            100f at 500
            250f at 1000
            200f at 1500
            300f at 2000
        }
    )

    Column {
        Box(
            modifier = Modifier
                .size(50.dp)
                .offset(y = value.dp)
                .background(Color.Red)
        )
        
        Button(
            onClick = { started = !started },
            modifier = Modifier.padding(top = 16.dp)
        ) {
            Text("关键帧动画")
        }
    }
}
```

## 快速对照表

| 动画类型 | 函数/组件 | 用途 |
|----------|----------|------|
| 透明度 | `animateFloatAsState` + `alpha` | 淡入淡出 |
| 位移 | `animateIntAsState` + `offset` | 位置移动 |
| 缩放 | `animateFloatAsState` + `scale` | 放大缩小 |
| 旋转 | `animateFloatAsState` + `rotationZ` | 旋转效果 |
| 颜色 | `animateColorAsState` | 颜色渐变 |
| 大小 | `animateContentSize` | 内容大小变化 |
| 显示隐藏 | `AnimatedVisibility` | 条件显示 |
| 无限动画 | `rememberInfiniteTransition` | 加载/脉冲 |

| 动画规格 | 用途 |
|----------|------|
| `tween()` | 基于时间，可设置时长/延迟 |
| `spring()` | 基于物理，弹性效果 |
| `keyframes` | 关键帧动画 |
| `infiniteRepeatable()` | 无限循环 |

| 插值器 | 效果 |
|--------|------|
| `LinearEasing` | 匀速 |
| `FastOutSlowInEasing` | 快出慢入（默认） |
| `FastOutLinearInEasing` | 快出匀速入 |
| `LinearOutSlowInEasing` | 匀速出慢入 |
| `PathEasing` | 自定义贝塞尔曲线 |

**记忆口诀**：
- **animateFloatAsState = 浮点动画（透明度/缩放/旋转）**
- **animateIntAsState = 整数动画（位移）**
- **animateColorAsState = 颜色动画**
- **animateContentSize = 内容大小自动动画**
- **AnimatedVisibility = 显示隐藏动画**
- **rememberInfiniteTransition = 无限循环动画**
- **tween = 时间动画，spring = 弹簧动画**
- **FastOutSlowInEasing = 默认插值器**
