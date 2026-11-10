# 检索、阅读及中间判断

日期：2026-11-10。以下按实际研究顺序整理；不是逐秒工具日志。搜索结果只是线索，正式结论优先采用打开的官方正文。没有访问企业内部系统。

## 1. 先检验“厂商都做持续 CDC”这个假设

检索词：Debezium MySQL initial snapshot incremental snapshot binlog；Airbyte CDC；Fivetran MySQL initial sync；Confluent JDBC source timestamp。

选取 N01–N05。发现 Airbyte 明确描述定时读取日志，与前版容易造成的“CDC＝常驻服务”印象不同。还发现真正影响可靠性的细节是快照与日志的衔接、主键、状态确认，不只是有没有 connector。

据此决定新增[连接器如何工作](../../../docs/deep-dive/02-connectors.md)，用一个批次失败的例子解释状态归属。

## 2. 扩展到湖仓，追问湖仓之前是谁搬数据

检索词：Snowflake Openflow MySQL；Fabric Open Mirroring；AWS zero ETL lakehouse；Databricks Auto Loader / federation / medallion。

选取 N06–N13。区分了原始数据库捕获、文件接收、状态合并、统一查询四件事。Fabric 的 landing zone 协议给出了可借鉴的“生产者与平台握手”边界；AWS 文档明确 zero-ETL 仍执行复制；Databricks 查询联邦会使用远端算力。

据此新增[湖仓完整链路](../../../docs/deep-dive/03-lakehouse.md)，避免把“建湖”当成源系统接入问题的自动答案。

## 3. 回到当前来源，逐页点开华为 CDM 配置

从源目录依次进入 MySQL、Mongo、ES、Hive、DWS，选取 N14–N18，再找 DRS 的 MySQL→Kafka 链路 N19。

发现 CDM 的 ES 只支持 q 过滤而不是任意 JSON DSL；Hive 文件与 JDBC 模式能力不同；Mongo 字段样例取首条；DWS 的 NULL 分片配置影响完整性。这些是比“支持某来源”更有用的对照。

保留疑点：N14 页面在 MySQL 的分区权限处出现 Oracle 风格视图名，不将其作为部署授权指令。N19 搜索结果包含续传去重字段，但后续正文 find 没有稳定定位，不将该字段写进推荐协议。

## 4. 查国内入湖和企业真实案例

DataWorks MySQL→OSS、DTS synchronization scenarios 得到 N20/N21。后者说明只同步小表仍可能读取全库日志，修正“领域小所以 CDC 必然轻量”的假设。

Netflix DBLog 检索未稳定命中原技术博客，改采用原作者论文摘要 N27，明确未读全文。Uber N28 和 LinkedIn N29/N30 采用第一方工程描述，保留历史日期，不宣称为其当前架构。

## 5. 补齐企业连接与数据交付边界

选取 ADF 私网执行器 N22、Lake Formation N23、Unity Catalog N24、Salesforce Bulk API N25、Outbox N26。用它们回答网络上如何连、业务系统主动交付怎样做、授权在哪里执行。

N26 是 nightly 文档，只借鉴已描述的模式，不提供稳定版部署配置。N20 首次正文读取成功，后续章节查找一次超时，保留该读取限制。

## 本轮没有采用的材料

未将 Reddit、聚合站、厂商营销“零影响”措辞作为生产影响的证明；未将搜索结果里的敏感查询参数复制到引用链接；未凭文档支持列表推断源目标任意组合；未以定价宣传推断总体拥有成本。

## 中间结论如何保留

来源卡先落盘，再撰写比较、案例与场景建议。结论修订记录在[corrections](corrections.md)。没有执行生产测试，也没有把计划中的测试标记为已完成。
