# Databricks：持续 CDC 与定时查询是两种运行模型

关联任务 R02/R03；证据 E11/E12；状态：资料分析。

## 官方机制

[CDC 连接器](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/cdc-overview)由 connection、常驻 gateway、staging、ingestion pipeline 和 Delta 目标组成。gateway 持续采集以避免源日志截断；下游应用可另行调度，所以消费不运行时仍有采集计算成本。

[查询连接器](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/query-based-overview)通过单调 cursor 定时查询，无需 gateway 或 staging volume。它读取两次运行间的最新变化状态，不捕获所有中间状态。官方同时提醒查询源表的负载代价。删除能力有配置及发布阶段条件，不能概括为所有查询模式都支持。

## 对低频微调的启示

定时训练并不意味着 CDC 捕获也能随意停止：停机超过源日志保留可能失去恢复点。相反，若只需周期性最新状态，批量查询的运维形态可能更合适。这是条件化推论，不是对吞吐的测试结论。

建议平台在任务定义里分别表达“采集连续性”和“dataset 发布频率”，不要只用一个 cron 同时表示两者。

## 本场景匹配

治理 connection、执行与目标分离有借鉴价值。完整产品以 Delta/Unity Catalog 为中心，与当前 OBS 文件数据集并非直接等价；不为获得 connector 自动引入全部湖仓基础设施。

## 缺口与成本

需要核对具体连接器版本、网络、计算规格、服务可用区、schema 演进和恢复保障。成本必须分别计算常驻 gateway 与按需样本构建。权限对象能保护连接，并不自动批准最终训练用途。
