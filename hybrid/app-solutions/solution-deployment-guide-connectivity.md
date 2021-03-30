---
title: 在 Azure 和 Azure Stack Hub 中配置混合云连接
description: 了解如何使用 Azure 和 Azure Stack Hub 配置混合云连接。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4480f51b03082f2a0cbb7f2f213e05b7bf488646
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895366"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a><span data-ttu-id="fd10d-103">使用 Azure 和 Azure Stack Hub 配置混合云连接</span><span class="sxs-lookup"><span data-stu-id="fd10d-103">Configure hybrid cloud connectivity using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd10d-104">你可以使用混合连接模式在全局 Azure 和 Azure Stack Hub 中安全访问资源。</span><span class="sxs-lookup"><span data-stu-id="fd10d-104">You can access resources with security in global Azure and Azure Stack Hub using the hybrid connectivity pattern.</span></span>

<span data-ttu-id="fd10d-105">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="fd10d-105">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fd10d-106">将数据保留在本地以满足隐私或法规要求，但保持对全局 Azure 资源的访问权限。</span><span class="sxs-lookup"><span data-stu-id="fd10d-106">Keep data on-premises to meet privacy or regulatory requirements but keep access to global Azure resources.</span></span>
> - <span data-ttu-id="fd10d-107">在全局 Azure 中使用云缩放的应用部署和资源时，维护旧系统。</span><span class="sxs-lookup"><span data-stu-id="fd10d-107">Maintain a legacy system while using cloud-scaled app deployment and resources in global Azure.</span></span>

> [!Tip]  
> <span data-ttu-id="fd10d-108">![混合支柱图](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fd10d-108">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fd10d-109">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="fd10d-109">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fd10d-110">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="fd10d-110">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fd10d-111">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="fd10d-111">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fd10d-112">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="fd10d-112">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="fd10d-113">先决条件</span><span class="sxs-lookup"><span data-stu-id="fd10d-113">Prerequisites</span></span>

<span data-ttu-id="fd10d-114">生成混合连接部署需要一些组件。</span><span class="sxs-lookup"><span data-stu-id="fd10d-114">A few components are required to build a hybrid connectivity deployment.</span></span> <span data-ttu-id="fd10d-115">这其中的某些组件需要时间来准备，因此请进行相应的计划。</span><span class="sxs-lookup"><span data-stu-id="fd10d-115">Some of these components take time to prepare, so plan accordingly.</span></span>

### <a name="azure"></a><span data-ttu-id="fd10d-116">Azure</span><span class="sxs-lookup"><span data-stu-id="fd10d-116">Azure</span></span>

- <span data-ttu-id="fd10d-117">如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-117">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>
- <span data-ttu-id="fd10d-118">在 Azure 中创建 [Web 应用](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-118">Create a [web app](/aspnet/core/tutorials/publish-to-azure-webapp-using-vs) in Azure.</span></span> <span data-ttu-id="fd10d-119">请记下 Web 应用 URL，因为在解决方案中将会用到它。</span><span class="sxs-lookup"><span data-stu-id="fd10d-119">Make note of the web app URL because you'll need it in the solution.</span></span>

### <a name="azure-stack-hub"></a><span data-ttu-id="fd10d-120">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="fd10d-120">Azure Stack Hub</span></span>

<span data-ttu-id="fd10d-121">Azure OEM/硬件合作伙伴可以部署生产 Azure Stack Hub，并且所有用户都可以部署 Azure Stack 开发工具包 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-121">An Azure OEM/hardware partner can deploy a production Azure Stack Hub, and all users can deploy an Azure Stack Development Kit (ASDK).</span></span>

- <span data-ttu-id="fd10d-122">使用生产 Azure Stack Hub 或部署 ASDK。</span><span class="sxs-lookup"><span data-stu-id="fd10d-122">Use your production Azure Stack Hub or deploy the ASDK.</span></span>
   >[!Note]
   ><span data-ttu-id="fd10d-123">部署 ASDK 最多可能需要 7 个小时，因此请制定相应的计划。</span><span class="sxs-lookup"><span data-stu-id="fd10d-123">Deploying the ASDK can take up to 7 hours, so plan accordingly.</span></span>

- <span data-ttu-id="fd10d-124">将[应用服务](/azure-stack/operator/azure-stack-app-service-deploy) PaaS 服务部署到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="fd10d-124">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="fd10d-125">在 Azure Stack Hub 环境中[创建计划和套餐](/azure-stack/operator/service-plan-offer-subscription-overview)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-125">[Create plans and offers](/azure-stack/operator/service-plan-offer-subscription-overview) in the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="fd10d-126">在 Azure Stack Hub 环境中[创建租户订阅](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-126">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>

### <a name="azure-stack-hub-components"></a><span data-ttu-id="fd10d-127">Azure Stack Hub 组件</span><span class="sxs-lookup"><span data-stu-id="fd10d-127">Azure Stack Hub components</span></span>

<span data-ttu-id="fd10d-128">Azure Stack Hub 操作员必须部署应用服务、创建计划和套餐、创建租户订阅，并添加 Windows Server 2016 映像。</span><span class="sxs-lookup"><span data-stu-id="fd10d-128">An Azure Stack Hub operator must deploy the App Service, create plans and offers, create a tenant subscription, and add the Windows Server 2016 image.</span></span> <span data-ttu-id="fd10d-129">如果已具有这些组件，则在开始此解决方案之前，请先确保它们符合要求。</span><span class="sxs-lookup"><span data-stu-id="fd10d-129">If you already have these components, make sure they meet the requirements before you start this  solution.</span></span>

<span data-ttu-id="fd10d-130">此解决方案示例假设你了解 Azure 和 Azure Stack Hub 的一些基本知识。</span><span class="sxs-lookup"><span data-stu-id="fd10d-130">This solution example assumes that you have some basic knowledge of Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fd10d-131">若要在开始此解决方案之前了解详细信息，请阅读以下文章：</span><span class="sxs-lookup"><span data-stu-id="fd10d-131">To learn more before starting the solution, read the following articles:</span></span>

- [<span data-ttu-id="fd10d-132">Azure 简介</span><span class="sxs-lookup"><span data-stu-id="fd10d-132">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="fd10d-133">Azure Stack Hub 关键概念</span><span class="sxs-lookup"><span data-stu-id="fd10d-133">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

### <a name="before-you-begin"></a><span data-ttu-id="fd10d-134">开始之前</span><span class="sxs-lookup"><span data-stu-id="fd10d-134">Before you begin</span></span>

<span data-ttu-id="fd10d-135">在开始配置混合云连接之前，请先验证是否满足以下条件：</span><span class="sxs-lookup"><span data-stu-id="fd10d-135">Verify that you meet the following criteria before you start configuring hybrid cloud connectivity:</span></span>

- <span data-ttu-id="fd10d-136">需要一个用于 VPN 设备的面向外部的公共 IPv4 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-136">You need an externally facing public IPv4 address for your VPN device.</span></span> <span data-ttu-id="fd10d-137">此 IP 地址不能位于 NAT（网络地址转换）后面。</span><span class="sxs-lookup"><span data-stu-id="fd10d-137">This IP address can't be located behind a NAT (Network Address Translation).</span></span>
- <span data-ttu-id="fd10d-138">所有资源都部署在同一区域/位置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-138">All resources are deployed in the same region/location.</span></span>

#### <a name="solution-example-values"></a><span data-ttu-id="fd10d-139">解决方案示例值</span><span class="sxs-lookup"><span data-stu-id="fd10d-139">Solution example values</span></span>

<span data-ttu-id="fd10d-140">此解决方案中的示例使用以下值。</span><span class="sxs-lookup"><span data-stu-id="fd10d-140">The examples in this solution use the following values.</span></span> <span data-ttu-id="fd10d-141">可使用这些值创建测试环境，或参考这些值以更好地理解示例。</span><span class="sxs-lookup"><span data-stu-id="fd10d-141">You can use these values to create a test environment or refer to them for a better understanding of the examples.</span></span> <span data-ttu-id="fd10d-142">有关 VPN 网关设置的详细信息，请参阅[关于 VPN 网关设置](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-142">For more information about VPN gateway settings, see [About VPN Gateway Settings](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).</span></span>

<span data-ttu-id="fd10d-143">连接规范：</span><span class="sxs-lookup"><span data-stu-id="fd10d-143">Connection specifications:</span></span>

- <span data-ttu-id="fd10d-144">**VPN 类型**：基于路由</span><span class="sxs-lookup"><span data-stu-id="fd10d-144">**VPN type**: route-based</span></span>
- <span data-ttu-id="fd10d-145">**连接类型**：站点到站点 (IPsec)</span><span class="sxs-lookup"><span data-stu-id="fd10d-145">**Connection type**: site-to-site (IPsec)</span></span>
- <span data-ttu-id="fd10d-146">**网关类型**：VPN</span><span class="sxs-lookup"><span data-stu-id="fd10d-146">**Gateway type**: VPN</span></span>
- <span data-ttu-id="fd10d-147">**Azure 连接名称**：Azure-Gateway-AzureStack-S2SGateway（门户将自动填充此值）</span><span class="sxs-lookup"><span data-stu-id="fd10d-147">**Azure connection name**: Azure-Gateway-AzureStack-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="fd10d-148">**Azure Stack Hub 连接名称**：AzureStack-Gateway-Azure-S2SGateway（门户将自动填充此值）</span><span class="sxs-lookup"><span data-stu-id="fd10d-148">**Azure Stack Hub connection name**: AzureStack-Gateway-Azure-S2SGateway (the portal will autofill this value)</span></span>
- <span data-ttu-id="fd10d-149">**共享密钥**：与 VPN 硬件兼容的任何密钥，在连接两端都具有匹配值</span><span class="sxs-lookup"><span data-stu-id="fd10d-149">**Shared key**: any compatible with VPN hardware, with matching values on both sides of connection</span></span>
- <span data-ttu-id="fd10d-150">**订阅**：任何首选订阅</span><span class="sxs-lookup"><span data-stu-id="fd10d-150">**Subscription**: any preferred subscription</span></span>
- <span data-ttu-id="fd10d-151">资源组：Test-Infra</span><span class="sxs-lookup"><span data-stu-id="fd10d-151">**Resource group**: Test-Infra</span></span>

<span data-ttu-id="fd10d-152">网络和子网 IP 地址：</span><span class="sxs-lookup"><span data-stu-id="fd10d-152">Network and subnet IP addresses:</span></span>

| <span data-ttu-id="fd10d-153">Azure/Azure Stack Hub 连接</span><span class="sxs-lookup"><span data-stu-id="fd10d-153">Azure/Azure Stack Hub Connection</span></span> | <span data-ttu-id="fd10d-154">名称</span><span class="sxs-lookup"><span data-stu-id="fd10d-154">Name</span></span> | <span data-ttu-id="fd10d-155">子网</span><span class="sxs-lookup"><span data-stu-id="fd10d-155">Subnet</span></span> | <span data-ttu-id="fd10d-156">IP 地址</span><span class="sxs-lookup"><span data-stu-id="fd10d-156">IP Address</span></span> |
|---|---|---|---|
| <span data-ttu-id="fd10d-157">Azure vNet</span><span class="sxs-lookup"><span data-stu-id="fd10d-157">Azure vNet</span></span> | <span data-ttu-id="fd10d-158">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="fd10d-158">ApplicationvNet</span></span><br><span data-ttu-id="fd10d-159">10.100.102.9/23</span><span class="sxs-lookup"><span data-stu-id="fd10d-159">10.100.102.9/23</span></span> | <span data-ttu-id="fd10d-160">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="fd10d-160">ApplicationSubnet</span></span><br><span data-ttu-id="fd10d-161">10.100.102.0/24</span><span class="sxs-lookup"><span data-stu-id="fd10d-161">10.100.102.0/24</span></span> |  |
|  |  | <span data-ttu-id="fd10d-162">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="fd10d-162">GatewaySubnet</span></span><br><span data-ttu-id="fd10d-163">10.100.103.0/24</span><span class="sxs-lookup"><span data-stu-id="fd10d-163">10.100.103.0/24</span></span> |  |
| <span data-ttu-id="fd10d-164">Azure Stack Hub vNet</span><span class="sxs-lookup"><span data-stu-id="fd10d-164">Azure Stack Hub vNet</span></span> | <span data-ttu-id="fd10d-165">ApplicationvNet</span><span class="sxs-lookup"><span data-stu-id="fd10d-165">ApplicationvNet</span></span><br><span data-ttu-id="fd10d-166">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="fd10d-166">10.100.100.0/23</span></span> | <span data-ttu-id="fd10d-167">ApplicationSubnet</span><span class="sxs-lookup"><span data-stu-id="fd10d-167">ApplicationSubnet</span></span> <br><span data-ttu-id="fd10d-168">10.100.100.0/24</span><span class="sxs-lookup"><span data-stu-id="fd10d-168">10.100.100.0/24</span></span> |  |
|  |  | <span data-ttu-id="fd10d-169">GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="fd10d-169">GatewaySubnet</span></span> <br><span data-ttu-id="fd10d-170">10.100101.0/24</span><span class="sxs-lookup"><span data-stu-id="fd10d-170">10.100101.0/24</span></span> |  |
| <span data-ttu-id="fd10d-171">Azure 虚拟网络网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-171">Azure Virtual Network Gateway</span></span> | <span data-ttu-id="fd10d-172">Azure-Gateway</span><span class="sxs-lookup"><span data-stu-id="fd10d-172">Azure-Gateway</span></span> |  |  |
| <span data-ttu-id="fd10d-173">Azure Stack Hub 虚拟网络网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-173">Azure Stack Hub Virtual Network Gateway</span></span> | <span data-ttu-id="fd10d-174">AzureStack-Gateway</span><span class="sxs-lookup"><span data-stu-id="fd10d-174">AzureStack-Gateway</span></span> |  |  |
| <span data-ttu-id="fd10d-175">Azure 公共 IP</span><span class="sxs-lookup"><span data-stu-id="fd10d-175">Azure Public IP</span></span> | <span data-ttu-id="fd10d-176">Azure-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="fd10d-176">Azure-GatewayPublicIP</span></span> |  | <span data-ttu-id="fd10d-177">在创建时确定</span><span class="sxs-lookup"><span data-stu-id="fd10d-177">Determined at creation</span></span> |
| <span data-ttu-id="fd10d-178">Azure Stack Hub 公共 IP</span><span class="sxs-lookup"><span data-stu-id="fd10d-178">Azure Stack Hub Public IP</span></span> | <span data-ttu-id="fd10d-179">AzureStack-GatewayPublicIP</span><span class="sxs-lookup"><span data-stu-id="fd10d-179">AzureStack-GatewayPublicIP</span></span> |  | <span data-ttu-id="fd10d-180">在创建时确定</span><span class="sxs-lookup"><span data-stu-id="fd10d-180">Determined at creation</span></span> |
| <span data-ttu-id="fd10d-181">Azure 本地网络网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-181">Azure Local Network Gateway</span></span> | <span data-ttu-id="fd10d-182">AzureStack-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="fd10d-182">AzureStack-S2SGateway</span></span><br>   <span data-ttu-id="fd10d-183">10.100.100.0/23</span><span class="sxs-lookup"><span data-stu-id="fd10d-183">10.100.100.0/23</span></span> |  | <span data-ttu-id="fd10d-184">Azure Stack Hub 公共 IP 值</span><span class="sxs-lookup"><span data-stu-id="fd10d-184">Azure Stack Hub Public IP Value</span></span> |
| <span data-ttu-id="fd10d-185">Azure Stack Hub 本地网络网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-185">Azure Stack Hub Local Network Gateway</span></span> | <span data-ttu-id="fd10d-186">Azure-S2SGateway</span><span class="sxs-lookup"><span data-stu-id="fd10d-186">Azure-S2SGateway</span></span><br><span data-ttu-id="fd10d-187">10.100.102.0/23</span><span class="sxs-lookup"><span data-stu-id="fd10d-187">10.100.102.0/23</span></span> |  | <span data-ttu-id="fd10d-188">Azure 公共 IP 值</span><span class="sxs-lookup"><span data-stu-id="fd10d-188">Azure Public IP Value</span></span> |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a><span data-ttu-id="fd10d-189">在全局 Azure 和 Azure Stack Hub 中创建虚拟网络</span><span class="sxs-lookup"><span data-stu-id="fd10d-189">Create a virtual network in global Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd10d-190">按照以下步骤使用门户创建虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="fd10d-190">Use the following steps to create a virtual network by using the portal.</span></span> <span data-ttu-id="fd10d-191">如果仅将本文用作解决方案，则可以使用这些[示例值](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-191">You can use these [example values](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) if you're using this article as only a  solution.</span></span> <span data-ttu-id="fd10d-192">如果使用本文来配置生产环境，请使用自己的值替换示例设置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-192">If you're using this article to configure a production environment, replace the example settings with  your own values.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="fd10d-193">必须确保 Azure 或 Azure Stack Hub vNet 地址空间中的 IP 地址不重叠。</span><span class="sxs-lookup"><span data-stu-id="fd10d-193">You must ensure that there isn't an overlap of IP addresses in Azure or Azure Stack Hub vNet address spaces.</span></span>

<span data-ttu-id="fd10d-194">若要在 Azure 中创建 vNet，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="fd10d-194">To create a vNet in Azure:</span></span>

1. <span data-ttu-id="fd10d-195">使用浏览器连接到 [Azure 门户](https://portal.azure.com/)并使用 Azure 帐户登录。</span><span class="sxs-lookup"><span data-stu-id="fd10d-195">Use your browser to connect to the [Azure portal](https://portal.azure.com/) and sign in with your Azure account.</span></span>
2. <span data-ttu-id="fd10d-196">选择“创建资源”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-196">Select **Create a resource**.</span></span> <span data-ttu-id="fd10d-197">在“在市场中搜索”字段中，输入“虚拟网络”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-197">In the **Search the marketplace** field, enter 'virtual network'.</span></span> <span data-ttu-id="fd10d-198">从结果中选择“虚拟网络”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-198">Select **Virtual network** from the results.</span></span>
3. <span data-ttu-id="fd10d-199">在“选择部署模型”列表中，选择“资源管理器”，然后选择“创建”  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-199">From the **Select a deployment model** list, select **Resource Manager**, and then select **Create**.</span></span>
4. <span data-ttu-id="fd10d-200">在“创建虚拟网络”上，配置 VNet 设置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-200">On **Create virtual network**, configure the VNet settings.</span></span> <span data-ttu-id="fd10d-201">必填字段名称以红色星号作为前缀。</span><span class="sxs-lookup"><span data-stu-id="fd10d-201">The required fields names are prefixed with a red asterisk.</span></span>  <span data-ttu-id="fd10d-202">输入有效值时，星号变为绿色选中标记。</span><span class="sxs-lookup"><span data-stu-id="fd10d-202">When you enter a valid value, the asterisk changes to a green check mark.</span></span>

<span data-ttu-id="fd10d-203">若要在 Azure Stack Hub 中创建 vNet，请执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="fd10d-203">To create a vNet in Azure Stack Hub:</span></span>

1. <span data-ttu-id="fd10d-204">使用 Azure Stack Hub 租户门户重复上述步骤 (1-4)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-204">Repeat the steps above (1-4) using the Azure Stack Hub **tenant portal**.</span></span>

## <a name="add-a-gateway-subnet"></a><span data-ttu-id="fd10d-205">添加网关子网</span><span class="sxs-lookup"><span data-stu-id="fd10d-205">Add a gateway subnet</span></span>

<span data-ttu-id="fd10d-206">将虚拟网络连接到网关之前，需创建要连接的虚拟网络的网关子网。</span><span class="sxs-lookup"><span data-stu-id="fd10d-206">Before connecting your virtual network to a gateway, you need to create the gateway subnet for the virtual network that you want to connect to.</span></span> <span data-ttu-id="fd10d-207">网关服务使用网关子网中指定的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-207">The gateway services use the IP addresses you specify in the gateway subnet.</span></span>

<span data-ttu-id="fd10d-208">在 [Azure 门户](https://portal.azure.com/)中，导航到要在其中创建虚拟网络网关的资源管理器虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="fd10d-208">In the [Azure portal](https://portal.azure.com/), navigate to the Resource Manager virtual network where you want to create a virtual network gateway.</span></span>

1. <span data-ttu-id="fd10d-209">选择该 vNet 打开“虚拟网络”页。</span><span class="sxs-lookup"><span data-stu-id="fd10d-209">Select the vNet to open the **Virtual network** page.</span></span>
2. <span data-ttu-id="fd10d-210">在“设置”中选择“子网” 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-210">In **SETTINGS**, select **Subnets**.</span></span>
3. <span data-ttu-id="fd10d-211">在“子网”页中，选择“+网关子网”打开“添加子网”页  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-211">On the **Subnets** page, select **+Gateway subnet** to open the **Add subnet** page.</span></span>

    ![添加网关子网](media/solution-deployment-guide-connectivity/image4.png)

4. <span data-ttu-id="fd10d-213">子网的“名称”自动填充值“GatewaySubnet”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-213">The **Name** for the subnet is automatically filled in with the value 'GatewaySubnet'.</span></span> <span data-ttu-id="fd10d-214">Azure 需要此值才能识别作为网关子网的子网。</span><span class="sxs-lookup"><span data-stu-id="fd10d-214">This value is required for Azure to recognize the subnet as the gateway subnet.</span></span>
5. <span data-ttu-id="fd10d-215">更改提供的“地址范围”值以匹配配置要求，然后选择“确定” 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-215">Change the **Address range** values that are provided to match your configuration requirements and then select **OK**.</span></span>

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a><span data-ttu-id="fd10d-216">在 Azure 和 Azure Stack 中创建虚拟网络网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-216">Create a Virtual Network Gateway in Azure and Azure Stack</span></span>

<span data-ttu-id="fd10d-217">按照以下步骤在 Azure 中创建虚拟网络网关。</span><span class="sxs-lookup"><span data-stu-id="fd10d-217">Use the following steps to create a virtual network gateway in Azure.</span></span>

1. <span data-ttu-id="fd10d-218">在门户页左侧选择 +，并在搜索字段中输入“虚拟网络网关”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-218">On the left side of the portal page, select **+** and enter 'virtual network gateway' in the search field.</span></span>
2. <span data-ttu-id="fd10d-219">在“结果”中，选择“虚拟网络网关” 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-219">In **Results**, select **Virtual network gateway**.</span></span>
3. <span data-ttu-id="fd10d-220">在“虚拟网络网关”中，选择“创建”以打开“创建虚拟网络网关”页  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-220">In **Virtual network gateway**, select **Create** to open the **Create virtual network gateway** page.</span></span>
4. <span data-ttu-id="fd10d-221">在“创建虚拟网络网关”上，使用教程示例值指定网络网关的值 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-221">On **Create virtual network gateway**, specify the values for your network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="fd10d-222">包括以下其他值：</span><span class="sxs-lookup"><span data-stu-id="fd10d-222">Include the following additional values:</span></span>

   - <span data-ttu-id="fd10d-223">**SKU**：基本</span><span class="sxs-lookup"><span data-stu-id="fd10d-223">**SKU**: basic</span></span>
   - <span data-ttu-id="fd10d-224">**虚拟网络**：选择之前创建的虚拟网络。</span><span class="sxs-lookup"><span data-stu-id="fd10d-224">**Virtual Network**: Select the virtual network you created earlier.</span></span> <span data-ttu-id="fd10d-225">已创建的网关子网会自动选中。</span><span class="sxs-lookup"><span data-stu-id="fd10d-225">The gateway subnet you created is automatically selected.</span></span>
   - <span data-ttu-id="fd10d-226">**第一个 IP 配置**：网关的公共 IP。</span><span class="sxs-lookup"><span data-stu-id="fd10d-226">**First IP Configuration**:  The public IP of your gateway.</span></span>
     - <span data-ttu-id="fd10d-227">选择“创建网关 IP 配置”，然后就会转到“选择公共 IP 地址”页。</span><span class="sxs-lookup"><span data-stu-id="fd10d-227">Select **Create gateway IP configuration**, which takes you to the **Choose public IP address** page.</span></span>
     - <span data-ttu-id="fd10d-228">选择“+新建”，打开“创建公共 IP 地址”页   。</span><span class="sxs-lookup"><span data-stu-id="fd10d-228">Select **+Create new** to open the **Create public IP address** page.</span></span>
     - <span data-ttu-id="fd10d-229">输入公共 IP 地址的“名称”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-229">Enter a **Name** for your public IP address.</span></span> <span data-ttu-id="fd10d-230">将“SKU”保留为“基本”，然后选择“确定”，以便保存所做的更改。</span><span class="sxs-lookup"><span data-stu-id="fd10d-230">Leave the SKU as **Basic**, and then select **OK** to save your changes.</span></span>

       > [!Note]
       > <span data-ttu-id="fd10d-231">目前，VPN 网关仅支持动态公共 IP 地址分配。</span><span class="sxs-lookup"><span data-stu-id="fd10d-231">Currently, VPN Gateway only supports Dynamic Public IP address allocation.</span></span> <span data-ttu-id="fd10d-232">但是，这并不意味着 IP 地址在分配到 VPN 网关后会更改。</span><span class="sxs-lookup"><span data-stu-id="fd10d-232">However, this doesn't mean that the IP address changes after it's assigned to your VPN gateway.</span></span> <span data-ttu-id="fd10d-233">公共 IP 地址只在删除或重新创建网关时会更改。</span><span class="sxs-lookup"><span data-stu-id="fd10d-233">The only time the public IP address changes is when the gateway is deleted and re-created.</span></span> <span data-ttu-id="fd10d-234">VPN 网关进行大小调整、重置或其他内部维护/升级不会更改 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-234">Resizing, resetting, or other internal maintenance/upgrades to your VPN gateway don't change the IP address.</span></span>

5. <span data-ttu-id="fd10d-235">验证网关设置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-235">Verify your gateway settings.</span></span>
6. <span data-ttu-id="fd10d-236">选择“创建”，创建 VPN 网关。</span><span class="sxs-lookup"><span data-stu-id="fd10d-236">Select **Create** to create the VPN gateway.</span></span> <span data-ttu-id="fd10d-237">将会验证网关设置，并会在仪表板上显示“正在部署虚拟网关”磁贴。</span><span class="sxs-lookup"><span data-stu-id="fd10d-237">The gateway settings are validated and the "Deploying Virtual network gateway" tile is shown on your dashboard.</span></span>

   >[!Note]
   ><span data-ttu-id="fd10d-238">创建网关最多可能需要 45 分钟。</span><span class="sxs-lookup"><span data-stu-id="fd10d-238">Creating a gateway can take up to 45 minutes.</span></span> <span data-ttu-id="fd10d-239">可能需要刷新门户页才能看到完成状态。</span><span class="sxs-lookup"><span data-stu-id="fd10d-239">You may need to refresh your portal page to see the completed status.</span></span>

    <span data-ttu-id="fd10d-240">创建网关后，可以通过在门户中查看虚拟网络，来查看分配给网关的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-240">After the gateway is created, you can see the IP address assigned to it by looking at the virtual network in the portal.</span></span> <span data-ttu-id="fd10d-241">网关显示为连接的设备。</span><span class="sxs-lookup"><span data-stu-id="fd10d-241">The gateway appears as a connected device.</span></span> <span data-ttu-id="fd10d-242">若要查看网关的详细信息，请选择设备。</span><span class="sxs-lookup"><span data-stu-id="fd10d-242">To see more information about the gateway, select the device.</span></span>

7. <span data-ttu-id="fd10d-243">在 Azure Stack Hub 部署上重复前述步骤 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-243">Repeat the previous steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a><span data-ttu-id="fd10d-244">在 Azure 和 Azure Stack Hub 中创建本地网关</span><span class="sxs-lookup"><span data-stu-id="fd10d-244">Create the local network gateway in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fd10d-245">本地网络网关通常是指本地位置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-245">The local network gateway typically refers to your on-premises location.</span></span> <span data-ttu-id="fd10d-246">请为站点提供一个可供 Azure 或 Azure Stack Hub 引用的名称，然后指定以下配置：</span><span class="sxs-lookup"><span data-stu-id="fd10d-246">You give the site a name that Azure or Azure Stack Hub can refer to, and then specify:</span></span>

- <span data-ttu-id="fd10d-247">要为其创建连接的本地 VPN 设备的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-247">The IP address of the on-premises VPN device that you're creating a connection for.</span></span>
- <span data-ttu-id="fd10d-248">可以通过 VPN 网关将其路由到 VPN 设备的 IP 地址前缀。</span><span class="sxs-lookup"><span data-stu-id="fd10d-248">The IP address prefixes that will be routed through the VPN gateway to the VPN device.</span></span> <span data-ttu-id="fd10d-249">指定的地址前缀是位于本地网络的前缀。</span><span class="sxs-lookup"><span data-stu-id="fd10d-249">The address prefixes you specify are the prefixes located on your on-premises network.</span></span>

  >[!Note]
  ><span data-ttu-id="fd10d-250">如果之后本地网络发生了更改，或需要更改 VPN 设备的公共 IP 地址，可更新这些值。</span><span class="sxs-lookup"><span data-stu-id="fd10d-250">If your on-premises network changes or you need to change the public IP address for the VPN device, you can update these values later.</span></span>

1. <span data-ttu-id="fd10d-251">在门户中选择“+创建资源”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-251">In the portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="fd10d-252">在搜索框中输入“本地网关”，然后按 **Enter** 进行搜索  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-252">In the search box, enter **Local network gateway**, then select **Enter** to search.</span></span> <span data-ttu-id="fd10d-253">将会显示一个结果列表。</span><span class="sxs-lookup"><span data-stu-id="fd10d-253">A list of results will display.</span></span>
3. <span data-ttu-id="fd10d-254">选择“本地网关”，然后选择“创建”，打开“创建本地网关”页    。</span><span class="sxs-lookup"><span data-stu-id="fd10d-254">Select **Local network gateway**, then select **Create** to open the **Create local network gateway** page.</span></span>
4. <span data-ttu-id="fd10d-255">在“创建本地网络网关”上，使用教程示例值指定本地网络网关的值 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-255">On **Create local network gateway**, specify the values for your local network gateway using our **Tutorial example values**.</span></span> <span data-ttu-id="fd10d-256">包括以下其他值：</span><span class="sxs-lookup"><span data-stu-id="fd10d-256">Include the following additional values:</span></span>

    - <span data-ttu-id="fd10d-257">**IP 地址**：需要 Azure 或 Azure Stack Hub 连接到的 VPN 设备的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-257">**IP address**: The public IP address of the VPN device that you want Azure or Azure Stack Hub to connect to.</span></span> <span data-ttu-id="fd10d-258">指定一个有效的不在 NAT 后面的公共 IP 地址，这样 Azure 就能访问该地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-258">Specify a valid public IP address that isn't behind a NAT so Azure can reach the address.</span></span> <span data-ttu-id="fd10d-259">如果目前没有 IP 地址，可以使用示例中的值作为占位符。</span><span class="sxs-lookup"><span data-stu-id="fd10d-259">If you don't have the IP address right now, you can use a value from the example as a placeholder.</span></span> <span data-ttu-id="fd10d-260">你需要返回并将占位符替换为 VPN 设备的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-260">You'll have to go back and replace the placeholder with the public IP address of your VPN device.</span></span> <span data-ttu-id="fd10d-261">在提供有效地址之前，Azure 无法连接到设备。</span><span class="sxs-lookup"><span data-stu-id="fd10d-261">Azure can't connect to the device until you provide a valid address.</span></span>
    - <span data-ttu-id="fd10d-262">**地址空间**：此本地网络所代表的网络的地址范围。</span><span class="sxs-lookup"><span data-stu-id="fd10d-262">**Address Space**: the address range for the network that this local network represents.</span></span> <span data-ttu-id="fd10d-263">可以添加多个地址空间范围。</span><span class="sxs-lookup"><span data-stu-id="fd10d-263">You can add multiple address space ranges.</span></span> <span data-ttu-id="fd10d-264">请确保所指定的范围没有与要连接到的其他网络的范围相重叠。</span><span class="sxs-lookup"><span data-stu-id="fd10d-264">Make sure that the ranges you specify don't overlap with ranges of other networks that you want to connect to.</span></span> <span data-ttu-id="fd10d-265">Azure 会将指定的地址范围路由到本地 VPN 设备 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-265">Azure will route the address range that you specify to the on-premises VPN device IP address.</span></span> <span data-ttu-id="fd10d-266">如果需要连接到本地站点，请使用自己的值，而不是示例值。</span><span class="sxs-lookup"><span data-stu-id="fd10d-266">Use your own values if you want to connect to your on-premises site, not an example value.</span></span>
    - <span data-ttu-id="fd10d-267">**配置 BGP 设置**：仅在配置 BGP 时使用。</span><span class="sxs-lookup"><span data-stu-id="fd10d-267">**Configure BGP settings**: Use only when configuring BGP.</span></span> <span data-ttu-id="fd10d-268">否则，请不要选择此选项。</span><span class="sxs-lookup"><span data-stu-id="fd10d-268">Otherwise, don't select this option.</span></span>
    - <span data-ttu-id="fd10d-269">订阅：确保显示正确的订阅。</span><span class="sxs-lookup"><span data-stu-id="fd10d-269">**Subscription**: Verify that the correct subscription is showing.</span></span>
    - <span data-ttu-id="fd10d-270">**资源组**：选择要使用的资源组。</span><span class="sxs-lookup"><span data-stu-id="fd10d-270">**Resource Group**: Select the resource group that you want to use.</span></span> <span data-ttu-id="fd10d-271">可以创建新的资源组或选择已创建的资源组。</span><span class="sxs-lookup"><span data-stu-id="fd10d-271">You can either create a new resource group or select one that you've already created.</span></span>
    - <span data-ttu-id="fd10d-272">位置：选择将在其中创建此对象的位置。</span><span class="sxs-lookup"><span data-stu-id="fd10d-272">**Location**: Select the location that this object will be created in.</span></span> <span data-ttu-id="fd10d-273">可选择 VNet 所在的位置，但这不是必须的。</span><span class="sxs-lookup"><span data-stu-id="fd10d-273">You may want to select the same location that your VNet resides in, but you're not required to do so.</span></span>
5. <span data-ttu-id="fd10d-274">完成指定所需的值后，选择“创建”以创建本地网络网关。</span><span class="sxs-lookup"><span data-stu-id="fd10d-274">When you finish specifying the required values, select **Create** to create the local network gateway.</span></span>
6. <span data-ttu-id="fd10d-275">在 Azure Stack Hub 部署上重复上述步骤 (1-5)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-275">Repeat these steps (1-5) on your Azure Stack Hub deployment.</span></span>

## <a name="configure-your-connection"></a><span data-ttu-id="fd10d-276">配置连接</span><span class="sxs-lookup"><span data-stu-id="fd10d-276">Configure your connection</span></span>

<span data-ttu-id="fd10d-277">通过站点到站点连接连接到本地网络需要 VPN 设备。</span><span class="sxs-lookup"><span data-stu-id="fd10d-277">Site-to-site connections to an on-premises network require a VPN device.</span></span> <span data-ttu-id="fd10d-278">所配置的 VPN 设备称为连接。</span><span class="sxs-lookup"><span data-stu-id="fd10d-278">The VPN device you configure is referred to as a connection.</span></span> <span data-ttu-id="fd10d-279">若要配置连接，需要：</span><span class="sxs-lookup"><span data-stu-id="fd10d-279">To configure your connection, you need:</span></span>

- <span data-ttu-id="fd10d-280">共享密钥。</span><span class="sxs-lookup"><span data-stu-id="fd10d-280">A shared key.</span></span> <span data-ttu-id="fd10d-281">此密钥就是在创建站点到站点 VPN 连接时指定的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="fd10d-281">This key is the same shared key that you specify when creating your site-to-site VPN connection.</span></span> <span data-ttu-id="fd10d-282">在示例中，我们使用基本的共享密钥。</span><span class="sxs-lookup"><span data-stu-id="fd10d-282">In our examples, we use a basic shared key.</span></span> <span data-ttu-id="fd10d-283">建议生成更复杂的可用密钥。</span><span class="sxs-lookup"><span data-stu-id="fd10d-283">We recommend that you generate a more complex key to use.</span></span>
- <span data-ttu-id="fd10d-284">虚拟网络网关的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-284">The public IP address of your virtual network gateway.</span></span> <span data-ttu-id="fd10d-285">可以通过 Azure 门户、PowerShell 或 CLI 查看公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="fd10d-285">You can view the public IP address by using the Azure portal, PowerShell, or CLI.</span></span> <span data-ttu-id="fd10d-286">要使用 Azure 门户查找 VPN 网关的公共 IP 地址，请转到虚拟网络网关，然后选择网关的名称。</span><span class="sxs-lookup"><span data-stu-id="fd10d-286">To find the public IP address of your VPN gateway using the Azure portal, go to virtual network gateways, then select the name of your gateway.</span></span>

<span data-ttu-id="fd10d-287">按照以下步骤在虚拟网络网关和本地 VPN 设备之间创建站点到站点 VPN 连接。</span><span class="sxs-lookup"><span data-stu-id="fd10d-287">Use the following steps to create a site-to-site VPN connection between your virtual network gateway and your on-premises VPN device.</span></span>

1. <span data-ttu-id="fd10d-288">在 Azure 门户中，选择“+创建资源”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-288">In the Azure portal, select **+Create a resource**.</span></span>
2. <span data-ttu-id="fd10d-289">搜索“连接”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-289">Search for **connections**.</span></span>
3. <span data-ttu-id="fd10d-290">在“结果”中，选择“连接” 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-290">In **Results**, select **Connections**.</span></span>
4. <span data-ttu-id="fd10d-291">在“连接”上，选择“创建” 。</span><span class="sxs-lookup"><span data-stu-id="fd10d-291">On **Connection**, select **Create**.</span></span>
5. <span data-ttu-id="fd10d-292">在“创建连接”上，配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="fd10d-292">On **Create Connection**, configure the following settings:</span></span>

    - <span data-ttu-id="fd10d-293">**连接类型**：选择“站点到站点(IPsec)”。</span><span class="sxs-lookup"><span data-stu-id="fd10d-293">**Connection type**: Select site-to-site (IPSec).</span></span>
    - <span data-ttu-id="fd10d-294">**资源组**：选择测试资源组。</span><span class="sxs-lookup"><span data-stu-id="fd10d-294">**Resource Group**: Select your test resource group.</span></span>
    - <span data-ttu-id="fd10d-295">**虚拟网络网关**：选择所创建的虚拟网络网关。</span><span class="sxs-lookup"><span data-stu-id="fd10d-295">**Virtual Network Gateway**: Select the virtual network gateway you created.</span></span>
    - <span data-ttu-id="fd10d-296">**本地网络网关**：选择所创建的本地网络网关。</span><span class="sxs-lookup"><span data-stu-id="fd10d-296">**Local Network Gateway**: Select the local network gateway you created.</span></span>
    - <span data-ttu-id="fd10d-297">**连接名称**：使用来自两个网关的值自动填充此名称。</span><span class="sxs-lookup"><span data-stu-id="fd10d-297">**Connection Name**: This name is autopopulated using the values from the two gateways.</span></span>
    - <span data-ttu-id="fd10d-298">**共享密钥**：此值必须与用于本地 VPN 设备的值匹配。</span><span class="sxs-lookup"><span data-stu-id="fd10d-298">**Shared Key**: This value must match the value that you're using for your local on-premises VPN device.</span></span> <span data-ttu-id="fd10d-299">此教程示例使用“abc123”，但应该使用更复杂的密钥。</span><span class="sxs-lookup"><span data-stu-id="fd10d-299">The tutorial example uses 'abc123', but you should use something more complex.</span></span> <span data-ttu-id="fd10d-300">重要的是，此值必须与配置 VPN 设备时指定的值相同。</span><span class="sxs-lookup"><span data-stu-id="fd10d-300">The important thing is that this value *must* be the same value that you specify when configuring your VPN device.</span></span>
    - <span data-ttu-id="fd10d-301">“订阅”、“资源组”和“位置”值是固定的  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-301">The values for **Subscription**, **Resource Group**, and **Location** are fixed.</span></span>

6. <span data-ttu-id="fd10d-302">选择“确定”以创建连接。</span><span class="sxs-lookup"><span data-stu-id="fd10d-302">Select **OK** to create your connection.</span></span>

<span data-ttu-id="fd10d-303">可在虚拟网络网关的“连接”页中查看连接。</span><span class="sxs-lookup"><span data-stu-id="fd10d-303">You can see the connection in the **Connections** page of the virtual network gateway.</span></span> <span data-ttu-id="fd10d-304">“状态”会从“未知”转换为“正在连接”，再转换为“成功”  。</span><span class="sxs-lookup"><span data-stu-id="fd10d-304">The status will go from *Unknown* to *Connecting*, and then to *Succeeded*.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fd10d-305">后续步骤</span><span class="sxs-lookup"><span data-stu-id="fd10d-305">Next steps</span></span>

- <span data-ttu-id="fd10d-306">若要详细了解 Azure 云模式，请参阅[云设计模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="fd10d-306">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
