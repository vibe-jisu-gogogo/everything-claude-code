---
name: kotlin-patterns
description: Idiomatic Kotlin 模式、最佳实践和约定，用于使用 coroutines、null safety 和 DSL builders 创建健壮、高效和可维护的 Kotlin 应用程序。
origin: ECC
---

# Kotlin 开发模式

用于创建健壮、高效和可维护应用程序的 Idiomatic Kotlin 模式和最佳实践。

## 何时使用

- 编写新的 Kotlin 代码时
- 审查 Kotlin 代码时
- 重构现有 Kotlin 代码时
- 设计 Kotlin 模块或库时
- 配置 Gradle Kotlin DSL builds 时

## 工作原理

本技能在七个核心领域应用 idiomatic Kotlin 约定：使用类型系统和 safe-call 操作符的 null safety，通过 `val` 和 data class 中的 `copy()` 实现 immutability，用于穷举类型层次结构的 sealed class 和 interface，通过 coroutines 和 `Flow` 实现结构化并发，用于在不使用继承的情况下添加行为的 extension 函数，使用 `@DslMarker` 和 lambda receiver 实现类型安全的 DSL builders，以及用于构建配置的 Gradle Kotlin DSL。

## 示例

**使用 Elvis 操作符的 null safety：**
```kotlin
fun getUserEmail(userId: String): String {
    val user = userRepository.findById(userId)
    return user?.email ?: "unknown@example.com"
}
```

**用于穷举结果的 sealed class：**
```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failure(val error: AppError) : Result<Nothing>()
    data object Loading : Result<Nothing>()
}
```

**使用 async/await 的结构化并发：**
```kotlin
suspend fun fetchUserWithPosts(userId: String): UserProfile =
    coroutineScope {
        val user = async { userService.getUser(userId) }
        val posts = async { postService.getUserPosts(userId) }
        UserProfile(user = user.await(), posts = posts.await())
    }
```

## 核心原则

### 1. Null Safety

Kotlin 的类型系统区分 nullable 和 non-nullable 类型。精确使用它们。

```kotlin
// 好：默认使用 non-nullable 类型
fun getUser(id: String): User {
    return userRepository.findById(id)
        ?: throw UserNotFoundException("User $id not found")
}

// 好：Safe calls 和 Elvis 操作符
fun getUserEmail(userId: String): String {
    val user = userRepository.findById(userId)
    return user?.email ?: "unknown@example.com"
}

// 坏：强制展开 nullable 类型
fun getUserEmail(userId: String): String {
    val user = userRepository.findById(userId)
    return user!!.email // null 时会抛出 NPE
}
```

### 2. 默认使用 Immutability

优先使用 `val` 而非 `var`，使用 immutable 集合而非 mutable 集合。

```kotlin
// 好：Immutable 数据
data class User(
    val id: String,
    val name: String,
    val email: String,
)

// 好：使用 copy() 进行转换
fun updateEmail(user: User, newEmail: String): User =
    user.copy(email = newEmail)

// 好：Immutable 集合
val users: List<User> = listOf(user1, user2)
val filtered = users.filter { it.email.isNotBlank() }

// 坏：Mutable state
var currentUser: User? = null // 避免 mutable global state
val mutableUsers = mutableListOf<User>() // 除非真正必要否则避免
```

### 3. Expression Body 和单表达式函数

对于简短、可读的函数使用 expression body。

```kotlin
// 好：Expression body
fun isAdult(age: Int): Boolean = age >= 18

fun formatFullName(first: String, last: String): String =
    "$first $last".trim()

fun User.displayName(): String =
    name.ifBlank { email.substringBefore('@') }

// 好：when 作为 expression
fun statusMessage(code: Int): String = when (code) {
    200 -> "OK"
    404 -> "Not Found"
    500 -> "Internal Server Error"
    else -> "Unknown status: $code"
}

// 坏：不必要的 block body
fun isAdult(age: Int): Boolean {
    return age >= 18
}
```

### 4. 用于 Value 对象的 Data Class

对于主要保存数据的类型优先使用 data class。

```kotlin
// 好：带有 copy、equals、hashCode、toString 的 data class
data class CreateUserRequest(
    val name: String,
    val email: String,
    val role: Role = Role.USER,
)

// 好：用于类型安全的 value class（运行时零开销）
@JvmInline
value class UserId(val value: String) {
    init {
        require(value.isNotBlank()) { "UserId cannot be blank" }
    }
}

@JvmInline
value class Email(val value: String) {
    init {
        require('@' in value) { "Invalid email: $value" }
    }
}

fun getUser(id: UserId): User = userRepository.findById(id)
```

## Sealed Class 和 Interface

### 建模受限层次结构

```kotlin
// 好：用于穷举 when 的 sealed class
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Failure(val error: AppError) : Result<Nothing>()
    data object Loading : Result<Nothing>()
}

fun <T> Result<T>.getOrNull(): T? = when (this) {
    is Result.Success -> data
    is Result.Failure -> null
    is Result.Loading -> null
}

fun <T> Result<T>.getOrThrow(): T = when (this) {
    is Result.Success -> data
    is Result.Failure -> throw error.toException()
    is Result.Loading -> throw IllegalStateException("Still loading")
}
```

### 用于 API 响应的 Sealed Interface

```kotlin
sealed interface ApiError {
    val message: String

    data class NotFound(override val message: String) : ApiError
    data class Unauthorized(override val message: String) : ApiError
    data class Validation(
        override val message: String,
        val field: String,
    ) : ApiError
    data class Internal(
        override val message: String,
        val cause: Throwable? = null,
    ) : ApiError
}

fun ApiError.toStatusCode(): Int = when (this) {
    is ApiError.NotFound -> 404
    is ApiError.Unauthorized -> 401
    is ApiError.Validation -> 422
    is ApiError.Internal -> 500
}
```

## Scope 函数

### 每个函数的使用时机

```kotlin
// let：转换 nullable 或 scoped 结果
val length: Int? = name?.let { it.trim().length }

// apply：配置对象（返回对象本身）
val user = User().apply {
    name = "Alice"
    email = "alice@example.com"
}

// also：副作用（返回对象本身）
val user = createUser(request).also { logger.info("Created user: ${it.id}") }

// run：使用 receiver 运行 block（返回结果）
val result = connection.run {
    prepareStatement(sql)
    executeQuery()
}

// with：run 的非 extension 形式
val csv = with(StringBuilder()) {
    appendLine("name,email")
    users.forEach { appendLine("${it.name},${it.email}") }
    toString()
}
```

## Extension 函数

### 不使用继承添加功能

```kotlin
// 好：特定于领域的 extension
fun String.toSlug(): String =
    lowercase()
        .replace(Regex("[^a-z0-9\\s-]"), "")
        .replace(Regex("\\s+"), "-")
        .trim('-')

fun Instant.toLocalDate(zone: ZoneId = ZoneId.systemDefault()): LocalDate =
    atZone(zone).toLocalDate()

// 好：集合 extension
fun <T> List<T>.second(): T = this[1]

fun <T> List<T>.secondOrNull(): T? = getOrNull(1)

// 好：Scoped extension（不污染全局命名空间）
class UserService {
    private fun User.isActive(): Boolean =
        status == Status.ACTIVE && lastLogin.isAfter(Instant.now().minus(30, ChronoUnit.DAYS))

    fun getActiveUsers(): List<User> = userRepository.findAll().filter { it.isActive() }
}
```

## Coroutines

### 结构化并发

```kotlin
// 好：使用 coroutineScope 的结构化并发
suspend fun fetchUserWithPosts(userId: String): UserProfile =
    coroutineScope {
        val userDeferred = async { userService.getUser(userId) }
        val postsDeferred = async { postService.getUserPosts(userId) }

        UserProfile(
            user = userDeferred.await(),
            posts = postsDeferred.await(),
        )
    }

// 好：当 child 可以独立失败时使用 supervisorScope
suspend fun fetchDashboard(userId: String): Dashboard =
    supervisorScope {
        val user = async { userService.getUser(userId) }
        val notifications = async { notificationService.getRecent(userId) }
        val recommendations = async { recommendationService.getFor(userId) }

        Dashboard(
            user = user.await(),
            notifications = try {
                notifications.await()
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                emptyList()
            },
            recommendations = try {
                recommendations.await()
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                emptyList()
            },
        )
    }
```

### 用于 Reactive Stream 的 Flow

```kotlin
// 好：带有适当错误处理的 cold flow
fun observeUsers(): Flow<List<User>> = flow {
    while (currentCoroutineContext().isActive) {
        val users = userRepository.findAll()
        emit(users)
        delay(5.seconds)
    }
}.catch { e ->
    logger.error("Error observing users", e)
    emit(emptyList())
}

// 好：Flow 操作符
fun searchUsers(query: Flow<String>): Flow<List<User>> =
    query
        .debounce(300.milliseconds)
        .distinctUntilChanged()
        .filter { it.length >= 2 }
        .mapLatest { q -> userRepository.search(q) }
        .catch { emit(emptyList()) }
```

## DSL Builders

### 类型安全的 Builders

```kotlin
// 好：使用 @DslMarker 的 DSL
@DslMarker
annotation class HtmlDsl

@HtmlDsl
class HTML {
    private val children = mutableListOf<Element>()

    fun head(init: Head.() -> Unit) {
        children += Head().apply(init)
    }

    fun body(init: Body.() -> Unit) {
        children += Body().apply(init)
    }

    override fun toString(): String = children.joinToString("\n")
}

fun html(init: HTML.() -> Unit): HTML = HTML().apply(init)

// 使用
val page = html {
    head { title("My Page") }
    body {
        h1("Welcome")
        p("Hello, World!")
    }
}
```

## Gradle Kotlin DSL

### build.gradle.kts 配置

```kotlin
// 检查最新版本：https://kotlinlang.org/docs/releases.html
plugins {
    kotlin("jvm") version "2.3.10"
    kotlin("plugin.serialization") version "2.3.10"
    id("io.ktor.plugin") version "3.4.0"
    id("org.jetbrains.kotlinx.kover") version "0.9.7"
    id("io.gitlab.arturbosch.detekt") version "1.23.8"
}

group = "com.example"
version = "1.0.0"

kotlin {
    jvmToolchain(21)
}

dependencies {
    // Ktor
    implementation("io.ktor:ktor-server-core:3.4.0")
    implementation("io.ktor:ktor-server-netty:3.4.0")
    implementation("io.ktor:ktor-server-content-negotiation:3.4.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:3.4.0")

    // Exposed
    implementation("org.jetbrains.exposed:exposed-core:1.0.0")
    implementation("org.jetbrains.exposed:exposed-dao:1.0.0")
    implementation("org.jetbrains.exposed:exposed-jdbc:1.0.0")
    implementation("org.jetbrains.exposed:exposed-kotlin-datetime:1.0.0")

    // Koin
    implementation("io.insert-koin:koin-ktor:4.2.0")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.10.2")

    // Test
    testImplementation("io.kotest:kotest-runner-junit5:6.1.4")
    testImplementation("io.kotest:kotest-assertions-core:6.1.4")
    testImplementation("io.kotest:kotest-property:6.1.4")
    testImplementation("io.mockk:mockk:1.14.9")
    testImplementation("io.ktor:ktor-server-test-host:3.4.0")
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.2")
}

tasks.withType<Test> {
    useJUnitPlatform()
}

detekt {
    config.setFrom(files("config/detekt/detekt.yml"))
    buildUponDefaultConfig = true
}
```

## 错误处理模式

### 用于领域操作的 Result 类型

```kotlin
// 好：使用 Kotlin 的 Result 或自定义 sealed class
suspend fun createUser(request: CreateUserRequest): Result<User> = runCatching {
    require(request.name.isNotBlank()) { "Name cannot be blank" }
    require('@' in request.email) { "Invalid email format" }

    val user = User(
        id = UserId(UUID.randomUUID().toString()),
        name = request.name,
        email = Email(request.email),
    )
    userRepository.save(user)
    user
}

// 好：链式调用 Result
val displayName = createUser(request)
    .map { it.name }
    .getOrElse { "Unknown" }
```

### require, check, error

```kotlin
// 好：带有清晰消息的前置条件
fun withdraw(account: Account, amount: Money): Account {
    require(amount.value > 0) { "Amount must be positive: $amount" }
    check(account.balance >= amount) { "Insufficient balance: ${account.balance} < $amount" }

    return account.copy(balance = account.balance - amount)
}
```

## 快速参考：Kotlin 习语

| 习语 | 描述 |
|-------|-------------|
| `val` over `var` | 优先使用 immutable 变量 |
| `data class` | 用于带有 equals/hashCode/copy 的 value 对象 |
| `sealed class/interface` | 用于受限类型层次结构 |
| `value class` | 用于零开销类型安全包装器 |
| Expression `when` | 穷举 pattern matching |
| Safe call `?.` | Null-safe 成员访问 |
| Elvis `?:` | 用于 nullable 的默认值 |
| `let`/`apply`/`also`/`run`/`with` | 用于简洁代码的 scope 函数 |
| Extension 函数 | 不使用继承添加行为 |
| `copy()` | Data class 中的 immutable 更新 |
| `require`/`check` | 前置条件断言 |
| Coroutine `async`/`await` | 结构化并发执行 |
| `Flow` | Cold reactive streams |
| `sequence` | 惰性求值 |
| Delegation `by` | 不使用继承重用实现 |

## 应避免的反模式

```kotlin
// 坏：强制展开 nullable 类型
val name = user!!.name

// 坏：来自 Java 的平台类型泄漏
fun getLength(s: String) = s.length // 安全
fun getLength(s: String?) = s?.length ?: 0 // 处理来自 Java 的 null

// 坏：Mutable data class
data class MutableUser(var name: String, var email: String)

// 坏：使用 exception 进行控制流
try {
    val user = findUser(id)
} catch (e: NotFoundException) {
    // 不要对预期情况使用 exception
}

// 好：返回 nullable 或使用 Result
val user: User? = findUserOrNull(id)

// 坏：忽略 coroutine scope
GlobalScope.launch { /* 避免 GlobalScope */ }

// 好：使用结构化并发
coroutineScope {
    launch { /* 适当 scoped */ }
}

// 坏：深层嵌套的 scope 函数
user?.let { u ->
    u.address?.let { a ->
        a.city?.let { c -> process(c) }
    }
}

// 好：直接的 null-safe 链式调用
user?.address?.city?.let { process(it) }
```

**记住**：Kotlin 代码应该简洁但可读。利用类型系统确保安全，优先使用 immutability，并使用 coroutines 进行并发处理。如有疑问，让编译器帮助您。
