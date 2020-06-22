---
title: 在 Azure 和 Azure Stack 中心配置混合云连接
description: 了解如何使用 Azure 和 Azure Stack Hub 配置混合云连接。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0e1a0fc4fb4110fdb406d4b4b2e72abb8f5412c9
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909909"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a><span data-ttu-id="3bf82-103">使用 Azure 和 Azure Stack Hub 配置混合云连接</span><span class="sxs-lookup"><span data-stu-id="3bf82-103">Configure hybrid cloud connectivity using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3bf82-104">你可以使用混合连接模式，通过全球 Azure 中的安全性和 Azure Stack 中心访问资源。</span><span class="sxs-lookup"><span data-stu-id="3bf82-104">You can access resources with security in global Azure and Azure Stack Hub using the hybrid connectivity pattern.</span></span>

<span data-ttu-id="3bf82-105">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="3bf82-105">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3bf82-106">将数据保留在本地以满足隐私或法规要求，但要保持对全局 Azure 资源的访问权限。</span><span class="sxs-lookup"><span data-stu-id="3bf82-106">Keep data on-premises to meet privacy or regulatory requirements but keep access to global Azure resources.</span></span>
> - <span data-ttu-id="3bf82-107">在公有云 Azure 中使用云规模的应用部署和资源的同时，保留旧版系统。</span><span class="sxs-lookup"><span data-stu-id="3bf82-107">Maintain a legacy system while using cloud-scaled app deployment and resources in global Azure.</span></span>

> [!Tip]  
> <span data-ttu-id="3bf82-108">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3bf82-108">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3bf82-109">Microsoft Azure Stack 中心是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="3bf82-109">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3bf82-110">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="3bf82-110">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3bf82-111">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="3bf82-111">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3bf82-112">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="3bf82-112">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3bf82-113">先决条件</span><span class="sxs-lookup"><span data-stu-id="3bf82-113">Prerequisites</span></span>

<span data-ttu-id="3bf82-114">生成混合连接部署需要一些组件。</span><span class="sxs-lookup"><span data-stu-id="3bf82-114">A few components are required to build a hybrid connectivity deployment.</span></span> <span data-ttu-id="3bf82-115">这其中的某些组件需要时间来准备，因此请进行相应的计划。</span><span class="sxs-lookup"><span data-stu-id="3bf82-115">Some of these components take time to prepare, so plan accordingly.</span></span>

### <a name="azure"></a><span data-ttu-id="3bf82-116">Azure</span><span class="sxs-lookup"><span data-stu-id="3bf82-116">Azure</span></span>

- <span data-ttu-id="3bf82-117">如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-117">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>
- <span data-ttu-id="3bf82-118">在 Azure 中创建[web 应用](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-118">Create a [web app](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts) in Azure.</span></span> <span data-ttu-id="3bf82-119">请记下 web 应用 URL，因为你将在解决方案中需要它。</span><span class="sxs-lookup"><span data-stu-id="3bf82-119">Make note of the web app URL because you'll need it in the solution.</span></span>

### <a name="azure-stack-hub"></a><span data-ttu-id="3bf82-120">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3bf82-120">Azure Stack Hub</span></span>

<span data-ttu-id="3bf82-121">Azure OEM/硬件合作伙伴可以部署生产 Azure Stack 中心，所有用户都可以部署 Azure Stack 开发工具包（ASDK）。</span><span class="sxs-lookup"><span data-stu-id="3bf82-121">An Azure OEM/hardware partner can deploy a production Azure Stack Hub, and all users can deploy an Azure Stack Development Kit (ASDK).</span></span>

- <span data-ttu-id="3bf82-122">使用生产 Azure Stack 中心或部署 ASDK。</span><span class="sxs-lookup"><span data-stu-id="3bf82-122">Use your production Azure Stack Hub or deploy the ASDK.</span></span>
   >[!Note]
   ><span data-ttu-id="3bf82-123">部署 ASDK 可能需要长达 7 小时的时间，因此请进行相应的计划。</span><span class="sxs-lookup"><span data-stu-id="3bf82-123">Deploying the ASDK can take up to 7 hours, so plan accordingly.</span></span>

- <span data-ttu-id="3bf82-124">将[应用服务](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS 服务部署到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="3bf82-124">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy.md) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="3bf82-125">在 Azure Stack Hub 环境中[创建计划和套餐](/azure-stack/operator/service-plan-offer-subscription-overview.md)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-125">[Create plans and offers](/azure-stack/operator/service-plan-offer-subscription-overview.md) in the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="3bf82-126">在 Azure Stack Hub 环境中[创建租户订阅](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-126">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) within the Azure Stack Hub environment.</span></span>

### <a name="azure-stack-hub-components"></a><span data-ttu-id="3bf82-127">Azure Stack Hub 组件</span><span class="sxs-lookup"><span data-stu-id="3bf82-127">Azure Stack Hub components</span></span>

<span data-ttu-id="3bf82-128">Azure Stack Hub 操作员必须部署应用服务、创建计划和套餐、创建租户订阅，并添加 Windows Server 2016 映像。</span><span class="sxs-lookup"><span data-stu-id="3bf82-128">An Azure Stack Hub operator must deploy the App Service, create plans and offers, create a tenant subscription, and add the Windows Server 2016 image.</span></span> <span data-ttu-id="3bf82-129">如果已具备这些组件，请在开始此解决方案之前确保它们符合要求。</span><span class="sxs-lookup"><span data-stu-id="3bf82-129">If you already have these components, make sure they meet the requirements before you start this  solution.</span></span>

<span data-ttu-id="3bf82-130">此解决方案示例假设你对 Azure 和 Azure Stack Hub 有一些基本的了解。</span><span class="sxs-lookup"><span data-stu-id="3bf82-130">This solution example assumes that you have some basic knowledge of Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3bf82-131">若要在开始此解决方案之前了解更多信息，请阅读以下文章：</span><span class="sxs-lookup"><span data-stu-id="3bf82-131">To learn more before starting the solution, read the following articles:</span></span>

- [<span data-ttu-id="3bf82-132">Azure 简介</span><span class="sxs-lookup"><span data-stu-id="3bf82-132">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="3bf82-133">Azure Stack Hub 的重要概念</span><span class="sxs-lookup"><span data-stu-id="3bf82-133">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a><span data-ttu-id="3bf82-134">开始之前</span><span class="sxs-lookup"><span data-stu-id="3bf82-134">Before you begin</span></span>

<span data-ttu-id="3bf82-135">在开始配置混合云连接之前，请确保满足以下条件：</span><span class="sxs-lookup"><span data-stu-id="3bf82-135">Verify that you meet the following criteria before you start configuring hybrid cloud connectivity:</span></span>

- <span data-ttu-id="3bf82-136">需要一个用于 VPN 设备的面向外部的公共 IPv4 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-136">You need an externally facing public IPv4 address for your VPN device.</span></span> <span data-ttu-id="3bf82-137">此 IP 地址不得位于 NAT（网络地址转换）之后。</span><span class="sxs-lookup"><span data-stu-id="3bf82-137">This IP address can't be located behind a NAT (Network Address Translation).</span></span>
- <span data-ttu-id="3bf82-138">所有资源都部署在同一区域/位置。</span><span class="sxs-lookup"><span data-stu-id="3bf82-138">All resources are deployed in the same region/location.</span></span>

#### <a name="solution-example-values"></a><span data-ttu-id="3bf82-139">解决方案示例值</span><span class="sxs-lookup"><span data-stu-id="3bf82-139">Solution example values</span></span>

<span data-ttu-id="3bf82-140">此解决方案中的示例使用以下值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-140">The examples in this solution use the following values.</span></span> <span data-ttu-id="3bf82-141">可使用这些值创建测试环境，或根据这些值来更好地理解示例。</span><span class="sxs-lookup"><span data-stu-id="3bf82-141">You can use these values to create a test environment or refer to them for a better understanding of the examples.</span></span> <span data-ttu-id="3bf82-142">有关 VPN 网关设置的详细信息，请参阅[关于 Vpn 网关设置](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-142">For more information about VPN gateway settings, see [About VPN Gateway Settings](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).</span></span>

<span data-ttu-id="3bf82-143">连接规范：</span><span class="sxs-lookup"><span data-stu-id="3bf82-143">Connection specifications:</span></span>

- <span data-ttu-id="3bf82-144">**VPN 类型**：基于路由</span><span class="sxs-lookup"><span data-stu-id="3bf82-144">**VPN type**: route-based</span></span>
- <span data-ttu-id="3bf82-145">**连接类型**：站点到站点（IPsec）</span><span class="sxs-lookup"><span data-stu-id="3bf82-145">**Connection type**: site-to-site (IPsec)</span></span>
- <span data-ttu-id="3bf82-146">**网关类型**： VPN</span><span class="sxs-lookup"><span data-stu-id="3bf82-146">**Gateway type**: VPN</span></span>
- <span data-ttu-id="3bf82-147">**Azure 连接名称**： azure Test-azurestack-S2SGateway （门户将自动填充此值）</span><span class="sxs-lookup"><span data-stu-id="3bf82-147">**Azure connection name**: Azure-Gateway-AzureStack-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="3bf82-148">**Azure Stack 中心连接名称**： Test-azurestack-S2SGateway （门户会自动填充此值）</span><span class="sxs-lookup"><span data-stu-id="3bf82-148">**Azure Stack Hub connection name**: AzureStack-Gateway-Azure-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="3bf82-149">**共享密钥**：任何与 VPN 硬件兼容且与连接两端的值匹配的共享密钥</span><span class="sxs-lookup"><span data-stu-id="3bf82-149">**Shared key**: any compatible with VPN hardware, with matching values on both sides of connection</span></span>
- <span data-ttu-id="3bf82-150">**订阅**：任何首选订阅</span><span class="sxs-lookup"><span data-stu-id="3bf82-150">**Subscription**: any preferred subscription</span></span>
- <span data-ttu-id="3bf82-151">**资源组**：Test-Infra</span><span class="sxs-lookup"><span data-stu-id="3bf82-151">**Resource group**: Test-Infra</span></span>

<span data-ttu-id="3bf82-152">网络和子网 IP 地址：</span><span class="sxs-lookup"><span data-stu-id="3bf82-152">Network and subnet IP addresses:</span></span>

| <span data-ttu-id="3bf82-153">Azure/Azure Stack Hub 连接</span><span class="sxs-lookup"><span data-stu-id="3bf82-153">Azure/Azure Stack Hub Connection</span></span> | <span data-ttu-id="3bf82-154">名称</span><span class="sxs-lookup"><span data-stu-id="3bf82-154">Name</span></span> | <span data-ttu-id="3bf82-155">子网</span><span class="sxs-lookup"><span data-stu-id="3bf82-155">Subnet</span></span> | <span data-ttu-id="3bf82-156">IP 地址</span><span class="sxs-lookup"><span data-stu-id="3bf82-156">IP Address</span></span> |
|---|---|---|---|
| <span data-ttu-id="3bf82-157">Azure vNet</span><span class="sxs-lookup"><span data-stu-id="3bf82-157">Azure vNet</span></span> | <span data-ttu-id="3bf82-158">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="3bf82-158">ApplicationvNet</span></span><br><span data-ttu-id="3bf82-159">10.100.102.9/23</span><span class="sxs-lookup"><span data-stu-id="3bf82-159">10.100.102.9/23</span></span> | <span data-ttu-id="3bf82-160">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="3bf82-160">ApplicationSubnet</span></span><br><span data-ttu-id="3bf82-161">10.100.102.0/24</span><span class="sxs-lookup"><span data-stu-id="3bf82-161">10.100.102.0/24</span></span> |  |
|  |  | <span data-ttu-id="3bf82-162">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="3bf82-162">GatewaySubnet</span></span><br><span data-ttu-id="3bf82-163">10.100.103.0/24</span><span class="sxs-lookup"><span data-stu-id="3bf82-163">10.100.103.0/24</span></span> |  |
| <span data-ttu-id="3bf82-164">Azure Stack Hub vNet</span><span class="sxs-lookup"><span data-stu-id="3bf82-164">Azure Stack Hub vNet</span></span> | <span data-ttu-id="3bf82-165">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="3bf82-165">ApplicationvNet</span></span><br><span data-ttu-id="3bf82-166">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="3bf82-166">10.100.100.0/23</span></span> | <span data-ttu-id="3bf82-167">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="3bf82-167">ApplicationSubnet</span></span> <br><span data-ttu-id="3bf82-168">10.100.100.0/24</span><span class="sxs-lookup"><span data-stu-id="3bf82-168">10.100.100.0/24</span></span> |  |
|  |  | <span data-ttu-id="3bf82-169">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="3bf82-169">GatewaySubnet</span></span> <br><span data-ttu-id="3bf82-170">10.100101.0/24</span><span class="sxs-lookup"><span data-stu-id="3bf82-170">10.100101.0/24</span></span> |  |
| <span data-ttu-id="3bf82-171">Azure 虚拟网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-171">Azure Virtual Network Gateway</span></span> | <span data-ttu-id="3bf82-172">Azure-Gateway</span><span class="sxs-lookup"><span data-stu-id="3bf82-172">Azure-Gateway</span></span> |  |  |
| <span data-ttu-id="3bf82-173">Azure Stack Hub 虚拟网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-173">Azure Stack Hub Virtual Network Gateway</span></span> | <span data-ttu-id="3bf82-174">AzureStack-Gateway</span><span class="sxs-lookup"><span data-stu-id="3bf82-174">AzureStack-Gateway</span></span> |  |  |
| <span data-ttu-id="3bf82-175">Azure 公共 IP</span><span class="sxs-lookup"><span data-stu-id="3bf82-175">Azure Public IP</span></span> | <span data-ttu-id="3bf82-176">Azure-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="3bf82-176">Azure-GatewayPublicIP</span></span> |  | <span data-ttu-id="3bf82-177">在创建时确定</span><span class="sxs-lookup"><span data-stu-id="3bf82-177">Determined at creation</span></span> |
| <span data-ttu-id="3bf82-178">Azure Stack Hub 公共 IP</span><span class="sxs-lookup"><span data-stu-id="3bf82-178">Azure Stack Hub Public IP</span></span> | <span data-ttu-id="3bf82-179">AzureStack-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="3bf82-179">AzureStack-GatewayPublicIP</span></span> |  | <span data-ttu-id="3bf82-180">在创建时确定</span><span class="sxs-lookup"><span data-stu-id="3bf82-180">Determined at creation</span></span> |
| <span data-ttu-id="3bf82-181">Azure 本地网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-181">Azure Local Network Gateway</span></span> | <span data-ttu-id="3bf82-182">AzureStack-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="3bf82-182">AzureStack-S2SGateway</span></span><br>   <span data-ttu-id="3bf82-183">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="3bf82-183">10.100.100.0/23</span></span> |  | <span data-ttu-id="3bf82-184">Azure Stack Hub 公共 IP 值</span><span class="sxs-lookup"><span data-stu-id="3bf82-184">Azure Stack Hub Public IP Value</span></span> |
| <span data-ttu-id="3bf82-185">Azure Stack Hub 本地网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-185">Azure Stack Hub Local Network Gateway</span></span> | <span data-ttu-id="3bf82-186">Azure-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="3bf82-186">Azure-S2SGateway</span></span><br><span data-ttu-id="3bf82-187">10.100.102.0/23</span><span class="sxs-lookup"><span data-stu-id="3bf82-187">10.100.102.0/23</span></span> |  | <span data-ttu-id="3bf82-188">Azure 公共 IP 值</span><span class="sxs-lookup"><span data-stu-id="3bf82-188">Azure Public IP Value</span></span> |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a><span data-ttu-id="3bf82-189">在公有云 Azure 和 Azure Stack Hub 中创建虚拟网络</span><span class="sxs-lookup"><span data-stu-id="3bf82-189">Create a virtual network in global Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3bf82-190">请执行以下步骤，以便通过门户创建虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="3bf82-190">Use the following steps to create a virtual network by using the portal.</span></span> <span data-ttu-id="3bf82-191">如果只是将本文用作解决方案，则可使用这些[示例值](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-191">You can use these [example values](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) if you're using this article as only a  solution.</span></span> <span data-ttu-id="3bf82-192">如果根据本文来配置生产型环境，请将示例设置替换为你自己的值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-192">If you're using this article to configure a production environment, replace the example settings with  your own values.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3bf82-193">必须确保在 Azure 或 Azure Stack Hub vNet 地址空间中没有 IP 地址重叠现象。</span><span class="sxs-lookup"><span data-stu-id="3bf82-193">You must ensure that there isn't an overlap of IP addresses in Azure or Azure Stack Hub vNet address spaces.</span></span>

<span data-ttu-id="3bf82-194">若要在 Azure 中创建 VNet，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="3bf82-194">To create a vNet in Azure:</span></span>

1. <span data-ttu-id="3bf82-195">使用浏览器连接到 [Azure 门户](https://portal.azure.com/)并使用 Azure 帐户登录。</span><span class="sxs-lookup"><span data-stu-id="3bf82-195">Use your browser to connect to the [Azure portal](https://portal.azure.com/) and sign in with your Azure account.</span></span>
2. <span data-ttu-id="3bf82-196">选择“创建资源”。</span><span class="sxs-lookup"><span data-stu-id="3bf82-196">Select **Create a resource**.</span></span> <span data-ttu-id="3bf82-197">在“搜索市场”\*\*\*\* 字段中，输入“虚拟网络”。</span><span class="sxs-lookup"><span data-stu-id="3bf82-197">In the **Search the marketplace** field, enter 'virtual network'.</span></span> <span data-ttu-id="3bf82-198">从结果中选择“虚拟网络”。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-198">Select **Virtual network** from the results.</span></span>
3. <span data-ttu-id="3bf82-199">从“选择部署模型”\*\*\*\* 列表中选择“资源管理器”\*\*\*\*，然后选择“创建”\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="3bf82-199">From the **Select a deployment model** list, select **Resource Manager**, and then select **Create**.</span></span>
4. <span data-ttu-id="3bf82-200">在“创建虚拟网络”中\*\*\*\* 配置 VNet 设置。</span><span class="sxs-lookup"><span data-stu-id="3bf82-200">On **Create virtual network**, configure the VNet settings.</span></span> <span data-ttu-id="3bf82-201">必填字段名称带有红色星号前缀。</span><span class="sxs-lookup"><span data-stu-id="3bf82-201">The required fields names are prefixed with a red asterisk.</span></span>  <span data-ttu-id="3bf82-202">输入有效值时，该星号变为绿色复选标记。</span><span class="sxs-lookup"><span data-stu-id="3bf82-202">When you enter a valid value, the asterisk changes to a green check mark.</span></span>

<span data-ttu-id="3bf82-203">若要在 Azure Stack Hub 中创建 VNet，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="3bf82-203">To create a vNet in Azure Stack Hub:</span></span>

1. <span data-ttu-id="3bf82-204">使用 Azure Stack 中心**租户门户**重复以上步骤（1-4）。</span><span class="sxs-lookup"><span data-stu-id="3bf82-204">Repeat the steps above (1-4) using the Azure Stack Hub **tenant portal**.</span></span>

## <a name="add-a-gateway-subnet"></a><span data-ttu-id="3bf82-205">添加网关子网</span><span class="sxs-lookup"><span data-stu-id="3bf82-205">Add a gateway subnet</span></span>

<span data-ttu-id="3bf82-206">将虚拟网络连接到网关之前，需创建要连接的虚拟网络的网关子网。</span><span class="sxs-lookup"><span data-stu-id="3bf82-206">Before connecting your virtual network to a gateway, you need to create the gateway subnet for the virtual network that you want to connect to.</span></span> <span data-ttu-id="3bf82-207">网关服务使用在网关子网中指定的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-207">The gateway services use the IP addresses you specify in the gateway subnet.</span></span>

<span data-ttu-id="3bf82-208">在 [Azure 门户](https://portal.azure.com/)中，导航到要在其中创建虚拟网关的资源管理器虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="3bf82-208">In the [Azure portal](https://portal.azure.com/), navigate to the Resource Manager virtual network where you want to create a virtual network gateway.</span></span>

1. <span data-ttu-id="3bf82-209">选择 vNet，以便打开“虚拟网络”页。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-209">Select the vNet to open the **Virtual network** page.</span></span>
2. <span data-ttu-id="3bf82-210">在“设置”中选择“子网”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-210">In **SETTINGS**, select **Subnets**.</span></span>
3. <span data-ttu-id="3bf82-211">在 "**子**网" 页上，选择 " **+ 网关子网**" 打开 "**添加子网**" 页。</span><span class="sxs-lookup"><span data-stu-id="3bf82-211">On the **Subnets** page, select **+Gateway subnet** to open the **Add subnet** page.</span></span>

    ![添加网关子网](media/solution-deployment-guide-connectivity/image4.png)

4. <span data-ttu-id="3bf82-213">子网的“名称”\*\*\*\* 自动填充为值“GatewaySubnet”。</span><span class="sxs-lookup"><span data-stu-id="3bf82-213">The **Name** for the subnet is automatically filled in with the value 'GatewaySubnet'.</span></span> <span data-ttu-id="3bf82-214">Azure 需要此值才能识别作为网关子网的子网。</span><span class="sxs-lookup"><span data-stu-id="3bf82-214">This value is required for Azure to recognize the subnet as the gateway subnet.</span></span>
5. <span data-ttu-id="3bf82-215">更改提供的用于匹配配置要求的**地址范围**值，然后选择 **"确定"**。</span><span class="sxs-lookup"><span data-stu-id="3bf82-215">Change the **Address range** values that are provided to match your configuration requirements and then select **OK**.</span></span>

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a><span data-ttu-id="3bf82-216">在 Azure 和 Azure Stack 中创建虚拟网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-216">Create a Virtual Network Gateway in Azure and Azure Stack</span></span>

<span data-ttu-id="3bf82-217">请执行以下步骤，以便在 Azure 中创建虚拟网关。</span><span class="sxs-lookup"><span data-stu-id="3bf82-217">Use the following steps to create a virtual network gateway in Azure.</span></span>

1. <span data-ttu-id="3bf82-218">在门户页的左侧，选择 **+** ""，并在搜索字段中输入 "虚拟网络网关"。</span><span class="sxs-lookup"><span data-stu-id="3bf82-218">On the left side of the portal page, select **+** and enter 'virtual network gateway' in the search field.</span></span>
2. <span data-ttu-id="3bf82-219">在“结果”中选择“虚拟网关”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-219">In **Results**, select **Virtual network gateway**.</span></span>
3. <span data-ttu-id="3bf82-220">在“虚拟网关”中选择“创建”，打开“创建虚拟网关”页。\*\*\*\*\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-220">In **Virtual network gateway**, select **Create** to open the **Create virtual network gateway** page.</span></span>
4. <span data-ttu-id="3bf82-221">在 "**创建虚拟网络网关**" 上，使用**教程示例值**指定网络网关的值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-221">On **Create virtual network gateway**, specify the values for your network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="3bf82-222">包括以下其他值：</span><span class="sxs-lookup"><span data-stu-id="3bf82-222">Include the following additional values:</span></span>

   - <span data-ttu-id="3bf82-223">**SKU**：基本</span><span class="sxs-lookup"><span data-stu-id="3bf82-223">**SKU**: basic</span></span>
   - <span data-ttu-id="3bf82-224">**虚拟网络**：选择前面创建的虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="3bf82-224">**Virtual Network**: Select the virtual network you created earlier.</span></span> <span data-ttu-id="3bf82-225">已创建的网关子网会自动选中。</span><span class="sxs-lookup"><span data-stu-id="3bf82-225">The gateway subnet you created is automatically selected.</span></span>
   - <span data-ttu-id="3bf82-226">**第一个 IP 配置**：网关的公共 IP。</span><span class="sxs-lookup"><span data-stu-id="3bf82-226">**First IP Configuration**:  The public IP of your gateway.</span></span>
     - <span data-ttu-id="3bf82-227">选择“创建网关 IP 配置”，然后就会转到“选择公共 IP 地址”页。  </span><span class="sxs-lookup"><span data-stu-id="3bf82-227">Select **Create gateway IP configuration**, which takes you to the **Choose public IP address** page.</span></span>
     - <span data-ttu-id="3bf82-228">选择“+新建”，打开“创建公共 IP 地址”页   。</span><span class="sxs-lookup"><span data-stu-id="3bf82-228">Select **+Create new** to open the **Create public IP address** page.</span></span>
     - <span data-ttu-id="3bf82-229">输入公共 IP 地址的“名称”。 </span><span class="sxs-lookup"><span data-stu-id="3bf82-229">Enter a **Name** for your public IP address.</span></span> <span data-ttu-id="3bf82-230">将“SKU”保留为“基本”，然后选择“确定”，以便保存所做的更改。  </span><span class="sxs-lookup"><span data-stu-id="3bf82-230">Leave the SKU as **Basic**, and then select **OK** to save your changes.</span></span>

       > [!Note]
       > <span data-ttu-id="3bf82-231">目前，VPN 网关仅支持动态公共 IP 地址分配。</span><span class="sxs-lookup"><span data-stu-id="3bf82-231">Currently, VPN Gateway only supports Dynamic Public IP address allocation.</span></span> <span data-ttu-id="3bf82-232">但是，这并不意味着 IP 地址在分配到 VPN 网关后会更改。</span><span class="sxs-lookup"><span data-stu-id="3bf82-232">However, this doesn't mean that the IP address changes after it's assigned to your VPN gateway.</span></span> <span data-ttu-id="3bf82-233">公共 IP 地址只在删除或重新创建网关时会更改。</span><span class="sxs-lookup"><span data-stu-id="3bf82-233">The only time the public IP address changes is when the gateway is deleted and re-created.</span></span> <span data-ttu-id="3bf82-234">VPN 网关进行大小调整、重置或其他内部维护/升级不会更改 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-234">Resizing, resetting, or other internal maintenance/upgrades to your VPN gateway don't change the IP address.</span></span>

5. <span data-ttu-id="3bf82-235">验证网关设置。</span><span class="sxs-lookup"><span data-stu-id="3bf82-235">Verify your gateway settings.</span></span>
6. <span data-ttu-id="3bf82-236">选择“创建”，创建 VPN 网关。 </span><span class="sxs-lookup"><span data-stu-id="3bf82-236">Select **Create** to create the VPN gateway.</span></span> <span data-ttu-id="3bf82-237">将会验证网关设置，并会在仪表板上显示“正在部署虚拟网关”磁贴。</span><span class="sxs-lookup"><span data-stu-id="3bf82-237">The gateway settings are validated and the "Deploying Virtual network gateway" tile is shown on your dashboard.</span></span>

   >[!Note]
   ><span data-ttu-id="3bf82-238">创建网关最多可能需要 45 分钟。</span><span class="sxs-lookup"><span data-stu-id="3bf82-238">Creating a gateway can take up to 45 minutes.</span></span> <span data-ttu-id="3bf82-239">可能需要刷新门户页才能看到完成状态。</span><span class="sxs-lookup"><span data-stu-id="3bf82-239">You may need to refresh your portal page to see the completed status.</span></span>

    <span data-ttu-id="3bf82-240">创建网关后，可以通过在门户中查看虚拟网络，来查看分配给网关的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-240">After the gateway is created, you can see the IP address assigned to it by looking at the virtual network in the portal.</span></span> <span data-ttu-id="3bf82-241">网关显示为连接的设备。</span><span class="sxs-lookup"><span data-stu-id="3bf82-241">The gateway appears as a connected device.</span></span> <span data-ttu-id="3bf82-242">若要查看网关的详细信息，请选择设备。</span><span class="sxs-lookup"><span data-stu-id="3bf82-242">To see more information about the gateway, select the device.</span></span>

7. <span data-ttu-id="3bf82-243">在 Azure Stack Hub 部署上重复前述步骤 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-243">Repeat the previous steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a><span data-ttu-id="3bf82-244">在 Azure 和 Azure Stack Hub 中创建本地网关</span><span class="sxs-lookup"><span data-stu-id="3bf82-244">Create the local network gateway in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3bf82-245">本地网络网关通常是指本地位置。</span><span class="sxs-lookup"><span data-stu-id="3bf82-245">The local network gateway typically refers to your on-premises location.</span></span> <span data-ttu-id="3bf82-246">请为站点提供一个可供 Azure 或 Azure Stack Hub 引用的名称，然后指定以下配置：</span><span class="sxs-lookup"><span data-stu-id="3bf82-246">You give the site a name that Azure or Azure Stack Hub can refer to, and then specify:</span></span>

- <span data-ttu-id="3bf82-247">要为其创建连接的本地 VPN 设备的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-247">The IP address of the on-premises VPN device that you're creating a connection for.</span></span>
- <span data-ttu-id="3bf82-248">可以通过 VPN 网关将其路由到 VPN 设备的 IP 地址前缀。</span><span class="sxs-lookup"><span data-stu-id="3bf82-248">The IP address prefixes that will be routed through the VPN gateway to the VPN device.</span></span> <span data-ttu-id="3bf82-249">指定的地址前缀是位于本地网络的前缀。</span><span class="sxs-lookup"><span data-stu-id="3bf82-249">The address prefixes you specify are the prefixes located on your on-premises network.</span></span>

  >[!Note]
  ><span data-ttu-id="3bf82-250">如果你的本地网络发生了更改，或需要更改 VPN 设备的公共 IP 地址，则可以在以后更新这些值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-250">If your on-premises network changes or you need to change the public IP address for the VPN device, you can update these values later.</span></span>

1. <span data-ttu-id="3bf82-251">在门户中选择“+创建资源”。 </span><span class="sxs-lookup"><span data-stu-id="3bf82-251">In the portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="3bf82-252">在搜索框中输入“本地网关”，然后按 **Enter** 进行搜索  。</span><span class="sxs-lookup"><span data-stu-id="3bf82-252">In the search box, enter **Local network gateway**, then select **Enter** to search.</span></span> <span data-ttu-id="3bf82-253">将会显示一个结果列表。</span><span class="sxs-lookup"><span data-stu-id="3bf82-253">A list of results will display.</span></span>
3. <span data-ttu-id="3bf82-254">选择“本地网关”，然后选择“创建”，打开“创建本地网关”页    。</span><span class="sxs-lookup"><span data-stu-id="3bf82-254">Select **Local network gateway**, then select **Create** to open the **Create local network gateway** page.</span></span>
4. <span data-ttu-id="3bf82-255">在 "**创建本地网络网关**" 上，使用**教程示例值**指定本地网络网关的值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-255">On **Create local network gateway**, specify the values for your local network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="3bf82-256">包括以下其他值：</span><span class="sxs-lookup"><span data-stu-id="3bf82-256">Include the following additional values:</span></span>

    - <span data-ttu-id="3bf82-257">**IP 地址**：需要 Azure 或 Azure Stack Hub 连接到的 VPN 设备的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-257">**IP address**: The public IP address of the VPN device that you want Azure or Azure Stack Hub to connect to.</span></span> <span data-ttu-id="3bf82-258">指定一个有效的不在 NAT 后面的公共 IP 地址，这样 Azure 就能访问该地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-258">Specify a valid public IP address that isn't behind a NAT so Azure can reach the address.</span></span> <span data-ttu-id="3bf82-259">如果目前没有 IP 地址，可以使用示例中的值作为占位符。</span><span class="sxs-lookup"><span data-stu-id="3bf82-259">If you don't have the IP address right now, you can use a value from the example as a placeholder.</span></span> <span data-ttu-id="3bf82-260">需要返回并将占位符替换为 VPN 设备的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-260">You'll have to go back and replace the placeholder with the public IP address of your VPN device.</span></span> <span data-ttu-id="3bf82-261">除非提供有效的地址，否则 Azure 无法连接到设备。</span><span class="sxs-lookup"><span data-stu-id="3bf82-261">Azure can't connect to the device until you provide a valid address.</span></span>
    - <span data-ttu-id="3bf82-262">**地址空间**：此本地网络所代表的网络的地址范围。</span><span class="sxs-lookup"><span data-stu-id="3bf82-262">**Address Space**: the address range for the network that this local network represents.</span></span> <span data-ttu-id="3bf82-263">可以添加多个地址空间范围。</span><span class="sxs-lookup"><span data-stu-id="3bf82-263">You can add multiple address space ranges.</span></span> <span data-ttu-id="3bf82-264">请确保指定的范围没有与要连接到的其他网络的范围重叠。</span><span class="sxs-lookup"><span data-stu-id="3bf82-264">Make sure that the ranges you specify don't overlap with ranges of other networks that you want to connect to.</span></span> <span data-ttu-id="3bf82-265">Azure 会将指定的地址范围路由到本地 VPN 设备 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-265">Azure will route the address range that you specify to the on-premises VPN device IP address.</span></span> <span data-ttu-id="3bf82-266">如果需要连接到本地站点，请使用自己的值而不是示例值。</span><span class="sxs-lookup"><span data-stu-id="3bf82-266">Use your own values if you want to connect to your on-premises site, not an example value.</span></span>
    - <span data-ttu-id="3bf82-267">**配置 bgp 设置**：仅在配置 bgp 时使用。</span><span class="sxs-lookup"><span data-stu-id="3bf82-267">**Configure BGP settings**: Use only when configuring BGP.</span></span> <span data-ttu-id="3bf82-268">否则，不选择此项。</span><span class="sxs-lookup"><span data-stu-id="3bf82-268">Otherwise, don't select this option.</span></span>
    - <span data-ttu-id="3bf82-269">**订阅**：验证是否显示了正确的订阅。</span><span class="sxs-lookup"><span data-stu-id="3bf82-269">**Subscription**: Verify that the correct subscription is showing.</span></span>
    - <span data-ttu-id="3bf82-270">**资源组**：选择要使用的资源组。</span><span class="sxs-lookup"><span data-stu-id="3bf82-270">**Resource Group**: Select the resource group that you want to use.</span></span> <span data-ttu-id="3bf82-271">你可以创建新的资源组或选择已创建的资源组。</span><span class="sxs-lookup"><span data-stu-id="3bf82-271">You can either create a new resource group or select one that you've already created.</span></span>
    - <span data-ttu-id="3bf82-272">**位置**：选择将在其中创建此对象的位置。</span><span class="sxs-lookup"><span data-stu-id="3bf82-272">**Location**: Select the location that this object will be created in.</span></span> <span data-ttu-id="3bf82-273">可选择 VNet 所在的位置，但这不是必须的。</span><span class="sxs-lookup"><span data-stu-id="3bf82-273">You may want to select the same location that your VNet resides in, but you're not required to do so.</span></span>
5. <span data-ttu-id="3bf82-274">将所需值指定完以后，选择“创建”即可创建本地网关\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="3bf82-274">When you finish specifying the required values, select **Create** to create the local network gateway.</span></span>
6. <span data-ttu-id="3bf82-275">在 Azure Stack Hub 部署上重复这些步骤 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-275">Repeat these steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="configure-your-connection"></a><span data-ttu-id="3bf82-276">配置连接</span><span class="sxs-lookup"><span data-stu-id="3bf82-276">Configure your connection</span></span>

<span data-ttu-id="3bf82-277">通过站点到站点连接连接到本地网络需要 VPN 设备。</span><span class="sxs-lookup"><span data-stu-id="3bf82-277">Site-to-site connections to an on-premises network require a VPN device.</span></span> <span data-ttu-id="3bf82-278">你配置的 VPN 设备称为连接。</span><span class="sxs-lookup"><span data-stu-id="3bf82-278">The VPN device you configure is referred to as a connection.</span></span> <span data-ttu-id="3bf82-279">若要配置连接，需要：</span><span class="sxs-lookup"><span data-stu-id="3bf82-279">To configure your connection, you need:</span></span>

- <span data-ttu-id="3bf82-280">共享密钥。</span><span class="sxs-lookup"><span data-stu-id="3bf82-280">A shared key.</span></span> <span data-ttu-id="3bf82-281">此密钥就是在创建站点到站点 VPN 连接时指定的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="3bf82-281">This key is the same shared key that you specify when creating your site-to-site VPN connection.</span></span> <span data-ttu-id="3bf82-282">在示例中，我们使用基本的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="3bf82-282">In our examples, we use a basic shared key.</span></span> <span data-ttu-id="3bf82-283">建议生成更复杂的可用密钥。</span><span class="sxs-lookup"><span data-stu-id="3bf82-283">We recommend that you generate a more complex key to use.</span></span>
- <span data-ttu-id="3bf82-284">虚拟网关的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-284">The public IP address of your virtual network gateway.</span></span> <span data-ttu-id="3bf82-285">可以通过 Azure 门户、PowerShell 或 CLI 查看公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="3bf82-285">You can view the public IP address by using the Azure portal, PowerShell, or CLI.</span></span> <span data-ttu-id="3bf82-286">若要使用 Azure 门户查找 VPN 网关的公共 IP 地址，请在 "虚拟网络网关" 中，选择网关的名称。</span><span class="sxs-lookup"><span data-stu-id="3bf82-286">To find the public IP address of your VPN gateway using the Azure portal, go to virtual network gateways, then select the name of your gateway.</span></span>

<span data-ttu-id="3bf82-287">执行以下步骤，在虚拟网关和本地 VPN 设备之间创建站点到站点 VPN 连接。</span><span class="sxs-lookup"><span data-stu-id="3bf82-287">Use the following steps to create a site-to-site VPN connection between your virtual network gateway and your on-premises VPN device.</span></span>

1. <span data-ttu-id="3bf82-288">在 Azure 门户中，选择 " **+ 创建资源**"。</span><span class="sxs-lookup"><span data-stu-id="3bf82-288">In the Azure portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="3bf82-289">搜索**连接**。</span><span class="sxs-lookup"><span data-stu-id="3bf82-289">Search for **connections**.</span></span>
3. <span data-ttu-id="3bf82-290">在“结果”中选择“连接”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-290">In **Results**, select **Connections**.</span></span>
4. <span data-ttu-id="3bf82-291">在“连接”上选择“创建”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-291">On **Connection**, select **Create**.</span></span>
5. <span data-ttu-id="3bf82-292">在“创建连接”\*\*\*\* 上配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="3bf82-292">On **Create Connection**, configure the following settings:</span></span>

    - <span data-ttu-id="3bf82-293">**连接类型**：选择 "站点到站点（IPSec）"。</span><span class="sxs-lookup"><span data-stu-id="3bf82-293">**Connection type**: Select site-to-site (IPSec).</span></span>
    - <span data-ttu-id="3bf82-294">**资源组**：选择测试资源组。</span><span class="sxs-lookup"><span data-stu-id="3bf82-294">**Resource Group**: Select your test resource group.</span></span>
    - <span data-ttu-id="3bf82-295">**虚拟网络网关**：选择创建的虚拟网络网关。</span><span class="sxs-lookup"><span data-stu-id="3bf82-295">**Virtual Network Gateway**: Select the virtual network gateway you created.</span></span>
    - <span data-ttu-id="3bf82-296">**本地网络网关**：选择创建的本地网络网关。</span><span class="sxs-lookup"><span data-stu-id="3bf82-296">**Local Network Gateway**: Select the local network gateway you created.</span></span>
    - <span data-ttu-id="3bf82-297">**连接名称**：使用两个网关上的值会自动填充此名称。</span><span class="sxs-lookup"><span data-stu-id="3bf82-297">**Connection Name**: This name is autopopulated using the values from the two gateways.</span></span>
    - <span data-ttu-id="3bf82-298">**共享密钥**：此值必须与用于本地 VPN 设备的值匹配。</span><span class="sxs-lookup"><span data-stu-id="3bf82-298">**Shared Key**: This value must match the value that you're using for your local on-premises VPN device.</span></span> <span data-ttu-id="3bf82-299">教程示例使用“abc123”，但应使用更复杂的。</span><span class="sxs-lookup"><span data-stu-id="3bf82-299">The tutorial example uses 'abc123', but you should use something more complex.</span></span> <span data-ttu-id="3bf82-300">重要的是，此值*必须*与配置 VPN 设备时指定的值相同。</span><span class="sxs-lookup"><span data-stu-id="3bf82-300">The important thing is that this value *must* be the same value that you specify when configuring your VPN device.</span></span>
    - <span data-ttu-id="3bf82-301">“订阅”、“资源组”和“位置”值是固定的\*\*\*\*\*\*\*\*\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="3bf82-301">The values for **Subscription**, **Resource Group**, and **Location** are fixed.</span></span>

6. <span data-ttu-id="3bf82-302">选择“确定”以创建连接\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="3bf82-302">Select **OK** to create your connection.</span></span>

<span data-ttu-id="3bf82-303">可在虚拟网关的“连接”页中查看连接。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="3bf82-303">You can see the connection in the **Connections** page of the virtual network gateway.</span></span> <span data-ttu-id="3bf82-304">状态将从*未知*到*连接*，然后才会*成功*。</span><span class="sxs-lookup"><span data-stu-id="3bf82-304">The status will go from *Unknown* to *Connecting*, and then to *Succeeded*.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3bf82-305">后续步骤</span><span class="sxs-lookup"><span data-stu-id="3bf82-305">Next steps</span></span>

- <span data-ttu-id="3bf82-306">若要详细了解 Azure 云模式，请参阅[云设计模式](https://docs.microsoft.com/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="3bf82-306">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
