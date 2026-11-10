# 04｜华为与阿里：从多来源批搬运到持续入湖

这一章重点贴近你的来源组合。不能只查“支持 MySQL/Mongo/ES”，还要问用户现在的查询能否表达、怎么分片、结果写到哪里。

## 华为 CDM：更接近你现有的批量抽取框架

假设用户希望“取昨天已关闭工单”。CDM 的 MySQL 源支持自定义查询，也支持结构化表、分区字段和 WHERE 配置；自定义 SQL 有语法限制。[MySQL 文档](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0358.html)

这与用户提交查询、执行器读取、写入目的端的模式相近。相近不代表功能完全一致：如果用户依赖 WITH 或其他超出连接器支持范围的语法，直接替换会改变行为。

更重要的是，同一产品对不同来源采取不同方式：

| 来源 | 本轮核实的连接器行为 | 对例子的影响 |
|---|---|---|
| MongoDB/DDS | 首行用于字段样例，缺失字段需补充 | 第二条工单才有附件字段时，不能靠第一条推断所有结构 |
| Elasticsearch | q 过滤和 scroll 参数，非任意 JSON DSL | 现有 DSL 用复杂 bool 查询时要检查能否迁移 |
| Hive | HDFS 与 JDBC 路径，过滤和并发能力不同 | 只搬分区文件与执行行条件筛选是两种工作 |
| DWS | 分片字段及 NULL 配置会影响抽取 | 优化并发时不能漏掉分片键为空的记录 |

对应原文：[Mongo](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0066.html)、[ES](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0081.html)、[Hive](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0038.html)、[DWS](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0387.html)。

**一个具体取舍**：若工单已在 Hive，且按日期分区，可研究读取目标分区；若必须按敏感字段规则过滤，直接绕过查询引擎读 HDFS 可能无法落实同一授权。快一些不等于语义相同。

## 华为 DRS：与 CDM 不同的持续交付路径

官方另有 [MySQL → Kafka 的 DRS 链路](https://support.huaweicloud.com/realtimesyn-drs/drs_04_0128.html)。因此可以研究以下组合：

```text
MySQL → DRS → Kafka → 自有消费/整理 → OBS dataset
```

这里 DRS 到 Kafka 的能力有官方依据；Kafka 到 OBS 的消费器、事件语义与 dataset 发布是我们提出的集成环节，不能宣称 DRS 已替你全部完成。该页的某些续传字段在搜索与正文定位间未稳定核实，见 [N19](../../research/rounds/2026-11-10-deep-dive/sources/N19.md)，本轮不提供未经验证的解析代码。

这条链适合需要持续变化且已有消息基础设施的场景。若只有少量按周导出，引入两个长期服务可能反而增加负担。

## 阿里 DataWorks：批查询与 MySQL → OSS 入湖都有正式路径

前轮研究了 MySQL Reader 的自定义查询。本轮进一步核实了 [MySQL 到 OSS 的全量加增量任务](https://www.alibabacloud.com/help/en/dataworks/user-guide/synchronize-all-data-in-a-mysql-database-to-a-data-lake-in-oss-in-real-time)。它要求 binlog 与网络连通，可同步选定表，并有结构变更限制；可关联同区域 DLF 元数据。

把它放回工单例子：

```text
先复制现有工单 → 后续变化继续同步 → OSS 中的湖数据
    → 元数据服务描述这些表 → 查询/处理 → ML 样本
```

“入湖”并没有跳过初次读取、持续日志、运行资源和网络。这些被产品组合起来了。DLF 的目录能力也不自动完成业务问答筛选和训练授权。

## 阿里 DTS：一个很重要的源端成本反例

[DTS 的同步场景文档](https://www.alibabacloud.com/help/en/dts/user-guide/data-synchronization-scenarios)区分结构、全量和增量，并说明读取日志时可能处理整个数据库的日志，即使只同步小表。

**例子**：你只要一个小售后表，但同实例中另一个业务大量更新。输出到 ML 的文件很小，日志捕获仍可能繁忙。因此评估输入压力不能只看训练数据集大小。这一条直接影响“每个领域只取自己的数据，所以影响必然小”的判断。

## 该如何应用这些研究？

如果目标是尽快完善现有平台，CDM/Reader 的受控批查询是最近的参照；如果要长期共享变更，DRS/DTS/入湖链路值得比较。先确定需要哪种交付，再验证实际源目标组合。

OBS、OSS、S3 名称相近但不是可自动互换的产品目标；版本、认证、网络、multipart、文件格式、限流和生命周期都要验证。

文档疑点也保留：[N14](../../research/rounds/2026-11-10-deep-dive/sources/N14.md)记录了 MySQL 页中的可疑视图权限说明。本研究不把它复制成部署指令。
