---
name: iterative-retrieval
description: 为解决子代理上下文问题而逐步细化上下文获取的模式
---

# 迭代检索模式

解决多代理工作流中的"上下文问题"。子代理在开始工作之前不知道需要什么上下文。

## 问题

子代理在有限的上下文中启动。不知道：
- 哪些文件包含相关代码
- 代码库中存在什么模式
- 项目使用什么术语

标准方法会失败：
- **发送所有内容**: 超出上下文限制
- **不发送任何内容**: 代理缺少重要信息
- **猜测需要什么**: 经常出错

## 解决方案: 迭代检索

逐步细化上下文的4阶段循环：

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
│        最多3个循环，然后继续执行              │
└─────────────────────────────────────────────┘
```

### 阶段1: DISPATCH

收集候选文件的初始广泛查询：

```javascript
// 从高层意图开始
const initialQuery = {
  patterns: ['src/**/*.ts', 'lib/**/*.ts'],
  keywords: ['authentication', 'user', 'session'],
  excludes: ['*.test.ts', '*.spec.ts']
};

// 分派给检索代理
const candidates = await retrieveFiles(initialQuery);
```

### 阶段2: EVALUATE

评估获取内容的相关性：

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
- **高(0.8-1.0)**: 直接实现目标功能
- **中(0.5-0.7)**: 包含相关模式或类型
- **低(0.2-0.4)**: 间接相关
- **无(0-0.2)**: 不相关，排除

### 阶段3: REFINE

基于评估更新检索标准：

```javascript
function refineQuery(evaluation, previousQuery) {
  return {
    // 添加在高相关性文件中发现的新模式
    patterns: [...previousQuery.patterns, ...extractPatterns(evaluation)],

    // 添加在代码库中找到的术语
    keywords: [...previousQuery.keywords, ...extractKeywords(evaluation)],

    // 排除已确认不相关的路径
    excludes: [...previousQuery.excludes, ...evaluation
      .filter(e => e.relevance < 0.2)
      .map(e => e.path)
    ],

    // 目标特定差距
    focusAreas: evaluation
      .flatMap(e => e.missingContext)
      .filter(unique)
  };
}
```

### 阶段4: LOOP

使用细化后的标准重复（最多3个循环）：

```javascript
async function iterativeRetrieve(task, maxCycles = 3) {
  let query = createInitialQuery(task);
  let bestContext = [];

  for (let cycle = 0; cycle < maxCycles; cycle++) {
    const candidates = await retrieveFiles(query);
    const evaluation = evaluateRelevance(candidates, task);

    // 检查是否有足够的上下文
    const highRelevance = evaluation.filter(e => e.relevance >= 0.7);
    if (highRelevance.length >= 3 && !hasCriticalGaps(evaluation)) {
      return highRelevance;
    }

    // 细化并继续
    query = refineQuery(evaluation, query);
    bestContext = mergeContext(bestContext, highRelevance);
  }

  return bestContext;
}
```

## 实际示例

### 示例1: 修复bug上下文

```
任务: "修复认证令牌过期bug"

循环1:
  DISPATCH: 在src/**中检索"token"、"auth"、"expiry"
  EVALUATE: 发现auth.ts(0.9)、tokens.ts(0.8)、user.ts(0.3)
  REFINE: 添加"refresh"、"jwt"关键词；排除user.ts

循环2:
  DISPATCH: 使用细化后的术语检索
  EVALUATE: 发现session-manager.ts(0.95)、jwt-utils.ts(0.85)
  REFINE: 上下文足够（2个高相关性文件）

结果: auth.ts、tokens.ts、session-manager.ts、jwt-utils.ts
```

### 示例2: 功能实现

```
任务: "为API端点添加速率限制"

循环1:
  DISPATCH: 在routes/**中检索"rate"、"limit"、"api"
  EVALUATE: 无匹配 - 代码库使用"throttle"术语
  REFINE: 添加"throttle"、"middleware"关键词

循环2:
  DISPATCH: 使用细化后的术语检索
  EVALUATE: 发现throttle.ts(0.9)、middleware/index.ts(0.7)
  REFINE: 需要路由器模式

循环3:
  DISPATCH: 检索"router"、"express"模式
  EVALUATE: 发现router-setup.ts(0.8)
  REFINE: 上下文足够

结果: throttle.ts、middleware/index.ts、router-setup.ts
```

## 与代理集成

在代理提示中使用：

```markdown
获取此任务的上下文时：
1. 从广泛的关键词检索开始
2. 评估每个文件的相关性（0-1评分）
3. 识别仍然缺少的上下文
4. 细化检索标准并重复（最多3个循环）
5. 返回相关性>=0.7的文件
```

## 最佳实践

1. **广泛开始，逐步缩小** - 初始查询不要过于具体
2. **学习代码库术语** - 第一个循环通常会揭示命名约定
3. **跟踪缺少的内容** - 明确的差距识别促进细化
4. **在"足够好"时停止** - 3个高相关性文件优于10个普通文件
5. **有信心地排除** - 低相关性文件不会变得相关

## 相关项目

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 子代理编排部分
- `continuous-learning`技能 - 用于随时间改进的模式
- `~/.claude/agents/`中的代理定义
