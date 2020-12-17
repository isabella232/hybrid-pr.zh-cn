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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="11aef-103">在 Azure Stack Hub 上部署高可用性 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="11aef-104">本文介绍如何在不同的物理位置上构建部署在多个 Azure Stack Hub 实例上的高度可用的 Kubernetes 群集环境。</span><span class="sxs-lookup"><span data-stu-id="11aef-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="11aef-105">在此解决方案部署指南中，了解如何执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="11aef-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="11aef-106">下载并准备 AKS 引擎</span><span class="sxs-lookup"><span data-stu-id="11aef-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="11aef-107">连接到 AKS 引擎帮助程序 VM</span><span class="sxs-lookup"><span data-stu-id="11aef-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="11aef-108">部署 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="11aef-109">连接到 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="11aef-110">将 Azure Pipelines 连接到 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="11aef-111">配置监视</span><span class="sxs-lookup"><span data-stu-id="11aef-111">Configure monitoring</span></span>
> - <span data-ttu-id="11aef-112">部署应用程序</span><span class="sxs-lookup"><span data-stu-id="11aef-112">Deploy application</span></span>
> - <span data-ttu-id="11aef-113">自动缩放应用程序</span><span class="sxs-lookup"><span data-stu-id="11aef-113">Autoscale application</span></span>
> - <span data-ttu-id="11aef-114">配置流量管理器</span><span class="sxs-lookup"><span data-stu-id="11aef-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="11aef-115">升级 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="11aef-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="11aef-116">缩放 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="11aef-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="11aef-117">![混合要素](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="11aef-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="11aef-118">Microsoft Azure Stack Hub 是 Azure 的扩展。</span><span class="sxs-lookup"><span data-stu-id="11aef-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="11aef-119">Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。</span><span class="sxs-lookup"><span data-stu-id="11aef-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="11aef-120">[混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。</span><span class="sxs-lookup"><span data-stu-id="11aef-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="11aef-121">这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。</span><span class="sxs-lookup"><span data-stu-id="11aef-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="11aef-122">先决条件</span><span class="sxs-lookup"><span data-stu-id="11aef-122">Prerequisites</span></span>

<span data-ttu-id="11aef-123">在开始使用此部署指南之前，请确保：</span><span class="sxs-lookup"><span data-stu-id="11aef-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="11aef-124">查看[高可用性 Kubernetes 群集模式](pattern-highly-available-kubernetes.md)一文。</span><span class="sxs-lookup"><span data-stu-id="11aef-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="11aef-125">查看[随附 GitHub 存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)的内容，其中包含本文中引用的其他资产。</span><span class="sxs-lookup"><span data-stu-id="11aef-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="11aef-126">有一个可访问 [Azure Stack Hub 用户门户](/azure-stack/user/azure-stack-use-portal)且至少具有[“参与者”权限](/azure-stack/user/azure-stack-manage-permissions)的帐户。</span><span class="sxs-lookup"><span data-stu-id="11aef-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="11aef-127">下载并准备 AKS 引擎</span><span class="sxs-lookup"><span data-stu-id="11aef-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="11aef-128">AKS 引擎是可在能够访问 Azure Stack Hub Azure 资源管理器终结点的任何 Windows 或 Linux 主机中使用的二进制文件。</span><span class="sxs-lookup"><span data-stu-id="11aef-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="11aef-129">本指南介绍如何在 Azure Stack Hub 上部署新的 Linux（或 Windows）VM。</span><span class="sxs-lookup"><span data-stu-id="11aef-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="11aef-130">稍后在 AKS 引擎部署 Kubernetes 群集时将用到该 VM。</span><span class="sxs-lookup"><span data-stu-id="11aef-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="11aef-131">你也可以使用现有的 Windows 或 Linux VM，通过 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="11aef-132">下文记录了 AKS 引擎的分步过程和要求：</span><span class="sxs-lookup"><span data-stu-id="11aef-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="11aef-133">[在 Azure Stack Hub 中于 Linux 上安装 AKS 引擎](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux)（或使用 [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)）</span><span class="sxs-lookup"><span data-stu-id="11aef-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="11aef-134">AKS 引擎是帮助程序工具，用于（在 Azure 和 Azure Stack Hub 中）部署和操作（非托管）Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="11aef-135">下文介绍了 Azure Stack Hub 上 AKS 引擎的详细信息和区别：</span><span class="sxs-lookup"><span data-stu-id="11aef-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="11aef-136">Azure Stack Hub 上的 AKS 引擎是什么？</span><span class="sxs-lookup"><span data-stu-id="11aef-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="11aef-137">[Azure Stack Hub 上的 AKS 引擎](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md)（GitHub 上）</span><span class="sxs-lookup"><span data-stu-id="11aef-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="11aef-138">示例环境将使用 Terraform 自动部署 AKS 引擎 VM。</span><span class="sxs-lookup"><span data-stu-id="11aef-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="11aef-139">可以在[随附 GitHub 存储库](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)中找到详细信息和代码。</span><span class="sxs-lookup"><span data-stu-id="11aef-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="11aef-140">此步骤的结果是一个 Azure Stack Hub 上的新资源组，其中包含 AKS 引擎帮助程序 VM 和相关资源：</span><span class="sxs-lookup"><span data-stu-id="11aef-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Azure Stack Hub 中的 AKS 引擎 VM 资源](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="11aef-142">如果必须在已断开连接的气隙环境中部署 AKS 引擎，请查看[已断开连接的 Azure Stack Hub 实例](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances)以了解详细信息。</span><span class="sxs-lookup"><span data-stu-id="11aef-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="11aef-143">在下一步中，使用新部署的 AKS引擎 VM 部署 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="11aef-144">连接到 AKS 引擎帮助程序 VM</span><span class="sxs-lookup"><span data-stu-id="11aef-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="11aef-145">首先，必须连接到先前创建的 AKS 引擎帮助程序 VM。</span><span class="sxs-lookup"><span data-stu-id="11aef-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="11aef-146">该 VM 应具有公共 IP 地址，并且应可通过 SSH（端口 22/TCP）进行访问。</span><span class="sxs-lookup"><span data-stu-id="11aef-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![AKS 引擎 VM 概述页面](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="11aef-148">可以在 Windows 10 中使用所选工具（如 MobaXterm、puTTY 或 PowerShell）来通过 SSH 连接到 Linux VM。</span><span class="sxs-lookup"><span data-stu-id="11aef-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="11aef-149">连接后，运行 `aks-engine` 命令。</span><span class="sxs-lookup"><span data-stu-id="11aef-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="11aef-150">转到[支持的 AKS 引擎版本](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions)，详细了解 AKS 引擎和 Kubernetes 版本。</span><span class="sxs-lookup"><span data-stu-id="11aef-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![aks-engine 命令行示例](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="11aef-152">部署 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="11aef-153">AKS 引擎帮助程序 VM 本身尚未在 Azure Stack Hub 上创建 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="11aef-154">创建群集是在 AKS 引擎帮助程序 VM 中第一个要执行的操作。</span><span class="sxs-lookup"><span data-stu-id="11aef-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="11aef-155">下文记录了分步过程：</span><span class="sxs-lookup"><span data-stu-id="11aef-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="11aef-156">使用 AKS 引擎在 Azure Stack Hub 上部署 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="11aef-157">`aks-engine deploy` 命令和之前步骤的准备工作的最终结果是一个功能齐全的 Kubernetes 群集，该群集部署到第一个 Azure Stack Hub 实例的租户空间中。</span><span class="sxs-lookup"><span data-stu-id="11aef-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="11aef-158">群集本身包含 Azure IaaS 组件，例如 VM、负载均衡器、VNet 和磁盘等。</span><span class="sxs-lookup"><span data-stu-id="11aef-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Azure Stack Hub 门户的群集 IaaS 组件](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="11aef-160">Azure 负载均衡器（K8s API 终结点）</span><span class="sxs-lookup"><span data-stu-id="11aef-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="11aef-161">工作器节点（代理池）</span><span class="sxs-lookup"><span data-stu-id="11aef-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="11aef-162">主节点</span><span class="sxs-lookup"><span data-stu-id="11aef-162">Master Nodes</span></span>

<span data-ttu-id="11aef-163">群集现在已启动且正在运行，下一步我们将连接到该群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="11aef-164">连接到 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="11aef-165">现在，可以通过 SSH（使用在部署过程中指定的 SSH 密钥）或通过 `kubectl`（建议）连接到先前创建的 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="11aef-166">[此处](https://kubernetes.io/docs/tasks/tools/install-kubectl/)的 Kubernetes 命令行工具 `kubectl` 适用于 Windows、Linux 和 macOS。</span><span class="sxs-lookup"><span data-stu-id="11aef-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="11aef-167">该工具已预安装在群集的主节点上且已完成配置。</span><span class="sxs-lookup"><span data-stu-id="11aef-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![在主节点上执行 kubectl](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="11aef-169">建议不要将主节点用作管理任务的跳转盒。</span><span class="sxs-lookup"><span data-stu-id="11aef-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="11aef-170">`kubectl` 配置存储在主节点的 `.kube/config` 中以及 AKS 引擎 VM 上。</span><span class="sxs-lookup"><span data-stu-id="11aef-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="11aef-171">可以将配置复制到与 Kubernetes 群集连接的管理员计算机，然后在该处使用 `kubectl` 命令。</span><span class="sxs-lookup"><span data-stu-id="11aef-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="11aef-172">稍后还将使用 `.kube/config` 文件在 Azure Pipelines 中配置服务连接。</span><span class="sxs-lookup"><span data-stu-id="11aef-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="11aef-173">确保这些文件安全，因为它们包含 Kubernetes 群集的凭据。</span><span class="sxs-lookup"><span data-stu-id="11aef-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="11aef-174">有权访问该文件的攻击者拥有足够的信息来获得对该文件的管理员访问权限。</span><span class="sxs-lookup"><span data-stu-id="11aef-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="11aef-175">使用初始 `.kube/config` 文件完成的所有操作均使用群集管理员帐户完成。</span><span class="sxs-lookup"><span data-stu-id="11aef-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="11aef-176">现在，可以使用 `kubectl` 尝试执行各种命令来检查群集的状态。</span><span class="sxs-lookup"><span data-stu-id="11aef-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="11aef-177">Kubernetes 具有自己的基于角色的访问控制 (RBAC) 模型，该模型使你可以创建细粒度的角色定义和角色绑定。</span><span class="sxs-lookup"><span data-stu-id="11aef-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="11aef-178">这是控制对群集的访问的首选方法（而不是分配群集管理员权限）。</span><span class="sxs-lookup"><span data-stu-id="11aef-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="11aef-179">将 Azure Pipelines 连接到 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="11aef-180">若要将 Azure Pipelines 连接到新部署的 Kubernetes 群集，需要该群集的 kube 配置 (`.kube/config`) 文件，如上一步中所述。</span><span class="sxs-lookup"><span data-stu-id="11aef-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="11aef-181">连接到 Kubernetes 群集的主节点之一。</span><span class="sxs-lookup"><span data-stu-id="11aef-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="11aef-182">复制 `.kube/config` 文件的内容。</span><span class="sxs-lookup"><span data-stu-id="11aef-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="11aef-183">转到“Azure DevOps”>“项目设置”>“服务连接”，以创建新的“Kubernetes”服务连接（使用 KubeConfig 作为身份验证方法）</span><span class="sxs-lookup"><span data-stu-id="11aef-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="11aef-184">Azure Pipelines（或其生成代理）必须有权访问 Kubernetes API。</span><span class="sxs-lookup"><span data-stu-id="11aef-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="11aef-185">如果从 Azure Pipelines 到 Azure Stack Hub Kubernetes 群集之间存在 Internet 连接，则需要部署自托管的 Azure Pipelines 生成代理。</span><span class="sxs-lookup"><span data-stu-id="11aef-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="11aef-186">在为 Azure Pipelines 部署自托管代理时，可以在 Azure Stack Hub 上进行部署，也可以在与所有必需的管理终结点具有网络连接的机器上进行部署。</span><span class="sxs-lookup"><span data-stu-id="11aef-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="11aef-187">有关详细信息，请参阅下文：</span><span class="sxs-lookup"><span data-stu-id="11aef-187">See the details here:</span></span>

* <span data-ttu-id="11aef-188">[Windows](/azure/devops/pipelines/agents/v2-windows) 或 [Linux](/azure/devops/pipelines/agents/v2-linux) 上的 [Azure Pipelines 代理](/azure/devops/pipelines/agents/agents)</span><span class="sxs-lookup"><span data-stu-id="11aef-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="11aef-189">模式[部署 (CI/CD) 注意事项](pattern-highly-available-kubernetes.md#deployment-cicd-considerations)部分包含决策流，可帮助你了解是使用 Microsoft 托管代理还是自托管代理：</span><span class="sxs-lookup"><span data-stu-id="11aef-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="11aef-190">[![决策流自托管代理](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="11aef-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="11aef-191">在此示例解决方案中，拓扑对于每个 Azure Stack Hub 实例都包含一个自托管生成代理。</span><span class="sxs-lookup"><span data-stu-id="11aef-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="11aef-192">此代理可以访问 Azure Stack Hub 管理终结点和 Kubernetes 群集 API 终结点。</span><span class="sxs-lookup"><span data-stu-id="11aef-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="11aef-193">[![仅限出站流量](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="11aef-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="11aef-194">此设计满足常规法规要求，即仅具有来自应用程序解决方案的出站连接。</span><span class="sxs-lookup"><span data-stu-id="11aef-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="11aef-195">配置监视</span><span class="sxs-lookup"><span data-stu-id="11aef-195">Configure monitoring</span></span>

<span data-ttu-id="11aef-196">可以对容器使用 [Azure Monitor](/azure/azure-monitor/) 以监视解决方案中的容器。</span><span class="sxs-lookup"><span data-stu-id="11aef-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="11aef-197">这会将 Azure Monitor 指向 Azure Stack Hub 上部署了 AKS 引擎的 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="11aef-198">可通过两种方式在群集上启用 Azure Monitor。</span><span class="sxs-lookup"><span data-stu-id="11aef-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="11aef-199">这两种方式均要求你在 Azure 中设置 Log Analytics 工作区。</span><span class="sxs-lookup"><span data-stu-id="11aef-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="11aef-200">[方法一](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one)使用 Helm Chart</span><span class="sxs-lookup"><span data-stu-id="11aef-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="11aef-201">[方法二](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two)作为 AKS 引擎群集规范的一部分</span><span class="sxs-lookup"><span data-stu-id="11aef-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="11aef-202">示例拓扑中使用了“方法一”，通过该方法，可实现过程自动化，且更新可更轻松地进行安装。</span><span class="sxs-lookup"><span data-stu-id="11aef-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="11aef-203">下一步，需要在计算机上安装 Azure LogAnalytics 工作区（ID 和密钥）、`Helm`（版本 3）和 `kubectl`。</span><span class="sxs-lookup"><span data-stu-id="11aef-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="11aef-204">Helm 是 Kubernetes 包管理器，可用作在 macOS、Windows 和 Linux 上运行的二进制文件。</span><span class="sxs-lookup"><span data-stu-id="11aef-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="11aef-205">可在此处下载 Helm：[helm.sh](https://helm.sh/docs/intro/quickstart/)。Helm 依赖于 `kubectl` 命令使用的 Kubernetes 配置文件。</span><span class="sxs-lookup"><span data-stu-id="11aef-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="11aef-206">此命令将在 Kubernetes 群集上安装 Azure Monitor 代理：</span><span class="sxs-lookup"><span data-stu-id="11aef-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="11aef-207">Kubernetes 群集上的 Operations Management Suite (OMS) 代理会（使用出站 HTTPS）将监视数据发送到 Azure Log Analytics 工作区。</span><span class="sxs-lookup"><span data-stu-id="11aef-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="11aef-208">现在，可以使用 Azure Monitor 在 Azure Stack Hub 上获得有关 Kubernetes 群集的更深入见解。</span><span class="sxs-lookup"><span data-stu-id="11aef-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="11aef-209">此设计是一种功能强大的方法，用于演示可通过应用程序的群集自动部署的分析功能。</span><span class="sxs-lookup"><span data-stu-id="11aef-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="11aef-210">[![Azure monitor 中的 Azure Stack Hub 群集](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="11aef-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="11aef-211">[![Azure Monitor 群集详细信息](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="11aef-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="11aef-212">如果 Azure Monitor 不显示任何 Azure Stack Hub 数据，请确保你已仔细按照[如何将 AzureMonitor-Containers 解决方案添加到 Azure Loganalytics 工作区](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md)中的说明操作。</span><span class="sxs-lookup"><span data-stu-id="11aef-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="11aef-213">部署应用程序</span><span class="sxs-lookup"><span data-stu-id="11aef-213">Deploy the application</span></span>

<span data-ttu-id="11aef-214">在安装示例应用程序之前，还需执行一个步骤 - 在 Kubernetes 群集上配置基于 nginx 的入口控制器。</span><span class="sxs-lookup"><span data-stu-id="11aef-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="11aef-215">入口控制器用作第 7 层负载均衡器，可基于主机、路径或协议在群集中路由流量。</span><span class="sxs-lookup"><span data-stu-id="11aef-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="11aef-216">Nginx-ingress 可以作为 Helm Chart 使用。</span><span class="sxs-lookup"><span data-stu-id="11aef-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="11aef-217">有关详细说明，请参阅 [Helm Chart GitHub 存储库](https://github.com/helm/charts/tree/master/stable/nginx-ingress)。</span><span class="sxs-lookup"><span data-stu-id="11aef-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="11aef-218">示例应用程序也将打包为 Helm Chart，与上一步中的 [Azure Monitoring 代理](#configure-monitoring)一样。</span><span class="sxs-lookup"><span data-stu-id="11aef-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="11aef-219">同样，将应用程序部署到 Kubernetes 群集上也很简单。</span><span class="sxs-lookup"><span data-stu-id="11aef-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="11aef-220">可以[在随附 GitHub 存储库中找到 Helm Chart 文件](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="11aef-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="11aef-221">示例应用程序是一个三层应用程序，部署在两个 Azure Stack Hub 实例中每个实例的 Kubernetes 群集上。</span><span class="sxs-lookup"><span data-stu-id="11aef-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="11aef-222">该应用程序使用 MongoDB 数据库。</span><span class="sxs-lookup"><span data-stu-id="11aef-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="11aef-223">可以在模式[数据和存储注意事项](pattern-highly-available-kubernetes.md#data-and-storage-considerations)中详细了解如何跨多个实例复制数据。</span><span class="sxs-lookup"><span data-stu-id="11aef-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="11aef-224">在为应用程序部署 Helm Chart 后，可看到应用程序的三个层均用一个 Pod 表示为部署和 StatefulSet（针对数据库）：</span><span class="sxs-lookup"><span data-stu-id="11aef-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="11aef-225">在服务端，可以找到基于 nginx 的入口控制器及其公共 IP 地址：</span><span class="sxs-lookup"><span data-stu-id="11aef-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="11aef-226">“外部 IP”地址是“应用程序终结点”。</span><span class="sxs-lookup"><span data-stu-id="11aef-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="11aef-227">用户通过该地址进行连接以打开应用程序，并且该地址也将用作下一步[配置流量管理器](#configure-traffic-manager)的终结点。</span><span class="sxs-lookup"><span data-stu-id="11aef-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="11aef-228">自动缩放应用程序</span><span class="sxs-lookup"><span data-stu-id="11aef-228">Autoscale the application</span></span>
<span data-ttu-id="11aef-229">可以选择配置[水平 Pod 自动缩放程序](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)，以根据某些指标（例如 CPU 利用率）进行纵向扩展或纵向缩减。</span><span class="sxs-lookup"><span data-stu-id="11aef-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="11aef-230">以下命令将创建水平 Pod 自动缩放程序，用于维护由 ratings-web 部署控制的 Pod 的 1 至 10 个副本。</span><span class="sxs-lookup"><span data-stu-id="11aef-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="11aef-231">HPA 将（通过部署）增加和减少副本数量，以在所有 Pod 上维持 80% 的平均 CPU 利用率。</span><span class="sxs-lookup"><span data-stu-id="11aef-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="11aef-232">可以通过运行以下命令来检查自动缩放程序的当前状态：</span><span class="sxs-lookup"><span data-stu-id="11aef-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="11aef-233">配置流量管理器</span><span class="sxs-lookup"><span data-stu-id="11aef-233">Configure Traffic Manager</span></span>

<span data-ttu-id="11aef-234">若要在两个（或多个）应用程序部署之间分配流量，我们将使用 [Azure 流量管理器](/azure/traffic-manager/traffic-manager-overview)。</span><span class="sxs-lookup"><span data-stu-id="11aef-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="11aef-235">Azure 流量管理器是 Azure 中基于 DNS 的流量负载均衡器。</span><span class="sxs-lookup"><span data-stu-id="11aef-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="11aef-236">流量管理器根据流量路由方法和终结点的运行状况，使用 DNS 将客户端请求定向到最合适的服务终结点。</span><span class="sxs-lookup"><span data-stu-id="11aef-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="11aef-237">也可以使用在本地托管的其他全局负载均衡解决方案，而不用 Azure 流量管理器。</span><span class="sxs-lookup"><span data-stu-id="11aef-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="11aef-238">在示例场景中，使用 Azure 流量管理器在应用程序的两个实例之间分配流量。</span><span class="sxs-lookup"><span data-stu-id="11aef-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="11aef-239">它们可以在相同或不同位置的 Azure Stack Hub 实例上运行：</span><span class="sxs-lookup"><span data-stu-id="11aef-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![本地流量管理器](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="11aef-241">在 Azure 中，将流量管理器配置为指向应用程序的两个不同实例：</span><span class="sxs-lookup"><span data-stu-id="11aef-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="11aef-242">[![TM 终结点配置文件](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="11aef-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="11aef-243">如你所见，这两个终结点指向[上一部分](#deploy-the-application)中已部署应用程序的两个实例。</span><span class="sxs-lookup"><span data-stu-id="11aef-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="11aef-244">此时：</span><span class="sxs-lookup"><span data-stu-id="11aef-244">At this point:</span></span>
- <span data-ttu-id="11aef-245">已创建 Kubernetes 基础结构，包括入口控制器。</span><span class="sxs-lookup"><span data-stu-id="11aef-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="11aef-246">已跨两个 Azure Stack Hub 实例部署群集。</span><span class="sxs-lookup"><span data-stu-id="11aef-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="11aef-247">已配置监视。</span><span class="sxs-lookup"><span data-stu-id="11aef-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="11aef-248">Azure 流量管理器将跨两个 Azure Stack Hub 实例对流量进行负载均衡。</span><span class="sxs-lookup"><span data-stu-id="11aef-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="11aef-249">在此基础结构上，使用 Helm Chart 以自动化方式部署了示例三层应用程序。</span><span class="sxs-lookup"><span data-stu-id="11aef-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="11aef-250">现在，该解决方案应该已启动并可供用户使用！</span><span class="sxs-lookup"><span data-stu-id="11aef-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="11aef-251">接下来的两个部分中会介绍一些需要讨论的部署后操作注意事项。</span><span class="sxs-lookup"><span data-stu-id="11aef-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="11aef-252">升级 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="11aef-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="11aef-253">升级 Kubernetes 群集时，请考虑以下主题：</span><span class="sxs-lookup"><span data-stu-id="11aef-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="11aef-254">升级 Kubernetes 群集是复杂的 Day 2 操作，可以使用 AKS 引擎完成此操作。</span><span class="sxs-lookup"><span data-stu-id="11aef-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="11aef-255">有关详细信息，请参阅[升级 Azure Stack Hub 上的 Kubernetes 群集](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade)。</span><span class="sxs-lookup"><span data-stu-id="11aef-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="11aef-256">通过 AKS 引擎，可将群集升级到较新的 Kubernetes 和基础 OS 映像版本。</span><span class="sxs-lookup"><span data-stu-id="11aef-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="11aef-257">有关详细信息，请参阅[用于升级到 Kubernetes 较新版本的步骤](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)。</span><span class="sxs-lookup"><span data-stu-id="11aef-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="11aef-258">也可以只将基础节点升级到较新的基础 OS 映像版本。</span><span class="sxs-lookup"><span data-stu-id="11aef-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="11aef-259">有关详细信息，请参阅[用于仅升级 OS 映像的步骤](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)。</span><span class="sxs-lookup"><span data-stu-id="11aef-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="11aef-260">较新的基础 OS 映像包含安全性和内核更新。</span><span class="sxs-lookup"><span data-stu-id="11aef-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="11aef-261">群集操作员负责监视较新的 Kubernetes 版本和 OS 映像的可用性。</span><span class="sxs-lookup"><span data-stu-id="11aef-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="11aef-262">操作员应使用 AKS 引擎来计划并执行这些升级。</span><span class="sxs-lookup"><span data-stu-id="11aef-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="11aef-263">基础 OS 映像必须由 Azure Stack Hub 操作员从 Azure Stack Hub 市场下载。</span><span class="sxs-lookup"><span data-stu-id="11aef-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="11aef-264">缩放 Kubernetes</span><span class="sxs-lookup"><span data-stu-id="11aef-264">Scale Kubernetes</span></span>

<span data-ttu-id="11aef-265">缩放是另一项 Day 2 操作，可以使用 AKS 引擎进行协调。</span><span class="sxs-lookup"><span data-stu-id="11aef-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="11aef-266">缩放命令重用输出目录中的群集配置文件 (apimodel.json)，作为新 Azure 资源管理器部署的输入。</span><span class="sxs-lookup"><span data-stu-id="11aef-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="11aef-267">AKS 引擎针对特定的代理池执行缩放操作。</span><span class="sxs-lookup"><span data-stu-id="11aef-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="11aef-268">缩放操作完成后，AKS 引擎会在同一 apimodel.json 文件中更新群集定义。</span><span class="sxs-lookup"><span data-stu-id="11aef-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="11aef-269">群集定义反映了新的节点计数，以反映更新后的当前群集配置。</span><span class="sxs-lookup"><span data-stu-id="11aef-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="11aef-270">缩放 Azure Stack Hub 上的 Kubernetes 群集</span><span class="sxs-lookup"><span data-stu-id="11aef-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="11aef-271">后续步骤</span><span class="sxs-lookup"><span data-stu-id="11aef-271">Next steps</span></span>

- <span data-ttu-id="11aef-272">详细了解[混合应用设计注意事项](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="11aef-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="11aef-273">查看 [GitHub 上此示例的代码](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)并提出改进建议。</span><span class="sxs-lookup"><span data-stu-id="11aef-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>