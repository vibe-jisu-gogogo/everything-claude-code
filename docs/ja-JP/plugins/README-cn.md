# 插件与市场

插件通过新工具和功能扩展Claude Code。本指南仅涵盖安装 - 关于何时以及为何使用，请参阅[完整文章](https://x.com/affaanmustafa/status/2012378465664745795)。

---

## 市场

市场是可安装插件的仓库。

### 添加市场

```bash
# 添加官方Anthropic市场
claude plugin marketplace add https://github.com/anthropics/claude-plugins-official

# 添加社区市场
# mgrep plugin by @mixedbread-ai
claud plugin marketplace add https://github.com/mixedbread-ai/mgrep
```

### 推荐市场

| 市场 | 来源 |
|-------------|--------|
| claude-plugins-official | `anthropics/claude-plugins-official` |
| claude-code-plugins | `anthropics/claude-code` |
| Mixedbread-Grep | `mixedbread-ai/mgrep` |

---

## 安装插件

```bash
# 打开插件浏览器
/plugins

# 或者直接安装
claude plugin install typescript-lsp@claude-plugins-official
```

### 推荐插件

**开发:**
- `typescript-lsp` - TypeScript智能
- `pyright-lsp` - Python类型检查
- `hookify` - 通过对话创建钩子
- `code-simplifier` - 代码重构

**代码质量:**
- `code-review` - 代码审查
- `pr-review-toolkit` - PR自动化
- `security-guidance` - 安全检查

**搜索:**
- `mgrep` - 扩展搜索（优于ripgrep）
- `context7` - 实时文档搜索

**工作流:**
- `commit-commands` - Git工作流
- `frontend-patterns` - UI模式
- `feature-dev` - 功能开发

---

## 快速设置

```bash
# 添加市场
claude plugin marketplace add https://github.com/anthropics/claude-plugins-official
# mgrep plugin by @mixedbread-ai
claud plugin marketplace add https://github.com/mixedbread-ai/mgrep

# 打开/plugins并安装所需内容
```

---

## 插件文件位置

```
~/.claude/plugins/
|-- cache/                    # 已下载插件
|-- installed_plugins.json    # 已安装列表
|-- known_marketplaces.json   # 已添加市场
|-- marketplaces/             # 市场数据
```
