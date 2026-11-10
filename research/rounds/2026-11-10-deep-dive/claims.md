# 关键判断怎样追溯到证据

下表不是产品认证清单。“事实”限于所引文档，“推论”是本研究结合场景作出的判断。来源卡记录查阅范围及限制，点击后可以继续到原始页面。

| 编号 | 关键判断 | 原始资料入口 | 分析位置 | 边界 |
|---|---|---|---|---|
| K01 | CDC 可以按周期读取日志，不必全链路持续运行 | [N01](sources/N01.md)、[N31](sources/N31.md) | [连接器](../../../docs/deep-dive/02-connectors.md) | 不代表所有连接器都可任意暂停 |
| K02 | 初次快照、位置接续、恢复是连接器核心责任 | [N03](sources/N03.md)、[N04](sources/N04.md) | [生命周期](../../../docs/deep-dive/07-lifecycle.md) | 不同快照模式不能共用一条锁表结论 |
| K03 | 查询增量需要可靠游标，不能自动还原所有变化 | [N05](sources/N05.md)、[N32](sources/N32.md) | [连接器](../../../docs/deep-dive/02-connectors.md) | 删除、关联变化的判断结合查询语义推导 |
| K04 | 湖仓入口负责的阶段不同 | [N06](sources/N06.md)、[N07](sources/N07.md)、[N10](sources/N10.md)、[N11](sources/N11.md) | [湖仓](../../../docs/deep-dive/03-lakehouse.md) | 不将某一个受管路径推广到全部源目标 |
| K05 | 文件交付需要可恢复协议，不能只上传一堆文件 | [N08](sources/N08.md)、[N09](sources/N09.md) | [生命周期](../../../docs/deep-dive/07-lifecycle.md) | ADLS 原子重命名不可直接套到 S3/OBS |
| K06 | 联邦查询可能仍使用源库计算 | [N12](sources/N12.md) | [湖仓](../../../docs/deep-dive/03-lakehouse.md) | query federation 与 catalog federation 分开 |
| K07 | 同一厂商不同来源具有不同过滤、分页和并发约束 | [N14](sources/N14.md)至 [N18](sources/N18.md) | [国内云](../../../docs/deep-dive/04-china-cloud.md) | 原文可疑权限项保留疑问，不当成配置建议 |
| K08 | 实时到 Kafka 与实时到 OSS 是不同完整链路 | [N19](sources/N19.md)、[N20](sources/N20.md) | [国内云](../../../docs/deep-dive/04-china-cloud.md) | 本项目提出的 Kafka 消费器不是已验证厂商功能 |
| K09 | 小表、小 dataset 不保证采集日志成本小 | [N21](sources/N21.md) | [成本](../../../docs/deep-dive/12-cost-validation.md) | 针对文档描述的同步机制，不泛化所有 CDC |
| K10 | 内网执行器解决可达性，不自动隔离负载 | [N22](sources/N22.md) | [网络](../../../docs/deep-dive/06-network-api-files.md) | 后半句是责任边界推论 |
| K11 | 查询授权不能自动保护所有复制后的文件路径 | [N23](sources/N23.md)、[N24](sources/N24.md) | [治理](../../../docs/deep-dive/08-governance.md) | 并非对某企业法律合规的认定 |
| K12 | 批量 API 和业务 outbox 是数据库直读之外的交付方式 | [N25](sources/N25.md)、[N26](sources/N26.md) | [网络与业务交付](../../../docs/deep-dive/06-network-api-files.md) | outbox 不自动提供历史回填；N26 为 nightly 文档 |
| K13 | 企业共享采集来自重复负载和维护问题 | [N28](sources/N28.md)、[N29](sources/N29.md)、[N30](sources/N30.md) | [企业案例](../../../docs/deep-dive/05-enterprise-cases.md) | 历史案例，不代表当前全公司部署；N30 仅检索正文摘录 |
| K14 | 分块全量与并发日志可以通过水印窗口协调 | [N27](sources/N27.md)、[N33](sources/N33.md) | [企业案例](../../../docs/deep-dive/05-enterprise-cases.md) | 论文有主键、读取及写水印前提，不等同 updated_at |
| K15 | 你的平台可以按条件采用三条路线 | K01–K14 综合 | [平台设计](../../../docs/deep-dive/10-your-platform.md) | 本项目建议，不声称任何厂商背书或实测胜出 |

## 如何处理证据不足

来源登记表和卡片优先保留实际阅读状态。正文定位失败、仅查到摘要、前轮已读、版本未知分别记录。无法确认的接口字段、价格与吞吐没有转写成确定事实。完整兼容矩阵和实际性能留在任务表中，不以“业界通常如此”填空。

后续修改结论时，应同时更新对应分析和这张映射表，并在修正记录说明新证据为什么改变判断。不要删除原始阅读卡来掩盖认识变化。
