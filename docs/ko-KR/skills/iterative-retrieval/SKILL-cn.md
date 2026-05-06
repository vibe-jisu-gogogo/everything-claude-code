---
name: iterative-retrieval
description: 用于解决子代理上下文问题的渐进式上下文检索改进模式
origin: ECC
---

# 迭代检索模式

解决多代理工作流中的"上下文问题"——子代理在开始工作之前无法知道需要哪些上下文。

## 激活时机

- 创建需要无法预先预测的代码库上下文的子代理时
- 构建上下文渐进式改进的多代理工作流时
- 在代理任务中遇到"上下文超限"或"上下文缺失"失败时
- 设计用于代码探索的类RAG检索管道时
- 在代理编排中优化token使用量时

## 问题

子代理以有限的上下文创建，无法知道：
- 包含相关代码的文件
- 代码库中存在的模式
- 项目中使用的术语

标准方法的失败：
- **发送所有内容**：超出上下文限制
- **不发送任何内容**：代理缺少重要信息
- **猜测需要什么**：经常出错

## 解决方案：迭代检索

渐进式改进上下文的4步循环：

```
┌─────────────────────────────────────────────┐
│                                             │
│   ┌──────────┐      ┌──────────┐            │
│   │ DISPATCH │─────│ EVALUATE │            │
│   └──────────┘      └──────────┘            │
│        ▲                  │                 │
│        │                  ▼                 │
│   ┌──────────┐      ┌──────────┐            │
│   │   LOOP   │─────│  REFINE  │            │
│   └──────────┘      └──────────┘            │
│                                             │
│        Max 3 cycles, then proceed           │
└─────────────────────────────────────────────┘
```

### 第1步：DISPATCH

收集候选文件的初始广泛查询：

```javascript
// Start with high-level intent
const initialQuery = {
  patterns: ['src/**/*.ts', 'lib/**/*.ts'],
  keywords: ['authentication', 'user', 'session'],
  excludes: ['*.test.ts', '*.spec.ts']
};

// Dispatch to retrieval agent
const candidates = await retrieveFiles(initialQuery);
```

### 第2步：EVALUATE

评估检索内容的相关性：

```javascript
function evaluateRelevance(files, task) {
  return files.map(file => ({
    path: file.path,
    relevance: scoreRelevance(file.content, task),
    reason: explainRelevance(file.content, task),
    missingContext: identifyGaps(file.content, task)
  }));
}
```

评分标准：
- **高 (0.8-1.0)**：直接实现目标功能
- **中 (0.5-0.7)**：包含相关模式或类型
- **低 (0.2-0.4)**：间接相关
- **无 (0-0.2)**：不相关，排除

### 第3步：REFINE

基于评估更新检索标准：

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // Add new patterns discovered in high-relevance files
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // Add terminology found in codebase
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // Exclude confirmed irrelevant paths
    excludes: [...previousQuery.excludes, ...evaluation
      .filter(e => e.relevance < 0.2)
      .map(e => e.path)
    ],

    // Target specific gaps
    focusAreas: evaluation
      .flatMap(e => e.missingContext)
      .filter(unique)
  };
}
```

### 第4步：LOOP

使用改进的标准重复（最多3次）：

```javascript
async function iterativeRetrieve(task, maxCycles = 3) {
  let query = createInitialQuery(task);
  let bestContext = [];

  for (let cycle = 0; cycle < maxCycles; cycle++) {
    const candidates = await retrieveFiles(query);
    const evaluation = evaluateRelevance(candidates, task);

    // Check if we have sufficient context
    const highRelevance = evaluation.filter(e => e.relevance >= 0.7);
    if (highRelevance.length >= 3 && !hasCriticalGaps(evaluation)) {
      return highRelevance;
    }

    // Refine and continue
    query = refineQuery(evaluation, query);
    bestContext = mergeContext(bestContext, highRelevance);
  }

  return bestContext;
}
```

## 实际示例

### 示例1：Bug修复上下文

```
Task: "Fix the authentication token expiry bug"

Cycle 1:
  DISPATCH: Search for "token", "auth", "expiry" in src/**
  EVALUATE: Found auth.ts (0.9), tokens.ts (0.8), user.ts (0.3)
  REFINE: Add "refresh", "jwt" keywords; exclude user.ts

Cycle 2:
  DISPATCH: Search refined terms
  EVALUATE: Found session-manager.ts (0.95), jwt-utils.ts (0.85)
  REFINE: Sufficient context (2 high-relevance files)

Result: auth.ts, tokens.ts, session-manager.ts, jwt-utils.ts
```

### 示例2：功能实现

```
Task: "Add rate limiting to API endpoints"

Cycle 1:
  DISPATCH: Search "rate", "limit", "api" in routes/**
  EVALUATE: No matches - codebase uses "throttle" terminology
  REFINE: Add "throttle", "middleware" keywords

Cycle 2:
  DISPATCH: Search refined terms
  EVALUATE: Found throttle.ts (0.9), middleware/index.ts (0.7)
  REFINE: Need router patterns

Cycle 3:
  DISPATCH: Search "router", "express" patterns
  EVALUATE: Found router-setup.ts (0.8)
  REFINE: Sufficient context

Result: throttle.ts, middleware/index.ts, router-setup.ts
```

## 与代理集成

在代理提示中使用：

```markdown
When retrieving context for this task:
1. Start with broad keyword search
2. Evaluate each file's relevance (0-1 scale)
3. Identify what context is still missing
4. Refine search criteria and repeat (max 3 cycles)
5. Return files with relevance >= 0.7
```

## 最佳实践

1. **广泛开始，逐步缩小** - 不要过度指定初始查询
2. **学习代码库术语** - 命名约定通常在第一个周期中显现
3. **跟踪缺失的内容** - 明确的差距识别推动改进
4. **在"足够好"时停止** - 3个高相关性文件胜过10个普通文件
5. **自信地排除** - 低相关性文件不会变得更相关

## 相关条目

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 子代理编排部分
- `continuous-learning` 技能 - 随时间改进的模式
- `~/.claude/agents/` 中的代理定义
