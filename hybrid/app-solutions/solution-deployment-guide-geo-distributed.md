---
title: 使用 Azure 和 Azure Stack 集线器直接与地理分布式应用程序通信
description: 了解如何使用 Azure 和 Azure Stack 集线器将流量定向到使用异地分布式应用程序解决方案的特定终结点。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909974"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="bf743-103">使用 Azure 和 Azure Stack 集线器直接与地理分布式应用程序通信</span><span class="sxs-lookup"><span data-stu-id="bf743-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bf743-104">了解如何根据各种指标，使用异地分布式应用模式将流量定向到特定的终结点。</span><span class="sxs-lookup"><span data-stu-id="bf743-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="bf743-105">使用基于地理位置的路由和终结点配置创建流量管理器配置文件可确保根据区域要求、企业与国际法规和数据需求将信息路由到终结点。</span><span class="sxs-lookup"><span data-stu-id="bf743-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="bf743-106">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="bf743-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="bf743-107">创建地理分散的应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="bf743-108">使用流量管理器设置应用目标。</span><span class="sxs-lookup"><span data-stu-id="bf743-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="bf743-109">使用异地分布式应用模式</span><span class="sxs-lookup"><span data-stu-id="bf743-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="bf743-110">应用可以通过异地分布模式跨越区域。</span><span class="sxs-lookup"><span data-stu-id="bf743-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="bf743-111">可以默认使用公有云，但某些用户可能需要将数据保留在其所在区域中。</span><span class="sxs-lookup"><span data-stu-id="bf743-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="bf743-112">可根据用户的要求将他们定向到最合适的云。</span><span class="sxs-lookup"><span data-stu-id="bf743-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="bf743-113">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="bf743-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="bf743-114">可伸缩性注意事项</span><span class="sxs-lookup"><span data-stu-id="bf743-114">Scalability considerations</span></span>

<span data-ttu-id="bf743-115">在本文中生成的解决方案无法处理可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="bf743-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="bf743-116">但是，如果与其他 Azure 和本地解决方案结合使用，则可以满足可伸缩性方面的要求。</span><span class="sxs-lookup"><span data-stu-id="bf743-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="bf743-117">有关通过流量管理器创建可自动缩放的混合解决方案的信息，请参阅[在 Azure 中创建跨云缩放解决方案](solution-deployment-guide-cross-cloud-scaling.md)。</span><span class="sxs-lookup"><span data-stu-id="bf743-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="bf743-118">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="bf743-118">Availability considerations</span></span>

<span data-ttu-id="bf743-119">与可伸缩性注意事项中提到的一样，此解决方案无法直接处理可用性。</span><span class="sxs-lookup"><span data-stu-id="bf743-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="bf743-120">但是，可在此解决方案中实施 Azure 和本地解决方案，以确保所有相关组件具有高可用性。</span><span class="sxs-lookup"><span data-stu-id="bf743-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="bf743-121">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="bf743-121">When to use this pattern</span></span>

- <span data-ttu-id="bf743-122">组织有海外分公司，需要自定义的区域安全性和分发策略。</span><span class="sxs-lookup"><span data-stu-id="bf743-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="bf743-123">组织的每个办事处提取员工、业务和设施数据，需要根据当地法规和时区报告活动。</span><span class="sxs-lookup"><span data-stu-id="bf743-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="bf743-124">只要对单个区域内和跨区域的多个应用部署进行应用的水平扩展，即可达到高缩放性要求以处理极高的负载要求。</span><span class="sxs-lookup"><span data-stu-id="bf743-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="bf743-125">规划拓扑</span><span class="sxs-lookup"><span data-stu-id="bf743-125">Planning the topology</span></span>

<span data-ttu-id="bf743-126">在规划分布式应用的范围之前，最好先了解以下信息：</span><span class="sxs-lookup"><span data-stu-id="bf743-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="bf743-127">**应用的自定义域：** 客户访问应用时使用的自定义域名是什么？</span><span class="sxs-lookup"><span data-stu-id="bf743-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="bf743-128">示例应用的自定义域名是 *www\.scalableasedemo.com。*</span><span class="sxs-lookup"><span data-stu-id="bf743-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="bf743-129">**流量管理器域：** 创建 [Azure 流量管理器配置文件](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles)时会选择域名。</span><span class="sxs-lookup"><span data-stu-id="bf743-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="bf743-130">此名称与*trafficmanager.net*后缀结合使用，以注册由流量管理器管理的域项。</span><span class="sxs-lookup"><span data-stu-id="bf743-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="bf743-131">就示例应用而言，选择的名称是 *scalable-ase-demo*。</span><span class="sxs-lookup"><span data-stu-id="bf743-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="bf743-132">因此，流量管理器管理的完整域名为*scalable-ase-demo.trafficmanager.net*。</span><span class="sxs-lookup"><span data-stu-id="bf743-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="bf743-133">**缩放应用占用空间的策略：** 确定应用程序占用量是分布在单个区域、多个区域还是混合两种方法的多个应用服务环境中。</span><span class="sxs-lookup"><span data-stu-id="bf743-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="bf743-134">决策依据应来自于客户流量的来源位置，以及其余应用的支持后端基础结构的伸缩性。</span><span class="sxs-lookup"><span data-stu-id="bf743-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="bf743-135">例如，对于100% 无状态应用程序，可以使用每个 Azure 区域的多个应用服务环境的组合，乘以跨多个 Azure 区域部署的应用服务环境，来大幅缩放应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="bf743-136">由于有15个以上的 Azure 区域可供选择，客户可以真正构建全球超大规模应用程序。</span><span class="sxs-lookup"><span data-stu-id="bf743-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="bf743-137">对于此处使用的示例应用，在单个 Azure 区域（美国中南部）中创建了三个应用服务环境。</span><span class="sxs-lookup"><span data-stu-id="bf743-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="bf743-138">**应用服务环境的命名约定：** 每个应用服务环境都需要唯一名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="bf743-139">在一个或两个应用服务环境之外，具有命名约定有助于识别每个应用服务环境，这会很有帮助。</span><span class="sxs-lookup"><span data-stu-id="bf743-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="bf743-140">此处的示例应用中使用了简单的命名约定。</span><span class="sxs-lookup"><span data-stu-id="bf743-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="bf743-141">三个应用服务环境的名称为*fe1ase*、 *fe2ase*和*fe3ase*。</span><span class="sxs-lookup"><span data-stu-id="bf743-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="bf743-142">**应用的命名约定：** 由于将部署多个应用实例，每个部署的应用实例都要有名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="bf743-143">使用适用于电源应用的应用服务环境，可以在多个环境中使用相同的应用名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="bf743-144">由于每个应用服务环境都有一个唯一的域后缀，因此开发人员可以选择在每个环境中重复使用完全相同的应用名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="bf743-145">例如，开发人员可能具有名为的应用程序，如下所示： *myapp.foo1.p.azurewebsites.net*、 *myapp.foo2.p.azurewebsites.net*、 *myapp.foo3.p.azurewebsites.net*等。</span><span class="sxs-lookup"><span data-stu-id="bf743-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="bf743-146">对于此处使用的应用，每个应用实例具有唯一的名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="bf743-147">所用的应用实例名称是 *webfrontend1*、*webfrontend2* 和 *webfrontend3*。</span><span class="sxs-lookup"><span data-stu-id="bf743-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="bf743-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="bf743-148">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="bf743-149">Microsoft Azure Stack 中心是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="bf743-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="bf743-150">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="bf743-151">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="bf743-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="bf743-152">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="bf743-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="bf743-153">第 1 部分：创建异地分布式应用</span><span class="sxs-lookup"><span data-stu-id="bf743-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="bf743-154">在此部分创建 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="bf743-155">创建 Web 应用并发布。</span><span class="sxs-lookup"><span data-stu-id="bf743-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="bf743-156">将代码添加到 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="bf743-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="bf743-157">将应用生成指向多个云目标。</span><span class="sxs-lookup"><span data-stu-id="bf743-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="bf743-158">管理和配置 CD 过程。</span><span class="sxs-lookup"><span data-stu-id="bf743-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="bf743-159">先决条件</span><span class="sxs-lookup"><span data-stu-id="bf743-159">Prerequisites</span></span>

<span data-ttu-id="bf743-160">需要 Azure 订阅并安装 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="bf743-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="bf743-161">异地分布式应用步骤</span><span class="sxs-lookup"><span data-stu-id="bf743-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="bf743-162">获取自定义域并配置 DNS</span><span class="sxs-lookup"><span data-stu-id="bf743-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="bf743-163">更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="bf743-164">然后，Azure AD 可以验证自定义域名的所有权。</span><span class="sxs-lookup"><span data-stu-id="bf743-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="bf743-165">将 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 用于 Azure 中的 Azure/Office 365/外部 DNS 记录，或在[其他 DNS 注册机构](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)中添加 DNS 条目。</span><span class="sxs-lookup"><span data-stu-id="bf743-165">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

1. <span data-ttu-id="bf743-166">向公共注册机构注册自定义域。</span><span class="sxs-lookup"><span data-stu-id="bf743-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="bf743-167">登录到域的域名注册机构。</span><span class="sxs-lookup"><span data-stu-id="bf743-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="bf743-168">可能需要由获批准的管理员进行 DNS 更新。</span><span class="sxs-lookup"><span data-stu-id="bf743-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="bf743-169">通过添加 Azure AD 提供的 DNS 条目来更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="bf743-170">DNS 条目不会更改行为，例如邮件路由或 Web 托管。</span><span class="sxs-lookup"><span data-stu-id="bf743-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="bf743-171">创建 Web 应用并发布</span><span class="sxs-lookup"><span data-stu-id="bf743-171">Create web apps and publish</span></span>

<span data-ttu-id="bf743-172">设置混合持续集成/持续交付 (CI/CD)，以将 Web 应用部署到 Azure 和 Azure Stack Hub，并自动将更改推送到这两个云中。</span><span class="sxs-lookup"><span data-stu-id="bf743-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="bf743-173">需要在 Azure Stack Hub 上创建适当的合成映像用于运行 Windows Server 和 SQL，并需要部署应用服务。</span><span class="sxs-lookup"><span data-stu-id="bf743-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="bf743-174">有关详细信息，请参阅[在 Azure Stack 集线器上部署应用服务的先决条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)。</span><span class="sxs-lookup"><span data-stu-id="bf743-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="bf743-175">向 Azure Repos 中添加代码</span><span class="sxs-lookup"><span data-stu-id="bf743-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="bf743-176">使用在 Azure Repos 上**拥有项目创建权限的帐户**登录到 Visual Studio。</span><span class="sxs-lookup"><span data-stu-id="bf743-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="bf743-177">CI/CD 可同时应用到应用代码和基础结构代码。</span><span class="sxs-lookup"><span data-stu-id="bf743-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="bf743-178">使用 [Azure 资源管理器模板](https://azure.microsoft.com/resources/templates/)进行专用与托管的云开发。</span><span class="sxs-lookup"><span data-stu-id="bf743-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![在 Visual Studio 中连接到项目](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="bf743-180">创建并打开默认 Web 应用以**克隆存储库**。</span><span class="sxs-lookup"><span data-stu-id="bf743-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![在 Visual Studio 中克隆存储库](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="bf743-182">在这两个云中创建 Web 应用部署</span><span class="sxs-lookup"><span data-stu-id="bf743-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="bf743-183">编辑 **WebApplication.csproj** 文件：选择 `Runtimeidentifier` 并添加 `win10-x64`。</span><span class="sxs-lookup"><span data-stu-id="bf743-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="bf743-184">（请参阅[独立部署](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf)文档。）</span><span class="sxs-lookup"><span data-stu-id="bf743-184">(See [Self-contained Deployment](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![在 Visual Studio 中编辑 web 应用项目文件](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="bf743-186">使用团队资源管理器**将代码签入到 Azure Repos**。</span><span class="sxs-lookup"><span data-stu-id="bf743-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="bf743-187">确认**应用程序代码**已签入到 Azure Repos。</span><span class="sxs-lookup"><span data-stu-id="bf743-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="bf743-188">创建生成定义</span><span class="sxs-lookup"><span data-stu-id="bf743-188">Create the build definition</span></span>

1. <span data-ttu-id="bf743-189">**登录到 Azure Pipelines** 以确认能够创建生成定义。</span><span class="sxs-lookup"><span data-stu-id="bf743-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="bf743-190">添加 `-r win10-x64` 代码。</span><span class="sxs-lookup"><span data-stu-id="bf743-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="bf743-191">使用 .NET Core 触发独立部署时需要添加此代码。</span><span class="sxs-lookup"><span data-stu-id="bf743-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![将代码添加到中的生成定义 Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="bf743-193">**运行生成**。</span><span class="sxs-lookup"><span data-stu-id="bf743-193">**Run the build**.</span></span> <span data-ttu-id="bf743-194">[独立部署生成](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf)过程将发布可在 Azure 和 Azure Stack Hub 上运行的项目。</span><span class="sxs-lookup"><span data-stu-id="bf743-194">The [self-contained deployment build](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="bf743-195">使用 Azure 托管代理</span><span class="sxs-lookup"><span data-stu-id="bf743-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="bf743-196">在 Azure Pipelines 中使用托管代理是生成和部署 Web 应用的便捷做法。</span><span class="sxs-lookup"><span data-stu-id="bf743-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="bf743-197">维护和升级由 Microsoft Azure 自动执行，从而实现不间断的开发、测试和部署。</span><span class="sxs-lookup"><span data-stu-id="bf743-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="bf743-198">管理和配置 CD 过程</span><span class="sxs-lookup"><span data-stu-id="bf743-198">Manage and configure the CD process</span></span>

<span data-ttu-id="bf743-199">Azure DevOps Services 提供高度可配置、可管理的管道，用于将内容发布到多个环境（例如开发、过渡、QA 和生产环境）；在特定的阶段要求审批。</span><span class="sxs-lookup"><span data-stu-id="bf743-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="bf743-200">创建发布定义</span><span class="sxs-lookup"><span data-stu-id="bf743-200">Create release definition</span></span>

1. <span data-ttu-id="bf743-201">在 Azure DevOps Services 的 "**生成和发布**" 部分的 "**发布**" 选项卡下，选择**加号**按钮以添加新发布。</span><span class="sxs-lookup"><span data-stu-id="bf743-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![在 Azure DevOps Services 中创建发布定义](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="bf743-203">应用“Azure 应用服务部署”模板。</span><span class="sxs-lookup"><span data-stu-id="bf743-203">Apply the Azure App Service Deployment template.</span></span>

   ![应用 Azure App Service 部署模板 Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="bf743-205">在“添加项目”下，为 Azure 云生成应用添加项目。 </span><span class="sxs-lookup"><span data-stu-id="bf743-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![将项目添加到 Azure DevOps Services 中的 Azure Cloud build](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="bf743-207">在“管道”选项卡下选择环境的“阶段和任务”链接，并设置 Azure 云环境值。 </span><span class="sxs-lookup"><span data-stu-id="bf743-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![在 Azure DevOps Services 中设置 Azure 云环境值](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="bf743-209">设置**环境名称**，并选择 Azure 云终结点的 **Azure 订阅**。</span><span class="sxs-lookup"><span data-stu-id="bf743-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![在 Azure DevOps Services 中选择 Azure 云终结点的 Azure 订阅](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="bf743-211">在“应用服务名称”下，设置所需的 Azure 应用服务名称。 </span><span class="sxs-lookup"><span data-stu-id="bf743-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![在 Azure DevOps Services 中设置 Azure 应用服务名称](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="bf743-213">在 Azure 云托管环境的“代理队列”下输入“Hosted VS2017”。 </span><span class="sxs-lookup"><span data-stu-id="bf743-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![在 Azure DevOps Services 中为 Azure 云托管环境设置代理队列](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="bf743-215">在“部署 Azure 应用服务”菜单中，为环境选择有效的**包或文件夹**。</span><span class="sxs-lookup"><span data-stu-id="bf743-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="bf743-216">选择**文件夹位置**旁边的“确定”。 </span><span class="sxs-lookup"><span data-stu-id="bf743-216">Select **OK** to **folder location**.</span></span>
  
      ![在 Azure DevOps Services 中选择 Azure App Service 环境的包或文件夹](media/solution-deployment-guide-geo-distributed/image12.png)

      ![在 Azure DevOps Services 中选择 Azure App Service 环境的包或文件夹](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="bf743-219">保存所有更改并返回**发布管道**。</span><span class="sxs-lookup"><span data-stu-id="bf743-219">Save all changes and go back to **release pipeline**.</span></span>

    ![在 Azure DevOps Services 中保存发布管道中的更改](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="bf743-221">选择 Azure Stack Hub 应用的生成以添加新项目。</span><span class="sxs-lookup"><span data-stu-id="bf743-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![在 Azure DevOps Services 中为 Azure Stack 中心应用添加新项目](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="bf743-223">通过应用 Azure 应用服务部署额外添加一个环境。</span><span class="sxs-lookup"><span data-stu-id="bf743-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![将环境添加到 Azure DevOps Services 中的 Azure App Service 部署](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="bf743-225">将新环境命名为 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="bf743-225">Name the new environment Azure Stack Hub.</span></span>

    ![Azure DevOps Services 中 Azure App Service 部署中的名称环境](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="bf743-227">在“任务”选项卡下找到 Azure Stack Hub 环境。 </span><span class="sxs-lookup"><span data-stu-id="bf743-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure DevOps Services 中 Azure DevOps Services 的 Azure Stack 集线器环境](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="bf743-229">选择 Azure Stack Hub 终结点的订阅。</span><span class="sxs-lookup"><span data-stu-id="bf743-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![在 Azure DevOps Services 中选择 Azure Stack 中心终结点的订阅](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="bf743-231">将 Azure Stack Hub Web 应用名称设置为应用服务名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![在 Azure DevOps Services 中设置 Azure Stack 中心 web 应用名称](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="bf743-233">选择 Azure Stack Hub 代理。</span><span class="sxs-lookup"><span data-stu-id="bf743-233">Select the Azure Stack Hub agent.</span></span>

    ![在 Azure DevOps Services 中选择 Azure Stack 集线器代理](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="bf743-235">在“部署 Azure 应用服务”部分下，为环境选择有效的**包或文件夹**。</span><span class="sxs-lookup"><span data-stu-id="bf743-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="bf743-236">选择文件夹位置旁边的“确定”。 </span><span class="sxs-lookup"><span data-stu-id="bf743-236">Select **OK** to folder location.</span></span>

    ![在 Azure DevOps Services 中选择 Azure App Service 部署的文件夹](media/solution-deployment-guide-geo-distributed/image22.png)

    ![在 Azure DevOps Services 中选择 Azure App Service 部署的文件夹](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="bf743-239">在“变量”选项卡下添加名为 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的变量，将其值设置为 **true**，将范围设置为 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="bf743-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![将变量添加到 Azure DevOps Services 中的 Azure 应用部署](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="bf743-241">选择两个项目中的“持续”部署触发器图标，并启用“持续”部署触发器。  </span><span class="sxs-lookup"><span data-stu-id="bf743-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![在 Azure DevOps Services 中选择连续部署触发器](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="bf743-243">选择 Azure Stack Hub 环境中的“部署前”条件图标，并将触发器设置为“发布后”。  </span><span class="sxs-lookup"><span data-stu-id="bf743-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![在 Azure DevOps Services 中选择预先部署条件](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="bf743-245">保存所有更改。</span><span class="sxs-lookup"><span data-stu-id="bf743-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="bf743-246">任务的某些设置可能已在从模板创建发布定义时自动定义为[环境变量](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables)。</span><span class="sxs-lookup"><span data-stu-id="bf743-246">Some settings for the tasks may have been automatically defined as [environment variables](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="bf743-247">无法在任务设置中修改这些设置；必须选择父环境项才能编辑这些设置。</span><span class="sxs-lookup"><span data-stu-id="bf743-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="bf743-248">第 2 部分：更新 Web 应用选项</span><span class="sxs-lookup"><span data-stu-id="bf743-248">Part 2: Update web app options</span></span>

<span data-ttu-id="bf743-249">[Azure 应用服务](https://docs.microsoft.com/azure/app-service/overview)提供高度可缩放、自修复的 Web 托管服务。</span><span class="sxs-lookup"><span data-stu-id="bf743-249">[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure 应用服务](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="bf743-251">将现有的自定义 DNS 名称映射到 Azure Web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="bf743-252">使用 **CNAME 记录**和 **A 记录**将自定义 DNS 名称映射到应用服务。</span><span class="sxs-lookup"><span data-stu-id="bf743-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="bf743-253">将现有的自定义 DNS 名称映射到 Azure Web 应用</span><span class="sxs-lookup"><span data-stu-id="bf743-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="bf743-254">对除根域（例如 northwind.com）以外的所有自定义 DNS 名称使用 CNAME。</span><span class="sxs-lookup"><span data-stu-id="bf743-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="bf743-255">若要将实时站点及其 DNS 域名迁移到应用服务，请参阅[将活动 DNS 名称迁移到 Azure 应用服务](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain)。</span><span class="sxs-lookup"><span data-stu-id="bf743-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="bf743-256">先决条件</span><span class="sxs-lookup"><span data-stu-id="bf743-256">Prerequisites</span></span>

<span data-ttu-id="bf743-257">若要完成此解决方案：</span><span class="sxs-lookup"><span data-stu-id="bf743-257">To complete this solution:</span></span>

- <span data-ttu-id="bf743-258">[创建一个应用服务应用](https://docs.microsoft.com/azure/app-service/)，或者使用为其他解决方案创建的应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-258">[Create an App Service app](https://docs.microsoft.com/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="bf743-259">购买一个域名并确保你对你的域提供商的 DNS 注册表拥有访问权限。</span><span class="sxs-lookup"><span data-stu-id="bf743-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="bf743-260">更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="bf743-261">然后，Azure AD 将会验证自定义域名的所有权。</span><span class="sxs-lookup"><span data-stu-id="bf743-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="bf743-262">将 [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) 用于 Azure 中的 Azure/Office 365/外部 DNS 记录，或在[其他 DNS 注册机构](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)中添加 DNS 条目。</span><span class="sxs-lookup"><span data-stu-id="bf743-262">Use [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) for Azure/Office 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).</span></span>

- <span data-ttu-id="bf743-263">向公共注册机构注册自定义域。</span><span class="sxs-lookup"><span data-stu-id="bf743-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="bf743-264">登录到域的域名注册机构。</span><span class="sxs-lookup"><span data-stu-id="bf743-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="bf743-265">（可能需要由获批准的管理员进行 DNS 更新。）</span><span class="sxs-lookup"><span data-stu-id="bf743-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="bf743-266">通过添加 Azure AD 提供的 DNS 条目来更新域的 DNS 区域文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="bf743-267">例如，若要添加 northwindcloud.com 和 www\.northwindcloud.com 的 DNS 条目，请配置 northwindcloud.com 根域的 DNS 设置。</span><span class="sxs-lookup"><span data-stu-id="bf743-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="bf743-268">可以使用 [Azure 门户](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain)购买域名。</span><span class="sxs-lookup"><span data-stu-id="bf743-268">A domain name may be purchased using the [Azure portal](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="bf743-269">若要映射自定义 DNS 名称到 Web 应用，Web 应用的[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/)必须位于付费层（“共享”、“基本”、“标准”或“高级”）。\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="bf743-270">创建并映射 CNAME 和 A 记录</span><span class="sxs-lookup"><span data-stu-id="bf743-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="bf743-271">通过域提供商访问 DNS 记录</span><span class="sxs-lookup"><span data-stu-id="bf743-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="bf743-272">使用 Azure DNS 为 Azure Web 应用配置自定义 DNS 名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="bf743-273">有关详细信息，请参阅[使用 Azure DNS 为 Azure 服务提供自定义域设置](https://docs.microsoft.com/azure/dns/dns-custom-domain)。</span><span class="sxs-lookup"><span data-stu-id="bf743-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](https://docs.microsoft.com/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="bf743-274">登录到域提供商的网站。</span><span class="sxs-lookup"><span data-stu-id="bf743-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="bf743-275">查找管理 DNS 记录的页面。</span><span class="sxs-lookup"><span data-stu-id="bf743-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="bf743-276">每个域提供商都有其自己的 DNS 记录界面。</span><span class="sxs-lookup"><span data-stu-id="bf743-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="bf743-277">查找站点中标记为“域名”、“DNS”或“名称服务器管理”的区域。  </span><span class="sxs-lookup"><span data-stu-id="bf743-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="bf743-278">可以在“我的域”中查看 DNS 记录页。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="bf743-279">找到命名为“区域文件”、“DNS 记录”或“高级配置”的链接。\*\*\*\*\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="bf743-280">以下屏幕截图是 DNS 记录页的一个示例：</span><span class="sxs-lookup"><span data-stu-id="bf743-280">The following screenshot is an example of a DNS records page:</span></span>

![示例 DNS 记录页](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="bf743-282">在“域名注册机构”中，选择“添加或创建”以创建记录。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="bf743-283">某些提供商提供了不同的链接来添加不同的记录类型。</span><span class="sxs-lookup"><span data-stu-id="bf743-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="bf743-284">查阅提供商的文档。</span><span class="sxs-lookup"><span data-stu-id="bf743-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="bf743-285">添加一条 CNAME 记录来将子域映射到应用的默认主机名。</span><span class="sxs-lookup"><span data-stu-id="bf743-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="bf743-286">对于 www \. northwindcloud.com 域示例，添加将名称映射到的 CNAME 记录 `<app_name>.azurewebsites.net` 。</span><span class="sxs-lookup"><span data-stu-id="bf743-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="bf743-287">添加 CNAME 后，DNS 记录页类似于以下示例：</span><span class="sxs-lookup"><span data-stu-id="bf743-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![在门户中导航到 Azure 应用](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="bf743-289">在 Azure 中启用 CNAME 记录映射</span><span class="sxs-lookup"><span data-stu-id="bf743-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="bf743-290">在新选项卡中，登录到 Azure 门户。</span><span class="sxs-lookup"><span data-stu-id="bf743-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="bf743-291">转到“应用服务”。</span><span class="sxs-lookup"><span data-stu-id="bf743-291">Go to App Services.</span></span>

3. <span data-ttu-id="bf743-292">选择 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-292">Select web app.</span></span>

4. <span data-ttu-id="bf743-293">在 Azure 门户中，在应用页面的左侧导航窗格中，选择“自定义域”。</span><span class="sxs-lookup"><span data-stu-id="bf743-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="bf743-294">选择 " **+** **添加主机名**" 旁边的图标。</span><span class="sxs-lookup"><span data-stu-id="bf743-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="bf743-295">键入完全限定的域名，如 `www.northwindcloud.com` 。</span><span class="sxs-lookup"><span data-stu-id="bf743-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="bf743-296">选择“验证”。</span><span class="sxs-lookup"><span data-stu-id="bf743-296">Select **Validate**.</span></span>

8. <span data-ttu-id="bf743-297">如有出现提示，请将其他类型的记录（`A` 或 `TXT`）添加到域名注册机构 DNS 记录。</span><span class="sxs-lookup"><span data-stu-id="bf743-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="bf743-298">Azure 将提供这些记录的值和类型：</span><span class="sxs-lookup"><span data-stu-id="bf743-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="bf743-299">a.</span><span class="sxs-lookup"><span data-stu-id="bf743-299">a.</span></span>  <span data-ttu-id="bf743-300">**A** 记录映射到应用的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="bf743-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="bf743-301">b.</span><span class="sxs-lookup"><span data-stu-id="bf743-301">b.</span></span>  <span data-ttu-id="bf743-302">要映射到应用默认主机名 `<app_name>.azurewebsites.net` 的 TXT\*\*\*\* 记录。</span><span class="sxs-lookup"><span data-stu-id="bf743-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="bf743-303">应用服务仅在配置时使用此记录来验证自定义域所有权。</span><span class="sxs-lookup"><span data-stu-id="bf743-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="bf743-304">验证后，删除 TXT 记录。</span><span class="sxs-lookup"><span data-stu-id="bf743-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="bf743-305">在域注册机构选项卡中完成此任务并重新验证，直到“添加主机名”按钮激活为止。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="bf743-306">请确保 "**主机名记录类型**" 设置为 " **CNAME** （www.example.com 或任何子域）"。</span><span class="sxs-lookup"><span data-stu-id="bf743-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="bf743-307">选择“添加主机名”。</span><span class="sxs-lookup"><span data-stu-id="bf743-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="bf743-308">键入完全限定的域名，如 `northwindcloud.com` 。</span><span class="sxs-lookup"><span data-stu-id="bf743-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="bf743-309">选择“验证”。</span><span class="sxs-lookup"><span data-stu-id="bf743-309">Select **Validate**.</span></span> <span data-ttu-id="bf743-310">“添加”按钮将会激活。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="bf743-311">请确保 "**主机名记录类型**" 设置为 " **A 记录**（example.com）"。</span><span class="sxs-lookup"><span data-stu-id="bf743-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="bf743-312">**添加主机名**。</span><span class="sxs-lookup"><span data-stu-id="bf743-312">**Add hostname**.</span></span>

    <span data-ttu-id="bf743-313">新主机名可能需要经过一段时间后才会反映在应用的“自定义域”页中。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="bf743-314">请尝试刷新浏览器来更新数据。</span><span class="sxs-lookup"><span data-stu-id="bf743-314">Try refreshing the browser to update the data.</span></span>
  
    ![自定义域](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="bf743-316">如果发生错误，页面底部会显示验证错误通知。</span><span class="sxs-lookup"><span data-stu-id="bf743-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![域验证错误](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="bf743-318">可以重复上述步骤以映射通配符域 (\*.northwindcloud.com)。</span><span class="sxs-lookup"><span data-stu-id="bf743-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="bf743-319">如此可以直接将其他子域添加到此应用服务，而无需为每个子域单独创建 CNAME 记录。</span><span class="sxs-lookup"><span data-stu-id="bf743-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="bf743-320">请遵照注册机构的说明配置此设置。</span><span class="sxs-lookup"><span data-stu-id="bf743-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="bf743-321">在浏览器中测试</span><span class="sxs-lookup"><span data-stu-id="bf743-321">Test in a browser</span></span>

<span data-ttu-id="bf743-322">浏览到前面配置的 DNS 名称（例如 `northwindcloud.com` 或 `www.northwindcloud.com` ）。</span><span class="sxs-lookup"><span data-stu-id="bf743-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="bf743-323">第3部分：绑定自定义 SSL 证书</span><span class="sxs-lookup"><span data-stu-id="bf743-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="bf743-324">在此部分，我们要：</span><span class="sxs-lookup"><span data-stu-id="bf743-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="bf743-325">将自定义 SSL 证书绑定到应用服务。</span><span class="sxs-lookup"><span data-stu-id="bf743-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="bf743-326">为应用实施 HTTPS。</span><span class="sxs-lookup"><span data-stu-id="bf743-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="bf743-327">使用脚本自动执行 SSL 证书绑定。</span><span class="sxs-lookup"><span data-stu-id="bf743-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="bf743-328">如果需要，请在 Azure 门户中获取客户 SSL 证书并将其绑定到 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="bf743-329">有关详细信息，请参阅[应用服务证书教程](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site)。</span><span class="sxs-lookup"><span data-stu-id="bf743-329">For more information, see the [App Service Certificates tutorial](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="bf743-330">先决条件</span><span class="sxs-lookup"><span data-stu-id="bf743-330">Prerequisites</span></span>

<span data-ttu-id="bf743-331">若要完成此解决方案：</span><span class="sxs-lookup"><span data-stu-id="bf743-331">To complete this  solution:</span></span>

- [<span data-ttu-id="bf743-332">创建应用服务应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-332">Create an App Service app.</span></span>](https://docs.microsoft.com/azure/app-service/)
- [<span data-ttu-id="bf743-333">将自定义 DNS 名称映射到 web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-333">Map a custom DNS name to your web app.</span></span>](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="bf743-334">从受信任的证书颁发机构获取 SSL 证书，并使用密钥将请求签名。</span><span class="sxs-lookup"><span data-stu-id="bf743-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="bf743-335">SSL 证书的要求</span><span class="sxs-lookup"><span data-stu-id="bf743-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="bf743-336">若要在应用服务中使用证书，该证书必须满足以下所有要求：</span><span class="sxs-lookup"><span data-stu-id="bf743-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="bf743-337">已由受信任的证书颁发机构签名。</span><span class="sxs-lookup"><span data-stu-id="bf743-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="bf743-338">已导出为受密码保护的 PFX 文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="bf743-339">包含长度至少为 2048 位的私钥。</span><span class="sxs-lookup"><span data-stu-id="bf743-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="bf743-340">包含证书链中的所有中间证书。</span><span class="sxs-lookup"><span data-stu-id="bf743-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="bf743-341">**椭圆曲线加密 (ECC) 证书**可用于应用服务，但本指南不予讨论。</span><span class="sxs-lookup"><span data-stu-id="bf743-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="bf743-342">创建 ECC 证书时如需帮助，请咨询证书颁发机构。</span><span class="sxs-lookup"><span data-stu-id="bf743-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="bf743-343">准备 Web 应用</span><span class="sxs-lookup"><span data-stu-id="bf743-343">Prepare the web app</span></span>

<span data-ttu-id="bf743-344">若要将自定义 SSL 证书绑定到 Web 应用，[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/)必须位于“基本”、“标准”或“高级”层。\*\*\*\*\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="bf743-345">登录 Azure</span><span class="sxs-lookup"><span data-stu-id="bf743-345">Sign in to Azure</span></span>

1. <span data-ttu-id="bf743-346">打开[Azure 门户](https://portal.azure.com/)并中转到 web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="bf743-347">在左侧菜单中选择“应用服务”，然后选择 Web 应用名称。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![在 Azure 门户中选择 web 应用](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="bf743-349">检查定价层</span><span class="sxs-lookup"><span data-stu-id="bf743-349">Check the pricing tier</span></span>

1. <span data-ttu-id="bf743-350">在 Web 应用页面的左侧导航窗格中，向下滚动到“设置”部分，然后选择“纵向扩展(应用服务计划)”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Web 应用中的向上缩放菜单](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="bf743-352">确保 Web 应用不在“免费”或“共享”层中。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="bf743-353">深蓝色的框突出显示了 Web 应用的当前层。</span><span class="sxs-lookup"><span data-stu-id="bf743-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![检查 web 应用中的定价层](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="bf743-355">“免费”和“共享”层不支持自定义 SSL。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="bf743-356">若要纵向扩展，请遵循下一部分中的步骤，或者关闭“选择定价层”页面并跳到[上传和绑定 SSL 证书](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="bf743-357">纵向扩展应用服务计划</span><span class="sxs-lookup"><span data-stu-id="bf743-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="bf743-358">选择“基本”、“标准”或“高级”层。\*\*\*\*\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="bf743-359">选择“选择”。</span><span class="sxs-lookup"><span data-stu-id="bf743-359">Select **Select**.</span></span>

![选择 web 应用的定价层](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="bf743-361">显示通知即表示缩放操作已完成。</span><span class="sxs-lookup"><span data-stu-id="bf743-361">The scale operation is complete when notification is displayed.</span></span>

![扩展通知](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="bf743-363">绑定 SSL 证书并合并中间证书</span><span class="sxs-lookup"><span data-stu-id="bf743-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="bf743-364">在链中合并多个证书。</span><span class="sxs-lookup"><span data-stu-id="bf743-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="bf743-365">在文本编辑器中**打开收到的每个证书**。</span><span class="sxs-lookup"><span data-stu-id="bf743-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="bf743-366">创建名为*为 mergedcertificate.crt*的合并证书的文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="bf743-367">在文本编辑器中，将每个证书的内容复制到此文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="bf743-368">证书的顺序应遵循证书链中的顺序，以你的证书开头，以根证书结尾，</span><span class="sxs-lookup"><span data-stu-id="bf743-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="bf743-369">如以下示例所示：</span><span class="sxs-lookup"><span data-stu-id="bf743-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="bf743-370">将证书导出为 PFX</span><span class="sxs-lookup"><span data-stu-id="bf743-370">Export certificate to PFX</span></span>

<span data-ttu-id="bf743-371">使用证书生成的私钥导出已合并的 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="bf743-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="bf743-372">私钥文件是通过 OpenSSL 创建的。</span><span class="sxs-lookup"><span data-stu-id="bf743-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="bf743-373">若要将证书导出为 PFX，请运行以下命令，并将占位符 `<private-key-file>` 和 `<merged-certificate-file>` 私钥路径以及合并的证书文件一起替换：</span><span class="sxs-lookup"><span data-stu-id="bf743-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="bf743-374">出现提示时，请定义稍后将 SSL 证书上传到应用服务时要使用的密码。</span><span class="sxs-lookup"><span data-stu-id="bf743-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="bf743-375">如果使用 IIS 或 **Certreq.exe** 生成证书请求，请将证书安装到本地计算机，然后[将证书导出为 PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx)。</span><span class="sxs-lookup"><span data-stu-id="bf743-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="bf743-376">上传 SSL 证书</span><span class="sxs-lookup"><span data-stu-id="bf743-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="bf743-377">在 Web 应用的左侧导航窗格中选择“SSL 设置”。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="bf743-378">选择 "**上传证书**"。</span><span class="sxs-lookup"><span data-stu-id="bf743-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="bf743-379">在“PFX 证书文件”中选择 PFX 文件。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="bf743-380">在“证书密码”中，键入导出 PFX 文件时创建的密码。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="bf743-381">选择**上载**。</span><span class="sxs-lookup"><span data-stu-id="bf743-381">Select **Upload**.</span></span>

    ![上传 SSL 证书](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="bf743-383">应用服务上传完证书后，该证书会显示在“SSL 设置”页面中。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL 设置](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="bf743-385">绑定 SSL 证书</span><span class="sxs-lookup"><span data-stu-id="bf743-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="bf743-386">在“SSL 绑定”部分选择“添加绑定”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="bf743-387">如果证书已上传，但未显示在“主机名”下拉列表中的域名内，请尝试刷新浏览器页面。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="bf743-388">在 "**添加 SSL 绑定**" 页上，使用下拉菜单选择要保护的域名，并选择要使用的证书。</span><span class="sxs-lookup"><span data-stu-id="bf743-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="bf743-389">在“SSL 类型”中，选择是要使用 [**服务器名称指示 (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) 还是使用基于 IP 的 SSL。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="bf743-390">**基于 sni 的 ssl**：可以添加多个基于 SNI 的 ssl 绑定。</span><span class="sxs-lookup"><span data-stu-id="bf743-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="bf743-391">选择此选项可以使用多个 SSL 证书来保护同一 IP 地址上的多个域。</span><span class="sxs-lookup"><span data-stu-id="bf743-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="bf743-392">大多数新式浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI（在[服务器名称指示](https://wikipedia.org/wiki/Server_Name_Indication)中了解更全面的浏览器支持信息）。</span><span class="sxs-lookup"><span data-stu-id="bf743-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="bf743-393">**基于 ip 的 ssl**：只能添加一个基于 IP 的 ssl 绑定。</span><span class="sxs-lookup"><span data-stu-id="bf743-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="bf743-394">选择此选项只能使用一个 SSL 证书来保护专用公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="bf743-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="bf743-395">若要保护多个域，请使用同一个 SSL 证书来保护所有这些域。</span><span class="sxs-lookup"><span data-stu-id="bf743-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="bf743-396">基于 IP 的 SSL 是 SSL 绑定的传统选项。</span><span class="sxs-lookup"><span data-stu-id="bf743-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="bf743-397">选择“添加绑定”。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-397">Select **Add Binding**.</span></span>

    ![添加 SSL 绑定](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="bf743-399">应用服务上传完证书后，该证书将显示在“SSL 绑定”部分。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![SSL 绑定完成上传](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="bf743-401">重新映射 IP SSL 的 A 记录</span><span class="sxs-lookup"><span data-stu-id="bf743-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="bf743-402">如果未在 Web 应用中使用基于 IP 的 SSL，请转到[测试自定义域的 HTTPS](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl)。</span><span class="sxs-lookup"><span data-stu-id="bf743-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="bf743-403">默认情况下，Web 应用使用共享的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="bf743-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="bf743-404">将证书绑定到基于 IP 的 SSL 后，应用服务会立即为 Web 应用创建一个新的专用 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="bf743-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="bf743-405">将 A 记录映射到 Web 应用后，必须使用该专用 IP 地址更新域注册表。</span><span class="sxs-lookup"><span data-stu-id="bf743-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="bf743-406">“自定义域”页面将会更新，显示新的专用 IP 地址。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="bf743-407">复制此[IP 地址](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)，然后将[A 记录](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)重新映射到这个新的 ip 地址。</span><span class="sxs-lookup"><span data-stu-id="bf743-407">Copy this [IP address](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="bf743-408">测试 HTTPS</span><span class="sxs-lookup"><span data-stu-id="bf743-408">Test HTTPS</span></span>

<span data-ttu-id="bf743-409">在不同的浏览器中，请参阅以 `https://<your.custom.domain>` 确保 web 应用服务。</span><span class="sxs-lookup"><span data-stu-id="bf743-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![浏览到 Web 应用](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="bf743-411">如果发生证书验证错误，原因可能是自签名证书所致，或者在导出到 PFX 文件时遗漏了中间证书。</span><span class="sxs-lookup"><span data-stu-id="bf743-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="bf743-412">强制执行 HTTPS</span><span class="sxs-lookup"><span data-stu-id="bf743-412">Enforce HTTPS</span></span>

<span data-ttu-id="bf743-413">默认情况下，任何人都可以使用 HTTP 访问 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="bf743-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="bf743-414">可将所有 HTTP 请求重定向到 HTTPS 端口。</span><span class="sxs-lookup"><span data-stu-id="bf743-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="bf743-415">在 Web 应用页面中，选择“SL 设置”。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="bf743-416">然后，在“仅 HTTPS”中，选择“启用”。</span><span class="sxs-lookup"><span data-stu-id="bf743-416">Then, in **HTTPS Only**, select **On**.</span></span>

![实施 HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="bf743-418">操作完成后，请跳到指向应用的任何 HTTP Url。</span><span class="sxs-lookup"><span data-stu-id="bf743-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="bf743-419">例如：</span><span class="sxs-lookup"><span data-stu-id="bf743-419">For example:</span></span>

- <span data-ttu-id="bf743-420">https://<app_name>。 azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="bf743-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="bf743-421">强制实施 TLS 1.1/1.2</span><span class="sxs-lookup"><span data-stu-id="bf743-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="bf743-422">默认情况下，此应用程序允许使用[TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0，这不再被行业标准（如[PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)）视为安全。</span><span class="sxs-lookup"><span data-stu-id="bf743-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="bf743-423">若要强制实施更高的 TLS 版本，请按照下列步骤操作：</span><span class="sxs-lookup"><span data-stu-id="bf743-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="bf743-424">在 Web 应用页面的左侧导航窗格中，选择“SSL 设置”。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="bf743-425">在“TLS 版本”中，选择最低的 TLS 版本。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![强制实施 TLS 1.1 或 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="bf743-427">创建流量管理器配置文件</span><span class="sxs-lookup"><span data-stu-id="bf743-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="bf743-428">选择 "**创建资源**" "  >  **网络**  >  **流量管理器配置文件**" "  >  **创建**"。</span><span class="sxs-lookup"><span data-stu-id="bf743-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="bf743-429">在“创建流量管理器配置文件”\*\*\*\* 中，按如下所示完成操作：</span><span class="sxs-lookup"><span data-stu-id="bf743-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="bf743-430">在“名称”中提供配置文件的名称。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="bf743-431">此名称需要在流量 manager.net 区域中唯一，并产生 DNS 名称 trafficmanager.net，该名称用于访问流量管理器配置文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="bf743-432">在“路由方法”中，选择“地理路由方法”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="bf743-433">在“订阅”中，选择要在其中创建此配置文件的订阅。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="bf743-434">在**资源组**中，创建新的资源组，以在其下放置此配置文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="bf743-435">在**资源组位置**中，选择资源组的位置。</span><span class="sxs-lookup"><span data-stu-id="bf743-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="bf743-436">此设置指的是资源组的位置，对全局部署的流量管理器配置文件没有影响。</span><span class="sxs-lookup"><span data-stu-id="bf743-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="bf743-437">选择“创建”。</span><span class="sxs-lookup"><span data-stu-id="bf743-437">Select **Create**.</span></span>

    7. <span data-ttu-id="bf743-438">流量管理器配置文件的全局部署完成后，它会在相应的资源组中作为资源之一列出。</span><span class="sxs-lookup"><span data-stu-id="bf743-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![创建流量管理器配置文件中的资源组](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="bf743-440">添加流量管理器终结点</span><span class="sxs-lookup"><span data-stu-id="bf743-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="bf743-441">在门户搜索栏中，搜索在前面部分中创建的**流量管理器配置文件**名称，并在显示的结果中选择流量管理器配置文件。</span><span class="sxs-lookup"><span data-stu-id="bf743-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="bf743-442">在“流量管理器配置文件”\*\*\*\* 的“设置”\*\*\*\* 部分，选择“终结点”\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="bf743-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="bf743-443">选择 **添加** 。</span><span class="sxs-lookup"><span data-stu-id="bf743-443">Select **Add**.</span></span>

4. <span data-ttu-id="bf743-444">添加 Azure Stack Hub 终结点。</span><span class="sxs-lookup"><span data-stu-id="bf743-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="bf743-445">对于“类型”，请选择“外部终结点”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="bf743-446">提供此终结点的**名称**，最好是 Azure Stack Hub 的名称。</span><span class="sxs-lookup"><span data-stu-id="bf743-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="bf743-447">对于完全限定的域名 (**FQDN**)，请使用 Azure Stack Hub Web 应用的外部 URL。</span><span class="sxs-lookup"><span data-stu-id="bf743-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="bf743-448">在 "异地映射" 下，选择资源所在的区域/大洲。</span><span class="sxs-lookup"><span data-stu-id="bf743-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="bf743-449">例如，**欧洲。**</span><span class="sxs-lookup"><span data-stu-id="bf743-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="bf743-450">在显示的 "国家/地区" 下拉箭头下，选择适用于此终结点的国家/地区。</span><span class="sxs-lookup"><span data-stu-id="bf743-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="bf743-451">例如**德国**。</span><span class="sxs-lookup"><span data-stu-id="bf743-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="bf743-452">使“添加为已禁用”\*\*\*\* 保持未选中状态。</span><span class="sxs-lookup"><span data-stu-id="bf743-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="bf743-453">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="bf743-453">Select **OK**.</span></span>

12. <span data-ttu-id="bf743-454">添加 Azure 终结点：</span><span class="sxs-lookup"><span data-stu-id="bf743-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="bf743-455">对于“类型”，请选择“Azure 终结点”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="bf743-456">提供终结点的**名称**。</span><span class="sxs-lookup"><span data-stu-id="bf743-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="bf743-457">对于“目标资源类型”，请选择“应用服务”。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="bf743-458">对于“目标资源”，请选择“选择应用服务”以显示同一订阅下的 Web 应用列表。\*\*\*\*\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="bf743-459">在“资源”中，选取要用作第一个终结点的应用服务。\*\*\*\*</span><span class="sxs-lookup"><span data-stu-id="bf743-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="bf743-460">在 "异地映射" 下，选择资源所在的区域/大洲。</span><span class="sxs-lookup"><span data-stu-id="bf743-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="bf743-461">例如，**北美/中美洲/加勒比海。**</span><span class="sxs-lookup"><span data-stu-id="bf743-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="bf743-462">在显示的 "国家/地区" 下拉箭头下，将此位置留空以选择以上所有区域分组。</span><span class="sxs-lookup"><span data-stu-id="bf743-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="bf743-463">使“添加为已禁用”\*\*\*\* 保持未选中状态。</span><span class="sxs-lookup"><span data-stu-id="bf743-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="bf743-464">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="bf743-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="bf743-465">创建至少一个地理范围为“全部(全球)”的终结点，作为资源的默认终结点。</span><span class="sxs-lookup"><span data-stu-id="bf743-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="bf743-466">添加完这两个终结点后，这两个终结点会显示在“流量管理器配置文件”\*\*\*\* 中，并且其监视状态为“联机”\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="bf743-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![流量管理器配置文件终结点状态](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="bf743-468">全球企业依赖于 Azure 异地分发功能</span><span class="sxs-lookup"><span data-stu-id="bf743-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="bf743-469">通过 Azure 流量管理器和地理位置特定的终结点定向数据流量，全球企业可以遵守区域法规，保持数据的合规性和安全性，这对于本地和远程业务成功而言都至关重要。</span><span class="sxs-lookup"><span data-stu-id="bf743-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bf743-470">后续步骤</span><span class="sxs-lookup"><span data-stu-id="bf743-470">Next steps</span></span>

- <span data-ttu-id="bf743-471">若要详细了解 Azure 云模式，请参阅[云设计模式](https://docs.microsoft.com/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="bf743-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
