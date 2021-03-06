---
title: "服务总线中继概述 | Microsoft Docs"
description: "服务总线中继的概述。"
services: service-bus
documentationcenter: .net
author: sethmanheim
manager: timlt
editor: 
ms.assetid: 1038a2d8-5def-4f48-8703-cb0070fc5f10
ms.service: service-bus
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 09/01/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 0482096cbec6a5e4b7b13ea662a180cd9b96e85f


---
# <a name="overview-of-service-bus-relay"></a>服务总线中继概述
服务总线的一个主要组件是集中化（但高度负载平衡）的 *中继* 服务，该服务使你能够构建可在 Azure 数据中心和你自己的本地企业环境中运行的混合应用程序。  服务总线中继支持各种不同的传输协议和 Web 服务标准。 包括 SOAP、WS-* 甚至 REST。 中继服务简化了混合应用程序，通过允许安全地向公有云公开位于企业网络内的 Windows Communication Foundation (WCF) 服务，无需打开防火墙连接，也无需对企业网络基础结构进行彻底更改。 

![WCF 中继概念](./media/service-bus-relay-overview/sb-relay-01.png)

中继服务支持传统的单向消息传送、请求/响应消息传送和对等消息传送。 它还支持 Internet 范围的事件分发，以实现发布/订阅方案和双向套接字通信，从而提高点到点通信效率。 

在中继消息模式中，本地服务会通过出站端口连接至中继服务，并创建双向套接字用于绑定至特定集合地址的通信。 客户端然后可以通过将消息发送到抵达会合地址的中继服务来与内部服务通信。 中继服务接着通过已部署的双向套接字将消息“中继”到本地服务。 客户端不需要与本地服务建立直接连接，也不需要了解服务所在的位置，并且本地服务无需在防火墙上打开任何入站端口。

你可以使用一套 WCF“中继”绑定在本地服务与中继服务之间发起连接。 在幕后，中继绑定将映射到新的传输绑定元素，这些元素旨在创建与云中服务总线集成的 WCF 通道组件。 

## <a name="next-steps"></a>后续步骤
有关服务总线中继的详细信息，请参阅以下主题。

* [Azure 服务总线体系结构概述](../service-bus-messaging/service-bus-fundamentals-hybrid-solutions.md)
* [如何使用服务总线 WCF 中继服务](service-bus-dotnet-how-to-use-relay.md)




<!--HONumber=Nov16_HO2-->


