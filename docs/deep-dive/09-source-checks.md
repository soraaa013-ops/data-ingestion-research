# 09｜你的五类来源，分别需要核对什么？

这一章给出接入前的“读数语义合同”。它不是某产品全版本兼容矩阵：用户尚未提供实际版本，不能用一个勾表示全面支持。

## MySQL：同一条 SELECT 可以有完全不同的代价

要取已关闭的售后工单，先看过滤条件、索引、行宽、长事务和并发；结果小不证明扫描小。批量分片必须处理 NULL、边界和一致性，不只是开启多线程。

增量有三条不同路：增长 ID 适合相应追加语义；更新时间需要可靠更新及边界处理；binlog 需要源配置、权限与保留。参考 [Confluent 模式](https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/overview.html)、[Debezium 快照](https://debezium.io/documentation/reference/stable/connectors/mysql.html)。

**反例**：用户 JOIN 工单和标签表，却只按工单 updated_at 增量。标签变了但工单时间不变，就会漏掉训练标签更新。这类查询必须明确增量依赖，或在复制到中间状态后重新构建。

## MongoDB：先问文档形状，再问变化接口

同集合的文档不一定具有全部相同字段。CDM 用首行生成字段样例的行为提示，要确认后续文档的稀疏字段、数组和嵌套对象如何保留。[CDM 原文](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0066.html)。

需要持续更新时，原生 Change Streams 的 token 依赖 oplog 历史仍可用；前轮的[官方来源](https://www.mongodb.com/docs/manual/changestreams/)已记录。不能把 Mongo 协议兼容产品与原生 Mongo 的变化捕获能力视为完全相同。

接入确认：拓扑、版本、查询/聚合白名单、投影、全文档或变化片段、token 保存点、失效后的回填与删除。未验证的功能不进入产品承诺。

## Elasticsearch：它可能是派生副本，不一定是业务真相

先问问答是否完整保存在 ES，是否经过索引裁剪或转换。如果实际业务库或业务导出更可靠，直接从 ES 抽取并不一定最合适；这是待场景确认的问题，不假设 ES 一定不完整。

原生 ES 官方在需要保持索引状态并遍历大量结果的场景描述 PIT/search_after；CDM 适配器则暴露 scroll 与 q 语法。两者并存不矛盾。[原生分页](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results)、[CDM 适配器](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0081.html)。

接入确认：查询语言、排序稳定性、游标过期、失败后是否仍同一截面、删除获知方式以及索引切换。不能只检查能返回第一页数据。

## Hive：表入口背后可能是文件读取，也可能是执行查询

CDM 的两种读取模式具体展示了这个区别：HDFS 直接读文件与 JDBC 执行筛选具有不同能力。[官方说明](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0038.html)。

**例子**：昨天分区已完全关闭且不可变，复制该分区可能很简单；如果分区会回补，或表是事务表，盲目复制最新文件可能不是正确表状态。

接入确认：分区是否完成、是否会重写、格式、事务语义、元数据位置、权限执行点。已经入湖不代表可以绕过现有数据权限直接读文件。

## DWS：分析库也需要容量和完整性控制

DWS 是分析来源时可以优先复用，但共享数仓仍有其他负载。CDM 的分片字段、NULL 处理已有具体参数。[官方说明](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0387.html)。

**例子**：为提高速度把“字段无 NULL”设为真，但实际有空值，可能丢掉一部分数据。性能优化必须与计数及内容校验共同验收。

需要核对查询队列、资源隔离、长查询取消、并行导出限制、分布与数据倾斜。当前来源文档不能证明工作负载规则自动建立，不编造具体 DWS 配额。

## 接入结果必须像这样表达

“此版本 MySQL 经验证支持指定模板的批量导出、源级并发上限、超时取消和幂等发布；尚未支持物理删除增量。”

这比“支持 MySQL：是”更有用。UI 只显示经测试的策略，让用户知道这次拿到的是什么保证。
