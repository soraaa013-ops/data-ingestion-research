# 厂商研究目录

| 专题 | 重点 | 与本场景最相关的观察 |
|---|---|---|
| [AWS](aws.md) | DMS / RDS snapshot | 原始变更文件与训练快照之间存在处理环节 |
| [Microsoft](microsoft.md) | ADF / Fabric | 批量查询、水位、CDC 是并存选项 |
| [Google](google.md) | Datastream → GCS | 对象存储变更流需要消费端重建 |
| [阿里云](alibaba.md) | DataWorks MySQL Reader | 用户自定义 SQL 与结构化限制可能冲突 |
| [华为云](huawei.md) | CDM | 来源覆盖与当前场景接近，深度需逐连接器核实 |
| [Databricks](databricks.md) | CDC 与 query-based | 持续采集与低频消费可解耦，也可直接定时查询 |

可借鉴架构不等于推荐采购。每篇把产品事实与本平台设计推论分开，未给缺乏依据的产品总分。
