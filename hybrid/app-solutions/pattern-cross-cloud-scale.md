---
title: Azure Stack Hub 中的跨云缩放模式
description: 了解如何在 Azure 和 Azure Stack Hub 中生成可缩放的跨云应用。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909947"
---
# <a name="cross-cloud-scaling-pattern"></a>跨云缩放模式

自动将资源添加到现有的应用，以适应负载的增长。

## <a name="context-and-problem"></a>上下文和问题

应用无法提高容量来满足意外增长的需求。 缺少可伸缩性会导致用户在高峰期无法访问应用。 应用可为固定数目的用户提供服务。

全球企业都需要安全、可靠且可用的基于云的应用。 满足增长的需求并使用适当的基础结构来支持此需求，这一点至关重要。 为了在成本和维护以及业务数据安全、存储和实时可用性之间取得平衡，企业费尽了心力。

你可能无法在公有云中运行应用。 但是，要让企业在本地环境中保持用于处理应用需求高峰的容量，在经济上似乎不切实际。 通过此模式，可对本地解决方案利用公有云的弹性。

## <a name="solution"></a>解决方案

跨云缩放模式使用公有云资源扩展本地云中的应用。 该模式在需求增大或减小时触发，并相应地在云中添加或删除资源。 这些资源提供冗余、快速可用性和地域合规的路由。

![跨云缩放模式](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> 此模式仅适用于应用的无状态组件。

## <a name="components"></a>组件

跨云缩放模式由以下组件构成。

### <a name="outside-the-cloud"></a>云外部

#### <a name="traffic-manager"></a>流量管理器

在图中，流量管理器位于公有云组的外部，但它需要能够在本地数据中心和公有云中协调流量。 均衡器通过监视终结点并在必要时提供故障转移再分发，实现应用的高可用性。

#### <a name="domain-name-system-dns"></a>域名系统 (DNS)

域名系统或 DNS 负责将网站或服务名称转换（或解析）为它的 IP 地址。

### <a name="cloud"></a>云

#### <a name="hosted-build-server"></a>托管的生成服务器

用于托管生成管道的环境。

#### <a name="app-resources"></a>应用资源

应用资源（例如虚拟机规模集和容器）需要能够横向缩减和横向扩展。

#### <a name="custom-domain-name"></a>自定义域名

使用自定义域名来路由请求 glob。

#### <a name="public-ip-addresses"></a>公共 IP 地址

公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。  

### <a name="local-cloud"></a>本地云

#### <a name="hosted-build-server"></a>托管的生成服务器

用于托管生成管道的环境。

#### <a name="app-resources"></a>应用资源

应用资源（例如虚拟机规模集和容器）需要能够横向缩减和横向扩展。

#### <a name="custom-domain-name"></a>自定义域名

使用自定义域名来路由请求 glob。

#### <a name="public-ip-addresses"></a>公共 IP 地址

公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。

## <a name="issues-and-considerations"></a>问题和注意事项

在决定如何实现此模式时，请考虑以下几点：

### <a name="scalability"></a>可伸缩性

跨云缩放的关键组件提供按需缩放能力。 缩放必须在公有云和本地云基础结构之间进行，并根据需求提供一致且可靠的服务。

### <a name="availability"></a>可用性

确定通过本地硬件配置和软件部署来配置本地部署的应用，以实现高可用性。

### <a name="manageability"></a>可管理性

跨云模式确保在环境之间提供无缝的管理和熟悉的界面。

## <a name="when-to-use-this-pattern"></a>何时使用此模式

使用此模式：

- 需要提高应用程序容量来应对意外的需求或定期需求时。
- 不想投资购买只有高峰期才用到的资源。 只想为所用的资源付费时。

对于以下情况不建议使用此模式：

- 解决方案要求用户通过 Internet 进行连接。
- 企业所在地的法规要求发起的连接来自现场。
- 网络遇到使缩放性能受限的一般瓶颈。
- 环境已从 Internet 断开连接，因此无法访问公有云。

## <a name="next-steps"></a>后续步骤

若要详细了解本文中介绍的主题：

- 请参阅 [Azure 流量管理器概述](/azure/traffic-manager/traffic-manager-overview)，详细了解这个基于 DNS 的流量负载均衡器的工作原理。
- 请参阅[混合应用程序设计注意事项](overview-app-design-considerations.md)以了解有关最佳做法的详细信息，并获取其他问题的解答。
- 请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。

准备好测试解决方案示例时，请继续阅读[跨云缩放解决方案部署指南](solution-deployment-guide-cross-cloud-scaling.md)。 该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。 其中还介绍了如何创建可提供手动触发过程的跨云解决方案，以便从 Azure Stack Hub 托管的 Web 应用切换到 Azure 托管的 Web 应用。 此外，介绍了如何通过流量管理器使用自动缩放功能，来确保在承受负载的情况下保持云设施的灵活性和可伸缩性。
