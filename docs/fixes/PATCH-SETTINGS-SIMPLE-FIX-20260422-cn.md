# patch_settings_cl_v2_simple.ps1 argv-dup 漏洞临时解决方案 (2026-04-22)

## 摘要

`docs/fixes/patch_settings_cl_v2_simple.ps1` 是一个最小化的 PowerShell
辅助脚本，用于修补 `~/.claude/settings.local.json`，使 observer hook
指向 `observe-wrapper.sh`。它是 `docs/fixes/install_hook_wrapper.ps1`
(PR #1540) 的"简单"版本：它从不复制 wrapper 脚本，仅重写 settings 文件。

此辅助脚本的先前版本将原始 `observe.sh` 路径注册为 hook 命令，
在 `PreToolUse` 和 `PostToolUse` 之间共享单个命令字符串，
并依赖可能产生 CRLF 行结尾的 `ConvertTo-Json` 默认值。
在 Claude Code v2.1.116 下，第一个 argv 标记会被复制，
因此需要以特定形式调用 wrapper，并且两个 hook 阶段需要不同的条目。

## 修复内容

- 第一个标记是 PATH 解析的 `bash`（不带引号的 `.exe` 路径），
  因此 argv-dup 漏洞不再将二进制文件作为脚本传递。与 PR #1524 和
  PR #1540 保持一致。
- wrapper 路径在嵌入 hook 命令之前被标准化为正斜杠，
  避免 MSYS 反斜杠处理带来的意外问题。
- `PreToolUse` 和 `PostToolUse` 接收带有明确 `pre` / `post`
  位置参数的不同命令。
- settings 文件以 UTF-8（无 BOM）写入，并将 CRLF 标准化为 LF，
  确保下游 JSON 解析器不会遇到混合行结尾。
- 保留现有 hooks（包括传统的 `observe.sh` 条目和无关的第三方 hooks）
  —— 脚本仅在新的 wrapper 条目尚未注册时追加它们。
- 重复运行时具备幂等性：第二次调用会识别标准命令字符串并记录
  `[SKIP]`，而不是复制条目。

## 最终的命令形式

```
bash "C:/Users/<you>/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" pre
bash "C:/Users/<you>/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" post
```

## 用法

```powershell
pwsh -File docs/fixes/patch_settings_cl_v2_simple.ps1
# 同时支持 Windows PowerShell 5.1：
powershell -NoProfile -ExecutionPolicy Bypass -File docs/fixes/patch_settings_cl_v2_simple.ps1
```

脚本在写入前会将现有的 settings 文件备份到
`settings.local.json.bak-<timestamp>`。

## PowerShell 5.1 兼容性

`ConvertFrom-Json -AsHashtable` 仅在 PowerShell 7+ 中可用。
脚本首先尝试 `-AsHashtable`，在 Windows PowerShell 5.1 上
回退到手动的 `PSCustomObject` → `Hashtable` 转换。
两个 hook 桶（`PreToolUse`、`PostToolUse`）及其内部的 `hooks` 数组
在序列化前都被具体化为 `System.Collections.ArrayList`，
因此 PS 5.1 的 `ConvertTo-Json` 不会将单元素数组折叠为裸对象。

## 已验证的情况（dry-run）

1. 全新安装 — 没有现有 settings → 创建标准文件。
2. 幂等重复运行 — 现有标准文件 → 两个阶段都 `[SKIP]`，
   除了预写入备份外，文件内容不变。
3. 存在传统 `observe.sh` → 保留传统条目，并在其旁边追加
   新的 `observe-wrapper.sh` 条目。

所有三种情况都产生仅 LF 的输出，并且与 PR #1524 对
`settings.local.json` 的手动修复所注册的形式一致。

## 相关

- PR #1524 — settings.local.json 形式修复（相同的 argv-dup 根本原因）
- PR #1539 — 独立于语言环境的 `detect-project.sh`
- PR #1540 — `install_hook_wrapper.ps1` argv-dup 修复（配套脚本）
