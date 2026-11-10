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

## 首轮提出的深化项（保留原计划）

1. 华为：分别读取 MySQL/Mongo/ES/Hive/DWS 的 CDM 连接器限制；独立确认 DRS 的源目标组合，不能用 CDM 文档推断 DRS。
2. 治理：分别研究 AWS Lake Formation/IAM、Unity Catalog、DataWorks 权限、华为数据治理的源权限和复制后权限边界；对不上 OBS 的能力明确排除。
3. 补充独立集成产品及开源实现（如 Debezium、Airbyte、Fivetran）的快照/增量交接与运维成本，作为候选研究，当前不声称已评估。
4. 查明 MySQL binlog 配置、Mongo 拓扑、ES 版本与 Hive/DWS 队列语义的兼容矩阵。
5. 用合成数据执行 R09 后，才填写性能数字和量化成本。

## 任务依赖

R01 → R02/R06 → R03/R04 → R07/R08 → R09 → R10；R05 横跨接入、处理、发布和消费。当前汇总依赖公开证据，R09 未完成不会伪装为已验证。

## 深挖轮次的实际完成情况

本轮成果见[12 篇教程](../docs/deep-dive/README.md)和[过程记录](rounds/2026-11-10-deep-dive/journal.md)。上面的原计划不是当前待办清单。

| 工作 | 当前结果 | 仍然未知 |
|---|---|---|
| 华为五类来源、DRS | 已读逐来源限制及 MySQL → Kafka 链路 | 部署版本的组合兼容性、性能 |
| 独立连接器 | 已分析 Airbyte、Fivetran、Debezium、Confluent | 具体商业套餐、运行成本 |
| 湖仓第一跳 | 已拆解 Databricks、Snowflake、Fabric、AWS 及国内云 | 各区域/版本/目标的完整矩阵 |
| 企业案例 | 已读 Uber、LinkedIn、Netflix 原始工程资料 | 不将历史材料解释为当前全公司架构 |
| 治理边界 | 已分析 Lake Formation 与 Unity Catalog，以及复制后授权 | 国内云治理产品的完整策略等价性未验证 |
| 来源语义 | 已补 Hive/DWS 和各来源错误恢复检查点 | 未对所有引擎版本作全量兼容测试 |
| 方案与验收 | 已给出三种架构、故障场景和成本计算方法 | 未执行压测、未给出价格与吞吐承诺 |

资料研究已形成可审阅结果。剩余兼容性和性能问题应按选定候选与引擎版本缩小范围后验证，不能由公开文档直接替代。
