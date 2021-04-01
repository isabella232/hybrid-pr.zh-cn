---
title: 在 Azure 和 Azure Stack Hub 中部署跨云缩放的应用
description: 了解如何在 Azure 和 Azure Stack Hub 中部署跨云缩放的应用。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895408"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="3fac5-103">使用 Azure 和 Azure Stack Hub 部署可跨云缩放的应用</span><span class="sxs-lookup"><span data-stu-id="3fac5-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3fac5-104">了解如何创建可提供手动触发过程的跨云解决方案，以通过流量管理器使用自动缩放功能从 Azure Stack Hub 托管的 Web 应用切换到 Azure 托管的 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="3fac5-105">此过程确保云实用工具在承受负载时保持灵活性和可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="3fac5-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="3fac5-106">使用此模式时，租户可能尚未准备好在公有云中运行你的应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="3fac5-107">但是，要让企业在本地环境中保持用于处理应用需求高峰的容量，在经济上似乎不切实际。</span><span class="sxs-lookup"><span data-stu-id="3fac5-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="3fac5-108">租户可以通过其本地解决方案使用公有云的弹性。</span><span class="sxs-lookup"><span data-stu-id="3fac5-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="3fac5-109">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="3fac5-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3fac5-110">创建多节点 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="3fac5-111">配置和管理持续部署 (CD) 过程。</span><span class="sxs-lookup"><span data-stu-id="3fac5-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="3fac5-112">将 Web 应用发布到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="3fac5-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="3fac5-113">创建发布。</span><span class="sxs-lookup"><span data-stu-id="3fac5-113">Create a release.</span></span>
> - <span data-ttu-id="3fac5-114">了解如何监视和跟踪部署。</span><span class="sxs-lookup"><span data-stu-id="3fac5-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="3fac5-115">![混合支柱图](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3fac5-115">![hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3fac5-116">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="3fac5-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3fac5-117">Azure Stack Hub 将云计算的灵活性和创新性带入本地环境，并支持唯一的混合云，允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3fac5-118">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="3fac5-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3fac5-119">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="3fac5-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3fac5-120">先决条件</span><span class="sxs-lookup"><span data-stu-id="3fac5-120">Prerequisites</span></span>

- <span data-ttu-id="3fac5-121">Azure 订阅。</span><span class="sxs-lookup"><span data-stu-id="3fac5-121">Azure subscription.</span></span> <span data-ttu-id="3fac5-122">如果需要，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="3fac5-123">Azure Stack Hub 集成系统或 Azure Stack 开发工具包 (ASDK) 的部署。</span><span class="sxs-lookup"><span data-stu-id="3fac5-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="3fac5-124">有关安装 Azure Stack Hub 的说明，请参阅[安装 ASDK](/azure-stack/asdk/asdk-install)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install).</span></span>
  - <span data-ttu-id="3fac5-125">有关 ASDK 部署后自动化脚本，请参阅：[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="3fac5-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="3fac5-126">此项安装可能需要几个小时才能完成。</span><span class="sxs-lookup"><span data-stu-id="3fac5-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="3fac5-127">将[应用服务](/azure-stack/operator/azure-stack-app-service-deploy) PaaS 服务部署到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="3fac5-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="3fac5-128">在 Azure Stack Hub 环境中[创建计划/套餐](/azure-stack/operator/service-plan-offer-subscription-overview)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="3fac5-129">在 Azure Stack Hub 环境中[创建租户订阅](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="3fac5-130">在租户订阅中创建 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="3fac5-131">记下新 Web 应用的 URL，供稍后使用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="3fac5-132">在租户订阅中部署 Azure Pipelines 虚拟机 (VM)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="3fac5-133">需要装有 .NET 3.5 的 Windows Server 2016 VM。</span><span class="sxs-lookup"><span data-stu-id="3fac5-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="3fac5-134">将在 Azure Stack Hub 上的租户订阅中构建此 VM 作为专用生成代理。</span><span class="sxs-lookup"><span data-stu-id="3fac5-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="3fac5-135">Azure Stack Hub 市场中提供了[具有 SQL 2017 VM 映像的 Windows Server 2016](/azure-stack/operator/azure-stack-add-vm-image)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="3fac5-136">如果此映像不可用，请与 Azure Stack Hub 操作员协作，以确保将此映像添加到环境中。</span><span class="sxs-lookup"><span data-stu-id="3fac5-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="3fac5-137">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="3fac5-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="3fac5-138">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="3fac5-138">Scalability</span></span>

<span data-ttu-id="3fac5-139">跨云缩放的关键要素是能按需在公共和本地云基础结构之间提供即时缩放功能，证明服务可保持一致且可靠。</span><span class="sxs-lookup"><span data-stu-id="3fac5-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="3fac5-140">可用性</span><span class="sxs-lookup"><span data-stu-id="3fac5-140">Availability</span></span>

<span data-ttu-id="3fac5-141">确定通过本地硬件配置和软件部署来配置本地部署的应用，以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="3fac5-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="3fac5-142">可管理性</span><span class="sxs-lookup"><span data-stu-id="3fac5-142">Manageability</span></span>

<span data-ttu-id="3fac5-143">跨云解决方案确保在环境之间提供无缝的管理和熟悉的界面。</span><span class="sxs-lookup"><span data-stu-id="3fac5-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="3fac5-144">建议使用 PowerShell 进行跨平台管理。</span><span class="sxs-lookup"><span data-stu-id="3fac5-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="3fac5-145">跨云缩放</span><span class="sxs-lookup"><span data-stu-id="3fac5-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="3fac5-146">获取自定义域并配置 DNS</span><span class="sxs-lookup"><span data-stu-id="3fac5-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="3fac5-147">更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="3fac5-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="3fac5-148">然后，Azure AD 将会验证自定义域名的所有权。</span><span class="sxs-lookup"><span data-stu-id="3fac5-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="3fac5-149">可将 [Azure DNS](/azure/dns/dns-getstarted-portal) 用于 Azure 中的 Azure/Microsoft 365/外部 DNS 记录，或在[其他 DNS 注册机构](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)中添加 DNS 条目。</span><span class="sxs-lookup"><span data-stu-id="3fac5-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="3fac5-150">向公共注册机构注册自定义域。</span><span class="sxs-lookup"><span data-stu-id="3fac5-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="3fac5-151">登录到域的域名注册机构。</span><span class="sxs-lookup"><span data-stu-id="3fac5-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="3fac5-152">可能需要由获批准的管理员进行 DNS 更新。</span><span class="sxs-lookup"><span data-stu-id="3fac5-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="3fac5-153">通过添加 Azure AD 提供的 DNS 条目来更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="3fac5-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="3fac5-154">（DNS 条目不会影响电子邮件路由或 Web 托管行为。）</span><span class="sxs-lookup"><span data-stu-id="3fac5-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="3fac5-155">在 Azure Stack Hub 中创建默认的多节点 Web 应用</span><span class="sxs-lookup"><span data-stu-id="3fac5-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="3fac5-156">设置混合持续集成和持续部署 (CI/CD)，以将 Web 应用部署到 Azure 和 Azure Stack Hub，并自动将更改推送到这两个云中。</span><span class="sxs-lookup"><span data-stu-id="3fac5-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="3fac5-157">需要在 Azure Stack Hub 上创建适当的合成映像用于运行 Windows Server 和 SQL，并需要部署应用服务。</span><span class="sxs-lookup"><span data-stu-id="3fac5-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="3fac5-158">有关详细信息，请参阅应用服务文档[在 Azure Stack Hub 上部署应用服务的先决条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="3fac5-159">向 Azure Repos 中添加代码</span><span class="sxs-lookup"><span data-stu-id="3fac5-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="3fac5-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="3fac5-160">Azure Repos</span></span>

1. <span data-ttu-id="3fac5-161">使用在 Azure Repos 上拥有项目创建权限的帐户登录到 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="3fac5-162">混合 CI/CD 可同时应用到应用代码和基础结构代码。</span><span class="sxs-lookup"><span data-stu-id="3fac5-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="3fac5-163">使用 [Azure 资源管理器模板](https://azure.microsoft.com/resources/templates/)进行专用与托管的云开发。</span><span class="sxs-lookup"><span data-stu-id="3fac5-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![连接到 Azure Repos 上的项目](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="3fac5-165">创建并打开默认 Web 应用以 **克隆存储库**。</span><span class="sxs-lookup"><span data-stu-id="3fac5-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![在 Azure Web 应用中克隆存储库](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="3fac5-167">为这两个云中的应用服务创建独立的 Web 应用部署</span><span class="sxs-lookup"><span data-stu-id="3fac5-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="3fac5-168">编辑 **WebApplication.csproj** 文件。</span><span class="sxs-lookup"><span data-stu-id="3fac5-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="3fac5-169">选择 `Runtimeidentifier` 并添加 `win10-x64`。</span><span class="sxs-lookup"><span data-stu-id="3fac5-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="3fac5-170">（请参阅[独立部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文档。）</span><span class="sxs-lookup"><span data-stu-id="3fac5-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![编辑 Web 应用项目文件](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="3fac5-172">使用团队资源管理器将代码签入 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="3fac5-173">确认应用代码已签入到 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="3fac5-174">创建生成定义</span><span class="sxs-lookup"><span data-stu-id="3fac5-174">Create the build definition</span></span>

1. <span data-ttu-id="3fac5-175">登录到 Azure Pipelines 以确认能够创建生成定义。</span><span class="sxs-lookup"><span data-stu-id="3fac5-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="3fac5-176">添加 **-r win10-x64** 代码。</span><span class="sxs-lookup"><span data-stu-id="3fac5-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="3fac5-177">使用 .NET Core 触发独立部署时需要添加此代码。</span><span class="sxs-lookup"><span data-stu-id="3fac5-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![将代码添加到 Web 应用](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="3fac5-179">运行生成。</span><span class="sxs-lookup"><span data-stu-id="3fac5-179">Run the build.</span></span> <span data-ttu-id="3fac5-180">[独立部署生成](/dotnet/core/deploying/deploy-with-vs#simpleSelf)过程将发布可在 Azure 和 Azure Stack Hub 上运行的项目。</span><span class="sxs-lookup"><span data-stu-id="3fac5-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="3fac5-181">使用 Azure 托管代理</span><span class="sxs-lookup"><span data-stu-id="3fac5-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="3fac5-182">在 Azure Pipelines 中使用托管生成代理是生成和部署 Web 应用的便捷做法。</span><span class="sxs-lookup"><span data-stu-id="3fac5-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="3fac5-183">维护和升级由 Microsoft Azure 自动完成，从而实现了连续不断的开发周期。</span><span class="sxs-lookup"><span data-stu-id="3fac5-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="3fac5-184">管理和配置 CD 过程</span><span class="sxs-lookup"><span data-stu-id="3fac5-184">Manage and configure the CD process</span></span>

<span data-ttu-id="3fac5-185">Azure Pipelines 和 Azure DevOps Services 提供可配置度和可管理度高的管道，用于在多个环境中进行发布，例如开发、暂存、QA 和生产环境；包括在特定阶段需要审批。</span><span class="sxs-lookup"><span data-stu-id="3fac5-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="3fac5-186">创建发布定义</span><span class="sxs-lookup"><span data-stu-id="3fac5-186">Create release definition</span></span>

1. <span data-ttu-id="3fac5-187">选择“加号”按钮可在 Azure DevOps Services 的“生成和发布”部分的“发布”选项卡下添加新发布  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![创建发布定义](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="3fac5-189">应用“Azure 应用服务部署”模板。</span><span class="sxs-lookup"><span data-stu-id="3fac5-189">Apply the Azure App Service Deployment template.</span></span>

   ![应用 Azure 应用服务部署模板](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="3fac5-191">在“添加项目”下，为 Azure 云生成应用添加项目。</span><span class="sxs-lookup"><span data-stu-id="3fac5-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![向 Azure 云生成添加项目](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="3fac5-193">在“管道”选项卡下选择环境的“阶段和任务”链接，并设置 Azure 云环境值。</span><span class="sxs-lookup"><span data-stu-id="3fac5-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![设置 Azure 云环境值](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="3fac5-195">设置 **环境名称**，并选择 Azure 云终结点的 **Azure 订阅**。</span><span class="sxs-lookup"><span data-stu-id="3fac5-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![选择 Azure 云终结点的 Azure 订阅](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="3fac5-197">在“应用服务名称”下，设置所需的 Azure 应用服务名称。</span><span class="sxs-lookup"><span data-stu-id="3fac5-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![设置 Azure 应用服务名称](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="3fac5-199">在 Azure 云托管环境的“代理队列”下输入“Hosted VS2017”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![为 Azure 云托管环境设置代理队列](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="3fac5-201">在“部署 Azure 应用服务”菜单中，为环境选择有效的 **包或文件夹**。</span><span class="sxs-lookup"><span data-stu-id="3fac5-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3fac5-202">选择 **文件夹位置** 旁边的“确定”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-202">Select **OK** to **folder location**.</span></span>
  
      ![选择 Azure 应用服务环境的包或文件夹](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![文件夹选取器对话框 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="3fac5-205">保存所有更改并返回 **发布管道**。</span><span class="sxs-lookup"><span data-stu-id="3fac5-205">Save all changes and go back to **release pipeline**.</span></span>

    ![在发布管道中保存更改](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="3fac5-207">选择 Azure Stack Hub 应用的生成以添加新项目。</span><span class="sxs-lookup"><span data-stu-id="3fac5-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![为 Azure Stack Hub 应用添加新项目](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="3fac5-209">通过应用 Azure 应用服务部署额外添加一个环境。</span><span class="sxs-lookup"><span data-stu-id="3fac5-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![将环境添加到 Azure 应用服务部署](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="3fac5-211">将新环境命名为“Azure Stack”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-211">Name the new environment "Azure Stack".</span></span>

    ![为 Azure 应用服务部署中的环境命名](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="3fac5-213">在“任务”选项卡下找到 Azure Stack 环境。</span><span class="sxs-lookup"><span data-stu-id="3fac5-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack 环境](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="3fac5-215">选择 Azure Stack 终结点的订阅。</span><span class="sxs-lookup"><span data-stu-id="3fac5-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![选择 Azure Stack 终结点的订阅](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="3fac5-217">将 Azure Stack Web 应用名称设置为应用服务名称。</span><span class="sxs-lookup"><span data-stu-id="3fac5-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="3fac5-218">![设置 Azure Stack Web 应用名称](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="3fac5-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="3fac5-219">选择“Azure Stack 代理”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-219">Select the Azure Stack agent.</span></span>

    ![选择“Azure Stack 代理”](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="3fac5-221">在“部署 Azure 应用服务”部分下，为环境选择有效的 **包或文件夹**。</span><span class="sxs-lookup"><span data-stu-id="3fac5-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="3fac5-222">选择文件夹位置旁边的“确定”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-222">Select **OK** to folder location.</span></span>

    ![选择 Azure 应用服务部署的文件夹](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![文件夹选取器对话框 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="3fac5-225">在“变量”选项卡下添加名为 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的变量，将其值设置为 **true**，将范围设置为 Azure Stack。</span><span class="sxs-lookup"><span data-stu-id="3fac5-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![将变量添加到 Azure 应用部署](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="3fac5-227">选择两个项目中的“持续”部署触发器图标，并启用“持续”部署触发器。</span><span class="sxs-lookup"><span data-stu-id="3fac5-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![选择持续部署触发器](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="3fac5-229">选择 Azure Stack 环境中的“部署前”条件图标，并将触发器设置为“发布后”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![选择部署前的条件](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="3fac5-231">保存所有更改。</span><span class="sxs-lookup"><span data-stu-id="3fac5-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="3fac5-232">任务的某些设置可能已在从模板创建发布定义时自动定义为[环境变量](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="3fac5-233">无法在任务设置中修改这些设置；必须选择父环境项才能编辑这些设置。</span><span class="sxs-lookup"><span data-stu-id="3fac5-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="3fac5-234">通过 Visual Studio 发布到 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3fac5-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="3fac5-235">通过创建终结点，Azure DevOps Services 生成可以将 Azure 服务应用部署到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="3fac5-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="3fac5-236">Azure Pipelines 会连接到生成代理，而后者会连接到 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="3fac5-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="3fac5-237">登录到 Azure DevOps Services 并转到“应用设置”页。</span><span class="sxs-lookup"><span data-stu-id="3fac5-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="3fac5-238">在“设置”中，选择“安全性”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="3fac5-239">在“VSTS 组”中，选择“终结点创建者”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="3fac5-240">在“成员”选项卡上，选择“添加”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="3fac5-241">在“添加用户和组”中输入用户名，然后从用户列表中选择该用户。</span><span class="sxs-lookup"><span data-stu-id="3fac5-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="3fac5-242">选择“保存更改”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="3fac5-243">在“VSTS 组”列表中，选择“终结点管理员”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="3fac5-244">在“成员”选项卡上，选择“添加”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="3fac5-245">在“添加用户和组”中输入用户名，然后从用户列表中选择该用户。</span><span class="sxs-lookup"><span data-stu-id="3fac5-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="3fac5-246">选择“保存更改”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-246">Select **Save changes**.</span></span>

<span data-ttu-id="3fac5-247">获取终结点信息后，可以使用 Azure Pipelines 到 Azure Stack Hub 的连接。</span><span class="sxs-lookup"><span data-stu-id="3fac5-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="3fac5-248">Azure Stack Hub 中的生成代理从 Azure Pipelines 获取指令，然后该代理会传达终结点信息以与 Azure Stack Hub 进行通信。</span><span class="sxs-lookup"><span data-stu-id="3fac5-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="3fac5-249">开发应用生成</span><span class="sxs-lookup"><span data-stu-id="3fac5-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="3fac5-250">需要在 Azure Stack Hub 上创建适当的合成映像用于运行 Windows Server 和 SQL，并需要部署应用服务。</span><span class="sxs-lookup"><span data-stu-id="3fac5-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="3fac5-251">有关详细信息，请参阅[在 Azure Stack Hub 上部署应用服务的先决条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

<span data-ttu-id="3fac5-252">使用 [Azure 资源管理器模板](https://azure.microsoft.com/resources/templates/)（例如 Azure Repos 中的 Web 应用代码）将内容部署到这两个云。</span><span class="sxs-lookup"><span data-stu-id="3fac5-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="3fac5-253">向 Azure Repos 项目添加代码</span><span class="sxs-lookup"><span data-stu-id="3fac5-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="3fac5-254">使用在 Azure Stack Hub 上拥有项目创建权限的帐户登录到 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="3fac5-255">通过创建默认 Web 应用并将其打开，克隆存储库。</span><span class="sxs-lookup"><span data-stu-id="3fac5-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="3fac5-256">为两个云中的应用服务创建独立 Web 应用部署</span><span class="sxs-lookup"><span data-stu-id="3fac5-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="3fac5-257">编辑 WebApplication.csproj 文件：选择 `Runtimeidentifier`，然后添加 `win10-x64`。</span><span class="sxs-lookup"><span data-stu-id="3fac5-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="3fac5-258">有关详细信息，请参阅[独立部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文档。</span><span class="sxs-lookup"><span data-stu-id="3fac5-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="3fac5-259">使用团队资源管理器将代码签入 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="3fac5-260">确认应用代码已签入 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="3fac5-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="3fac5-261">创建生成定义</span><span class="sxs-lookup"><span data-stu-id="3fac5-261">Create the build definition</span></span>

1. <span data-ttu-id="3fac5-262">使用可以创建生成定义的帐户登录到 Azure Pipelines。</span><span class="sxs-lookup"><span data-stu-id="3fac5-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="3fac5-263">转到该项目的“生成 Web 应用”页。</span><span class="sxs-lookup"><span data-stu-id="3fac5-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="3fac5-264">在“参数”中，添加“-r win10-x64”代码 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="3fac5-265">使用 .NET Core 触发独立部署时需要此添加内容。</span><span class="sxs-lookup"><span data-stu-id="3fac5-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="3fac5-266">运行生成。</span><span class="sxs-lookup"><span data-stu-id="3fac5-266">Run the build.</span></span> <span data-ttu-id="3fac5-267">[独立部署生成](/dotnet/core/deploying/deploy-with-vs#simpleSelf)过程将发布可在 Azure 和 Azure Stack Hub 上运行的生成工件。</span><span class="sxs-lookup"><span data-stu-id="3fac5-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="3fac5-268">使用 Azure 托管的生成代理</span><span class="sxs-lookup"><span data-stu-id="3fac5-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="3fac5-269">在 Azure Pipelines 中使用托管生成代理是生成和部署 Web 应用的便捷做法。</span><span class="sxs-lookup"><span data-stu-id="3fac5-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="3fac5-270">维护和升级由 Microsoft Azure 自动完成，从而实现了连续不断的开发周期。</span><span class="sxs-lookup"><span data-stu-id="3fac5-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="3fac5-271">配置持续部署 (CD) 过程</span><span class="sxs-lookup"><span data-stu-id="3fac5-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="3fac5-272">Azure Pipelines 和 Azure DevOps Services 提供可配置度和可管理度高的管道，用于在多个环境中进行发布，例如开发、暂存、质量保证 (QA) 和生产环境。</span><span class="sxs-lookup"><span data-stu-id="3fac5-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="3fac5-273">此过程可能包括在应用生命周期的特定阶段需要审批。</span><span class="sxs-lookup"><span data-stu-id="3fac5-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="3fac5-274">创建发布定义</span><span class="sxs-lookup"><span data-stu-id="3fac5-274">Create release definition</span></span>

<span data-ttu-id="3fac5-275">创建发布定义是应用生成过程中的最后一步。</span><span class="sxs-lookup"><span data-stu-id="3fac5-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="3fac5-276">此发布定义用于创建发布和部署生成。</span><span class="sxs-lookup"><span data-stu-id="3fac5-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="3fac5-277">登录到 Azure Pipelines 并转到该项目的“生成和发布”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="3fac5-278">在“发布”选项卡上，选择“[+]”，然后选择“创建发布定义”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="3fac5-279">在“选择模板”上，选择“Azure 应用服务部署”，然后选择“应用”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="3fac5-280">在“添加生成工件”上，从“源(生成定义)”中选择 Azure 云生成应用 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="3fac5-281">在“管道”选项卡上，选择“查看环境任务”的“1 阶段”、“1 任务”链接   。</span><span class="sxs-lookup"><span data-stu-id="3fac5-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="3fac5-282">在“任务”选项卡上，输入“Azure”作为“环境名称”，然后从“Azure 订阅”列表中选择“AzureCloud Traders-Web EP”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="3fac5-283">输入 Azure 应用服务名称，在下一个屏幕截图中为 `northwindtraders`。</span><span class="sxs-lookup"><span data-stu-id="3fac5-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="3fac5-284">对于代理阶段，从“代理队列”列表中选择“Hosted VS2017” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="3fac5-285">在“部署 Azure 应用服务”中，为环境选择有效的包或文件夹 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="3fac5-286">在“选择文件或文件夹”中，选择“位置”下的“确定”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="3fac5-287">保存所有更改，然后返回“管道”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="3fac5-288">在“管道”选项卡上，选择“添加生成工件”，然后从“源(生成定义)”列表中选择“NorthwindCloud Traders-Vessel”   。</span><span class="sxs-lookup"><span data-stu-id="3fac5-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="3fac5-289">在“选择模板”上，添加另一个环境。</span><span class="sxs-lookup"><span data-stu-id="3fac5-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="3fac5-290">选择“Azure 应用服务部署”，然后选择“应用” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="3fac5-291">输入 `Azure Stack Hub` 作为“环境名称”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="3fac5-292">在“任务”选项卡上，找到并选择“Azure Stack Hub”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="3fac5-293">在“Azure 订阅”列表中，为 Azure Stack Hub 终结点选择“AzureStack Traders-Vessel EP” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="3fac5-294">输入 Azure Stack Hub Web 应用名称作为应用服务名称。</span><span class="sxs-lookup"><span data-stu-id="3fac5-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="3fac5-295">在“代理选择”下，从“代理队列”列表中选择“AzureStack -b Douglas Fir”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="3fac5-296">对于“部署 Azure 应用服务”，为环境选择有效的程序或文件夹 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="3fac5-297">在“选择文件或文件夹”上，在文件夹“位置”中选择“确定”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="3fac5-298">在“变量”选项卡上，找到名为 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的变量。</span><span class="sxs-lookup"><span data-stu-id="3fac5-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="3fac5-299">将变量值设置为“true”，并将其范围设置为“Azure Stack Hub” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="3fac5-300">在“管道”选项卡上，选择 NorthwindCloud Traders-Web 项目对应的“持续部署触发器”图标，然后将“持续部署触发器”设置为“启用”。   </span><span class="sxs-lookup"><span data-stu-id="3fac5-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="3fac5-301">对“NorthwindCloud Traders-Vessel”生成工件执行相同的操作。</span><span class="sxs-lookup"><span data-stu-id="3fac5-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="3fac5-302">对于 Azure Stack Hub 环境，选择“部署前的条件”图标，将触发器设置为“发布后” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="3fac5-303">保存所有更改。</span><span class="sxs-lookup"><span data-stu-id="3fac5-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="3fac5-304">根据模板创建发布定义时，发布任务的某些设置会自动定义为[环境变量](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="3fac5-305">这些设置不能在任务设置中进行修改，但可以在父环境项中进行修改。</span><span class="sxs-lookup"><span data-stu-id="3fac5-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="3fac5-306">创建发布</span><span class="sxs-lookup"><span data-stu-id="3fac5-306">Create a release</span></span>

1. <span data-ttu-id="3fac5-307">在“管道”选项卡上，打开“发布”列表，然后选择“创建发布”  。</span><span class="sxs-lookup"><span data-stu-id="3fac5-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="3fac5-308">输入发布说明，检查是否选择了正确的生成工件，然后选择“创建”。</span><span class="sxs-lookup"><span data-stu-id="3fac5-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="3fac5-309">几分钟后，会显示一个横幅，指示已创建新发布，发布名称显示为链接。</span><span class="sxs-lookup"><span data-stu-id="3fac5-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="3fac5-310">选择该链接以查看“发布摘要”页。</span><span class="sxs-lookup"><span data-stu-id="3fac5-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="3fac5-311">“发布摘要”页显示有关发布的详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fac5-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="3fac5-312">在下面的针对“Release-2”的屏幕捕获中，“环境”部分显示 Azure 的“部署状态”为“正在进行”，Azure Stack Hub 的状态为“成功” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="3fac5-313">当 Azure 环境的部署状态更改为“成功”时，将显示一个横幅，指示发布已准备好进行审批。</span><span class="sxs-lookup"><span data-stu-id="3fac5-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="3fac5-314">部署挂起或失败时，将显示蓝色 (i) 信息图标。</span><span class="sxs-lookup"><span data-stu-id="3fac5-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="3fac5-315">将鼠标悬停在该图标上，可查看包含延迟或失败原因的弹出窗口。</span><span class="sxs-lookup"><span data-stu-id="3fac5-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="3fac5-316">其他视图（如发布列表）也会显示一个图标，指示审批处于挂起状态。</span><span class="sxs-lookup"><span data-stu-id="3fac5-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="3fac5-317">该图标的弹出窗口显示环境名称以及与部署相关的更多详细信息。</span><span class="sxs-lookup"><span data-stu-id="3fac5-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="3fac5-318">管理员可以轻松查看发布的总体进度以及哪些发布正在等待审批。</span><span class="sxs-lookup"><span data-stu-id="3fac5-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="3fac5-319">监视和跟踪部署</span><span class="sxs-lookup"><span data-stu-id="3fac5-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="3fac5-320">在“发布-2”摘要页上，选择“日志” 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="3fac5-321">在部署期间，此页显示来自代理的实时日志。</span><span class="sxs-lookup"><span data-stu-id="3fac5-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="3fac5-322">左窗格显示部署中每个环境对应的每个操作的状态。</span><span class="sxs-lookup"><span data-stu-id="3fac5-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="3fac5-323">在“操作”列中选择“人员”图标以进行部署前或部署后审批，以查看批准（或拒绝）部署的人员以及他们提供的消息。</span><span class="sxs-lookup"><span data-stu-id="3fac5-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="3fac5-324">部署完成后，整个日志文件将显示在右窗格中。</span><span class="sxs-lookup"><span data-stu-id="3fac5-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="3fac5-325">在左窗格中选择任何“步骤”以查看单个步骤（如初始化作业）的日志文件 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="3fac5-326">借助查看单个日志的功能，我们可以更轻松地跟踪和调试整个部署的各个部分。</span><span class="sxs-lookup"><span data-stu-id="3fac5-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="3fac5-327">将日志文件保存为一个步骤，或将所有日志下载为 zip 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="3fac5-328">打开“摘要”选项卡以查看有关发布的一般信息。</span><span class="sxs-lookup"><span data-stu-id="3fac5-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="3fac5-329">此视图显示有关生成、部署到的环境、部署状态的详细信息以及有关发布的其他信息。</span><span class="sxs-lookup"><span data-stu-id="3fac5-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="3fac5-330">选择环境链接（Azure 或 Azure Stack Hub）以查看有关针对特定环境的现有和待定部署的信息 。</span><span class="sxs-lookup"><span data-stu-id="3fac5-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="3fac5-331">使用这些视图可以快速检查是否在两个环境中部署了相同的生成。</span><span class="sxs-lookup"><span data-stu-id="3fac5-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="3fac5-332">在浏览器中打开部署的生产应用。</span><span class="sxs-lookup"><span data-stu-id="3fac5-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="3fac5-333">例如，对于 Azure 应用服务网站，打开 URL `https://[your-app-name\].azurewebsites.net`。</span><span class="sxs-lookup"><span data-stu-id="3fac5-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="3fac5-334">Azure 与 Azure Stack Hub 的集成提供了一个可缩放的跨云解决方案</span><span class="sxs-lookup"><span data-stu-id="3fac5-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="3fac5-335">灵活可靠的多云服务提供数据安全性、备份和冗余、一致且快速的可用性、可缩放的存储和分发，以及异地兼容的路由。</span><span class="sxs-lookup"><span data-stu-id="3fac5-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="3fac5-336">此手动触发过程可确保在托管 Web 应用之间可靠、高效地进行负载切换，并可立即提供重要数据。</span><span class="sxs-lookup"><span data-stu-id="3fac5-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3fac5-337">后续步骤</span><span class="sxs-lookup"><span data-stu-id="3fac5-337">Next steps</span></span>

- <span data-ttu-id="3fac5-338">若要详细了解 Azure 云模式，请参阅[云设计模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="3fac5-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
