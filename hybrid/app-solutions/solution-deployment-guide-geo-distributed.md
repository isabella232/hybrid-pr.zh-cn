---
title: 使用 Azure 和 Azure Stack Hub 通过地理分散式应用定向流量
description: 了解如何使用 Azure 和 Azure Stack Hub 通过地理分散式应用解决方案将流量定向到特定终结点。
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 741ddf2c3ed234788af359dd233f6a656fbea13c
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477348"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>使用 Azure 和 Azure Stack Hub 通过地理分散式应用定向流量

了解如何根据各种指标，使用异地分布式应用模式将流量定向到特定的终结点。 使用基于地理位置的路由和终结点配置创建流量管理器配置文件可确保根据区域要求、企业与国际法规和数据需求将信息路由到终结点。

在此解决方案中，你将构建一个示例环境来完成以下任务：

> [!div class="checklist"]
> - 创建地理分散式应用。
> - 使用流量管理器设置应用目标。

## <a name="use-the-geo-distributed-apps-pattern"></a>使用异地分布式应用模式

应用可以通过异地分布模式跨越区域。 可以默认使用公有云，但某些用户可能需要将数据保留在其所在区域中。 可根据用户的要求将他们定向到最合适的云。

### <a name="issues-and-considerations"></a>问题和注意事项

#### <a name="scalability-considerations"></a>可伸缩性注意事项

在本文中生成的解决方案无法处理可伸缩性。 但是，如果与其他 Azure 和本地解决方案结合使用，则可以满足可伸缩性方面的要求。 有关通过流量管理器创建可自动缩放的混合解决方案的信息，请参阅[在 Azure 中创建跨云缩放解决方案](solution-deployment-guide-cross-cloud-scaling.md)。

#### <a name="availability-considerations"></a>可用性注意事项

与可伸缩性注意事项中提到的一样，此解决方案无法直接处理可用性。 但是，可在此解决方案中实施 Azure 和本地解决方案，以确保所有相关组件具有高可用性。

### <a name="when-to-use-this-pattern"></a>何时使用此模式

- 组织有海外分公司，需要自定义的区域安全性和分发策略。

- 组织的每个办事处提取员工、业务和设施数据，需要根据当地法规和时区报告活动。

- 只要对单个区域内和跨区域的多个应用部署进行应用的水平扩展，即可达到高缩放性要求以处理极高的负载要求。

### <a name="planning-the-topology"></a>规划拓扑

在规划分布式应用的范围之前，最好先了解以下信息：

- **应用的自定义域：** 客户访问应用时使用的自定义域名是什么？ 示例应用的自定义域名是 *www\.scalableasedemo.com。*

- **流量管理器域：** 创建 [Azure 流量管理器配置文件](/azure/traffic-manager/traffic-manager-manage-profiles)时会选择域名。 此名称与 trafficmanager.net 后缀相结合，以注册流量管理器所管理的域条目。 就示例应用而言，选择的名称是 *scalable-ase-demo*。 因此，流量管理器所管理的完整域名是 scalable-ase-demo.trafficmanager.net。

- **缩放应用占用空间的策略：** 确定应用占用空间是否将分布到单个区域、多个区域或这两者的混合区域中的多个应用服务环境。 决策依据应来自于客户流量的来源位置，以及其余应用的支持后端基础结构的伸缩性。 例如，对于 100% 无状态的应用，可以使用每一 Azure 区域多个应用服务环境的组合，乘以跨多个 Azure 区域部署的应用服务环境数，来大幅缩放应用。 由于有 15 个以上的全球 Azure 区域可供选择，客户将可真正构建全球性超高缩放性的应用占用空间。 此处所使用的示例应用中，有三个应用服务环境创建于单个 Azure 区域（美国中南部）。

- **应用服务环境的命名约定：** 每个应用服务环境都需要具有一个唯一名称。 有两个或更多应用服务环境时，命名约定将有助于标识每个应用服务环境。 此处的示例应用中使用了简单的命名约定。 三个应用服务环境的名称分别是 fe1ase、fe2ase 和 fe3ase  。

- **应用的命名约定：** 由于将部署多个应用实例，每个部署的应用实例都要有名称。 使用适用于 Power Apps 的应用服务环境，可以在多个环境中使用相同的应用名称。 由于每个应用服务环境都有唯一的域后缀，开发人员可以选择在每个环境中重复使用完全相同的应用名称。 例如，开发人员可以对应用进行如下命名：myapp.foo1.p.azurewebsites.net、myapp.foo2.p.azurewebsites.net、myapp.foo3.p.azurewebsites.net 等  。 对于此处使用的应用，每个应用实例具有唯一的名称。 所用的应用实例名称是 *webfrontend1*、*webfrontend2* 和 *webfrontend3*。

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub 是 Azure 的扩展。 Azure Stack Hub 将云计算的灵活性和创新性带入你的本地环境，并支持唯一的混合云，以允许你在任何地方构建和部署混合应用。  
> 
> [混合应用设计注意事项](overview-app-design-considerations.md)一文回顾了设计、部署和运行混合应用所需的软件质量要素（位置、可伸缩性、可用性、复原能力、可管理性和安全性）。 这些设计注意事项有助于优化混合应用设计，从而最大限度地减少生产环境中的难题。

## <a name="part-1-create-a-geo-distributed-app"></a>第 1 部分：创建异地分布式应用

在此部分创建 Web 应用。

> [!div class="checklist"]
> - 创建 Web 应用并发布。
> - 向 Azure Repos 中添加代码。
> - 将应用生成指向多个云目标。
> - 管理和配置 CD 过程。

### <a name="prerequisites"></a>先决条件

需要 Azure 订阅并安装 Azure Stack Hub。

### <a name="geo-distributed-app-steps"></a>异地分布式应用步骤

### <a name="obtain-a-custom-domain-and-configure-dns"></a>获取自定义域并配置 DNS

更新域的 DNS 区域文件。 然后，Azure AD 可以验证自定义域名的所有权。 将 [Azure DNS](/azure/dns/dns-getstarted-portal) 用于 Azure 中的 Azure/Office 365/外部 DNS 记录，或在[其他 DNS 注册机构](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)中添加 DNS 条目。

1. 向公共注册机构注册自定义域。

2. 登录到域的域名注册机构。 可能需要由获批准的管理员进行 DNS 更新。

3. 通过添加 Azure AD 提供的 DNS 条目来更新域的 DNS 区域文件。 DNS 条目不会更改行为，例如邮件路由或 Web 托管。

### <a name="create-web-apps-and-publish"></a>创建 Web 应用并发布

设置混合持续集成/持续交付 (CI/CD)，以将 Web 应用部署到 Azure 和 Azure Stack Hub，并自动将更改推送到这两个云中。

> [!Note]  
> 需要在 Azure Stack Hub 上创建适当的合成映像用于运行 Windows Server 和 SQL，并需要部署应用服务。 有关详细信息，请参阅[在 Azure Stack Hub 上部署应用服务的先决条件](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md)。

#### <a name="add-code-to-azure-repos"></a>向 Azure Repos 中添加代码

1. 使用在 Azure Repos 上**拥有项目创建权限的帐户**登录到 Visual Studio。

    CI/CD 可同时应用到应用代码和基础结构代码。 使用 [Azure 资源管理器模板](https://azure.microsoft.com/resources/templates/)进行专用与托管的云开发。

    ![在 Visual Studio 中连接到项目](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. 创建并打开默认 Web 应用以**克隆存储库**。

    ![在 Visual Studio 中克隆存储库](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>在这两个云中创建 Web 应用部署

1. 编辑 **WebApplication.csproj** 文件：选择 `Runtimeidentifier` 并添加 `win10-x64`。 （请参阅[独立部署](/dotnet/core/deploying/deploy-with-vs#simpleSelf)文档。）

    ![在 Visual Studio 中编辑 Web 应用项目文件](media/solution-deployment-guide-geo-distributed/image3.png)

2. 使用团队资源管理器**将代码签入到 Azure Repos**。

3. 确认**应用程序代码**已签入到 Azure Repos。

### <a name="create-the-build-definition"></a>创建生成定义

1. **登录到 Azure Pipelines** 以确认能够创建生成定义。

2. 添加 `-r win10-x64` 代码。 使用 .NET Core 触发独立部署时需要添加此代码。

    ![在 Azure Pipelines 中向生成定义中添加代码](media/solution-deployment-guide-geo-distributed/image4.png)

3. **运行生成**。 [独立部署生成](/dotnet/core/deploying/deploy-with-vs#simpleSelf)过程将发布可在 Azure 和 Azure Stack Hub 上运行的项目。

#### <a name="using-an-azure-hosted-agent"></a>使用 Azure 托管代理

在 Azure Pipelines 中使用托管代理是生成和部署 Web 应用的便捷做法。 维护和升级由 Microsoft Azure 自动执行，从而实现不间断的开发、测试和部署。

### <a name="manage-and-configure-the-cd-process"></a>管理和配置 CD 过程

Azure DevOps Services 提供高度可配置、可管理的管道，用于将内容发布到多个环境（例如开发、过渡、QA 和生产环境）；在特定的阶段要求审批。

## <a name="create-release-definition"></a>创建发布定义

1. 选择“加号”按钮可在 Azure DevOps Services 的“生成和发布”部分的“发布”选项卡下添加新发布  。

    ![在 Azure DevOps Services 中创建发布定义](media/solution-deployment-guide-geo-distributed/image5.png)

2. 应用“Azure 应用服务部署”模板。

   ![在 Azure DevOps Services 中应用 Azure 应用服务部署模板](media/solution-deployment-guide-geo-distributed/image6.png)

3. 在“添加项目”下，为 Azure 云生成应用添加项目。

   ![在 Azure DevOps Services 中将生成工件添加到 Azure 云生成](media/solution-deployment-guide-geo-distributed/image7.png)

4. 在“管道”选项卡下选择环境的“阶段和任务”链接，并设置 Azure 云环境值。

   ![在 Azure DevOps Services 中设置 Azure 云环境值](media/solution-deployment-guide-geo-distributed/image8.png)

5. 设置**环境名称**，并选择 Azure 云终结点的 **Azure 订阅**。

      ![在 Azure DevOps Services 中选择 Azure 订阅的 Azure 云终结点](media/solution-deployment-guide-geo-distributed/image9.png)

6. 在“应用服务名称”下，设置所需的 Azure 应用服务名称。

      ![在 Azure DevOps Services 中设置 Azure 应用服务名称](media/solution-deployment-guide-geo-distributed/image10.png)

7. 在 Azure 云托管环境的“代理队列”下输入“Hosted VS2017”。

      ![在 Azure DevOps Services 中为 Azure 云托管环境设置“代理队列”](media/solution-deployment-guide-geo-distributed/image11.png)

8. 在“部署 Azure 应用服务”菜单中，为环境选择有效的**包或文件夹**。 选择**文件夹位置**旁边的“确定”。
  
      ![在 Azure DevOps Services 中为 Azure App Service 环境选择包或文件夹](media/solution-deployment-guide-geo-distributed/image12.png)

      ![在 Azure DevOps Services 中为 Azure App Service 环境选择包或文件夹](media/solution-deployment-guide-geo-distributed/image13.png)

9. 保存所有更改并返回**发布管道**。

    ![在 Azure DevOps Services 中保存发布管道中的更改](media/solution-deployment-guide-geo-distributed/image14.png)

10. 选择 Azure Stack Hub 应用的生成以添加新项目。

    ![在 Azure DevOps Services 中为 Azure Stack Hub 应用添加新的生成工件](media/solution-deployment-guide-geo-distributed/image15.png)


11. 通过应用 Azure 应用服务部署额外添加一个环境。

    ![在 Azure DevOps Services 中将环境添加到 Azure 应用服务部署](media/solution-deployment-guide-geo-distributed/image16.png)

12. 将新环境命名为 Azure Stack Hub。

    ![在 Azure DevOps Services 中为 Azure 应用服务部署中的环境命名](media/solution-deployment-guide-geo-distributed/image17.png)

13. 在“任务”选项卡下找到 Azure Stack Hub 环境。

    ![在 Azure DevOps Services 中 Azure Stack Hub 环境](media/solution-deployment-guide-geo-distributed/image18.png)

14. 选择 Azure Stack Hub 终结点的订阅。

    ![在 Azure DevOps Services 中选择 Azure Stack Hub 终结点的订阅](media/solution-deployment-guide-geo-distributed/image19.png)

15. 将 Azure Stack Hub Web 应用名称设置为应用服务名称。

    ![在 Azure DevOps Services 中设置 Azure Stack Hub Web 应用名称](media/solution-deployment-guide-geo-distributed/image20.png)

16. 选择 Azure Stack Hub 代理。

    ![在 Azure DevOps Services 中选择 Azure Stack Hub 代理名称](media/solution-deployment-guide-geo-distributed/image21.png)

17. 在“部署 Azure 应用服务”部分下，为环境选择有效的**包或文件夹**。 选择文件夹位置旁边的“确定”。

    ![在 Azure DevOps Services 中为 Azure 应用服务部署选择文件夹](media/solution-deployment-guide-geo-distributed/image22.png)

    ![在 Azure DevOps Services 中为 Azure 应用服务部署选择文件夹](media/solution-deployment-guide-geo-distributed/image23.png)

18. 在“变量”选项卡下添加名为 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` 的变量，将其值设置为 **true**，将范围设置为 Azure Stack Hub。

    ![在 Azure DevOps Services 中向 Azure 应用部署中添加变量](media/solution-deployment-guide-geo-distributed/image24.png)

19. 选择两个项目中的“持续”部署触发器图标，并启用“持续”部署触发器。

    ![在 Azure DevOps Services 中选择连续部署触发器](media/solution-deployment-guide-geo-distributed/image25.png)

20. 选择 Azure Stack Hub 环境中的“部署前”条件图标，并将触发器设置为“发布后”。

    ![在 Azure DevOps Services 中选择部署前的条件](media/solution-deployment-guide-geo-distributed/image26.png)

21. 保存所有更改。

> [!Note]  
> 任务的某些设置可能已在从模板创建发布定义时自动定义为[环境变量](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables)。 无法在任务设置中修改这些设置；必须选择父环境项才能编辑这些设置。

## <a name="part-2-update-web-app-options"></a>第 2 部分：更新 Web 应用选项

[Azure 应用服务](/azure/app-service/overview)提供高度可缩放、自修复的 Web 托管服务。

![Azure 应用服务](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - 将现有的自定义 DNS 名称映射到 Azure Web 应用。
> - 使用 **CNAME 记录**和 **A 记录**将自定义 DNS 名称映射到应用服务。

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>将现有的自定义 DNS 名称映射到 Azure Web 应用

> [!Note]  
> 对除根域（例如 northwind.com）以外的所有自定义 DNS 名称使用 CNAME。

若要将实时站点及其 DNS 域名迁移到应用服务，请参阅[将活动 DNS 名称迁移到 Azure 应用服务](/azure/app-service/manage-custom-dns-migrate-domain)。

### <a name="prerequisites"></a>先决条件

若要完成此解决方案，请执行以下操作：

- [创建一个应用服务应用](/azure/app-service/)，或者使用为其他解决方案创建的应用。

- 购买一个域名并确保你对你的域提供商的 DNS 注册表拥有访问权限。

更新域的 DNS 区域文件。 然后，Azure AD 将会验证自定义域名的所有权。 将 [Azure DNS](/azure/dns/dns-getstarted-portal) 用于 Azure 中的 Azure/Office 365/外部 DNS 记录，或在[其他 DNS 注册机构](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/)中添加 DNS 条目。

- 向公共注册机构注册自定义域。

- 登录到域的域名注册机构。 （可能需要由获批准的管理员进行 DNS 更新。）

- 通过添加 Azure AD 提供的 DNS 条目来更新域的 DNS 区域文件。

例如，若要添加 northwindcloud.com 和 www\.northwindcloud.com 的 DNS 条目，请配置 northwindcloud.com 根域的 DNS 设置。

> [!Note]  
> 可以通过 [Azure 门户](/azure/app-service/manage-custom-dns-buy-domain)购买域名。 若要映射自定义 DNS 名称到 Web 应用，Web 应用的[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/)必须位于付费层（“共享”、“基本”、“标准”或“高级”）。   

### <a name="create-and-map-cname-and-a-records"></a>创建和映射 CNAME 和 A 记录

#### <a name="access-dns-records-with-domain-provider"></a>通过域提供商访问 DNS 记录

> [!Note]  
>  使用 Azure DNS 为 Azure Web 应用配置自定义 DNS 名称。 有关详细信息，请参阅[使用 Azure DNS 为 Azure 服务提供自定义域设置](/azure/dns/dns-custom-domain)。

1. 登录到主要提供商的网站。

2. 查找管理 DNS 记录的页面。 每个域提供商都有自己的 DNS 记录接口。 查找站点中标记为“域名”、“DNS”或“名称服务器管理”的区域。  

可以在“我的域”中查看“DNS 记录”页。 找到名为“区域文件”、“DNS 记录”或“高级配置”的链接  。

以下屏幕截图是 DNS 记录页的一个示例：

![示例 DNS 记录页](media/solution-deployment-guide-geo-distributed/image28.png)

1. 在域名注册器中，选择“添加或创建”以创建记录。 某些提供商提供了不同的链接来添加不同的记录类型。 查阅提供商的文档。

2. 添加一条 CNAME 记录，以便将子域映射到应用的默认主机名。

   在 www\.northwindcloud.com 域示例中，添加将名称映射到 `<app_name>.azurewebsites.net` 的 CNAME 记录。

添加 CNAME 后，“DNS 记录”页如以下示例所示：

![在门户中导航到 Azure 应用](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>在 Azure 中启用 CNAME 记录映射

1. 在新选项卡中，登录到 Azure 门户。

2. 转到“应用服务”。

3. 选择 Web 应用。

4. 在 Azure 门户中的应用页左侧导航窗格中，选择“自定义域”。

5. 选择“添加主机名”旁边的 **+** 图标。

6. 键入完全限定的域名，如 `www.northwindcloud.com`。

7. 选择“验证”。

8. 如果指示，将其他类型（`A` 或 `TXT`）的附加记录添加到域名注册器 DNS 记录。 Azure 将提供这些记录的值和类型：

   a.  要映射到应用 IP 地址的 A 记录。

   b.  要映射到应用默认主机名 `<app_name>.azurewebsites.net` 的 TXT 记录。 应用服务仅在配置时使用此记录验证自定义域所有权。 验证后，删除 TXT 记录。

9. 在“域注册器”选项卡中完成此任务，然后重新验证，直到激活“添加主机名”按钮。

10. 确保“主机名记录类型”设置为“CNAME”（ www.example.com 或任何子域） 。

11. 选择“添加主机名”。

12. 键入完全限定的域名，如 `northwindcloud.com`。

13. 选择“验证”。 “添加”已激活。

14. 确保“主机名记录类型”设置为“A 记录”(example.com) 。

15. 添加主机名。

    新主机名可能需要经过一段时间后才会反映在应用的“自定义域”页面中。 请尝试刷新浏览器来更新数据。
  
    ![自定义域](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    如果出现错误，则页面底部会显示一条验证错误通知。 ![域验证错误](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  可以重复上述步骤以映射通配符域 (\*.northwindcloud.com)。 这样就可以向此应用服务添加任何其他子域，且无需为每个子域创建单独的 CNAME 记录。 按照注册器说明配置此设置。

#### <a name="test-in-a-browser"></a>在浏览器中测试

浏览至之前配置的 DNS 名称（例如，`northwindcloud.com` 或 `www.northwindcloud.com`）。

## <a name="part-3-bind-a-custom-ssl-cert"></a>第 3 部分：绑定自定义 SSL 证书

在此部分中，我们将执行以下操作：

> [!div class="checklist"]
> - 将自定义 SSL 证书绑定到应用服务。
> - 为应用实施 HTTPS。
> - 使用脚本自动执行 SSL 证书绑定。

> [!Note]  
> 如果需要，请在 Azure 门户中获取客户 SSL 证书，并将其绑定到 Web 应用。 有关详细信息，请参阅 [Azure 应用服务证书教程](/azure/app-service/web-sites-purchase-ssl-web-site)。

### <a name="prerequisites"></a>先决条件

若要完成此解决方案，请执行以下操作：

- [创建应用服务应用。](/azure/app-service/)
- [将自定义 DNS 名称映射到 Web 应用。](/azure/app-service/app-service-web-tutorial-custom-domain)
- 从受信任的证书颁发机构获取 SSL 证书，并使用密钥对请求进行签名。

### <a name="requirements-for-your-ssl-certificate"></a>SSL 证书的要求

若要在应用服务中使用证书，该证书必须满足以下所有要求：

- 已由受信任的证书颁发机构签名。

- 已导出为受密码保护的 PFX 文件。

- 包含长度至少为 2048 位的私钥。

- 包含证书链中的所有中间证书。

> [!Note]  
> 椭圆曲线加密 (ECC) 证书可用于应用服务，但本指南不予讨论。 请咨询证书颁发机构，以获得创建 ECC 证书方面的帮助。

#### <a name="prepare-the-web-app"></a>准备 Web 应用

要将自定义 SSL 证书绑定到 Web 应用，[应用服务计划](https://azure.microsoft.com/pricing/details/app-service/)必须位于“基本”、“标准”或“高级”层  。

#### <a name="sign-in-to-azure"></a>登录 Azure

1. 打开 [Azure 门户](https://portal.azure.com/)，然后转到 Web 应用。

2. 从左侧菜单中选择“应用服务”，然后选择 Web 应用名称。

![在 Azure 门户中选择“Web 应用”](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>检查定价层

1. 在“Web 应用”页的左侧导航窗格中，滚动到“设置”部分，然后选择“纵向扩展(应用服务计划)” 。

    ![Web 应用中的“纵向扩展”菜单](media/solution-deployment-guide-geo-distributed/image34.png)

1. 请确保 Web 应用不在“免费”或“共享”层中 。 深蓝色的框突出显示了 Web 应用的当前层。

    ![查看 Web 应用中的定价层](media/solution-deployment-guide-geo-distributed/image35.png)

“免费”或“共享”层不支持自定义 SSL 。 若要纵向扩展，请遵循下一部分中的步骤，或者关闭“选择定价层”页面并跳到[上传和绑定 SSL 证书](/azure/app-service/app-service-web-tutorial-custom-ssl)。

#### <a name="scale-up-your-app-service-plan"></a>纵向扩展应用服务计划

1. 选择“基本”、“标准”或“高级”层。  

2. 选择“选择”。

![选择 Web 应用的定价层](media/solution-deployment-guide-geo-distributed/image36.png)

显示通知时，缩放操作完成。

![扩展通知](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>绑定 SSL 证书并合并中间证书

合并链中的多个证书。

1. 在文本编辑器中打开收到的所有证书。

2. 创建名为 mergedcertificate.crt 的合并证书文件。 在文本编辑器中，将每个证书的内容复制到此文件。 证书的顺序应遵循证书链中的顺序，以你的证书开头，以根证书结尾， 如以下示例所示：

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>将证书导出为 PFX

使用证书生成的私钥导出合并的 SSL 证书。

私钥文件是通过 OpenSSL 创建的。 若要将证书导出为 PFX，请运行以下命令，并将占位符 `<private-key-file>` 和 `<merged-certificate-file>` 分别替换为私钥路径和合并的证书文件：

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

出现提示时，请定义导出密码，以便稍后将 SSL 证书上传到应用服务。

如果使用 IIS 或 **Certreq.exe** 生成证书请求，请将证书安装到本地计算机，然后[将证书导出为 PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11))。

#### <a name="upload-the-ssl-certificate"></a>上传 SSL 证书

1. 在 Web 应用的左侧导航栏中，选择“SSL 设置”。

2. 选择“上传证书”。

3. 在“PFX 证书文件”中，选择 PFX 文件。

4. 在“证书密码”中，键入导出 PFX 文件时创建的密码。

5. 选择“上传”。

    ![上传 SSL 证书](media/solution-deployment-guide-geo-distributed/image38.png)

应用服务上传完证书后，该证书会显示在“SSL 设置”页中。

![SSL 设置](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>绑定 SSL 证书

1. 在“SSL 绑定”部分中，选择“添加绑定” 。

    > [!Note]  
    >  如果证书已上传，但未出现在“主机名”下拉列表的域名中，请尝试刷新浏览器页面。

2. 在“添加 SSL 绑定”页中，使用下拉列表选择要保护的域名，然后选择要使用的证书。

3. 在“SSL 类型”中，选择是要使用 [**服务器名称指示 (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) 还是使用基于 IP 的 SSL。

    - **基于 SNI 的 SSL**：可添加多个基于 SNI 的 SSL 绑定。 选择此选项可以使用多个 SSL 证书来保护同一 IP 地址上的多个域。 大多数新式浏览器（包括 Internet Explorer、Chrome、Firefox 和 Opera）都支持 SNI（在[服务器名称指示](https://wikipedia.org/wiki/Server_Name_Indication)中了解更全面的浏览器支持信息）。

    - **基于 IP 的 SSL**：只能添加一个基于 IP 的 SSL 绑定。 选择此选项只能使用一个 SSL 证书来保护专用公共 IP 地址。 若要保护多个域，必须使用同一个 SSL 证书来保护所有这些域。 基于 IP 的 SSL 是 SSL 绑定的传统选项。

4. 选择“添加绑定”。

    ![添加 SSL 绑定](media/solution-deployment-guide-geo-distributed/image40.png)

应用服务上传完证书后，该证书会显示在“SSL 绑定”部分中。

![SSL 绑定完成上传](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>重新映射 IP SSL 的 A 记录

如果不在 Web 应用中使用基于 IP 的 SSL，请跳到[针对自定义域测试 HTTPS](/azure/app-service/app-service-web-tutorial-custom-ssl)。

默认情况下，Web 应用使用共享的公共 IP 地址。 将证书与基于 IP 的 SSL 绑定时，应用服务会为 Web 应用创建新的专用 IP 地址。

将 A 记录映射到 Web 应用时，必须使用专用 IP 地址更新域注册表。

将使用新的专用 IP 地址更新“自定义域”页。 复制此 [IP 地址](/azure/app-service/app-service-web-tutorial-custom-domain)，然后将 [A 记录](/azure/app-service/app-service-web-tutorial-custom-domain)重新映射到此新 IP 地址。

#### <a name="test-https"></a>测试 HTTPS

在不同的浏览器中，请转到 `https://<your.custom.domain>` 以确保提供了 Web 应用。

![浏览到 Web 应用](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> 如果发生证书验证错误，原因可能是自签名证书所致，或者在导出到 PFX 文件时遗漏了中间证书。

#### <a name="enforce-https"></a>实施 HTTPS

默认情况下，任何人都可使用 HTTP 访问 Web 应用。 到 HTTPS 端口的所有 HTTP 请求都可进行重定向。

在“Web 应用”页中，选择“SL 设置”。 然后，在“仅 HTTPS”中，选择“启用”。

![实施 HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

该操作完成后，将转到指向应用的任一 HTTP URL。 例如：

- https://<app_name>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>强制实施 TLS 1.1/1.2

默认情况下，应用允许 [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0，但后者已不再被行业标准（如 [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)）视为安全。 若要强制实施更高的 TLS 版本，请按照下列步骤操作：

1. 在“Web 应用”页的左侧导航窗格中，选择“SSL 设置”。

2. 在“TLS 版本”中，选择最低 TLS 版本。

    ![强制实施 TLS 1.1 或 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>创建流量管理器配置文件

1. 选择“创建资源” > “网络” > “流量管理器配置文件” > “创建”   。

2. 在“创建流量管理器配置文件”中，按如下所示完成操作：

    1. 在“名称”中，提供配置文件的名称。 此名称在 traffic manager.net 区域中必须是唯一的，并将生成 DNS 名称 trafficmanager.net，该名称用于访问流量管理器配置文件。

    2. 在“路由方法”中，选择“地理路由方法” 。

    3. 在“订阅”中，选择要在其下创建此配置文件的订阅。

    4. 在**资源组**中，创建新的资源组，以在其下放置此配置文件。

    5. 在**资源组位置**中，选择资源组的位置。 此设置指的是资源组的位置，对全局部署的流量管理器配置文件没有影响。

    6. 选择“创建”。

    7. 流量管理器配置文件的全局部署完成后，它会在相应的资源组中作为资源之一列出。

        ![创建流量管理器配置文件中的资源组](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>添加流量管理器终结点

1. 在门户的搜索栏中，搜索在前面部分创建的流量管理器配置文件名称，并在显示的结果中选择该流量管理器配置文件。

2. 在“流量管理器配置文件”的“设置”部分，选择“终结点”  。

3. 选择 **添加** 。

4. 添加 Azure Stack Hub 终结点。

5. 对于“类型”，请选择“外部终结点” 。

6. 为此终结点提供“名称”，最好是 Azure Stack Hub 的名称。

7. 对于完全限定的域名 (FQDN)，请使用 Azure Stack Hub Web 应用的外部 URL。

8. 在“异地映射”下，选择资源所在的区域/洲。 例如欧洲。

9. 在显示的“国家/地区”下拉列表中，选择适用于此终结点的国家/地区。 例如德国。

10. 使“添加为已禁用”保持未选中状态。

11. 选择“确定”。

12. 添加 Azure 终结点：

    1. 对于“类型”，选择“Azure 终结点” 。

    2. 为终结点提供名称。

    3. 对于“目标资源类型”，选择“应用服务” 。

    4. 对于“目标资源”，选择“选择应用服务”以显示同一订阅下的 Web 应用列表 。 在“资源”中，选取用作第一个终结点的应用服务。

13. 在“异地映射”下，选择资源所在的区域/洲。 例如，北美/中美/加勒比地区。

14. 在显示“国家/地区”下拉菜单下，将此位置留空以选择上述所有区域分组。

15. 使“添加为已禁用”保持未选中状态。

16. 选择“确定”。

    > [!Note]  
    >  至少创建一个地理范围为“全部(世界)”的终结点，作为资源的默认终结点。

17. 添加完这两个终结点后，这两个终结点会显示在“流量管理器配置文件”中，并且其监视状态为“联机” 。

    ![流量管理器配置文件终结点状态](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>全球企业依赖 Azure 的异地分发功能

通过 Azure 流量管理器和地理位置特定的终结点定向数据流量，全球企业可以遵守区域法规，保持数据的合规性和安全性，这对于本地和远程业务成功而言都至关重要。

## <a name="next-steps"></a>后续步骤

- 若要详细了解 Azure 云模式，请参阅[云设计模式](/azure/architecture/patterns)。
