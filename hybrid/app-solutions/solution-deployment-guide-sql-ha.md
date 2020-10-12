---
title: 将 SQL Server 2016 可用性组部署到 Azure 和 Azure Stack Hub
description: 了解如何将 SQL Server 2016 可用性组部署到 Azure 和 Azure Stack Hub。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852467"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="aad9e-103">将 SQL Server 2016 可用性组部署到 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="aad9e-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="aad9e-104">本文逐步指导你跨两个 Azure Stack Hub 环境，通过异步的灾难恢复 (DR) 站点自动部署基本的高度可用 (HA) SQL Server 2016 Enterprise 群集。</span><span class="sxs-lookup"><span data-stu-id="aad9e-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="aad9e-105">有关 SQL Server 2016 和高可用性的详细信息，请参阅 [Always On 可用性组：高可用性和灾难恢复解决方案](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="aad9e-106">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="aad9e-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="aad9e-107">跨两个 Azure Stack Hub 对部署进行协调。</span><span class="sxs-lookup"><span data-stu-id="aad9e-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="aad9e-108">使用 Docker 尽量减少使用 Azure API 配置文件时出现的依赖项问题。</span><span class="sxs-lookup"><span data-stu-id="aad9e-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="aad9e-109">通过灾难恢复站点部署基本的高可用性 SQL Server 2016 Enterprise 群集。</span><span class="sxs-lookup"><span data-stu-id="aad9e-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="aad9e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="aad9e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="aad9e-111">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="aad9e-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="aad9e-112">Azure Stack Hub 将云计算的灵活性和创新性带入本地环境，并支持唯一的混合云，允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="aad9e-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="aad9e-113">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="aad9e-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="aad9e-114">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="aad9e-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="aad9e-115">2016 SQL Server 的体系结构</span><span class="sxs-lookup"><span data-stu-id="aad9e-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack Hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="aad9e-117">SQL Server 2016 的先决条件</span><span class="sxs-lookup"><span data-stu-id="aad9e-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="aad9e-118">两个连接的 Azure Stack Hub 集成系统 (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="aad9e-119">此部署不适用于 Azure Stack 开发工具包 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="aad9e-120">若要详细了解 Azure Stack Hub，请参阅 [Azure Stack 概述](https://azure.microsoft.com/overview/azure-stack/)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="aad9e-121">每个 Azure Stack Hub 上有一个租户订阅。</span><span class="sxs-lookup"><span data-stu-id="aad9e-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="aad9e-122">**记下每个订阅 ID 以及每个 Azure Stack Hub 的 Azure 资源管理器终结点。**</span><span class="sxs-lookup"><span data-stu-id="aad9e-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="aad9e-123">一个 Azure Active Directory (Azure AD) 服务主体，该主体有权访问每个 Azure Stack Hub 上的租户订阅。</span><span class="sxs-lookup"><span data-stu-id="aad9e-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="aad9e-124">如果 Azure Stack Hub 是针对不同 Azure AD 租户部署的，则可能需要创建两个服务主体。</span><span class="sxs-lookup"><span data-stu-id="aad9e-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="aad9e-125">若要了解如何为 Azure Stack Hub 创建服务主体，请参阅[创建为应用提供 Azure Stack Hub 资源访问权限的服务主体](/azure-stack/user/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="aad9e-126">**记下每个服务主体的应用程序 ID、客户端密码和租户名称 (xxxxx.onmicrosoft.com)。**</span><span class="sxs-lookup"><span data-stu-id="aad9e-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="aad9e-127">SQL Server 2016 Enterprise 已联合到 Azure Stack Hub 的每个市场。</span><span class="sxs-lookup"><span data-stu-id="aad9e-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="aad9e-128">若要详细了解市场联合，请参阅[将市场项下载到 Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="aad9e-129">**请确保你的组织具有适当的 SQL 许可证。**</span><span class="sxs-lookup"><span data-stu-id="aad9e-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="aad9e-130">在本地计算机上安装[用于 Windows 的 Docker](https://docs.docker.com/docker-for-windows/)。</span><span class="sxs-lookup"><span data-stu-id="aad9e-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="aad9e-131">获取 Docker 映像</span><span class="sxs-lookup"><span data-stu-id="aad9e-131">Get the Docker image</span></span>

<span data-ttu-id="aad9e-132">适用于每个部署的 Docker 映像消除了不同版 Azure PowerShell 之间出现的依赖项问题。</span><span class="sxs-lookup"><span data-stu-id="aad9e-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="aad9e-133">确保用于 Windows 的 Docker 使用 Windows 容器。</span><span class="sxs-lookup"><span data-stu-id="aad9e-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="aad9e-134">在提升的命令提示符下运行以下脚本，获取包含部署脚本的 Docker 容器。</span><span class="sxs-lookup"><span data-stu-id="aad9e-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="aad9e-135">部署可用性组</span><span class="sxs-lookup"><span data-stu-id="aad9e-135">Deploy the availability group</span></span>

1. <span data-ttu-id="aad9e-136">成功拉取容器映像以后，即可启动映像。</span><span class="sxs-lookup"><span data-stu-id="aad9e-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="aad9e-137">容器启动以后，系统会在容器中为你提供提升的 PowerShell 终端。</span><span class="sxs-lookup"><span data-stu-id="aad9e-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="aad9e-138">将目录切换到部署脚本的位置。</span><span class="sxs-lookup"><span data-stu-id="aad9e-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="aad9e-139">运行此部署。</span><span class="sxs-lookup"><span data-stu-id="aad9e-139">Run the deployment.</span></span> <span data-ttu-id="aad9e-140">根据需要提供凭据和资源名称。</span><span class="sxs-lookup"><span data-stu-id="aad9e-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="aad9e-141">HA 是指将在其中部署 HA 群集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="aad9e-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="aad9e-142">DR 是指将在其中部署 DR 群集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="aad9e-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="aad9e-143">键入 `Y` 以安装 NuGet 提供程序，然后就会安装 API 配置文件“2018-03-01-hybrid”模块。</span><span class="sxs-lookup"><span data-stu-id="aad9e-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="aad9e-144">等待资源部署完成。</span><span class="sxs-lookup"><span data-stu-id="aad9e-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="aad9e-145">等 DR 资源部署完成后，退出容器。</span><span class="sxs-lookup"><span data-stu-id="aad9e-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="aad9e-146">查看每个 Azure Stack Hub 门户中的资源以检查部署。</span><span class="sxs-lookup"><span data-stu-id="aad9e-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="aad9e-147">连接到 HA 环境中的某个 SQL 实例，并通过 SQL Server Management Studio (SSMS) 检查可用性组。</span><span class="sxs-lookup"><span data-stu-id="aad9e-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="aad9e-149">后续步骤</span><span class="sxs-lookup"><span data-stu-id="aad9e-149">Next steps</span></span>

- <span data-ttu-id="aad9e-150">使用 SQL Server Management Studio 手动故障转移群集。</span><span class="sxs-lookup"><span data-stu-id="aad9e-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="aad9e-151">请参阅[执行 Always On 可用性组的强制手动故障转移 (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="aad9e-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="aad9e-152">详细了解混合云应用。</span><span class="sxs-lookup"><span data-stu-id="aad9e-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="aad9e-153">请参阅[混合云解决方案。](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="aad9e-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="aad9e-154">使用自己的数据，或修改 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上此示例的代码。</span><span class="sxs-lookup"><span data-stu-id="aad9e-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>