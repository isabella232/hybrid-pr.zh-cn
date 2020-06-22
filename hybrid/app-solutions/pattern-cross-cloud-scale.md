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
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="63000-103">跨云缩放模式</span><span class="sxs-lookup"><span data-stu-id="63000-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="63000-104">自动将资源添加到现有的应用，以适应负载的增长。</span><span class="sxs-lookup"><span data-stu-id="63000-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="63000-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="63000-105">Context and problem</span></span>

<span data-ttu-id="63000-106">应用无法提高容量来满足意外增长的需求。</span><span class="sxs-lookup"><span data-stu-id="63000-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="63000-107">缺少可伸缩性会导致用户在高峰期无法访问应用。</span><span class="sxs-lookup"><span data-stu-id="63000-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="63000-108">应用可为固定数目的用户提供服务。</span><span class="sxs-lookup"><span data-stu-id="63000-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="63000-109">全球企业都需要安全、可靠且可用的基于云的应用。</span><span class="sxs-lookup"><span data-stu-id="63000-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="63000-110">满足增长的需求并使用适当的基础结构来支持此需求，这一点至关重要。</span><span class="sxs-lookup"><span data-stu-id="63000-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="63000-111">为了在成本和维护以及业务数据安全、存储和实时可用性之间取得平衡，企业费尽了心力。</span><span class="sxs-lookup"><span data-stu-id="63000-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="63000-112">你可能无法在公有云中运行应用。</span><span class="sxs-lookup"><span data-stu-id="63000-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="63000-113">但是，要让企业在本地环境中保持用于处理应用需求高峰的容量，在经济上似乎不切实际。</span><span class="sxs-lookup"><span data-stu-id="63000-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="63000-114">通过此模式，可对本地解决方案利用公有云的弹性。</span><span class="sxs-lookup"><span data-stu-id="63000-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="63000-115">解决方案</span><span class="sxs-lookup"><span data-stu-id="63000-115">Solution</span></span>

<span data-ttu-id="63000-116">跨云缩放模式使用公有云资源扩展本地云中的应用。</span><span class="sxs-lookup"><span data-stu-id="63000-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="63000-117">该模式在需求增大或减小时触发，并相应地在云中添加或删除资源。</span><span class="sxs-lookup"><span data-stu-id="63000-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="63000-118">这些资源提供冗余、快速可用性和地域合规的路由。</span><span class="sxs-lookup"><span data-stu-id="63000-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![跨云缩放模式](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="63000-120">此模式仅适用于应用的无状态组件。</span><span class="sxs-lookup"><span data-stu-id="63000-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="63000-121">组件</span><span class="sxs-lookup"><span data-stu-id="63000-121">Components</span></span>

<span data-ttu-id="63000-122">跨云缩放模式由以下组件构成。</span><span class="sxs-lookup"><span data-stu-id="63000-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="63000-123">云外部</span><span class="sxs-lookup"><span data-stu-id="63000-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="63000-124">流量管理器</span><span class="sxs-lookup"><span data-stu-id="63000-124">Traffic Manager</span></span>

<span data-ttu-id="63000-125">在图中，流量管理器位于公有云组的外部，但它需要能够在本地数据中心和公有云中协调流量。</span><span class="sxs-lookup"><span data-stu-id="63000-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="63000-126">均衡器通过监视终结点并在必要时提供故障转移再分发，实现应用的高可用性。</span><span class="sxs-lookup"><span data-stu-id="63000-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="63000-127">域名系统 (DNS)</span><span class="sxs-lookup"><span data-stu-id="63000-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="63000-128">域名系统或 DNS 负责将网站或服务名称转换（或解析）为它的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="63000-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="63000-129">云</span><span class="sxs-lookup"><span data-stu-id="63000-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="63000-130">托管的生成服务器</span><span class="sxs-lookup"><span data-stu-id="63000-130">Hosted build server</span></span>

<span data-ttu-id="63000-131">用于托管生成管道的环境。</span><span class="sxs-lookup"><span data-stu-id="63000-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="63000-132">应用资源</span><span class="sxs-lookup"><span data-stu-id="63000-132">App resources</span></span>

<span data-ttu-id="63000-133">应用资源（例如虚拟机规模集和容器）需要能够横向缩减和横向扩展。</span><span class="sxs-lookup"><span data-stu-id="63000-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="63000-134">自定义域名</span><span class="sxs-lookup"><span data-stu-id="63000-134">Custom domain name</span></span>

<span data-ttu-id="63000-135">使用自定义域名来路由请求 glob。</span><span class="sxs-lookup"><span data-stu-id="63000-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="63000-136">公共 IP 地址</span><span class="sxs-lookup"><span data-stu-id="63000-136">Public IP addresses</span></span>

<span data-ttu-id="63000-137">公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。</span><span class="sxs-lookup"><span data-stu-id="63000-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="63000-138">本地云</span><span class="sxs-lookup"><span data-stu-id="63000-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="63000-139">托管的生成服务器</span><span class="sxs-lookup"><span data-stu-id="63000-139">Hosted build server</span></span>

<span data-ttu-id="63000-140">用于托管生成管道的环境。</span><span class="sxs-lookup"><span data-stu-id="63000-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="63000-141">应用资源</span><span class="sxs-lookup"><span data-stu-id="63000-141">App resources</span></span>

<span data-ttu-id="63000-142">应用资源（例如虚拟机规模集和容器）需要能够横向缩减和横向扩展。</span><span class="sxs-lookup"><span data-stu-id="63000-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="63000-143">自定义域名</span><span class="sxs-lookup"><span data-stu-id="63000-143">Custom domain name</span></span>

<span data-ttu-id="63000-144">使用自定义域名来路由请求 glob。</span><span class="sxs-lookup"><span data-stu-id="63000-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="63000-145">公共 IP 地址</span><span class="sxs-lookup"><span data-stu-id="63000-145">Public IP addresses</span></span>

<span data-ttu-id="63000-146">公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。</span><span class="sxs-lookup"><span data-stu-id="63000-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="63000-147">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="63000-147">Issues and considerations</span></span>

<span data-ttu-id="63000-148">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="63000-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="63000-149">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="63000-149">Scalability</span></span>

<span data-ttu-id="63000-150">跨云缩放的关键组件提供按需缩放能力。</span><span class="sxs-lookup"><span data-stu-id="63000-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="63000-151">缩放必须在公有云和本地云基础结构之间进行，并根据需求提供一致且可靠的服务。</span><span class="sxs-lookup"><span data-stu-id="63000-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="63000-152">可用性</span><span class="sxs-lookup"><span data-stu-id="63000-152">Availability</span></span>

<span data-ttu-id="63000-153">确定通过本地硬件配置和软件部署来配置本地部署的应用，以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="63000-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="63000-154">可管理性</span><span class="sxs-lookup"><span data-stu-id="63000-154">Manageability</span></span>

<span data-ttu-id="63000-155">跨云模式确保在环境之间提供无缝的管理和熟悉的界面。</span><span class="sxs-lookup"><span data-stu-id="63000-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="63000-156">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="63000-156">When to use this pattern</span></span>

<span data-ttu-id="63000-157">使用此模式：</span><span class="sxs-lookup"><span data-stu-id="63000-157">Use this pattern:</span></span>

- <span data-ttu-id="63000-158">需要提高应用程序容量来应对意外的需求或定期需求时。</span><span class="sxs-lookup"><span data-stu-id="63000-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="63000-159">不想投资购买只有高峰期才用到的资源。</span><span class="sxs-lookup"><span data-stu-id="63000-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="63000-160">只想为所用的资源付费时。</span><span class="sxs-lookup"><span data-stu-id="63000-160">Pay for what you use.</span></span>

<span data-ttu-id="63000-161">对于以下情况不建议使用此模式：</span><span class="sxs-lookup"><span data-stu-id="63000-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="63000-162">解决方案要求用户通过 Internet 进行连接。</span><span class="sxs-lookup"><span data-stu-id="63000-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="63000-163">企业所在地的法规要求发起的连接来自现场。</span><span class="sxs-lookup"><span data-stu-id="63000-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="63000-164">网络遇到使缩放性能受限的一般瓶颈。</span><span class="sxs-lookup"><span data-stu-id="63000-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="63000-165">环境已从 Internet 断开连接，因此无法访问公有云。</span><span class="sxs-lookup"><span data-stu-id="63000-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="63000-166">后续步骤</span><span class="sxs-lookup"><span data-stu-id="63000-166">Next steps</span></span>

<span data-ttu-id="63000-167">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="63000-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="63000-168">请参阅 [Azure 流量管理器概述](/azure/traffic-manager/traffic-manager-overview)，详细了解这个基于 DNS 的流量负载均衡器的工作原理。</span><span class="sxs-lookup"><span data-stu-id="63000-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="63000-169">请参阅[混合应用程序设计注意事项](overview-app-design-considerations.md)以了解有关最佳做法的详细信息，并获取其他问题的解答。</span><span class="sxs-lookup"><span data-stu-id="63000-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="63000-170">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="63000-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="63000-171">准备好测试解决方案示例时，请继续阅读[跨云缩放解决方案部署指南](solution-deployment-guide-cross-cloud-scaling.md)。</span><span class="sxs-lookup"><span data-stu-id="63000-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="63000-172">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="63000-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="63000-173">其中还介绍了如何创建可提供手动触发过程的跨云解决方案，以便从 Azure Stack Hub 托管的 Web 应用切换到 Azure 托管的 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="63000-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="63000-174">此外，介绍了如何通过流量管理器使用自动缩放功能，来确保在承受负载的情况下保持云设施的灵活性和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="63000-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
