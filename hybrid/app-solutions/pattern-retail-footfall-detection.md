---
title: 使用 Azure 和 Azure Stack 集线器的 Footfall 检测模式
description: 了解如何使用 Azure 和 Azure Stack 中心实现基于 AI 的 footfall 检测解决方案，以便分析零售商店流量。
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909960"
---
# <a name="footfall-detection-pattern"></a><span data-ttu-id="361eb-103">Footfall 检测模式</span><span class="sxs-lookup"><span data-stu-id="361eb-103">Footfall detection pattern</span></span>

<span data-ttu-id="361eb-104">此模式概述了如何实现基于 AI 的 footfall 检测解决方案，以便在零售商店中分析访问者流量。</span><span class="sxs-lookup"><span data-stu-id="361eb-104">This pattern provides an overview for implementing an AI-based footfall detection solution for analyzing visitor traffic in retail stores.</span></span> <span data-ttu-id="361eb-105">该解决方案使用 Azure、Azure Stack 中心和自定义视觉 AI 开发工具包从实际操作中生成见解。</span><span class="sxs-lookup"><span data-stu-id="361eb-105">The solution generates insights from real world actions, using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="361eb-106">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="361eb-106">Context and problem</span></span>

<span data-ttu-id="361eb-107">Contoso 商店希望获得有关客户如何接收其当前产品与商店布局的见解。</span><span class="sxs-lookup"><span data-stu-id="361eb-107">Contoso Stores would like to gain insights on how customers are receiving their current products in relation to store layout.</span></span> <span data-ttu-id="361eb-108">他们无法在每个部分中放置人员，因此，拥有一组分析人员审查整个商店的摄像机素材是低效的。</span><span class="sxs-lookup"><span data-stu-id="361eb-108">They're unable to place staff in every section and it's inefficient to have a team of analysts review an entire store's camera footage.</span></span> <span data-ttu-id="361eb-109">此外，其任何商店都没有足够的带宽来将视频从其所有相机流式传输到云以进行分析。</span><span class="sxs-lookup"><span data-stu-id="361eb-109">In addition, none of their stores have enough bandwidth to stream video from all their cameras to the cloud for analysis.</span></span>

<span data-ttu-id="361eb-110">Contoso 想要找出一种不引人注目的、可通过隐私识别的方式来确定其客户的人口统计、忠诚度和反应，以存储显示器和产品。</span><span class="sxs-lookup"><span data-stu-id="361eb-110">Contoso would like to find an unobtrusive, privacy-friendly way to determine their customers' demographics, loyalty, and reactions to store displays and products.</span></span>

## <a name="solution"></a><span data-ttu-id="361eb-111">解决方案</span><span class="sxs-lookup"><span data-stu-id="361eb-111">Solution</span></span>

<span data-ttu-id="361eb-112">此零售 analytics 模式使用分层方法在边缘推断。</span><span class="sxs-lookup"><span data-stu-id="361eb-112">This retail analytics pattern uses a tiered approach to inferencing at the edge.</span></span> <span data-ttu-id="361eb-113">通过使用自定义视觉 AI 开发工具包，只会将具有人脸的图像发送到运行 Azure 认知服务的专用 Azure Stack 中心。</span><span class="sxs-lookup"><span data-stu-id="361eb-113">By using the Custom Vision AI Dev Kit, only images with human faces are sent for analysis to a private Azure Stack Hub that runs Azure Cognitive Services.</span></span> <span data-ttu-id="361eb-114">匿名，聚合数据将发送到 Azure，以便在 Power BI 中的所有存储和可视化对象中进行聚合。</span><span class="sxs-lookup"><span data-stu-id="361eb-114">Anonymized, aggregated data is sent to Azure for aggregation across all stores and visualization in Power BI.</span></span> <span data-ttu-id="361eb-115">通过结合使用边缘和公有云，Contoso 可以利用新式 AI 技术，同时还会遵守其公司政策并遵从客户的隐私。</span><span class="sxs-lookup"><span data-stu-id="361eb-115">Combining the edge and public cloud lets Contoso take advantage of modern AI technology while also remaining in compliance with their corporate policies and respecting their customers' privacy.</span></span>

<span data-ttu-id="361eb-116">[![Footfall 检测模式解决方案](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="361eb-116">[![Footfall detection pattern solution](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span></span>

<span data-ttu-id="361eb-117">下面是该解决方案的工作原理的摘要：</span><span class="sxs-lookup"><span data-stu-id="361eb-117">Here's a summary of how the solution works:</span></span>

1. <span data-ttu-id="361eb-118">自定义视觉 AI 开发工具包从 IoT 中心获取配置，该配置将安装 IoT Edge 运行时和 ML 模型。</span><span class="sxs-lookup"><span data-stu-id="361eb-118">The Custom Vision AI Dev Kit gets a configuration from IoT Hub, which installs the IoT Edge Runtime and an ML model.</span></span>
2. <span data-ttu-id="361eb-119">如果该模型看到某个人，则会拍摄一张图片，并将其上传到 Azure Stack 中心 blob 存储。</span><span class="sxs-lookup"><span data-stu-id="361eb-119">If the model sees a person, it takes a picture and uploads it to Azure Stack Hub blob storage.</span></span>
3. <span data-ttu-id="361eb-120">Blob 服务在 Azure Stack 集线器上触发 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="361eb-120">The blob service triggers an Azure Function on Azure Stack Hub.</span></span>
4. <span data-ttu-id="361eb-121">Azure 函数会调用容器，其中包含人脸 API 以获取图像中的人口统计数据和情感数据。</span><span class="sxs-lookup"><span data-stu-id="361eb-121">The Azure Function calls a container with the Face API to get demographic and emotion data from the image.</span></span>
5. <span data-ttu-id="361eb-122">数据匿名并发送到 Azure 事件中心群集。</span><span class="sxs-lookup"><span data-stu-id="361eb-122">The data is anonymized and sent to an Azure Event Hubs cluster.</span></span>
6. <span data-ttu-id="361eb-123">事件中心群集可将数据推送到流分析。</span><span class="sxs-lookup"><span data-stu-id="361eb-123">The Event Hubs cluster pushes the data to Stream Analytics.</span></span>
7. <span data-ttu-id="361eb-124">流分析聚合数据并将其推送到 Power BI。</span><span class="sxs-lookup"><span data-stu-id="361eb-124">Stream Analytics aggregates the data and pushes it to Power BI.</span></span>

## <a name="components"></a><span data-ttu-id="361eb-125">组件</span><span class="sxs-lookup"><span data-stu-id="361eb-125">Components</span></span>

<span data-ttu-id="361eb-126">此解决方案使用以下组件：</span><span class="sxs-lookup"><span data-stu-id="361eb-126">This solution uses the following components:</span></span>

| <span data-ttu-id="361eb-127">层</span><span class="sxs-lookup"><span data-stu-id="361eb-127">Layer</span></span> | <span data-ttu-id="361eb-128">组件</span><span class="sxs-lookup"><span data-stu-id="361eb-128">Component</span></span> | <span data-ttu-id="361eb-129">说明</span><span class="sxs-lookup"><span data-stu-id="361eb-129">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="361eb-130">应用商店内硬件</span><span class="sxs-lookup"><span data-stu-id="361eb-130">In-store hardware</span></span> | [<span data-ttu-id="361eb-131">自定义视觉 AI 开发工具包</span><span class="sxs-lookup"><span data-stu-id="361eb-131">Custom Vision AI Dev Kit</span></span>](https://azure.github.io/Vision-AI-DevKit-Pages/) | <span data-ttu-id="361eb-132">使用仅捕获要分析的人员图像的本地 ML 模型提供存储中筛选。</span><span class="sxs-lookup"><span data-stu-id="361eb-132">Provides in-store filtering using a local ML model that only captures images of people for analysis.</span></span> <span data-ttu-id="361eb-133">通过 IoT 中心安全地预配和更新。</span><span class="sxs-lookup"><span data-stu-id="361eb-133">Securely provisioned and updated through IoT Hub.</span></span><br><br>|
| <span data-ttu-id="361eb-134">Azure</span><span class="sxs-lookup"><span data-stu-id="361eb-134">Azure</span></span> | [<span data-ttu-id="361eb-135">Azure 事件中心</span><span class="sxs-lookup"><span data-stu-id="361eb-135">Azure Event Hubs</span></span>](/azure/event-hubs/) | <span data-ttu-id="361eb-136">Azure 事件中心提供可缩放的平台，适用于与 Azure 流分析完美集成的引入匿名数据。</span><span class="sxs-lookup"><span data-stu-id="361eb-136">Azure Event Hubs provides a scalable platform for ingesting anonymized data that integrates neatly with Azure Stream Analytics.</span></span> |
|  | [<span data-ttu-id="361eb-137">Azure 流分析</span><span class="sxs-lookup"><span data-stu-id="361eb-137">Azure Stream Analytics</span></span>](/azure/stream-analytics/) | <span data-ttu-id="361eb-138">Azure 流分析作业聚合匿名数据，并将其分组到15秒的窗口中进行可视化。</span><span class="sxs-lookup"><span data-stu-id="361eb-138">An Azure Stream Analytics job aggregates the anonymized data and groups it into 15-second windows for visualization.</span></span> |
|  | [<span data-ttu-id="361eb-139">Microsoft Power BI</span><span class="sxs-lookup"><span data-stu-id="361eb-139">Microsoft Power BI</span></span>](https://powerbi.microsoft.com/) | <span data-ttu-id="361eb-140">Power BI 提供易用的仪表板界面用于查看 Azure 流分析的输出。</span><span class="sxs-lookup"><span data-stu-id="361eb-140">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="361eb-141">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="361eb-141">Azure Stack Hub</span></span> | [<span data-ttu-id="361eb-142">应用服务</span><span class="sxs-lookup"><span data-stu-id="361eb-142">App Service</span></span>](/azure-stack/operator/azure-stack-app-service-overview.md) | <span data-ttu-id="361eb-143">应用服务资源提供程序（RP）为边缘组件提供基础，其中包括 web 应用/Api 和函数的托管和管理功能。</span><span class="sxs-lookup"><span data-stu-id="361eb-143">The App Service resource provider (RP) provides a base for edge components, including hosting and management features for web apps/APIs and Functions.</span></span> |
| | <span data-ttu-id="361eb-144">Azure Kubernetes 服务[（AKS）引擎](https://github.com/Azure/aks-engine)群集</span><span class="sxs-lookup"><span data-stu-id="361eb-144">Azure Kubernetes Service [(AKS) Engine](https://github.com/Azure/aks-engine) cluster</span></span> | <span data-ttu-id="361eb-145">部署到 Azure Stack 集线器中的 AKS 的 AKS RP 提供可缩放的弹性引擎来运行人脸 API 容器。</span><span class="sxs-lookup"><span data-stu-id="361eb-145">The AKS RP with AKS-Engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span> |
| | <span data-ttu-id="361eb-146">Azure 认知服务[人脸 API 容器](/azure/cognitive-services/face/face-how-to-install-containers)</span><span class="sxs-lookup"><span data-stu-id="361eb-146">Azure Cognitive Services [Face API containers](/azure/cognitive-services/face/face-how-to-install-containers)</span></span>| <span data-ttu-id="361eb-147">具有人脸 API 容器的 Azure 认知服务 RP 在 Contoso 的专用网络上提供了人口统计、情感和独特的访问者检测。</span><span class="sxs-lookup"><span data-stu-id="361eb-147">The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network.</span></span> |
| | <span data-ttu-id="361eb-148">Blob 存储</span><span class="sxs-lookup"><span data-stu-id="361eb-148">Blob Storage</span></span> | <span data-ttu-id="361eb-149">从 AI 开发工具包捕获的图像将上传到 Azure Stack 集线器的 blob 存储。</span><span class="sxs-lookup"><span data-stu-id="361eb-149">Images captured from the AI Dev Kit are uploaded to Azure Stack Hub's blob storage.</span></span> |
| | <span data-ttu-id="361eb-150">Azure Functions</span><span class="sxs-lookup"><span data-stu-id="361eb-150">Azure Functions</span></span> | <span data-ttu-id="361eb-151">在 Azure Stack 集线器上运行的 Azure 函数接收来自 blob 存储的输入，并管理与人脸 API 的交互。</span><span class="sxs-lookup"><span data-stu-id="361eb-151">An Azure Function running on Azure Stack Hub receives input from blob storage and manages the interactions with the Face API.</span></span> <span data-ttu-id="361eb-152">它将匿名数据发送到位于 Azure 中的事件中心群集。</span><span class="sxs-lookup"><span data-stu-id="361eb-152">It emits anonymized data to an Event Hubs cluster located in Azure.</span></span><br><br>|

## <a name="issues-and-considerations"></a><span data-ttu-id="361eb-153">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="361eb-153">Issues and considerations</span></span>

<span data-ttu-id="361eb-154">在决定如何实现此解决方案时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="361eb-154">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="361eb-155">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="361eb-155">Scalability</span></span>

<span data-ttu-id="361eb-156">若要使此解决方案能够跨多个照相机和位置进行扩展，需要确保所有组件都能处理增加的负载。</span><span class="sxs-lookup"><span data-stu-id="361eb-156">To enable this solution to scale across multiple cameras and locations, you'll need to make sure that all of the components can handle the increased load.</span></span> <span data-ttu-id="361eb-157">可能需要采取如下操作：</span><span class="sxs-lookup"><span data-stu-id="361eb-157">You may need to take actions like:</span></span>

- <span data-ttu-id="361eb-158">增加流分析流式处理单元的数量。</span><span class="sxs-lookup"><span data-stu-id="361eb-158">Increase the number of Stream Analytics streaming units.</span></span>
- <span data-ttu-id="361eb-159">横向扩展人脸 API 部署。</span><span class="sxs-lookup"><span data-stu-id="361eb-159">Scale out the Face API deployment.</span></span>
- <span data-ttu-id="361eb-160">提高事件中心群集吞吐量。</span><span class="sxs-lookup"><span data-stu-id="361eb-160">Increase the Event Hubs cluster throughput.</span></span>
- <span data-ttu-id="361eb-161">对于极端情况，可能需要从 Azure Functions 迁移到虚拟机。</span><span class="sxs-lookup"><span data-stu-id="361eb-161">For extreme cases, migrate from Azure Functions to a virtual machine may be necessary.</span></span>

### <a name="availability"></a><span data-ttu-id="361eb-162">可用性</span><span class="sxs-lookup"><span data-stu-id="361eb-162">Availability</span></span>

<span data-ttu-id="361eb-163">由于此解决方案是分层的，因此请务必考虑如何处理网络或电源故障。</span><span class="sxs-lookup"><span data-stu-id="361eb-163">Since this solution is tiered, it's important to think about how to deal with networking or power failures.</span></span> <span data-ttu-id="361eb-164">根据业务需要，你可能想要实现一种机制，以便在本地缓存映像，然后在连接返回时转发到 Azure Stack 中心。</span><span class="sxs-lookup"><span data-stu-id="361eb-164">Depending on business needs, you might want to implement a mechanism to cache images locally, then forward to Azure Stack Hub when connectivity returns.</span></span> <span data-ttu-id="361eb-165">如果位置足够大，则使用人脸 API 容器将 Data Box Edge 部署到该位置可能是更好的选择。</span><span class="sxs-lookup"><span data-stu-id="361eb-165">If the location is large enough, deploying a Data Box Edge with the Face API container to that location might be a better option.</span></span>

### <a name="manageability"></a><span data-ttu-id="361eb-166">可管理性</span><span class="sxs-lookup"><span data-stu-id="361eb-166">Manageability</span></span>

<span data-ttu-id="361eb-167">此解决方案可能跨越许多设备和位置，因而变得不好管理。</span><span class="sxs-lookup"><span data-stu-id="361eb-167">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="361eb-168">可以使用[Azure IoT 服务](/azure/iot-fundamentals/)自动使新的位置和设备联机，并使其保持最新状态。</span><span class="sxs-lookup"><span data-stu-id="361eb-168">[Azure's IoT services](/azure/iot-fundamentals/) can be used to automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="361eb-169">安全性</span><span class="sxs-lookup"><span data-stu-id="361eb-169">Security</span></span>

<span data-ttu-id="361eb-170">此解决方案捕获客户映像，使安全性成为极其重要的考虑因素。</span><span class="sxs-lookup"><span data-stu-id="361eb-170">This solution captures customer images, making security a paramount consideration.</span></span> <span data-ttu-id="361eb-171">确保所有存储帐户都使用正确的访问策略和定期轮换密钥进行保护。</span><span class="sxs-lookup"><span data-stu-id="361eb-171">Make sure all storage accounts are secured with the proper access policies and rotate keys regularly.</span></span> <span data-ttu-id="361eb-172">确保存储帐户和事件中心具有符合公司和政府隐私法规的保留策略。</span><span class="sxs-lookup"><span data-stu-id="361eb-172">Ensure storage accounts and Event Hubs have retention policies that meet corporate and government privacy regulations.</span></span> <span data-ttu-id="361eb-173">另外，请确保对用户访问级别进行分层。</span><span class="sxs-lookup"><span data-stu-id="361eb-173">Also make sure to tier the user access levels.</span></span> <span data-ttu-id="361eb-174">分层确保用户只能访问其角色所需的数据。</span><span class="sxs-lookup"><span data-stu-id="361eb-174">Tiering ensures that users only have access to the data they need for their role.</span></span>

## <a name="next-steps"></a><span data-ttu-id="361eb-175">后续步骤</span><span class="sxs-lookup"><span data-stu-id="361eb-175">Next steps</span></span>

<span data-ttu-id="361eb-176">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="361eb-176">To learn more about the topics introduced in this article:</span></span>

- <span data-ttu-id="361eb-177">请参阅 footfall 检测模式使用的[分层数据模式](https://aka.ms/tiereddatadeploy)。</span><span class="sxs-lookup"><span data-stu-id="361eb-177">See the [Tiered Data pattern](https://aka.ms/tiereddatadeploy), which is leveraged by the footfall detection pattern.</span></span>
- <span data-ttu-id="361eb-178">有关使用自定义视觉的详细信息，请参阅[自定义视觉 AI 开发工具包](https://azure.github.io/Vision-AI-DevKit-Pages/)。</span><span class="sxs-lookup"><span data-stu-id="361eb-178">See the [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) to learn more about using custom vision.</span></span> 

<span data-ttu-id="361eb-179">准备好测试解决方案示例时，请继续学习[Footfall 检测部署指南](solution-deployment-guide-retail-footfall-detection.md)。</span><span class="sxs-lookup"><span data-stu-id="361eb-179">When you're ready to test the solution example, continue with the [Footfall detection deployment guide](solution-deployment-guide-retail-footfall-detection.md).</span></span> <span data-ttu-id="361eb-180">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="361eb-180">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
