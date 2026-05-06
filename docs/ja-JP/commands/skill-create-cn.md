---
name: skill-create
description: 分析本地git历史提取编码模式，生成SKILL.md文件。这是Skill Creator GitHub App的本地版本。
allowed_tools: ["Bash", "Read", "Write", "Grep", "Glob"]
---

# /skill-create - 本地技能生成

分析仓库的git历史提取编码模式，生成SKILL.md文件来教Claude团队的最佳实践。

## 使用方法

```bash
/skill-create                    # 分析当前仓库
/skill-create --commits 100      # 分析最后100个提交
/skill-create --output ./skills  # 自定义输出目录
/skill-create --instincts        # 同时生成continuous-learning-v2用的instincts
```

## 执行内容

1. **Git历史解析** - 分析提交、文件变更、模式
2. **模式检测** - 识别重复的工作流和惯例
3. **SKILL.md生成** - 创建有效的Claude Code技能文件
4. **可选创建Instincts** - 用于continuous-learning-v2系统

## 分析步骤

### 步骤1: Git数据收集

```bash
# 获取包含文件变更的最近提交
git log --oneline -n ${COMMITS:-200} --name-only --pretty=format:"%H|%s|%ad" --date=short

# 获取按文件的提交频率
git log --oneline -n 200 --name-only | grep -v "^$" | grep -v "^[a-f0-9]" | sort | uniq -c | sort -rn | head -20

# 获取提交消息模式
git log --oneline -n 200 | cut -d' ' -f2- | head -50
```

### 步骤2: 模式检测

寻找以下模式类型:

| 模式 | 检测方法 |
|---------|-----------------|
| **提交规约** | 提交消息的正则表达式(feat:, fix:, chore:) |
| **文件共变更** | 总是一起变更的文件 |
| **工作流序列** | 重复的文件变更模式 |
| **架构** | 文件夹结构和命名规约 |
| **测试模式** | 测试文件的位置、命名、覆盖率 |

### 步骤3: SKILL.md生成

输出格式:

```markdown
---
name: {repo-name}-patterns
description: 从{repo-name}提取的编码模式
version: 1.0.0
source: local-git-analysis
analyzed_commits: {count}
---

# {Repo Name} Patterns

## 提交规约
{检测到的提交消息模式}

## 代码架构
{检测到的文件夹结构和配置}

## 工作流
{检测到的重复文件变更模式}

## 测试模式
{检测到的测试规约}
```

### 步骤4: Instincts生成(--instincts选项)

continuous-learning-v2集成用:

```yaml
---
id: {repo}-commit-convention
trigger: "when writing a commit message"
confidence: 0.8
domain: git
source: local-repo-analysis
---

# 使用Conventional Commits

## Action
提交使用前缀: feat:, fix:, chore:, docs:, test:, refactor:

## Evidence
- 分析{n}个提交
- {percentage}%遵循conventional commit格式
```

## 输出示例

在TypeScript项目中执行`/skill-create`，可能生成如下输出:

```markdown
---
name: my-app-patterns
description: 从my-app仓库提取的编码模式
version: 1.0.0
source: local-git-analysis
analyzed_commits: 150
---

# My App Patterns

## 提交规约

本项目使用**conventional commits**:
- `feat:` - 新功能
- `fix:` - 修复bug
- `chore:` - 维护任务
- `docs:` - 文档更新

## 代码架构

```
src/
├── components/     # React组件(PascalCase.tsx)
├── hooks/          # 自定义hook(use*.ts)
├── utils/          # 工具函数
├── types/          # TypeScript类型定义
└── services/       # API和外部服务
```

## 工作流

### 添加新组件
1. 创建`src/components/ComponentName.tsx`
2. 在`src/components/__tests__/ComponentName.test.tsx`添加测试
3. 从`src/components/index.ts`导出

### 数据库迁移
1. 变更`src/db/schema.ts`
2. 执行`pnpm db:generate`
3. 执行`pnpm db:migrate`

## 测试模式

- 测试文件: `__tests__/`目录或`.test.ts`后缀
- 覆盖率目标: 80%以上
- 框架: Vitest
```

## GitHub App集成

关于高级功能(10k以上提交、团队共享、自动PR)，请使用[Skill Creator GitHub App](https://github.com/apps/skill-creator):

- 安装: [github.com/apps/skill-creator](https://github.com/apps/skill-creator)
- 在任意issue评论`/skill-creator analyze`
- 接收包含生成技能的PR

## 相关命令

- `/instinct-import` - 导入生成的instincts
- `/instinct-status` - 显示已学习的instincts
- `/evolve` - 将instincts聚类为技能/代理

---

*[Everything Claude Code](https://github.com/affaan-m/everything-claude-code)的一部分*
