---
title: 使用 Azure 和 Azure Stack Hub 的客流量检测模式
description: 了解如何使用 Azure 和 Azure Stack Hub 实现基于 AI 的客流量检测解决方案，以分析零售商店流量。
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895323"
---
# <a name="footfall-detection-pattern"></a>客流量检测模式

此模式概述了如何实现基于 AI 的客流量检测解决方案，以便分析零售商店中的访客流量。 该解决方案使用 Azure、Azure Stack Hub 和自定义视觉 AI 开发工具包从实际操作生成见解。

## <a name="context-and-problem"></a>上下文和问题

Contoso Stores 希望就商店布局方面获得有关客户如何接收其当前产品的见解。 他们无法在每个地区派驻员工，并且派分析师团队查看整个商店的摄像机素材也较为低效。 此外，他们的所有商店都没有足够的带宽来将所有摄像机视频流式传输到云进行分析。

Contoso 想找到一种不引人注目且不侵犯隐私的方式来确定其客户的人口统计数据、忠诚度以及对店面展示和产品的反应。

## <a name="solution"></a>解决方案

此零售分析模式使用分层方法在边缘进行推断。 通过使用自定义视觉 AI 开发工具包，只会将包含人脸的图像发送到运行 Azure 认知服务的专用 Azure Stack Hub 进行分析。 匿名的聚合数据将发送到 Azure，以便在 Power BI 中跨所有商店进行聚合并可视化呈现。 通过结合使用边缘和公有云，Contoso 可以利用新式 AI 技术，同时遵守其公司政策并保护客户的隐私。

[![客流量检测模式解决方案](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

下面是该解决方案的工作原理的摘要：

1. 自定义视觉 AI 开发工具包从 IoT 中心获取配置，该配置将安装 IoT Edge 运行时和 ML 模型。
2. 如果该模型看到某个人，则会拍照并将其上传到 Azure Stack Hub blob 存储。
3. blob 服务在 Azure Stack Hub 上触发 Azure Function。
4. Azure Function 会调用使用人脸 API 的容器，以通过图像获取人口统计数据和情绪数据。
5. 该数据是匿名的并将发送到 Azure 事件中心群集。
6. 事件中心群集会将数据推送到流分析。
7. 流分析将聚合数据并将其推送到 Power BI。

## <a name="components"></a>组件

此解决方案使用以下组件：

| 层 | 组件 | 说明 |
|----------|-----------|-------------|
| 店内硬件 | [自定义视觉 AI 开发工具包](https://azure.github.io/Vision-AI-DevKit-Pages/) | 使用仅捕获人像的本地 ML 模型来执行店内筛选，以便进行分析。 通过 IoT 中心安全地预配和更新。<br><br>|
| Azure | [Azure 事件中心](/azure/event-hubs/) | Azure 事件中心提供与 Azure 流分析完美集成的可缩放平台，用于引入匿名数据。 |
|  | [Azure 流分析](/azure/stream-analytics/) | Azure 流分析作业聚合匿名数据，并将其分组成 15 秒的时段以进行可视化呈现。 |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI 提供易用的仪表板界面用于查看 Azure 流分析的输出。 |
| Azure Stack Hub | [应用服务](/azure-stack/operator/azure-stack-app-service-overview) | 应用服务资源提供程序 (RP) 为边缘组件提供基础，其中包括用于 Web 应用/API 和 Functions 的托管和管理功能。 |
| | Azure Kubernetes 服务 [(AKS) 引擎](https://github.com/Azure/aks-engine)群集 | 包含已部署到 Azure Stack Hub 的 AKS 引擎群集的 AKS RP 提供可缩放且可复原的引擎来运行人脸 API 容器。 |
| | Azure 认知服务[人脸 API 容器](/azure/cognitive-services/face/face-how-to-install-containers)| 使用人脸 API 容器的 Azure 认知服务 RP 在 Contoso 的专用网络上提供人口统计数据、情绪和独立访客检测。 |
| | Blob 存储 | 从 AI 开发工具包捕获的图像将上传到 Azure Stack Hub 的 blob 存储。 |
| | Azure Functions | 在 Azure Stack Hub 上运行的 Azure Function 会接收 blob 存储的输入，并管理与人脸 API 的交互。 它将匿名数据发送到位于 Azure 中的事件中心群集。<br><br>|

## <a name="issues-and-considerations"></a>问题和注意事项

在决定如何实现此解决方案时，请考虑以下几点：

### <a name="scalability"></a>可伸缩性

若要使此解决方案能够跨多个摄像机和位置进行扩展，需要确保所有组件都能处理增加的负载。 可能需要执行如下操作：

- 增加流分析流式处理单元的数量。
- 横向扩展人脸 API 部署。
- 提高事件中心群集吞吐量。
- 对于极端情况，可能需要从 Azure Functions 迁移到虚拟机。

### <a name="availability"></a>可用性

由于此解决方案是分层的，因此请务必考虑如何应对网络或电源故障。 根据业务需要，你可能想要实现一种机制 - 可在本地缓存图像，然后在连接返回时将图像转发到 Azure Stack Hub。 如果位置足够大，将使用人脸 API 容器的 Data Box Edge 部署到该位置可能是更好的选择。

### <a name="manageability"></a>可管理性

此解决方案可能跨越许多设备和位置，因而变得不好管理。 [Azure 的 IoT 服务](/azure/iot-fundamentals/)可用于自动将新的位置和设备联机，并使其保持最新状态。

### <a name="security"></a>安全性

此解决方案捕获客户图像，因此安全性是极其重要的考虑因素。 请确保所有存储帐户都通过适当的访问策略进行保护并定期轮换密钥。 确保存储帐户和事件中心具有符合公司和政府隐私法规的保留策略。 另外，请确保对用户访问级别进行分层。 分层可确保用户只能访问其角色所需的数据。

## <a name="next-steps"></a>后续步骤

若要详细了解本文中介绍的主题：

- 请参阅[分层数据模式](https://aka.ms/tiereddatadeploy)，客流量检测模式使用该模式。
- 有关使用自定义视觉的详细信息，请参阅[自定义视觉 AI 开发工具包](https://azure.github.io/Vision-AI-DevKit-Pages/)。 

准备好测试解决方案示例时，请阅读[客流量检测部署指南](solution-deployment-guide-retail-footfall-detection.md)并继续操作。 该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。
