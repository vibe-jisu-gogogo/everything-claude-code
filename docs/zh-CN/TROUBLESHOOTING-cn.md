# 故障排除指南

Everything Claude Code (ECC) 插件的常见问题与解决方案。

## 目录

* [内存与上下文问题](#内存与上下文问题)
* [代理工具故障](#代理工具故障)
* [钩子与工作流错误](#钩子与工作流错误)
* [安装与设置](#安装与设置)
* [性能问题](#性能问题)
* [常见错误信息](#常见错误信息)
* [获取帮助](#获取帮助)

***

## 内存与上下文问题

### 上下文窗口溢出

**症状：** 出现"上下文过长"错误或响应不完整

**原因：**

* 上传的大文件超出令牌限制
* 累积的对话历史记录
* 单次会话中包含多个大型工具输出

**解决方案：**

```bash
# 1. 清除对话历史并重新开始
# 使用 Claude Code: "New Chat" 或 Cmd/Ctrl+Shift+N

# 2. 分析前减小文件大小
head -n 100 large-file.log > sample.log

# 3. 对大型输出使用流式处理
head -n 50 large-file.txt

# 4. 将任务拆分为更小的块
# 不要: "分析所有 50 个文件"
# 使用: "分析 src/components/ 目录中的文件"
```

### 内存持久化失败

**症状：** 代理不记得先前的上下文或观察结果

**原因：**

* 连续学习钩子被禁用
* 观察文件损坏
* 项目检测失败

**解决方案：**

```bash
# 检查是否正在记录观察
ls ~/.claude/homunculus/projects/*/observations.jsonl

# 查找当前项目的哈希 ID
python3 - <<'PY'
import json, os
registry_path = os.path.expanduser("~/.claude/homunculus/projects.json")
with open(registry_path) as f:
    registry = json.load(f)
for project_id, meta in registry.items():
    if meta.get("root") == os.getcwd():
        print(project_id)
        break
else:
    raise SystemExit("Project hash not found in ~/.claude/homunculus/projects.json")
PY

# 查看该项目的最近观察
tail -20 ~/.claude/homunculus/projects/<project-hash>/observations.jsonl

# 在重新创建之前备份损坏的观察文件
mv ~/.claude/homunculus/projects/<project-hash>/observations.jsonl \
  ~/.claude/homunculus/projects/<project-hash>/observations.jsonl.bak.$(date +%Y%m%d-%H%M%S)

# 验证钩子已启用
grep -r "observe" ~/.claude/settings.json
```

***

## 代理工具故障

### 未找到代理

**症状：** 出现"代理未加载"或"未知代理"错误

**原因：**

* 插件未正确安装
* 代理路径配置错误
* 市场安装与手动安装不匹配

**解决方案：**

```bash
# 检查插件安装
ls ~/.claude/plugins/cache/

# 验证代理存在（市场安装）
ls ~/.claude/plugins/cache/*/agents/

# 对于手动安装，代理应在：
ls ~/.claude/agents/  # 仅自定义代理

# 重新加载插件
# Claude Code → 设置 → 扩展 → 重新加载
```

### 工作流执行挂起

**症状：** 代理启动但从未完成

**原因：**

* 代理逻辑中存在无限循环
* 等待用户输入时被阻塞
* 等待 API 响应时网络超时

**解决方案：**

```bash
# 1. 检查卡住的进程
ps aux | grep claude

# 2. 启用调试模式
export CLAUDE_DEBUG=1

# 3. 设置更短的超时
export CLAUDE_TIMEOUT=30

# 4. 检查网络连接
curl -I https://api.anthropic.com
```

### 工具使用错误

**症状：** 出现"工具执行失败"或权限被拒绝

**原因：**

* 缺少依赖项（npm、python 等）
* 文件权限不足
* 路径未找到

**解决方案：**

```bash
# 验证所需工具已安装
which node python3 npm git

# 修复钩子脚本的权限
chmod +x ~/.claude/plugins/cache/*/hooks/*.sh
chmod +x ~/.claude/plugins/cache/*/skills/*/hooks/*.sh

# 检查 PATH 包含必要的二进制文件
echo $PATH
```

***

## 钩子与工作流错误

### 钩子未触发

**症状：** 前置/后置钩子未执行

**原因：**

* 钩子未在 settings.json 中注册
* 钩子语法无效
* 钩子脚本不可执行

**解决方案：**

```bash
# 检查钩子已注册
grep -A 10 '"hooks"' ~/.claude/settings.json

# 验证钩子文件存在且可执行
ls -la ~/.claude/plugins/cache/*/hooks/

# 手动测试钩子
bash ~/.claude/plugins/cache/*/hooks/pre-bash.sh <<< '{"command":"echo test"}'

# 重新注册钩子（如果使用插件）
# 在 Claude Code 设置中禁用并重新启用插件
```

### Python/Node 版本不匹配

**症状：** 出现"未找到 python3"或"node: 命令未找到"

**原因：**

* 缺少 Python/Node 安装
* PATH 未配置
* Python 版本错误（Windows）

**解决方案：**

```bash
# 安装 Python 3（如果缺失）
# macOS: brew install python3
# Ubuntu: sudo apt install python3
# Windows: 从 python.org 下载

# 安装 Node.js（如果缺失）
# macOS: brew install node
# Ubuntu: sudo apt install nodejs npm
# Windows: 从 nodejs.org 下载

# 验证安装
python3 --version
node --version
npm --version

# Windows: 确保 python（而非 python3）可用
python --version
```

### 开发服务器拦截器误报

**症状：** 钩子拦截了提及"dev"的合法命令

**原因：**

* Heredoc 内容触发模式匹配
* 参数中包含"dev"的非开发命令

**解决方案：**

```bash
# 此问题已在 v1.8.0+ 中修复（PR #371）
# 将插件升级到最新版本

# 解决方法：在 tmux 中运行开发服务器
tmux new-session -d -s dev "npm run dev"
tmux attach -t dev

# 如需要，临时禁用钩子
# 编辑 ~/.claude/settings.json 并移除 pre-bash 钩子
```

***

## 安装与设置

### 插件未加载

**症状：** 安装后插件功能不可用

**原因：**

* 市场缓存未更新
* Claude Code 版本不兼容
* 插件文件损坏

**解决方案：**

```bash
# 在更改前检查插件缓存
ls -la ~/.claude/plugins/cache/

# 备份插件缓存而不是原地删除
mv ~/.claude/plugins/cache ~/.claude/plugins/cache.backup.$(date +%Y%m%d-%H%M%S)
mkdir -p ~/.claude/plugins/cache

# 从市场重新安装
# Claude Code → 扩展 → Everything Claude Code → 卸载
# 然后从市场重新安装

# 检查 Claude Code 版本
claude --version
# 需要 Claude Code 2.0+

# 手动安装（如果市场安装失败）
git clone https://github.com/affaan-m/everything-claude-code.git
cp -r everything-claude-code ~/.claude/plugins/ecc
```

### 包管理器检测失败

**症状：** 使用了错误的包管理器（用 npm 而不是 pnpm）

**原因：**

* 没有 lock 文件
* 未设置 CLAUDE_PACKAGE_MANAGER
* 多个 lock 文件导致检测混乱

**解决方案：**

```bash
# 全局设置首选包管理器
export CLAUDE_PACKAGE_MANAGER=pnpm
# 添加到 ~/.bashrc 或 ~/.zshrc

# 或按项目设置
echo '{"packageManager": "pnpm"}' > .claude/package-manager.json

# 或使用 package.json 字段
npm pkg set packageManager="pnpm@8.15.0"

# 警告：删除 lock 文件可能会更改已安装的依赖版本。
# 首先提交或备份 lock 文件，然后运行全新安装并重新运行 CI。
# 仅在有意切换包管理器时执行此操作。
rm package-lock.json  # 如果使用 pnpm/yarn/bun
```

***

## 性能问题

### 响应时间缓慢

**症状：** 代理需要 30 秒以上才能响应

**原因：**

* 大型观察文件
* 活动钩子过多
* 到 API 的网络延迟

**解决方案：**

```bash
# 归档大型观察文件而不是删除
archive_dir="$HOME/.claude/homunculus/archive/$(date +%Y%m%d)"
mkdir -p "$archive_dir"
find ~/.claude/homunculus/projects -name "observations.jsonl" -size +10M -exec sh -c '
  for file do
    base=$(basename "$(dirname "$file")")
    gzip -c "$file" > "'"$archive_dir"'/${base}-observations.jsonl.gz"
    : > "$file"
  done
' sh {} +

# 临时禁用未使用的钩子
# 编辑 ~/.claude/settings.json

# 保持活动观察文件较小
# 大型归档文件应存放在 ~/.claude/homunculus/archive/ 下
```

### CPU 使用率高

**症状：** Claude Code 占用 100% CPU

**原因：**

* 无限观察循环
* 对大型目录的文件监视
* 钩子中的内存泄漏

**解决方案：**

```bash
# 检查失控进程
top -o cpu | grep claude

# 临时禁用连续学习
touch ~/.claude/homunculus/disabled

# 重启 Claude Code
# Cmd/Ctrl+Q 然后重新打开

# 检查观察文件大小
du -sh ~/.claude/homunculus/*/
```

***

## 常见错误信息

### "EACCES: permission denied"

```bash
# 修复钩子权限
find ~/.claude/plugins -name "*.sh" -exec chmod +x {} \;

# 修复观察目录权限
chmod -R u+rwX,go+rX ~/.claude/homunculus
```

### "MODULE_NOT_FOUND"

```bash
# 安装插件依赖
cd ~/.claude/plugins/cache/ecc
npm install

# 或对于手动安装
cd ~/.claude/plugins/ecc
npm install
```

### "spawn UNKNOWN"

```bash
# Windows 特定：确保脚本使用正确的行结尾
# 将 CRLF 转换为 LF
find ~/.claude/plugins -name "*.sh" -exec dos2unix {} \;

# 或安装 dos2unix
# macOS: brew install dos2unix
# Ubuntu: sudo apt install dos2unix
```

***

## 获取帮助

如果您仍然遇到问题：

1. **检查 GitHub Issues**：[github.com/affaan-m/everything-claude-code/issues](https://github.com/affaan-m/everything-claude-code/issues)
2. **启用调试日志记录**：
   ```bash
   export CLAUDE_DEBUG=1
   export CLAUDE_LOG_LEVEL=debug
   ```
3. **收集诊断信息**：
   ```bash
   claude --version
   node --version
   python3 --version
   echo $CLAUDE_PACKAGE_MANAGER
   ls -la ~/.claude/plugins/cache/
   ```
4. **提交 Issue**：包括调试日志、错误信息和诊断信息

***

## 相关文档

* [README.md](README.md) - 安装与功能
* [CONTRIBUTING.md](CONTRIBUTING.md) - 开发指南
* [docs/](..) - 详细文档
* [examples/](../../examples) - 使用示例
