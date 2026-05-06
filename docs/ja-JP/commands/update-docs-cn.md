# Update Documentation

从可靠来源同步文档：

1. 读取 package.json 的 scripts 部分
   - 生成脚本参考表
   - 包含来自注释的说明

2. 读取 .env.example
   - 提取所有环境变量
   - 记录目的和格式

3. 生成 docs/CONTRIB.md：
   - 开发工作流
   - 可用脚本
   - 环境设置
   - 测试步骤

4. 生成 docs/RUNBOOK.md：
   - 部署步骤
   - 监控与告警
   - 常见问题与修复
   - 回滚步骤

5. 识别过期文档：
   - 检测超过 90 天未修改的文档
   - 列出以供手动审查

6. 显示差异摘要

唯一可信的来源：package.json 和 .env.example
