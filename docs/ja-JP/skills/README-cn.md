# Skills

Skills 是 Claude Code 根据上下文加载的知识模块，包含 workflow 定义和 domain 知识。

## Skill Categories

### 按语言分类的 Patterns
- `python-patterns/` - Python 设计 patterns
- `golang-patterns/` - Go 设计 patterns
- `frontend-patterns/` - React/Next.js patterns
- `backend-patterns/` - API 和 database patterns

### 按语言分类的 Testing
- `python-testing/` - Python testing 策略
- `golang-testing/` - Go testing 策略
- `cpp-testing/` - C++ testing

### Frameworks
- `django-patterns/` - Django best practices
- `django-tdd/` - Django test-driven development
- `django-security/` - Django security
- `springboot-patterns/` - Spring Boot patterns
- `springboot-tdd/` - Spring Boot testing
- `springboot-security/` - Spring Boot security

### Databases
- `postgres-patterns/` - PostgreSQL patterns
- `jpa-patterns/` - JPA/Hibernate patterns

### Security
- `security-review/` - Security checklist
- `security-scan/` - Security scan

### Workflows
- `tdd-workflow/` - Test-driven development workflow
- `continuous-learning/` - 持续学习

### 特定 Domain
- `eval-harness/` - 评估 harness
- `iterative-retrieval/` - 迭代式 retrieval

## Skill 结构

每个 skill 在自己的目录中包含 SKILL.md 文件：

```
skills/
├── python-patterns/
│   └── SKILL.md          # 实现 patterns、示例、best practices
├── golang-testing/
│   └── SKILL.md
├── django-patterns/
│   └── SKILL.md
...
```

## 使用 Skills

Claude Code 会根据上下文自动加载 skills。例如：

- 正在编辑 Python 文件 → 加载 `python-patterns` 和 `python-testing`
- 是 Django 项目 → 加载 `django-*` skills
- 正在进行 test-driven development → 加载 `tdd-workflow`

## 创建 Skills

创建新 skill 的步骤：

1. 创建 `skills/your-skill-name/` 目录
2. 添加 `SKILL.md` 文件
3. 模板：

```markdown
---
name: your-skill-name
description: Brief description shown in skill list
---

# Your Skill Title

Brief overview.

## Core Concepts

Key patterns and guidelines.

## Code Examples

```language
// Practical, tested examples
```

## Best Practices

- Actionable guideline 1
- Actionable guideline 2

## When to Use

Describe scenarios where this skill applies.
```

---

**请记住**：Skills 是参考资料。它们提供实现指导并展示 best practices。结合使用 skills 和 rules 来确保高质量代码。
