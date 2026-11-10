# 官方证据索引

查阅日期 2026-11-10。链接是可追踪的资料入口，不是网页归档；页面会更新。本文不复制厂商全文。

| ID | 来源 | 支撑事实 | 使用位置 |
|---|---|---|---|
| E01 | [DMS S3 target](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.S3.html) | 全量/CDC 文件、事务顺序参数限制 | AWS、正确性 |
| E02 | [RDS export](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ExportSnapshot.html) | 快照导出到 S3 的独立路径 | AWS |
| E03 | [ADF SQL Server](https://learn.microsoft.com/en-us/azure/data-factory/connector-sql-server) | 分区查询及并发 | Microsoft |
| E04 | [ADF incremental](https://learn.microsoft.com/en-us/azure/data-factory/tutorial-incremental-copy-overview) | 水位与 Change Tracking | Microsoft |
| E05 | [Fabric incremental](https://learn.microsoft.com/en-us/fabric/data-factory/incremental-copy-job) | 水位不能捕获物理删除、CDC 差别 | Microsoft、正确性 |
| E06 | [Datastream events](https://docs.cloud.google.com/datastream/docs/events-and-streams) | 至少一次、不保证事件有序 | Google |
| E07 | [Datastream GCS](https://docs.cloud.google.com/datastream/docs/destination-gcs) | 文件组织与轮转、回填重叠 | Google |
| E08 | [DataWorks MySQL](https://www.alibabacloud.com/help/en/dataworks/user-guide/mysql-data-source) | querySql 优先级、ROW binlog、源限制 | 阿里云 |
| E09 | [CDM sources](https://support.huaweicloud.com/usermanual-cdm/cdm_01_0034.html) | 多来源的专用配置入口 | 华为 |
| E10 | [CDM jobs](https://support.huaweicloud.com/intl/zh-cn/usermanual-cdm/cdm_01_0153.html) | 抽取并发和分片限制 | 华为 |
| E11 | [Lakeflow CDC](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/cdc-overview) | 常驻 gateway、staging、pipeline | Databricks |
| E12 | [Lakeflow query](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/query-based-overview) | 定时水位查询，无需 gateway | Databricks |
| E13 | [Mongo change streams](https://www.mongodb.com/docs/manual/changestreams/) | resume token 与 oplog 历史前提 | 来源适配 |
| E14 | [ES pagination](https://www.elastic.co/docs/reference/elasticsearch/rest-apis/paginate-search-results) | search_after + PIT | 来源适配 |
| E15 | [Datastream FAQ](https://docs.cloud.google.com/datastream/docs/faq) | 回填负载与日志捕获 | 模式 |
| E16 | [DMS best practices](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html) | CDC 延迟边界 | AWS |

## 不从这些资料推导的结论

连接器存在不代表生产环境允许使用；没有查到某功能不代表厂商不支持；AWS 的 S3 路径不能直接推导为支持任意 OBS；产品具备权限控制不代表已经批准训练用途。
