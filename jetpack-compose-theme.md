# JetPack Compose 样式、主题、适配

## 1. 字体、颜色、样式抽取

### 颜色抽取

```kotlin
// 方式一：对象管理（推荐）
object AppColors {
    val Primary = Color(0xFF1976D2)
    val PrimaryVariant = Color(0xFF1565C0)
    val Secondary = Color(0xFF03DAC6)
    val Background = Color(0xFFFFFBFE)
    val Surface = Color(0xFFFFFBFE)
    val Error = Color(0xFFB00020)
    val OnPrimary = Color.White
    val OnSecondary = Color.Black
    val OnBackground = Color(0xFF1C1B1F)
    val OnSurface = Color(0xFF1C1B1F)
    val OnError = Color.White
}

// 使用
@Composable
fun ThemedText() {
    Text(
        text = "Hello",
        color = AppColors.Primary
    )
}

// 方式二：Color.kt 文件
// res/values/colors.kt
val Blue500 = Color(0xFF2196F3)
val Blue700 = Color(0xFF1976D2)
val Green500 = Color(0xFF4CAF50)
val Red500 = Color(0xFFF44336)

// 方式三：Compose 主题颜色
@Composable
fun ThemeColors() {
    Text(
        text = "Themed",
        color = MaterialTheme.colorScheme.primary
    )
}
```

### 字体抽取

```kotlin
// 方式一：对象管理
object AppTypography {
    val TitleLarge = TextStyle(
        fontSize = 22.sp,
        fontWeight = FontWeight.Bold,
        lineHeight = 28.sp,
        letterSpacing = 0.sp
    )

    val TitleMedium = TextStyle(
        fontSize = 16.sp,
        fontWeight = FontWeight.SemiBold,
        lineHeight = 24.sp,
        letterSpacing = 0.15.sp
    )

    val BodyLarge = TextStyle(
        fontSize = 16.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    )

    val BodyMedium = TextStyle(
        fontSize = 14.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 20.sp,
        letterSpacing = 0.25.sp
    )

    val LabelSmall = TextStyle(
        fontSize = 11.sp,
        fontWeight = FontWeight.Medium,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp
    )
}

// 使用
@Composable
fun StyledText() {
    Text(
        text = "Title",
        style = AppTypography.TitleLarge
    )
}

// 方式二：使用 Material Theme
@Composable
fun MaterialStyledText() {
    Column {
        Text(
            text = "Headline Large",
            style = MaterialTheme.typography.headlineLarge
        )
        Text(
            text = "Title Medium",
            style = MaterialTheme.typography.titleMedium
        )
        Text(
            text = "Body Large",
            style = MaterialTheme.typography.bodyLarge
        )
        Text(
            text = "Label Small",
            style = MaterialTheme.typography.labelSmall
        )
    }
}

// 方式三：自定义字体
// 1. 字体文件放入 res/font/
// 2. 创建 FontFamily
val RobotoFontFamily = FontFamily(
    Font(R.font.roboto_regular, FontWeight.Normal),
    Font(R.font.roboto_bold, FontWeight.Bold),
    Font(R.font.roboto_italic, FontWeight.Normal, FontStyle.Italic),
    Font(R.font.roboto_bold_italic, FontWeight.Bold, FontStyle.Italic)
)

// 3. 使用
@Composable
fun CustomFontText() {
    Text(
        text = "Custom Font",
        fontFamily = RobotoFontFamily,
        fontWeight = FontWeight.Bold
    )
}
```

### 样式抽取

```kotlin
// 方式一：Modifier 扩展函数
fun Modifier.cardStyle(): Modifier = this
    .fillMaxWidth()
    .padding(16.dp)
    .background(Color.White, RoundedCornerShape(12.dp))
    .shadow(4.dp, RoundedCornerShape(12.dp))
    .padding(16.dp)

fun Modifier.buttonStyle(): Modifier = this
    .fillMaxWidth()
    .height(48.dp)
    .background(Color.Blue, RoundedCornerShape(24.dp))

// 使用
@Composable
fun StyledCard() {
    Card(modifier = Modifier.cardStyle()) {
        Text("Content")
    }
}

// 方式二：@Composable 样式组件
@Composable
fun PrimaryButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) {
    Button(
        onClick = onClick,
        enabled = enabled,
        modifier = modifier
            .fillMaxWidth()
            .height(48.dp),
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary,
            contentColor = MaterialTheme.colorScheme.onPrimary,
            disabledContainerColor = Color.Gray,
            disabledContentColor = Color.DarkGray
        )
    ) {
        Text(
            text = text,
            style = MaterialTheme.typography.titleMedium
        )
    }
}

// 方式三：TextStyle 扩展
fun TextStyle.bold(): TextStyle = this.copy(fontWeight = FontWeight.Bold)
fun TextStyle.large(): TextStyle = this.copy(fontSize = this.fontSize * 1.2)

// 使用
@Composable
fun ExtendedTextStyle() {
    Text(
        text = "Bold Text",
        style = MaterialTheme.typography.bodyLarge.bold()
    )
}
```

## 2. Material3 主题体系

### 完整主题定义

```kotlin
// Theme.kt
private val LightColorScheme = lightColorScheme(
    primary = Color(0xFF1976D2),
    onPrimary = Color.White,
    primaryContainer = Color(0xFFBBDEFB),
    onPrimaryContainer = Color(0xFF0D47A1),
    secondary = Color(0xFF03DAC6),
    onSecondary = Color.Black,
    secondaryContainer = Color(0xFFB2DFDB),
    onSecondaryContainer = Color(0xFF004D40),
    tertiary = Color(0xFF03DAC6),
    onTertiary = Color.Black,
    tertiaryContainer = Color(0xFFB2DFDB),
    onTertiaryContainer = Color(0xFF004D40),
    error = Color(0xFFB00020),
    onError = Color.White,
    errorContainer = Color(0xFFFFDAD6),
    onErrorContainer = Color(0xFF410002),
    background = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    surface = Color(0xFFFFFBFE),
    onSurface = Color(0xFF1C1B1F),
    surfaceVariant = Color(0xFFE7E0EC),
    onSurfaceVariant = Color(0xFF49454F),
    outline = Color(0xFF79747E)
)

private val DarkColorScheme = darkColorScheme(
    primary = Color(0xFF90CAF9),
    onPrimary = Color(0xFF0D47A1),
    primaryContainer = Color(0xFF1565C0),
    onPrimaryContainer = Color(0xFFBBDEFB),
    secondary = Color(0xFF80CBC4),
    onSecondary = Color(0xFF004D40),
    secondaryContainer = Color(0xFF004D40),
    onSecondaryContainer = Color(0xFFB2DFDB),
    tertiary = Color(0xFF80CBC4),
    onTertiary = Color(0xFF004D40),
    tertiaryContainer = Color(0xFF004D40),
    onTertiaryContainer = Color(0xFFB2DFDB),
    error = Color(0xFFCF6679),
    onError = Color(0xFF690005),
    errorContainer = Color(0xFF93000A),
    onErrorContainer = Color(0xFFFFDAD6),
    background = Color(0xFF1C1B1F),
    onBackground = Color(0xFFE6E1E5),
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFE6E1E5),
    surfaceVariant = Color(0xFF49454F),
    onSurfaceVariant = Color(0xFFCAC4D0),
    outline = Color(0xFF938F99)
)

// 自定义 Typography
private val AppTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    headlineLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.SemiBold,
        fontSize = 32.sp,
        lineHeight = 40.sp,
        letterSpacing = 0.sp
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 22.sp,
        lineHeight = 28.sp,
        letterSpacing = 0.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Normal,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    labelLarge = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 0.1.sp
    )
)

// 自定义 Shapes
private val AppShapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)

// 主题入口
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        // Android 12+ 动态颜色
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) {
                dynamicDarkColorScheme(context)
            } else {
                dynamicLightColorScheme(context)
            }
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        shapes = AppShapes,
        content = content
    )
}
```

### 使用主题

```kotlin
// Activity 入口
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            AppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    MainScreen()
                }
            }
        }
    }
}

// Compose 中使用主题
@Composable
fun ThemedScreen() {
    // 颜色
    val primaryColor = MaterialTheme.colorScheme.primary
    val backgroundColor = MaterialTheme.colorScheme.background
    val surfaceColor = MaterialTheme.colorScheme.surface

    // 字体
    val titleStyle = MaterialTheme.typography.titleLarge
    val bodyStyle = MaterialTheme.typography.bodyLarge

    // 形状
    val smallShape = MaterialTheme.shapes.small
    val mediumShape = MaterialTheme.shapes.medium

    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(backgroundColor)
    ) {
        Text(
            text = "Title",
            style = titleStyle,
            color = primaryColor
        )

        Card(
            shape = mediumShape,
            colors = CardDefaults.cardColors(
                containerColor = surfaceColor
            )
        ) {
            Text(
                text = "Content",
                style = bodyStyle
            )
        }
    }
}
```

## 3. ColorScheme、Typography、Shape

### ColorScheme 完整使用

```kotlin
// ColorScheme 所有颜色
@Composable
fun ColorSchemeDemo() {
    Column {
        // 主色
        ColorItem("primary", MaterialTheme.colorScheme.primary)
        ColorItem("onPrimary", MaterialTheme.colorScheme.onPrimary)
        ColorItem("primaryContainer", MaterialTheme.colorScheme.primaryContainer)
        ColorItem("onPrimaryContainer", MaterialTheme.colorScheme.onPrimaryContainer)

        // 次级色
        ColorItem("secondary", MaterialTheme.colorScheme.secondary)
        ColorItem("onSecondary", MaterialTheme.colorScheme.onSecondary)
        ColorItem("secondaryContainer", MaterialTheme.colorScheme.secondaryContainer)
        ColorItem("onSecondaryContainer", MaterialTheme.colorScheme.onSecondaryContainer)

        // 第三色
        ColorItem("tertiary", MaterialTheme.colorScheme.tertiary)
        ColorItem("onTertiary", MaterialTheme.colorScheme.onTertiary)
        ColorItem("tertiaryContainer", MaterialTheme.colorScheme.tertiaryContainer)
        ColorItem("onTertiaryContainer", MaterialTheme.colorScheme.onTertiaryContainer)

        // 错误色
        ColorItem("error", MaterialTheme.colorScheme.error)
        ColorItem("onError", MaterialTheme.colorScheme.onError)
        ColorItem("errorContainer", MaterialTheme.colorScheme.errorContainer)
        ColorItem("onErrorContainer", MaterialTheme.colorScheme.onErrorContainer)

        // 背景色
        ColorItem("background", MaterialTheme.colorScheme.background)
        ColorItem("onBackground", MaterialTheme.colorScheme.onBackground)

        // 表面色
        ColorItem("surface", MaterialTheme.colorScheme.surface)
        ColorItem("onSurface", MaterialTheme.colorScheme.onSurface)
        ColorItem("surfaceVariant", MaterialTheme.colorScheme.surfaceVariant)
        ColorItem("onSurfaceVariant", MaterialTheme.colorScheme.onSurfaceVariant)

        // 轮廓色
        ColorItem("outline", MaterialTheme.colorScheme.outline)
    }
}

@Composable
fun ColorItem(name: String, color: Color) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Box(
            modifier = Modifier
                .size(40.dp)
                .background(color, RoundedCornerShape(4.dp))
                .border(1.dp, Color.Gray, RoundedCornerShape(4.dp))
        )
        Spacer(modifier = Modifier.width(16.dp))
        Text(name)
    }
}

// 实际使用场景
@Composable
fun ThemedCard() {
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(
                text = "标题",
                color = MaterialTheme.colorScheme.onSurface,
                style = MaterialTheme.typography.titleMedium
            )
            Text(
                text = "内容",
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}
```

### Typography 完整使用

```kotlin
// Typography 所有样式
@Composable
fun TypographyDemo() {
    Column(modifier = Modifier.padding(16.dp)) {
        // Display
        Text("Display Large", style = MaterialTheme.typography.displayLarge)
        Text("Display Medium", style = MaterialTheme.typography.displayMedium)
        Text("Display Small", style = MaterialTheme.typography.displaySmall)

        Spacer(modifier = Modifier.height(16.dp))

        // Headline
        Text("Headline Large", style = MaterialTheme.typography.headlineLarge)
        Text("Headline Medium", style = MaterialTheme.typography.headlineMedium)
        Text("Headline Small", style = MaterialTheme.typography.headlineSmall)

        Spacer(modifier = Modifier.height(16.dp))

        // Title
        Text("Title Large", style = MaterialTheme.typography.titleLarge)
        Text("Title Medium", style = MaterialTheme.typography.titleMedium)
        Text("Title Small", style = MaterialTheme.typography.titleSmall)

        Spacer(modifier = Modifier.height(16.dp))

        // Body
        Text("Body Large", style = MaterialTheme.typography.bodyLarge)
        Text("Body Medium", style = MaterialTheme.typography.bodyMedium)
        Text("Body Small", style = MaterialTheme.typography.bodySmall)

        Spacer(modifier = Modifier.height(16.dp))

        // Label
        Text("Label Large", style = MaterialTheme.typography.labelLarge)
        Text("Label Medium", style = MaterialTheme.typography.labelMedium)
        Text("Label Small", style = MaterialTheme.typography.labelSmall)
    }
}

// 自定义 Typography
val CustomTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FontFamily(
            Font(R.font.noto_sans_bold, FontWeight.Bold)
        ),
        fontSize = 57.sp,
        lineHeight = 64.sp,
        letterSpacing = (-0.25).sp
    ),
    titleLarge = TextStyle(
        fontFamily = FontFamily(
            Font(R.font.noto_sans_medium, FontWeight.Medium)
        ),
        fontSize = 22.sp,
        lineHeight = 28.sp
    )
)

// 在主题中使用
@Composable
fun CustomTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        typography = CustomTypography,
        content = content
    )
}
```

### Shape 完整使用

```kotlin
// Shape 所有尺寸
@Composable
fun ShapeDemo() {
    Column(modifier = Modifier.padding(16.dp)) {
        ShapeItem("extraSmall", MaterialTheme.shapes.extraSmall)
        ShapeItem("small", MaterialTheme.shapes.small)
        ShapeItem("medium", MaterialTheme.shapes.medium)
        ShapeItem("large", MaterialTheme.shapes.large)
        ShapeItem("extraLarge", MaterialTheme.shapes.extraLarge)
    }
}

@Composable
fun ShapeItem(name: String, shape: Shape) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 8.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Box(
            modifier = Modifier
                .size(60.dp)
                .background(Color.Blue, shape)
        )
        Spacer(modifier = Modifier.width(16.dp))
        Text(name)
    }
}

// 自定义 Shape
val CustomShapes = Shapes(
    extraSmall = RoundedCornerShape(2.dp),
    small = RoundedCornerShape(4.dp),
    medium = RoundedCornerShape(8.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(32.dp)
)

// 不同角形状
@Composable
fun CustomCornerShapes() {
    Column {
        // 统一圆角
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(Color.Blue, RoundedCornerShape(16.dp))
        )

        // 百分比圆角
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(Color.Green, RoundedCornerShape(50))
        )

        // 单独角圆角
        Box(
            modifier = Modifier
                .size(100.dp)
                .background(
                    Color.Red,
                    RoundedCornerShape(
                        topStart = 16.dp,
                        topEnd = 0.dp,
                        bottomEnd = 16.dp,
                        bottomStart = 0.dp
                    )
                )
        )

        // 裁剪形状
        Box(
            modifier = Modifier
                .size(100.dp)
                .clip(RoundedCornerShape(16.dp))
                .background(Color.Yellow)
        )
    }
}
```

## 4. 深色模式适配

### 自动深色模式

```kotlin
// 检测系统深色模式
@Composable
fun DarkModeDetection() {
    val isDarkTheme = isSystemInDarkTheme()

    Text(
        text = if (isDarkTheme) "深色模式" else "浅色模式",
        style = MaterialTheme.typography.titleLarge
    )
}

// 强制深色/浅色模式
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 强制深色模式
        AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES)

        // 强制浅色模式
        // AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO)

        // 跟随系统
        // AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM)

        setContent {
            AppTheme {
                MainScreen()
            }
        }
    }
}

// 应用内切换主题
@Composable
fun ThemeSwitcher(viewModel: SettingsViewModel) {
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()

    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween
    ) {
        Text("深色模式")
        Switch(
            checked = isDarkTheme,
            onCheckedChange = { viewModel.setDarkTheme(it) }
        )
    }
}

// ViewModel
class SettingsViewModel : ViewModel() {
    private val _isDarkTheme = MutableStateFlow(false)
    val isDarkTheme: StateFlow<Boolean> = _isDarkTheme.asStateFlow()

    fun setDarkTheme(isDark: Boolean) {
        _isDarkTheme.value = isDark
        // 保存到 SharedPreferences
    }
}

// 在 Application 中应用主题
@Composable
fun AppContent() {
    val viewModel: SettingsViewModel = viewModel()
    val isDarkTheme by viewModel.isDarkTheme.collectAsState()

    AppTheme(darkTheme = isDarkTheme) {
        MainScreen()
    }
}
```

### 深色模式资源适配

```kotlin
// 颜色资源适配
// res/values/colors.xml (浅色)
<color name="background">#FFFFFF</color>
<color name="text_primary">#000000</color>

// res/values-night/colors.xml (深色)
<color name="background">#121212</color>
<color name="text_primary">#FFFFFF</color>

// Compose 中使用
@Composable
fun ThemedText() {
    Text(
        text = "Hello",
        color = MaterialTheme.colorScheme.onBackground
    )
}

// 图片资源适配
@Composable
fun ThemedImage() {
    Image(
        painter = painterResource(
            id = if (isSystemInDarkTheme()) {
                R.drawable.logo_dark
            } else {
                R.drawable.logo_light
            }
        ),
        contentDescription = "Logo"
    )
}

// 使用夜间限定资源目录
// res/drawable/ → 默认
// res/drawable-night/ → 深色模式
```

### 深色模式最佳实践

```kotlin
// 使用 Material 颜色，不要硬编码
@Composable
fun GoodThemedCard() {
    Card(
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    ) {
        Text(
            text = "Content",
            color = MaterialTheme.colorScheme.onSurface
        )
    }
}

// ❌ 错误：硬编码颜色
@Composable
fun BadThemedCard() {
    Card(
        modifier = Modifier.background(Color.White)
    ) {
        Text(
            text = "Content",
            color = Color.Black
        )
    }
}

// 深色模式下的表面层级
@Composable
fun SurfaceLevels() {
    Column {
        // 基础表面
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(100.dp)
                .background(MaterialTheme.colorScheme.surface)
        ) {
            Text("Surface", modifier = Modifier.padding(16.dp))
        }

        // 表面 +1 层级（卡片等）
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(100.dp)
                .background(MaterialTheme.colorScheme.surfaceVariant)
        ) {
            Text("Surface Variant", modifier = Modifier.padding(16.dp))
        }
    }
}
```

## 5. 屏幕尺寸适配

### 窗口大小类别

```kotlin
// 添加依赖
// implementation "androidx.compose.material3:material3-window-size-class:1.1.0"

@Composable
fun ResponsiveApp() {
    val windowSizeClass = calculateWindowSizeClass(activity = LocalContext.current as Activity)

    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> {
            // 手机：单栏布局
            CompactLayout()
        }
        WindowWidthSizeClass.Medium -> {
            // 平板：双栏布局
            MediumLayout()
        }
        WindowWidthSizeClass.Expanded -> {
            // 桌面/大屏：三栏布局
            ExpandedLayout()
        }
    }
}

// 完整响应式布局
@Composable
fun AdaptiveLayout() {
    val windowSizeClass = calculateWindowSizeClass()

    Row(modifier = Modifier.fillMaxSize()) {
        // 导航栏：大屏显示，小屏隐藏
        if (windowSizeClass.widthSizeClass != WindowWidthSizeClass.Compact) {
            NavigationRail(
                modifier = Modifier.width(80.dp),
                items = navItems
            )
        }

        // 主内容
        Box(modifier = Modifier.weight(1f)) {
            MainContent()
        }

        // 详情栏：仅大屏显示
        if (windowSizeClass.widthSizeClass == WindowWidthSizeClass.Expanded) {
            DetailPane(
                modifier = Modifier.width(300.dp)
            )
        }
    }
}

// 使用 Modifier 适配
@Composable
fun ResponsiveCard() {
    val windowSizeClass = calculateWindowSizeClass()

    Card(
        modifier = Modifier
            .fillMaxWidth(
                when (windowSizeClass.widthSizeClass) {
                    WindowWidthSizeClass.Compact -> 1f
                    WindowWidthSizeClass.Medium -> 0.8f
                    WindowWidthSizeClass.Expanded -> 0.6f
                    else -> 1f
                }
            )
    ) {
        Text("Responsive Card")
    }
}
```

### 横竖屏适配

```kotlin
// 检测屏幕方向
@Composable
fun OrientationDetection() {
    val configuration = LocalConfiguration.current

    when (configuration.orientation) {
        Configuration.ORIENTATION_PORTRAIT -> {
            Text("竖屏模式")
            PortraitLayout()
        }
        Configuration.ORIENTATION_LANDSCAPE -> {
            Text("横屏模式")
            LandscapeLayout()
        }
    }
}

// 竖屏布局
@Composable
fun PortraitLayout() {
    Column(modifier = Modifier.fillMaxSize()) {
        TopSection()
        MiddleSection()
        BottomSection()
    }
}

// 横屏布局
@Composable
fun LandscapeLayout() {
    Row(modifier = Modifier.fillMaxSize()) {
        LeftSection(modifier = Modifier.weight(1f))
        RightSection(modifier = Modifier.weight(1f))
    }
}

// 根据方向调整内容
@Composable
fun AdaptiveContent() {
    val configuration = LocalConfiguration.current
    val isLandscape = configuration.orientation == Configuration.ORIENTATION_LANDSCAPE

    if (isLandscape) {
        // 横屏：显示更多内容
        Row {
            ContentList()
            ContentDetail()
        }
    } else {
        // 竖屏：只显示列表
        ContentList()
    }
}

// 锁定屏幕方向（在 Activity 中）
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 锁定竖屏
        requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_PORTRAIT

        // 锁定横屏
        // requestedOrientation = ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE

        setContent {
            AppTheme {
                MainScreen()
            }
        }
    }
}
```

### 多屏幕尺寸适配

```kotlin
// 使用 dp 和权重
@Composable
fun ResponsiveContainer() {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .padding(
                horizontal = 16.dp,  // 小屏
                vertical = 8.dp
            )
    ) {
        // 内容
    }
}

// 根据屏幕宽度调整 padding
@Composable
fun AdaptivePadding() {
    val windowSizeClass = calculateWindowSizeClass()

    val horizontalPadding = when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> 16.dp
        WindowWidthSizeClass.Medium -> 32.dp
        WindowWidthSizeClass.Expanded -> 64.dp
        else -> 16.dp
    }

    Column(
        modifier = Modifier.padding(horizontal = horizontalPadding)
    ) {
        // 内容
    }
}

// 响应式图片
@Composable
fun ResponsiveImage() {
    val windowSizeClass = calculateWindowSizeClass()

    val imageSize = when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> 100.dp
        WindowWidthSizeClass.Medium -> 200.dp
        WindowWidthSizeClass.Expanded -> 300.dp
        else -> 100.dp
    }

    Image(
        painter = painterResource(id = R.drawable.image),
        contentDescription = null,
        modifier = Modifier.size(imageSize)
    )
}
```

## 6. 资源引用

### 字符串资源

```kotlin
// res/values/strings.xml
<resources>
    <string name="app_name">我的应用</string>
    <string name="hello">你好，%1$s!</string>
    <string name="login">登录</string>
    <string name="register">注册</string>
</resources>

// Compose 中使用
@Composable
fun StringResourceUsage(name: String) {
    Column {
        // 简单字符串
        Text(
            text = stringResource(R.string.app_name)
        )

        // 带参数的字符串
        Text(
            text = stringResource(R.string.hello, name)
        )

        // 按钮文本
        Button(onClick = { }) {
            Text(stringResource(R.string.login))
        }
    }
}

// 复数字符串
// res/values/strings.xml
<plurals name="item_count">
    <item quantity="one">%d 项</item>
    <item quantity="other">%d 项</item>
</plurals>

// 使用
@Composable
fun PluralStringUsage(count: Int) {
    Text(
        text = resources.getQuantityString(R.plurals.item_count, count, count)
    )
}
```

### 尺寸资源

```kotlin
// res/values/dimens.xml
<resources>
    <dimen name="spacing_small">8.dp</dimen>
    <dimen name="spacing_medium">16.dp</dimen>
    <dimen name="spacing_large">32.dp</dimen>

    <dimen name="text_size_small">12.sp</dimen>
    <dimen name="text_size_medium">16.sp</dimen>
    <dimen name="text_size_large">20.sp</dimen>

    <dimen name="card_height">100.dp</dimen>
    <dimen name="button_height">48.dp</dimen>
</resources>

// 不同屏幕尺寸的资源
// res/values-sw600dp/dimens.xml (平板)
<resources>
    <dimen name="spacing_small">16.dp</dimen>
    <dimen name="spacing_medium">24.dp</dimen>
    <dimen name="spacing_large">48.dp</dimen>
</resources>

// Compose 中使用
@Composable
fun DimensionResourceUsage() {
    val spacingSmall = dimensionResource(id = R.dimen.spacing_small)
    val spacingMedium = dimensionResource(id = R.dimen.spacing_medium)
    val textSizeMedium = dimensionResource(id = R.dimen.text_size_medium)

    Column(
        modifier = Modifier.padding(spacingMedium)
    ) {
        Text(
            text = "标题",
            fontSize = dimensionResource(id = R.dimen.text_size_large),
            modifier = Modifier.padding(bottom = spacingSmall)
        )

        Text(
            text = "内容",
            fontSize = textSizeMedium
        )
    }
}
```

### 颜色资源

```kotlin
// res/values/colors.xml
<resources>
    <color name="primary">#1976D2</color>
    <color name="primary_variant">#1565C0</color>
    <color name="secondary">#03DAC6</color>
    <color name="background">#FFFFFF</color>
    <color name="surface">#FFFFFF</color>
    <color name="error">#B00020</color>
</resources>

// Compose 中使用
@Composable
fun ColorResourceUsage() {
    val primaryColor = colorResource(id = R.color.primary)
    val backgroundColor = colorResource(id = R.color.background)

    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(backgroundColor)
    ) {
        Text(
            text = "Primary Color",
            color = primaryColor
        )

        Button(
            onClick = { },
            colors = ButtonDefaults.buttonColors(
                containerColor = colorResource(id = R.color.primary)
            )
        ) {
            Text("按钮")
        }
    }
}

// 深色模式颜色
// res/values-night/colors.xml
<resources>
    <color name="primary">#90CAF9</color>
    <color name="background">#121212</color>
    <color name="surface">#1E1E1E</color>
</resources>
```

### 资源文件完整结构

```
res/
├── values/
│   ├── strings.xml       # 字符串
│   ├── colors.xml        # 颜色
│   ├── dimens.xml        # 尺寸
│   ├── themes.xml        # 主题
│   └── styles.xml        # 样式
├── values-night/
│   ├── colors.xml        # 深色模式颜色
│   └── strings.xml       # 深色模式字符串（可选）
├── values-sw600dp/
│   └── dimens.xml        # 平板尺寸
├── values-zh-rCN/
│   └── strings.xml       # 中文翻译
├── drawable/
│   ├── ic_launcher.xml   # 图标
│   └── background.xml    # 背景
├── drawable-night/
│   └── background.xml    # 深色模式背景
└── font/
    ├── roboto_regular.ttf
    └── roboto_bold.ttf
```

## 快速对照表

| 类别 | 资源类型 | 文件位置 | 使用方式 |
|------|----------|----------|----------|
| 颜色 | `colors.xml` | `res/values/` | `colorResource(R.color.xxx)` |
| 字符串 | `strings.xml` | `res/values/` | `stringResource(R.string.xxx)` |
| 尺寸 | `dimens.xml` | `res/values/` | `dimensionResource(R.dimen.xxx)` |
| 字体 | `.ttf/.otf` | `res/font/` | `Font(R.font.xxx)` |
| 图片 | `.png/.svg` | `res/drawable/` | `painterResource(R.drawable.xxx)` |
| 主题 | `themes.xml` | `res/values/` | `MaterialTheme` |

| 适配类型 | 实现方式 |
|----------|----------|
| 深色模式 | `isSystemInDarkTheme()` + `darkColorScheme()` |
| 屏幕尺寸 | `calculateWindowSizeClass()` |
| 横竖屏 | `LocalConfiguration.current.orientation` |
| 多语言 | `values-zh-rCN/strings.xml` |
| 平板适配 | `values-sw600dp/dimens.xml` |

**记忆口诀**：
- **MaterialTheme = 主题入口，colorScheme/typography/shapes**
- **isSystemInDarkTheme() = 检测深色模式**
- **calculateWindowSizeClass() = 窗口大小分类**
- **stringResource = 字符串，colorResource = 颜色**
- **dimensionResource = 尺寸，painterResource = 图片**
- **values-night = 深色资源，values-sw600dp = 平板资源**
