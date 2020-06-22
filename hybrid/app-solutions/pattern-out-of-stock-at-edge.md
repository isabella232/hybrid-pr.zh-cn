---
title: 使用 Azure 和 Azure Stack Edge 进行缺货检测
description: 了解如何使用 Azure 和 Azure Stack Edge 服务实现缺货检测。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 865f63bc4234e50ed169aa29cefdb1886750594c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909955"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a><span data-ttu-id="1659a-103">边缘缺货检测模式</span><span class="sxs-lookup"><span data-stu-id="1659a-103">Out of stock detection at the edge pattern</span></span>

<span data-ttu-id="1659a-104">本模式演示如何使用 Azure Stack Edge 或 Azure IoT Edge 设备与网络摄像头来确定货架上是否有商品缺货。</span><span class="sxs-lookup"><span data-stu-id="1659a-104">This pattern illustrates how to determine if shelves have out of stock items using an Azure Stack Edge or Azure IoT Edge device and network cameras.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="1659a-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="1659a-105">Context and problem</span></span>

<span data-ttu-id="1659a-106">实体零售店的销售额下降，因为当客户查找某个商品时，货架上没有该商品。</span><span class="sxs-lookup"><span data-stu-id="1659a-106">Physical retail stores lose sales because when customers look for an item, it's not present on the shelf.</span></span> <span data-ttu-id="1659a-107">但是，该商品可能位于商店后面，只是未补货上架。</span><span class="sxs-lookup"><span data-stu-id="1659a-107">However, the item could have been in the back of the store and not been restocked.</span></span> <span data-ttu-id="1659a-108">商店想要提高员工的工作效率，并在商品需要补货时自动接收通知。</span><span class="sxs-lookup"><span data-stu-id="1659a-108">Stores would like to use their staff more efficiently and get automatically notified when items need restocking.</span></span>

## <a name="solution"></a><span data-ttu-id="1659a-109">解决方案</span><span class="sxs-lookup"><span data-stu-id="1659a-109">Solution</span></span>

<span data-ttu-id="1659a-110">解决方案示例使用边缘设备（例如每家商店布置一个 Azure Stack Edge），以有效处理商店中的摄像头数据。</span><span class="sxs-lookup"><span data-stu-id="1659a-110">The solution example uses an edge device, like an Azure Stack Edge in each store, which efficiently processes data from cameras in the store.</span></span> <span data-ttu-id="1659a-111">这种优化的设计可让商店只将相关的事件与图像发送到云。</span><span class="sxs-lookup"><span data-stu-id="1659a-111">This optimized design lets stores send only relevant events and images to the cloud.</span></span> <span data-ttu-id="1659a-112">该设计可节省带宽、存储空间，并保护客户的隐私。</span><span class="sxs-lookup"><span data-stu-id="1659a-112">The design saves bandwidth, storage space, and ensures customer privacy.</span></span> <span data-ttu-id="1659a-113">从每个摄像头读取帧时，机器学习模型将处理图像，并返回任何缺货区域。</span><span class="sxs-lookup"><span data-stu-id="1659a-113">As frames are read from each camera, an ML model processes the image and returns any out of stock areas.</span></span> <span data-ttu-id="1659a-114">图像与缺货区域显示在本地 Web 应用上。</span><span class="sxs-lookup"><span data-stu-id="1659a-114">The image and out of stock areas are displayed on a local web app.</span></span> <span data-ttu-id="1659a-115">此数据可以发送到时序见解环境，以在 Power BI 中显示见解。</span><span class="sxs-lookup"><span data-stu-id="1659a-115">This data can be sent to a Time Series Insight environment to show insights in Power BI.</span></span>

![边缘缺货解决方案体系结构](media/pattern-out-of-stock-at-edge/solution-architecture.png)

<span data-ttu-id="1659a-117">该解决方案的工作原理如下：</span><span class="sxs-lookup"><span data-stu-id="1659a-117">Here's how the solution works:</span></span>

1. <span data-ttu-id="1659a-118">通过 HTTP 或 RTSP 捕获网络摄像头的图像。</span><span class="sxs-lookup"><span data-stu-id="1659a-118">Images are captured from a network camera over HTTP or RTSP.</span></span>
2. <span data-ttu-id="1659a-119">图像在调整大小后发送到推理驱动程序，后者与机器学习模型通信，以确定是否有任何缺货图像。</span><span class="sxs-lookup"><span data-stu-id="1659a-119">The image is resized and sent to the inference driver, which communicates with the ML model to determine if there are any out of stock images.</span></span>
3. <span data-ttu-id="1659a-120">机器学习模型返回任何缺货区域。</span><span class="sxs-lookup"><span data-stu-id="1659a-120">The ML model returns any out of stock areas.</span></span>
4. <span data-ttu-id="1659a-121">推理驱动程序将未经处理的图像上传到 Blob（如果已指定），并将模型的结果发送到 Azure IoT 中心，以及设备上的边界框处理器。</span><span class="sxs-lookup"><span data-stu-id="1659a-121">The inferencing driver uploads the raw image to a blob (if specified), and sends the results from the model to Azure IoT Hub and a bounding box processor on the device.</span></span>
5. <span data-ttu-id="1659a-122">边界框处理器将边界框添加到图像，并在内存中数据库中缓存图像路径。</span><span class="sxs-lookup"><span data-stu-id="1659a-122">The bounding box processor adds bounding boxes to the image and caches the image path in an in-memory database.</span></span>
6. <span data-ttu-id="1659a-123">Web 应用查询图像，并按接收顺序显示图像。</span><span class="sxs-lookup"><span data-stu-id="1659a-123">The web app queries for images and shows them in the order received.</span></span>
7. <span data-ttu-id="1659a-124">在时序见解中聚合来自 IoT 中心的消息。</span><span class="sxs-lookup"><span data-stu-id="1659a-124">Messages from IoT Hub are aggregated in Time Series Insights.</span></span>
8. <span data-ttu-id="1659a-125">Power BI 显示一段时间内缺货商品的交互式报表，其中包括时序见解数据。</span><span class="sxs-lookup"><span data-stu-id="1659a-125">Power BI displays an interactive report of out of stock items over time with the data from Time Series Insights.</span></span>


## <a name="components"></a><span data-ttu-id="1659a-126">组件</span><span class="sxs-lookup"><span data-stu-id="1659a-126">Components</span></span>

<span data-ttu-id="1659a-127">此解决方案使用以下组件：</span><span class="sxs-lookup"><span data-stu-id="1659a-127">This solution uses the following components:</span></span>

| <span data-ttu-id="1659a-128">层</span><span class="sxs-lookup"><span data-stu-id="1659a-128">Layer</span></span> | <span data-ttu-id="1659a-129">组件</span><span class="sxs-lookup"><span data-stu-id="1659a-129">Component</span></span> | <span data-ttu-id="1659a-130">说明</span><span class="sxs-lookup"><span data-stu-id="1659a-130">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="1659a-131">本地硬件</span><span class="sxs-lookup"><span data-stu-id="1659a-131">On-premises hardware</span></span> | <span data-ttu-id="1659a-132">网络摄像头</span><span class="sxs-lookup"><span data-stu-id="1659a-132">Network camera</span></span> | <span data-ttu-id="1659a-133">需要一个网络摄像头，并配合 HTTP 或 RTSP 源提供用于推理的图像。</span><span class="sxs-lookup"><span data-stu-id="1659a-133">A network camera is required, with either an HTTP or RTSP feed to provide the images for inference.</span></span> |
| <span data-ttu-id="1659a-134">Azure</span><span class="sxs-lookup"><span data-stu-id="1659a-134">Azure</span></span> | <span data-ttu-id="1659a-135">Azure IoT 中心</span><span class="sxs-lookup"><span data-stu-id="1659a-135">Azure IoT Hub</span></span> | <span data-ttu-id="1659a-136">[Azure IoT 中心](/azure/iot-hub/)为边缘设备处理设备预配和消息传递。</span><span class="sxs-lookup"><span data-stu-id="1659a-136">[Azure IoT Hub](/azure/iot-hub/) handles device provisioning and messaging for the edge devices.</span></span> |
|  | <span data-ttu-id="1659a-137">Azure Time Series Insights</span><span class="sxs-lookup"><span data-stu-id="1659a-137">Azure Time Series Insights</span></span> | <span data-ttu-id="1659a-138">[Azure 时序见解](/azure/time-series-insights/)存储来自 IoT 中心的消息用于可视化。</span><span class="sxs-lookup"><span data-stu-id="1659a-138">[Azure Time Series Insights](/azure/time-series-insights/) stores the messages from IoT Hub for visualization.</span></span> |
|  | <span data-ttu-id="1659a-139">Power BI</span><span class="sxs-lookup"><span data-stu-id="1659a-139">Power BI</span></span> | <span data-ttu-id="1659a-140">[Microsoft Power BI](https://powerbi.microsoft.com/) 提供缺货事件的以业务为中心的报表。</span><span class="sxs-lookup"><span data-stu-id="1659a-140">[Microsoft Power BI](https://powerbi.microsoft.com/) provides business-focused reports of out of stock events.</span></span> <span data-ttu-id="1659a-141">Power BI 提供易用的仪表板界面用于查看 Azure 流分析的输出。</span><span class="sxs-lookup"><span data-stu-id="1659a-141">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="1659a-142">Azure Stack Edge 或</span><span class="sxs-lookup"><span data-stu-id="1659a-142">Azure Stack Edge or</span></span><br><span data-ttu-id="1659a-143">Azure IoT Edge 设备</span><span class="sxs-lookup"><span data-stu-id="1659a-143">Azure IoT Edge device</span></span> | <span data-ttu-id="1659a-144">Azure IoT Edge</span><span class="sxs-lookup"><span data-stu-id="1659a-144">Azure IoT Edge</span></span> | <span data-ttu-id="1659a-145">[Azure IoT Edge](/azure/iot-edge/) 协调本地容器的运行时，并处理设备管理和更新。</span><span class="sxs-lookup"><span data-stu-id="1659a-145">[Azure IoT Edge](/azure/iot-edge/) orchestrates the runtime for the on-premises containers and handles device management and updates.</span></span>|
| | <span data-ttu-id="1659a-146">Azure Project Brainwave</span><span class="sxs-lookup"><span data-stu-id="1659a-146">Azure project brainwave</span></span> | <span data-ttu-id="1659a-147">在 Azure Stack Edge 设备上，[Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) 使用现场可编程门阵列 (FPGA) 来加速机器学习推理。</span><span class="sxs-lookup"><span data-stu-id="1659a-147">On an Azure Stack Edge device, [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) uses Field-Programmable Gate Arrays (FPGAs) to accelerate ML inferencing.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="1659a-148">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="1659a-148">Issues and considerations</span></span>

<span data-ttu-id="1659a-149">在决定如何实现此解决方案时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="1659a-149">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="1659a-150">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="1659a-150">Scalability</span></span>

<span data-ttu-id="1659a-151">根据提供的硬件，大多数机器学习模型只能按特定的每秒帧数运行。</span><span class="sxs-lookup"><span data-stu-id="1659a-151">Most machine learning models can only run at a certain number of frames per second, depending on the provided hardware.</span></span> <span data-ttu-id="1659a-152">请确定摄像头的最佳采样率，以确保机器学习管道不会备份。</span><span class="sxs-lookup"><span data-stu-id="1659a-152">Determine the optimal sample rate from your camera(s) to ensure that the ML pipeline doesn't back up.</span></span> <span data-ttu-id="1659a-153">不同类型的硬件将处理不同数目的摄像头与帧速率。</span><span class="sxs-lookup"><span data-stu-id="1659a-153">Different types of hardware will handle different numbers of cameras and frame rates.</span></span>

### <a name="availability"></a><span data-ttu-id="1659a-154">可用性</span><span class="sxs-lookup"><span data-stu-id="1659a-154">Availability</span></span>

<span data-ttu-id="1659a-155">必须考虑到边缘设备断开连接时可能发生的情况。</span><span class="sxs-lookup"><span data-stu-id="1659a-155">It's important to consider what might happen if the edge device loses connectivity.</span></span> <span data-ttu-id="1659a-156">请考虑时序见解与 Power BI 仪表板可能丢失哪些数据。</span><span class="sxs-lookup"><span data-stu-id="1659a-156">Consider what data might be lost from the Time Series Insights and Power BI dashboard.</span></span> <span data-ttu-id="1659a-157">提供的示例解决方案在设计上不具备高可用性。</span><span class="sxs-lookup"><span data-stu-id="1659a-157">The example solution as provided isn't designed to be highly available.</span></span>

### <a name="manageability"></a><span data-ttu-id="1659a-158">可管理性</span><span class="sxs-lookup"><span data-stu-id="1659a-158">Manageability</span></span>

<span data-ttu-id="1659a-159">此解决方案可能跨越许多设备和位置，因而变得不好管理。</span><span class="sxs-lookup"><span data-stu-id="1659a-159">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="1659a-160">Azure 的 IoT 服务可自动将新的位置和设备联机，并使其保持最新状态。</span><span class="sxs-lookup"><span data-stu-id="1659a-160">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span> <span data-ttu-id="1659a-161">此外，必须遵循适当的数据监管过程。</span><span class="sxs-lookup"><span data-stu-id="1659a-161">Proper data governance procedures must be followed as well.</span></span>

### <a name="security"></a><span data-ttu-id="1659a-162">安全性</span><span class="sxs-lookup"><span data-stu-id="1659a-162">Security</span></span>

<span data-ttu-id="1659a-163">本模式将处理潜在敏感的数据。</span><span class="sxs-lookup"><span data-stu-id="1659a-163">This pattern handles potentially sensitive data.</span></span> <span data-ttu-id="1659a-164">请确保密钥定期轮换，并已正确设置 Azure 存储帐户和本地共享的权限。</span><span class="sxs-lookup"><span data-stu-id="1659a-164">Make sure keys are regularly rotated and the permissions on the Azure Storage Account and local shares are correctly set.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1659a-165">后续步骤</span><span class="sxs-lookup"><span data-stu-id="1659a-165">Next steps</span></span>

<span data-ttu-id="1659a-166">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="1659a-166">To learn more about topics introduced in this article:</span></span>
- <span data-ttu-id="1659a-167">本模式中使用了多个 IoT 相关服务，包括 [Azure IoT Edge](/azure/iot-edge/)、[Azure IoT 中心](/azure/iot-hub/)和 [Azure 时序见解](/azure/time-series-insights/)。</span><span class="sxs-lookup"><span data-stu-id="1659a-167">Multiple IoT related services are used in this pattern, including [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/), and [Azure Time Series Insights](/azure/time-series-insights/).</span></span>
- <span data-ttu-id="1659a-168">若要了解有关 Microsoft Project 脑电波的详细信息，请参阅[博客公告](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)和签出[Azure 加速机器学习与 Project 脑电波视频](https://www.youtube.com/watch?v=DJfMobMjCX0)。</span><span class="sxs-lookup"><span data-stu-id="1659a-168">To learn more about Microsoft Project Brainwave, see [the blog announcement](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) and checkout out the [Azure Accelerated Machine Learning with Project Brainwave video](https://www.youtube.com/watch?v=DJfMobMjCX0).</span></span>
- <span data-ttu-id="1659a-169">请参阅[混合应用设计注意事项](overview-app-design-considerations.md)，了解有关最佳做法的详细信息，并获取对其他任何问题的解答。</span><span class="sxs-lookup"><span data-stu-id="1659a-169">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers to any additional questions.</span></span>
- <span data-ttu-id="1659a-170">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="1659a-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="1659a-171">准备好测试解决方案示例时，请继续阅读[分层式数据分析解决方案部署指南](https://aka.ms/edgeinferencingdeploy)。</span><span class="sxs-lookup"><span data-stu-id="1659a-171">When you're ready to test the solution example, continue with the [Tiered data for analytics solution deployment guide](https://aka.ms/edgeinferencingdeploy).</span></span> <span data-ttu-id="1659a-172">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="1659a-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
