---
title: 使用 Azure 和 Azure Stack Hub 的分层数据分析模式
description: 了解如何使用 Azure 和 Azure Stack Hub 在整个混合云中实现分层数据解决方案。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b671fa9f47fa51ab6e40633c04964957d613fec2
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909963"
---
# <a name="tiered-data-for-analytics-pattern"></a>分层数据分析模式

本模式演示如何使用 Azure Stack Hub 和 Azure 跨多个本地与云位置暂存、分析、处理及存储数据。

## <a name="context-and-problem"></a>上下文和问题

新式技术领域中的企业组织所面临的一个问题是如何实现安全的数据存储、处理和分析。 考虑因素包括：

- 数据内容
- location
- 安全和隐私要求
- 访问权限
- 维护
- 存储仓库

将 Azure 与 Azure Stack Hub 相结合可以解决数据方面的忧虑，并提供低成本的解决方案。 此解决方案最适合通过分布式制造或物流公司来表达。

此解决方案基于以下方案：

- 大型的多分支制造组织。
- 需要在全球远地位置与中央总部之间快速安全地存储、处理和分发数据。
- 必须保持员工与机器活动、设施信息和企业报告数据的安全。 必须适当分发数据，并符合区域合规性策略与行业法规。

## <a name="solution"></a>解决方案

使用本地和公有云环境来满足多设施企业的需求。 Azure Stack Hub 提供快速、安全且灵活的解决方案用于收集、处理、存储和分发本地与远程数据。 当安全性、保密性、企业政策与法规要求根据不同的位置和用户而异时，此模式特别有用。

![用于分析解决方案体系结构的分层数据模式](media/pattern-tiered-data-analytics/solution-architecture.png)

## <a name="components"></a>组件

此模式使用以下组件：

| 层 | 组件 | 说明 |
|----------|-----------|-------------|
| Azure | 存储 | [Azure 存储](/azure/storage/)帐户提供干净的数据使用终结点。 Azure 存储是 Microsoft 提供的适用于现代数据存储场景的云存储解决方案。 Azure 存储为数据对象提供可大规模缩放的对象存储，并为云提供文件系统服务。 它也提供用于可靠传送消息的消息存储，以及 NoSQL 存储。 |
| Azure Stack Hub | 存储 | [Azure Stack Hub 存储](/azure-stack/user/azure-stack-storage-overview)帐户用于多个服务：<br><br>- 用于存储原始数据的 **Blob 存储**。 Blob 存储可以保存任何类型的文本或二进制数据，例如文档、媒体文件或应用安装程序。 每个 Blob 组织在容器中。 容器提供了一种有用的方式来向对象组分配安全策略。 一个存储帐户可以包含任意数目的容器，一个容器可以包含任意数目的 Blob，直至达到存储帐户的容量限制 500 TB。<br>- 用于数据存档的 **Blob 存储**。 用于存档冷数据的低成本存储有其优势。 冷数据的示例包括备份、媒体内容、科研数据、合规性和存档数据。 一般而言，不经常访问的任何数据都被视为冷存储。 根据访问频率和保留期等属性将数据分层。 客户数据并不经常被访问，但其访问延迟和性能需要与热数据类似。<br>- 用于存储已处理的数据的**队列存储**。 队列存储用于在应用组件之间进行云消息传送。 在设计应用以实现可伸缩性时，通常要将各个应用组件分离，使其可以独立地进行伸缩。 队列存储提供的异步消息传送适用于在应用组件之间进行通信，无论这些应用组件是运行在云中、桌面上、本地服务器上还是移动设备上。 队列存储还支持管理异步任务以及构建过程工作流。 |
| | Azure Functions | [Azure Functions](/azure/azure-functions/) 服务由 [Azure Stack Hub 上的 Azure 应用服务](/azure-stack/operator/azure-stack-app-service-overview)资源提供程序提供。 使用 Azure Functions 可在简单的无服务器环境中执行代码，以响应各种事件。 Azure Functions 可以使用所选编程语言按需缩放，而无需创建 VM 或发布 Web 应用。 该解决方案将 Functions 用于以下操作：：<br><br>- **数据引入**<br>- **数据净化。** 手动触发的函数可以执行计划的数据处理、清理和存档。 示例可能包括夜间客户列表清理与每月报表处理。|

## <a name="issues-and-considerations"></a>问题和注意事项

在决定如何实现此解决方案时，请考虑以下几点：

### <a name="scalability"></a>可伸缩性

Azure Functions 和存储解决方案可以通过缩放来满足数据量和处理方面的需求。 有关 Azure 可伸缩性的信息和目标，请参阅 [Azure 存储可伸缩性文档](/azure/storage/common/storage-scalability-targets)。

### <a name="availability"></a>可用性

就此模式来说，存储是主要的可用性注意事项。 处理和分配大量的数据时，必须通过快速链路进行连接。

### <a name="manageability"></a>可管理性

此解决方案的可管理性取决于所用的创作工具，以及是否采用源代码管理。

## <a name="next-steps"></a>后续步骤

若要详细了解本文中介绍的主题：

- 请参阅 [Azure 存储](/azure/storage/)和 [Azure Functions](/azure/azure-functions/) 文档。 此模式大量使用 azure 存储帐户，并可在 Azure 和 Azure Stack 中心 Azure Functions。
- 请参阅[混合应用程序设计注意事项](overview-app-design-considerations.md)以了解有关最佳做法的详细信息，并获取对其他问题的解答。
- 请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。

准备好测试解决方案示例时，请继续阅读[分层式数据分析解决方案部署指南](https://aka.ms/tiereddatadeploy)。 该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。
