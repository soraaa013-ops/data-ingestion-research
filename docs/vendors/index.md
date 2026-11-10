# 厂商研究目录

建议先读 [README 的工单案例](../../README.md)，再带着一个具体问题进入这里。各篇使用同一组虚构工单解释机制；它们不是厂商客户案例。

## 按问题阅读

1. 想判断现在的 SQL 导入是否合理？先看[阿里云](alibaba.md)：自定义 SQL 是正式能力，但最终查询与批准范围如何对应？
2. 想解决多个团队一起抽取的负载？看[华为](huawei.md)：任务怎样排队，集群并发为什么不等于源数据库预算？
3. 单次导出本身就太重？看[AWS](aws.md)：从已有快照导出，改变了哪一部分工作的位置？
4. 不想每次重读全部历史？看[Microsoft](microsoft.md)：保存上次进度能解决什么，又为什么会漏掉修改或删除？
5. 考虑是否需要 CDC？看[Databricks](databricks.md)：每周训练与持续采集为什么是两个问题？
6. 变更文件已经落盘，准备直接训练？看[Google](google.md)：用一条工单的乱序修改说明为什么还要重建状态。

## 按厂商查找

| 专题 | 重点 | 与本场景最相关的观察 |
|---|---|---|
| [AWS](aws.md) | DMS / RDS snapshot | 原始变更文件与训练快照之间存在处理环节 |
| [Microsoft](microsoft.md) | ADF / Fabric | 批量查询、水位、CDC 是并存选项 |
| [Google](google.md) | Datastream → GCS | 对象存储变更流需要消费端重建 |
| [阿里云](alibaba.md) | DataWorks MySQL Reader | 用户自定义 SQL 与结构化限制可能冲突 |
| [华为云](huawei.md) | CDM | 来源覆盖与当前场景接近，深度需逐连接器核实 |
| [Databricks](databricks.md) | CDC 与 query-based | 持续采集与低频消费可解耦，也可直接定时查询 |

可借鉴架构不等于推荐采购。每篇把产品事实与本平台设计推论分开，未给缺乏依据的产品总分。

## 本轮深入阅读入口

原有六家厂商文档保留为入门。本轮新增[连接器内部机制](../deep-dive/02-connectors.md)、[Databricks / Snowflake / Fabric / AWS 湖仓接入](../deep-dive/03-lakehouse.md)、[华为与阿里逐链路分析](../deep-dive/04-china-cloud.md)，并提供[跨厂商责任比较](../deep-dive/11-comparison.md)。各章包含具体例子及原始文档链接；完整顺序见[深度教程目录](../deep-dive/README.md)。
