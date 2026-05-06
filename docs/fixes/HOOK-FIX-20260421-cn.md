# ECC Hook 修复 — 2026-04-21

## 摘要

Windows 上的 Claude Code CLI v2.1.116 所有 Bash tool hook 调用都失败了，错误信息如下：

```
PreToolUse:Bash hook error
Failed with non-blocking status code:
C:\Program Files\Git\bin\bash.exe: C:\Program Files\Git\bin\bash.exe:
cannot execute binary file

PostToolUse:Bash hook error  (同上)
```

结果：`observations.jsonl` 在 `2026-04-20T23:03:38Z` 后停止更新
（最后一条记录是来自之前 BOM-on-stdin 问题的 `parse_error`）。

## 根本原因

`C:\Users\sugig\.claude\settings.local.json` 存在两个缺陷：

### 缺陷 1 — UTF-8 BOM + CRLF 换行符

文件以 `EF BB BF`（UTF-8 BOM）开头，并使用 `CRLF` 行终止符。
这是 PowerShell `ConvertTo-Json | Out-File` 的默认行为，也是
`patch_settings_cl_v2_simple.ps1` 重写文件后留下的结果。

```
00000000: efbb bf7b 0d0a 2020 2020 2268 6f6f 6b73  ...{..    "hooks
```

### 缺陷 2 — 双重包装的 bash.exe 调用

命令字符串显式地重新调用 bash.exe：

```json
"command": "\"C:\\Program Files\\Git\\bin\\bash.exe\" \"C:\\Users\\sugig\\.claude\\skills\\continuous-learning\\hooks\\observe-wrapper.sh\""
```

当 Claude Code 在 Windows 上生成此命令时，参数拆分无法正确
保留带引号的 `"C:\Program Files\..."` 标记。`Program Files` 中嵌入的
空格会拆分 `argv[0]`，导致 `bash.exe` 自身作为脚本文件被传递，
产生：

```
bash.exe: bash.exe: cannot execute binary file
```

### 之前可用的格式（供参考）

在 `patch_settings_cl_v2_simple.ps1` 运行之前，命令只是：

```json
"command": "C:\\Users\\sugig\\.claude\\skills\\continuous-learning\\hooks\\observe.sh"
```

Windows 上的 Claude Code 会检测 `.sh` 并通过 Git Bash 调用它——不需要
手动的 `bash.exe` 包装。

## 修复

`C:\Users\sugig\.claude\settings.local.json` 已重写为 UTF-8（无 BOM）、LF
换行符，命令直接指向包装器 `.sh` 并将 hook phase 作为普通参数传递：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh pre"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh post"
          }
        ]
      }
    ]
  }
}
```

附带好处：`pre` / `post` 参数现在被路由到 `observe.sh` 的
`HOOK_PHASE` 变量，因此事件被正确记录为 `tool_start` 和
`tool_complete`（之前所有内容都被记录为 `tool_complete`）。

## 验证

直接调用新的命令格式，模拟两个 hook 阶段：

```bash
# PostToolUse 路径
echo '{"tool_name":"Bash","tool_input":{"command":"pwd"},"session_id":"post-fix-verify-001","cwd":"...","hook_event_name":"PostToolUse"}' \
  | "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" post
# exit=0

# PreToolUse 路径
echo '{"tool_name":"Bash","tool_input":{"command":"ls"},"session_id":"post-fix-verify-pre-001","cwd":"...","hook_event_name":"PreToolUse"}' \
  | "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" pre
# exit=0
```

`observations.jsonl` 新增了：

```
{"timestamp":"2026-04-21T05:57:54Z","event":"tool_complete","tool":"Bash","session":"post-fix-verify-001",...}
{"timestamp":"2026-04-21T05:57:55Z","event":"tool_start","tool":"Bash","session":"post-fix-verify-pre-001","input":"{\"command\":\"ls\"}",...}
```

两个阶段现在都能产生正确类型的事件。

**关于实时 CLI 验证的说明：** 设置更改在下次
`claude` CLI 会话启动时生效。重启 CLI 并运行 Bash 工具调用，
以确认实际 CLI 会话的新行出现在 `observations.jsonl` 中。

## 涉及的文件

- `C:\Users\sugig\.claude\settings.local.json` — 已重写
- `C:\Users\sugig\.claude\settings.local.json.bak-hookfix-20260421-145718` — 修复前备份

## 已知的上游 Bug（此处未修复）

- `install_hook_wrapper.ps1` — 在步骤 [3/4] 停止，永远无法到达 [4/4]。
- `patch_settings_cl_v2_simple.ps1` — 用 UTF-8-BOM + CRLF 覆盖 `settings.local.json`，
  并重新引入双重包装的 `bash.exe` 命令。
  应替换为输出 UTF-8（无 BOM）、LF 和
  直接 `.sh` 路径的补丁程序。

## 分支

`claude/hook-fix-20260421`
