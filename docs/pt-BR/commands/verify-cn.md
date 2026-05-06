# Verification 命令

对当前 codebase 状态运行全面验证。

## 说明

按此确切顺序执行验证：

1. **Build Check**
   - 运行此项目的 build 命令
   - 如果失败，报告错误并停止

2. **Type Check**
   - 运行 TypeScript/type checker
   - 报告所有错误，格式为 file:line

3. **Lint Check**
   - 运行 linter
   - 报告 warnings 和 errors

4. **Test Suite**
   - 运行所有测试
   - 报告 pass/fail 计数
   - 报告覆盖率百分比

5. **Console.log Audit**
   - 在源代码文件中查找 console.log
   - 报告位置

6. **Git Status**
   - 显示未提交的更改
   - 显示自上次提交以来修改的文件

## 输出

生成简洁的验证报告：

```
VERIFICATION: [PASS/FAIL]

Build:    [OK/FAIL]
Types:    [OK/X errors]
Lint:     [OK/X issues]
Tests:    [X/Y passed, Z% coverage]
Secrets:  [OK/X found]
Logs:     [OK/X console.logs]

Ready for PR: [YES/NO]
```

如果有严重问题，列出它们并提供修复建议。

## 参数

$ARGUMENTS 可以是：
- `quick` - 仅 build + types
- `full` - 所有检查（默认）
- `pre-commit` - 与 commit 相关的检查
- `pre-pr` - 完整检查加上 security scan
