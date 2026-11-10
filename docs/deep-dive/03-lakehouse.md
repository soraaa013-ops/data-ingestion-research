# 03｜数据湖和数据中台如何接收生产数据？

现在把例子扩展：售后要训练，财务要报表，分析团队要做流失预测。三个团队反复连接同一业务库，开始希望有一份共享、可查询的数据副本。湖仓在这里有价值，但必须说明第一批数据是谁搬进去的。

## Databricks：先分清四条入口

沿着工单走，常见路径可以这样理解：

```text
业务数据库 ── Lakeflow CDC 或定时查询 ──> 湖仓表
业务导出文件 ── 对象存储 ── Auto Loader ──> 湖仓表
业务事件 ── 消息系统 / 流读取 ──> 湖仓表
外部数据库 <── 联邦查询发回去执行 ── Databricks 查询入口
```

前两条是复制路径；最后一条不应被误认成业务库已卸载。

Lakeflow 的 CDC 路径中，gateway 捕获、staging 缓冲、pipeline 应用更新；查询路径则按 cursor 定时读源。它们回答“从数据库怎么取得数据”。[CDC 原文](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/cdc-overview)、[查询原文](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/query-based-overview)

Auto Loader 回答的是“文件已经到对象存储后，哪些还没有处理”。官方要求妥善保存检查点，文件过期配置过激会影响完整性。因此只部署 Auto Loader，不能解决源库怎样导出。[原文](https://docs.databricks.com/aws/en/ingestion/cloud-object-storage/auto-loader/production)，[N11](../../research/rounds/2026-11-10-deep-dive/sources/N11.md)

查询联邦将操作下推外部数据库；目录联邦则可能直接读取外部对象存储中的表。这两个名称接近，但生产影响不同。[原文](https://docs.databricks.com/aws/en/query-federation)，[N12](../../research/rounds/2026-11-10-deep-dive/sources/N12.md)

**例子**：售后在 Databricks 界面发一条很重的工单查询，如果它访问的是 foreign database，重工作仍可能回到业务库。换了入口不代表换了算力承担者。

## Bronze / Silver / Gold 到底解决什么？

Databricks 的分层建议把原始、清理和业务可用数据分开，也明确分层是推荐模式而非硬要求。[原文](https://docs.databricks.com/aws/en/lakehouse/medallion)，[N13](../../research/rounds/2026-11-10-deep-dive/sources/N13.md)

把名称换成业务例子更容易理解：

- 原始层保存获准保留的工单输入，便于重处理。
- 清理层把重复、删除和类型处理好，得到当前可信工单。
- 面向业务的产物可以是销售指标，也可以是问答训练样本。

不是所有 ML 都必须消费聚合后的 gold；问答训练往往需要逐条明细。分层也不意味着必须永久保存敏感原始数据。对一个部门，可以用已有 OBS 上的隔离前缀与版本实现部分职责。

## Snowflake：除了搬运，还维护变化历史和当前表

Openflow MySQL 的文档按结构发现、快照、增量三个阶段描述复制，保留 journal 并应用到当前表。[官方机制与限制](https://docs.snowflake.com/en/user-guide/data-integration/openflow/connectors/mysql/about)，[N06](../../research/rounds/2026-11-10-deep-dive/sources/N06.md)

```text
MySQL 工单
  → Openflow 捕获与快照
  → Snowflake journal / 当前表
  → 获准的业务查询
  → ML 样本或导出
```

它帮助你免去一部分状态应用工作，但仍要核对复制键、数据类型、TRUNCATE/级联删除、结构变化等限制。

**例子**：业务运维用 TRUNCATE 清空临时工单表，若该连接器不传播此操作，下游表可能仍保留旧行。不能因为产品名含 CDC 就把所有删除场景标为通过。

训练查询还要按实际软删除字段等约定选当前有效行。目标维护了历史，不等于历史都可以训练。

## Microsoft Fabric：用明确的文件协议接纳外部生产者

Open Mirroring 的设计很值得你的平台参考：采集器可以由外部提供，只要按 landing zone 协议提交，Fabric 负责应用到 Delta 表并提供查询入口。[概览](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring)

协议区分插入、更新、删除等操作，有键与顺序要求。它不是“把任意 JSON 扔进桶”。[格式](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-landing-zone-format)

**例子**：售后服务自己生成“101 的答案改为 B”的完整更新记录，发布到约定区域；平台按约定应用。这样生产者拥有提取业务数据的职责，平台拥有验证和合并职责。

官方建议临时写入完成后用 ADLS 原子 rename 发布，并将已发布路径视为不可变。[可靠发布](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices)

我们可以借鉴“不可变批次 + 明确发布动作”，但 OBS/S3 不能未经验证照搬 ADLS rename。当前平台可以在元数据服务中提交 manifest 引用，作为发布边界。

## AWS：zero-ETL 把复制托管，不把复制消除

RDS zero-ETL 文档明确数据从 RDS 复制到支持的分析目标，有同区域和引擎等限制。[官方说明](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/zero-etl.html)，[N10](../../research/rounds/2026-11-10-deep-dive/sources/N10.md)

**例子**：如果企业已有对应 AWS 源和分析目标，可以让托管集成维护数据副本；若是自建 Mongo 到 OBS，不能因为“zero-ETL”这个名字就认为它是通用连接协议。

托管解决的是维护负担的一部分。网络、源侧初始化、目标存储、查询算力、用途授权和训练样本仍然存在。

## 对本平台的结论

湖仓适合多个消费者共享持续更新的数据、重复分析和统一治理。它的第一跳仍是查询、CDC、文件或业务接口。你可以复用企业已有湖仓；如果没有，也可以只建设领域级交付与 dataset 服务。

选型不应问“是否必须入湖”，而应问：哪一层值得共享？如果只有一个领域一周取一次数据，复杂的共享副本可能不划算；若十个消费者反复读取同一来源，共享捕获和整理就可能减少重复工作。
