---
name: springboot-verification
description: "Verification loop for Spring Boot projects: build, static analysis, tests with coverage, security scans, and diff review before release or PR."
origin: ECC
---

# Spring Boot 验证循环

在 PR 之前、重大变更后以及部署前运行。

## 何时激活

- 在为 Spring Boot 服务创建 pull request 之前
- 在重大重构或依赖升级之后
- 在 staging 或 production 部署之前进行验证
- 运行完整的 build → lint → test → 安全扫描 pipeline
- 验证测试覆盖率是否达到阈值

## 阶段 1: Build

```bash
mvn -T 4 clean verify -DskipTests
# 或
./gradlew clean assemble -x test
```

如果 build 失败，停止并修复。

## 阶段 2: Static Analysis

Maven (常用插件):
```bash
mvn -T 4 spotbugs:check pmd:check checkstyle:check
```

Gradle (如果已配置):
```bash
./gradlew checkstyleMain pmdMain spotbugsMain
```

## 阶段 3: Tests + Coverage

```bash
mvn -T 4 test
mvn jacoco:report   # 验证 80%+ 覆盖率
# 或
./gradlew test jacocoTestReport
```

报告:
- 总测试数，通过/失败
- 覆盖率 % (行/分支)

### Unit Tests

使用 Mock 依赖隔离测试服务逻辑:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

  @Mock private UserRepository userRepository;
  @InjectMocks private UserService userService;

  @Test
  void createUser_validInput_returnsUser() {
    var dto = new CreateUserDto("Alice", "alice@example.com");
    var expected = new User(1L, "Alice", "alice@example.com");
    when(userRepository.save(any(User.class))).thenReturn(expected);

    var result = userService.create(dto);

    assertThat(result.name()).isEqualTo("Alice");
    verify(userRepository).save(any(User.class));
  }

  @Test
  void createUser_duplicateEmail_throwsException() {
    var dto = new CreateUserDto("Alice", "existing@example.com");
    when(userRepository.existsByEmail(dto.email())).thenReturn(true);

    assertThatThrownBy(() -> userService.create(dto))
        .isInstanceOf(DuplicateEmailException.class);
  }
}
```

### 集成测试 Testcontainers

针对真实数据库测试，而不是 H2:

```java
@SpringBootTest
@Testcontainers
class UserRepositoryIntegrationTest {

  @Container
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine")
      .withDatabaseName("testdb");

  @DynamicPropertySource
  static void configureProperties(DynamicPropertyRegistry registry) {
    registry.add("spring.datasource.url", postgres::getJdbcUrl);
    registry.add("spring.datasource.username", postgres::getUsername);
    registry.add("spring.datasource.password", postgres::getPassword);
  }

  @Autowired private UserRepository userRepository;

  @Test
  void findByEmail_existingUser_returnsUser() {
    userRepository.save(new User("Alice", "alice@example.com"));

    var found = userRepository.findByEmail("alice@example.com");

    assertThat(found).isPresent();
    assertThat(found.get().getName()).isEqualTo("Alice");
  }
}
```

### API 测试 MockMvc

使用完整 Spring context 测试 controller 层:

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

  @Autowired private MockMvc mockMvc;
  @MockBean private UserService userService;

  @Test
  void createUser_validInput_returns201() throws Exception {
    var user = new UserDto(1L, "Alice", "alice@example.com");
    when(userService.create(any())).thenReturn(user);

    mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content("""
                {"name": "Alice", "email": "alice@example.com"}
                """))
        .andExpect(status().isCreated())
        .andExpect(jsonPath("$.name").value("Alice"));
  }

  @Test
  void createUser_invalidEmail_returns400() throws Exception {
    mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content("""
                {"name": "Alice", "email": "not-an-email"}
                """))
        .andExpect(status().isBadRequest());
  }
}
```

## 阶段 4: Security Scan

```bash
# 依赖项 CVE
mvn org.owasp:dependency-check-maven:check
# 或
./gradlew dependencyCheckAnalyze

# 源代码中的秘密
grep -rn "password\s*=\s*\"" src/ --include="*.java" --include="*.yml" --include="*.properties"
grep -rn "sk-\|api_key\|secret" src/ --include="*.java" --include="*.yml"

# 秘密 (git 历史)
git secrets --scan  # 如果已配置
```

### 常见安全问题

```
# System.out.println 检查 (使用 logger 替代)
grep -rn "System\.out\.print" src/main/ --include="*.java"

# 检查响应中的原始异常消息
grep -rn "e\.getMessage()" src/main/ --include="*.java"

# Wildcard CORS 检查
grep -rn "allowedOrigins.*\*" src/main/ --include="*.java"
```

## 阶段 5: Lint/Format (可选检查点)

```bash
mvn spotless:apply   # 如果使用 Spotless 插件
./gradlew spotlessApply
```

## 阶段 6: Diff Review

```bash
git diff --stat
git diff
```

检查清单:
- 没有遗留的调试日志 (`System.out`, 无保护的 `log.debug`)
- 有意义的错误和 HTTP 状态码
- 在需要的地方有 transaction 和 validation
- 配置变更已记录文档

## 输出模板

```
验证报告
===================
Build:     [通过/失败]
Static:    [通过/失败] (spotbugs/pmd/checkstyle)
Tests:     [通过/失败] (X/Y 通过, Z% 覆盖率)
Security:  [通过/失败] (CVE 发现: N)
Diff:      [X 文件变更]

总体:     [准备就绪 / 未准备就绪]

需要修复的问题:
1. ...
2. ...
```

## 连续模式

- 在重大变更期间或长会话中，每 30-60 分钟重新运行各阶段
- 保持短周期: 为快速反馈运行 `mvn -T 4 test` + spotbugs

**记住**: 快速反馈胜过后期意外。严格把关 — 在生产系统中将警告视为缺陷。
