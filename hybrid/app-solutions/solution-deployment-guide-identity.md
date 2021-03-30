---
title: 为 Azure 和 Azure Stack Hub 应用配置混合云标识
description: 了解如何为 Azure 和 Azure Stack Hub 应用配置混合云标识。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895340"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="a2e2c-103">为 Azure 和 Azure Stack Hub 应用配置混合云标识</span><span class="sxs-lookup"><span data-stu-id="a2e2c-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="a2e2c-104">了解如何为 Azure 和 Azure Stack Hub 应用配置混合云标识。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="a2e2c-105">在全局 Azure 和 Azure Stack Hub 中，有两个选项可用于授予对应用的访问权限。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="a2e2c-106">当 Azure Stack Hub 与 Internet 建立了不间断的连接时，可以使用 Azure Active Directory (Azure AD)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="a2e2c-107">当 Azure Stack Hub 从 Internet 断开了连接时，可以使用 Azure Directory 联合身份验证服务 (AD FS)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="a2e2c-108">使用服务主体向 Azure Stack Hub 应用授予访问权限，以便在 Azure Stack Hub 中使用 Azure 资源管理器进行部署或配置。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="a2e2c-109">在此解决方案中，你将构建一个示例环境来完成以下任务：</span><span class="sxs-lookup"><span data-stu-id="a2e2c-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="a2e2c-110">在全局 Azure 和 Azure Stack Hub 中建立混合标识</span><span class="sxs-lookup"><span data-stu-id="a2e2c-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="a2e2c-111">检索用于访问 Azure Stack Hub API 的令牌。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="a2e2c-112">必须拥有 Azure Stack Hub 操作员权限才能完成此解决方案中的步骤。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="a2e2c-113">![混合支柱图](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="a2e2c-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="a2e2c-114">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="a2e2c-115">Azure Stack Hub 将云计算的灵活性和创新性带入本地环境，并支持唯一的混合云，允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="a2e2c-116">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="a2e2c-117">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="a2e2c-118">在门户中创建适用于 Azure AD 的服务主体</span><span class="sxs-lookup"><span data-stu-id="a2e2c-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="a2e2c-119">如果已使用 Azure AD 部署 Azure Stack Hub 作为标识存储，则可以创建服务主体，就像对 Azure 所做的那样。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="a2e2c-120">[使用应用标识访问资源](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity)介绍了如何通过门户执行这些步骤。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="a2e2c-121">在开始之前，请确保你具有[所需的 Azure AD 权限](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="a2e2c-122">使用 PowerShell 创建适用于 AD FS 的服务主体</span><span class="sxs-lookup"><span data-stu-id="a2e2c-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="a2e2c-123">如果已使用 AD FS 部署 Azure Stack Hub，则可以使用 PowerShell 创建服务主体、为角色分配访问权限以及使用该标识从 PowerShell 登录。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="a2e2c-124">[使用应用标识访问资源](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity)介绍了如何使用 PowerShell 执行所需步骤。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="a2e2c-125">使用 Azure Stack Hub API</span><span class="sxs-lookup"><span data-stu-id="a2e2c-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="a2e2c-126">[Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use) 解决方案将引导你完成检索令牌以访问 Azure Stack Hub API 的过程。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="a2e2c-127">使用 PowerShell 连接到 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="a2e2c-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="a2e2c-128">[在 Azure Stack Hub 中使用 PowerShell 启动并运行](/azure-stack/operator/azure-stack-powershell-install)快速入门演练了安装 Azure PowerShell 并连接到 Azure Stack Hub 安装所要执行的步骤。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a2e2c-129">必备条件</span><span class="sxs-lookup"><span data-stu-id="a2e2c-129">Prerequisites</span></span>

<span data-ttu-id="a2e2c-130">你需要已连接到 Azure AD 的 Azure Stack Hub 安装，以及可访问的订阅。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="a2e2c-131">如果没有 Azure Stack Hub 安装，则可以按照以下说明设置 [Azure Stack 开发工具包 (ASDK)](/azure-stack/asdk/asdk-install)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="a2e2c-132">使用代码连接到 Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="a2e2c-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="a2e2c-133">若要使用代码连接到 Azure Stack Hub，请使用 Azure 资源管理器终结点 API 获取 Azure Stack Hub 安装的身份验证和图终结点。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="a2e2c-134">然后使用 REST 请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="a2e2c-135">可以在 [GitHub](https://github.com/shriramnat/HybridARMApplication) 上找到示例客户端应用程序。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="a2e2c-136">除非所选语言的 Azure SDK 支持 Azure API 配置文件，否则该 SDK 可能无法与 Azure Stack Hub 配合使用。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="a2e2c-137">若要了解有关 Azure API 配置文件的详细信息，请参阅[管理 API 版本配置文件](/azure-stack/user/azure-stack-version-profiles)一文。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="a2e2c-138">后续步骤</span><span class="sxs-lookup"><span data-stu-id="a2e2c-138">Next steps</span></span>

- <span data-ttu-id="a2e2c-139">若要了解有关如何在 Azure Stack Hub 中处理标识的详细信息，请参阅 [Azure Stack Hub 的标识体系结构](/azure-stack/operator/azure-stack-identity-architecture)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="a2e2c-140">若要详细了解 Azure 云模式，请参阅[云设计模式](/azure/architecture/patterns)。</span><span class="sxs-lookup"><span data-stu-id="a2e2c-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
