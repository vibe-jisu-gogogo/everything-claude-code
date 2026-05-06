---
description: 运行 Flutter/Dart 测试，报告失败并逐步修复测试问题。涵盖 unit、widget、golden 和 integration 测试。
---

# Flutter 测试

此命令运行 Flutter 测试套件并报告结果。出现失败时，它会逐步诊断并修复问题。

## 命令功能

1. **运行测试**：执行 `flutter test`（或限定为已修改的文件）
2. **解析失败**：按类型和原因识别失败的测试
3. **逐步修复**：尽可能一次修复一个失败
4. **验证**：每次修复后重新运行测试
5. **报告**：显示覆盖率摘要和剩余的失败项

## 何时使用

使用 `/flutter-test` 的场景：
- 实现功能后验证没有破坏现有功能
- 运行 `/flutter-build` 后确保测试通过
- 检查新代码的测试覆盖率
- 当某个特定测试文件失败时
- 提交 PR 之前

## 运行的命令

```bash
# 运行所有测试
flutter test 2>&1

# 带覆盖率运行
flutter test --coverage 2>&1

# 运行特定测试文件
flutter test test/unit/domain/usecases/get_user_test.dart 2>&1

# 运行匹配名称模式的测试
flutter test --name "CartBloc" 2>&1

# 运行 integration 测试（需要设备/模拟器）
flutter test integration_test/ 2>&1

# 当有有意的视觉变更时更新 golden 文件
flutter test --update-goldens 2>&1
```

## 会话示例

````text
用户：/flutter-test

智能体：
# Flutter 测试运行

## 测试执行

```
$ flutter test
00:05 +42 -2: 部分测试失败。

FAILED: test/unit/data/user_repository_test.dart
  UserRepository getById
    用户不存在时返回 null
    Expected: null
    Actual: User(id: 'test-id', name: 'Test User')

FAILED: test/widget/cart_page_test.dart
  CartPage
    购物车为空时显示空状态
    Expected: exactly one widget with text 'Your cart is empty'
    Found: no widget with text 'Your cart is empty'
```

## 修复 1：Repository 测试

文件：test/unit/data/user_repository_test.dart
失败：`getById` 应该为不存在的 ID 返回 null，但返回了一个用户

根本原因：测试设置中添加了 ID 为 'test-id' 的用户，但查询时使用了 'missing-id'。
修复：更新测试以使用 'missing-id' 查询 —— 设置是正确的，查询有误。

```
$ flutter test test/unit/data/user_repository_test.dart
1 test passed.
```

## 修复 2：Widget 测试

文件：test/widget/cart_page_test.dart
失败：未找到空状态文本组件

根本原因：组件中的空状态消息从 'Your cart is empty' 重命名为 'Cart is empty'。
修复：更新测试字符串以匹配当前组件文本。

```
$ flutter test test/widget/cart_page_test.dart
1 test passed.
```

## 最终运行

```
$ flutter test --coverage
所有 44 个测试通过。
覆盖率：84.2%（目标：80%）
```

## 摘要

| 指标 | 值 |
|--------|-------|
| 总测试数 | 44 |
| 通过 | 44 |
| 失败 | 0 |
| 覆盖率 | 84.2% |

测试状态：通过 ✓
````

## 常见测试失败

| 失败 | 典型修复 |
|---------|-------------|
| `Expected: <X> Actual: <Y>` | 更新断言或修复实现 |
| `Widget not found` | 修复查找器选择器，或在组件重命名后更新测试 |
| `Golden file not found` | 运行 `flutter test --update-goldens` 生成文件 |
| `Golden mismatch` | 检查差异；如果变更是有意的，运行 `--update-goldens` |
| `MissingPluginException` | 在测试设置中模拟平台通道 |
| `LateInitializationError` | 在 `setUp()` 中初始化 `late` 字段 |
| `pumpAndSettle timed out` | 替换为显式的 `pump(Duration)` 调用 |

## 相关命令

- `/flutter-build` —— 运行测试前修复构建错误
- `/flutter-review` —— 测试通过后审查代码
- `tdd-workflow` 技能 —— 测试驱动开发工作流

## 相关资源

- 智能体：`agents/flutter-reviewer.md`
- 智能体：`agents/dart-build-resolver.md`
- 技能：`skills/flutter-dart-code-review/`
- 规则：`rules/dart/testing.md`
