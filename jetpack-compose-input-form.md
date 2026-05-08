# JetPack Compose 输入与表单模块

## 1. TextField 输入框

### 基础 TextField

```kotlin
// 基础 TextField
@Composable
fun BasicTextField() {
    var text by remember { mutableStateOf("") }

    TextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("输入内容") },
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}

// OutlinedTextField（带边框）
@Composable
fun OutlinedTextFieldExample() {
    var text by remember { mutableStateOf("") }

    OutlinedTextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("输入内容") },
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}

// TextField 完整参数
@Composable
fun FullTextField() {
    var text by remember { mutableStateOf("") }

    OutlinedTextField(
        value = text,
        onValueChange = { text = it },
        label = { Text("标签") },
        placeholder = { Text("提示文字") },
        leadingIcon = {
            Icon(Icons.Default.Person, contentDescription = null)
        },
        trailingIcon = {
            if (text.isNotEmpty()) {
                IconButton(onClick = { text = "" }) {
                    Icon(Icons.Default.Clear, contentDescription = "Clear")
                }
            }
        },
        helperText = { Text("辅助说明文字") },
        isError = false,
        singleLine = true,
        maxLines = 1,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}
```

### 密码框

```kotlin
// 密码输入框（带显示/隐藏切换）
@Composable
fun PasswordField() {
    var password by remember { mutableStateOf("") }
    var passwordVisible by remember { mutableStateOf(false) }

    OutlinedTextField(
        value = password,
        onValueChange = { password = it },
        label = { Text("密码") },
        placeholder = { Text("请输入密码") },
        visualTransformation = if (passwordVisible) {
            VisualTransformation.None
        } else {
            PasswordVisualTransformation()
        },
        leadingIcon = {
            Icon(Icons.Default.Lock, contentDescription = null)
        },
        trailingIcon = {
            IconButton(onClick = { passwordVisible = !passwordVisible }) {
                Icon(
                    imageVector = if (passwordVisible) {
                        Icons.Default.Visibility
                    } else {
                        Icons.Default.VisibilityOff
                    },
                    contentDescription = if (passwordVisible) {
                        "隐藏密码"
                    } else {
                        "显示密码"
                    }
                )
            }
        },
        singleLine = true,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}

// 确认密码框（带匹配校验）
@Composable
fun ConfirmPasswordField() {
    var password by remember { mutableStateOf("") }
    var confirmPassword by remember { mutableStateOf("") }
    var passwordVisible by remember { mutableStateOf(false) }

    val isPasswordMatch = remember(password, confirmPassword) {
        password == confirmPassword && confirmPassword.isNotEmpty()
    }

    Column(modifier = Modifier.padding(16.dp)) {
        // 密码
        OutlinedTextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("密码") },
            visualTransformation = if (passwordVisible) {
                VisualTransformation.None
            } else {
                PasswordVisualTransformation()
            },
            trailingIcon = {
                IconButton(onClick = { passwordVisible = !passwordVisible }) {
                    Icon(
                        imageVector = if (passwordVisible) {
                            Icons.Default.Visibility
                        } else {
                            Icons.Default.VisibilityOff
                        },
                        contentDescription = null
                    )
                }
            },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        // 确认密码
        OutlinedTextField(
            value = confirmPassword,
            onValueChange = { confirmPassword = it },
            label = { Text("确认密码") },
            visualTransformation = if (passwordVisible) {
                VisualTransformation.None
            } else {
                PasswordVisualTransformation()
            },
            isError = confirmPassword.isNotEmpty() && !isPasswordMatch,
            helperText = {
                if (confirmPassword.isNotEmpty() && !isPasswordMatch) {
                    Text("密码不匹配", color = Color.Red)
                }
            },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```

### 输入框校验

```kotlin
// 邮箱输入校验
@Composable
fun EmailField() {
    var email by remember { mutableStateOf("") }
    var emailError by remember { mutableStateOf<String?>(null) }

    fun validateEmail(email: String): Boolean {
        return android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
    }

    OutlinedTextField(
        value = email,
        onValueChange = {
            email = it
            emailError = null  // 清除错误
        },
        label = { Text("邮箱") },
        placeholder = { Text("example@email.com") },
        leadingIcon = {
            Icon(Icons.Default.Email, contentDescription = null)
        },
        isError = emailError != null,
        helperText = {
            emailError?.let { error ->
                Text(error, color = Color.Red)
            }
        },
        singleLine = true,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )

    // 失焦时校验
    LaunchedEffect(email) {
        if (email.isNotEmpty() && !validateEmail(email)) {
            emailError = "邮箱格式不正确"
        }
    }
}

// 手机号输入校验
@Composable
fun PhoneField() {
    var phone by remember { mutableStateOf("") }
    var phoneError by remember { mutableStateOf<String?>(null) }

    OutlinedTextField(
        value = phone,
        onValueChange = {
            // 只允许数字
            phone = it.filter { char -> char.isDigit() }
            phoneError = null
        },
        label = { Text("手机号") },
        placeholder = { Text("11 位手机号") },
        leadingIcon = {
            Icon(Icons.Default.Phone, contentDescription = null)
        },
        isError = phoneError != null,
        helperText = {
            phoneError?.let { error ->
                Text(error, color = Color.Red)
            } ?: run {
                Text("${phone.length}/11")
            }
        },
        singleLine = true,
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Phone),
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )

    // 长度校验
    LaunchedEffect(phone) {
        if (phone.isNotEmpty() && phone.length != 11) {
            phoneError = "手机号必须为 11 位"
        } else {
            phoneError = null
        }
    }
}

// 多字段表单校验
@Composable
fun RegistrationForm() {
    var name by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    var nameError by remember { mutableStateOf<String?>(null) }
    var emailError by remember { mutableStateOf<String?>(null) }
    var passwordError by remember { mutableStateOf<String?>(null) }

    var isFormValid by remember {
        derivedStateOf {
            nameError == null && emailError == null && passwordError == null &&
            name.isNotEmpty() && email.isNotEmpty() && password.isNotEmpty()
        }
    }

    Column(modifier = Modifier.padding(16.dp)) {
        // 姓名
        OutlinedTextField(
            value = name,
            onValueChange = {
                name = it
                nameError = if (it.isEmpty()) "姓名不能为空" else null
            },
            label = { Text("姓名") },
            isError = nameError != null,
            helperText = { nameError?.let { Text(it, color = Color.Red) } },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        // 邮箱
        OutlinedTextField(
            value = email,
            onValueChange = {
                email = it
                emailError = if (!isValidEmail(it)) "邮箱格式不正确" else null
            },
            label = { Text("邮箱") },
            isError = emailError != null,
            helperText = { emailError?.let { Text(it, color = Color.Red) } },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        // 密码
        OutlinedTextField(
            value = password,
            onValueChange = {
                password = it
                passwordError = if (it.length < 6) "密码至少 6 位" else null
            },
            label = { Text("密码") },
            isError = passwordError != null,
            helperText = { passwordError?.let { Text(it, color = Color.Red) } },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(24.dp))

        // 提交按钮
        Button(
            onClick = { /* 提交逻辑 */ },
            enabled = isFormValid,
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("注册")
        }
    }
}

fun isValidEmail(email: String): Boolean {
    return android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()
}
```

### 自定义 VisualTransformation

```kotlin
// 信用卡号格式化（4 位一组）
class CreditCardVisualTransformation : VisualTransformation {
    override fun filter(text: AnnotatedString): TransformedText {
        val trimmed = if (text.text.length >= 16) {
            text.text.substring(0..15)
        } else {
            text.text
        }

        var output = ""
        for (i in trimmed.indices) {
            output += trimmed[i]
            if ((i + 1) % 4 == 0 && i != 15) {
                output += " "
            }
        }

        return TransformedText(
            AnnotatedString(output),
            OffsetMapping.Identity
        )
    }
}

// 使用
@Composable
fun CreditCardField() {
    var cardNumber by remember { mutableStateOf("") }

    OutlinedTextField(
        value = cardNumber,
        onValueChange = {
            // 只允许数字
            cardNumber = it.filter { char -> char.isDigit() }
        },
        label = { Text("信用卡号") },
        visualTransformation = CreditCardVisualTransformation(),
        keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Number),
        singleLine = true,
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    )
}
```

## 2. 单选 RadioButton

### 基础 RadioButton

```kotlin
// 基础单选
@Composable
fun BasicRadioButton() {
    var selectedOption by remember { mutableStateOf("选项 1") }
    val options = listOf("选项 1", "选项 2", "选项 3")

    Column(modifier = Modifier.padding(16.dp)) {
        options.forEach { option ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable { selectedOption = option }
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                RadioButton(
                    selected = selectedOption == option,
                    onClick = { selectedOption = option }
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(option)
            }
        }
    }
}

// 带标题的单选组
@Composable
fun RadioButtonGroup() {
    var selectedOption by remember { mutableStateOf<Int?>(null) }
    val options = listOf(
        Pair(1, "经济舱"),
        Pair(2, "商务舱"),
        Pair(3, "头等舱")
    )

    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "选择舱位",
            style = MaterialTheme.typography.titleMedium,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        options.forEach { (value, label) ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable { selectedOption = value }
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                RadioButton(
                    selected = selectedOption == value,
                    onClick = { selectedOption = value }
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(label)
            }
        }
    }
}
```

### 带样式的 RadioButton

```kotlin
// 卡片式单选
@Composable
fun CardRadioButton() {
    var selectedOption by remember { mutableStateOf(0) }
    val options = listOf(
        Triple(0, Icons.Default.Storage, "标准版"),
        Triple(1, Icons.Default.Speed, "专业版"),
        Triple(2, Icons.Default.Star, "企业版")
    )

    Column(modifier = Modifier.padding(16.dp)) {
        options.forEach { (value, icon, label) ->
            Card(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(vertical = 4.dp)
                    .clickable { selectedOption = value },
                colors = CardDefaults.cardColors(
                    containerColor = if (selectedOption == value) {
                        Color.Blue.copy(0.1f)
                    } else {
                        Color.White
                    }
                ),
                elevation = CardDefaults.cardElevation(
                    defaultElevation = if (selectedOption == value) 4.dp else 1.dp
                )
            ) {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(16.dp),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Icon(
                        imageVector = icon,
                        contentDescription = null,
                        tint = if (selectedOption == value) Color.Blue else Color.Gray
                    )
                    Spacer(modifier = Modifier.width(16.dp))
                    Text(
                        text = label,
                        modifier = Modifier.weight(1f)
                    )
                    RadioButton(
                        selected = selectedOption == value,
                        onClick = { selectedOption = value }
                    )
                }
            }
        }
    }
}
```

## 3. 复选 CheckBox

### 基础 CheckBox

```kotlin
// 基础复选
@Composable
fun BasicCheckBox() {
    var checked by remember { mutableStateOf(false) }

    Row(
        modifier = Modifier
            .clickable { checked = !checked }
            .padding(16.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        CheckBox(
            checked = checked,
            onCheckedChange = { checked = it }
        )
        Spacer(modifier = Modifier.width(8.dp))
        Text("同意条款和条件")
    }
}

// 多选列表
@Composable
fun CheckBoxList() {
    val items = listOf("苹果", "香蕉", "橙子", "葡萄")
    var checkedItems by remember { mutableStateOf(setOf<String>()) }

    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "选择水果",
            style = MaterialTheme.typography.titleMedium,
            modifier = Modifier.padding(bottom = 16.dp)
        )

        items.forEach { item ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable {
                        checkedItems = if (item in checkedItems) {
                            checkedItems - item
                        } else {
                            checkedItems + item
                        }
                    }
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                CheckBox(
                    checked = item in checkedItems,
                    onCheckedChange = { isChecked ->
                        checkedItems = if (isChecked) {
                            checkedItems + item
                        } else {
                            checkedItems - item
                        }
                    }
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(item)
            }
        }

        Spacer(modifier = Modifier.height(16.dp))

        Text("已选择：${checkedItems.joinToString()}")
    }
}

// 带状态的 CheckBox（三态）
@Composable
fun TriStateCheckBox() {
    val items = listOf("项目 1", "项目 2", "项目 3")
    var checkedItems by remember { mutableStateOf(setOf<String>()) }

    // 计算父级选中状态
    val parentChecked = when {
        checkedItems.isEmpty() -> false
        checkedItems.size == items.size -> true
        else -> null  // 部分选中
    }

    Column(modifier = Modifier.padding(16.dp)) {
        // 全选
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .clickable {
                    checkedItems = if (parentChecked == true) {
                        emptySet()
                    } else {
                        items.toSet()
                    }
                }
                .padding(vertical = 8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            CheckBox(
                checked = parentChecked ?: false,
                onCheckedChange = null,  // null 表示三态
                colors = CheckboxDefaults.colors(
                    checkmarkColor = if (parentChecked == null) {
                        Color.Gray
                    } else {
                        Color.Unspecified
                    }
                )
            )
            Spacer(modifier = Modifier.width(8.dp))
            Text("全选")
        }

        Divider()

        // 子项
        items.forEach { item ->
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .clickable {
                        checkedItems = if (item in checkedItems) {
                            checkedItems - item
                        } else {
                            checkedItems + item
                        }
                    }
                    .padding(vertical = 8.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                CheckBox(
                    checked = item in checkedItems,
                    onCheckedChange = { isChecked ->
                        checkedItems = if (isChecked) {
                            checkedItems + item
                        } else {
                            checkedItems - item
                        }
                    }
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(item)
            }
        }
    }
}
```

## 4. 开关 Switch

### 基础 Switch

```kotlin
// 基础开关
@Composable
fun BasicSwitch() {
    var checked by remember { mutableStateOf(false) }

    Row(
        modifier = Modifier
            .clickable { checked = !checked }
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text("开启通知")
        Switch(
            checked = checked,
            onCheckedChange = { checked = it }
        )
    }
}

// 带图标的开关
@Composable
fun SwitchWithIcon() {
    var checked by remember { mutableStateOf(false) }

    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable { checked = !checked }
            .padding(16.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Row(verticalAlignment = Alignment.CenterVertically) {
            Icon(
                imageVector = if (checked) Icons.Default.Notifications else Icons.Default.NotificationsOff,
                contentDescription = null,
                tint = if (checked) Color.Blue else Color.Gray
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text("通知")
                Text(
                    text = if (checked) "已开启" else "已关闭",
                    style = MaterialTheme.typography.bodySmall,
                    color = Color.Gray
                )
            }
        }
        Switch(
            checked = checked,
            onCheckedChange = { checked = it }
        )
    }
}

// 设置列表中的开关
@Composable
fun SettingsSwitchList() {
    var wifi by remember { mutableStateOf(true) }
    var bluetooth by remember { mutableStateOf(false) }
    var location by remember { mutableStateOf(true) }

    Column(modifier = Modifier.padding(16.dp)) {
        SettingSwitchRow(
            icon = Icons.Default.Wifi,
            title = "Wi-Fi",
            subtitle = "无线网络连接",
            checked = wifi,
            onCheckedChange = { wifi = it }
        )

        SettingSwitchRow(
            icon = Icons.Default.Bluetooth,
            title = "蓝牙",
            subtitle = "蓝牙设备连接",
            checked = bluetooth,
            onCheckedChange = { bluetooth = it }
        )

        SettingSwitchRow(
            icon = Icons.Default.LocationOn,
            title = "定位服务",
            subtitle = "允许应用访问位置",
            checked = location,
            onCheckedChange = { location = it }
        )
    }
}

@Composable
fun SettingSwitchRow(
    icon: ImageVector,
    title: String,
    subtitle: String,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable { onCheckedChange(!checked) }
            .padding(vertical = 12.dp),
        horizontalArrangement = Arrangement.SpaceBetween,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Row(
            modifier = Modifier.weight(1f),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Icon(
                imageVector = icon,
                contentDescription = null,
                tint = if (checked) Color.Blue else Color.Gray,
                modifier = Modifier.size(24.dp)
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text(title)
                Text(
                    text = subtitle,
                    style = MaterialTheme.typography.bodySmall,
                    color = Color.Gray
                )
            }
        }
        Switch(
            checked = checked,
            onCheckedChange = onCheckedChange
        )
    }
}
```

## 5. 滑块 Slider

### 基础 Slider

```kotlin
// 基础滑块
@Composable
fun BasicSlider() {
    var sliderPosition by remember { mutableStateOf(50f) }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("音量：${sliderPosition.toInt()}%")
        Slider(
            value = sliderPosition,
            onValueChange = { sliderPosition = it },
            valueRange = 0f..100f,
            modifier = Modifier.fillMaxWidth()
        )
    }
}

// 带步长的滑块
@Composable
fun SliderWithSteps() {
    var sliderPosition by remember { mutableStateOf(3f) }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("亮度：${sliderPosition.toInt()}")
        Slider(
            value = sliderPosition,
            onValueChange = { sliderPosition = it },
            valueRange = 1f..5f,
            steps = 4,  // 分成 5 档
            modifier = Modifier.fillMaxWidth()
        )
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.SpaceBetween
        ) {
            Text("1")
            Text("2")
            Text("3")
            Text("4")
            Text("5")
        }
    }
}

// 范围滑块（双滑块）
@Composable
fun RangeSlider() {
    var range by remember { mutableStateOf(20f..80f) }

    Column(modifier = Modifier.padding(16.dp)) {
        Text("价格范围：¥${range.start.toInt()} - ¥${range.endInclusive.toInt()}")
        RangeSlider(
            value = range,
            onValueChange = { range = it },
            valueRange = 0f..100f,
            modifier = Modifier.fillMaxWidth()
        )
    }
}

// 自定义颜色滑块
@Composable
fun CustomColorSlider() {
    var sliderPosition by remember { mutableStateOf(50f) }

    Column(modifier = Modifier.padding(16.dp)) {
        Slider(
            value = sliderPosition,
            onValueChange = { sliderPosition = it },
            valueRange = 0f..100f,
            colors = SliderDefaults.colors(
                thumbColor = Color.Blue,
                activeTrackColor = Color.Blue,
                inactiveTrackColor = Color.Blue.copy(0.3f)
            ),
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```

## 6. 表单双向绑定

### ViewModel 双向绑定

```kotlin
// ViewModel
class LoginFormViewModel : ViewModel() {
    private val _username = MutableStateFlow("")
    val username: StateFlow<String> = _username.asStateFlow()

    private val _password = MutableStateFlow("")
    val password: StateFlow<String> = _password.asStateFlow()

    private val _isValid = MutableStateFlow(false)
    val isValid: StateFlow<Boolean> = _isValid.asStateFlow()

    fun updateUsername(username: String) {
        _username.value = username
        validateForm()
    }

    fun updatePassword(password: String) {
        _password.value = password
        validateForm()
    }

    private fun validateForm() {
        _isValid.value = _username.value.length >= 3 && _password.value.length >= 6
    }

    fun submit() {
        if (_isValid.value) {
            // 提交逻辑
        }
    }
}

// Compose 中使用
@Composable
fun LoginForm(viewModel: LoginFormViewModel = viewModel()) {
    val username by viewModel.username.collectAsState()
    val password by viewModel.password.collectAsState()
    val isValid by viewModel.isValid.collectAsState()

    Column(modifier = Modifier.padding(16.dp)) {
        OutlinedTextField(
            value = username,
            onValueChange = { viewModel.updateUsername(it) },
            label = { Text("用户名") },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(16.dp))

        OutlinedTextField(
            value = password,
            onValueChange = { viewModel.updatePassword(it) },
            label = { Text("密码") },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(
            onClick = { viewModel.submit() },
            enabled = isValid,
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("登录")
        }
    }
}
```

### 输入防抖

```kotlin
// 防抖搜索输入
@Composable
fun SearchField(viewModel: SearchViewModel) {
    var query by rememberSaveable { mutableStateOf("") }
    val searchResults by viewModel.searchResults.collectAsState()

    // 防抖：300ms 无新输入才执行搜索
    LaunchedEffect(query) {
        snapshotFlow { query }
            .debounce(300)
            .collect { searchedQuery ->
                if (searchedQuery.length >= 2) {
                    viewModel.search(searchedQuery)
                }
            }
    }

    Column(modifier = Modifier.padding(16.dp)) {
        OutlinedTextField(
            value = query,
            onValueChange = { query = it },
            label = { Text("搜索") },
            leadingIcon = {
                Icon(Icons.Default.Search, contentDescription = null)
            },
            trailingIcon = {
                if (query.isNotEmpty()) {
                    IconButton(onClick = { query = "" }) {
                        Icon(Icons.Default.Clear, contentDescription = "Clear")
                    }
                }
            },
            singleLine = true,
            modifier = Modifier.fillMaxWidth()
        )

        // 搜索结果显示
        searchResults.forEach { result ->
            Text(result, modifier = Modifier.padding(vertical = 4.dp))
        }
    }
}

// ViewModel
class SearchViewModel : ViewModel() {
    private val _searchResults = MutableStateFlow<List<String>>(emptyList())
    val searchResults: StateFlow<List<String>> = _searchResults.asStateFlow()

    fun search(query: String) {
        viewModelScope.launch {
            // 模拟网络搜索
            _searchResults.value = repository.search(query)
        }
    }
}

// 防抖工具函数
object DebounceUtils {
    fun <T> Flow<T>.debounce(
        timeoutMillis: Long,
        scope: CoroutineScope
    ): StateFlow<T?> {
        return this
            .debounce(timeoutMillis)
            .stateIn(scope, SharingStarted.Lazily, null)
    }
}

// 使用防抖工具
@Composable
fun DebouncedSearchField() {
    var query by rememberSaveable { mutableStateOf("") }
    val scope = rememberCoroutineScope()

    val debouncedQuery = remember {
        snapshotFlow { query }
            .debounce(300)
            .stateIn(scope, SharingStarted.Lazily, "")
    }

    val debouncedValue by debouncedQuery.collectAsState()

    // 使用 debouncedValue 进行搜索
}
```

## 快速对照表

| 组件 | 用途 | 关键属性 |
|------|------|----------|
| `TextField` | 输入框（填充式） | value, onValueChange, label |
| `OutlinedTextField` | 输入框（边框式） | value, onValueChange, label |
| `PasswordVisualTransformation` | 密码隐藏 | visualTransformation |
| `RadioButton` | 单选 | selected, onClick |
| `CheckBox` | 复选 | checked, onCheckedChange |
| `Switch` | 开关 | checked, onCheckedChange |
| `Slider` | 滑块 | value, onValueChange, valueRange |
| `RangeSlider` | 范围滑块 | value, onValueChange |

| 校验类型 | 实现方式 |
|----------|----------|
| 邮箱校验 | `Patterns.EMAIL_ADDRESS` |
| 手机号校验 | 长度 + 数字过滤 |
| 密码强度 | 长度 + 字符类型检查 |
| 表单验证 | `derivedStateOf` 组合状态 |

| 优化技术 | 用途 |
|----------|------|
| `debounce(300)` | 输入防抖 |
| `snapshotFlow { }` | State 转 Flow |
| `derivedStateOf { }` | 派生状态 |
| `rememberSaveable` | 配置变更保持 |

**记忆口诀**：
- **TextField = 输入框，OutlinedTextField = 带边框**
- **PasswordVisualTransformation = 密码隐藏**
- **RadioButton = 单选，CheckBox = 复选**
- **Switch = 开关，Slider = 滑块**
- **双向绑定 = StateFlow + collectAsState**
- **输入防抖 = debounce(300) + snapshotFlow**
