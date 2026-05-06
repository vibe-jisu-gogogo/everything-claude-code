# 验证命令

对当前代码库状态执行全面验证。

## 步骤

请按此准确顺序执行验证：

1. **Build检查**
   - 执行此项目的build命令
   - 如果失败，报告错误并**停止**

2. **Type检查**
   - 运行TypeScript/type checker
   - 报告所有错误及文件:行号

3. **Lint检查**
   - 运行linter
   - 报告警告和错误

4. **Test suite**
   - 运行所有测试
   - 报告通过/失败数量
   - 报告覆盖率百分比

5. **Console.log审计**
   - 在源文件中搜索console.log
   - 报告位置

6. **Git状态**
   - 显示未提交的更改
   - 显示自上次提交以来更改的文件

## 输出

生成简洁的验证报告：

```
验证结果: [PASS/FAIL]

Build:        [OK/FAIL]
Type:         [OK/X错误]
Lint:         [OK/X问题]
Test:         [X/Y通过, Z%覆盖率]
Secret:       [OK/X发现]
Log:          [OK/X个console.log]

PR准备就绪: [YES/NO]
```

如果有严重问题，列出并提供修复建议。

## 参数

$ARGUMENTS 可以是以下任意一项：
- `quick` - 仅build + type检查
- `full` - 所有检查（默认）
- `pre-commit` - 与提交相关的检查
- `pre-pr` - 完整检查 + security scan
