# JetPack Compose 弹窗、菜单、交互组件

## 1. Dialog 弹窗

### 基础 Dialog

```kotlin
// Dialog：模态对话框，阻塞底层交互
@Composable
fun BasicDialogExample() {
    var showDialog by remember { mutableStateOf(false) }

    Column {
        Button(onClick = { showDialog = true }) {
            Text("Show Dialog")
        }

        if (showDialog) {
            Dialog(onDismissRequest = { showDialog = false }) {
                Card(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp)
                ) {
                    Column(modifier = Modifier.padding(24.dp)) {
                        Text(
                            text = "This is a custom dialog",
                            style = MaterialTheme.typography.titleLarge
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                        Text("Dialog content goes here...")
                        Spacer(modifier = Modifier.height(24.dp))
                        Row(
                            modifier = Modifier.fillMaxWidth(),
                            horizontalArrangement = Arrangement.End
                        ) {
                            TextButton(onClick = { showDialog = false }) {
                                Text("Close")
                            }
                        }
                    }
                }
            }
        }
    }
}

// Dialog 完整参数
@Composable
fun FullDialogExample() {
    var showDialog by remember { mutableStateOf(false) }

    if (showDialog) {
        Dialog(
            onDismissRequest = { showDialog = false },
            properties = DialogProperties(
                dismissOnBackPress = true,      // 返回键关闭
                dismissOnClickOutside = true,   // 点击外部关闭
                usePlatformDefaultWidth = false // 不使用默认宽度
            )
        ) {
            Box(
                modifier = Modifier
                    .size(300.dp, 200.dp)
                    .background(Color.White, RoundedCornerShape(16.dp))
            ) {
                Text("Custom size dialog", modifier = Modifier.padding(16.dp))
            }
        }
    }
}
```

### AlertDialog 确认弹窗

```kotlin
// 基础 AlertDialog
@Composable
fun BasicAlertDialog() {
    var showDialog by remember { mutableStateOf(false) }

    Column {
        Button(onClick = { showDialog = true }) {
            Text("Show Alert")
        }

        if (showDialog) {
            AlertDialog(
                onDismissRequest = { showDialog = false },
                title = { Text("确认操作") },
                text = { Text("确定要执行此操作吗？") },
                confirmButton = {
                    TextButton(onClick = {
                        // 确认逻辑
                        showDialog = false
                    }) {
                        Text("确认")
                    }
                },
                dismissButton = {
                    TextButton(onClick = { showDialog = false }) {
                        Text("取消")
                    }
                }
            )
        }
    }
}

// 带图标的 AlertDialog
@Composable
fun AlertDialogWithIcon() {
    var showDialog by remember { mutableStateOf(false) }

    if (showDialog) {
        AlertDialog(
            icon = {
                Icon(
                    Icons.Default.Warning,
                    contentDescription = "Warning",
                    tint = Color.Red
                )
            },
            title = { Text("删除确认") },
            text = { Text("删除后无法恢复，确定继续？") },
            onDismissRequest = { showDialog = false },
            confirmButton = {
                Button(
                    onClick = {
                        // 删除逻辑
                        showDialog = false
                    },
                    colors = ButtonDefaults.buttonColors(
                        containerColor = Color.Red
                    )
                ) {
                    Text("删除")
                }
            },
            dismissButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("取消")
                }
            }
        )
    }
}

// 带输入框的 AlertDialog
@Composable
fun AlertDialogWithInput() {
    var showDialog by remember { mutableStateOf(false) }
    var inputText by remember { mutableStateOf("") }

    if (showDialog) {
        AlertDialog(
            title = { Text("输入名称") },
            text = {
                TextField(
                    value = inputText,
                    onValueChange = { inputText = it },
                    label = { Text("名称") },
                    singleLine = true,
                    modifier = Modifier.fillMaxWidth()
                )
            },
            onDismissRequest = {
                inputText = ""
                showDialog = false
            },
            confirmButton = {
                TextButton(
                    onClick = {
                        // 使用 inputText
                        showDialog = false
                        inputText = ""
                    },
                    enabled = inputText.isNotBlank()
                ) {
                    Text("确定")
                }
            },
            dismissButton = {
                TextButton(onClick = {
                    inputText = ""
                    showDialog = false
                }) {
                    Text("取消")
                }
            }
        )
    }
}

// 带列表选择的 AlertDialog
@Composable
fun AlertDialogWithList() {
    var showDialog by remember { mutableStateOf(false) }
    val options = listOf("选项 1", "选项 2", "选项 3")

    if (showDialog) {
        AlertDialog(
            title = { Text("选择选项") },
            text = {
                Column {
                    options.forEach { option ->
                        Row(
                            modifier = Modifier
                                .fillMaxWidth()
                                .clickable { /* 选择逻辑 */ }
                                .padding(vertical = 12.dp)
                        ) {
                            Text(option)
                        }
                    }
                }
            },
            onDismissRequest = { showDialog = false },
            confirmButton = {
                TextButton(onClick = { showDialog = false }) {
                    Text("确定")
                }
            }
        )
    }
}

// 自定义 AlertDialog 样式
@Composable
fun CustomAlertDialog() {
    var showDialog by remember { mutableStateOf(false) }

    if (showDialog) {
        AlertDialog(
            containerColor = Color.White,
            titleContentColor = Color.Black,
            textContentColor = Color.Gray,
            tonalElevation = 8.dp,
            shape = RoundedCornerShape(16.dp),
            onDismissRequest = { showDialog = false },
            title = {
                Text(
                    "自定义样式",
                    style = MaterialTheme.typography.titleLarge
                )
            },
            text = {
                Text("这是一个样式自定义的对话框")
            },
            confirmButton = {
                Button(onClick = { showDialog = false }) {
                    Text("确定")
                }
            },
            dismissButton = {
                OutlinedButton(onClick = { showDialog = false }) {
                    Text("取消")
                }
            }
        )
    }
}
```

## 2. DropdownMenu 下拉菜单

### 基础下拉菜单

```kotlin
// 基础 DropdownMenu
@Composable
fun BasicDropdownMenu() {
    var expanded by remember { mutableStateOf(false) }
    var selectedText by remember { mutableStateOf("请选择") }

    Box {
        Button(onClick = { expanded = true }) {
            Text(selectedText)
            Icon(Icons.Default.ArrowDropDown, contentDescription = null)
        }

        DropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            DropdownMenuItem(
                text = { Text("选项 1") },
                onClick = {
                    selectedText = "选项 1"
                    expanded = false
                }
            )
            DropdownMenuItem(
                text = { Text("选项 2") },
                onClick = {
                    selectedText = "选项 2"
                    expanded = false
                }
            )
            DropdownMenuItem(
                text = { Text("选项 3") },
                onClick = {
                    selectedText = "选项 3"
                    expanded = false
                }
            )
        }
    }
}

// 带图标的下拉菜单
@Composable
fun DropdownMenuWithIcons() {
    var expanded by remember { mutableStateOf(false) }

    Box {
        IconButton(onClick = { expanded = true }) {
            Icon(Icons.Default.MoreVert, contentDescription = "More")
        }

        DropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            DropdownMenuItem(
                text = { Text("编辑") },
                onClick = { expanded = false },
                leadingIcon = {
                    Icon(Icons.Default.Edit, contentDescription = null)
                }
            )
            DropdownMenuItem(
                text = { Text("分享") },
                onClick = { expanded = false },
                leadingIcon = {
                    Icon(Icons.Default.Share, contentDescription = null)
                }
            )
            DropdownMenuItem(
                text = { Text("删除") },
                onClick = { expanded = false },
                leadingIcon = {
                    Icon(Icons.Default.Delete, contentDescription = null, tint = Color.Red)
                }
            )
        }
    }
}

// 带分隔线的下拉菜单
@Composable
fun DropdownMenuWithDivider() {
    var expanded by remember { mutableStateOf(false) }

    Box {
        IconButton(onClick = { expanded = true }) {
            Icon(Icons.Default.MoreVert, contentDescription = "More")
        }

        DropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            DropdownMenuItem(
                text = { Text("查看") },
                onClick = { expanded = false }
            )
            DropdownMenuItem(
                text = { Text("编辑") },
                onClick = { expanded = false }
            )
            HorizontalDivider()
            DropdownMenuItem(
                text = { Text("删除") },
                onClick = { expanded = false }
            )
        }
    }
}

// 带选中状态的下拉菜单
@Composable
fun DropdownMenuWithSelection() {
    var expanded by remember { mutableStateOf(false) }
    var selectedIndex by remember { mutableStateOf(0) }
    val options = listOf("列表视图", "网格视图", "详情视图")

    Box {
        Button(onClick = { expanded = true }) {
            Text(options[selectedIndex])
        }

        DropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            options.forEachIndexed { index, option ->
                DropdownMenuItem(
                    text = { Text(option) },
                    onClick = {
                        selectedIndex = index
                        expanded = false
                    },
                    trailingIcon = {
                        if (selectedIndex == index) {
                            Icon(
                                Icons.Default.Check,
                                contentDescription = "Selected",
                                tint = Color.Blue
                            )
                        }
                    }
                )
            }
        }
    }
}

// ExposedDropdownMenu（带文本框的下拉菜单）
@Composable
fun ExposedDropdownMenuExample() {
    var expanded by remember { mutableStateOf(false) }
    var selectedValue by remember { mutableStateOf("") }
    val options = listOf("北京", "上海", "广州", "深圳")

    ExposedDropdownMenuBox(
        expanded = expanded,
        onExpandedChange = { expanded = !expanded }
    ) {
        OutlinedTextField(
            value = selectedValue,
            onValueChange = { },
            readOnly = true,
            label = { Text("选择城市") },
            trailingIcon = {
                ExposedDropdownMenuDefaults.TrailingIcon(expanded = expanded)
            },
            modifier = Modifier
                .fillMaxWidth()
                .menuAnchor()
        )

        ExposedDropdownMenu(
            expanded = expanded,
            onDismissRequest = { expanded = false }
        ) {
            options.forEach { option ->
                DropdownMenuItem(
                    text = { Text(option) },
                    onClick = {
                        selectedValue = option
                        expanded = false
                    }
                )
            }
        }
    }
}
```

### 级联下拉菜单

```kotlin
// 二级级联菜单
@Composable
fun CascadingDropdownMenu() {
    var parentExpanded by remember { mutableStateOf(false) }
    var childExpanded by remember { mutableStateOf(false) }

    Box {
        Button(onClick = { parentExpanded = true }) {
            Text("菜单")
        }

        DropdownMenu(
            expanded = parentExpanded,
            onDismissRequest = {
                parentExpanded = false
                childExpanded = false
            }
        ) {
            // 带子菜单的项
            Box {
                DropdownMenuItem(
                    text = { Text("更多操作") },
                    onClick = { childExpanded = true },
                    trailingIcon = {
                        Icon(Icons.Default.ChevronRight, contentDescription = null)
                    }
                )

                // 子菜单
                DropdownMenu(
                    expanded = childExpanded,
                    onDismissRequest = { childExpanded = false }
                ) {
                    DropdownMenuItem(
                        text = { Text("子选项 1") },
                        onClick = { /* 处理 */ }
                    )
                    DropdownMenuItem(
                        text = { Text("子选项 2") },
                        onClick = { /* 处理 */ }
                    )
                }
            }
        }
    }
}
```

## 3. ModalBottomSheet 底部弹窗

### 基础 ModalBottomSheet

```kotlin
// ModalBottomSheet（Material 3）
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BasicBottomSheet() {
    var showBottomSheet by remember { mutableStateOf(false) }
    val sheetState = rememberModalBottomSheetState()

    Column {
        Button(onClick = { showBottomSheet = true }) {
            Text("Show Bottom Sheet")
        }

        if (showBottomSheet) {
            ModalBottomSheet(
                onDismissRequest = { showBottomSheet = false },
                sheetState = sheetState
            ) {
                Column(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp)
                ) {
                    Text(
                        "Bottom Sheet Content",
                        style = MaterialTheme.typography.titleLarge
                    )
                    Spacer(modifier = Modifier.height(16.dp))
                    Text("This is the bottom sheet content")
                    Spacer(modifier = Modifier.height(24.dp))
                    Button(
                        onClick = { showBottomSheet = false },
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Text("Close")
                    }
                }
            }
        }
    }
}

// 带拖拽手柄的 BottomSheet
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun BottomSheetWithHandle() {
    var showBottomSheet by remember { mutableStateOf(false) }
    val sheetState = rememberModalBottomSheetState()

    if (showBottomSheet) {
        ModalBottomSheet(
            onDismissRequest = { showBottomSheet = false },
            sheetState = sheetState,
            dragHandle = {
                Box(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(vertical = 8.dp),
                    contentAlignment = Alignment.Center
                ) {
                    Box(
                        modifier = Modifier
                            .width(40.dp)
                            .height(4.dp)
                            .background(Color.Gray, RoundedCornerShape(2.dp))
                    )
                }
            }
        ) {
            BottomSheetContent(onClose = { showBottomSheet = false })
        }
    }
}

// BottomSheet 内容
@Composable
fun BottomSheetContent(onClose: () -> Unit) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Text(
            "操作菜单",
            style = MaterialTheme.typography.titleLarge,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        // 操作项列表
        BottomSheetItem(
            icon = Icons.Default.Edit,
            text = "编辑",
            onClick = onClose
        )
        BottomSheetItem(
            icon = Icons.Default.Share,
            text = "分享",
            onClick = onClose
        )
        BottomSheetItem(
            icon = Icons.Default.Delete,
            text = "删除",
            onClick = onClose,
            tint = Color.Red
        )

        Spacer(modifier = Modifier.height(16.dp))

        Button(
            onClick = onClose,
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("取消")
        }
    }
}

@Composable
fun BottomSheetItem(
    icon: ImageVector,
    text: String,
    onClick: () -> Unit,
    tint: Color = Color.Unspecified
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(onClick = onClick)
            .padding(vertical = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Icon(
            imageVector = icon,
            contentDescription = null,
            tint = tint,
            modifier = Modifier.padding(end = 16.dp)
        )
        Text(text)
    }
}
```

### BottomSheet 状态控制

```kotlin
// 控制 BottomSheet 状态
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ControlledBottomSheet() {
    var showBottomSheet by remember { mutableStateOf(false) }
    val sheetState = rememberModalBottomSheetState(
        initialValue = SheetValue.Hidden,
        skipPartiallyExpanded = true  // 跳过部分展开状态
    )
    val scope = rememberCoroutineScope()

    Column {
        Button(onClick = {
            showBottomSheet = true
            scope.launch { sheetState.show() }
        }) {
            Text("Show Full Sheet")
        }

        Button(onClick = {
            scope.launch { sheetState.partialExpand() }
        }) {
            Text("Partial Expand")
        }

        Button(onClick = {
            scope.launch { sheetState.hide() }
            showBottomSheet = false
        }) {
            Text("Hide")
        }
    }

    if (showBottomSheet) {
        ModalBottomSheet(
            onDismissRequest = {
                showBottomSheet = false
            },
            sheetState = sheetState
        ) {
            Column(modifier = Modifier.padding(16.dp)) {
                Text("Full height bottom sheet")
            }
        }
    }
}

// BottomSheet 嵌套滚动内容
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ScrollableBottomSheet() {
    var showBottomSheet by remember { mutableStateOf(false) }
    val sheetState = rememberModalBottomSheetState()

    if (showBottomSheet) {
        ModalBottomSheet(
            onDismissRequest = { showBottomSheet = false },
            sheetState = sheetState
        ) {
            LazyColumn(
                modifier = Modifier
                    .fillMaxWidth()
                    .height(400.dp)
            ) {
                items(50) { i ->
                    Text(
                        "Item $i",
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(16.dp)
                    )
                }
            }
        }
    }
}
```

## 4. Toast 封装

### 基础 Toast

```kotlin
// 使用 Snackbar 模拟 Toast
@Composable
fun BasicToastExample() {
    val scope = rememberCoroutineScope()
    val snackbarHostState = remember { SnackbarHostState() }

    Scaffold(
        snackbarHost = { SnackbarHost(snackbarHostState) }
    ) { padding ->
        Column(
            modifier = Modifier
                .padding(padding)
                .padding(16.dp)
        ) {
            Button(
                onClick = {
                    scope.launch {
                        snackbarHostState.showSnackbar(
                            message = "操作成功",
                            duration = SnackbarDuration.Short
                        )
                    }
                }
            ) {
                Text("Show Toast")
            }
        }
    }
}

// SnackbarDuration 选项
// - Short: 约 2 秒
// - Long: 约 4 秒
// - Indefinite: 直到用户操作
```

### 自定义 Toast

```kotlin
// Toast 状态管理
class ToastState {
    private val _toastMessage = MutableStateFlow<String?>(null)
    val toastMessage: StateFlow<String?> = _toastMessage.asStateFlow()

    fun showToast(message: String) {
        _toastMessage.value = message
    }

    fun clearToast() {
        _toastMessage.value = null
    }
}

// 自定义 Toast 组件
@Composable
fun CustomToast(
    message: String?,
    onDismiss: () -> Unit
) {
    if (message != null) {
        // 自动消失
        LaunchedEffect(message) {
            delay(2000)
            onDismiss()
        }

        Box(
            modifier = Modifier
                .fillMaxSize()
                .pointerInput(Unit) {
                    detectTapGestures { onDismiss() }
                },
            contentAlignment = Alignment.BottomCenter
        ) {
            Card(
                modifier = Modifier
                    .padding(bottom = 64.dp)
                    .wrapContentSize(),
                colors = CardDefaults.cardColors(
                    containerColor = Color.Black.copy(0.8f)
                ),
                shape = RoundedCornerShape(8.dp)
            ) {
                Text(
                    text = message,
                    color = Color.White,
                    modifier = Modifier.padding(horizontal = 24.dp, vertical = 12.dp)
                )
            }
        }
    }
}

// 使用自定义 Toast
@Composable
fun ScreenWithCustomToast(viewModel: MyViewModel) {
    val toastMessage by viewModel.toastMessage.collectAsState()

    Box {
        // 主内容
        Column {
            Button(onClick = { viewModel.showToast("操作成功") }) {
                Text("Show Toast")
            }
        }

        // Toast
        CustomToast(
            message = toastMessage,
            onDismiss = { viewModel.clearToast() }
        )
    }
}
```

### Toast 工具类

```kotlin
// 全局 Toast 工具
object ToastUtils {
    private val _toastMessage = MutableStateFlow<String?>(null)
    val toastMessage: StateFlow<String?> = _toastMessage.asStateFlow()

    fun show(message: String) {
        _toastMessage.value = message
    }

    fun show(@StringRes resId: Int) {
        _toastMessage.value = getString(resId)
    }

    fun clear() {
        _toastMessage.value = null
    }
}

// 在 Application 中注册 Toast
@Composable
fun AppTheme(content: @Composable () -> Unit) {
    val toastMessage by ToastUtils.toastMessage.collectAsState()

    MaterialTheme {
        Box {
            content()
            CustomToast(
                message = toastMessage,
                onDismiss = { ToastUtils.clear() }
            )
        }
    }
}

// ViewModel 中使用
class MyViewModel : ViewModel() {
    fun doSomething() {
        // 业务逻辑
        ToastUtils.show("操作成功")
    }
}
```

## 5. 加载弹窗

### 加载 Dialog

```kotlin
// 基础加载 Dialog
@Composable
fun LoadingDialog(isLoading: Boolean) {
    if (isLoading) {
        Dialog(onDismissRequest = { }) {
            Card(
                modifier = Modifier
                    .size(120.dp)
                    .padding(16.dp),
                shape = RoundedCornerShape(16.dp)
            ) {
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
        }
    }
}

// 使用加载 Dialog
@Composable
fun ScreenWithLoading(viewModel: MyViewModel) {
    val isLoading by viewModel.isLoading.collectAsState()

    Box {
        // 主内容
        Column {
            Button(onClick = { viewModel.loadData() }) {
                Text("Load Data")
            }
        }

        // 加载 Dialog
        LoadingDialog(isLoading = isLoading)
    }
}

// 带文字的加载 Dialog
@Composable
fun LoadingDialogWithText(
    isLoading: Boolean,
    message: String = "加载中..."
) {
    if (isLoading) {
        Dialog(onDismissRequest = { }) {
            Card(
                modifier = Modifier
                    .width(150.dp)
                    .padding(16.dp),
                shape = RoundedCornerShape(16.dp)
            ) {
                Column(
                    modifier = Modifier.padding(24.dp),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    CircularProgressIndicator(modifier = Modifier.size(40.dp))
                    Spacer(modifier = Modifier.height(16.dp))
                    Text(
                        text = message,
                        style = MaterialTheme.typography.bodyMedium
                    )
                }
            }
        }
    }
}
```

### 全屏加载

```kotlin
// 全屏加载覆盖层
@Composable
fun FullScreenLoading(isLoading: Boolean) {
    if (isLoading) {
        Box(
            modifier = Modifier
                .fillMaxSize()
                .background(Color.Black.copy(0.5f)),
            contentAlignment = Alignment.Center
        ) {
            Card(
                modifier = Modifier.padding(16.dp),
                shape = RoundedCornerShape(16.dp)
            ) {
                Column(
                    modifier = Modifier.padding(32.dp),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    CircularProgressIndicator()
                    Spacer(modifier = Modifier.height(16.dp))
                    Text("正在加载...")
                }
            }
        }
    }
}

// 使用全屏加载
@Composable
fun ScreenWithFullScreenLoading(viewModel: MyViewModel) {
    val isLoading by viewModel.isLoading.collectAsState()

    Box {
        // 主内容
        MainContent()

        // 全屏加载
        FullScreenLoading(isLoading = isLoading)
    }
}
```

## 快速对照表

| 组件 | 用途 | 关键属性 |
|------|------|----------|
| `Dialog` | 自定义弹窗 | onDismissRequest, properties |
| `AlertDialog` | 确认弹窗 | title, text, confirmButton, dismissButton |
| `DropdownMenu` | 下拉菜单 | expanded, onDismissRequest |
| `DropdownMenuItem` | 菜单项 | text, onClick, leadingIcon |
| `ModalBottomSheet` | 底部弹窗 | onDismissRequest, sheetState, dragHandle |
| `SnackbarHost` | Toast/Snackbar | hostState |

| 弹窗类型 | 适用场景 |
|----------|----------|
| `AlertDialog` | 确认操作、警告提示 |
| `Dialog` | 自定义内容弹窗 |
| `DropdownMenu` | 选项选择、操作菜单 |
| `ModalBottomSheet` | 操作列表、表单输入 |
| `Snackbar` | 轻量提示、Toast 替代 |
| `LoadingDialog` | 加载状态显示 |

**记忆口诀**：
- **Dialog = 自定义弹窗，onDismissRequest 关闭**
- **AlertDialog = 确认弹窗，title + text + buttons**
- **DropdownMenu = 下拉菜单，expanded 控制展开**
- **ModalBottomSheet = 底部弹窗，sheetState 控制状态**
- **Snackbar = Toast 替代，showSnackbar 显示**
- **LoadingDialog = 加载弹窗，isLoading 控制显示**
