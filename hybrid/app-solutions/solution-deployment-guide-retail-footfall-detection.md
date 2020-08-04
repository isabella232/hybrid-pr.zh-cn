---
title: 在 Azure 和 Azure Stack Hub 中部署基于 AI 的客流量检测解决方案
description: 了解如何使用 Azure 和 Azure Stack Hub 部署基于 AI 的客流量检测解决方案，以分析零售店的客流量。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5f2e18e164e54f60b1bb7a14026a0c75c7d7ce69
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477161"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>使用 Azure 和 Azure Stack Hub 部署基于 AI 的客流量检测解决方案

本文介绍如何部署基于 AI 的解决方案，该解决方案使用 Azure、Azure Stack Hub 和自定义视觉 AI 开发工具包从实际操作生成见解。

在此解决方案中，你将了解如何：

> [!div class="checklist"]
> - 在边缘部署云本机应用程序捆绑包 (CNAB)。 
> - 部署跨越云边界的应用。
> - 使用自定义视觉 AI 开发工具包在边缘进行推理。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的扩展。 Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。  
> 
> [混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。 这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。

## <a name="prerequisites"></a>先决条件

在开始使用此部署指南之前，请确保：

- 查看[客流量检测模式](pattern-retail-footfall-detection.md)主题。
- 在具备以下条件的情况下，获取对 Azure Stack 开发工具包 (ASDK) 或 Azure Stack Hub 集成系统实例的用户访问权限：
  - 已安装 [Azure Stack Hub 资源提供程序上的 Azure 应用服务](/azure-stack/operator/azure-stack-app-service-overview.md)。 需要具有对 Azure Stack Hub 实例的操作员访问权限，或与管理员合作进行安装。
  - 订阅可提供应用服务和存储配额的套餐。 需要具有操作员访问权限才能创建套餐。
- 获取对 Azure 订阅的访问权限。
  - 如果还没有 Azure 订阅，可以在开始前注册一个[免费试用帐户](https://azure.microsoft.com/free/)。
- 在目录中创建两个服务主体：
  - 一个设置用于 Azure 资源，可在 Azure 订阅范围内进行访问。
  - 一个设置用于 Azure Stack Hub 资源，可在 Azure Stack Hub 订阅范围内进行访问。
  - 若要了解有关创建服务主体和授权访问的详细信息，请参阅[使用应用标识访问资源](/azure-stack/operator/azure-stack-create-service-principals.md)。 如果希望使用 Azure CLI，请参阅[使用 Azure CLI 创建 Azure 服务主体](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)。
- 在 Azure 或 Azure Stack Hub 中部署 Azure 认知服务。
  - 首先，[了解有关认知服务的详细信息](https://azure.microsoft.com/services/cognitive-services/)。
  - 然后，访问[将 Azure 认知服务部署到 Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) 以在 Azure Stack Hub 上部署认知服务。 首先需要注册才能访问预览。
- 克隆或下载未配置的 Azure 自定义视觉 AI 开发工具包。 有关详细信息，请参阅[视觉 AI 开发工具包](https://azure.github.io/Vision-AI-DevKit-Pages/)。
- 注册 Power BI 帐户。
- Azure 认知服务人脸 API 订阅密钥和终结点 URL。 可通过[试用认知服务](https://azure.microsoft.com/try/cognitive-services/?api=face-api)免费试用版获得这两项。 或者，按照[创建认知服务帐户](/azure/cognitive-services/cognitive-services-apis-create-account)中的说明进行操作。
- 安装以下开发资源：
  - [Azure CLI 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/)。 可以通过 Porter 使用所提供的 CNAB 捆绑包清单来部署云应用。
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [适用于 Visual Studio Code 的 Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [适用于 Visual Studio Code 的 Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>部署混合云应用

首先，使用 Porter CLI 生成凭据集，然后部署云应用。  

1. 从 https://github.com/azure-samples/azure-intelligent-edge-patterns 克隆或下载解决方案示例代码。 

1. Porter 将生成一组会自动部署应用的凭据。 在运行凭据生成命令之前，请确保具有以下各项：

    - 用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。
    - Azure 订阅的订阅 ID。
    - 用于访问 Azure Stack Hub 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。
    - Azure Stack Hub 订阅的订阅 ID。
    - Azure 认知服务人脸 API 密钥和资源终结点 URL。

1. 运行 Porter 凭据生成进程，并按照提示进行操作：

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter 还需要运行一组参数。 创建参数文本文件，然后输入以下名称/值对。 如果需要有关任何所需值的帮助，请咨询 Azure Stack Hub 管理员。

   > [!NOTE] 
   > `resource suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。 它必须是由字母和数字组成的唯一字符串，不得超过 8 个字符。

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
   保存文本文件并记下其路径。

1. 现在可以使用 Porter 部署混合云应用。 运行安装命令并监视资源是否部署到 Azure 和 Azure Stack Hub：

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. 部署完成后，请记下以下值：
    - 相机的连接字符串。
    - 图像存储帐户连接字符串。
    - 资源组名称。

## <a name="prepare-the-custom-vision-ai-devkit"></a>准备自定义视觉 AI 开发工具包

接下来，按照[视觉 AI 开发工具包快速入门](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)中所示设置自定义视觉 AI 开发工具包。 此外，还可以使用上一步骤中提供的连接字符串设置和测试相机。

## <a name="deploy-the-camera-app"></a>部署相机应用

使用 Porter CLI 生成凭据集，然后部署相机应用。

1. Porter 将生成一组会自动部署应用的凭据。 在运行凭据生成命令之前，请确保具有以下各项：

    - 用于访问 Azure 资源的服务主体，包括服务主体 ID、密钥和租户 DNS。
    - Azure 订阅的订阅 ID。
    - 在部署云应用时提供的图像存储帐户连接字符串。

1. 运行 Porter 凭据生成进程，并按照提示进行操作：

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter 还需要运行一组参数。 创建参数文本文件，然后输入以下文本。 如果不了解某些所需值，请咨询 Azure Stack Hub 管理员。

    > [!NOTE]
    > `deployment suffix` 值用于确保部署的资源在 Azure 中具有唯一的名称。 它必须是由字母和数字组成的唯一字符串，不得超过 8 个字符。

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    保存文本文件并记下其路径。

4. 现在可以使用 Porter 部署相机应用。 运行安装命令并监视 IoT Edge 部署的创建过程。

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. 通过查看位于 `https://<camera-ip>:3000/`（其中 `<camara-ip>` 是相机 IP 地址）处的相机源，验证相机的部署是否完成。 此步骤最多可能需要 10 分钟。

## <a name="configure-azure-stream-analytics"></a>配置 Azure 流分析

现在，数据已从相机流向 Azure 流分析，我们需要手动授权它与 Power BI 通信。

1. 在 Azure 门户中，打开“所有资源”，然后打开 process-footfall\[yoursuffix\] 作业。

2. 在“流分析作业”窗格的“作业拓扑”部分，选择“输出”选项。

3. 选择“traffic-output”输出接收器。

4. 选择“续订授权”并登录到 Power BI 帐户。
  
    ![Power BI 中的续订授权提示](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. 保存输出设置。

6. 转到“概述”窗格，然后选择“开始”以开始将数据发送到 Power BI 。

7. 选择“现在”作为作业输出启动时间，然后选择“启动”。  可以在通知栏中查看作业状态。

## <a name="create-a-power-bi-dashboard"></a>创建 Power BI 面板

1. 作业成功以后，请转到 [Power BI](https://powerbi.com/)，然后使用工作或学校帐户登录。 如果流分析作业查询输出结果，则创建的 footfall-dataset 数据集存在于“数据集”选项卡下。

2. 从 Power BI 工作区选择“+ 创建”，创建名为“客流量分析”的新面板。

3. 在窗口顶部，选择“添加磁贴”。 然后选择“自定义流数据”和“下一步”。  在“你的数据集”下选择“footfall-dataset” 。 在“可视化效果类型”下拉列表中选择“卡”，然后向“字段”添加“年龄”   。 选择“下一步”，为磁贴输入一个名称，然后选择“应用”，创建该磁贴。

4. 你可以根据需要添加其他字段和卡。

## <a name="test-your-solution"></a>测试解决方案

观察当不同的人走过相机时，Power BI 中创建的卡片中的数据是如何变化的。 录制后，最多可能需要 20 秒才会显示推理。

## <a name="remove-your-solution"></a>删除解决方案

如果要删除解决方案，请使用 Porter 以及针对部署创建的相同参数文件运行以下命令：

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>后续步骤

- 了解有关[混合应用设计注意事项].(overview-app-design-considerations.md)的详细信息
- 查看 [GitHub 上此示例的代码](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)并提出改进建议。
