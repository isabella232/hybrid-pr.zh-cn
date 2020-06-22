---
title: Azure 和 Azure Stack Hub 中的混合中继模式
description: 使用 Azure 和 Azure Stack Hub 中的混合中继模式来连接到受防火墙保护的边缘资源。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909949"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="b78ee-103">混合中继模式</span><span class="sxs-lookup"><span data-stu-id="b78ee-103">Hybrid relay pattern</span></span>

<span data-ttu-id="b78ee-104">了解如何使用混合中继模式和 Azure 中继连接到边缘资源或受防火墙保护的设备。</span><span class="sxs-lookup"><span data-stu-id="b78ee-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b78ee-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="b78ee-105">Context and problem</span></span>

<span data-ttu-id="b78ee-106">边缘设备通常位于企业防火墙或 NAT 设备的后面。</span><span class="sxs-lookup"><span data-stu-id="b78ee-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="b78ee-107">尽管它们是安全的，但它们可能无法与公有云或其他企业网络中的边缘设备通信。</span><span class="sxs-lookup"><span data-stu-id="b78ee-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="b78ee-108">可能有必要以安全方式向公有云中的用户公开特定的端口和功能。</span><span class="sxs-lookup"><span data-stu-id="b78ee-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="b78ee-109">解决方案</span><span class="sxs-lookup"><span data-stu-id="b78ee-109">Solution</span></span>

<span data-ttu-id="b78ee-110">混合中继模式使用 Azure 中继在两个终结点之间建立不能直接进行通信的 Websocket 隧道。</span><span class="sxs-lookup"><span data-stu-id="b78ee-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="b78ee-111">不在本地但需要连接到本地终结点的设备将连接到公有云中的终结点。</span><span class="sxs-lookup"><span data-stu-id="b78ee-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="b78ee-112">此终结点通过安全通道按预定义的路由重定向流量。</span><span class="sxs-lookup"><span data-stu-id="b78ee-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="b78ee-113">本地环境内部的终结点接收流量，并将流量路由到正确的目标。</span><span class="sxs-lookup"><span data-stu-id="b78ee-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![混合中继模式解决方案体系结构](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="b78ee-115">混合中继模式的工作原理：</span><span class="sxs-lookup"><span data-stu-id="b78ee-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="b78ee-116">设备通过预定义的端口连接到 Azure 中的虚拟机 (VM)。</span><span class="sxs-lookup"><span data-stu-id="b78ee-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="b78ee-117">流量会转发到 Azure 中的 Azure 中继。</span><span class="sxs-lookup"><span data-stu-id="b78ee-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="b78ee-118">Azure Stack 集线器上已建立与 Azure 中继的长期连接的 VM，接收流量并将其转发到目标。</span><span class="sxs-lookup"><span data-stu-id="b78ee-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="b78ee-119">本地服务或终结点处理请求。</span><span class="sxs-lookup"><span data-stu-id="b78ee-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="b78ee-120">组件</span><span class="sxs-lookup"><span data-stu-id="b78ee-120">Components</span></span>

<span data-ttu-id="b78ee-121">此解决方案使用以下组件：</span><span class="sxs-lookup"><span data-stu-id="b78ee-121">This solution uses the following components:</span></span>

| <span data-ttu-id="b78ee-122">层</span><span class="sxs-lookup"><span data-stu-id="b78ee-122">Layer</span></span> | <span data-ttu-id="b78ee-123">组件</span><span class="sxs-lookup"><span data-stu-id="b78ee-123">Component</span></span> | <span data-ttu-id="b78ee-124">说明</span><span class="sxs-lookup"><span data-stu-id="b78ee-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="b78ee-125">Azure</span><span class="sxs-lookup"><span data-stu-id="b78ee-125">Azure</span></span> | <span data-ttu-id="b78ee-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="b78ee-126">Azure VM</span></span> | <span data-ttu-id="b78ee-127">Azure VM 提供本地资源的可公开访问终结点。</span><span class="sxs-lookup"><span data-stu-id="b78ee-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="b78ee-128">Azure 中继</span><span class="sxs-lookup"><span data-stu-id="b78ee-128">Azure Relay</span></span> | <span data-ttu-id="b78ee-129">[Azure 中继](/azure/azure-relay/)提供基础结构，用于维护 Azure vm 与 AZURE STACK 中心 vm 之间的隧道和连接。</span><span class="sxs-lookup"><span data-stu-id="b78ee-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="b78ee-130">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="b78ee-130">Azure Stack Hub</span></span> | <span data-ttu-id="b78ee-131">计算</span><span class="sxs-lookup"><span data-stu-id="b78ee-131">Compute</span></span> | <span data-ttu-id="b78ee-132">Azure Stack Hub VM 提供混合中继隧道的服务器端。</span><span class="sxs-lookup"><span data-stu-id="b78ee-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="b78ee-133">存储</span><span class="sxs-lookup"><span data-stu-id="b78ee-133">Storage</span></span> | <span data-ttu-id="b78ee-134">已部署到 Azure Stack Hub 中的 AKS 引擎群集提供可弹性缩放的引擎来运行人脸 API 容器。</span><span class="sxs-lookup"><span data-stu-id="b78ee-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="b78ee-135">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="b78ee-135">Issues and considerations</span></span>

<span data-ttu-id="b78ee-136">在决定如何实现此解决方案时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="b78ee-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="b78ee-137">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="b78ee-137">Scalability</span></span>

<span data-ttu-id="b78ee-138">此模式只允许在客户端与服务器上进行 1:1 端口映射。</span><span class="sxs-lookup"><span data-stu-id="b78ee-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="b78ee-139">例如，如果端口 80 已用于 Azure 终结点上的一个服务的隧道传输，则该端口不可用于另一个服务。</span><span class="sxs-lookup"><span data-stu-id="b78ee-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="b78ee-140">应该相应地规划好端口映射。</span><span class="sxs-lookup"><span data-stu-id="b78ee-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="b78ee-141">应适当调整 Azure 中继和 Vm 来处理流量。</span><span class="sxs-lookup"><span data-stu-id="b78ee-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="b78ee-142">可用性</span><span class="sxs-lookup"><span data-stu-id="b78ee-142">Availability</span></span>

<span data-ttu-id="b78ee-143">这些隧道和连接不是冗余的。</span><span class="sxs-lookup"><span data-stu-id="b78ee-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="b78ee-144">若要确保高可用性，可以实现错误检查代码。</span><span class="sxs-lookup"><span data-stu-id="b78ee-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="b78ee-145">另一种方法是在负载均衡器后面使用 Azure 中继连接的虚拟机池。</span><span class="sxs-lookup"><span data-stu-id="b78ee-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="b78ee-146">可管理性</span><span class="sxs-lookup"><span data-stu-id="b78ee-146">Manageability</span></span>

<span data-ttu-id="b78ee-147">此解决方案可能跨越许多设备和位置，因而变得不好管理。</span><span class="sxs-lookup"><span data-stu-id="b78ee-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="b78ee-148">Azure 的 IoT 服务可自动将新的位置和设备联机，并使其保持最新状态。</span><span class="sxs-lookup"><span data-stu-id="b78ee-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="b78ee-149">安全性</span><span class="sxs-lookup"><span data-stu-id="b78ee-149">Security</span></span>

<span data-ttu-id="b78ee-150">这种模式允许从边缘自由访问内部设备上的端口。</span><span class="sxs-lookup"><span data-stu-id="b78ee-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="b78ee-151">请考虑对内部设备上的服务或者混合中继终结点前面的服务添加身份验证机制。</span><span class="sxs-lookup"><span data-stu-id="b78ee-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b78ee-152">后续步骤</span><span class="sxs-lookup"><span data-stu-id="b78ee-152">Next steps</span></span>

<span data-ttu-id="b78ee-153">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="b78ee-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="b78ee-154">此模式使用 Azure 中继。</span><span class="sxs-lookup"><span data-stu-id="b78ee-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="b78ee-155">有关详细信息，请参阅[Azure 中继文档](/azure/azure-relay/)。</span><span class="sxs-lookup"><span data-stu-id="b78ee-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="b78ee-156">请参阅[混合应用程序设计注意事项](overview-app-design-considerations.md)以了解有关最佳做法的详细信息，并获取对其他任何问题的解答。</span><span class="sxs-lookup"><span data-stu-id="b78ee-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="b78ee-157">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="b78ee-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="b78ee-158">准备好测试解决方案示例时，请继续阅读[混合中继解决方案部署指南](https://aka.ms/hybridrelaydeployment)。</span><span class="sxs-lookup"><span data-stu-id="b78ee-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="b78ee-159">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="b78ee-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>