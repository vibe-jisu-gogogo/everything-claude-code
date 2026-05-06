# Execute - 多模型协同实现

多模型协同实现 - 从计划获取原型 → Claude进行重构和实现 → 多模型审计与交付。

$ARGUMENTS

---

## 核心协议

- **语言协议**: 与工具/模型交互时使用**English**，与用户交互时使用用户的语言
- **代码主权**: 外部模型对**文件系统零写入权限**，所有变更由Claude执行
- **脏原型重构**: 将Codex/Gemini的统一差异视为"脏原型"，需重构为生产级代码
- **损失限制机制**: 当前阶段输出未通过验证前，不进入下一阶段
- **前提条件**: 仅在用户对`/ccg:plan`输出明确回复"Y"后执行(缺失时需先确认)

---

## 多模型调用规范

**调用语法**(并行: 使用`run_in_background: true`):

```
# 会话恢复调用(推荐) - 实现原型
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Requirement: <任务描述>
Context: <计划内容 + 目标文件>
</TASK>
OUTPUT: 仅统一差异补丁。严格禁止实际变更。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁描述"
})

# 新会话调用 - 实现原型
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}- \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Requirement: <任务描述>
Context: <计划内容 + 目标文件>
</TASK>
OUTPUT: 仅统一差异补丁。严格禁止实际变更。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁描述"
})
```

**审计调用语法**(代码审查/审计):

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper {{LITE_MODE_FLAG}}--backend <codex|gemini> {{GEMINI_MODEL_FLAG}}resume <SESSION_ID> - \"$PWD\" <<'EOF'
ROLE_FILE: <角色提示词路径>
<TASK>
Scope: 审计最终代码变更。
Inputs:
- 已应用的补丁(git diff / 最终统一差异)
- 变更的文件(必要时附相关摘录)
Constraints:
- 不修改文件。
- 不输出假设文件系统访问的工具命令。
</TASK>
OUTPUT:
1) 按优先级排序的问题列表(严重性、文件、依据)
2) 具体修复；如需要代码变更，在围栏代码块中包含统一差异补丁。
EOF",
  run_in_background: true,
  timeout: 3600000,
  description: "简洁描述"
})
```

**模型参数注意事项**:
- `{{GEMINI_MODEL_FLAG}}`: 使用`--backend gemini`时替换为`--gemini-model gemini-3-pro-preview`(注意末尾空格)；codex时使用空字符串

**角色提示词**:

| 阶段 | Codex | Gemini |
|-------|-------|--------|
| 实现 | `~/.claude/.ccg/prompts/codex/architect.md` | `~/.claude/.ccg/prompts/gemini/frontend.md` |
| 审查 | `~/.claude/.ccg/prompts/codex/reviewer.md` | `~/.claude/.ccg/prompts/gemini/reviewer.md` |

**会话复用**: 若`/ccg:plan`提供了SESSION_ID，使用`resume <SESSION_ID>`复用上下文。

**后台任务等待**(最大超时600000ms = 10分钟):

```
TaskOutput({ task_id: "<task_id>", block: true, timeout: 600000 })
```

**重要**:
- 必须指定`timeout: 600000`，否则默认30秒会提前超时
- 10分钟后仍未完成时，继续用`TaskOutput`轮询，**不强制终止进程**
- 因超时跳过等待时，**必须调用`AskUserQuestion`询问用户是继续等待还是强制终止任务**

---

## 执行工作流

**执行任务**: $ARGUMENTS

### 阶段 0: 计划读取

`[Mode: Prepare]`

1. **输入类型识别**:
   - 计划文件路径(例: `.claude/plan/xxx.md`)
   - 直接任务描述

2. **计划内容读取**:
   - 提供计划文件路径时，读取并解析
   - 提取: 任务类型、实现步骤、关键文件、SESSION_ID

3. **执行前确认**:
   - 输入为"直接任务描述"或计划缺失`SESSION_ID`/关键文件时: 先向用户确认
   - 无法确认用户已对计划回复"Y"时: 需在推进前再次确认

4. **任务类型路由**:

   | 任务类型 | 检测 | 路由 |
   |-----------|-----------|-------|
   | **前端** | 页面、组件、UI、样式、布局 | Gemini |
   | **后端** | API、接口、数据库、逻辑、算法 | Codex |
   | **全栈** | 同时包含前端和后端 | Codex ∥ Gemini 并行 |

---

### 阶段 1: 快速上下文获取

`[Mode: Retrieval]`

**ace-tool MCP可用时**，用于快速上下文获取:

基于计划的"关键文件"列表，调用`mcp__ace-tool__search_context`:

```
mcp__ace-tool__search_context({
  query: "<基于计划内容的语义查询，包含关键文件、模块、函数名>",
  project_root_path: "$PWD"
})
```

**获取策略**:
- 从计划的"关键文件"表提取目标路径
- 构建覆盖范围的语义查询: 入口文件、依赖模块、相关类型定义
- 结果不足时，追加1-2次递归获取

**ace-tool MCP不可用时**，使用Claude Code内置工具回退:
1. **Glob**: 从计划的"关键文件"表搜索目标文件 (例: `Glob("src/components/**/*.tsx")`)
2. **Grep**: 在整个代码库中搜索关键符号、函数名、类型定义
3. **Read**: 读取发现的文件，收集完整上下文
4. **Task (Explore 代理)**: 需要更广泛探索时，使用`subagent_type: "Explore"`的`Task`

**获取后**:
- 整理获取的代码片段
- 确认实现所需的完整上下文
- 进入阶段3

---

### 阶段 3: 原型获取

`[Mode: Prototype]`

**基于任务类型路由**:

#### 路由 A: 前端/UI/样式 → Gemini

**限制**: 上下文 < 32k tokens

1. 调用Gemini(使用`~/.claude/.ccg/prompts/gemini/frontend.md`)
2. 输入: 计划内容 + 获取的上下文 + 目标文件
3. OUTPUT: `仅统一差异补丁。严格禁止实际变更。`
4. **Gemini是前端设计权威，其CSS/React/Vue原型是最终视觉基准**
5. **警告**: 忽略Gemini的后端逻辑提案
6. 计划包含`GEMINI_SESSION`时: 优先使用`resume <GEMINI_SESSION>`

#### 路由 B: 后端/逻辑/算法 → Codex

1. 调用Codex(使用`~/.claude/.ccg/prompts/codex/architect.md`)
2. 输入: 计划内容 + 获取的上下文 + 目标文件
3. OUTPUT: `仅统一差异补丁。严格禁止实际变更。`
4. **Codex是后端逻辑权威，利用其逻辑推理和调试能力**
5. 计划包含`CODEX_SESSION`时: 优先使用`resume <CODEX_SESSION>`

#### 路由 C: 全栈 → 并行调用

1. **并行调用**(`run_in_background: true`):
   - Gemini: 处理前端部分
   - Codex: 处理后端部分
2. 用`TaskOutput`等待两个模型的完整结果
3. 分别使用计划中对应的`SESSION_ID`进行`resume`(缺失时创建新会话)

**请遵循上述`多模型调用规范`的`重要`指示**

---

### 阶段 4: 代码实现

`[Mode: Implement]`

**Claude作为代码主权者执行以下步骤**:

1. **差异读取**: 解析Codex/Gemini返回的统一差异补丁

2. **思维沙盒**:
   - 模拟差异应用到目标文件
   - 检查逻辑一致性
   - 识别潜在冲突和副作用

3. **重构与清理**:
   - 将"脏原型"重构为**高可读性、可维护性、企业级代码**
   - 删除冗余代码
   - 确保符合项目现有代码标准
   - **非必要不生成注释/文档**，代码应自解释

4. **最小化范围**:
   - 变更仅限需求范围内
   - **必须审查**副作用
   - 实施针对性修复

5. **应用变更**:
   - 使用Edit/Write工具执行实际变更
   - **仅修改必要代码**，不影响用户其他现有功能

6. **自我验证**(强烈推荐):
   - 运行项目现有lint / typecheck / tests(优先最小相关范围)
   - 失败时: 先修复回归，再进入阶段5

---

### 阶段 5: 审计与交付

`[Mode: Audit]`

#### 5.1 自动审计

**变更生效后，必须立即并行调用Codex和Gemini执行代码审查**:

1. **Codex审查**(`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/codex/reviewer.md`
   - 输入: 变更的差异 + 目标文件
   - 关注: 安全性、性能、错误处理、逻辑正确性

2. **Gemini审查**(`run_in_background: true`):
   - ROLE_FILE: `~/.claude/.ccg/prompts/gemini/reviewer.md`
   - 输入: 变更的差异 + 目标文件
   - 关注: 可访问性、设计一致性、用户体验

用`TaskOutput`等待两个模型的完整审查结果。为保持上下文一致性，优先复用阶段3的会话(`resume <SESSION_ID>`)。

#### 5.2 整合与修复

1. 整合Codex + Gemini审查反馈
2. 基于信任规则加权: 后端遵循Codex，前端遵循Gemini
3. 执行必要修复
4. 必要时重复阶段5.1(直到风险可接受)

#### 5.3 交付确认

审计通过后，向用户报告:

```markdown
## 实现完成

### 变更摘要
| 文件 | 操作 | 说明 |
|------|-----------|-------------|
| path/to/file.ts | 变更 | 说明 |

### 审计结果
- Codex: <通过/发现N个问题>
- Gemini: <通过/发现N个问题>

### 推荐事项
1. [ ] <推荐测试步骤>
2. [ ] <推荐验证步骤>
```

---

## 重要规则

1. **代码主权** – 所有文件变更由Claude执行，外部模型零写入权限
2. **脏原型重构** – Codex/Gemini输出视为草稿，需重构
3. **信任规则** – 后端遵循Codex，前端遵循Gemini
4. **最小化变更** – 仅修改必要代码，无副作用
5. **强制审计** – 变更后必须执行多模型代码审查

---

## 使用方法

```bash
# 执行计划文件
/ccg:execute .claude/plan/feature-name.md

# 直接执行任务(上下文中已讨论计划时)
/ccg:execute 基于之前的计划实现用户认证
```

---

## 与/ccg:plan的关系

1. `/ccg:plan`生成计划 + SESSION_ID
2. 用户以"Y"确认
3. `/ccg:execute`读取计划，复用SESSION_ID，执行实现
