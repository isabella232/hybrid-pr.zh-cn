---
title: 使用 Azure 和 Azure Stack Edge 进行缺货检测
description: 了解如何使用 Azure 和 Azure Stack Edge 服务实现缺货检测。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909955"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>边缘缺货检测模式

本模式演示如何使用 Azure Stack Edge 或 Azure IoT Edge 设备与网络摄像头来确定货架上是否有商品缺货。

## <a name="context-and-problem"></a>上下文和问题

实体零售店的销售额下降，因为当客户查找某个商品时，货架上没有该商品。 但是，该商品可能位于商店后面，只是未补货上架。 商店想要提高员工的工作效率，并在商品需要补货时自动接收通知。

## <a name="solution"></a>解决方案

解决方案示例使用边缘设备（例如每家商店布置一个 Azure Stack Edge），以有效处理商店中的摄像头数据。 这种优化的设计可让商店只将相关的事件与图像发送到云。 该设计可节省带宽、存储空间，并保护客户的隐私。 从每个摄像头读取帧时，机器学习模型将处理图像，并返回任何缺货区域。 图像与缺货区域显示在本地 Web 应用上。 此数据可以发送到时序见解环境，以在 Power BI 中显示见解。

![边缘缺货解决方案体系结构](media/pattern-out-of-stock-at-edge/solution-architecture.png)

该解决方案的工作原理如下：

1. 通过 HTTP 或 RTSP 捕获网络摄像头的图像。
2. 图像在调整大小后发送到推理驱动程序，后者与机器学习模型通信，以确定是否有任何缺货图像。
3. 机器学习模型返回任何缺货区域。
4. 推理驱动程序将未经处理的图像上传到 Blob（如果已指定），并将模型的结果发送到 Azure IoT 中心，以及设备上的边界框处理器。
5. 边界框处理器将边界框添加到图像，并在内存中数据库中缓存图像路径。
6. Web 应用查询图像，并按接收顺序显示图像。
7. 在时序见解中聚合来自 IoT 中心的消息。
8. Power BI 显示一段时间内缺货商品的交互式报表，其中包括时序见解数据。


## <a name="components"></a>组件

此解决方案使用以下组件：

| 层 | 组件 | 说明 |
|----------|-----------|-------------|
| 本地硬件 | 网络摄像头 | 需要一个网络摄像头，并配合 HTTP 或 RTSP 源提供用于推理的图像。 |
| Azure | Azure IoT 中心 | [Azure IoT 中心](/azure/iot-hub/)为边缘设备处理设备预配和消息传递。 |
|  | Azure Time Series Insights | [Azure 时序见解](/azure/time-series-insights/)存储来自 IoT 中心的消息用于可视化。 |
|  | Power BI | [Microsoft Power BI](https://powerbi.microsoft.com/) 提供缺货事件的以业务为中心的报表。 Power BI 提供易用的仪表板界面用于查看 Azure 流分析的输出。 |
| Azure Stack Edge 或<br>Azure IoT Edge 设备 | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) 协调本地容器的运行时，并处理设备管理和更新。|
| | Azure Project Brainwave | 在 Azure Stack Edge 设备上，[Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 使用现场可编程门阵列 (FPGA) 来加速机器学习推理。|

## <a name="issues-and-considerations"></a>问题和注意事项

在决定如何实现此解决方案时，请考虑以下几点：

### <a name="scalability"></a>可伸缩性

根据提供的硬件，大多数机器学习模型只能按特定的每秒帧数运行。 请确定摄像头的最佳采样率，以确保机器学习管道不会备份。 不同类型的硬件将处理不同数目的摄像头与帧速率。

### <a name="availability"></a>可用性

必须考虑到边缘设备断开连接时可能发生的情况。 请考虑时序见解与 Power BI 仪表板可能丢失哪些数据。 提供的示例解决方案在设计上不具备高可用性。

### <a name="manageability"></a>可管理性

此解决方案可能跨越许多设备和位置，因而变得不好管理。 Azure 的 IoT 服务可自动将新的位置和设备联机，并使其保持最新状态。 此外，必须遵循适当的数据监管过程。

### <a name="security"></a>安全性

本模式将处理潜在敏感的数据。 请确保密钥定期轮换，并已正确设置 Azure 存储帐户和本地共享的权限。

## <a name="next-steps"></a>后续步骤

若要详细了解本文中介绍的主题：
- 本模式中使用了多个 IoT 相关服务，包括 [Azure IoT Edge](/azure/iot-edge/)、[Azure IoT 中心](/azure/iot-hub/)和 [Azure 时序见解](/azure/time-series-insights/)。
- 若要了解有关 Microsoft Project 脑电波的详细信息，请参阅[博客公告](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)和签出[Azure 加速机器学习与 Project 脑电波视频](https://www.youtube.com/watch?v=DJfMobMjCX0)。
- 请参阅[混合应用设计注意事项](overview-app-design-considerations.md)，了解有关最佳做法的详细信息，并获取对其他任何问题的解答。
- 请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。

准备好测试解决方案示例时，请继续阅读[分层式数据分析解决方案部署指南](https://aka.ms/edgeinferencingdeploy)。 该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。
