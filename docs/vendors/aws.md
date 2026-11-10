# AWS：数据库复制、快照交付与 ML 消费分层

关联任务 R02/R04；证据 E01/E02/E16；状态：资料分析。

## 官方机制

[DMS S3 target](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.S3.html)将全量和 CDC 写为文件；PreserveTransactions 可保留 CDC 事务顺序，但该模式使用 CSV，且与部分其他参数不兼容。因此文件格式、分区和顺序要求不能独立随意组合。

[RDS 快照导出](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ExportSnapshot.html)是另一种数据库快照到 S3 的路径；不能把其支持范围推广到任意自建 MySQL 或 OBS。

[DMS 最佳实践](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_BestPractices.html)明确 CDC 延迟受源负载、网络、目标吞吐等影响，不提供 CDC 延迟 SLA。

## 数据路径与工程解释

路径 A：数据库 → DMS 全量/日志捕获 → S3 原始文件 → 状态重建 → ML dataset。路径 B：已具备可导出快照 → S3 文件 → 筛选/样本构建。路径 A 适合持续更新；路径 B 适合定期获取固定截面。

增删改操作文件不是当前表的直接替代。消费端需按操作语义处理删除与重放，再发布训练版本。跨表一致性不能只靠文件到达时间推导。

## 本场景匹配

借鉴价值高：将“搬运”和“样本构建”分层，允许不同导入策略共用 dataset 服务。直接采购匹配未知：现有对象存储是 OBS，网络、出口费用、跨云传输及目标兼容必须核验。

一次性训练数据不应仅因产品可用就建立永久 CDC。成本模型要计入复制运行时、日志保留、文件请求、状态合并与运维，而不仅是传输字节。

## 授权、恢复与未决项

建议源身份与对象存储写身份分开；连接权限不代替训练用途授权。官方来源本身不能证明此用途已合规。

待核：具体 MySQL/Mongo 来源版本；S3 文件验证限制；schema 变化及断点恢复边界；快照生成影响与导出区域；定价。本篇未声称这些已经验证。
