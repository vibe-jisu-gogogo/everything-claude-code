---
description: 全面的Kotlin代码审查，涵盖惯用模式、空安全、协程安全和安全性。调用kotlin-reviewer代理。
---

# Kotlin代码审查

该命令调用**kotlin-reviewer**代理进行全面的Kotlin特定代码审查。

## 命令功能

1. **识别Kotlin变更**: 通过`git diff`查找修改过的`.kt`和`.kts`文件
2. **运行构建与静态分析**: 执行`./gradlew build`、`detekt`、`ktlintCheck`
3. **安全扫描**: 检查SQL注入、命令注入、硬编码密钥
4. **空安全审查**: 分析`!!`使用、平台类型处理、不安全类型转换
5. **协程审查**: 检查结构化并发、调度器使用、取消机制
6. **生成报告**: 按严重程度对问题进行分类

## 使用场景

使用`/kotlin-review`的时机：
- 编写或修改Kotlin代码后
- 提交Kotlin变更前
- 审查包含Kotlin代码的拉取请求时
- 适配新的Kotlin代码库时
- 学习Kotlin惯用模式时

## 审查分类

### 严重(必须修复)
- SQL/命令注入漏洞
- 无正当理由使用强制解包`!!`
- 平台类型空安全违规
- GlobalScope使用(违反结构化并发)
- 硬编码凭证
- 不安全反序列化

### 高危(应该修复)
- 可以使用不可变状态的场景下使用可变状态
- 协程上下文内部使用阻塞调用
- 长循环中缺少取消检查
- 密封类型的`when`语句不完整
- 函数过大(>50行)
- 嵌套过深(>4层)

### 中等(建议考虑)
- 非惯用Kotlin写法(Java风格模式)
- 缺少尾随逗号
- 作用域函数误用或嵌套
- 大型集合链式调用缺少序列优化
- 冗余的显式类型声明

## 自动执行的检查

```bash
# 构建检查
./gradlew build

# 静态分析
./gradlew detekt

# 格式检查
./gradlew ktlintCheck

# 测试
./gradlew test
```

## 使用示例

````text
用户: /kotlin-review

代理:
# Kotlin代码审查报告

## 已审查文件
- src/main/kotlin/com/example/service/UserService.kt (已修改)
- src/main/kotlin/com/example/routes/UserRoutes.kt (已修改)

## 静态分析结果
✓ 构建: 成功
✓ detekt: 无问题
警告: ktlint: 2个格式警告

## 发现的问题

[严重] 强制解包空安全问题
文件: src/main/kotlin/com/example/service/UserService.kt:28
问题: 对可空的repository返回结果使用!!
```kotlin
val user = repository.findById(id)!!  // NPE风险
```
修复方案: 使用安全调用配合错误处理
```kotlin
val user = repository.findById(id)
    ?: throw UserNotFoundException("User $id not found")
```

[高危] GlobalScope使用问题
文件: src/main/kotlin/com/example/routes/UserRoutes.kt:45
问题: 使用GlobalScope破坏结构化并发
```kotlin
GlobalScope.launch {
    notificationService.sendWelcome(user)
}
```
修复方案: 使用调用上下文的协程作用域
```kotlin
launch {
    notificationService.sendWelcome(user)
}
```

## 总结
- 严重: 1
- 高危: 1
- 中等: 0

建议: 失败: 修复严重问题前禁止合并
````

## 审批标准

| 状态 | 条件 |
|--------|-----------|
| 通过: 批准 | 无严重或高危问题 |
| 警告: 提醒 | 仅存在中等问题(谨慎合并) |
| 失败: 阻止 | 发现严重或高危问题 |

## 与其他命令的集成

- 首先使用`/kotlin-test`确保测试通过
- 如果发生构建错误使用`/kotlin-build`
- 提交前使用`/kotlin-review`
- 非Kotlin特定的问题使用`/code-review`

## 相关资源

- 代理: `agents/kotlin-reviewer.md`
- 技能: `skills/kotlin-patterns/`、`skills/kotlin-testing/`
