# install_hook_wrapper.ps1 argv-dup 缺陷变通方案 (2026-04-22)

## 摘要

`docs/fixes/install_hook_wrapper.ps1` 是 PowerShell 辅助脚本，负责将
`observe-wrapper.sh` 复制到 `~/.claude/skills/continuous-learning/hooks/` 目录，
并重写 `~/.claude/settings.local.json` 使 observer hook 指向该脚本。

之前的版本生成如下形式的 hook 命令：

```
"C:\Program Files\Git\bin\bash.exe" "C:\Users\...\observe-wrapper.sh"
```

在 Claude Code v2.1.116 下，第一个 argv 标记会被重复。当该标记是带引号的 Windows
可执行文件路径时，`bash.exe` 会以自身作为 `$0` 被重新调用，导致 `cannot execute binary
file` 错误（退出码 126）。PR #1524 记录了根本原因；此脚本作为配套文件，使安装器与
修复后的 `settings.local.json` 结构保持同步。

## 修复内容

- 第一个标记现在是 PATH 解析的 `bash`（无带引号的 `.exe` 路径），因此 argv-dup
  缺陷不会再将二进制文件当作脚本传递。
- wrapper 路径在嵌入 hook 命令前已标准化为正斜杠，避免 MSYS 反斜杠处理异常。
- `PreToolUse` 和 `PostToolUse` 接收带有显式 `pre` / `post` 位置参数的不同命令，
  与 wrapper 预期的形式一致。
- 配置文件使用 LF 行结尾写入，因此下游 JSON 解析器不会看到 `ConvertTo-Json`
  输出的混合 CRLF/LF 格式。

## 最终命令形式

```
bash "C:/Users/<you>/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" pre
bash "C:/Users/<you>/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" post
```

## 使用方法

```powershell
# 将 observe-wrapper.sh 与此脚本放在同一目录，然后执行：
pwsh -File docs/fixes/install_hook_wrapper.ps1
```

脚本在写入前会将 `settings.local.json` 备份为
`settings.local.json.bak-<timestamp>`。

## PowerShell 5.1 兼容性

`ConvertFrom-Json -AsHashtable` 仅支持 PowerShell 7+。脚本会先尝试
`-AsHashtable`，若失败则在 Windows PowerShell 5.1 上回退到手动的 `PSCustomObject` →
`Hashtable` 转换。两个 hook 桶（`PreToolUse`、`PostToolUse`）及其内部的 `hooks` 数组
在序列化前都已具体化为 `System.Collections.ArrayList`，因此 PS 5.1 的
`ConvertTo-Json` 不会将单元素数组折叠为裸对象。已在仅安装 Windows PowerShell 5.1
（无 `pwsh`）的 Windows 11 机器上通过 `powershell -NoProfile -File
docs/fixes/install_hook_wrapper.ps1` 验证。

## 相关

- PR #1524 — settings.local.json 结构修复（相同的 argv-dup 根本原因）
- PR #1511 — 在 observer python 解析中跳过 `AppInstallerPythonRedirector.exe`
- PR #1539 — 与 locale 无关的 `detect-project.sh`
- PR #1542 — `patch_settings_cl_v2_simple.ps1` 配套修复
