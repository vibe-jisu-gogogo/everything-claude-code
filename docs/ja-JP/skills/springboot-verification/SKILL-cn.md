---
name: springboot-verification
description: Verification loop for Spring Boot projects: build, static analysis, tests with coverage, security scans, and diff review before release or PR.
---

# Spring Boot 验证循环

在 PR 之前、重大变更之后、部署之前执行。

## 阶段 1: 构建

```bash
mvn -T 4 clean verify -DskipTests
# 或者
./gradlew clean assemble -x test
```

如果构建失败，停止并修复。

## 阶段 2: 静态分析

Maven（通用插件）:
```bash
mvn -T 4 spotbugs:check pmd:check checkstyle:check
```

Gradle（如果已配置）:
```bash
./gradlew checkstyleMain pmdMain spotbugsMain
```

## 阶段 3: 测试 + 覆盖率

```bash
mvn -T 4 test
mvn jacoco:report   # 确认 80% 以上的覆盖率
# 或者
./gradlew test jacocoTestReport
```

报告:
- 总测试数，通过/失败
- 覆盖率%（行/分支）

## 阶段 4: 安全扫描

```bash
# 依赖项 CVE
mvn org.owasp:dependency-check-maven:check
# 或者
./gradlew dependencyCheckAnalyze

# 密钥（git）
git secrets --scan  # 如果已配置
```

## 阶段 5: Lint/格式化（可选关卡）

```bash
mvn spotless:apply   # 如果使用 Spotless 插件
./gradlew spotlessApply
```

## 阶段 6: 差异审查

```bash
git diff --stat
git diff
```

检查清单:
- 没有残留调试日志（`System.out`、无保护的 `log.debug`）
- 有意义的错误和 HTTP 状态
- 在需要的地方有事务和验证
- 配置变更已文档化

## 输出模板

```
验证报告
===================
构建:     [通过/失败]
静态分析:   [通过/失败] (spotbugs/pmd/checkstyle)
测试:     [通过/失败] (X/Y 通过, Z% 覆盖率)
安全: [通过/失败] (发现 CVE: N)
差异:       [X 文件变更]

整体:       [准备就绪 / 未完成]

需要修复的问题:
1. ...
2. ...
```

## 持续模式

- 当有重大变更时，或在长会话中每 30-60 分钟重新执行阶段
- 保持短循环: `mvn -T 4 test` + spotbugs 以获得快速反馈

**注意**: 快速反馈胜过延迟的意外。保持关卡严格，在生产系统中将警告视为缺陷。
