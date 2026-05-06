---
description: 增量修复 Dart analyzer 错误和 Flutter 构建失败。调用 dart-build-resolver 代理进行最小化的精准修复。
---

# Flutter 构建修复

此命令调用 **dart-build-resolver** 代理以最小变更增量修复 Dart/Flutter 构建错误。

## 命令功能

1. **运行诊断**: 执行 `flutter analyze`, `flutter pub get`
2. **解析错误**: 按文件分组并按严重性排序
3. **增量修复**: 一次修复一个错误
4. **验证每个修复**: 每次变更后重新运行分析
5. **报告摘要**: 显示已修复内容和剩余问题

## 何时使用

当出现以下情况时使用 `/flutter-build`:
- `flutter analyze` 报告错误
- 任何平台的 `flutter build` 失败
- `dart pub get` / `flutter pub get` 因版本冲突失败
- `build_runner` 生成代码失败
- 拉取更改后破坏了构建

## 运行的诊断命令

```bash
# 分析
flutter analyze 2>&1

# 依赖
flutter pub get 2>&1

# 代码生成（如果项目使用 build_runner）
dart run build_runner build --delete-conflicting-outputs 2>&1

# 平台构建
flutter build apk 2>&1
flutter build web 2>&1
```

## 示例会话

````text
用户: /flutter-build

代理:
# Flutter 构建问题解决

## 初始诊断

```
$ flutter analyze
Analyzing project...

  error • lib/features/user/data/user_repository_impl.dart:34:16 •
    类型 'String?' 的值不能分配给类型 'String' 的变量。 •
    invalid_assignment

  error • lib/features/cart/presentation/cart_page.dart:58:5 •
    类型 'List<Item>' 未定义 'add' 方法。
    尝试将名称更正为现有方法的名称，或定义名为 'add' 的方法。 •
    undefined_method

发现 2 个错误。
```

## 修复 1：空安全

文件: lib/features/user/data/user_repository_impl.dart:34
错误: 类型 'String?' 的值不能分配给类型 'String' 的变量

变更:
```dart
final id = response.id;
```
改为:
```dart
final id = response.id ?? '';
```

```
$ flutter analyze
发现 1 个错误。
```

## 修复 2：不可变列表

文件: lib/features/cart/presentation/cart_page.dart:58
错误: 类型 'List<Item>' 未定义 'add' 方法
原因: 状态持有不可修改列表；变更需通过 Cubit 处理

变更:
```dart
state.items.add(item);
```
改为:
```dart
context.read<CartCubit>().addItem(item);
// 注意: Cubit 暴露命名方法 (addItem, removeItem)；
// .add(event) 是 BLoC 事件 API — 不要混用。
```

```
$ flutter analyze
未发现问题！
```

## 最终验证

```
$ flutter test
所有测试通过。
```

## 摘要

| 指标 | 数量 |
|--------|-------|
| 已修复分析错误 | 2 |
| 修改文件数 | 2 |
| 剩余问题 | 0 |

构建状态: 已通过 ✓
````

## 常见修复的错误

| 错误 | 典型修复方案 |
|-------|-------------|
| `A value of type 'X?' can't be assigned to 'X'` | 添加 `?? 默认值` 或空检查 |
| `The name 'X' isn't defined` | 添加导入或修正拼写错误 |
| `Non-nullable instance field must be initialized` | 添加初始化器或 `late` 修饰符 |
| `Version solving failed` | 调整 pubspec.yaml 中的版本约束 |
| `Missing concrete implementation of 'X'` | 实现缺失的接口方法 |
| `build_runner: Part of X expected` | 删除陈旧的 `.g.dart` 文件并重建 |

## 修复策略

1. **优先修复分析错误** — 代码必须无错误
2. **其次处理警告** — 修复可能导致运行时 bug 的警告
3. **然后解决 pub 冲突** — 修复依赖解析问题
4. **一次一个修复** — 验证每次变更
5. **最小化变更** — 不要重构，仅修复问题

## 停止条件

出现以下情况时代理将停止并报告：
- 3 次尝试后相同错误仍然存在
- 修复引入更多错误
- 需要架构变更
- 包升级冲突需要用户决策

## 相关命令

- `/flutter-test` — 构建成功后运行测试
- `/flutter-review` — 审查代码质量
- `verification-loop` skill — 完整验证循环

## 相关资源

- 代理: `agents/dart-build-resolver.md`
- 技能: `skills/flutter-dart-code-review/`
