# data-ingestion-research

生产系统数据如何合理交付给 ML 消费：持续调研项目。

研究对象是领域内按需构建训练/微调数据集，现有实现为 DSL → Kubernetes Job → OBS/S3。关注交付效率、生产影响、数据正确性、ML 可用性和授权；不预设必须入湖，不安排内部访谈。

## 阅读入口

- [研究命题与证据方法](research/charter.md)：区分已知事实、假设和结论。
- [任务与进度](research/tasks.md)：每项任务的产物、完成条件和缺口。
- [证据索引](research/evidence.md)：官方来源与结论对应关系。
- [厂商专题](docs/vendors/index.md)：AWS、Microsoft、Google、阿里云、华为云、Databricks。
- [交付模式比较](docs/patterns/delivery.md)、[正确性与恢复](docs/patterns/correctness.md)、[授权与数据生命周期](docs/patterns/governance.md)。
- [来源专项](docs/sources/source-adapters.md)：MySQL、MongoDB、ES、Hive/DWS。
- [当前场景匹配](docs/scenario/fit.md)、[参考架构](docs/scenario/architecture.md)、[验证计划](docs/scenario/validation.md)。
- [汇总报告](reports/summary.md)：面向架构评审的阶段性综合判断。

## 当前状态

第二版：已形成分专题的公开资料分析和场景方案。不是完成选型或已验证的生产设计。厂商版本覆盖、价格、治理产品深挖和负载实测尚有明确待办，见任务表。

所有专题使用“官方事实 → 工程解释 → 场景匹配 → 待验证项”的结构。文档之间通过链接连接，汇总不替代原始分析。原[第一版报告](docs/industry-report.md)仅作历史记录，后续以专题及汇总为准。

[MIT License](LICENSE)
