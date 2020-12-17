---
title: 在 Azure Stack Hub 上部署高度可用的 Kubernetes 群集
description: 了解如何使用 Azure 和 Azure Stack Hub 部署 Kubernetes 群集解决方案以实现高可用性。
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911918"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>在 Azure Stack Hub 上部署高可用性 Kubernetes 群集

本文介绍如何在不同的物理位置上构建部署在多个 Azure Stack Hub 实例上的高度可用的 Kubernetes 群集环境。

在此解决方案部署指南中，了解如何执行以下操作：

> [!div class="checklist"]
> - 下载并准备 AKS 引擎
> - 连接到 AKS 引擎帮助程序 VM
> - 部署 Kubernetes 群集
> - 连接到 Kubernetes 群集
> - 将 Azure Pipelines 连接到 Kubernetes 群集
> - 配置监视
> - 部署应用程序
> - 自动缩放应用程序
> - 配置流量管理器
> - 升级 Kubernetes
> - 缩放 Kubernetes

> [!Tip]  
> ![混合要素](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的扩展。 Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。  
> 
> [混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。 这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。

## <a name="prerequisites"></a>先决条件

在开始使用此部署指南之前，请确保：

- 查看[高可用性 Kubernetes 群集模式](pattern-highly-available-kubernetes.md)一文。
- 查看[随附 GitHub 存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)的内容，其中包含本文中引用的其他资产。
- 有一个可访问 [Azure Stack Hub 用户门户](/azure-stack/user/azure-stack-use-portal)且至少具有[“参与者”权限](/azure-stack/user/azure-stack-manage-permissions)的帐户。

## <a name="download-and-prepare-aks-engine"></a>下载并准备 AKS 引擎

AKS 引擎是可在能够访问 Azure Stack Hub Azure 资源管理器终结点的任何 Windows 或 Linux 主机中使用的二进制文件。 本指南介绍如何在 Azure Stack Hub 上部署新的 Linux（或 Windows）VM。 稍后在 AKS 引擎部署 Kubernetes 群集时将用到该 VM。

> [!NOTE]
> 你也可以使用现有的 Windows 或 Linux VM，通过 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 群集。

下文记录了 AKS 引擎的分步过程和要求：

* [在 Azure Stack Hub 中于 Linux 上安装 AKS 引擎](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux)（或使用 [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)）

AKS 引擎是帮助程序工具，用于（在 Azure 和 Azure Stack Hub 中）部署和操作（非托管）Kubernetes 群集。

下文介绍了 Azure Stack Hub 上 AKS 引擎的详细信息和区别：

* [Azure Stack Hub 上的 AKS 引擎是什么？](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Azure Stack Hub 上的 AKS 引擎](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md)（GitHub 上）

示例环境将使用 Terraform 自动部署 AKS 引擎 VM。 可以在[随附 GitHub 存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)中找到详细信息和代码。

此步骤的结果是一个 Azure Stack Hub 上的新资源组，其中包含 AKS 引擎帮助程序 VM 和相关资源：

![Azure Stack Hub 中的 AKS 引擎 VM 资源](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> 如果必须在已断开连接的气隙环境中部署 AKS 引擎，请查看[已断开连接的 Azure Stack Hub 实例](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)以了解详细信息。

在下一步中，使用新部署的 AKS引擎 VM 部署 Kubernetes 群集。

## <a name="connect-to-the-aks-engine-helper-vm"></a>连接到 AKS 引擎帮助程序 VM

首先，必须连接到先前创建的 AKS 引擎帮助程序 VM。

该 VM 应具有公共 IP 地址，并且应可通过 SSH（端口 22/TCP）进行访问。

![AKS 引擎 VM 概述页面](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> 可以在 Windows 10 中使用所选工具（如 MobaXterm、puTTY 或 PowerShell）来通过 SSH 连接到 Linux VM。

```console
ssh <username>@<ipaddress>
```

连接后，运行 `aks-engine` 命令。 转到[支持的 AKS 引擎版本](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)，详细了解 AKS 引擎和 Kubernetes 版本。

![aks-engine 命令行示例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>部署 Kubernetes 群集

AKS 引擎帮助程序 VM 本身尚未在 Azure Stack Hub 上创建 Kubernetes 群集。 创建群集是在 AKS 引擎帮助程序 VM 中第一个要执行的操作。

下文记录了分步过程：

* [使用 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 群集](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

`aks-engine deploy` 命令和之前步骤的准备工作的最终结果是一个功能齐全的 Kubernetes 群集，该群集部署到第一个 Azure Stack Hub 实例的租户空间中。 群集本身包含 Azure IaaS 组件，例如 VM、负载均衡器、VNet 和磁盘等。

![Azure Stack Hub 门户的群集 IaaS 组件](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure 负载均衡器（K8s API 终结点）
2) 工作器节点（代理池）
3) 主节点

群集现在已启动且正在运行，下一步我们将连接到该群集。

## <a name="connect-to-the-kubernetes-cluster"></a>连接到 Kubernetes 群集

现在，可以通过 SSH（使用在部署过程中指定的 SSH 密钥）或通过 `kubectl`（建议）连接到先前创建的 Kubernetes 群集。 [此处](https://kubernetes.io/docs/tasks/tools/install-kubectl/)的 Kubernetes 命令行工具 `kubectl` 适用于 Windows、Linux 和 macOS。 该工具已预安装在群集的主节点上且已完成配置。

```console
ssh azureuser@<k8s-master-lb-ip>
```

![在主节点上执行 kubectl](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

建议不要将主节点用作管理任务的跳转盒。 `kubectl` 配置存储在主节点的 `.kube/config` 中以及 AKS 引擎 VM 上。 可以将配置复制到与 Kubernetes 群集连接的管理员计算机，然后在该处使用 `kubectl` 命令。 稍后还将使用 `.kube/config` 文件在 Azure Pipelines 中配置服务连接。

> [!IMPORTANT]
> 确保这些文件安全，因为它们包含 Kubernetes 群集的凭据。 有权访问该文件的攻击者拥有足够的信息来获得对该文件的管理员访问权限。 使用初始 `.kube/config` 文件完成的所有操作均使用群集管理员帐户完成。

现在，可以使用 `kubectl` 尝试执行各种命令来检查群集的状态。

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes 具有自己的基于角色的访问控制 (RBAC) 模型，该模型使你可以创建细粒度的角色定义和角色绑定。 这是控制对群集的访问的首选方法（而不是分配群集管理员权限）。

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>将 Azure Pipelines 连接到 Kubernetes 群集

若要将 Azure Pipelines 连接到新部署的 Kubernetes 群集，需要该群集的 kube 配置 (`.kube/config`) 文件，如上一步中所述。

* 连接到 Kubernetes 群集的主节点之一。
* 复制 `.kube/config` 文件的内容。
* 转到“Azure DevOps”>“项目设置”>“服务连接”，以创建新的“Kubernetes”服务连接（使用 KubeConfig 作为身份验证方法）

> [!IMPORTANT]
> Azure Pipelines（或其生成代理）必须有权访问 Kubernetes API。 如果从 Azure Pipelines 到 Azure Stack Hub Kubernetes 群集之间存在 Internet 连接，则需要部署自托管的 Azure Pipelines 生成代理。

在为 Azure Pipelines 部署自托管代理时，可以在 Azure Stack Hub 上进行部署，也可以在与所有必需的管理终结点具有网络连接的机器上进行部署。 有关详细信息，请参阅下文：

* [Windows](/azure/devops/pipelines/agents/v2-windows) 或 [Linux](/azure/devops/pipelines/agents/v2-linux) 上的 [Azure Pipelines 代理](/azure/devops/pipelines/agents/agents)

模式[部署 (CI/CD) 注意事项](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)部分包含决策流，可帮助你了解是使用 Microsoft 托管代理还是自托管代理：

[![决策流自托管代理](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

在此示例解决方案中，拓扑对于每个 Azure Stack Hub 实例都包含一个自托管生成代理。 此代理可以访问 Azure Stack Hub 管理终结点和 Kubernetes 群集 API 终结点。

[![仅限出站流量](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

此设计满足常规法规要求，即仅具有来自应用程序解决方案的出站连接。

## <a name="configure-monitoring"></a>配置监视

可以对容器使用 [Azure Monitor](/azure/azure-monitor/) 以监视解决方案中的容器。 这会将 Azure Monitor 指向 Azure Stack Hub 上部署了 AKS 引擎的 Kubernetes 群集。

可通过两种方式在群集上启用 Azure Monitor。 这两种方式均要求你在 Azure 中设置 Log Analytics 工作区。

* [方法一](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one)使用 Helm Chart
* [方法二](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)作为 AKS 引擎群集规范的一部分

示例拓扑中使用了“方法一”，通过该方法，可实现过程自动化，且更新可更轻松地进行安装。

下一步，需要在计算机上安装 Azure LogAnalytics 工作区（ID 和密钥）、`Helm`（版本 3）和 `kubectl`。

Helm 是 Kubernetes 包管理器，可用作在 macOS、Windows 和 Linux 上运行的二进制文件。 可在此处下载 Helm：[helm.sh](https://helm.sh/docs/intro/quickstart/)。Helm 依赖于 `kubectl` 命令使用的 Kubernetes 配置文件。

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

此命令将在 Kubernetes 群集上安装 Azure Monitor 代理：

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

Kubernetes 群集上的 Operations Management Suite (OMS) 代理会（使用出站 HTTPS）将监视数据发送到 Azure Log Analytics 工作区。 现在，可以使用 Azure Monitor 在 Azure Stack Hub 上获得有关 Kubernetes 群集的更深入见解。 此设计是一种功能强大的方法，用于演示可通过应用程序的群集自动部署的分析功能。

[![Azure monitor 中的 Azure Stack Hub 群集](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor 群集详细信息](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> 如果 Azure Monitor 不显示任何 Azure Stack Hub 数据，请确保你已仔细按照[如何将 AzureMonitor-Containers 解决方案添加到 Azure Loganalytics 工作区](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)中的说明操作。

## <a name="deploy-the-application"></a>部署应用程序

在安装示例应用程序之前，还需执行一个步骤 - 在 Kubernetes 群集上配置基于 nginx 的入口控制器。 入口控制器用作第 7 层负载均衡器，可基于主机、路径或协议在群集中路由流量。 Nginx-ingress 可以作为 Helm Chart 使用。 有关详细说明，请参阅 [Helm Chart GitHub 存储库](https://github.com/helm/charts/tree/master/stable/nginx-ingress)。

示例应用程序也将打包为 Helm Chart，与上一步中的 [Azure Monitoring 代理](#configure-monitoring)一样。 同样，将应用程序部署到 Kubernetes 群集上也很简单。 可以[在随附 GitHub 存储库中找到 Helm Chart 文件](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

示例应用程序是一个三层应用程序，部署在两个 Azure Stack Hub 实例中每个实例的 Kubernetes 群集上。 该应用程序使用 MongoDB 数据库。 可以在模式[数据和存储注意事项](pattern-highly-available-kubernetes.md#data-and-storage-considerations)中详细了解如何跨多个实例复制数据。

在为应用程序部署 Helm Chart 后，可看到应用程序的三个层均用一个 Pod 表示为部署和 StatefulSet（针对数据库）：

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

在服务端，可以找到基于 nginx 的入口控制器及其公共 IP 地址：

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

“外部 IP”地址是“应用程序终结点”。 用户通过该地址进行连接以打开应用程序，并且该地址也将用作下一步[配置流量管理器](#configure-traffic-manager)的终结点。

## <a name="autoscale-the-application"></a>自动缩放应用程序
可以选择配置[水平 Pod 自动缩放程序](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)，以根据某些指标（例如 CPU 利用率）进行纵向扩展或纵向缩减。 以下命令将创建水平 Pod 自动缩放程序，用于维护由 ratings-web 部署控制的 Pod 的 1 至 10 个副本。 HPA 将（通过部署）增加和减少副本数量，以在所有 Pod 上维持 80% 的平均 CPU 利用率。

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
可以通过运行以下命令来检查自动缩放程序的当前状态：

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>配置流量管理器

若要在两个（或多个）应用程序部署之间分配流量，我们将使用 [Azure 流量管理器](/azure/traffic-manager/traffic-manager-overview)。 Azure 流量管理器是 Azure 中基于 DNS 的流量负载均衡器。

> [!NOTE]
> 流量管理器根据流量路由方法和终结点的运行状况，使用 DNS 将客户端请求定向到最合适的服务终结点。

也可以使用在本地托管的其他全局负载均衡解决方案，而不用 Azure 流量管理器。 在示例场景中，使用 Azure 流量管理器在应用程序的两个实例之间分配流量。 它们可以在相同或不同位置的 Azure Stack Hub 实例上运行：

![本地流量管理器](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

在 Azure 中，将流量管理器配置为指向应用程序的两个不同实例：

[![TM 终结点配置文件](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

如你所见，这两个终结点指向[上一部分](#deploy-the-application)中已部署应用程序的两个实例。

此时：
- 已创建 Kubernetes 基础结构，包括入口控制器。
- 已跨两个 Azure Stack Hub 实例部署群集。
- 已配置监视。
- Azure 流量管理器将跨两个 Azure Stack Hub 实例对流量进行负载均衡。
- 在此基础结构上，使用 Helm Chart 以自动化方式部署了示例三层应用程序。 

现在，该解决方案应该已启动并可供用户使用！

接下来的两个部分中会介绍一些需要讨论的部署后操作注意事项。

## <a name="upgrade-kubernetes"></a>升级 Kubernetes

升级 Kubernetes 群集时，请考虑以下主题：

- 升级 Kubernetes 群集是复杂的 Day 2 操作，可以使用 AKS 引擎完成此操作。 有关详细信息，请参阅[升级 Azure Stack Hub 上的 Kubernetes 群集](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)。
- 通过 AKS 引擎，可将群集升级到较新的 Kubernetes 和基础 OS 映像版本。 有关详细信息，请参阅[用于升级到 Kubernetes 较新版本的步骤](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)。 
- 也可以只将基础节点升级到较新的基础 OS 映像版本。 有关详细信息，请参阅[用于仅升级 OS 映像的步骤](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)。

较新的基础 OS 映像包含安全性和内核更新。 群集操作员负责监视较新的 Kubernetes 版本和 OS 映像的可用性。 操作员应使用 AKS 引擎来计划并执行这些升级。 基础 OS 映像必须由 Azure Stack Hub 操作员从 Azure Stack Hub 市场下载。

## <a name="scale-kubernetes"></a>缩放 Kubernetes

缩放是另一项 Day 2 操作，可以使用 AKS 引擎进行协调。

缩放命令重用输出目录中的群集配置文件 (apimodel.json)，作为新 Azure 资源管理器部署的输入。 AKS 引擎针对特定的代理池执行缩放操作。 缩放操作完成后，AKS 引擎会在同一 apimodel.json 文件中更新群集定义。 群集定义反映了新的节点计数，以反映更新后的当前群集配置。

- [缩放 Azure Stack Hub 上的 Kubernetes 群集](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>后续步骤

- 详细了解[混合应用设计注意事项](overview-app-design-considerations.md)
- 查看 [GitHub 上此示例的代码](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)并提出改进建议。