# 调研任务与进度

状态：资料分析＝已形成有来源的阶段性文档；待深化＝已有结论但证据不覆盖全部维度；待验证＝尚未做实验。不是排期承诺。

| ID | 任务与问题 | 产物 | 状态/退出条件 |
|---|---|---|---|
| R01 | 界定主问题与证据标准 | [研究章程](charter.md) | 已完成本轮范围定义 |
| R02 | 六家厂商如何接入、运行、交付 | [厂商目录](../docs/vendors/index.md) | 资料分析；精确版本/区域与价格待深化 |
| R03 | 比较查询、副本、快照、CDC、API | [交付模式](../docs/patterns/delivery.md) | 资料分析；退出条件为每种模式有选用及拒绝条件 |
| R04 | 删除、回补、重放、跨表一致性 | [正确性](../docs/patterns/correctness.md) | 资料分析；故障测试待验证 |
| R05 | 数据权限与用途授权的差别 | [治理](../docs/patterns/governance.md) | 设计建议；厂商治理实现待深化 |
| R06 | MySQL/Mongo/ES/Hive/DWS 差异 | [来源适配](../docs/sources/source-adapters.md) | Mongo/ES 有新增官方证据；Hive/DWS 深挖待办 |
| R07 | 对现有平台逐项映射 | [匹配分析](../docs/scenario/fit.md) | 条件化分析完成，未知参数不做虚假评分 |
| R08 | 执行及发布参考架构 | [架构](../docs/scenario/architecture.md) | 设计建议；取消、权限、发布待实现验证 |
| R09 | 负载与成本验证 | [验证计划](../docs/scenario/validation.md) | 计划完成、测试未执行 |
| R10 | 形成可追溯汇总 | [汇总](../reports/summary.md) | 阶段版，随专题证据更新 |

## 下一轮具体工作项

1. 华为：分别读取 MySQL/Mongo/ES/Hive/DWS 的 CDM 连接器限制；独立确认 DRS 的源目标组合，不能用 CDM 文档推断 DRS。
2. 治理：分别研究 AWS Lake Formation/IAM、Unity Catalog、DataWorks 权限、华为数据治理的源权限和复制后权限边界；对不上 OBS 的能力明确排除。
3. 补充独立集成产品及开源实现（如 Debezium、Airbyte、Fivetran）的快照/增量交接与运维成本，作为候选研究，当前不声称已评估。
4. 查明 MySQL binlog 配置、Mongo 拓扑、ES 版本与 Hive/DWS 队列语义的兼容矩阵。
5. 用合成数据执行 R09 后，才填写性能数字和量化成本。

## 任务依赖

R01 → R02/R06 → R03/R04 → R07/R08 → R09 → R10；R05 横跨接入、处理、发布和消费。当前汇总依赖公开证据，R09 未完成不会伪装为已验证。
