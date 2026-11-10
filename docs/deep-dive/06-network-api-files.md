# 06｜内网、API、文件和附件：数据库之外的第一跳

假设业务数据库只允许内网访问，ML 平台在另一个网络。最直接的 SQL 方案现在连不上。此时首先要解决连接和运行位置，而不是把“连不上”解释为必须购买湖仓。

## 私网执行器：让搬运工靠近数据

ADF self-hosted integration runtime 可以部署在私网内，由它连接数据源及目标，并与云端控制服务通信。官方分别列控制服务与数据存储的网络要求。[原文](https://learn.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime)，[N22](../../research/rounds/2026-11-10-deep-dive/sources/N22.md)。

对我们的平台可以借鉴：

```text
平台控制服务 ── 下发受限任务 ──> 源网络内的执行器
                                   ├─ 读取获准数据库
                                   └─ 写入指定对象存储前缀
```

这是设计示意，不要求部署 ADF。执行器要可达源与目标，控制网络和数据网络分别验证；不能笼统说只开一个 HTTPS 端口就满足所有来源。

它减少了平台直接暴露数据库入口的需求，但执行器发的扫描仍然由数据库承担。agent 是网络/身份选择，不是性能隔离的替代品。

## 业务批量 API：把数据交付变成源系统的正式能力

Salesforce Bulk API 2.0 Query 提供异步任务创建、查询状态、获取结果和取消。[官方接口](https://developer.salesforce.com/docs/platform/api-asynch/guide/queries.html)，[N25](../../research/rounds/2026-11-10-deep-dive/sources/N25.md)。

借鉴到工单系统，可设计一个由业务系统拥有的导出接口：

1. ML 平台申请“售后已关闭问答”，提交用途和范围。
2. 源系统验证并启动批量导出，返回 job_id。
3. 平台等待完成，读取文件清单。
4. 平台校验并构建 dataset。

这样业务系统可以控制语义、限流和交付时间。但它要承担导出实现与维护，也仍可能读取业务库。不能声称“换成 API 就没有数据库负载”。

逐条请求每张工单详情与批量异步导出不同。前者可能产生大量请求，还可能在分页期间读到不同时间状态。

## 文件推送：最容易低估、也常常足够的一条路径

如果系统每天本来就生成报表或脱敏导出，可以约定它向隔离前缀发布文件。ML 平台需要接收的是一个完整批次，而不是看到某个文件名就立即训练。

建议合同包含：batch_id、数据范围、schema 版本、文件列表/校验和、记录数、生成时间、一致性声明、授权标识和删除/更新语义。这个合同是本平台建议，可参考 [Fabric 的可靠文件发布](https://learn.microsoft.com/en-us/fabric/mirroring/open-mirroring-best-practices)，不直接复制其 ADLS 操作。

## Outbox：想表达“批准了一条可交付问答”，不只是“某列改了”

原始数据库 CDC 可能产生很多中间修订，业务真正希望交付的是“已审核完成”的事件。业务系统可在同一事务写业务状态与 outbox 记录，再由捕获器发送。Debezium 有对应路由模式。[原文](https://debezium.io/documentation/reference/transformations/outbox-event-router.html)，[N26](../../research/rounds/2026-11-10-deep-dive/sources/N26.md)。

这是需要源系统改造的选择，不应作为所有接入的硬前提。它也不自动解决既有历史回填和下游重复，事件仍需唯一标识和消费规则。本轮读的是 nightly 文档，不提供可直接部署的稳定版配置。

## 附件才是内容，数据库可能只有一个链接

工单的截图、PDF、录音也许在对象存储或文档系统中。只同步表里的 URL，不等于 ML 平台拿到了文件；临时签名链接可能在训练前过期。[LinkedIn 集成实践](https://engineering.linkedin.com/teams/data/analytics-platform-apps/big-data-engineering/data-integration)说明了媒体与期限链接这类额外问题。

建议按附件对象 ID、版本、hash 及授权来交付，明确是否要复制字节。OCR、音频转写、文档解析是内容准备，原始附件复制是 ingestion；两者可以串联但要有各自的进度和失败状态。公共研究材料不应记录真实签名 URL 或凭据。
