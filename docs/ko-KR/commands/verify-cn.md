# 验证命令

对当前代码库状态执行全面验证。

## 指令

严格按照以下顺序执行验证：

1. **Build 检查**
   - 执行本项目的 build 命令
   - 失败时报告错误并停止

2. **类型检查**
   - 运行 TypeScript/类型检查器
   - 以 文件:行号 格式报告所有错误

3. **Lint 检查**
   - 运行 linter
   - 报告警告和错误

4. **测试执行**
   - 运行所有测试
   - 报告通过/失败数量
   - 报告覆盖率百分比

5. **密钥扫描**
   - 在源文件中搜索 API 密钥、token、敏感值模式
   - 报告发现位置

6. **Console.log 审计**
   - 在源文件中搜索 console.log
   - 报告位置

7. **Git 状态**
   - 显示未提交的更改
   - 显示最后一次提交后修改的文件

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

如发现严重问题，列出问题并提供修复建议。

## 参数

$ARGUMENTS:
- `quick` - 仅 build + 类型检查
- `full` - 所有检查（默认值）
- `pre-commit` - 与提交相关的检查
- `pre-pr` - 完整检查 + 安全扫描
