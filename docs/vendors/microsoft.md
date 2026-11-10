# Microsoft：查询抽取与增量状态管理

关联任务 R02/R04；证据 E03–E05；状态：资料分析。ADF 与 Fabric 为不同产品，以下能力不相互默认继承。

## 官方机制

[ADF SQL Server connector](https://learn.microsoft.com/en-us/azure/data-factory/connector-sql-server)提供分区并行复制，parallelCopies 控制查询并发。这证明查询抽取是产品正式机制，不证明生产库可以无限并行。

[ADF 增量说明](https://learn.microsoft.com/en-us/azure/data-factory/tutorial-incremental-copy-overview)区分水位区间抽取与 Change Tracking。水位可选更新时间或增长键，处理旧水位与新水位之间的数据。

[Fabric 增量说明](https://learn.microsoft.com/en-us/fabric/data-factory/incremental-copy-job)明确水位不能识别物理删除；CDC 可捕获插入、更新、删除。重置增量状态不会自动清理目标，追加模式存在重复风险。

## 路径与失败语义

源数据库 → 计划好的分区/水位查询 → 文件或表目标。控制面保存任务状态与水位，执行面搬运结果。设计上应在批次持久化成功后推进检查点；写入失败后重试需识别重复批次。

只用自增 ID 只能捕获新行，不能捕获旧行修改。更新时间需要每次更新可靠变化；同时间戳及迟提交事务需要额外策略。对纯追加数据可以简单，对会修订/删除的会话则不能这样假设。

## 本场景匹配

与现有 Job + DSL 模式接近，可借鉴“范围化查询 + 状态表 + 幂等目标”的组织方式。可保留用户表达能力，但把源侧并发预算独立于用户任务数量。

对训练消费，目标应有 dataset 发布步骤，而非直接消费正在复制的目录。重置和回补必须形成可辨认的新版本。

## 缺口

精确来源/目标组合、私网执行运行时、托管身份与凭据轮换、取消传播、行列权限继承、费用未完成验证。Fabric 对某来源的 CDC 支持不能直接当作 ADF 的同等能力。
