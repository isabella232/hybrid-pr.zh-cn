---
title: 适用于 Azure 和 Azure Stack Hub 的混合模式与解决方案示例
description: 概述混合模式和解决方案示例，用于在 Azure 和 Azure Stack Hub 上学习和构建混合解决方案。
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343852"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>适用于 Azure 和 Azure Stack 的混合解决方案模式与示例

Microsoft 通过单个一致的 Azure 生态系统提供 Azure 和 Azure Stack 产品与解决方案。 Microsoft Azure Stack 系列是 Azure 的扩展。

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>混合云和混合应用

Azure Stack 通过实现混合云来为本地环境和 Edge 提供云计算的敏捷性。 Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 将 Azure 从云扩展到主权数据中心、分支机构、现场和更远的范围。 利用这组多样化的功能，可以：

- 重复使用代码并在 Azure 与本地环境中一致地运行云原生应用。
- 使用 Azure 服务的可选连接来运行传统的虚拟化工作负荷。
- 将数据传输到云，或将数据保留在主权数据中心，以保持符合性。
- 运行硬件加速的机器学习、容器化或虚拟化工作负荷，所有这些操作都可以在智能边缘进行。

跨云的应用也称为“混合应用”。 你可以在 Azure 中构建混合云应用，并将其部署到位于任何位置的已连接或已断开连接的数据中心。

混合应用方案因适用于开发的资源而差异很大。 此外，它们还涉及地理位置、安全性、Internet 访问等考虑因素。 尽管此处所述的解决方案模式和示例可能无法满足所有需求，但它们提供了可供探索的指导原则和示例，并且可以在实施混合解决方案时重复使用。

## <a name="solution-patterns"></a>解决方案模式

解决方案模式从真实的客户场景和经验中得出一般化且可重复的设计指导。 模式是抽象的，可适用于不同类型的方案或垂直行业。 每种模式阐述了上下文和问题，并提供解决方案示例的概述。 解决方案示例旨在用作模式的可能实施方案。

模式文章有两种类型：

- 单模式：提供适用于单个通用方案的设计指导。
- 多模式：提供使用多模式应用程序的设计指导。 为了解决更复杂的方案或行业特定的问题，我们经常需要使用这种模式。

## <a name="solution-deployment-guides"></a>解决方案部署指南

分步式部署指南可帮助用户部署解决方案示例。 该指南也可能会参考 GitHub [解决方案示例存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中存储的随附代码示例。

## <a name="next-steps"></a>后续步骤

- 请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个组合。
- 浏览目录中的“模式”和“解决方案部署指南”部分，以详细了解每种模式和解决方案。
- 阅读[混合应用设计注意事项](overview-app-design-considerations.md)，以了解设计、部署和操作混合应用时的软件质量要素。
- [在 Azure Stack 上设置开发环境](/azure-stack/user/azure-stack-dev-start)和在 Azure Stack 上[部署第一个应用](/azure-stack/user/azure-stack-dev-start-deploy-app)。
