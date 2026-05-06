---
name: java-coding-standards
description: Spring Boot 服务的 Java 编码标准：命名、不可变性、Optional 使用、Stream、异常、泛型、项目布局。
---

# Java 编码标准

Spring Boot 服务中易读、可维护的 Java(17+) 代码标准。

## 核心原则

- 优先选择清晰性而非巧妙
- 默认不可变；最小化共享可变状态
- 使用有意义的异常实现快速失败
- 一致的命名和包结构

## 命名

```java
// PASS: 类/记录: PascalCase
public class MarketService {}
public record Money(BigDecimal amount, Currency currency) {}

// PASS: 方法/字段: camelCase
private final MarketRepository marketRepository;
public Market findBySlug(String slug) {}

// PASS: 常量: UPPER_SNAKE_CASE
private static final int MAX_PAGE_SIZE = 100;
```

## 不可变性

```java
// PASS: 优先使用 record 和 final 字段
public record MarketDto(Long id, String name, MarketStatus status) {}

public class Market {
  private final Long id;
  private final String name;
  // 只有 getter，没有 setter
}
```

## Optional 的使用

```java
// PASS: 从 find* 方法返回 Optional
Optional<Market> market = marketRepository.findBySlug(slug);

// PASS: 使用 map/flatMap 代替 get()
return market
    .map(MarketResponse::from)
    .orElseThrow(() -> new EntityNotFoundException("Market not found"));
```

## Stream 最佳实践

```java
// PASS: 使用 Stream 进行转换，保持管道简短
List<String> names = markets.stream()
    .map(Market::name)
    .filter(Objects::nonNull)
    .toList();

// FAIL: 避免复杂的嵌套 Stream；为清晰起见优先使用循环
```

## 异常

- 对领域错误使用非受检异常；技术异常需附带上下文重新包装
- 创建领域特定的异常（例如：`MarketNotFoundException`）
- 避免宽泛的 `catch (Exception ex)`（除非在中心位置重新抛出/记录日志）

```java
throw new MarketNotFoundException(slug);
```

## 泛型与类型安全

- 避免原始类型；声明泛型参数
- 对可复用工具类优先使用有界泛型

```java
public <T extends Identifiable> Map<Long, T> indexById(Collection<T> items) { ... }
```

## 项目结构 (Maven/Gradle)

```
src/main/java/com/example/app/
  config/
  controller/
  service/
  repository/
  domain/
  dto/
  util/
src/main/resources/
  application.yml
src/test/java/... (镜像 main)
```

## 格式与风格

- 一致使用 2 或 4 空格（项目标准）
- 每个文件一个公共顶级类型
- 保持方法简短专注；提取辅助方法
- 成员顺序：常量、字段、构造函数、公共方法、受保护方法、私有方法

## 需要避免的代码坏味道

- 长参数列表 → 使用 DTO/Builder
- 深层嵌套 → 提前返回
- 魔法数字 → 命名常量
- 静态可变状态 → 优先依赖注入
- 静默 catch 块 → 记录日志并采取行动，或重新抛出

## 日志记录

```java
private static final Logger log = LoggerFactory.getLogger(MarketService.class);
log.info("fetch_market slug={}", slug);
log.error("failed_fetch_market slug={}", slug, ex);
```

## Null 处理

- 仅在不得已时接受 `@Nullable`；其他情况使用 `@NonNull`
- 对输入使用 Bean Validation（`@NotNull`、`@NotBlank`）

## 测试期望

- JUnit 5 + AssertJ 实现流畅断言
- Mockito 用于模拟；尽可能避免部分模拟
- 优先确定性测试；无隐藏的 sleep

**记住**：保持代码有意、有类型、可观察。除非证明有必要，否则优先优化可维护性而非微观优化。
