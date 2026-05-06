---
name: kotlin-testing
description: 使用 Kotest、MockK、coroutine testing、property-based testing 和 Kover coverage 的 Kotlin 测试模式。遵循 idiomatic Kotlin 实践的 TDD 方法论。
origin: ECC
---

# Kotlin 测试模式

使用 Kotest 和 MockK 遵循 TDD 方法论，编写可靠、可维护的测试的全面 Kotlin 测试模式。

## 何时使用

- 编写新的 Kotlin 函数或类时
- 为现有 Kotlin 代码添加 test coverage 时
- 应用 property-based tests 时
- 在 Kotlin 项目中遵循 TDD 工作流时
- 为 code coverage 配置 Kover 时

## 如何工作

1. **确定目标代码** — 找到要测试的函数、类或模块
2. **编写 Kotest spec** — 选择适合测试范围的 spec 风格（StringSpec、FunSpec、BehaviorSpec）
3. **Mock 依赖项** — 使用 MockK 隔离被测单元
4. **运行测试（RED）** — 验证测试因预期错误而失败
5. **实现代码（GREEN）** — 编写通过测试所需的最少代码
6. **重构** — 在保持测试通过的同时改进实现
7. **检查 Coverage** — 运行 `./gradlew koverHtmlReport` 并验证 80%+ coverage

## Kotlin 的 TDD 工作流

### RED-GREEN-REFACTOR 循环

```
RED     -> 首先编写失败的测试
GREEN   -> 编写使测试通过的最少代码
REFACTOR -> 在保持测试通过的同时改进代码
REPEAT  -> 继续下一个需求
```

### Kotlin 中的分步 TDD

```kotlin
// 步骤 1：定义 Interface/signature
// EmailValidator.kt
package com.example.validator

fun validateEmail(email: String): Result<String> {
    TODO("not implemented")
}

// 步骤 2：编写失败的测试（RED）
// EmailValidatorTest.kt
package com.example.validator

import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.result.shouldBeFailure
import io.kotest.matchers.result.shouldBeSuccess

class EmailValidatorTest : StringSpec({
    "valid email returns success" {
        validateEmail("user@example.com").shouldBeSuccess("user@example.com")
    }

    "empty email returns failure" {
        validateEmail("").shouldBeFailure()
    }

    "email without @ returns failure" {
        validateEmail("userexample.com").shouldBeFailure()
    }
})

// 步骤 3：运行测试 — 验证失败
// $ ./gradlew test
// EmailValidatorTest > valid email returns success FAILED
//   kotlin.NotImplementedError: An operation is not implemented

// 步骤 4：实现最少代码（GREEN）
fun validateEmail(email: String): Result<String> {
    if (email.isBlank()) return Result.failure(IllegalArgumentException("Email cannot be blank"))
    if ('@' !in email) return Result.failure(IllegalArgumentException("Email must contain @"))
    val regex = Regex("^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$")
    if (!regex.matches(email)) return Result.failure(IllegalArgumentException("Invalid email format"))
    return Result.success(email)
}

// 步骤 5：运行测试 — 验证通过
// $ ./gradlew test
// EmailValidatorTest > valid email returns success PASSED
// EmailValidatorTest > empty email returns failure PASSED
// EmailValidatorTest > email without @ returns failure PASSED

// 步骤 6：必要时重构，验证测试仍通过
```

## Kotest Spec 风格

### StringSpec（最简单）

```kotlin
class CalculatorTest : StringSpec({
    "add two positive numbers" {
        Calculator.add(2, 3) shouldBe 5
    }

    "add negative numbers" {
        Calculator.add(-1, -2) shouldBe -3
    }

    "add zero" {
        Calculator.add(0, 5) shouldBe 5
    }
})
```

### FunSpec（类似 JUnit）

```kotlin
class UserServiceTest : FunSpec({
    val repository = mockk<UserRepository>()
    val service = UserService(repository)

    test("getUser returns user when found") {
        val expected = User(id = "1", name = "Alice")
        coEvery { repository.findById("1") } returns expected

        val result = service.getUser("1")

        result shouldBe expected
    }

    test("getUser throws when not found") {
        coEvery { repository.findById("999") } returns null

        shouldThrow<UserNotFoundException> {
            service.getUser("999")
        }
    }
})
```

### BehaviorSpec（BDD 风格）

```kotlin
class OrderServiceTest : BehaviorSpec({
    val repository = mockk<OrderRepository>()
    val paymentService = mockk<PaymentService>()
    val service = OrderService(repository, paymentService)

    Given("a valid order request") {
        val request = CreateOrderRequest(
            userId = "user-1",
            items = listOf(OrderItem("product-1", quantity = 2)),
        )

        When("the order is placed") {
            coEvery { paymentService.charge(any()) } returns PaymentResult.Success
            coEvery { repository.save(any()) } answers { firstArg() }

            val result = service.placeOrder(request)

            Then("it should return a confirmed order") {
                result.status shouldBe OrderStatus.CONFIRMED
            }

            Then("it should charge payment") {
                coVerify(exactly = 1) { paymentService.charge(any()) }
            }
        }

        When("payment fails") {
            coEvery { paymentService.charge(any()) } returns PaymentResult.Declined

            Then("it should throw PaymentException") {
                shouldThrow<PaymentException> {
                    service.placeOrder(request)
                }
            }
        }
    }
})
```

## Kotest Matcher

### 基础 Matcher

```kotlin
import io.kotest.matchers.shouldBe
import io.kotest.matchers.shouldNotBe
import io.kotest.matchers.string.*
import io.kotest.matchers.collections.*
import io.kotest.matchers.nulls.*

// 相等性
result shouldBe expected
result shouldNotBe unexpected

// 字符串
name shouldStartWith "Al"
name shouldEndWith "ice"
name shouldContain "lic"
name shouldMatch Regex("[A-Z][a-z]+")
name.shouldBeBlank()

// 集合
list shouldContain "item"
list shouldHaveSize 3
list.shouldBeSorted()
list.shouldContainAll("a", "b", "c")
list.shouldBeEmpty()

// Null
result.shouldNotBeNull()
result.shouldBeNull()

// 类型
result.shouldBeInstanceOf<User>()

// 数字
count shouldBeGreaterThan 0
price shouldBeInRange 1.0..100.0

// 异常
shouldThrow<IllegalArgumentException> {
    validateAge(-1)
}.message shouldBe "Age must be positive"

shouldNotThrow<Exception> {
    validateAge(25)
}
```

## MockK

### 基础 Mocking

```kotlin
class UserServiceTest : FunSpec({
    val repository = mockk<UserRepository>()
    val logger = mockk<Logger>(relaxed = true) // Relaxed：返回默认值
    val service = UserService(repository, logger)

    beforeTest {
        clearMocks(repository, logger)
    }

    test("findUser delegates to repository") {
        val expected = User(id = "1", name = "Alice")
        every { repository.findById("1") } returns expected

        val result = service.findUser("1")

        result shouldBe expected
        verify(exactly = 1) { repository.findById("1") }
    }

    test("findUser returns null for unknown id") {
        every { repository.findById(any()) } returns null

        val result = service.findUser("unknown")

        result.shouldBeNull()
    }
})
```

### Coroutine Mocking

```kotlin
class AsyncUserServiceTest : FunSpec({
    val repository = mockk<UserRepository>()
    val service = UserService(repository)

    test("getUser suspending function") {
        coEvery { repository.findById("1") } returns User(id = "1", name = "Alice")

        val result = service.getUser("1")

        result.name shouldBe "Alice"
        coVerify { repository.findById("1") }
    }

    test("getUser with delay") {
        coEvery { repository.findById("1") } coAnswers {
            delay(100) // 模拟异步工作
            User(id = "1", name = "Alice")
        }

        val result = service.getUser("1")
        result.name shouldBe "Alice"
    }
})
```

## Coroutine 测试

### 用于 Suspend 函数的 runTest

```kotlin
import kotlinx.coroutines.test.runTest

class CoroutineServiceTest : FunSpec({
    test("concurrent fetches complete together") {
        runTest {
            val service = DataService(testScope = this)

            val result = service.fetchAllData()

            result.users.shouldNotBeEmpty()
            result.products.shouldNotBeEmpty()
        }
    }

    test("timeout after delay") {
        runTest {
            val service = SlowService()

            shouldThrow<TimeoutCancellationException> {
                withTimeout(100) {
                    service.slowOperation() // 耗时 > 100ms
                }
            }
        }
    }
})
```

### Flow 测试

```kotlin
import io.kotest.matchers.collections.shouldContainInOrder
import kotlinx.coroutines.flow.MutableSharedFlow
import kotlinx.coroutines.flow.toList
import kotlinx.coroutines.launch
import kotlinx.coroutines.test.advanceTimeBy
import kotlinx.coroutines.test.runTest

class FlowServiceTest : FunSpec({
    test("observeUsers emits updates") {
        runTest {
            val service = UserFlowService()

            val emissions = service.observeUsers()
                .take(3)
                .toList()

            emissions shouldHaveSize 3
            emissions.last().shouldNotBeEmpty()
        }
    }

    test("searchUsers debounces input") {
        runTest {
            val service = SearchService()
            val queries = MutableSharedFlow<String>()

            val results = mutableListOf<List<User>>()
            val job = launch {
                service.searchUsers(queries).collect { results.add(it) }
            }

            queries.emit("a")
            queries.emit("ab")
            queries.emit("abc") // 只应触发这次搜索
            advanceTimeBy(500)

            results shouldHaveSize 1
            job.cancel()
        }
    }
})
```

## Property-Based Testing

### Kotest Property Testing

```kotlin
import io.kotest.core.spec.style.FunSpec
import io.kotest.property.Arb
import io.kotest.property.arbitrary.*
import io.kotest.property.forAll
import io.kotest.property.checkAll

class PropertyTest : FunSpec({
    test("string reverse is involutory") {
        forAll<String> { s ->
            s.reversed().reversed() == s
        }
    }

    test("list sort is idempotent") {
        forAll(Arb.list(Arb.int())) { list ->
            list.sorted() == list.sorted().sorted()
        }
    }

    test("serialization roundtrip preserves data") {
        checkAll(Arb.bind(Arb.string(1..50), Arb.string(5..100)) { name, email ->
            User(name = name, email = "$email@test.com")
        }) { user ->
            val json = Json.encodeToString(user)
            val decoded = Json.decodeFromString<User>(json)
            decoded shouldBe user
        }
    }
})
```

## Kover Coverage

### Gradle 配置

```kotlin
// build.gradle.kts
plugins {
    id("org.jetbrains.kotlinx.kover") version "0.9.7"
}

kover {
    reports {
        total {
            html { onCheck = true }
            xml { onCheck = true }
        }
        filters {
            excludes {
                classes("*.generated.*", "*.config.*")
            }
        }
        verify {
            rule {
                minBound(80) // coverage 低于 80% 则 build 失败
            }
        }
    }
}
```

### Coverage 命令

```bash
# 运行测试并生成 coverage
./gradlew koverHtmlReport

# 验证 coverage 阈值
./gradlew koverVerify

# 为 CI 生成 XML 报告
./gradlew koverXmlReport

# 查看 HTML 报告（根据你的操作系统使用命令）
# macOS:   open build/reports/kover/html/index.html
# Linux:   xdg-open build/reports/kover/html/index.html
# Windows: start build/reports/kover/html/index.html
```

### Coverage 目标

| 代码类型 | 目标 |
|-----------|--------|
| 关键 business 逻辑 | 100% |
| Public API | 90%+ |
| 一般代码 | 80%+ |
| Generated / config 代码 | 排除 |

## Ktor testApplication 测试

```kotlin
class ApiRoutesTest : FunSpec({
    test("GET /users returns list") {
        testApplication {
            application {
                configureRouting()
                configureSerialization()
            }

            val response = client.get("/users")

            response.status shouldBe HttpStatusCode.OK
            val users = response.body<List<UserResponse>>()
            users.shouldNotBeEmpty()
        }
    }

    test("POST /users creates user") {
        testApplication {
            application {
                configureRouting()
                configureSerialization()
            }

            val response = client.post("/users") {
                contentType(ContentType.Application.Json)
                setBody(CreateUserRequest("Alice", "alice@example.com"))
            }

            response.status shouldBe HttpStatusCode.Created
        }
    }
})
```

## 测试命令

```bash
# 运行所有测试
./gradlew test

# 运行特定的 test class
./gradlew test --tests "com.example.UserServiceTest"

# 运行特定的测试
./gradlew test --tests "com.example.UserServiceTest.getUser returns user when found"

# 运行并输出详细信息
./gradlew test --info

# 运行并生成 coverage
./gradlew koverHtmlReport

# 运行 detekt（静态分析）
./gradlew detekt

# 运行 ktlint（格式检查）
./gradlew ktlintCheck

# 持续测试
./gradlew test --continuous
```

## 最佳实践

**应该做的：**
- 先写测试（TDD）
- 在整个项目中一致使用 Kotest 的 spec 风格
- 对于 suspend 函数使用 MockK 的 `coEvery`/`coVerify`
- 对于 coroutine 测试使用 `runTest`
- 测试行为而非实现
- 对于 pure 函数使用 property-based testing
- 为清晰起见使用 `data class` test fixture

**不应该做的：**
- 不要混合使用 test framework（选择 Kotest 并坚持）
- 不要 mock data class（使用真实实例）
- 不要在 coroutine 测试中使用 `Thread.sleep()`（使用 `advanceTimeBy`）
- 不要跳过 TDD 的 RED 阶段
- 不要直接测试 private 函数
- 不要忽略不稳定的测试

## 与 CI/CD 集成

```yaml
# GitHub Actions 示例
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        distribution: 'temurin'
        java-version: '21'

    - name: Run tests with coverage
      run: ./gradlew test koverXmlReport

    - name: Verify coverage
      run: ./gradlew koverVerify

    - name: Upload coverage
      uses: codecov/codecov-action@v5
      with:
        files: build/reports/kover/report.xml
        token: ${{ secrets.CODECOV_TOKEN }}
```

**记住**：测试就是文档。它们展示了你的 Kotlin 代码应该如何使用。使用 Kotest 的表达性 matcher 使测试可读，并使用 MockK 干净地 mock 依赖项。
