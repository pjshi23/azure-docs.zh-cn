---
title: "在 Linux 上使用 C# 创建你的第一个 Service Fabric 应用程序 | Microsoft Docs"
description: "使用 C 创建和部署 Service Fabric 应用程序#"
services: service-fabric
documentationcenter: csharp
author: mani-ramaswamy
manager: timlt
editor: 
ms.assetid: 5a96d21d-fa4a-4dc2-abe8-a830a3482fb1
ms.service: service-fabric
ms.devlang: csharp
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/04/2016
ms.author: subramar
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 9486fcb56b05b22120aef5a8373c6558b2d88d6c


---
# <a name="create-your-first-azure-service-fabric-application"></a>创建第一个 Azure Service Fabric 应用程序
> [!div class="op_single_selector"]
> * [C# - Windows](service-fabric-create-your-first-application-in-visual-studio.md)
> * [Java-Linux](service-fabric-create-your-first-linux-application-with-java.md)
> * [C# - Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
> 
> 

Service Fabric 提供用于在 Linux 上使用 .NET Core 和 Java 构建服务的 SDK。 本教程介绍了如何使用 C# (.NET Core) 创建适用于 Linux 的应用程序并构建服务。

## <a name="prerequisites"></a>先决条件
开始之前，请确保已[设置 Linux 开发环境](service-fabric-get-started-linux.md)。 如果使用的是 Mac OS X，则可以[使用 Vagrant 在虚拟机中设置 Linux 单机环境](service-fabric-get-started-mac.md)。

## <a name="create-the-application"></a>创建应用程序
Service Fabric 应用程序可以包含一个或多个服务，每个服务都在提供应用程序功能时具有特定角色。 用于 Linux 的 Service Fabric SDK 包括 [Yeoman](http://yeoman.io/) 生成器，利用它可以轻松地创建第一个服务和在以后添加其他服务。 让我们使用 Yeoman 来创建具有单项服务的应用程序。

1. 在终端中键入以下命令以开始生成基架：`yo azuresfcsharp`
2. 命名应用程序。
3. 选择第一个服务的类型并将其命名。 对于本教程，我们会选择“可靠角色服务”。
   
   ![适用于 C 的 Service Fabric Yeoman 生成器#][sf-yeoman]

> [!NOTE]
> 有关选项的详细信息，请参阅 [Service Fabric 编程模型概述](service-fabric-choose-framework.md)。
> 
> 

## <a name="build-the-application"></a>构建应用程序
Service Fabric Yeoman 模板包含一个可用于（在导航到应用程序文件夹后）从终端生成应用的生成脚本。

  ```bash
 cd myapp 
 ./build.sh 
  ```

## <a name="deploy-the-application"></a>部署应用程序
生成应用程序后，可以使用 Azure CLI 将其部署到本地群集。

1. 连接到本地 Service Fabric 群集。
   
    ```bash
    azure servicefabric cluster connect
    ```
2. 使用模板中提供的安装脚本可将应用程序包复制到群集的映像存储、注册应用程序类型和创建应用程序的实例。
   
    ```bash
    ./install.sh
    ```
3. 打开浏览器并导航到 http://localhost:19080/Explorer 的 Service Fabric Explorer（如果在 Mac OS X 上使用 Vagrant，则使用 VM 的专用 IP 替换 localhost）。
4. 展开应用程序节点，注意现在有一个条目是用于你的应用程序类型，另一个条目用于该类型的第一个实例。

## <a name="start-the-test-client-and-perform-a-failover"></a>启动测试客户端并执行故障转移
Actor 项目自身未执行任何操作。 它们需要另一个服务或客户端向其发送消息。 Actor 模板包括可用于与 actor 服务进行交互的简单测试脚本。

1. 使用监视实用程序运行该脚本来查看 actor 服务的输出。
   
    ```bash
    cd myactorsvcTestClient
    watch -n 1 ./testclient.sh
    ```
2. 在 Service Fabric Explorer 中，找到承载 actor 服务的主副本的节点。 以下屏幕截图中显示的是节点 3。
   
    ![在 Service Fabric Explorer 中查找主副本][sfx-primary]
3. 单击上一步找到的节点，然后在“操作”菜单中选择“停用(重启)”。 此操作在本地群集中重新启动五个节点中的一个，从而强制故障转移到在另一个节点上运行的一个辅助副本。 在执行此操作时，请注意来自测试客户端的输出，并注意虽然发生故障转移，但是计数器仍将继续递增。

## <a name="next-steps"></a>后续步骤
* [了解有关 Reliable Actors 的详细信息](service-fabric-reliable-actors-introduction.md)
* [使用 Azure CLI 与 Service Fabric 群集交互](service-fabric-azure-cli.md)

<!-- Images -->
[sf-yeoman]: ./media/service-fabric-create-your-first-linux-application-with-csharp/yeoman-csharp.png
[sfx-primary]: ./media/service-fabric-create-your-first-linux-application-with-csharp/sfx-primary.png



<!--HONumber=Nov16_HO2-->


