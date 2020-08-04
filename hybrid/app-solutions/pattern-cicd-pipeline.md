---
title: Azure Stack Hub 中的 DevOps 模式
description: 了解 DevOps 模式，以便能够确保 Azure 与 Azure Stack Hub 中的部署保持一致。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: e26056a9507a7467473b009725d4f210d9d59ec8
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477229"
---
# <a name="devops-pattern"></a><span data-ttu-id="c4bdb-103">DevOps 模式</span><span class="sxs-lookup"><span data-stu-id="c4bdb-103">DevOps pattern</span></span>

<span data-ttu-id="c4bdb-104">从单个位置编写代码，并部署到开发、测试和生产环境中可能位于本地数据中心、私有云或公有云的多个目标。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-104">Code from a single location and deploy to multiple targets in development, test, and production environments that may be in your local datacenter, private clouds, or the public cloud.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c4bdb-105">上下文和问题</span><span class="sxs-lookup"><span data-stu-id="c4bdb-105">Context and problem</span></span>

<span data-ttu-id="c4bdb-106">应用程序部署持续性、安全性和可靠性对于组织和开发团队而言至关重要。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-106">Application deployment continuity, security, and reliability are essential to organizations and critical to development teams.</span></span>

<span data-ttu-id="c4bdb-107">应用通常需要重构的代码才能在每个目标环境中运行。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-107">Apps often require refactored code to run in each target environment.</span></span> <span data-ttu-id="c4bdb-108">这意味着应用不一定完全可移植。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-108">This means that an app isn't completely portable.</span></span> <span data-ttu-id="c4bdb-109">它在每个环境之间移动时，必须进行更新、测试和验证。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-109">It must be updated, tested, and validated as it moves through each environment.</span></span> <span data-ttu-id="c4bdb-110">例如，在开发环境中编写的代码必须经过重写才能在测试环境中运行，且在最后进入生产环境时也必须重写。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-110">For example, code written in a development environment must then be rewritten to work in a test environment and rewritten when it finally lands in a production environment.</span></span> <span data-ttu-id="c4bdb-111">此外，此代码绑定到具体的主机。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-111">Furthermore, this code is specifically tied to the host.</span></span> <span data-ttu-id="c4bdb-112">这可能会提高维护应用的成本和复杂性。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-112">This increases the cost and complexity of maintaining your app.</span></span> <span data-ttu-id="c4bdb-113">应用的每个版本绑定到每个环境。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-113">Each version of the app is tied to each environment.</span></span> <span data-ttu-id="c4bdb-114">提高的复杂性和重复性使安全和代码质量承受更高的风险。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-114">The increased complexity and duplication increase the risk of security and code quality.</span></span> <span data-ttu-id="c4bdb-115">此外，在删除还原失败的主机或部署其他主机以应对增大的需求时，代码无法直接重新部署。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-115">In addition, the code can't be readily redeployed when you remove restore failed hosts or deploy additional hosts to handle increases in demand.</span></span>

## <a name="solution"></a><span data-ttu-id="c4bdb-116">解决方案</span><span class="sxs-lookup"><span data-stu-id="c4bdb-116">Solution</span></span>

<span data-ttu-id="c4bdb-117">可以使用 DevOps 模式来生成、测试和部署在多个云上运行的应用。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-117">The DevOps Pattern enables you to build, test, and deploy an app that runs on multiple clouds.</span></span> <span data-ttu-id="c4bdb-118">此模式统合了持续集成和持续交付的做法。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-118">This pattern unites the practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="c4bdb-119">借助持续集成，每当团队成员对版本控制提交更改时，都会生成并测试代码。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-119">With continuous integration, code is built and tested every time a team member commits a change to version control.</span></span> <span data-ttu-id="c4bdb-120">持续交付在生产环境中将每个生成步骤自动化。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-120">Continuous delivery automates each step from a build to a production environment.</span></span> <span data-ttu-id="c4bdb-121">这些过程共同建立了一个发布过程，可支持跨不同环境的部署。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-121">Together, these processes create a release process that supports deployment across diverse environments.</span></span> <span data-ttu-id="c4bdb-122">使用此模式可以草拟代码，然后将相同的代码部署到本地环境、不同的私有云和公有云。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-122">With this pattern, you can draft your code and then deploy the same code to a premise environment, different private clouds, and the public clouds.</span></span> <span data-ttu-id="c4bdb-123">环境中的差异需要对配置文件进行更改，而不是更改代码。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-123">Differences in environment require a change to a configuration file rather than changes to the code.</span></span>

![DevOps 模式](media/pattern-cicd-pipeline/hybrid-ci-cd.png)

<span data-ttu-id="c4bdb-125">使用在本地、私有云和公有云环境之间一致的一组开发工具可以实现持续集成和持续交付的做法。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-125">With a consistent set of development tools across on-premises, private cloud, and public cloud environments, you can implement a practice of continuous integration and continuous delivery.</span></span> <span data-ttu-id="c4bdb-126">使用 DevOps 模式部署的应用和服务可互换，可在其中任一位置运行，并可利用本地和公有云的特性与功能。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-126">Apps and services deployed using the DevOps Pattern are interchangeable and can run in any of these locations, taking advantage of on-premises and public cloud features and capabilities.</span></span>

<span data-ttu-id="c4bdb-127">使用 DevOps 发布管道有助于：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-127">Using a DevOps release pipeline helps you:</span></span>

- <span data-ttu-id="c4bdb-128">根据单个存储库中提交的代码启动新的生成。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-128">Initiate a new build based on code commits to a single repository.</span></span>
- <span data-ttu-id="c4bdb-129">自动将新生成的代码部署到公有云进行用户验收测试。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-129">Automatically deploy your newly built code to the public cloud for user acceptance testing.</span></span>
- <span data-ttu-id="c4bdb-130">代码通过测试后，可自动部署到私有云。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-130">Automatically deploy to a private cloud once your code has passed testing.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c4bdb-131">问题和注意事项</span><span class="sxs-lookup"><span data-stu-id="c4bdb-131">Issues and considerations</span></span>

<span data-ttu-id="c4bdb-132">DevOps 模式旨在确保无论目标环境为何，都能实现跨部署的一致性。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-132">The DevOps Pattern is intended to ensure consistency across deployments regardless of the target environment.</span></span> <span data-ttu-id="c4bdb-133">但是，云和本地环境中的功能有所不同。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-133">However, capabilities vary across cloud and on-premises environments.</span></span> <span data-ttu-id="c4bdb-134">请考虑以下要点：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-134">Consider the following points:</span></span>

- <span data-ttu-id="c4bdb-135">部署中的函数、终结点、服务和其他资源在目标部署位置是否可用？</span><span class="sxs-lookup"><span data-stu-id="c4bdb-135">Are the functions, endpoints, services, and other resources in your deployment available in the target deployment locations?</span></span>
- <span data-ttu-id="c4bdb-136">配置项目是否存储在可跨云访问的位置？</span><span class="sxs-lookup"><span data-stu-id="c4bdb-136">Are configuration artifacts stored in locations that are accessible across clouds?</span></span>
- <span data-ttu-id="c4bdb-137">部署参数是否可在所有目标环境中正常运行？</span><span class="sxs-lookup"><span data-stu-id="c4bdb-137">Will deployment parameters work in all the target environments?</span></span>
- <span data-ttu-id="c4bdb-138">是否在所有目标云中都可以使用资源特定的属性？</span><span class="sxs-lookup"><span data-stu-id="c4bdb-138">Are resource-specific properties available in all target clouds?</span></span>

<span data-ttu-id="c4bdb-139">有关详细信息，请参阅[开发用于实现云一致性的 Azure 资源管理器模板](/azure/azure-resource-manager/templates-cloud-consistency)。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-139">For more information, see [Develop Azure Resource Manager templates for cloud consistency](/azure/azure-resource-manager/templates-cloud-consistency).</span></span>

<span data-ttu-id="c4bdb-140">此外，在决定如何实现此模式时，请考虑以下几点：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-140">In addition, consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="c4bdb-141">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="c4bdb-141">Scalability</span></span>

<span data-ttu-id="c4bdb-142">部署自动化系统是 DevOps 模式中的关键控制点。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-142">Deployment automation systems are the key control point in the DevOps Patterns.</span></span> <span data-ttu-id="c4bdb-143">实现方式可能有所不同。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-143">Implementations can vary.</span></span> <span data-ttu-id="c4bdb-144">哪个服务器大小合适取决于预期工作负荷的大小。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-144">The selection of the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="c4bdb-145">VM 的缩放成本高于容器。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-145">VMs cost more to scale than containers.</span></span> <span data-ttu-id="c4bdb-146">但是，若要使用容器进行缩放，则必须配合容器运行生成过程。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-146">To use containers for scaling, however, your build process must run with containers.</span></span>

### <a name="availability"></a><span data-ttu-id="c4bdb-147">可用性</span><span class="sxs-lookup"><span data-stu-id="c4bdb-147">Availability</span></span>

<span data-ttu-id="c4bdb-148">DevPattern 上下文中的可用性是指能够恢复与工作流关联的任何状态信息，例如测试结果、代码依赖项，或其他项目。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-148">Availability in the context of the DevPattern means being able to recover any state information associated with your workflow, such as test results, code dependencies, or other artifacts.</span></span> <span data-ttu-id="c4bdb-149">若要评估可用性要求，请考虑两个常见指标：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-149">To assess your availability requirements, consider two common metrics:</span></span>

- <span data-ttu-id="c4bdb-150">恢复时间目标 (RTO) 指定允许系统发生故障多长时间。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-150">Recovery Time Objective (RTO) specifies how long you can go without a system.</span></span>

- <span data-ttu-id="c4bdb-151">恢复点目标 (RPO) 表示服务中断影响到系统时，允许丢失多少数据。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-151">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects the system.</span></span>

<span data-ttu-id="c4bdb-152">在实践中，RTO 和 RPO 涉及到冗余和备份。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-152">In practice, RTO, and RPO imply redundancy and backup.</span></span> <span data-ttu-id="c4bdb-153">在全局 Azure 云上，可用性不是硬件恢复的问题（这是 Azure 的部分责任），而是为了确保维持 DevOps 系统的状态。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-153">On the global Azure cloud, availability isn't a question of hardware recovery—that's part of Azure—but rather ensuring you maintain the state of your DevOps systems.</span></span> <span data-ttu-id="c4bdb-154">在 Azure Stack Hub 上，硬件恢复可能是一个考虑因素。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-154">On Azure Stack Hub, hardware recovery may be a consideration.</span></span>

<span data-ttu-id="c4bdb-155">设计用于部署自动化的系统时，另一个主要考虑因素是将服务部署到云环境所需的访问控制和适当的权限管理。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-155">Another major consideration when designing the system used for deployment automation is access control and the proper management of the rights needed to deploy services to cloud environments.</span></span> <span data-ttu-id="c4bdb-156">创建、删除或修改部署时需要哪些权限？</span><span class="sxs-lookup"><span data-stu-id="c4bdb-156">What rights are needed to create, delete, or modify deployments?</span></span> <span data-ttu-id="c4bdb-157">例如，在 Azure 中创建资源组时通常需要一组权限，而在资源组中部署服务时则需要另一组权限。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-157">For example, one set of rights is typically required to create a resource group in Azure and another to deploy services in the resource group.</span></span>

### <a name="manageability"></a><span data-ttu-id="c4bdb-158">可管理性</span><span class="sxs-lookup"><span data-stu-id="c4bdb-158">Manageability</span></span>

<span data-ttu-id="c4bdb-159">基于 DevOps 模式设计任何系统时，必须考虑整个组合中各项服务的自动化、日志记录和警报。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-159">The design of any system based on the DevOps pattern must consider automation, logging, and alerting for each service across the portfolio.</span></span> <span data-ttu-id="c4bdb-160">请使用共享服务和/或应用程序团队，并跟踪安全策略和监管情况。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-160">Use shared services, an application team, or both, and track security policies and governance as well.</span></span>

<span data-ttu-id="c4bdb-161">在 Azure 或 Azure Stack Hub 上的独立资源组中部署生产/测试环境。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-161">Deploy production environments and development/test environments in separate resource groups on Azure or Azure Stack Hub.</span></span> <span data-ttu-id="c4bdb-162">然后，可以监视每个环境的资源，并按资源组汇总计费成本。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-162">Then you can monitor each environment's resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="c4bdb-163">还可以删除作为集的资源，这适用于测试部署。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-163">You can also delete resources as a set, which is useful for test deployments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c4bdb-164">何时使用此模式</span><span class="sxs-lookup"><span data-stu-id="c4bdb-164">When to use this pattern</span></span>

<span data-ttu-id="c4bdb-165">使用此模式的时机：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-165">Use this pattern if:</span></span>

- <span data-ttu-id="c4bdb-166">可以在一个符合开发人员需求的环境中开发代码，并部署到特定于解决方案的、可能难以开发新代码的环境中。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-166">You can develop code in one environment that meets the needs of your developers, and deploy to an environment specific to your solution where it may be difficult to develop new code.</span></span>
- <span data-ttu-id="c4bdb-167">可以使用开发人员所需的代码和工具，只要他们能够遵守 DevOps 模式中的持续集成和持续交付过程即可。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-167">You can use the code and tools your developers would like, as long as they're able to follow the continuous integration and continuous delivery process in the DevOps Pattern.</span></span>

<span data-ttu-id="c4bdb-168">不推荐使用此模式：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-168">This pattern isn't recommended:</span></span>

- <span data-ttu-id="c4bdb-169">如果无法自动完成基础结构、预配资源、配置、标识和安全任务。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-169">If you can't automate infrastructure, provisioning resources, configuration, identity, and security tasks.</span></span>
- <span data-ttu-id="c4bdb-170">如果团队无权访问混合云资源，因此无法实施持续集成/持续开发 (CI/CD) 方法。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-170">If teams don't have access to hybrid cloud resources to implement a Continuous Integration/Continuous Development (CI/CD) approach.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c4bdb-171">后续步骤</span><span class="sxs-lookup"><span data-stu-id="c4bdb-171">Next steps</span></span>

<span data-ttu-id="c4bdb-172">若要详细了解本文中介绍的主题：</span><span class="sxs-lookup"><span data-stu-id="c4bdb-172">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="c4bdb-173">请参阅 [Azure DevOps 文档](/azure/devops)来详细了解 Azure DevOps 和相关工具，包括 Azure Repos 与 Azure Pipelines。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-173">See the [Azure DevOps documentation](/azure/devops) to learn more about Azure DevOps and related tools, including Azure Repos, and Azure Pipelines.</span></span>
- <span data-ttu-id="c4bdb-174">请参阅 [Azure Stack 产品和解决方案系列](/azure-stack)，详细了解产品和解决方案的整个阵容。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-174">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="c4bdb-175">准备好测试解决方案示例时，请继续阅读 [DevOps 混合 CI/CD 解决方案部署指南](https://aka.ms/hybriddevopsdeploy)。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-175">When you're ready to test the solution example, continue with the [DevOps hybrid CI/CD solution deployment guide](https://aka.ms/hybriddevopsdeploy).</span></span> <span data-ttu-id="c4bdb-176">该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-176">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="c4bdb-177">了解如何使用混合持续集成/持续交付 (CI/CD) 管道将应用部署到 Azure 和 Azure Stack Hub。</span><span class="sxs-lookup"><span data-stu-id="c4bdb-177">You learn how to deploy an app to Azure and Azure Stack Hub using a hybrid continuous integration/continuous delivery (CI/CD) pipeline.</span></span>
