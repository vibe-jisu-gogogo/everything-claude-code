# 规则

## 必须始终遵守
- 对于领域任务，委托给 specialized agents 处理。
- 在实现代码前编写测试，并验证关键路径。
- 验证输入并保持安全检查完整。
- 优先使用 immutable updates，而不是修改共享状态。
- 在发明新模式前，遵循仓库已有的模式。
- 保持贡献内容聚焦、可评审且描述清晰。

## 绝对禁止
- 在输出中包含敏感数据，如 API keys、tokens、secrets 或绝对/系统文件路径。
- 提交未经测试的变更。
- 绕过安全检查或 validation hooks。
- 没有明确理由的情况下，重复实现已有的功能。
- 不检查相关测试套件就发布代码。

## Agent 格式
- Agents 存放在 `agents/*.md` 目录下。
- 每个文件包含 YAML frontmatter，字段包括 `name`、`description`、`tools` 和 `model`。
- 文件名使用小写字母和连字符，且必须与 agent 名称匹配。
- 描述必须清晰说明应该何时调用该 agent。

## Skill 格式
- Skills 存放在 `skills/<name>/SKILL.md` 目录下。
- 每个 skill 包含 YAML frontmatter，字段包括 `name`、`description` 和 `origin`。
- 第一方 skill 使用 `origin: ECC`，导入/社区 skill 使用 `origin: community`。
- Skill 主体应包含实用指导、经过测试的示例和清晰的"何时使用"章节。

## Hook 格式
- Hooks 使用 matcher-driven JSON 注册方式，以及 shell 或 Node 入口点。
- Matchers 应具体明确，而不是宽泛的全匹配。
- 仅在有意阻止行为时 exit code 为 `1`；否则 exit code 为 `0`。
- 错误和信息消息应具备可操作性。

## 提交风格
- 使用 conventional commits，例如 `feat(skills):`、`fix(hooks):` 或 `docs:`。
- 保持变更模块化，并在 PR 摘要中说明对用户的影响。
