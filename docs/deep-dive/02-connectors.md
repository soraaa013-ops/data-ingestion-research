# 02｜连接器不是一个数据库驱动：四种产品怎样完成搬运

本章使用同一个虚构场景：已有工单需要初次导入，之后每天修订少量答案，某次任务写到一半断开。读到最后，应能回答“进度在哪里、失败后谁负责重新来”。

## Airbyte：让来源和目标通过协议配合

Airbyte 把读数据的 Source、写目标的 Destination 和平台调度分开，协议包括记录与状态。[协议原文](https://docs.airbyte.com/platform/understanding-airbyte/airbyte-protocol)，[阅读卡 N02](../../research/rounds/2026-11-10-deep-dive/sources/N02.md)。

这与现有脚本最大的结构差别，是读写与恢复不应靠每个脚本自己猜。比如来源已经读完 101–200，但目的地只写成功 101–150，平台不能把“读到 200”当成“可靠搬到 200”。这是我们设计连接器接口时需要遵守的责任边界；具体 state 确认细节要核验对应协议版本和 sink，不能只凭概念保证可靠。

Airbyte 的 CDC 文档特别说明：按同步周期读取一段日志，并不当成无限流持续消费。第一次仍需快照。[CDC 原文](https://docs.airbyte.com/platform/understanding-airbyte/cdc)，[N01](../../research/rounds/2026-11-10-deep-dive/sources/N01.md)。

**例子**：每小时消费一次日志可以符合某个来源的保留窗口；每周才运行一次但日志只保留一天则不行。这里的小时/天是教学假设，不是产品默认值。

对你的平台，Airbyte 更适合用于研究“多来源、多目标的统一搬运协议”。是否直接集成还要验证具体 OBS 目标、连接器维护状态、部署成本和许可证，不能因为有 S3 connector 就推定所有 S3 兼容存储都可用。

## Fivetran：把初次搬运和长期维护交给托管服务

Fivetran MySQL 的官方描述是先读取选定数据，后续使用 binlog 或其 Teleport 机制。初次读取大表时，主键会影响能否保存分段进度；无主键表有恢复限制。[官方 MySQL 页](https://fivetran.com/docs/connectors/databases/mysql)，[N04](../../research/rounds/2026-11-10-deep-dive/sources/N04.md)。

**例子**：工单有稳定 ticket_id，搬到一半断网时有条件从分段进度恢复；如果一张巨大的无主键日志表只能整表重新开始，“同样支持 MySQL”的实际运维成本会完全不同。

托管方案减少自建维护，但不是免除源侧准备、网络与目标数据建模。评估时要问清楚：删除在目标是删除还是标记？新增列是否自动放行？谁承担历史重同步？本轮没有验证所有 destination 行为，不把通用 MySQL 能力扩展到任意目标。

这里尤其不能把 Teleport 与普通 updated_at 查询混为一谈。它是独立产品机制；“有不靠 binlog 的同步方法”不证明我们自己的时间戳查询也具有同样完整性。

## Debezium：给你捕获能力，也把运行责任交给你

Debezium MySQL 文档详细说明快照、日志位置及恢复。指定快照流程会在建立边界时取得锁；读表阶段的锁行为取决于配置。日志已清理时恢复可能需要新快照。[官方文档](https://debezium.io/documentation/reference/stable/connectors/mysql.html)，[N03](../../research/rounds/2026-11-10-deep-dive/sources/N03.md)。

**例子**：101 在快照扫描前后发生修订。正确结果依赖快照与日志衔接，不能让旧快照覆盖新变化。成熟捕获器处理这类问题，比自己写一段定时 SELECT 更复杂。

自建时至少要落实：offset 与 schema history 的持久化、源日志保留、故障重启、版本升级、删除事件消费和目标写入。Debezium 捕获输出不是已经治理好的训练数据集。Kafka Connect 只是运行方式之一，不能据此宣称采用 Debezium 必须建设 Kafka；其他部署形态也要按其支持目标核对。

它适合已有流平台、需要定制事件分发的团队；只有每周少量导出的团队不应把这些运行责任当作免费。

## Confluent JDBC：写进 Kafka 的数据也可能来自轮询 SQL

Confluent JDBC Source 周期执行查询，可按增长 ID、时间戳或二者组合检测变化，输出至少一次到 Kafka。[官方文档](https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/overview.html)，[N05](../../research/rounds/2026-11-10-deep-dive/sources/N05.md)。

**例子**：101 和 102 的更新时间相同，只处理完 101 就崩溃。恢复逻辑若只保存时间且使用严格大于，可能跳过 102。组合游标改善排序，但迟提交事务仍需要额外假设与策略。topic 有消息不代表这些源侧遗漏已经解决。

对用户任意 JOIN 的查询还要再问：关联表改了，主表更新时间是否也变化？如果没有，按主表水位捕获不到答案变化。这是我们的工程推论，不是说厂商自动解决了任意查询增量。

## 应该怎样选，而不是给产品排总分？

- 只需批量多连接器：先比较搬运协议与目标支持，Airbyte/CDM/DataWorks 是参考。
- 希望长期同步由服务方维护：研究 Fivetran/托管云复制，但核对合同与目标语义。
- 已有事件基础设施且需要细控：研究 Debezium/Connect，并承担运行责任。
- 已有 JDBC 轮询足够：可以继续，但要明确更新、删除、关联变化和恢复边界。

“自建还是采购”可以按组件分开：采购捕获与传输，自己保留 dataset 授权、样本构建和版本发布。
