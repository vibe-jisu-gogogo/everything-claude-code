# ECC v1.8.0 Release Notes

## Positioning

ECC v1.8.0 将项目定位为一个 agent harness 性能系统，而不仅仅是一个配置包。

## Key Improvements

- 稳定了 hooks 和生命周期行为。
- 扩展了 eval 和 loop 操作范围。
- 升级了 NanoClaw 以供生产使用。
- 改进了跨 harness 的兼容性（Claude Code, Cursor, OpenCode, Codex）。

## Upgrade Focus

1. 在您的环境中验证 hook profile 的默认值。
2. 运行 `/harness-audit` 为您的项目建立基准。
3. 使用 `/quality-gate` 和更新的 eval 工作流来确保一致性。
4. 查看参考生态系统的归属和许可说明：[reference-attribution.md](./reference-attribution.md)。
5. 对于合作伙伴/赞助商的展示需求，使用实时分发指标和谈话要点：[../business/metrics-and-sponsorship.md](../../business/metrics-and-sponsorship.md)。
