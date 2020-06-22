---
title: 在 Azure 中部署基于 AI 的 footfall 检测解决方案并 Azure Stack 中心
description: 了解如何部署基于 AI 的 footfall 检测解决方案，以便使用 Azure 和 Azure Stack 集线器分析零售店在零售商店中的流量。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909913"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="8912d-103">使用 Azure 和 Azure Stack 中心部署基于 AI 的 footfall 检测解决方案</span><span class="sxs-lookup"><span data-stu-id="8912d-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="8912d-104">本文介绍如何部署基于 AI 的解决方案，该解决方案使用 Azure、Azure Stack 中心和自定义视觉 AI 开发工具包从实际操作生成见解。</span><span class="sxs-lookup"><span data-stu-id="8912d-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="8912d-105">在此解决方案中，你将了解如何执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="8912d-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="8912d-106">在边缘部署云本机应用程序捆绑包（CNAB）。</span><span class="sxs-lookup"><span data-stu-id="8912d-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="8912d-107">部署跨云边界的应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="8912d-108">使用自定义视觉 AI 开发工具包在边缘进行推断。</span><span class="sxs-lookup"><span data-stu-id="8912d-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="8912d-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="8912d-109">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="8912d-110">Microsoft Azure Stack 中心是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="8912d-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="8912d-111">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="8912d-112">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="8912d-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="8912d-113">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="8912d-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="8912d-114">先决条件</span><span class="sxs-lookup"><span data-stu-id="8912d-114">Prerequisites</span></span>

<span data-ttu-id="8912d-115">在开始本部署指南之前，请确保：</span><span class="sxs-lookup"><span data-stu-id="8912d-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="8912d-116">查看[Footfall 检测模式](pattern-retail-footfall-detection.md)主题。</span><span class="sxs-lookup"><span data-stu-id="8912d-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="8912d-117">获取用户对 Azure Stack 开发工具包（ASDK）或 Azure Stack 中心集成系统实例的访问权限，其中包括：</span><span class="sxs-lookup"><span data-stu-id="8912d-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="8912d-118">[Azure Stack 中心资源提供程序上](/azure-stack/operator/azure-stack-app-service-overview.md)安装了 Azure App Service。</span><span class="sxs-lookup"><span data-stu-id="8912d-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="8912d-119">需要操作员访问 Azure Stack 中心实例，或与管理员合作来安装。</span><span class="sxs-lookup"><span data-stu-id="8912d-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="8912d-120">提供应用服务和存储配额的产品/服务的订阅。</span><span class="sxs-lookup"><span data-stu-id="8912d-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="8912d-121">需要操作员访问权限才能创建产品/服务。</span><span class="sxs-lookup"><span data-stu-id="8912d-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="8912d-122">获取对 Azure 订阅的访问权限。</span><span class="sxs-lookup"><span data-stu-id="8912d-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="8912d-123">如果你没有 Azure 订阅，请在开始前注册一个[免费试用帐户](https://azure.microsoft.com/free/)。</span><span class="sxs-lookup"><span data-stu-id="8912d-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="8912d-124">在目录中创建两个服务主体：</span><span class="sxs-lookup"><span data-stu-id="8912d-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="8912d-125">一种设置，可与 Azure 资源一起使用，并可通过 Azure 订阅作用域进行访问。</span><span class="sxs-lookup"><span data-stu-id="8912d-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="8912d-126">一个设置用于 Azure Stack 中心资源，并具有 Azure Stack 中心订阅范围内的访问权限。</span><span class="sxs-lookup"><span data-stu-id="8912d-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="8912d-127">若要详细了解如何创建服务主体和授权访问，请参阅[使用应用标识访问资源](/azure-stack/operator/azure-stack-create-service-principals.md)。</span><span class="sxs-lookup"><span data-stu-id="8912d-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="8912d-128">如果希望使用 Azure CLI，请参阅[使用 Azure CLI 创建 Azure 服务主体](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)。</span><span class="sxs-lookup"><span data-stu-id="8912d-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest).</span></span>
- <span data-ttu-id="8912d-129">在 Azure 或 Azure Stack 中心部署 Azure 认知服务。</span><span class="sxs-lookup"><span data-stu-id="8912d-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="8912d-130">首先，[详细了解认知服务](https://azure.microsoft.com/services/cognitive-services/)。</span><span class="sxs-lookup"><span data-stu-id="8912d-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="8912d-131">然后，请访问将[Azure 认知服务部署到 Azure Stack 中心](/azure-stack/user/azure-stack-solution-template-cognitive-services.md)，在 Azure Stack 中心部署认知服务。</span><span class="sxs-lookup"><span data-stu-id="8912d-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="8912d-132">首先需要注册预览版的访问权限。</span><span class="sxs-lookup"><span data-stu-id="8912d-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="8912d-133">克隆或下载未配置的 Azure 自定义视觉 AI 开发工具包。</span><span class="sxs-lookup"><span data-stu-id="8912d-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="8912d-134">有关详细信息，请参阅[远景 AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/)。</span><span class="sxs-lookup"><span data-stu-id="8912d-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="8912d-135">注册 Power BI 帐户。</span><span class="sxs-lookup"><span data-stu-id="8912d-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="8912d-136">Azure 认知服务人脸 API 订阅密钥和终结点 URL。</span><span class="sxs-lookup"><span data-stu-id="8912d-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="8912d-137">可以通过试用[认知服务](https://azure.microsoft.com/try/cognitive-services/?api=face-api)免费试用版。</span><span class="sxs-lookup"><span data-stu-id="8912d-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="8912d-138">或者，按照[创建认知服务帐户](/azure/cognitive-services/cognitive-services-apis-create-account)中的说明进行操作。</span><span class="sxs-lookup"><span data-stu-id="8912d-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="8912d-139">安装以下开发资源：</span><span class="sxs-lookup"><span data-stu-id="8912d-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="8912d-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="8912d-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="8912d-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="8912d-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="8912d-142">[Porter](https://porter.sh/)。</span><span class="sxs-lookup"><span data-stu-id="8912d-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="8912d-143">你可以使用 Porter 来部署使用为你提供的 CNAB 捆绑清单的云应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="8912d-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="8912d-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="8912d-145">适用于 Visual Studio Code 的 Azure IoT 工具</span><span class="sxs-lookup"><span data-stu-id="8912d-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="8912d-146">适用于 Visual Studio Code 的 Python 扩展</span><span class="sxs-lookup"><span data-stu-id="8912d-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="8912d-147">Python</span><span class="sxs-lookup"><span data-stu-id="8912d-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="8912d-148">部署混合云应用</span><span class="sxs-lookup"><span data-stu-id="8912d-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="8912d-149">首先，使用 Porter CLI 生成凭据集，然后部署云应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="8912d-150">从中克隆或下载解决方案示例代码 https://github.com/azure-samples/azure-intelligent-edge-patterns 。</span><span class="sxs-lookup"><span data-stu-id="8912d-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="8912d-151">Porter 将生成一组将自动部署应用的凭据。</span><span class="sxs-lookup"><span data-stu-id="8912d-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="8912d-152">在运行凭据生成命令之前，请确保具有以下各项：</span><span class="sxs-lookup"><span data-stu-id="8912d-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="8912d-153">用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="8912d-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8912d-154">你的 Azure 订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="8912d-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="8912d-155">用于访问 Azure Stack 集线器资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="8912d-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8912d-156">Azure Stack 中心订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="8912d-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="8912d-157">Azure 认知服务人脸 API 密钥和资源终结点 URL。</span><span class="sxs-lookup"><span data-stu-id="8912d-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="8912d-158">运行 Porter credential 生成过程，并按照提示进行操作：</span><span class="sxs-lookup"><span data-stu-id="8912d-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="8912d-159">Porter 还需要一组要运行的参数。</span><span class="sxs-lookup"><span data-stu-id="8912d-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="8912d-160">创建参数文本文件，并输入以下名称/值对。</span><span class="sxs-lookup"><span data-stu-id="8912d-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="8912d-161">如果需要有关任何所需值的帮助，请咨询 Azure Stack 中心管理员。</span><span class="sxs-lookup"><span data-stu-id="8912d-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="8912d-162">此 `resource suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。</span><span class="sxs-lookup"><span data-stu-id="8912d-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="8912d-163">它必须是字母和数字的唯一字符串，不能超过8个字符。</span><span class="sxs-lookup"><span data-stu-id="8912d-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="8912d-164">保存文本文件并记下其路径。</span><span class="sxs-lookup"><span data-stu-id="8912d-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="8912d-165">现已准备好使用 Porter 部署混合云应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="8912d-166">运行 install 命令并监视资源是否部署到 Azure 和 Azure Stack 中心：</span><span class="sxs-lookup"><span data-stu-id="8912d-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="8912d-167">部署完成后，请记下以下值：</span><span class="sxs-lookup"><span data-stu-id="8912d-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="8912d-168">照相机的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8912d-168">The camera's connection string.</span></span>
    - <span data-ttu-id="8912d-169">映像存储帐户连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8912d-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="8912d-170">资源组名称。</span><span class="sxs-lookup"><span data-stu-id="8912d-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="8912d-171">准备自定义视觉 AI DevKit</span><span class="sxs-lookup"><span data-stu-id="8912d-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="8912d-172">接下来，设置自定义视觉 AI 开发工具包，如[远景 Ai DevKit 快速入门](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)中所示。</span><span class="sxs-lookup"><span data-stu-id="8912d-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="8912d-173">你还可以使用上一步骤中提供的连接字符串来设置和测试相机。</span><span class="sxs-lookup"><span data-stu-id="8912d-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="8912d-174">部署照相机应用</span><span class="sxs-lookup"><span data-stu-id="8912d-174">Deploy the camera app</span></span>

<span data-ttu-id="8912d-175">使用 Porter CLI 生成凭据集，然后部署相机应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="8912d-176">Porter 将生成一组将自动部署应用的凭据。</span><span class="sxs-lookup"><span data-stu-id="8912d-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="8912d-177">在运行凭据生成命令之前，请确保具有以下各项：</span><span class="sxs-lookup"><span data-stu-id="8912d-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="8912d-178">用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="8912d-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="8912d-179">你的 Azure 订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="8912d-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="8912d-180">在部署云应用时提供的映像存储帐户连接字符串。</span><span class="sxs-lookup"><span data-stu-id="8912d-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="8912d-181">运行 Porter credential 生成过程，并按照提示进行操作：</span><span class="sxs-lookup"><span data-stu-id="8912d-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="8912d-182">Porter 还需要一组要运行的参数。</span><span class="sxs-lookup"><span data-stu-id="8912d-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="8912d-183">创建参数文本文件，并输入以下文本。</span><span class="sxs-lookup"><span data-stu-id="8912d-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="8912d-184">如果您不知道某些必需的值，请向您的 Azure Stack 中心管理员咨询。</span><span class="sxs-lookup"><span data-stu-id="8912d-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="8912d-185">此 `deployment suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。</span><span class="sxs-lookup"><span data-stu-id="8912d-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="8912d-186">它必须是字母和数字的唯一字符串，不能超过8个字符。</span><span class="sxs-lookup"><span data-stu-id="8912d-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="8912d-187">保存文本文件并记下其路径。</span><span class="sxs-lookup"><span data-stu-id="8912d-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="8912d-188">现已准备好使用 Porter 部署相机应用。</span><span class="sxs-lookup"><span data-stu-id="8912d-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="8912d-189">运行 "安装" 命令，并在创建 IoT Edge 部署时监视。</span><span class="sxs-lookup"><span data-stu-id="8912d-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="8912d-190">通过查看上的相机源来验证相机的部署是否已完成 `https://<camera-ip>:3000/` ，其中 `<camara-ip>` 是相机 IP 地址。</span><span class="sxs-lookup"><span data-stu-id="8912d-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="8912d-191">此步骤可能需要长达10分钟的时间。</span><span class="sxs-lookup"><span data-stu-id="8912d-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="8912d-192">配置 Azure 流分析</span><span class="sxs-lookup"><span data-stu-id="8912d-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="8912d-193">现在，数据已从照相机流向 Azure 流分析，我们需要手动授权它与 Power BI 通信。</span><span class="sxs-lookup"><span data-stu-id="8912d-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="8912d-194">在 Azure 门户中，打开 "**所有资源**"，并打开\*footfall \[ yoursuffix \] \*作业。</span><span class="sxs-lookup"><span data-stu-id="8912d-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="8912d-195">在“流分析作业”窗格的“作业拓扑”  部分，选择“输出”  选项。</span><span class="sxs-lookup"><span data-stu-id="8912d-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="8912d-196">选择**流量输出**输出接收器。</span><span class="sxs-lookup"><span data-stu-id="8912d-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="8912d-197">选择 "**续订授权**"，并登录到你的 Power BI 帐户。</span><span class="sxs-lookup"><span data-stu-id="8912d-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![在 Power BI 中续订授权提示](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="8912d-199">保存输出设置。</span><span class="sxs-lookup"><span data-stu-id="8912d-199">Save the output settings.</span></span>

6. <span data-ttu-id="8912d-200">请在 "**概述**" 窗格中，选择 "**开始**"，开始将数据发送到 Power BI。</span><span class="sxs-lookup"><span data-stu-id="8912d-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="8912d-201">选择“现在”作为作业输出启动时间，然后选择“启动”。  </span><span class="sxs-lookup"><span data-stu-id="8912d-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="8912d-202">可以在通知栏中查看作业状态。</span><span class="sxs-lookup"><span data-stu-id="8912d-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="8912d-203">创建 Power BI 仪表板</span><span class="sxs-lookup"><span data-stu-id="8912d-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="8912d-204">作业成功后，请前往[Power BI](https://powerbi.com/)并使用工作或学校帐户登录。</span><span class="sxs-lookup"><span data-stu-id="8912d-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="8912d-205">如果流分析作业查询输出了结果，则您创建的*footfall*数据集在 "**数据**集" 选项卡下。</span><span class="sxs-lookup"><span data-stu-id="8912d-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="8912d-206">在 Power BI 工作区中，选择 " **+ 创建**" 创建名为 " *Footfall 分析*" 的新仪表板。</span><span class="sxs-lookup"><span data-stu-id="8912d-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="8912d-207">在窗口顶部，选择“添加磁贴”  。</span><span class="sxs-lookup"><span data-stu-id="8912d-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="8912d-208">然后选择“自定义流数据”和“下一步”。  </span><span class="sxs-lookup"><span data-stu-id="8912d-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="8912d-209">选择**数据**集下的**footfall 数据集**。</span><span class="sxs-lookup"><span data-stu-id="8912d-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="8912d-210">从 "**可视化效果类型**" 下拉列表中选择 "**卡**"，并向**字段**添加**age** 。</span><span class="sxs-lookup"><span data-stu-id="8912d-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="8912d-211">选择“下一步”  ，为磁贴输入一个名称，然后选择“应用”，创建该磁贴。 </span><span class="sxs-lookup"><span data-stu-id="8912d-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="8912d-212">您可以根据需要添加其他字段和卡。</span><span class="sxs-lookup"><span data-stu-id="8912d-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="8912d-213">测试你的解决方案</span><span class="sxs-lookup"><span data-stu-id="8912d-213">Test Your Solution</span></span>

<span data-ttu-id="8912d-214">观察你在中创建的卡中的数据如何在不同的人在相机前面进行 Power BI 更改。</span><span class="sxs-lookup"><span data-stu-id="8912d-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="8912d-215">记录后，推断可能需要长达20秒的时间。</span><span class="sxs-lookup"><span data-stu-id="8912d-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="8912d-216">删除你的解决方案</span><span class="sxs-lookup"><span data-stu-id="8912d-216">Remove Your Solution</span></span>

<span data-ttu-id="8912d-217">如果要删除解决方案，请使用 Porter 运行以下命令，并使用为部署创建的相同参数文件：</span><span class="sxs-lookup"><span data-stu-id="8912d-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="8912d-218">后续步骤</span><span class="sxs-lookup"><span data-stu-id="8912d-218">Next steps</span></span>

- <span data-ttu-id="8912d-219">了解有关 [混合应用设计注意事项] 的详细信息。（overview-app-design-considerations.md）</span><span class="sxs-lookup"><span data-stu-id="8912d-219">Learn more about [Hybrid app design considerations].(overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="8912d-220">查看并建议[在 GitHub 上改进此示例的代码](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)。</span><span class="sxs-lookup"><span data-stu-id="8912d-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
