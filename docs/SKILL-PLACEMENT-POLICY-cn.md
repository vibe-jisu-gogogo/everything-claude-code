# Skill Placement and Provenance Policy

本文档定义了生成、导入和精选 skill 的存放位置、标识方式以及哪些内容会被发布。

## Skill Types and Placement

| 类型 | 根路径 | 是否发布 | Provenance |
|------|--------|----------|------------|
| Curated | `skills/` (仓库内) | 是 | 不需要 |
| Learned | `~/.claude/skills/learned/` | 否 | 需要 |
| Imported | `~/.claude/skills/imported/` | 否 | 需要 |
| Evolved | `~/.claude/homunculus/evolved/skills/` (全局) 或 `projects/<hash>/evolved/skills/` (项目级) | 否 | 继承自 instinct 源 |

Curated skill 存放在仓库的 `skills/` 目录下。安装清单仅引用精选路径。生成和导入的 skill 存放在用户主目录下，永远不会被发布。

## Curated Skills

位置：`skills/<skill-name>/`，根目录下有 `SKILL.md`。

- 包含在 `manifests/install-modules.json` 的路径中。
- 由 `scripts/ci/validate-skills.js` 验证。
- 不需要 provenance 文件。在 SKILL.md 的 frontmatter 中使用 `origin`（ECC, community）进行归属说明。

## Learned Skills

位置：`~/.claude/skills/learned/<skill-name>/`。

由持续学习（evaluate-session hook、/learn 命令）创建。默认路径可通过 `skills/continuous-learning/config.json` → `learned_skills_path` 配置。

- 不在仓库中，不发布。
- 必须在 `SKILL.md` 同级目录有 `.provenance.json` 文件。
- 目录存在时在运行时加载。

## Imported Skills

位置：`~/.claude/skills/imported/<skill-name>/`。

从外部来源（URL、文件复制等）用户安装的 skill。目前还没有自动导入器；按约定存放。

- 不在仓库中，不发布。
- 必须在 `SKILL.md` 同级目录有 `.provenance.json` 文件。

## Evolved Skills (Continuous Learning v2)

位置：`~/.claude/homunculus/evolved/skills/` (全局) 或 `~/.claude/homunculus/projects/<hash>/evolved/skills/` (项目级)。

由 instinct-cli evolve 从聚类的 instincts 生成。与 learned/imported 是独立系统。

- 不在仓库中，不发布。
- Provenance 继承自源 instincts；不需要单独的 `.provenance.json`。

## Provenance Metadata

Learned 和 imported skill 需要。文件：skill 目录中的 `.provenance.json`。

必填字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| source | string | 来源（URL、路径或标识符） |
| created_at | string | ISO 8601 时间戳 |
| confidence | number | 0–1 |
| author | string | 谁或什么产生了该 skill |

Schema：`schemas/provenance.schema.json`。验证：`scripts/lib/skill-evolution/provenance.js` → `validateProvenance`。

## Validator Behavior

### validate-skills.js

范围：仅 Curated skill（仓库中的 `skills/`）。

- 如果 `skills/` 不存在：exit 0（没有需要验证的内容）。
- 对于每个子目录：必须包含 `SKILL.md` 且非空。
- 不接触 learned/imported/evolved 根目录。

### validate-install-manifests.js

范围：仅 Curated 路径。模块中的所有 `paths` 必须在仓库中存在。

- 生成/导入的根目录不在范围内。清单不引用它们。
- 路径缺失 → 错误。没有可选路径处理。

### 使用生成路径的脚本

`scripts/skills-health.js`、`scripts/lib/skill-evolution/health.js`、session hooks：它们探测 `~/.claude/skills/learned` 和 `~/.claude/skills/imported`。目录不存在时视为空；不报错。

## Publishable vs Local-Only

| 可发布 | 仅本地 |
|--------|--------|
| `skills/*` (curated) | `~/.claude/skills/learned/*` |
| | `~/.claude/skills/imported/*` |
| | `~/.claude/homunculus/**/evolved/**` |

只有 curated skill 出现在安装清单中并在安装时被复制。

## Implementation Roadmap

1. 策略文档和 provenance schema（本次变更）。
2. 在 learned-skill 写入路径（evaluate-session、/learn 输出）中添加 provenance 验证，确保新的 learned skill 始终有 `.provenance.json`。
3. 更新 instinct-cli evolve 在生成 evolved skill 时写入可选的 provenance。
4. 如果需要，在 CI 中添加 `scripts/validate-provenance.js` 检查任何不应包含 learned/imported 内容的仓库路径。
5. 在 CONTRIBUTING.md 或用户文档中记录 learned/imported 根目录，让贡献者知道不要提交它们。
