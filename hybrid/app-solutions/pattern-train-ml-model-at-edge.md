---
title: 在边缘模式训练机器学习模型
description: 了解如何在 Azure 和 Azure Stack 集线器的边缘执行机器学习模型定型。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/17/2020
ms.locfileid: "84909942"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="18bf7-103">在边缘模式训练机器学习模型</span><span class="sxs-lookup"><span data-stu-id="18bf7-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="18bf7-104">从仅在本地存在的数据生成便携式机器学习（ML）模型。</span><span class="sxs-lookup"><span data-stu-id="18bf7-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="18bf7-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="18bf7-105">Context and problem</span></span>

<span data-ttu-id="18bf7-106">很多组织想要使用数据科学家理解的工具从其本地数据或旧数据中解除见解。</span><span class="sxs-lookup"><span data-stu-id="18bf7-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="18bf7-107">[Azure 机器学习](/azure/machine-learning/)提供了用于训练、调整和部署 ML 和深度学习模型的云本机工具。</span><span class="sxs-lookup"><span data-stu-id="18bf7-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="18bf7-108">但是，某些数据太大，无法发送到云，或者出于法规原因无法发送到云。</span><span class="sxs-lookup"><span data-stu-id="18bf7-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="18bf7-109">使用此模式，数据科学家可以使用 Azure 机器学习来使用本地数据和计算来定型模型。</span><span class="sxs-lookup"><span data-stu-id="18bf7-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="18bf7-110">解决方案</span><span class="sxs-lookup"><span data-stu-id="18bf7-110">Solution</span></span>

<span data-ttu-id="18bf7-111">边缘模式的定型使用 Azure Stack 集线器上运行的虚拟机（VM）。</span><span class="sxs-lookup"><span data-stu-id="18bf7-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="18bf7-112">VM 在 Azure ML 中注册为计算目标，从而使其访问仅在本地可用的数据。</span><span class="sxs-lookup"><span data-stu-id="18bf7-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="18bf7-113">在这种情况下，数据存储在 Azure Stack 集线器的 blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="18bf7-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="18bf7-114">在对模型进行训练后，会向 Azure ML 注册，并将其添加到 Azure 容器注册表以进行部署。</span><span class="sxs-lookup"><span data-stu-id="18bf7-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="18bf7-115">对于这种模式迭代，Azure Stack 中心训练 VM 必须可通过公共 internet 访问。</span><span class="sxs-lookup"><span data-stu-id="18bf7-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="18bf7-116">[![在边缘体系结构上训练 ML 模型](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="18bf7-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="18bf7-117">此模式的工作方式如下：</span><span class="sxs-lookup"><span data-stu-id="18bf7-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="18bf7-118">使用 Azure ML 作为计算目标部署和注册 Azure Stack 中心 VM。</span><span class="sxs-lookup"><span data-stu-id="18bf7-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="18bf7-119">在 Azure ML 中创建了一个使用 Azure Stack 中心 VM 作为计算目标的实验。</span><span class="sxs-lookup"><span data-stu-id="18bf7-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="18bf7-120">在对模型进行训练后，将对其进行注册和容器化。</span><span class="sxs-lookup"><span data-stu-id="18bf7-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="18bf7-121">现在可以将模型部署到本地或云中的位置。</span><span class="sxs-lookup"><span data-stu-id="18bf7-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="18bf7-122">组件</span><span class="sxs-lookup"><span data-stu-id="18bf7-122">Components</span></span>

<span data-ttu-id="18bf7-123">此解决方案使用以下组件：</span><span class="sxs-lookup"><span data-stu-id="18bf7-123">This solution uses the following components:</span></span>

| <span data-ttu-id="18bf7-124">层</span><span class="sxs-lookup"><span data-stu-id="18bf7-124">Layer</span></span> | <span data-ttu-id="18bf7-125">组件</span><span class="sxs-lookup"><span data-stu-id="18bf7-125">Component</span></span> | <span data-ttu-id="18bf7-126">说明</span><span class="sxs-lookup"><span data-stu-id="18bf7-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="18bf7-127">Azure</span><span class="sxs-lookup"><span data-stu-id="18bf7-127">Azure</span></span> | <span data-ttu-id="18bf7-128">Azure 机器学习</span><span class="sxs-lookup"><span data-stu-id="18bf7-128">Azure Machine Learning</span></span> | <span data-ttu-id="18bf7-129">[Azure 机器学习](/azure/machine-learning/)协调 ML 模型的培训。</span><span class="sxs-lookup"><span data-stu-id="18bf7-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="18bf7-130">Azure 容器注册表</span><span class="sxs-lookup"><span data-stu-id="18bf7-130">Azure Container Registry</span></span> | <span data-ttu-id="18bf7-131">Azure ML 将模型打包到容器中，并将其存储在[Azure 容器注册表](/azure/container-registry/)中进行部署。</span><span class="sxs-lookup"><span data-stu-id="18bf7-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="18bf7-132">Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="18bf7-132">Azure Stack Hub</span></span> | <span data-ttu-id="18bf7-133">应用服务</span><span class="sxs-lookup"><span data-stu-id="18bf7-133">App Service</span></span> | <span data-ttu-id="18bf7-134">[应用服务 Azure Stack 集线器](/azure-stack/operator/azure-stack-app-service-overview)为边缘提供组件的基础。</span><span class="sxs-lookup"><span data-stu-id="18bf7-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="18bf7-135">计算</span><span class="sxs-lookup"><span data-stu-id="18bf7-135">Compute</span></span> | <span data-ttu-id="18bf7-136">使用 Docker 运行 Ubuntu 的 Azure Stack 集线器 VM 用于训练 ML 模型。</span><span class="sxs-lookup"><span data-stu-id="18bf7-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="18bf7-137">存储</span><span class="sxs-lookup"><span data-stu-id="18bf7-137">Storage</span></span> | <span data-ttu-id="18bf7-138">可以在 Azure Stack Hub blob 存储中承载私有数据。</span><span class="sxs-lookup"><span data-stu-id="18bf7-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="18bf7-139">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="18bf7-139">Issues and considerations</span></span>

<span data-ttu-id="18bf7-140">在决定如何实现此解决方案时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="18bf7-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="18bf7-141">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="18bf7-141">Scalability</span></span>

<span data-ttu-id="18bf7-142">若要启用此解决方案进行缩放，需要在 Azure Stack 集线器上创建适当大小的 VM 以进行培训。</span><span class="sxs-lookup"><span data-stu-id="18bf7-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="18bf7-143">可用性</span><span class="sxs-lookup"><span data-stu-id="18bf7-143">Availability</span></span>

<span data-ttu-id="18bf7-144">确保训练脚本和 Azure Stack 中心 VM 有权访问用于定型的本地数据。</span><span class="sxs-lookup"><span data-stu-id="18bf7-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="18bf7-145">可管理性</span><span class="sxs-lookup"><span data-stu-id="18bf7-145">Manageability</span></span>

<span data-ttu-id="18bf7-146">确保模型和试验经过适当的注册、版本控制和标记，以避免在模型部署过程中出现混乱。</span><span class="sxs-lookup"><span data-stu-id="18bf7-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="18bf7-147">安全性</span><span class="sxs-lookup"><span data-stu-id="18bf7-147">Security</span></span>

<span data-ttu-id="18bf7-148">此模式允许 Azure ML 访问本地可能的敏感数据。</span><span class="sxs-lookup"><span data-stu-id="18bf7-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="18bf7-149">确保用于通过 SSH 连接到 Azure Stack 中心 VM 的帐户具有强密码，并且训练脚本不会将数据保留或上传到云。</span><span class="sxs-lookup"><span data-stu-id="18bf7-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="18bf7-150">后续步骤</span><span class="sxs-lookup"><span data-stu-id="18bf7-150">Next steps</span></span>

<span data-ttu-id="18bf7-151">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="18bf7-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="18bf7-152">有关 ML 和相关主题的概述，请参阅[Azure 机器学习文档](/azure/machine-learning)。</span><span class="sxs-lookup"><span data-stu-id="18bf7-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="18bf7-153">请参阅[Azure 容器注册表](/azure/container-registry/)，了解如何为容器部署生成、存储和管理映像。</span><span class="sxs-lookup"><span data-stu-id="18bf7-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="18bf7-154">请参阅[Azure Stack 集线器上的应用服务](/azure-stack/operator/azure-stack-app-service-overview)，详细了解资源提供程序和部署方式。</span><span class="sxs-lookup"><span data-stu-id="18bf7-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="18bf7-155">有关最佳实践的详细信息，请参阅[混合应用程序设计注意事项](overview-app-design-considerations.md)。</span><span class="sxs-lookup"><span data-stu-id="18bf7-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="18bf7-156">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="18bf7-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="18bf7-157">准备好测试解决方案示例时，请[在边缘部署指南中继续学习训练 ML 模型](https://aka.ms/edgetrainingdeploy)。</span><span class="sxs-lookup"><span data-stu-id="18bf7-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="18bf7-158">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="18bf7-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
