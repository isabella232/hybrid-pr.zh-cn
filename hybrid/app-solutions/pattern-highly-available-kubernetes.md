---
title: 使用 Azure 和 Azure Stack Hub 的高可用性 Kubernetes 模式
description: 了解 Kubernetes 群集解决方案如何使用 Azure 和 Azure Stack Hub 提供高可用性。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281306"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>高可用性 Kubernetes 群集模式

本文介绍如何在 Azure Stack Hub 上使用 Azure Kubernetes 服务 (AKS) 引擎构建和运行基于 Kubernetes 的高可用性基础结构。 对于在高度受限和受监管的环境中具有关键工作负载的组织来说，这种情形很常见。 金融、国防和政府等领域的组织。

## <a name="context-and-problem"></a>上下文和问题

许多组织正在开发利用先进服务和技术（如 Kubernetes）的云原生解决方案。 虽然 Azure 在世界上大多数区域都提供数据中心，但有时也存在一些边缘用例和场景，其中业务关键型应用程序必须在特定位置运行。 考虑因素包括：

- 位置敏感性
- 应用程序和本地系统之间的延迟
- 带宽节省
- 连接
- 法规或法定要求

Azure 与 Azure Stack Hub 结合使用即可解决这其中绝大部分问题。 下面介绍了成功实现在 Azure Stack Hub 上运行 Kubernetes 的一系列选项、决策和注意事项。

## <a name="solution"></a>解决方案

此模式假设必须处理一组严格的约束。 应用程序必须在本地运行，并且所有个人数据都不得访问公有云服务。 监视和其他非 PII 数据可以发送到 Azure 并在其中进行处理。 公共容器注册表等外部服务可以进行访问，但可能会通过防火墙或代理服务器进行筛选。

此处所示的示例应用程序（基于 [Azure Kubernetes 服务研讨会](/learn/modules/aks-workshop/)）旨在尽可能使用 Kubernetes 原生解决方案。 此设计可以避免供应商锁定，而不是使用平台原生服务。 例如，应用程序使用自承载 MongoDB 数据库后端，而不是 PaaS 服务或外部数据库服务。

[![应用程序模式混合](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

上图说明了在 Azure Stack Hub 的 Kubernetes 上运行的示例应用程序的应用程序体系结构。 该应用由几个组件组成，包括：

 1) Azure Stack Hub 上基于 AKS 引擎的 Kubernetes 群集。
 2) [cert-manager](https://www.jetstack.io/cert-manager/)，为 Kubernetes 中的证书管理提供了一套工具，用于自动从 Let's Encrypt 请求证书。
 3) Kubernetes 命名空间，包含前端 (ratings-web)、API (ratings-api) 和数据库 (ratings-mongodb) 的应用程序组件。
 4) 将 HTTP/HTTPS 流量路由到 Kubernetes 群集中的终结点的入口控制器。

示例应用程序用于说明应用程序体系结构。 所有组件都是示例。 该体系结构只包含一种应用程序部署。 为实现高可用性 (HA)，我们将在两个不同的 Azure Stack Hub 实例上至少运行两次部署，它们可以在相同的位置运行，也可以在两个（或多个）不同的位置运行：

![基础结构体系结构](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Azure 容器注册表、Azure Monitor 等服务托管在 Azure 中的 Azure Stack Hub 之外或本地。 这种混合设计可保护解决方案不受单个 Azure Stack Hub 实例中断的影响。

## <a name="components"></a>组件

整体体系结构包括以下组件：

Azure Stack Hub 是 Azure 的扩展，它通过在数据中心提供 Azure 服务，可以在本地环境中运行工作负载。 请转到·[Azure Stack Hub 概述](/azure-stack/operator/azure-stack-overview)，了解详细信息。

Azure Kubernetes 服务引擎（AKS 引擎）是托管 Kubernetes 服务产品 Azure Kubernetes 服务 (AKS) 背后的引擎，Azure 中现已提供该产品。 对于 Azure Stack Hub，AKS 引擎允许我们使用 Azure Stack Hub 的 IaaS 功能部署、扩展和升级功能齐全的自托管 Kubernetes 群集。 请转到·[AKS 引擎概述](https://github.com/Azure/aks-engine)，了解详细信息。

若要详细了解 Azure 上的 AKS 引擎与 Azure Stack Hub 上的 AKS 引擎之间的差异，请转到[已知问题和限制](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations)。

Azure 虚拟网络 (VNet) 用于在每个 Azure Stack Hub 上为托管 Kubernetes 群集基础结构的虚拟机 (VM) 提供网络基础结构。

Azure 负载均衡器用于 Kubernetes API 终结点和 Nginx 入口控制器。 负载均衡器将外部（例如 Internet）流量路由到提供特定服务的节点和 VM。

Azure 容器注册表 (ACR) 用于存储部署到群集中的 Docker 映像和 Helm 图表。 AKS 引擎可以使用 Azure AD 标识对容器注册表进行身份验证。 Kubernetes 不需要 ACR。 可以使用其他容器注册表，例如 Docker 中心。

Azure Repos 是一组版本控制工具，可以用来管理代码。 还可以使用 GitHub 或其他基于 Git 的存储库。 请转到·[Azure Repos 概述](/azure/devops/repos/get-started/what-is-repos)，了解详细信息。

Azure Pipelines 是 Azure DevOps Services 的一部分，可运行自动化的生成、测试和部署。 也可以使用 Jenkins 等第三方 CI/CD 解决方案。 请转到·[Azure Pipelines 概述](/azure/devops/pipelines/get-started/what-is-azure-pipelines)，了解详细信息。

Azure Monitor 收集并存储指标和日志，包括解决方案中 Azure 服务的平台指标，以及应用程序遥测数据。 使用此数据可以监视应用程序、设置警报和仪表板，以及针对故障执行根本原因分析。 Azure Monitor 与 Kubernetes 相集成，可以从控制器、节点和容器收集指标，并可以收集容器日志和主节点日志。 请转到·[Azure Monitor 概述](/azure/azure-monitor/overview)，了解详细信息。

Azure 流量管理器是一种基于 DNS 的流量负载均衡器，借助它可以跨不同 Azure 区域或 Azure Stack Hub 部署以最佳方式向服务分发流量。 流量管理器还提供高可用性和响应能力。 应用程序终结点必须可从外部访问。 还有其他本地解决方案可用。

Kubernetes 入口控制器会向 Kubernetes 群集中的服务公开 HTTP(S) 路由。 可以使用 Nginx 或任何适合的入口控制器来实现此目的。

Helm 是 Kubernetes 部署的包管理器，提供了一种将不同 Kubernetes 对象（如部署、服务、机密）捆绑到一个“图表”中的方法。 可以发布、部署、控制版本管理和更新图表对象。 Azure 容器注册表可以用作存储打包的 Helm 图表的存储库。

## <a name="design-considerations"></a>设计注意事项

此模式遵循一些大致注意事项，将在本文的下面部分中进行详细说明：

- 该应用程序使用 Kubernetes 原生解决方案，避免供应商锁定。
- 该应用程序使用微服务体系结构。
- Azure Stack Hub 不需要入站连接，但允许出站 Internet 连接。

这些建议的做法也适用于实际工作负载和场景。

## <a name="scalability-considerations"></a>可伸缩性注意事项

若要为用户提供对应用程序的一致、可靠且性能良好的访问，可伸缩性非常重要。

该示例场景涵盖了应用程序堆栈在多层上的可伸缩性。 下面是不同层的大致概述：

| 体系结构级别 | 影响 | 如何操作？ |
| --- | --- | ---
| 应用程序 | 应用程序 | 基于 Pod/副本/容器实例数量的横向缩放* |
| 群集 | Kubernetes 群集 | 节点数（1 至 50）、VM-SKU 大小和节点池（Azure Stack Hub 上的 AKS 引擎目前仅支持单个节点池）；使用 AKS 引擎的缩放命令（手动） |
| 基础结构 | Azure Stack Hub | Azure Stack Hub 部署中的节点数、容量和缩放单元 |

\* 使用 Kubernetes 的水平 Pod 自动缩放程序 (HPA)；通过调整容器实例的大小（CPU/内存）来实现基于指标的自动缩放或纵向缩放。

Azure Stack Hub（基础结构级别）

Azure Stack Hub 基础结构是此实现的基础，因为 Azure Stack Hub 在数据中心的物理硬件上运行。 选择 Hub 硬件时，需要选择 CPU、内存密度、存储配置和服务器数量。 若要详细了解 Azure Stack Hub 的可伸缩性，请查看以下资源：

- [Azure Stack Hub 的容量计划概述](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [在 Azure Stack Hub 中添加更多的缩放单元节点](/azure-stack/operator/azure-stack-add-scale-node)

Kubernetes 群集（群集级别）

Kubernetes 群集本身由 Azure (Stack) IaaS 组件（包括计算、存储和网络资源）组成，并在这些组件的基础上生成。 Kubernetes 解决方案涉及主节点和工作器节点，它们作为 VM 部署在 Azure（和 Azure Stack Hub）中。

- [控制平面节点](/azure/aks/concepts-clusters-workloads#control-plane)（主节点）提供核心 Kubernetes 服务和应用程序工作负载的业务流程。
- [工作器节点](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)（工作器节点）运行应用程序工作负载。

为初始部署选择 VM 大小时，需要注意以下事项：  

- **成本** - 在规划工作器节点时，请记住每个 VM 将产生的总成本。 例如，如果应用程序工作负载需要有限的资源，则应计划部署规模较小的 VM。 与 Azure 一样，Azure Stack Hub 通常按使用量计费，因此，针对 Kubernetes 角色适当调整 VM 大小对于优化使用成本至关重要。 

- **可伸缩性** - 群集的可伸缩性通过以下方式实现：增加和减少主节点和工作器节点的数量，或添加其他节点池（目前 Azure Stack Hub 上不可用）。 群集缩放可以根据使用容器见解 (Azure Monitor + Log Analytics) 收集的性能数据来实现。 

    如果应用程序需要更多（或更少）资源，则可以横向扩展（或横向缩减）当前节点（介于 1 到 50 个节点之间）。 如果需要 50 个以上的节点，则可以在单独的订阅中创建其他群集。 若不重新部署群集，就不能将实际 VM 垂直（纵向）扩展到其他 VM 大小。

    缩放是使用最初用于部署 Kubernetes 群集的 AKS 引擎帮助程序 VM 手动完成的。 有关详细信息，请参阅[缩放 Kubernetes 群集](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **配额** - 请考虑在 Azure Stack Hub 上计划 AKS 部署时所配置的 [配额](/azure-stack/operator/azure-stack-quota-types)。 请确保每个[订阅](/azure-stack/operator/service-plan-offer-subscription-overview)都具有适当计划并配置了配额。 订阅将需要容纳群集横向扩展所需的计算、存储和其他服务的数量。

- **应用程序工作负载** - 请参阅 Azure Kubernetes 服务的 Kubernetes 核心概念中的 [群集和工作负载概念](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools)。 本文将帮助你根据应用程序的计算和内存需求来确定合适的 VM 大小。  

**应用程序（应用程序级别）**

在应用程序层上，我们使用 Kubernetes [水平 Pod 自动缩放程序 (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)。 HPA 可以根据不同指标（例如 CPU 利用率）来增加或减少部署中的副本（Pod/容器实例）数量。

另一种方法是垂直缩放容器实例。 可以通过更改特定部署所请求的和可用的 CPU 和内存数量来实现此举。 请参阅 kubernetes.io 上的[管理容器资源](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)以了解详细信息。

## <a name="networking-and-connectivity-considerations"></a>网络和连接性注意事项

网络和连接性也会影响前面提到的 Azure Stack Hub 上的 Kubernetes 的三种层。 下表显示了这些层以及其中包含的服务：

| 层 | 影响 | 什么？ |
| --- | --- | ---
| 应用程序 | 应用程序 | 如何访问应用程序？ 是否会向 Internet 公开？ |
| 群集 | Kubernetes 群集 | Kubernetes API、AKS 引擎 VM、拉取容器映像（流出量）、发送监视数据和遥测（流出量） |
| 基础结构 | Azure Stack Hub | Azure Stack Hub 管理终结点（如门户网站和 Azure 资源管理器终结点）的可访问性。 |

**应用程序**

对于应用程序层，最重要的注意事项是应用程序是否公开并可从 Internet 访问。 从 Kubernetes 的角度来看，Internet 可访问性意味着使用 Kubernetes 服务或入口控制器公开部署或 Pod。

> [!NOTE]
> 建议使用入口控制器公开 Kubernetes 服务，因为 Azure Stack Hub 上的前端公共 IP 数限制为 5。 这种设计还将 Kubernetes 服务（类型为 LoadBalancer）限制为 5，这对于许多部署来说都过小。 请转到 [AKS 引擎文档](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips)，了解详细信息。

通过负载均衡器或入口控制器使用公共 IP 公开应用程序并不意味着现在可以通过 Internet 访问该应用程序。 Azure Stack Hub 可能有一个仅在本地 Intranet 上可见的公共 IP 地址，这是因为并非所有公共 IP 都真正面向 Internet。

上一个块考虑应用程序的入口流量。 成功的 Kubernetes 部署必须考虑的另一个主题是出站/出口流量。 以下是需要出口流量的几个用例：

- 拉取存储在 DockerHub 或 Azure 容器注册表中的容器映像
- 检索 Helm 图表
- 发出 Application Insights 数据（或其他监视数据）

某些企业环境可能需要使用透明或非透明代理服务器 。 这些服务器需要在群集的各个组件上进行特定配置。 AKS 引擎文档包含有关如何适应网络代理的各种详细信息。 有关更多详细信息，请参阅 [AKS 引擎和代理服务器](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

最后，跨群集流量必须在 Azure Stack Hub 实例之间流动。 该示例部署由在各个 Azure Stack Hub 实例上运行的各个 Kubernetes 群集组成。 它们之间的流量（例如两个数据库之间的复制流量）是“外部流量”。 必须通过站点到站点 VPN 或 Azure Stack Hub 公共 IP 地址路由外部流量，以连接两个 Azure Stack Hub 实例上的 Kubernetes：

![群集间和群集内流量](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Cluster**  

Kubernetes 群集不一定需要可通过 Internet 访问。 相关部分是用于操作群集的 Kubernetes API，例如使用 `kubectl`。 Kubernetes API 终结点必须可供所有操作群集或部署以群集为基础的应用程序和服务的用户访问。 下面的[部署 (CI/CD) 注意事项](#deployment-cicd-considerations)部分从 DevOps 角度更详细地介绍了这一主题。

在群集级别，还需要考虑有关出口流量的一些注意事项：

- 节点更新（对于 Ubuntu）
- 监视数据（发送到 Azure LogAnalytics）
- 需要出站流量的其他代理（特定于每个部署人员的环境）

使用 AKS 引擎部署 Kubernetes 群集之前，请规划最终网络设计。 将群集部署到现有网络可能更高效，而不是创建专用虚拟网络。 例如，你可以利用已在 Azure Stack Hub 环境中配置的现有站点到站点 VPN 连接。

**基础结构**  

基础结构是指访问 Azure Stack Hub 管理终结点。 终结点包括租户和管理门户，以及 Azure 资源管理器管理员和租户终结点。 需要这些终结点才能操作 Azure Stack Hub 及其核心服务。

## <a name="data-and-storage-considerations"></a>数据和存储注意事项

我们的应用程序的两个实例将跨两个 Azure Stack Hub 实例部署在两个单独的 Kubernetes 群集上。 这种设计将需要我们考虑如何在实例之间复制和同步数据。

Azure 向我们提供了内置功能，可以跨云中的多个区域和地区复制存储。 当前，对于 Azure Stack Hub，没有原生方式可用于在两个不同的 Azure Stack Hub 实例之间复制存储，这两个实例会形成两个独立的云，没有办法将其作为一个集合来管理。 规划跨 Azure Stack Hub 运行的应用程序的复原能力会迫使你在应用程序设计和部署中考虑这种独立性。

在大多数情况下，对于在 AKS 上部署的可复原和高可用性应用程序，存储复制不是必需的。 但在应用程序设计中，应考虑每个 Azure Stack Hub 实例的独立存储。 如果这种设计是在 Azure Stack Hub 上部署解决方案的一种顾虑或障碍，或许 Microsoft 合作伙伴的解决方案可提供存储附件。 存储附件提供了跨多个 Azure Stack Hub 和 Azure 的存储复制解决方案。 有关详细信息，请参阅[合作伙伴解决方案](#partner-solutions)。

在我们的体系结构中，考虑了以下各层：

**配置**

配置包括 Azure Stack Hub、AKS 引擎以及 Kubernetes 群集本身的配置。 配置应尽可能自动化，并以基础结构即代码的形式存储在基于 Git 的版本控制系统（如 Azure DevOps 或 GitHub）中。 这些设置不能轻松在多种部署之间同步。 因此，我们建议在外部存储和应用配置，并使用 DevOps 管道。

**应用程序**

应用程序应存储在基于 Git 的存储库中。 每当进行新部署、对应用程序进行更改或进行灾难恢复时，都可以使用 Azure Pipelines 轻松进行部署。

数据

在大多数应用程序设计中，数据是最重要的考虑因素。 应用程序数据必须在应用程序的不同实例之间保持同步。 如果发生中断，数据还需要备份和灾难恢复策略。

这种设计的实现很大程度上取决于技术选择。 下面是一些在 Azure Stack Hub 上以高可用性方式实现数据库的解决方案示例：

- [将 SQL Server 2016 可用性组部署到 Azure 和 Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [将高度可用的 MongoDB 解决方案部署到 Azure 和 Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

对于高可用性和可复原解决方案，跨多个位置处理数据时的注意事项甚至更为复杂。 请注意以下几点：

- Azure Stack Hub 之间的延迟和网络连接。
- 服务和权限的标识可用性。 每个 Azure Stack Hub 实例都与一个外部目录集成。 在部署期间，可选择使用 Azure Active Directory (Azure AD) 或Active Directory 联合身份验证服务 (ADFS)。 这样，就有可能仅使用一个标识，该标识可与多个独立的 Azure Stack Hub 实例进行交互。

## <a name="business-continuity-and-disaster-recovery"></a>业务连续性和灾难恢复

业务连续性和灾难恢复 (BCDR) 是 Azure Stack Hub 和 Azure 中的一个重要主题。 主要区别在于，在 Azure Stack Hub，操作员必须管理整个 BCDR 过程。 在 Azure 中，BCDR 的部件由 Microsoft 自动管理。

BCDR 会影响前一部分[数据和存储注意事项](#data-and-storage-considerations)中提到的相同区域：

- 基础结构/配置
- 应用程序可用性
- 应用程序数据

如前一部分中所述，这些区域由 Azure Stack Hub 操作员负责，可能会因组织而异。 根据可用的工具和进程规划 BCDR。

**基础结构和配置**

本部分介绍物理和逻辑基础结构以及 Azure Stack Hub 的配置。 其中包括管理员和租户空间中的操作。

Azure Stack Hub 操作员（或管理员）负责维护 Azure Stack Hub 实例。 包括网络、存储、标识等组件，以及不在本文讨论范围内的其他主题。 若要详细了解 Azure Stack Hub 操作的具体信息，请参阅以下资源：

- [使用基础结构备份服务恢复 Azure Stack Hub 中的数据](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [从管理员门户为 Azure Stack Hub 启用备份](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [在发生灾难性数据丢失后进行恢复](/azure-stack/operator/azure-stack-backup-recover-data)
- [基础结构备份服务最佳做法](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub 是用于部署 Kubernetes 应用程序的平台和结构。 Kubernetes 应用程序的应用程序所有者将是 Azure Stack Hub 的用户，该用户有权部署解决方案所需的应用程序基础结构。 本例中的应用程序基础结构是指使用 AKS 引擎部署的 Kubernetes 群集以及周边服务。 这些组件将部署到 Azure Stack Hub，受 Azure Stack Hub 产品/服务的约束。 确保 Kubernetes 应用程序所有者接受的产品/服务具有足够的容量（以 Azure Stack Hub 配额表示）来部署整个解决方案。 前一部分中给出了建议，应使用基础结构即代码和 Azure DevOps Azure Pipelines 等部署管道来自动进行应用程序部署。

有关 Azure Stack Hub 产品/服务和配额的详细信息，请参阅 [Azure Stack Hub 服务、计划、产品/服务和订阅概述](/azure-stack/operator/service-plan-offer-subscription-overview)

务必安全地保存并存储 AKS 引擎配置，包括其输出。 这些文件包含用于访问 Kubernetes 群集的机密信息，因此必须对其进行保护，以免向非管理员公开。

**应用程序可用性**

应用程序不应依赖于已部署实例的备份。 作为标准做法，请完全按照基础结构即代码模式重新部署应用程序。 例如，使用 Azure DevOps Azure Pipelines 重新部署。 BCDR 过程应包括将应用程序重新部署到同一个或另一个 Kubernetes 群集。

**应用程序数据**

应用程序数据是最大程度减少数据丢失的关键部分。 前一部分介绍了在应用程序的两个（或多个）实例之间复制和同步数据的技术。 根据用于存储数据的数据库基础结构（MySQL、MongoDB、MSSQL 或其他），将有不同的数据库可用性和备份技术可供选择。

为实现完整性，建议使用以下其中一种方法：
- 特定数据库的原生备份解决方案。
- 正式支持备份和恢复应用程序使用的数据库类型的备份解决方案。

> [!IMPORTANT]
> 不要将备份数据存储在应用程序数据所在的同一 Azure Stack Hub 实例上。 Azure Stack Hub 实例的完全中断也将损害你的备份。

## <a name="availability-considerations"></a>可用性注意事项

通过 AKS 引擎部署的 Azure Stack Hub 上的 Kubernetes 不是托管服务， 而是使用 Azure 基础结构即服务 (IaaS) 的 Kubernetes 群集的自动部署和配置。 因此，它提供与基础结构相同的可用性。

Azure Stack Hub 基础结构已具备从故障种恢复的能力，并提供了可用性集等功能，可以跨多个[容错域和更新域](/azure-stack/user/azure-stack-vm-considerations#high-availability)分布组件。 但是，如果出现硬件故障，底层技术（故障转移群集）仍然会导致受影响物理服务器上的 VM 发生故障。

最好将生产 Kubernetes 群集和工作负载部署到两个（或多个）群集。 这些群集应托管在不同的位置或数据中心，并使用 Azure 流量管理器等技术根据群集响应时间或地理位置来路由用户。

![使用流量管理器控制流量流](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

拥有单个 Kubernetes 群集的客户通常连接到给定应用程序的服务 IP 或 DNS 名称。 在多群集部署中，客户应连接到指向每个 Kubernetes 群集上的服务/流入量的流量管理器 DNS 名称。

![使用流量管理器路由到本地群集](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> 这种模式也是 [适用于 Azure 中（托管）AKS 群集的最佳做法](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)。

通过 AKS 引擎部署的 Kubernetes 群集本身应至少包含三个主节点和两个工作器节点。

## <a name="identity-and-security-considerations"></a>标识和安全注意事项

标识和安全是重要的主题。 尤其是在解决方案跨越独立的 Azure Stack Hub 实例这种情况下。 Kubernetes 和 Azure（包括 Azure Stack Hub）都提供不同的基于角色的访问控制 (RBAC) 机制：

- Azure RBAC 控制对 Azure（和 Azure Stack Hub）中的资源的访问，还可以创建新的 Azure 资源。 可将权限分配给用户、组或服务主体。 （服务主体是应用程序使用的安全标识。）
- Kubernetes RBAC 控制 Kubernetes API 的权限。 例如，可以通过 RBAC 授权（或拒绝）用户执行创建 pod 和列出 pod 的操作。 若要将 Kubernetes 权限分配给用户，请创建角色和角色绑定。

**Azure Stack Hub 标识和 RBAC**

Azure Stack Hub 提供两个标识提供者选项。 你使用的提供程序依赖于环境以及是在已联网还是断开网络连接的环境中运行：

- Azure AD - 只能在连接的环境中使用。
- ADFS 到传统 Active Directory 林 - 可用于已联网还是断开网络连接的环境。

标识提供程序管理用户和组，包括用于访问资源的身份验证和授权。 可以授予对 Azure Stack Hub 资源（如订阅、资源组）和单个资源（如 VM 或负载均衡器）的访问权限。 若要获得一致的访问模型，应考虑对所有 Azure Stack Hub 使用相同的组（直接或嵌套）。 下面是一个配置示例：

![Azure Stack Hub 的嵌套 AAD 组](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

该示例包含一个有特定用途的专用组（使用 AAD 或 ADFS）。 例如，在特定的 Azure Stack Hub 实例上为包含 Kubernetes 群集基础结构的资源组提供参与者权限（此处为“Seattle K8s 群集参与者”）。 然后，这些组嵌套到一个整体组中，该组包含每个 Azure Stack Hub 的“子组”。

我们的示例用户现在将具有对包含整个 Kubernetes 基础结构资源集的两个资源组的“参与者”权限。 由于实例共用同一标识提供者，因此用户可以访问这两个 Azure Stack Hub 实例上的资源。

> [!IMPORTANT]
> 这些权限只会影响 Azure Stack Hub 和在其基础上部署的一些资源。 具有这种访问级别的用户可能会造成很多损害，但如果没有对 Kubernetes 部署的更多访问权限，则无法访问 Kubernetes IaaS VM 或 Kubernetes API。

**Kubernetes 标识和 RBAC**

默认情况下，Kubernetes 群集使用的标识提供者与基础 Azure Stack Hub 使用的不同。 托管 Kubernetes 群集、主节点和工作器节点的 VM 使用在群集部署期间指定的 SSH 密钥。 使用 SSH 连接到这些节点需要此 SSH 密钥。

Kubernetes API（例如使用 `kubectl` 进行访问）还受到服务帐户（包括默认的“群集管理员”服务帐户）的保护。 此服务帐户的凭据最初存储在 Kubernetes 主节点上的 `.kube/config` 文件中。

**机密管理和应用程序凭据**

若要存储机密（如连接字符串或数据库凭据），有多种选择，包括：

- Azure Key Vault
- Kubernetes 机密
- 第三方解决方案，如 HashiCorp Vault（在 Kubernetes 上运行）

请勿在配置文件、应用程序代码或脚本中以明文形式存储机密或凭据。 并且不要将其存储在版本控制系统中。 而应该让部署自动化根据需要检索机密。

## <a name="patch-and-update"></a>修补和更新

Azure Kubernetes 服务中的修补程序和更新 (PNU) 过程是部分自动化的。 Kubernetes 版本升级是手动触发的，而安全更新是自动应用的。 这些更新可能包括 OS 安全修复项或内核更新。 AKS 不会自动重启这些 Linux 节点以完成更新进程。 

使用 Azure Stack Hub 上的 AKS 引擎部署的 Kubernetes 群集的 PNU 流程不受管理，并且由群集操作员负责。 

AKS 引擎有助于执行两个最重要的任务：

- [升级到较新的 Kubernetes 和基础 OS 映像版本](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [仅升级基础 OS 映像](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

较新的基础 OS 映像包含最新 OS 安全修补程序和内核更新。 

[无人参与升级](https://wiki.debian.org/UnattendedUpgrades)机制会自动安装在 Azure Stack Hub 市场提供新基础 OS 映像版本之前发布的安全更新。 默认情况下会启用无人参与升级，并自动安装安全更新，但不重启 Kubernetes 群集节点。 可以使用开源 [KUbernetes REboot Daemon (kured)](/azure/aks/node-updates-kured) 自动重启节点  。 Kured 监视需要重启的 Linux 节点，然后自动处理正在运行的 Pod 和节点重启过程的重新安排。

## <a name="deployment-cicd-considerations"></a>部署 (CI/CD) 注意事项

Azure 和 Azure Stack Hub 公开相同的 Azure 资源管理器 REST API。 这些 API 的处理方式与其他任何 Azure 云（Azure、Azure 中国世纪互联、Azure 政府）相同。 不同云之间的 API 版本可能有所不同，Azure Stack Hub 仅提供一部分服务。 每个云以及 Azure Stack Hub 的每个实例的管理终结点 URI 也不同。

除了提到的细微差异外，Azure 资源管理器 REST API 提供了一种一致的方法来与 Azure 和 Azure Stack Hub 交互。 这里可以使用的工具集与任何其他 Azure 云要使用的相同。 可以使用 Azure DevOps、Jenkins 等工具或 PowerShell 将服务部署和编排到 Azure Stack Hub。

**注意事项**

在 Azure Stack Hub 部署方面，一个主要区别是 Internet 可访问性问题。 Internet 可访问性决定是否为 CI/CD 作业选择 Microsoft 托管的生成代理或自托管生成代理。

自托管代理可以在 Azure Stack Hub 上运行（作为 IaaS VM），也可以在可以访问 Azure Stack Hub 的网络子网中运行。 请转到 [Azure Pipelines 代理](/azure/devops/pipelines/agents/agents)，了解相关差异的详细信息。

下图可帮助你决定是需要自托管生成代理还是 Microsoft 托管的生成代理：

![自托管生成代理“是”或“否”](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Azure Stack Hub 管理终结点是否可通过 Internet 访问？
  - 是：我们可以将 Azure Pipelines 与 Microsoft 托管的代理结合使用，以连接到 Azure Stack Hub。
  - 否：我们需要可连接到 Azure Stack Hub 管理终结点的自托管代理。
- 我们的 Kubernetes 群集是否可通过 Internet 访问？
  - 是：我们可以将 Azure Pipelines 与 Microsoft 托管的代理结合使用，以直接与 Kubernetes API 终结点进行交互。
  - 否：我们需要可连接到 Kubernetes 群集 API 终结点的自托管代理。

在可以通过 Internet 访问 Azure Stack Hub 管理终结点和 Kubernetes API 的情况下，部署可以使用 Microsoft 托管的代理。 这一部署将产生如下的应用程序体系结构：

[![公共体系结构概述](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

如果无法通过 Internet 直接访问 Azure 资源管理器终结点、Kubernetes API 或这两者，则可以利用自托管生成代理来运行管道步骤。 此设计需要较少的连接，并且可以仅通过与 Azure 资源管理器终结点和 Kubernetes API 的本地网络连接进行部署：

[![本地体系结构概述](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> 已断开连接的场景如何？ 在 Azure Stack Hub、Kubernetes 或两者都没有面向 Internet 的管理终结点这种情况下，仍然可以使用 Azure DevOps 进行部署。 可以在本地使用自托管的代理池（这是在本地或在 Azure Stack Hub 自身上运行的 DevOps 代理），也可以使用完全自托管的 Azure DevOps Server。 自托管代理只需要出站 HTTPS (TCP/443) Internet 连接。

该模式可以在每个 Azure Stack Hub 实例上使用 Kubernetes 群集（使用 AKS 引擎进行部署和协调）。 其中包括一个由前端、中间层、后端服务（例如 MongoDB）和基于 Nginx 的入口控制器组成的应用程序。 可以使用“外部数据存储”，而不是使用 K8s 群集上托管的数据库。 数据库选项包括 MySQL、SQL Server 或 Azure Stack Hub 之外或 IaaS 中托管的任何类型的数据库。 此类配置不在本文范围内。

## <a name="partner-solutions"></a>合作伙伴解决方案

有一些 Microsoft 合作伙伴解决方案可以拓展 Azure Stack Hub 的功能。 这些解决方案在部署 Kubernetes 群集上运行的应用程序时很有用。  

## <a name="storage-and-data-solutions"></a>存储和数据解决方案

如[数据和存储注意事项](#data-and-storage-considerations)中所述，Azure Stack Hub 当前没有原生解决方案来跨多个实例复制存储。 与 Azure 不同，Azure Stack Hub 并没有跨多个区域复制存储的功能。 在 Azure Stack Hub 中，每个实例都是其自己独有的云。 但是，可以从 Microsoft 合作伙伴处获得解决方案，这些解决方案可以跨 Azure Stack Hub 和 Azure 进行存储复制。 

**SCALITY**

[Scality](https://www.scality.com/) 提供 Web 规模的存储，自 2009 年以来便为数字业务提供支持。 Scality RING 是我们的软件定义存储，可将商用 x86 服务器转变为无限制的存储池，用于存储 PB 级的任何类型的数据（文件和对象）。

**CLOUDIAN**

[Cloudian](https://www.cloudian.com/) 通过无限的可缩放存储简化了企业存储，这种可缩放存储会将大量数据集整合到一种易于管理的环境中。

## <a name="next-steps"></a>后续步骤

若要详细了解本文介绍的概念，请参阅：

- Azure Stack Hub 中的[跨云缩放](pattern-cross-cloud-scale.md)和[地理分布式应用模式](pattern-geo-distributed.md)。
- [Azure Kubernetes 服务 (AKS) 上的微服务体系结构](/azure/architecture/reference-architectures/microservices/aks)。

准备好测试解决方案示例时，请继续学习[高可用性 Kubernetes 群集部署指南](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)。 该部署指南逐步说明了如何部署和测试 Azure Stack 的组件。