# 阿里云：DataWorks MySQL Reader 的表达能力与边界

关联任务 R02/R06；证据 E08；状态：MySQL 来源专题，未覆盖所有 DataWorks 连接器。

## 官方事实

[MySQL 数据源文档](https://www.alibabacloud.com/help/en/dataworks/user-guide/mysql-data-source)同时覆盖离线 Reader 与实时同步。Reader 支持字段、过滤、拆分键和 querySql；querySql 优先于 table、column、where、splitPk。

同页列明实时同步依赖 ROW binlog，并存在版本及事务语义限制。针对 Aurora MySQL，文档要求连接 writer，而不能统一假设 CDC 可改读副本。

## 对当前 DSL 设计的直接启示

参数化范围与自由 SQL 如果同时存在，必须定义谁最终生效。平台若审批 column/where，却允许 DSL 覆盖这些参数，就会产生“审批范围与实际查询不同”的风险。这是本平台设计推论，不声称厂商存在此漏洞。

可选实现：受限 AST 编译器校验最终查询的所有表/字段；或只允许对获批视图/数据对象查询。源库最小权限作为独立防线。

## 本场景匹配

批 Reader 的模式与现有实现高度相似，说明用户查询不是需要整体推翻的能力。splitPk 类分片提高吞吐，但必须受源级预算约束。实时同步应作为独立策略，不让用户 DSL 承担日志捕获语义。

## 成本、治理与缺口

需要分别评估运行资源、同步次数、源侧扫描、出网与重试开销。DataWorks 自身的权限机制不自动给复制出的训练数据授予用途。

待核：具体区域的资源组/网络、目标 OBS 支持、实时删除与 DDL 的逐项限制、平台权限模型。不能把 MySQL 支持情况推广到 Mongo/ES。
