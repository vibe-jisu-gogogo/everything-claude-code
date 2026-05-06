# 代码映射更新

分析代码库结构并更新架构文档。

1. 扫描所有源文件的 import、export、依赖关系
2. 按以下格式生成 token 高效的 codemap:
   - codemaps/architecture.md - 整体架构
   - codemaps/backend.md - 后端结构
   - codemaps/frontend.md - 前端结构
   - codemaps/data.md - 数据模型和 schema

3. 计算与前一版本的差异百分比
4. 如果变更超过 30%，在更新前请求用户确认
5. 为每个 codemap 添加 freshness timestamp
6. 将报告保存到 .reports/codemap-diff.txt

使用 TypeScript/Node.js 进行分析。重点关注高层结构，而非实现细节。
