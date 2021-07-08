---
title: 适用于 Azure 和 Azure Stack Hub 的混合模式与解决方案示例
description: 概述混合模式和解决方案示例，用于在 Azure 和 Azure Stack Hub 上学习和构建混合解决方案。
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343852"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a><span data-ttu-id="124ab-103">适用于 Azure 和 Azure Stack 的混合解决方案模式与示例</span><span class="sxs-lookup"><span data-stu-id="124ab-103">Hybrid solution patterns and examples for Azure and Azure Stack</span></span>

<span data-ttu-id="124ab-104">Microsoft 通过单个一致的 Azure 生态系统提供 Azure 和 Azure Stack 产品与解决方案。</span><span class="sxs-lookup"><span data-stu-id="124ab-104">Microsoft provides Azure and Azure Stack products and solutions as one consistent Azure ecosystem.</span></span> <span data-ttu-id="124ab-105">Microsoft Azure Stack 系列是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="124ab-105">The Microsoft Azure Stack family is an extension of Azure.</span></span>

## <a name="the-hybrid-cloud-and-hybrid-apps"></a><span data-ttu-id="124ab-106">混合云和混合应用</span><span class="sxs-lookup"><span data-stu-id="124ab-106">The hybrid cloud and hybrid apps</span></span>

<span data-ttu-id="124ab-107">Azure Stack 通过实现混合云来为本地环境和 Edge 提供云计算的敏捷性。</span><span class="sxs-lookup"><span data-stu-id="124ab-107">Azure Stack brings the agility of cloud computing to your on-premises environment and the edge by enabling a *hybrid cloud*.</span></span> <span data-ttu-id="124ab-108">Azure Stack Hub、Azure Stack HCI 和 Azure Stack Edge 将 Azure 从云扩展到主权数据中心、分支机构、现场和更远的范围。</span><span class="sxs-lookup"><span data-stu-id="124ab-108">Azure Stack Hub, Azure Stack HCI, and Azure Stack Edge extend Azure from the cloud into your sovereign datacenters, branch offices, field, and beyond.</span></span> <span data-ttu-id="124ab-109">利用这组多样化的功能，可以：</span><span class="sxs-lookup"><span data-stu-id="124ab-109">With this diverse set of capabilities, you can:</span></span>

- <span data-ttu-id="124ab-110">重复使用代码并在 Azure 与本地环境中一致地运行云原生应用。</span><span class="sxs-lookup"><span data-stu-id="124ab-110">Reuse code and run cloud-native apps consistently across Azure and your on-premises environments.</span></span>
- <span data-ttu-id="124ab-111">使用 Azure 服务的可选连接来运行传统的虚拟化工作负荷。</span><span class="sxs-lookup"><span data-stu-id="124ab-111">Run traditional virtualized workloads with optional connections to Azure services.</span></span>
- <span data-ttu-id="124ab-112">将数据传输到云，或将数据保留在主权数据中心，以保持符合性。</span><span class="sxs-lookup"><span data-stu-id="124ab-112">Transfer data to the cloud, or keep it in your sovereign datacenter to maintain compliance.</span></span>
- <span data-ttu-id="124ab-113">运行硬件加速的机器学习、容器化或虚拟化工作负荷，所有这些操作都可以在智能边缘进行。</span><span class="sxs-lookup"><span data-stu-id="124ab-113">Run hardware-accelerated machine-learning, containerized, or virtualized workloads, all at the intelligent edge.</span></span>

<span data-ttu-id="124ab-114">跨云的应用也称为“混合应用”。</span><span class="sxs-lookup"><span data-stu-id="124ab-114">Apps that span clouds are also referred to as *hybrid apps*.</span></span> <span data-ttu-id="124ab-115">你可以在 Azure 中构建混合云应用，并将其部署到位于任何位置的已连接或已断开连接的数据中心。</span><span class="sxs-lookup"><span data-stu-id="124ab-115">You can build hybrid cloud apps in Azure and deploy them to your connected or disconnected datacenter located anywhere.</span></span>

<span data-ttu-id="124ab-116">混合应用方案因适用于开发的资源而差异很大。</span><span class="sxs-lookup"><span data-stu-id="124ab-116">Hybrid app scenarios vary greatly with the resources that are available for development.</span></span> <span data-ttu-id="124ab-117">此外，它们还涉及地理位置、安全性、Internet 访问等考虑因素。</span><span class="sxs-lookup"><span data-stu-id="124ab-117">They also span considerations such as geography, security, internet access, and others.</span></span> <span data-ttu-id="124ab-118">尽管此处所述的解决方案模式和示例可能无法满足所有需求，但它们提供了可供探索的指导原则和示例，并且可以在实施混合解决方案时重复使用。</span><span class="sxs-lookup"><span data-stu-id="124ab-118">Although the solution patterns and examples described here may not address all requirements, they provide guidelines and examples to explore and reuse while implementing hybrid solutions.</span></span>

## <a name="solution-patterns"></a><span data-ttu-id="124ab-119">解决方案模式</span><span class="sxs-lookup"><span data-stu-id="124ab-119">Solution patterns</span></span>

<span data-ttu-id="124ab-120">解决方案模式从真实的客户场景和经验中得出一般化且可重复的设计指导。</span><span class="sxs-lookup"><span data-stu-id="124ab-120">Solution patterns cull generalized repeatable design guidance, from real world customer scenarios and experiences.</span></span> <span data-ttu-id="124ab-121">模式是抽象的，可适用于不同类型的方案或垂直行业。</span><span class="sxs-lookup"><span data-stu-id="124ab-121">A pattern is abstract, allowing it to be applicable to different types of scenarios or vertical industries.</span></span> <span data-ttu-id="124ab-122">每种模式阐述了上下文和问题，并提供解决方案示例的概述。</span><span class="sxs-lookup"><span data-stu-id="124ab-122">Each pattern documents the context and problem, and provides an overview of a solution example.</span></span> <span data-ttu-id="124ab-123">解决方案示例旨在用作模式的可能实施方案。</span><span class="sxs-lookup"><span data-stu-id="124ab-123">The solution example is meant as a possible implementation of the pattern.</span></span>

<span data-ttu-id="124ab-124">模式文章有两种类型：</span><span class="sxs-lookup"><span data-stu-id="124ab-124">There are two types of pattern articles:</span></span>

- <span data-ttu-id="124ab-125">单模式：提供适用于单个通用方案的设计指导。</span><span class="sxs-lookup"><span data-stu-id="124ab-125">Single pattern: provides design guidance for a single general-purpose scenario.</span></span>
- <span data-ttu-id="124ab-126">多模式：提供使用多模式应用程序的设计指导。</span><span class="sxs-lookup"><span data-stu-id="124ab-126">Multi-pattern: provides design guidance where the application of multiple patterns is used.</span></span> <span data-ttu-id="124ab-127">为了解决更复杂的方案或行业特定的问题，我们经常需要使用这种模式。</span><span class="sxs-lookup"><span data-stu-id="124ab-127">This pattern is frequently required for solving more complex scenarios or industry-specific problems.</span></span>

## <a name="solution-deployment-guides"></a><span data-ttu-id="124ab-128">解决方案部署指南</span><span class="sxs-lookup"><span data-stu-id="124ab-128">Solution deployment guides</span></span>

<span data-ttu-id="124ab-129">分步式部署指南可帮助用户部署解决方案示例。</span><span class="sxs-lookup"><span data-stu-id="124ab-129">Step-by-step deployment guides assist in deploying a solution example.</span></span> <span data-ttu-id="124ab-130">该指南也可能会参考 GitHub [解决方案示例存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)中存储的随附代码示例。</span><span class="sxs-lookup"><span data-stu-id="124ab-130">The guide may also refer to a companion code sample, stored in the GitHub [solutions sample repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>

## <a name="next-steps"></a><span data-ttu-id="124ab-131">后续步骤</span><span class="sxs-lookup"><span data-stu-id="124ab-131">Next steps</span></span>

- <span data-ttu-id="124ab-132">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个组合。</span><span class="sxs-lookup"><span data-stu-id="124ab-132">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>
- <span data-ttu-id="124ab-133">浏览目录中的“模式”和“解决方案部署指南”部分，以详细了解每种模式和解决方案。</span><span class="sxs-lookup"><span data-stu-id="124ab-133">Explore the "Patterns" and the "Solution deployment guides" sections of the TOC to learn more about each.</span></span>
- <span data-ttu-id="124ab-134">阅读[混合应用设计注意事项](overview-app-design-considerations.md)，以了解设计、部署和操作混合应用时的软件质量要素。</span><span class="sxs-lookup"><span data-stu-id="124ab-134">Read about [Hybrid app design considerations](overview-app-design-considerations.md) to review pillars of software quality for designing, deploying, and operating hybrid apps.</span></span>
- <span data-ttu-id="124ab-135">[在 Azure Stack 上设置开发环境](/azure-stack/user/azure-stack-dev-start)和在 Azure Stack 上[部署第一个应用](/azure-stack/user/azure-stack-dev-start-deploy-app)。</span><span class="sxs-lookup"><span data-stu-id="124ab-135">[Set up a development environment on Azure Stack](/azure-stack/user/azure-stack-dev-start) and [deploy your first app](/azure-stack/user/azure-stack-dev-start-deploy-app) on Azure Stack.</span></span>
