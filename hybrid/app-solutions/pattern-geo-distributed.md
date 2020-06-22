---
title: Azure Stack Hub 中的地理分散式应用模式
description: 了解使用 Azure 和 Azure Stack Hub 的智能边缘的地理分散式应用模式。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909911"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="506e4-103">地理分散式应用模式</span><span class="sxs-lookup"><span data-stu-id="506e4-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="506e4-104">了解如何跨多个区域提供应用终结点，并根据位置和合规性需求路由用户流量。</span><span class="sxs-lookup"><span data-stu-id="506e4-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="506e4-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="506e4-105">Context and problem</span></span>

<span data-ttu-id="506e4-106">在地理上分散的组织需致力于以安全且准确的方式分发数据并允许对数据进行访问，同时确保按用户、位置和设备跨边界实现所需级别的安全性、符合性和性能。</span><span class="sxs-lookup"><span data-stu-id="506e4-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="506e4-107">解决方案</span><span class="sxs-lookup"><span data-stu-id="506e4-107">Solution</span></span>

<span data-ttu-id="506e4-108">Azure Stack Hub 地理流量路由模式或地理分散式应用允许根据各种指标将流量定向到特定终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="506e4-109">使用基于地理位置的路由和终结点配置创建流量管理器即可根据区域要求、企业与国际法规和数据需求将流量路由到终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![地理分散模式](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="506e4-111">组件</span><span class="sxs-lookup"><span data-stu-id="506e4-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="506e4-112">云外部</span><span class="sxs-lookup"><span data-stu-id="506e4-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="506e4-113">流量管理器</span><span class="sxs-lookup"><span data-stu-id="506e4-113">Traffic Manager</span></span>

<span data-ttu-id="506e4-114">在图中，流量管理器位于公有云外部，但它需要能够在本地数据中心和公有云中协调流量。</span><span class="sxs-lookup"><span data-stu-id="506e4-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="506e4-115">均衡器将流量路由到各个地理位置。</span><span class="sxs-lookup"><span data-stu-id="506e4-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="506e4-116">域名系统 (DNS)</span><span class="sxs-lookup"><span data-stu-id="506e4-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="506e4-117">域名系统或 DNS 负责将网站或服务名称转换（或解析）为它的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="506e4-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="506e4-118">公有云</span><span class="sxs-lookup"><span data-stu-id="506e4-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="506e4-119">云终结点</span><span class="sxs-lookup"><span data-stu-id="506e4-119">Cloud Endpoint</span></span>

<span data-ttu-id="506e4-120">公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="506e4-121">本地云</span><span class="sxs-lookup"><span data-stu-id="506e4-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="506e4-122">本地终结点</span><span class="sxs-lookup"><span data-stu-id="506e4-122">Local endpoint</span></span>

<span data-ttu-id="506e4-123">公共 IP 地址用于将传入流量通过流量管理器路由到公有云应用资源终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="506e4-124">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="506e4-124">Issues and considerations</span></span>

<span data-ttu-id="506e4-125">在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="506e4-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="506e4-126">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="506e4-126">Scalability</span></span>

<span data-ttu-id="506e4-127">此模式用于处理地理流量路由，而不是通过缩放来应对流量增加的情况。</span><span class="sxs-lookup"><span data-stu-id="506e4-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="506e4-128">但是，可以将此模式与其他 Azure 和本地解决方案结合使用。</span><span class="sxs-lookup"><span data-stu-id="506e4-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="506e4-129">例如，此模式可与跨云缩放模式配合使用。</span><span class="sxs-lookup"><span data-stu-id="506e4-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="506e4-130">可用性</span><span class="sxs-lookup"><span data-stu-id="506e4-130">Availability</span></span>

<span data-ttu-id="506e4-131">确定通过本地硬件配置和软件部署来配置本地部署的应用，以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="506e4-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="506e4-132">可管理性</span><span class="sxs-lookup"><span data-stu-id="506e4-132">Manageability</span></span>

<span data-ttu-id="506e4-133">此模式可确保在环境之间提供无缝的管理和熟悉的界面。</span><span class="sxs-lookup"><span data-stu-id="506e4-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="506e4-134">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="506e4-134">When to use this pattern</span></span>

- <span data-ttu-id="506e4-135">我的组织有海外分公司，需要自定义的区域安全性和分发策略。</span><span class="sxs-lookup"><span data-stu-id="506e4-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="506e4-136">我组织的每个办事处提取员工、业务和设施数据，需要根据当地法规和时区报告活动。</span><span class="sxs-lookup"><span data-stu-id="506e4-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="506e4-137">只要对单个区域内和跨区域的多个应用部署进行应用的水平扩展，即可达到高缩放性要求以处理极高的负载要求。</span><span class="sxs-lookup"><span data-stu-id="506e4-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="506e4-138">这些应用必须高度可用并能够快速响应客户端请求，即使发生了单区域中断。</span><span class="sxs-lookup"><span data-stu-id="506e4-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="506e4-139">后续步骤</span><span class="sxs-lookup"><span data-stu-id="506e4-139">Next steps</span></span>

<span data-ttu-id="506e4-140">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="506e4-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="506e4-141">请参阅 [Azure 流量管理器概述](/azure/traffic-manager/traffic-manager-overview)，详细了解这个基于 DNS 的流量负载均衡器的工作原理。</span><span class="sxs-lookup"><span data-stu-id="506e4-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="506e4-142">请参阅[混合应用设计注意事项](overview-app-design-considerations.md)以了解有关最佳做法的详细信息，并获取对其他任何问题的解答。</span><span class="sxs-lookup"><span data-stu-id="506e4-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="506e4-143">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="506e4-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="506e4-144">准备好测试解决方案示例时，请继续阅读[地理分散式应用解决方案部署指南](solution-deployment-guide-geo-distributed.md)。</span><span class="sxs-lookup"><span data-stu-id="506e4-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="506e4-145">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="506e4-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="506e4-146">了解如何根据各种指标，使用地理分散式应用模式将流量定向到特定的终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="506e4-147">使用基于地理位置的路由和终结点配置创建流量管理器配置文件可确保根据区域要求、企业与国际法规和数据需求将信息路由到终结点。</span><span class="sxs-lookup"><span data-stu-id="506e4-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
