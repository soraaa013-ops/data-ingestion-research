# 深挖轮次：IT 数据怎样真正搬到 ML 平台和湖仓

本轮保留 32 张来源阅读卡。目标是解释端到端搬运，而不是罗列连接器名称。

## 证据如何流向结论

原始网页/论文 → [来源登记表](sources.json)与下面的阅读卡 → [检索与阅读过程](journal.md) → [认识修正](corrections.md) → [深度研究教程](../../../docs/deep-dive/README.md) → [深度汇总](../../../reports/deep-research-report.md)。

每张卡保留 URL、定位章节、查阅日、版本线索、短摘录或事实释义及适用边界。它们是独立的中间结果，不会因为汇总改写而消失。

## 原始信息保全的边界

这里保全的是来源身份、有限原文摘录、阅读笔记和推导关系，**不是完整网页镜像**。第三方内容不在项目 MIT 授权范围内；不把有版权网页全文重新发布到公开仓库。引用页可变，来源摘要的 Git 历史只能证明本项目何时记录了什么，不能证明原网页永远未变。

论文 N27 使用固定的 arXiv v1 线索；动态产品页记录可见的更新时间，没有看到更新时间就标明未知。全文未读取、检索摘录、正文定位失败均单独标明，不用数量掩盖证据质量。

## 阅读卡目录

| ID | 来源 | 分组 |
|---|---|---|
| [N01](sources/N01.md) | Airbyte — Change Data Capture | connectors |
| [N02](sources/N02.md) | Airbyte Protocol | connectors |
| [N03](sources/N03.md) | Debezium connector for MySQL | connectors |
| [N04](sources/N04.md) | Fivetran MySQL | connectors |
| [N05](sources/N05.md) | Confluent JDBC Source Connector | connectors |
| [N06](sources/N06.md) | Snowflake Openflow Connector for MySQL | lakehouse |
| [N07](sources/N07.md) | Microsoft Fabric Open Mirroring | lakehouse |
| [N08](sources/N08.md) | Open Mirroring Landing Zone Requirements | lakehouse |
| [N09](sources/N09.md) | Open Mirroring Best Practices | lakehouse |
| [N10](sources/N10.md) | Amazon RDS zero-ETL integrations | lakehouse |
| [N11](sources/N11.md) | Databricks Auto Loader production | lakehouse |
| [N12](sources/N12.md) | Databricks external databases and catalogs | lakehouse |
| [N13](sources/N13.md) | Databricks medallion architecture | lakehouse |
| [N14](sources/N14.md) | CDM MySQL source parameters | huawei |
| [N15](sources/N15.md) | CDM MongoDB/DDS source parameters | huawei |
| [N16](sources/N16.md) | CDM Elasticsearch source parameters | huawei |
| [N17](sources/N17.md) | CDM Hive source parameters | huawei |
| [N18](sources/N18.md) | CDM DWS source parameters | huawei |
| [N19](sources/N19.md) | DRS MySQL to Kafka | huawei |
| [N20](sources/N20.md) | DataWorks MySQL to OSS | china-cloud |
| [N21](sources/N21.md) | DTS data synchronization scenarios | china-cloud |
| [N22](sources/N22.md) | ADF self-hosted integration runtime | operations |
| [N23](sources/N23.md) | Lake Formation data filtering limitations | governance |
| [N24](sources/N24.md) | Unity Catalog external locations | governance |
| [N25](sources/N25.md) | Salesforce Bulk API 2.0 Query | operations |
| [N26](sources/N26.md) | Debezium Outbox Event Router | operations |
| [N27](sources/N27.md) | DBLog: A Watermark Based Change-Data-Capture Framework | cases |
| [N28](sources/N28.md) | Uber DBEvents ingestion framework | cases |
| [N29](sources/N29.md) | LinkedIn Gobblin | cases |
| [N30](sources/N30.md) | LinkedIn Data Integration | cases |
| [N31](sources/N31.md) | Databricks managed database connectors | lakehouse |
| [N32](sources/N32.md) | Databricks query-based connectors | lakehouse |

早期来源 E01–E16 保留在[旧证据索引](../../evidence.md)。N31/N32 明确沿用前轮已经读过的 Databricks 文档，不伪装成本轮新增阅读。
