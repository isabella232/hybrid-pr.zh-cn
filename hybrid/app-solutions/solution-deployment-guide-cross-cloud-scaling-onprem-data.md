---
title: 部署使用本地数据并跨云缩放的混合应用
description: 了解如何利用 Azure 和 Azure Stack Hub 部署使用本地数据并跨云缩放的应用。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0989859fd68847932d3e69defee59740a2bffd44
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895391"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="a12f6-103">部署使用本地数据并跨云缩放的混合应用</span><span class="sxs-lookup"><span data-stu-id="a12f6-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="a12f6-104">此解决方案指南介绍如何部署跨 Azure 和 Azure Stack Hub 并使用单个本地数据源的混合应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="a12f6-105">使用混合云解决方案，可以结合私有云在合规性方面的优势与公有云的可伸缩性。</span><span class="sxs-lookup"><span data-stu-id="a12f6-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="a12f6-106">开发人员还可以利用 Microsoft 开发人员生态系统在云和本地环境中应用他们的技能。</span><span class="sxs-lookup"><span data-stu-id="a12f6-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="a12f6-107">概述和假设</span><span class="sxs-lookup"><span data-stu-id="a12f6-107">Overview and assumptions</span></span>

<span data-ttu-id="a12f6-108">遵循本教程设置工作流，让开发人员将相同的 Web 应用部署到公有云和私有云。</span><span class="sxs-lookup"><span data-stu-id="a12f6-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="a12f6-109">此应用可以访问私有云中托管的、无法通过 Internet 路由的网络。</span><span class="sxs-lookup"><span data-stu-id="a12f6-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="a12f6-110">这些 Web 应用受到监视；出现流量高峰时，某个程序会修改 DNS 记录，以将流量重定向到公有云。</span><span class="sxs-lookup"><span data-stu-id="a12f6-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="a12f6-111">如果流量下降到高峰出现之前的水平，则流量将路由回到私有云。</span><span class="sxs-lookup"><span data-stu-id="a12f6-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="a12f6-112">本教程涵盖以下任务：</span><span class="sxs-lookup"><span data-stu-id="a12f6-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a12f6-113">部署混合连接的 SQL Server 数据库服务器。</span><span class="sxs-lookup"><span data-stu-id="a12f6-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="a12f6-114">将全局 Azure 中的 Web 应用连接到混合网络。</span><span class="sxs-lookup"><span data-stu-id="a12f6-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="a12f6-115">为跨云缩放配置 DNS。</span><span class="sxs-lookup"><span data-stu-id="a12f6-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="a12f6-116">为跨云缩放配置 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="a12f6-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="a12f6-117">配置并部署 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="a12f6-118">创建流量管理器配置文件，并根据跨云缩放对其进行配置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="a12f6-119">针对增大的流量设置 Application Insights 监视和警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="a12f6-120">配置全局 Azure 与 Azure Stack Hub 之间的自动流量切换。</span><span class="sxs-lookup"><span data-stu-id="a12f6-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="a12f6-121">![混合支柱图](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a12f6-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a12f6-122">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="a12f6-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a12f6-123">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a12f6-124">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="a12f6-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a12f6-125">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="a12f6-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="a12f6-126">假设</span><span class="sxs-lookup"><span data-stu-id="a12f6-126">Assumptions</span></span>

<span data-ttu-id="a12f6-127">此教程假设你了解全局 Azure 和 Azure Stack Hub 的基本知识。</span><span class="sxs-lookup"><span data-stu-id="a12f6-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="a12f6-128">若要在开始本教程之前了解详细信息，请查看以下文章：</span><span class="sxs-lookup"><span data-stu-id="a12f6-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="a12f6-129">Azure 简介</span><span class="sxs-lookup"><span data-stu-id="a12f6-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="a12f6-130">Azure Stack Hub 的重要概念</span><span class="sxs-lookup"><span data-stu-id="a12f6-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview)

<span data-ttu-id="a12f6-131">本教程还假设你有一个 Azure 订阅。</span><span class="sxs-lookup"><span data-stu-id="a12f6-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="a12f6-132">如果没有订阅，请在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a12f6-133">先决条件</span><span class="sxs-lookup"><span data-stu-id="a12f6-133">Prerequisites</span></span>

<span data-ttu-id="a12f6-134">在开始此解决方案之前，请确保符合以下要求：</span><span class="sxs-lookup"><span data-stu-id="a12f6-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="a12f6-135">Azure Stack 开发工具包 (ASDK) 或 Azure Stack Hub 集成系统的订阅。</span><span class="sxs-lookup"><span data-stu-id="a12f6-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="a12f6-136">若要部署 ASDK，请按照[使用安装程序部署 ASDK](/azure-stack/asdk/asdk-install) 中的说明进行操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install).</span></span>
- <span data-ttu-id="a12f6-137">Azure Stack Hub 安装中应包含以下组件：</span><span class="sxs-lookup"><span data-stu-id="a12f6-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="a12f6-138">Azure 应用服务。</span><span class="sxs-lookup"><span data-stu-id="a12f6-138">The Azure App Service.</span></span> <span data-ttu-id="a12f6-139">请与 Azure Stack Hub 操作员协作，在环境中部署并配置 Azure 应用服务。</span><span class="sxs-lookup"><span data-stu-id="a12f6-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="a12f6-140">在本教程中，应用服务必须至少有一 (1) 个可用的专用辅助角色。</span><span class="sxs-lookup"><span data-stu-id="a12f6-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="a12f6-141">Windows Server 2016 映像。</span><span class="sxs-lookup"><span data-stu-id="a12f6-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="a12f6-142">包含 Microsoft SQL Server 映像的 Windows Server 2016。</span><span class="sxs-lookup"><span data-stu-id="a12f6-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="a12f6-143">相应的计划和产品/服务。</span><span class="sxs-lookup"><span data-stu-id="a12f6-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="a12f6-144">Web 应用的域名。</span><span class="sxs-lookup"><span data-stu-id="a12f6-144">A domain name for your web app.</span></span> <span data-ttu-id="a12f6-145">如果没有域名，可以从 GoDaddy、Bluehost 和 InMotion 等域提供商处购买一个。</span><span class="sxs-lookup"><span data-stu-id="a12f6-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="a12f6-146">LetsEncrypt 等受信任的证书颁发机构颁发的用于你的域的 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="a12f6-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="a12f6-147">与 SQL Server 数据库通信且支持 Application Insights 的 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="a12f6-148">可以从 GitHub 下载 [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) 示例应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="a12f6-149">Azure 虚拟网络与 Azure Stack Hub 虚拟网络之间的混合网络。</span><span class="sxs-lookup"><span data-stu-id="a12f6-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="a12f6-150">有关详细说明，请参阅[使用 Azure 和 Azure Stack Hub 配置混合云连接](solution-deployment-guide-connectivity.md)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="a12f6-151">Azure Stack Hub 中包含专用生成代理的混合持续集成/持续部署 (CI/CD) 管道。</span><span class="sxs-lookup"><span data-stu-id="a12f6-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="a12f6-152">有关详细说明，请参阅[使用 Azure 和 Azure Stack Hub 应用配置混合云标识](solution-deployment-guide-identity.md)</span><span class="sxs-lookup"><span data-stu-id="a12f6-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="a12f6-153">部署混合连接的 SQL Server 数据库服务器</span><span class="sxs-lookup"><span data-stu-id="a12f6-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="a12f6-154">登录到 Azure Stack Hub 用户门户。</span><span class="sxs-lookup"><span data-stu-id="a12f6-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="a12f6-155">在“仪表板”中选择“市场”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack Hub 市场](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="a12f6-157">在“市场”中选择“计算”，然后选择“更多”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="a12f6-158">在“更多”下面，选择“免费 SQL Server 许可证:   Windows Server 上的 SQL Server 2017 Developer”映像。</span><span class="sxs-lookup"><span data-stu-id="a12f6-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![在 Azure Stack Hub 用户门户中选择虚拟机映像](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="a12f6-160">在“免费 SQL Server 许可证:   Windows Server 上的 SQL Server 2017 Developer”中，选择“创建”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="a12f6-161">在“基本信息”>“配置基本设置”中，提供虚拟机 (VM) 的 **名称**、SQL Server SA 的 **用户名**，以及 SA 的 **密码**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="a12f6-162">在“订阅”下拉列表中，选择要部署到的订阅。</span><span class="sxs-lookup"><span data-stu-id="a12f6-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="a12f6-163">对于“资源组”，请使用“选择现有项”，并将 VM 放到 Azure Stack Hub Web 应用所在的同一资源组中。</span><span class="sxs-lookup"><span data-stu-id="a12f6-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![在 Azure Stack Hub 用户门户中为 VM 配置基本设置](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="a12f6-165">在“大小”下面，选择 VM 的大小。</span><span class="sxs-lookup"><span data-stu-id="a12f6-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="a12f6-166">对于本教程，建议使用 A2_Standard 或 DS2_V2_Standard。</span><span class="sxs-lookup"><span data-stu-id="a12f6-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="a12f6-167">在“设置”>“配置可选功能”下面配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="a12f6-168">**存储帐户**：根据需要创建新帐户。</span><span class="sxs-lookup"><span data-stu-id="a12f6-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="a12f6-169">**虚拟网络**：</span><span class="sxs-lookup"><span data-stu-id="a12f6-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="a12f6-170">请务必将 SQL Server VM 部署到 VPN 网关所在的同一虚拟网络中。</span><span class="sxs-lookup"><span data-stu-id="a12f6-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="a12f6-171">**公共 IP 地址**：使用默认设置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="a12f6-172">**网络安全组**：(NSG)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="a12f6-173">创建新 NSG。</span><span class="sxs-lookup"><span data-stu-id="a12f6-173">Create a new NSG.</span></span>
   - <span data-ttu-id="a12f6-174">**扩展和监视**：保留默认设置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="a12f6-175">**诊断存储帐户**：根据需要创建新帐户。</span><span class="sxs-lookup"><span data-stu-id="a12f6-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="a12f6-176">选择“确定”以保存配置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-176">Select **OK** to save your configuration.</span></span>

     ![在 Azure Stack Hub 用户门户中配置可选 VM 功能](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="a12f6-178">在“SQL Server 设置”下面配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="a12f6-179">对于“SQL 连接”，选择“公共(Internet)” 。</span><span class="sxs-lookup"><span data-stu-id="a12f6-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="a12f6-180">对于“端口”，请保留默认值 **1433**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="a12f6-181">对于“SQL 身份验证”，请选择“启用”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="a12f6-182">启用 SQL 身份验证时，应会自动填充在“基本信息”中配置的“SQLAdmin”信息。</span><span class="sxs-lookup"><span data-stu-id="a12f6-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="a12f6-183">对于剩余的设置，请保留默认值。</span><span class="sxs-lookup"><span data-stu-id="a12f6-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="a12f6-184">选择“确定”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-184">Select **OK**.</span></span>

     ![在 Azure Stack Hub 用户门户中配置 SQL Server 设置](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="a12f6-186">在“摘要”上，查看 VM 配置，然后选择“确定”以开始部署 。</span><span class="sxs-lookup"><span data-stu-id="a12f6-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Azure Stack Hub 用户门户中的配置摘要](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="a12f6-188">创建新 VM 需要花费一段时间。</span><span class="sxs-lookup"><span data-stu-id="a12f6-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="a12f6-189">可以在“虚拟机”中查看 VM 的状态。</span><span class="sxs-lookup"><span data-stu-id="a12f6-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Azure Stack Hub 用户门户中的虚拟机状态](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="a12f6-191">在 Azure 和 Azure Stack Hub 中创建 Web 应用</span><span class="sxs-lookup"><span data-stu-id="a12f6-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-192">Azure 应用服务简化了运行和管理 Web 应用的过程。</span><span class="sxs-lookup"><span data-stu-id="a12f6-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="a12f6-193">由于 Azure Stack Hub 与 Azure 相一致，因此，应用服务可在这两个环境中运行。</span><span class="sxs-lookup"><span data-stu-id="a12f6-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="a12f6-194">你将使用应用服务来托管应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="a12f6-195">创建 Web 应用</span><span class="sxs-lookup"><span data-stu-id="a12f6-195">Create web apps</span></span>

1. <span data-ttu-id="a12f6-196">遵照[在 Azure 中管理应用服务计划](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)中的说明，在 Azure 中创建 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="a12f6-197">请务必将 Web 应用放到混合网络所在的同一订阅和资源组中。</span><span class="sxs-lookup"><span data-stu-id="a12f6-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="a12f6-198">在 Azure Stack Hub 中重复上述步骤 (1)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="a12f6-199">为 Azure Stack Hub 添加路由</span><span class="sxs-lookup"><span data-stu-id="a12f6-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-200">Azure Stack Hub 上的应用服务必须可从公共 Internet 进行路由，使用户能够访问你的应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="a12f6-201">如果 Azure Stack Hub 可从 Internet 访问，请记下 Azure Stack Hub Web 应用的面向公众的 IP 地址或 URL。</span><span class="sxs-lookup"><span data-stu-id="a12f6-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="a12f6-202">如果使用 ASDK，则可以[配置静态 NAT 映射](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal)，以便在虚拟环境外部公开应用服务。</span><span class="sxs-lookup"><span data-stu-id="a12f6-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="a12f6-203">将 Azure 中的 Web 应用连接到混合网络</span><span class="sxs-lookup"><span data-stu-id="a12f6-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="a12f6-204">若要让 Azure 中的 Web 前端与 Azure Stack Hub 中的 SQL Server 数据库相互连接，必须将 Web 应用连接到 Azure 与 Azure Stack Hub 之间的混合网络。</span><span class="sxs-lookup"><span data-stu-id="a12f6-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="a12f6-205">若要启用连接，必须：</span><span class="sxs-lookup"><span data-stu-id="a12f6-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="a12f6-206">配置点到站点连接。</span><span class="sxs-lookup"><span data-stu-id="a12f6-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="a12f6-207">配置 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-207">Configure the web app.</span></span>
- <span data-ttu-id="a12f6-208">修改 Azure Stack Hub 中的本地网络网关。</span><span class="sxs-lookup"><span data-stu-id="a12f6-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="a12f6-209">为点到站点连接配置 Azure 虚拟网络</span><span class="sxs-lookup"><span data-stu-id="a12f6-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="a12f6-210">在混合网络中，Azure 端的虚拟网络网关必须允许点到站点连接，以便与 Azure 应用服务集成。</span><span class="sxs-lookup"><span data-stu-id="a12f6-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="a12f6-211">在 Azure 门户中，转到虚拟网络网关页。</span><span class="sxs-lookup"><span data-stu-id="a12f6-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="a12f6-212">在“设置”下面，选择“点到站点配置”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Azure 虚拟网络网关中的点到站点选项](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="a12f6-214">选择“立即配置”以配置点到站点连接。</span><span class="sxs-lookup"><span data-stu-id="a12f6-214">Select **Configure now** to configure point-to-site.</span></span>

    ![在 Azure 虚拟网络网关中启动点到站点配置](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="a12f6-216">在“点到站点”配置页上的“地址池”中，输入要使用的专用 IP 地址范围。</span><span class="sxs-lookup"><span data-stu-id="a12f6-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="a12f6-217">确保指定的范围不与混合网络的全局 Azure 或 Azure Stack Hub 组件中的子网使用的任何地址范围重叠。</span><span class="sxs-lookup"><span data-stu-id="a12f6-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="a12f6-218">在“隧道类型”下面，取消选中“IKEv2 VPN”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="a12f6-219">选择“保存”完成点到站点配置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Azure 虚拟网络网关中的点到站点设置](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="a12f6-221">将 Azure 应用服务应用与混合网络集成</span><span class="sxs-lookup"><span data-stu-id="a12f6-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="a12f6-222">若要将应用连接到 Azure VNet，请遵照[网关所需的 VNet 集成](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)中的说明操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="a12f6-223">转到托管 Web 应用的应用服务计划的“设置”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="a12f6-224">在“设置”中，选择“网络”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-224">In **Settings**, select **Networking**.</span></span>

    ![为应用服务计划配置网络](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="a12f6-226">在“VNET 集成”中，选择“单击此处进行管理”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![为应用服务计划管理 VNET 集成](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="a12f6-228">选择要配置的 VNET。</span><span class="sxs-lookup"><span data-stu-id="a12f6-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="a12f6-229">在“路由到 VNET 的 IP 地址”下面，输入 Azure VNet、Azure Stack Hub VNet 和点到站点地址空间的 IP 地址范围。</span><span class="sxs-lookup"><span data-stu-id="a12f6-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="a12f6-230">选择“保存”以验证并保存这些设置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-230">Select **Save** to validate and save these settings.</span></span>

    ![在虚拟网络集成中路由的 IP 地址范围](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="a12f6-232">若要详细了解应用服务如何与 Azure VNet 集成，请参阅[将应用与 Azure 虚拟网络集成](/azure/app-service/web-sites-integrate-with-vnet)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="a12f6-233">配置 Azure Stack Hub 虚拟网络</span><span class="sxs-lookup"><span data-stu-id="a12f6-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="a12f6-234">需将 Azure Stack Hub 虚拟网络中的本地网络网关配置为路由来自应用服务点到站点地址范围的流量。</span><span class="sxs-lookup"><span data-stu-id="a12f6-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="a12f6-235">在 Azure Stack Hub 门户中，转到“本地网络网关”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-235">In the Azure Stack Hub portal, go to **Local network gateway**.</span></span> <span data-ttu-id="a12f6-236">在“设置”下，选择“配置”   。</span><span class="sxs-lookup"><span data-stu-id="a12f6-236">Under **Settings**, select **Configuration**.</span></span>

    ![Azure Stack Hub 本地网络网关中的网关配置选项](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="a12f6-238">在“地址空间”中，输入 Azure 中虚拟网络网关的点到站点地址范围。</span><span class="sxs-lookup"><span data-stu-id="a12f6-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Azure Stack Hub 本地网络网关中的点到站点地址空间](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="a12f6-240">选择“保存”以验证并保存配置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="a12f6-241">为跨云缩放配置 DNS</span><span class="sxs-lookup"><span data-stu-id="a12f6-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="a12f6-242">通过为跨云应用正确配置 DNS，用户可以访问 Web 应用的全局 Azure 和 Azure Stack Hub 实例。</span><span class="sxs-lookup"><span data-stu-id="a12f6-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="a12f6-243">本教程中的 DNS 配置还可让 Azure 流量管理器在负载增加或减少时路由流量。</span><span class="sxs-lookup"><span data-stu-id="a12f6-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="a12f6-244">本教程使用 Azure DNS 来管理 DNS，因为应用服务域无法正常运行。</span><span class="sxs-lookup"><span data-stu-id="a12f6-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="a12f6-245">创建子域</span><span class="sxs-lookup"><span data-stu-id="a12f6-245">Create subdomains</span></span>

<span data-ttu-id="a12f6-246">由于流量管理器依赖于 DNS CNAME，因此需要使用子域来正确将流量路由到终结点</span><span class="sxs-lookup"><span data-stu-id="a12f6-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="a12f6-247">有关 DNS 记录和域映射的详细信息，请参阅[使用流量管理器映射域](/azure/app-service/web-sites-traffic-manager-custom-domain-name)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="a12f6-248">对于 Azure 终结点，需要创建一个可让用户用来访问你的 Web 应用的子域。</span><span class="sxs-lookup"><span data-stu-id="a12f6-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="a12f6-249">在本教程中可以使用 **app.northwind.com**，但应根据自己的域自定义此值。</span><span class="sxs-lookup"><span data-stu-id="a12f6-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="a12f6-250">此外，需要为 Azure Stack Hub 终结点创建包含 A 记录的子域。</span><span class="sxs-lookup"><span data-stu-id="a12f6-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="a12f6-251">可以使用 **azurestack.northwind.com**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="a12f6-252">在 Azure 中配置自定义域</span><span class="sxs-lookup"><span data-stu-id="a12f6-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="a12f6-253">通过 [将 CNAME 映射到 Azure 应用服务](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)，将 **app.northwind.com** 主机名添加到 Azure Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="a12f6-254">在 Azure Stack Hub 中配置自定义域</span><span class="sxs-lookup"><span data-stu-id="a12f6-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="a12f6-255">通过 [将 A 记录映射到 Azure 应用服务](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record)，将 **azurestack.northwind.com** 主机名添加到 Azure Stack Hub Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="a12f6-256">将可通过 Internet 路由的 IP 地址用于应用服务应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="a12f6-257">通过 [将 CNAME 映射到 Azure 应用服务](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record)，将 **app.northwind.com** 主机名添加到 Azure Stack Hub Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="a12f6-258">使用在前一步骤 (1) 中配置的主机名作为 CNAME 的目标。</span><span class="sxs-lookup"><span data-stu-id="a12f6-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="a12f6-259">为跨云缩放配置 SSL 证书</span><span class="sxs-lookup"><span data-stu-id="a12f6-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="a12f6-260">必须确保 Web 应用收集的敏感数据在传输到 SQL 数据库以及存储在该数据库中时受到保护。</span><span class="sxs-lookup"><span data-stu-id="a12f6-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="a12f6-261">将 Azure 和 Azure Stack Hub Web 应用配置为对所有传入流量使用 SSL 证书。</span><span class="sxs-lookup"><span data-stu-id="a12f6-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="a12f6-262">将 SSL 添加到 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="a12f6-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-263">将 SSL 添加到 Azure：</span><span class="sxs-lookup"><span data-stu-id="a12f6-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="a12f6-264">确保获取的 SSL 证书对于创建的子域有效。</span><span class="sxs-lookup"><span data-stu-id="a12f6-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="a12f6-265">（也可以使用通配符证书。）</span><span class="sxs-lookup"><span data-stu-id="a12f6-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="a12f6-266">在 Azure 门户中，按照 [将现有 SSL 证书绑定到 Azure Web 应用](/azure/app-service/app-service-web-tutorial-custom-ssl)一文中的“准备 Web 应用”和“绑定 SSL 证书”部分的指导操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="a12f6-267">为“SSL 类型”选择“基于 SNI 的 SSL”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="a12f6-268">将所有流量重定向到 HTTPS 端口。</span><span class="sxs-lookup"><span data-stu-id="a12f6-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="a12f6-269">遵照[将现有的自定义 SSL 证书绑定到 Azure Web 应用](/azure/app-service/app-service-web-tutorial-custom-ssl)一文的“强制实施 HTTPS”部分的说明操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="a12f6-270">将 SSL 添加到 Azure Stack Hub：</span><span class="sxs-lookup"><span data-stu-id="a12f6-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="a12f6-271">使用 Azure Stack Hub 门户，重复你用于 Azure 的第 1-3 步。</span><span class="sxs-lookup"><span data-stu-id="a12f6-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="a12f6-272">配置并部署 Web 应用</span><span class="sxs-lookup"><span data-stu-id="a12f6-272">Configure and deploy the web app</span></span>

<span data-ttu-id="a12f6-273">你将配置应用代码，以向正确的 Application Insights 实例报告遥测，并使用正确的连接字符串配置 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="a12f6-274">若要详细了解 Application Insights，请参阅[什么是 Application Insights？](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="a12f6-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="a12f6-275">添加 Application Insights</span><span class="sxs-lookup"><span data-stu-id="a12f6-275">Add Application Insights</span></span>

1. <span data-ttu-id="a12f6-276">在 Microsoft Visual Studio 中打开 Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="a12f6-277">向项目中[添加 Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications)，以传输在 Web 流量增加或减少时 Application Insights 用于创建警报的遥测。</span><span class="sxs-lookup"><span data-stu-id="a12f6-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="a12f6-278">配置动态连接字符串</span><span class="sxs-lookup"><span data-stu-id="a12f6-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="a12f6-279">Web 应用的每个实例都会使用不同的方法连接到 SQL 数据库。</span><span class="sxs-lookup"><span data-stu-id="a12f6-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="a12f6-280">Azure 中的应用使用 SQL Server VM 的专用 IP 地址，而 Azure Stack Hub 中的应用使用 SQL Server VM 的公共 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="a12f6-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="a12f6-281">在 Azure Stack Hub 集成系统上，公共 IP 地址不应是可通过 Internet 路由的 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="a12f6-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="a12f6-282">在 ASDK 上，公共 IP 地址不可在 ASDK 外部路由。</span><span class="sxs-lookup"><span data-stu-id="a12f6-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="a12f6-283">可以使用应用服务环境变量将不同的连接字符串传递给应用的每个实例。</span><span class="sxs-lookup"><span data-stu-id="a12f6-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="a12f6-284">在 Visual Studio 中打开应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="a12f6-285">打开 Startup.cs 并找到以下代码块：</span><span class="sxs-lookup"><span data-stu-id="a12f6-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="a12f6-286">将前面的代码块替换为以下代码，此代码使用在 appsettings.json  文件中定义的连接字符串：</span><span class="sxs-lookup"><span data-stu-id="a12f6-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="a12f6-287">配置应用服务应用设置</span><span class="sxs-lookup"><span data-stu-id="a12f6-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="a12f6-288">创建适用于 Azure 和 Azure Stack Hub 的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="a12f6-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="a12f6-289">在这些字符串中，除使用的 IP 地址外，其余部分应该相同。</span><span class="sxs-lookup"><span data-stu-id="a12f6-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="a12f6-290">在 Azure 和 Azure Stack Hub 中，使用 `SQLCONNSTR\_` 作为名称中的前缀，添加相应的连接字符串作为 Web 应用中的[应用设置](/azure/app-service/web-sites-configure)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="a12f6-291">**保存** Web 应用设置并重启应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="a12f6-292">在全局 Azure 中启用自动缩放</span><span class="sxs-lookup"><span data-stu-id="a12f6-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="a12f6-293">在应用服务环境中创建 Web 应用时，它最初有一个实例。</span><span class="sxs-lookup"><span data-stu-id="a12f6-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="a12f6-294">可通过自动横向扩展来添加实例，以便为应用提供更多的计算资源。</span><span class="sxs-lookup"><span data-stu-id="a12f6-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="a12f6-295">同样，可以自动缩减并减少应用所需的实例数。</span><span class="sxs-lookup"><span data-stu-id="a12f6-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="a12f6-296">需要创建应用服务计划来配置横向扩展和缩减。</span><span class="sxs-lookup"><span data-stu-id="a12f6-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="a12f6-297">如果没有计划，请在开始执行后续步骤之前创建一个计划。</span><span class="sxs-lookup"><span data-stu-id="a12f6-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="a12f6-298">启用自动横向扩展</span><span class="sxs-lookup"><span data-stu-id="a12f6-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="a12f6-299">在 Azure 门户中，找到你想横向扩展的站点的应用服务计划，然后选择“横向扩展(应用服务计划)”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![横向扩展 Azure 应用服务](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="a12f6-301">选择“启用自动缩放”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-301">Select **Enable autoscale**.</span></span>

    ![在 Azure 应用服务中启用自动缩放](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="a12f6-303">在“自动缩放设置名称”中输入名称。</span><span class="sxs-lookup"><span data-stu-id="a12f6-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="a12f6-304">对于“默认”自动缩放规则，请选择“基于指标缩放”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="a12f6-305">将“实例限制”设置为“最小值:   1”、“最大值:  10”和“默认值:  1”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![在 Azure 应用服务中配置自动缩放](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="a12f6-307">选择“+添加规则”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="a12f6-308">在“指标源”中，选择“当前资源”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="a12f6-309">对规则使用以下条件和操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="a12f6-310">条件</span><span class="sxs-lookup"><span data-stu-id="a12f6-310">Criteria</span></span>

1. <span data-ttu-id="a12f6-311">在“时间聚合”下面，选择“平均”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="a12f6-312">在“指标名称”下面，选择“CPU 百分比”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="a12f6-313">在“运算符”下面，选择“大于”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="a12f6-314">将“阈值”设置为 **50**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="a12f6-315">将“持续时间”设置为 **10**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="a12f6-316">操作</span><span class="sxs-lookup"><span data-stu-id="a12f6-316">Action</span></span>

1. <span data-ttu-id="a12f6-317">在“操作”下面，选择“计数增量”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="a12f6-318">将“实例计数”设置为 **2**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="a12f6-319">将“冷却时间”设置为 **5**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="a12f6-320">选择“添加”   。</span><span class="sxs-lookup"><span data-stu-id="a12f6-320">Select **Add**.</span></span>

5. <span data-ttu-id="a12f6-321">选择“+添加规则”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="a12f6-322">在“指标源”中，选择“当前资源”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="a12f6-323">当前资源将包含应用服务计划的名称/GUID，并且“资源类型”和“资源”下拉列表将不可用 。</span><span class="sxs-lookup"><span data-stu-id="a12f6-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="a12f6-324">启用自动横向缩减</span><span class="sxs-lookup"><span data-stu-id="a12f6-324">Enable automatic scale in</span></span>

<span data-ttu-id="a12f6-325">当流量减少时，Azure Web 应用可以自动减少活动实例的数目，以降低成本。</span><span class="sxs-lookup"><span data-stu-id="a12f6-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="a12f6-326">此操作的力度不如横向扩展，并可尽量降低对应用用户造成的影响。</span><span class="sxs-lookup"><span data-stu-id="a12f6-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="a12f6-327">转到“默认”横向扩展条件，然后选择“+ 添加规则” 。</span><span class="sxs-lookup"><span data-stu-id="a12f6-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="a12f6-328">对规则使用以下条件和操作。</span><span class="sxs-lookup"><span data-stu-id="a12f6-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="a12f6-329">条件</span><span class="sxs-lookup"><span data-stu-id="a12f6-329">Criteria</span></span>

1. <span data-ttu-id="a12f6-330">在“时间聚合”下面，选择“平均”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="a12f6-331">在“指标名称”下面，选择“CPU 百分比”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="a12f6-332">在“运算符”下面，选择“小于”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="a12f6-333">将“阈值”设置为 **30**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="a12f6-334">将“持续时间”设置为 **10**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="a12f6-335">操作</span><span class="sxs-lookup"><span data-stu-id="a12f6-335">Action</span></span>

1. <span data-ttu-id="a12f6-336">在“操作”下面，选择“计数减量”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="a12f6-337">将“实例计数”设置为 **1**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="a12f6-338">将“冷却时间”设置为 **5**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="a12f6-339">选择“添加”   。</span><span class="sxs-lookup"><span data-stu-id="a12f6-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="a12f6-340">创建流量管理器配置文件并配置跨云缩放</span><span class="sxs-lookup"><span data-stu-id="a12f6-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="a12f6-341">使用 Azure 门户创建流量管理器配置文件，然后配置终结点以启用跨云缩放。</span><span class="sxs-lookup"><span data-stu-id="a12f6-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="a12f6-342">创建流量管理器配置文件</span><span class="sxs-lookup"><span data-stu-id="a12f6-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="a12f6-343">选择“创建资源”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="a12f6-344">选择“网络”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-344">Select **Networking**.</span></span>
3. <span data-ttu-id="a12f6-345">选择“流量管理器配置文件”并配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="a12f6-346">在“名称”中，输入配置文件的名称。</span><span class="sxs-lookup"><span data-stu-id="a12f6-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="a12f6-347">此名称在 trafficmanager.net 区域中必须是唯一的，并且用于创建新的 DNS 名称（例如 northwindstore.trafficmanager.net）。</span><span class="sxs-lookup"><span data-stu-id="a12f6-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="a12f6-348">对于“路由方法”，请选择“加权”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="a12f6-349">对于“订阅”，请选择要在其中创建此配置文件的订阅。</span><span class="sxs-lookup"><span data-stu-id="a12f6-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="a12f6-350">在“资源组”中，为此配置文件创建新的资源组。</span><span class="sxs-lookup"><span data-stu-id="a12f6-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="a12f6-351">在 **资源组位置** 中，选择资源组的位置。</span><span class="sxs-lookup"><span data-stu-id="a12f6-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="a12f6-352">此设置指的是资源组的位置，对全局部署的流量管理器配置文件没有影响。</span><span class="sxs-lookup"><span data-stu-id="a12f6-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="a12f6-353">选择“创建”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-353">Select **Create**.</span></span>

    ![创建流量管理器配置文件](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="a12f6-355">流量管理器配置文件的全局部署完成后，会显示在它所属的资源组的资源列表中。</span><span class="sxs-lookup"><span data-stu-id="a12f6-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="a12f6-356">添加流量管理器终结点</span><span class="sxs-lookup"><span data-stu-id="a12f6-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="a12f6-357">搜索创建的流量管理器配置文件。</span><span class="sxs-lookup"><span data-stu-id="a12f6-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="a12f6-358">如果已导航到配置文件的资源组，请选择该配置文件。</span><span class="sxs-lookup"><span data-stu-id="a12f6-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="a12f6-359">在“流量管理器配置文件”中的“设置”下面，选择“终结点”    。</span><span class="sxs-lookup"><span data-stu-id="a12f6-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="a12f6-360">选择“添加”   。</span><span class="sxs-lookup"><span data-stu-id="a12f6-360">Select **Add**.</span></span>

4. <span data-ttu-id="a12f6-361">在“添加终结点”中，对 Azure Stack Hub 使用以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="a12f6-362">对于“类型”，请选择“外部终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="a12f6-363">为终结点输入 **名称**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="a12f6-364">对于“完全限定的域名(FQDN)或 IP”，请输入 Azure Stack Hub Web 应用的外部 URL。</span><span class="sxs-lookup"><span data-stu-id="a12f6-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="a12f6-365">对于“权重”，请保留默认值 **1**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="a12f6-366">如果此终结点处于正常状态，此权重会使所有流量转到此终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="a12f6-367">将“添加为已禁用”保持未选中状态。</span><span class="sxs-lookup"><span data-stu-id="a12f6-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="a12f6-368">选择“确定”保存 Azure Stack Hub 终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="a12f6-369">接下来将配置 Azure 终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="a12f6-370">在“流量管理器配置文件”中选择“终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="a12f6-371">选择“+添加”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-371">Select **+Add**.</span></span>
3. <span data-ttu-id="a12f6-372">在“添加终结点”中，对 Azure 使用以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="a12f6-373">对于“类型”，请选择“Azure 终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="a12f6-374">为终结点输入 **名称**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="a12f6-375">对于“目标资源类型”，请选择“应用服务”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="a12f6-376">对于“目标资源”，请选择“选择应用服务”以查看同一订阅中的 Web 应用列表。</span><span class="sxs-lookup"><span data-stu-id="a12f6-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="a12f6-377">在“资源”  中，选取要添加为第一个终结点的应用服务。</span><span class="sxs-lookup"><span data-stu-id="a12f6-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="a12f6-378">对于“权重”，请选择 **2**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="a12f6-379">如果主要终结点不正常，或者触发的某个规则/警报会重定向流量，则此设置会使所有流量转到此终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="a12f6-380">将“添加为已禁用”保持未选中状态。</span><span class="sxs-lookup"><span data-stu-id="a12f6-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="a12f6-381">选择“确定”保存 Azure 终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="a12f6-382">配置这两个终结点之后，选择“终结点”时，它们会列在“流量管理器配置文件”中。</span><span class="sxs-lookup"><span data-stu-id="a12f6-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="a12f6-383">以下屏幕截图中的示例显示了两个终结点及其状态和配置信息。</span><span class="sxs-lookup"><span data-stu-id="a12f6-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![流量管理器配置文件中的终结点](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="a12f6-385">在 Azure 中设置 Application Insights 监视和警报</span><span class="sxs-lookup"><span data-stu-id="a12f6-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="a12f6-386">Azure Application Insights 可让你监视应用，并根据配置的条件发送警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="a12f6-387">部分示例包括：应用不可用、遇到故障或出现性能问题。</span><span class="sxs-lookup"><span data-stu-id="a12f6-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="a12f6-388">你将使用 Azure Application Insights 指标创建警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="a12f6-389">这些警报触发时，Web 应用的实例将自动从 Azure Stack Hub 切换到 Azure 以进行扩展，然后切换回到 Azure Stack Hub 以进行缩减。</span><span class="sxs-lookup"><span data-stu-id="a12f6-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="a12f6-390">从指标创建警报</span><span class="sxs-lookup"><span data-stu-id="a12f6-390">Create an alert from metrics</span></span>

<span data-ttu-id="a12f6-391">在 Azure 门户中，转到本教程的资源组，然后选择 Application Insights 实例以打开“Application Insights”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="a12f6-393">你将使用此视图创建横向扩展警报和横向缩减警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="a12f6-394">创建扩展警报</span><span class="sxs-lookup"><span data-stu-id="a12f6-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="a12f6-395">在“配置”下，选择“警报(经典)”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="a12f6-396">选择“添加指标警报(经典)”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="a12f6-397">在“添加规则”中配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="a12f6-398">对于“名称”，请输入“突发到 Azure 云”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="a12f6-399">“说明”是可选字段。</span><span class="sxs-lookup"><span data-stu-id="a12f6-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="a12f6-400">在“源” > “警报依据”下，选择“指标”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="a12f6-401">在“条件”下，选择你的订阅、流量管理器配置文件的资源组，以及资源的流量管理器配置文件名称。</span><span class="sxs-lookup"><span data-stu-id="a12f6-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="a12f6-402">对于“指标”，请选择“请求速率”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="a12f6-403">对于“条件”，请选择“大于”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="a12f6-404">对于“阈值”，请输入 **2**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="a12f6-405">对于“时段”，请选择“过去 5 分钟”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="a12f6-406">在“通知方式”下：</span><span class="sxs-lookup"><span data-stu-id="a12f6-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="a12f6-407">选中“电子邮件所有者、参与者和阅读者”对应的复选框。</span><span class="sxs-lookup"><span data-stu-id="a12f6-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="a12f6-408">在“其他管理员电子邮件”中输入你的电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="a12f6-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="a12f6-409">在菜单栏上选择“保存”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="a12f6-410">创建横向缩减警报</span><span class="sxs-lookup"><span data-stu-id="a12f6-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="a12f6-411">在“配置”下，选择“警报(经典)”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="a12f6-412">选择“添加指标警报(经典)”  。</span><span class="sxs-lookup"><span data-stu-id="a12f6-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="a12f6-413">在“添加规则”中配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="a12f6-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="a12f6-414">对于“名称”，请输入“重新缩减至 Azure Stack Hub”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="a12f6-415">“说明”是可选字段。</span><span class="sxs-lookup"><span data-stu-id="a12f6-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="a12f6-416">在“源” > “警报依据”下，选择“指标”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="a12f6-417">在“条件”下，选择你的订阅、流量管理器配置文件的资源组，以及资源的流量管理器配置文件名称。</span><span class="sxs-lookup"><span data-stu-id="a12f6-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="a12f6-418">对于“指标”，请选择“请求速率”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="a12f6-419">对于“条件”，请选择“小于”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="a12f6-420">对于“阈值”，请输入 **2**。</span><span class="sxs-lookup"><span data-stu-id="a12f6-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="a12f6-421">对于“时段”，请选择“过去 5 分钟”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="a12f6-422">在“通知方式”下：</span><span class="sxs-lookup"><span data-stu-id="a12f6-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="a12f6-423">选中“电子邮件所有者、参与者和阅读者”对应的复选框。</span><span class="sxs-lookup"><span data-stu-id="a12f6-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="a12f6-424">在“其他管理员电子邮件”中输入你的电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="a12f6-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="a12f6-425">在菜单栏上选择“保存”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="a12f6-426">以下屏幕截图显示了横向扩展和横向缩减警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights 警报(经典)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="a12f6-428">在 Azure 与 Azure Stack Hub 之间重定向流量</span><span class="sxs-lookup"><span data-stu-id="a12f6-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-429">可以在 Azure 与 Azure Stack Hub 之间配置手动或自动切换 Web 应用流量。</span><span class="sxs-lookup"><span data-stu-id="a12f6-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="a12f6-430">配置 Azure 与 Azure Stack Hub 之间的手动切换</span><span class="sxs-lookup"><span data-stu-id="a12f6-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-431">当网站达到配置的阈值时，你会收到警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="a12f6-432">使用以下步骤将流量手动重定向到 Azure。</span><span class="sxs-lookup"><span data-stu-id="a12f6-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="a12f6-433">在 Azure 门户中，选择你的流量管理器配置文件。</span><span class="sxs-lookup"><span data-stu-id="a12f6-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure 门户中的流量管理器终结点](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="a12f6-435">选择“终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="a12f6-436">选择“Azure 终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="a12f6-437">在“状态”下面，依次选择“已启用”、“保存”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![在 Azure 门户中启用 Azure 终结点](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="a12f6-439">在流量管理器配置文件的“终结点”中，选择“外部终结点”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="a12f6-440">在“状态”下面，依次选择“已禁用”、“保存”。</span><span class="sxs-lookup"><span data-stu-id="a12f6-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![在 Azure 门户中禁用 Azure Stack Hub 终结点](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="a12f6-442">配置终结点之后，应用流量将转到 Azure 横向扩展 Web 应用，而不是 Azure Stack Hub Web 应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Azure Web 应用流量中更改后的终结点](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="a12f6-444">若要将流量回送到 Azure Stack Hub，请使用上述步骤执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="a12f6-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="a12f6-445">启用 Azure Stack Hub 终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="a12f6-446">禁用 Azure 终结点。</span><span class="sxs-lookup"><span data-stu-id="a12f6-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="a12f6-447">配置 Azure 与 Azure Stack Hub 之间的自动切换</span><span class="sxs-lookup"><span data-stu-id="a12f6-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="a12f6-448">如果应用在 Azure Functions 提供的[无服务器](https://azure.microsoft.com/overview/serverless-computing/)环境中运行，则你还可以使用 Application Insights 监视。</span><span class="sxs-lookup"><span data-stu-id="a12f6-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="a12f6-449">在此方案中，可将 Application Insights 配置为使用调用函数应用的 Webhook。</span><span class="sxs-lookup"><span data-stu-id="a12f6-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="a12f6-450">此应用会自动启用或禁用某个终结点来响应警报。</span><span class="sxs-lookup"><span data-stu-id="a12f6-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="a12f6-451">参考以下步骤配置自动流量切换。</span><span class="sxs-lookup"><span data-stu-id="a12f6-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="a12f6-452">创建 Azure 函数应用。</span><span class="sxs-lookup"><span data-stu-id="a12f6-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="a12f6-453">创建 HTTP 触发的函数。</span><span class="sxs-lookup"><span data-stu-id="a12f6-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="a12f6-454">导入适用于资源管理器、Web 应用和流量管理器的 Azure SDK。</span><span class="sxs-lookup"><span data-stu-id="a12f6-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="a12f6-455">开发执行以下操作的代码：</span><span class="sxs-lookup"><span data-stu-id="a12f6-455">Develop code to:</span></span>

   - <span data-ttu-id="a12f6-456">对 Azure 订阅进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a12f6-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="a12f6-457">使用参数切换流量管理器终结点，以将流量定向到 Azure 或 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="a12f6-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="a12f6-458">保存代码，并将包含相应参数的函数应用的 URL 添加到 Application Insights 警报规则设置的 **Webhook** 节。</span><span class="sxs-lookup"><span data-stu-id="a12f6-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="a12f6-459">当 Application Insights 警报激发时，流量将自动重定向。</span><span class="sxs-lookup"><span data-stu-id="a12f6-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a12f6-460">后续步骤</span><span class="sxs-lookup"><span data-stu-id="a12f6-460">Next steps</span></span>

- <span data-ttu-id="a12f6-461">若要详细了解 Azure 云模式，请参阅[云设计模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="a12f6-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
