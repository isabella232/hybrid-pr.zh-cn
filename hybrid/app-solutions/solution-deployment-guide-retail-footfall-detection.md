---
title: 在 Azure 和 Azure Stack Hub 中部署基于 AI 的客流量检测解决方案
description: 了解如何使用 Azure 和 Azure Stack Hub 部署基于 AI 的客流量检测解决方案，以分析零售店的客流量。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: caedbd4758b9ae8c93cf9bb625ed9aac68bfa196
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895353"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="da451-103">使用 Azure 和 Azure Stack Hub 部署基于 AI 的客流量检测解决方案</span><span class="sxs-lookup"><span data-stu-id="da451-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="da451-104">本文介绍如何部署基于 AI 的解决方案，该解决方案使用 Azure、Azure Stack Hub 和自定义视觉 AI 开发工具包从实际操作生成见解。</span><span class="sxs-lookup"><span data-stu-id="da451-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="da451-105">在此解决方案中，你将了解如何：</span><span class="sxs-lookup"><span data-stu-id="da451-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="da451-106">在边缘部署云本机应用程序捆绑包 (CNAB)。</span><span class="sxs-lookup"><span data-stu-id="da451-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="da451-107">部署跨越云边界的应用。</span><span class="sxs-lookup"><span data-stu-id="da451-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="da451-108">使用自定义视觉 AI 开发工具包在边缘进行推理。</span><span class="sxs-lookup"><span data-stu-id="da451-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="da451-109">![混合支柱图](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="da451-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="da451-110">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="da451-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="da451-111">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="da451-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="da451-112">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="da451-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="da451-113">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="da451-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="da451-114">先决条件</span><span class="sxs-lookup"><span data-stu-id="da451-114">Prerequisites</span></span>

<span data-ttu-id="da451-115">在开始使用此部署指南之前，请确保：</span><span class="sxs-lookup"><span data-stu-id="da451-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="da451-116">查看[客流量检测模式](pattern-retail-footfall-detection.md)主题。</span><span class="sxs-lookup"><span data-stu-id="da451-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="da451-117">在具备以下条件的情况下，获取对 Azure Stack 开发工具包 (ASDK) 或 Azure Stack Hub 集成系统实例的用户访问权限：</span><span class="sxs-lookup"><span data-stu-id="da451-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="da451-118">已安装 [Azure Stack Hub 资源提供程序上的 Azure 应用服务](/azure-stack/operator/azure-stack-app-service-overview)。</span><span class="sxs-lookup"><span data-stu-id="da451-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview) installed.</span></span> <span data-ttu-id="da451-119">需要具有对 Azure Stack Hub 实例的操作员访问权限，或与管理员合作进行安装。</span><span class="sxs-lookup"><span data-stu-id="da451-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="da451-120">订阅可提供应用服务和存储配额的套餐。</span><span class="sxs-lookup"><span data-stu-id="da451-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="da451-121">需要具有操作员访问权限才能创建套餐。</span><span class="sxs-lookup"><span data-stu-id="da451-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="da451-122">获取对 Azure 订阅的访问权限。</span><span class="sxs-lookup"><span data-stu-id="da451-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="da451-123">如果还没有 Azure 订阅，可以在开始前注册一个[免费试用帐户](https://azure.microsoft.com/free/)。</span><span class="sxs-lookup"><span data-stu-id="da451-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="da451-124">在目录中创建两个服务主体：</span><span class="sxs-lookup"><span data-stu-id="da451-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="da451-125">一个设置用于 Azure 资源，可在 Azure 订阅范围内进行访问。</span><span class="sxs-lookup"><span data-stu-id="da451-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="da451-126">一个设置用于 Azure Stack Hub 资源，可在 Azure Stack Hub 订阅范围内进行访问。</span><span class="sxs-lookup"><span data-stu-id="da451-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="da451-127">若要了解有关创建服务主体和授权访问的详细信息，请参阅[使用应用标识访问资源](/azure-stack/operator/azure-stack-create-service-principals)。</span><span class="sxs-lookup"><span data-stu-id="da451-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals).</span></span> <span data-ttu-id="da451-128">如果希望使用 Azure CLI，请参阅[使用 Azure CLI 创建 Azure 服务主体](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)。</span><span class="sxs-lookup"><span data-stu-id="da451-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="da451-129">在 Azure 或 Azure Stack Hub 中部署 Azure 认知服务。</span><span class="sxs-lookup"><span data-stu-id="da451-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="da451-130">首先，[了解有关认知服务的详细信息](https://azure.microsoft.com/services/cognitive-services/)。</span><span class="sxs-lookup"><span data-stu-id="da451-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="da451-131">然后，访问[将 Azure 认知服务部署到 Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) 以在 Azure Stack Hub 上部署认知服务。</span><span class="sxs-lookup"><span data-stu-id="da451-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="da451-132">首先需要注册才能访问预览。</span><span class="sxs-lookup"><span data-stu-id="da451-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="da451-133">克隆或下载未配置的 Azure 自定义视觉 AI 开发工具包。</span><span class="sxs-lookup"><span data-stu-id="da451-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="da451-134">有关详细信息，请参阅[视觉 AI 开发工具包](https://azure.github.io/Vision-AI-DevKit-Pages/)。</span><span class="sxs-lookup"><span data-stu-id="da451-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="da451-135">注册 Power BI 帐户。</span><span class="sxs-lookup"><span data-stu-id="da451-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="da451-136">Azure 认知服务人脸 API 订阅密钥和终结点 URL。</span><span class="sxs-lookup"><span data-stu-id="da451-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="da451-137">可通过[试用认知服务](https://azure.microsoft.com/try/cognitive-services/?api=face-api)免费试用版获得这两项。</span><span class="sxs-lookup"><span data-stu-id="da451-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="da451-138">或者，按照[创建认知服务帐户](/azure/cognitive-services/cognitive-services-apis-create-account)中的说明进行操作。</span><span class="sxs-lookup"><span data-stu-id="da451-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="da451-139">安装以下开发资源：</span><span class="sxs-lookup"><span data-stu-id="da451-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="da451-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="da451-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2)
  - [<span data-ttu-id="da451-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="da451-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="da451-142">[Porter](https://porter.sh/)。</span><span class="sxs-lookup"><span data-stu-id="da451-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="da451-143">可以通过 Porter 使用所提供的 CNAB 捆绑包清单来部署云应用。</span><span class="sxs-lookup"><span data-stu-id="da451-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="da451-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="da451-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="da451-145">适用于 Visual Studio Code 的 Azure IoT Tools</span><span class="sxs-lookup"><span data-stu-id="da451-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="da451-146">适用于 Visual Studio Code 的 Python 扩展</span><span class="sxs-lookup"><span data-stu-id="da451-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="da451-147">Python</span><span class="sxs-lookup"><span data-stu-id="da451-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="da451-148">部署混合云应用</span><span class="sxs-lookup"><span data-stu-id="da451-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="da451-149">首先，使用 Porter CLI 生成凭据集，然后部署云应用。</span><span class="sxs-lookup"><span data-stu-id="da451-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="da451-150">从 https://github.com/azure-samples/azure-intelligent-edge-patterns 克隆或下载解决方案示例代码。</span><span class="sxs-lookup"><span data-stu-id="da451-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="da451-151">Porter 将生成一组会自动部署应用的凭据。</span><span class="sxs-lookup"><span data-stu-id="da451-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="da451-152">在运行凭据生成命令之前，请确保具有以下各项：</span><span class="sxs-lookup"><span data-stu-id="da451-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="da451-153">用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="da451-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="da451-154">Azure 订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="da451-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="da451-155">用于访问 Azure Stack Hub 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="da451-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="da451-156">Azure Stack Hub 订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="da451-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="da451-157">Azure 认知服务人脸 API 密钥和资源终结点 URL。</span><span class="sxs-lookup"><span data-stu-id="da451-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="da451-158">运行 Porter 凭据生成进程，并按照提示进行操作：</span><span class="sxs-lookup"><span data-stu-id="da451-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="da451-159">Porter 还需要运行一组参数。</span><span class="sxs-lookup"><span data-stu-id="da451-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="da451-160">创建参数文本文件，然后输入以下名称/值对。</span><span class="sxs-lookup"><span data-stu-id="da451-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="da451-161">如果需要有关任何所需值的帮助，请咨询 Azure Stack Hub 管理员。</span><span class="sxs-lookup"><span data-stu-id="da451-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="da451-162">`resource suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。</span><span class="sxs-lookup"><span data-stu-id="da451-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="da451-163">它必须是由字母和数字组成的唯一字符串，不得超过 8 个字符。</span><span class="sxs-lookup"><span data-stu-id="da451-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

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
   <span data-ttu-id="da451-164">保存文本文件并记下其路径。</span><span class="sxs-lookup"><span data-stu-id="da451-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="da451-165">现在可以使用 Porter 部署混合云应用。</span><span class="sxs-lookup"><span data-stu-id="da451-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="da451-166">运行安装命令并监视资源是否部署到 Azure 和 Azure Stack Hub：</span><span class="sxs-lookup"><span data-stu-id="da451-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="da451-167">部署完成后，请记下以下值：</span><span class="sxs-lookup"><span data-stu-id="da451-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="da451-168">相机的连接字符串。</span><span class="sxs-lookup"><span data-stu-id="da451-168">The camera's connection string.</span></span>
    - <span data-ttu-id="da451-169">图像存储帐户连接字符串。</span><span class="sxs-lookup"><span data-stu-id="da451-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="da451-170">资源组名称。</span><span class="sxs-lookup"><span data-stu-id="da451-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="da451-171">准备自定义视觉 AI 开发工具包</span><span class="sxs-lookup"><span data-stu-id="da451-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="da451-172">接下来，按照[视觉 AI 开发工具包快速入门](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)中所示设置自定义视觉 AI 开发工具包。</span><span class="sxs-lookup"><span data-stu-id="da451-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="da451-173">此外，还可以使用上一步骤中提供的连接字符串设置和测试相机。</span><span class="sxs-lookup"><span data-stu-id="da451-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="da451-174">部署相机应用</span><span class="sxs-lookup"><span data-stu-id="da451-174">Deploy the camera app</span></span>

<span data-ttu-id="da451-175">使用 Porter CLI 生成凭据集，然后部署相机应用。</span><span class="sxs-lookup"><span data-stu-id="da451-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="da451-176">Porter 将生成一组会自动部署应用的凭据。</span><span class="sxs-lookup"><span data-stu-id="da451-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="da451-177">在运行凭据生成命令之前，请确保具有以下各项：</span><span class="sxs-lookup"><span data-stu-id="da451-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="da451-178">用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。</span><span class="sxs-lookup"><span data-stu-id="da451-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="da451-179">Azure 订阅的订阅 ID。</span><span class="sxs-lookup"><span data-stu-id="da451-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="da451-180">在部署云应用时提供的图像存储帐户连接字符串。</span><span class="sxs-lookup"><span data-stu-id="da451-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="da451-181">运行 Porter 凭据生成进程，并按照提示进行操作：</span><span class="sxs-lookup"><span data-stu-id="da451-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="da451-182">Porter 还需要运行一组参数。</span><span class="sxs-lookup"><span data-stu-id="da451-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="da451-183">创建参数文本文件，然后输入以下文本。</span><span class="sxs-lookup"><span data-stu-id="da451-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="da451-184">如果不了解某些所需值，请咨询 Azure Stack Hub 管理员。</span><span class="sxs-lookup"><span data-stu-id="da451-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="da451-185">`deployment suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。</span><span class="sxs-lookup"><span data-stu-id="da451-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="da451-186">它必须是由字母和数字组成的唯一字符串，不得超过 8 个字符。</span><span class="sxs-lookup"><span data-stu-id="da451-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="da451-187">保存文本文件并记下其路径。</span><span class="sxs-lookup"><span data-stu-id="da451-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="da451-188">现在可以使用 Porter 部署相机应用。</span><span class="sxs-lookup"><span data-stu-id="da451-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="da451-189">运行安装命令并监视 IoT Edge 部署的创建过程。</span><span class="sxs-lookup"><span data-stu-id="da451-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="da451-190">通过查看位于 `https://<camera-ip>:3000/`（其中 `<camara-ip>` 是相机 IP 地址）处的相机源，验证相机的部署是否完成。</span><span class="sxs-lookup"><span data-stu-id="da451-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="da451-191">此步骤最多可能需要 10 分钟。</span><span class="sxs-lookup"><span data-stu-id="da451-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="da451-192">配置 Azure 流分析</span><span class="sxs-lookup"><span data-stu-id="da451-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="da451-193">现在，数据已从相机流向 Azure 流分析，我们需要手动授权它与 Power BI 通信。</span><span class="sxs-lookup"><span data-stu-id="da451-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="da451-194">在 Azure 门户中，打开“所有资源”，然后打开 process-footfall\[yoursuffix\] 作业。</span><span class="sxs-lookup"><span data-stu-id="da451-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="da451-195">在“流分析作业”窗格的“作业拓扑”部分，选择“输出”选项。</span><span class="sxs-lookup"><span data-stu-id="da451-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="da451-196">选择“traffic-output”输出接收器。</span><span class="sxs-lookup"><span data-stu-id="da451-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="da451-197">选择“续订授权”并登录到 Power BI 帐户。</span><span class="sxs-lookup"><span data-stu-id="da451-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Power BI 中的续订授权提示](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="da451-199">保存输出设置。</span><span class="sxs-lookup"><span data-stu-id="da451-199">Save the output settings.</span></span>

6. <span data-ttu-id="da451-200">转到“概述”窗格，然后选择“开始”以开始将数据发送到 Power BI 。</span><span class="sxs-lookup"><span data-stu-id="da451-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="da451-201">选择“现在”作为作业输出启动时间，然后选择“启动”。 </span><span class="sxs-lookup"><span data-stu-id="da451-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="da451-202">可以在通知栏中查看作业状态。</span><span class="sxs-lookup"><span data-stu-id="da451-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="da451-203">创建 Power BI 面板</span><span class="sxs-lookup"><span data-stu-id="da451-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="da451-204">作业成功以后，请转到 [Power BI](https://powerbi.com/)，然后使用工作或学校帐户登录。</span><span class="sxs-lookup"><span data-stu-id="da451-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="da451-205">如果流分析作业查询输出结果，则创建的 footfall-dataset 数据集存在于“数据集”选项卡下。</span><span class="sxs-lookup"><span data-stu-id="da451-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="da451-206">从 Power BI 工作区选择“+ 创建”，创建名为“客流量分析”的新面板。</span><span class="sxs-lookup"><span data-stu-id="da451-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="da451-207">在窗口顶部，选择“添加磁贴”。</span><span class="sxs-lookup"><span data-stu-id="da451-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="da451-208">然后选择“自定义流数据”和“下一步”。 </span><span class="sxs-lookup"><span data-stu-id="da451-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="da451-209">在“你的数据集”下选择“footfall-dataset” 。</span><span class="sxs-lookup"><span data-stu-id="da451-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="da451-210">在“可视化效果类型”下拉列表中选择“卡”，然后向“字段”添加“年龄”   。</span><span class="sxs-lookup"><span data-stu-id="da451-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="da451-211">选择“下一步”，为磁贴输入一个名称，然后选择“应用”，创建该磁贴。</span><span class="sxs-lookup"><span data-stu-id="da451-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="da451-212">你可以根据需要添加其他字段和卡。</span><span class="sxs-lookup"><span data-stu-id="da451-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="da451-213">测试解决方案</span><span class="sxs-lookup"><span data-stu-id="da451-213">Test Your Solution</span></span>

<span data-ttu-id="da451-214">观察当不同的人走过相机时，Power BI 中创建的卡片中的数据是如何变化的。</span><span class="sxs-lookup"><span data-stu-id="da451-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="da451-215">录制后，最多可能需要 20 秒才会显示推理。</span><span class="sxs-lookup"><span data-stu-id="da451-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="da451-216">删除解决方案</span><span class="sxs-lookup"><span data-stu-id="da451-216">Remove Your Solution</span></span>

<span data-ttu-id="da451-217">如果要删除解决方案，请使用 Porter 以及针对部署创建的相同参数文件运行以下命令：</span><span class="sxs-lookup"><span data-stu-id="da451-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="da451-218">后续步骤</span><span class="sxs-lookup"><span data-stu-id="da451-218">Next steps</span></span>

- <span data-ttu-id="da451-219">详细了解[混合应用设计注意事项](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="da451-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="da451-220">查看 [GitHub 上此示例的代码](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)并提出改进建议。</span><span class="sxs-lookup"><span data-stu-id="da451-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
