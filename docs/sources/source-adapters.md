# 按来源拆解：统一 DSL 无法代替来源语义

任务 R06；Mongo/ES 有官方资料，MySQL 部分来自厂商连接器；Hive/DWS 当前为设计检查项。

| 来源 | 批量设计 | 增量/恢复 | 不能直接套用的假设 |
|---|---|---|---|
| MySQL | 受限 SQL、索引、主键分片、事务范围 | 水位或 binlog；须确认源版本与配置 | 自增 ID 不覆盖旧行更新；副本不一定支持目标 CDC |
| MongoDB | filter/projection/aggregation 各自限制 | Change Streams resume token，关注 oplog 窗口 | SQL SELECT 校验无效；恢复点不永久有效 |
| Elasticsearch | 有稳定排序的分页，评估 PIT | 查询导出并非通用 CDC，删除需另行设计 | 深 offset 分页与 search_after 不等价 |
| Hive | 分区选择、查询队列、底层格式 | 分区重写、迟到和事务表语义待核 | 新分区不等于全部变化 |
| DWS | 查询资源隔离与范围限定 | SQL 批量/增量能力待版本验证 | 数仓也不是无限免费扫描来源 |

## MongoDB 官方事实与推论

[Change Streams 文档](https://www.mongodb.com/docs/manual/changestreams/)要求恢复 token 或时间戳对应历史仍在 oplog 中。因此要以最长中断和追赶时间评估恢复窗口，保存 token 必须与目标持久化协调。具体拓扑、版本、pre/post images 和读取偏好应单独确认，不能将兼容 Mongo 协议的产品都当成原生 MongoDB。

## Elasticsearch 官方事实与推论

[分页文档](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results)说明需要保持索引状态并遍历大量结果时可用 search_after 配合 PIT。平台需要维护分页游标与 PIT 生命周期；恢复后若重新开 PIT，不能自动声称仍是同一截面。PIT 成本与并发需验证，不给固定安全阈值。

## 来源能力合同

每个适配器声明：查询类型、过滤/投影、分页、并行、一致性级别、增量、删除、取消、检查点、schema 演进。未知能力标为 UNKNOWN，不能被统一 UI 显示为支持。

后续按五种来源分别建立版本矩阵和测试用例。当前这份横向表不等于五种连接器实现都已完成选型。
