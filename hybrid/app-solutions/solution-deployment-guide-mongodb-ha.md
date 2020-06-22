---
title: 将高度可用的 MongoDB 解决方案部署到 Azure 和 Azure Stack Hub
description: 了解如何将高度可用的 MongoDB 解决方案部署到 Azure 和 Azure Stack Hub
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909992"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="49976-103">将高度可用的 MongoDB 解决方案部署到 Azure 和 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="49976-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="49976-104">本文详细介绍如何跨两个 Azure Stack Hub 环境通过灾难恢复 (DR) 站点来自动部署基本的高度可用 (HA) MongoDB 群集。</span><span class="sxs-lookup"><span data-stu-id="49976-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="49976-105">若要详细了解 MongoDB 和高可用性，请参阅 [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/)（副本集成员）。</span><span class="sxs-lookup"><span data-stu-id="49976-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="49976-106">在此解决方案中，你将创建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="49976-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="49976-107">在两个 Azure Stack Hub 之间协调部署。</span><span class="sxs-lookup"><span data-stu-id="49976-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="49976-108">使用 Docker 最大限度减少使用 Azure API 配置文件时出现的依赖项问题。</span><span class="sxs-lookup"><span data-stu-id="49976-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="49976-109">使用灾难恢复站点部署基本的高度可用的 MongoDB 群集。</span><span class="sxs-lookup"><span data-stu-id="49976-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="49976-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="49976-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="49976-111">Microsoft Azure Stack 中心是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="49976-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="49976-112">Azure Stack Hub 将云计算的灵活性和创新性带入本地环境，并支持唯一的混合云，允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="49976-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="49976-113">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="49976-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="49976-114">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="49976-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="49976-115">Azure Stack Hub 中的 MongoDB 的体系结构</span><span class="sxs-lookup"><span data-stu-id="49976-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![Azure Stack Hub 中高度可用的 MongoDB 体系结构](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="49976-117">将 MongoDB 与 Azure Stack Hub 配合使用的先决条件</span><span class="sxs-lookup"><span data-stu-id="49976-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="49976-118">两个连接的 Azure Stack Hub 集成系统 (Azure Stack Hub)。</span><span class="sxs-lookup"><span data-stu-id="49976-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="49976-119">此部署不适用于 Azure Stack 开发工具包 (ASDK)。</span><span class="sxs-lookup"><span data-stu-id="49976-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="49976-120">若要详细了解 Azure Stack 中心，请参阅[什么是 Azure Stack 中心？](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="49976-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="49976-121">每个 Azure Stack Hub 上有一个租户订阅。</span><span class="sxs-lookup"><span data-stu-id="49976-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="49976-122">**记下每个订阅 ID 以及每个 Azure Stack Hub 的 Azure 资源管理器终结点。**</span><span class="sxs-lookup"><span data-stu-id="49976-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="49976-123">一个 Azure Active Directory (Azure AD) 服务主体，该主体有权访问每个 Azure Stack Hub 上的租户订阅。</span><span class="sxs-lookup"><span data-stu-id="49976-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="49976-124">如果 Azure Stack Hub 是针对不同 Azure AD 租户部署的，则可能需要创建两个服务主体。</span><span class="sxs-lookup"><span data-stu-id="49976-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="49976-125">若要了解如何为 Azure Stack Hub 创建服务主体，请参阅[使用应用标识访问 Azure Stack Hub 资源](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="49976-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="49976-126">**记下每个服务主体的应用程序 ID、客户端机密和租户名称（xxxxx.onmicrosoft.com）。**</span><span class="sxs-lookup"><span data-stu-id="49976-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="49976-127">与每个 Azure Stack Hub 市场联合的 Ubuntu 16.04。</span><span class="sxs-lookup"><span data-stu-id="49976-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="49976-128">若要详细了解市场联合，请参阅[将市场项下载到 Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item)。</span><span class="sxs-lookup"><span data-stu-id="49976-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="49976-129">在本地计算机上安装[用于 Windows 的 Docker](https://docs.docker.com/docker-for-windows/)。</span><span class="sxs-lookup"><span data-stu-id="49976-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="49976-130">获取 Docker 映像</span><span class="sxs-lookup"><span data-stu-id="49976-130">Get the Docker image</span></span>

<span data-ttu-id="49976-131">适用于每个部署的 Docker 映像消除了不同版 Azure PowerShell 之间出现的依赖项问题。</span><span class="sxs-lookup"><span data-stu-id="49976-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="49976-132">确保用于 Windows 的 Docker 使用 Windows 容器。</span><span class="sxs-lookup"><span data-stu-id="49976-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="49976-133">在提升的命令提示符下运行以下命令，获取包含部署脚本的 Docker 容器。</span><span class="sxs-lookup"><span data-stu-id="49976-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="49976-134">部署群集</span><span class="sxs-lookup"><span data-stu-id="49976-134">Deploy the clusters</span></span>

1. <span data-ttu-id="49976-135">成功拉取容器映像以后，即可启动映像。</span><span class="sxs-lookup"><span data-stu-id="49976-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="49976-136">容器启动以后，系统会在容器中为你提供提升的 PowerShell 终端。</span><span class="sxs-lookup"><span data-stu-id="49976-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="49976-137">将目录切换到部署脚本的位置。</span><span class="sxs-lookup"><span data-stu-id="49976-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="49976-138">运行此部署。</span><span class="sxs-lookup"><span data-stu-id="49976-138">Run the deployment.</span></span> <span data-ttu-id="49976-139">根据需要提供凭据和资源名称。</span><span class="sxs-lookup"><span data-stu-id="49976-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="49976-140">HA 是指将在其中部署 HA 群集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="49976-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="49976-141">DR 是指将在其中部署 DR 群集的 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="49976-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="49976-142">键入 `Y` 以安装 NuGet 提供程序，然后就会安装 API 配置文件“2018-03-01-hybrid”模块。</span><span class="sxs-lookup"><span data-stu-id="49976-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="49976-143">HA 资源会先部署。</span><span class="sxs-lookup"><span data-stu-id="49976-143">The HA resources will deploy first.</span></span> <span data-ttu-id="49976-144">监视部署，等待部署完成。</span><span class="sxs-lookup"><span data-stu-id="49976-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="49976-145">出现指示 HA 部署已完成的消息后，即可检查 HA Azure Stack Hub 的门户，查看已部署的资源。</span><span class="sxs-lookup"><span data-stu-id="49976-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="49976-146">继续部署 DR 资源，决定是否要在 DR Azure Stack Hub 上启用与群集交互所需的跳转盒。</span><span class="sxs-lookup"><span data-stu-id="49976-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="49976-147">等待 DR 资源部署完成。</span><span class="sxs-lookup"><span data-stu-id="49976-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="49976-148">DR 资源部署完成后，退出容器。</span><span class="sxs-lookup"><span data-stu-id="49976-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="49976-149">后续步骤</span><span class="sxs-lookup"><span data-stu-id="49976-149">Next steps</span></span>

- <span data-ttu-id="49976-150">如果在 DR Azure Stack Hub 上启用了跳转盒，则可通过 SSH 进行连接，并通过安装 mongo CLI 与 MongoDB 群集进行交互。</span><span class="sxs-lookup"><span data-stu-id="49976-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="49976-151">若要详细了解如何与 MongoDB 交互，请参阅 [The mongo Shell](https://docs.mongodb.com/manual/mongo/)（mongo Shell）。</span><span class="sxs-lookup"><span data-stu-id="49976-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="49976-152">若要详细了解混合云应用，请参阅[混合云解决方案](https://aka.ms/azsdevtutorials)。</span><span class="sxs-lookup"><span data-stu-id="49976-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="49976-153">在 [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns) 上修改此示例的代码。</span><span class="sxs-lookup"><span data-stu-id="49976-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
