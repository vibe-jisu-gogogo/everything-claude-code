# HOOK-FIX-20260421 Addendum — v2.1.116 argv 重复 bug

早会 session 中已作为 commit 527c18b 修复。晚会 session 中进行了额外验证，并识别了早间 fix 未能完全覆盖的 Claude Code 特有 bug，因此记录此补充说明。

## 早间 fix 的形式

```json
"command": "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh pre"
```

将 `.sh` 文件直接作为 command 的形式。前提是 Git Bash 通过 shebang 执行。

## 晚间 额外验证中发现的问题

在 Node.js 的 `child_process.spawn` 中直接执行 `.sh` 文件时，Windows 环境下会因 **EFTYPE** 失败：

```js
spawn('C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh', 
      ['post'], {stdio:['pipe','pipe','pipe']});
// → Error: spawn EFTYPE (errno -4028)
```

如果添加 `shell:true` 可以通过 cmd.exe 执行，但仍存在依赖 Claude Code 实现的风险。

## 晚间 应用的追加 fix

将第一 token 改为 `bash`（PATH 解析）的显式调用形式更新：

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "bash \"C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh\" pre"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "bash \"C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh\" post"
      }]
    }]
  }
}
```

此形式与 `~/.claude/hooks/hooks.json` 中的 ECC 标准 observer 注册使用相同模式，已有实际无错误运行的实绩。

### Node spawn 验证

```js
spawn('bash "C:/Users/sugig/.claude/skills/continuous-learning/hooks/observe-wrapper.sh" post',
      [], {shell:true});
// exit=0 → observations.jsonl 正常追加
```

## Claude Code v2.1.116 的 argv 重复 bug（详细）

早间 fix doc 中作为「Defect 2」记录了 `bash.exe: bash.exe: cannot execute binary file`，现已确定其根本机制，特此记录。

### 重现

```bash
"C:\Program Files\Git\bin\bash.exe" "C:\Program Files\Git\bin\bash.exe"
# stderr: "C:\Program Files\Git\bin\bash.exe: C:\Program Files\Git\bin\bash.exe: cannot execute binary file"
# exit: 126
```

bash 将 argv[1] 视为 script 并尝试读取。如果 argv[1] 是 bash.exe 自身，则会因 ELF/PE 二进制检测而失败 → exit 126。错误信息完全一致。

### Claude Code 侧的行为

当 hook command 为 `"C:\Program Files\Git\bin\bash.exe" "C:\Users\...\wrapper.sh"` 时，推测 v2.1.116 会**将第一 token（= bash.exe 完整路径）同时传递给 argv[0] 和 argv[1]**。结果 bash 尝试将 argv[1] = bash.exe 作为 script 读取，导致以 126 退出。

### 回避策

第一 token 不要使用 bash.exe 的完整路径＋带空格的路径：
1. `OK:` `bash` （PATH 解析的单一 token）— 晚间 fix / hooks.json 模式
2. `OK:` `.sh` 直接路径（依赖 Claude Code 的 .sh 处理）— 早间 fix
3. `BAD:` `"C:\Program Files\Git\bin\bash.exe" "<path>"` — 第一 token 被引号包裹且含空格

## 结论

早间 fix（直接指定 .sh）和晚间 fix（显式 bash prefix）都不会触发 argv 重复 bug，但**晚间 fix 对 Claude Code 实现的依赖更少**，因此推荐。

不过由于早间 fix commit 527c18b 已放入 docs/fixes/，通过追加此 Addendum 实现两种方案并存记录。下次 CLI 重启时晚间 fix 将在实际运行中保留。

## 关联

- 早间 fix commit: 527c18b
- 早间 fix doc: docs/fixes/HOOK-FIX-20260421.md
- 早间 apply script: docs/fixes/apply-hook-fix.sh
- 晚间 fix 记录（本地）: C:\Users\sugig\Documents\Claude\Projects\ECC作成\hook-fix-report-20260421.md
- 晚间 fix 应用文件: C:\Users\sugig\.claude\settings.local.json
- 晚间 backup: C:\Users\sugig\.claude\settings.local.json.bak-hook-fix-20260421
