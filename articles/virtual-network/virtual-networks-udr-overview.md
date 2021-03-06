---
title: "什么是用户定义的路由和 IP 转发？"
description: "了解如何在 Azure 中使用用户定义的路由 (UDR) 和 IP 转发将流量转发到虚拟设备。"
services: virtual-network
documentationcenter: na
author: jimdial
manager: carmonm
editor: tysonn
ms.assetid: c39076c4-11b7-4b46-a904-817503c4b486
ms.service: virtual-network
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: jdial
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7ae1803a299a5fb569ea0ca8a1ce68c33df1a769


---
# <a name="what-are-user-defined-routes-and-ip-forwarding"></a>什么是用户定义的路由和 IP 转发？
当在 Azure 中将虚拟机 (VM) 添加到虚拟网络 (VNet) 时，将会注意到 VM 能够自动通过网络进行相互通信。 你不需要指定网关，即使这些 VM 位于不同子网中。 存在从 Azure 到你自己的数据中心的混合连接时，这同样适用于从 VM 到公共 Internet 甚至到本地网络的通信。

之所以允许这种通信流，是因为 Azure 使用一系列系统路由来定义 IP 流量的流动方式。 系统路由通过以下方案来控制通信流：

* 从同一子网内。
* 在 VNet 中从一个子网到另一个子网。
* 从 VM 到 Internet。
* 通过 VPN 网关从一个 VNet 到另一个 VNet。
* 通过 VPN 网关从 VNet 到本地网络。

下图显示了通过一个 VNet、两个子网、数个 VM 以及允许 IP 通信流动的系统路由完成的简单设置。

![Azure 中的系统路由](./media/virtual-networks-udr-overview/Figure1.png)

尽管使用系统路由可以自动加快通信以方便部署，但在某些情况下，你需要通过虚拟设备来控制数据包的路由。 为此，你可以创建用户定义的路由来指定下一跃点，方便数据包流向特定的子网并转到你的虚拟设备，并可为作为虚拟设备运行的 VM 启用 IP 转发。

下图显示了用户定义的路由和 IP 转发的一个示例，它强制将数据包从一个子网发送到另一个子网，继而通过第三个子网上的虚拟设备。

![Azure 中的系统路由](./media/virtual-networks-udr-overview/Figure2.png)

> [!IMPORTANT]
> 用户定义的路由将仅应用于离开子网的流量。 例如，无法创建路由以指定流量如何从 Internet 流入子网。 此外，流量所转发到的设备不能与流量来源设备位于同一子网中。 始终为你的设备创建单独的子网。 
> 
> 

## <a name="route-resource"></a>路由资源
数据包将根据在物理网络的每个节点上定义的路由表经 TCP/IP 网络进行路由。 路由表是各个路由的集合，用于确定如何根据目标 IP 地址来转发数据包。 路由由以下项组成：

| 属性 | 说明 | 约束 | 注意事项 |
| --- | --- | --- | --- |
| 地址前缀 |将路由应用到的目标 CIDR，例如 10.1.0.0/16。 |必须是表示公共 Internet、Azure 虚拟网络或本地数据中心中的地址的有效 CIDR 范围。 |请确保**地址前缀**中不包含**下一跃点地址**的地址，否则数据包将进入从源到下一跃点的循环，而从不会到达目的地。 |
| 下一跃点类型 |数据包应发送到的 Azure 跃点的类型。 |必须是以下值之一： <br/> **虚拟网络**。 表示本地虚拟网络。 例如，如果你在同一虚拟网络中有两个子网 10.1.0.0/16 和 10.2.0.0/16，则路由表中每个子网的路由的下一跃点值将为 *虚拟网络*。 <br/> **虚拟网络网关**。 表示 Azure S2S VPN 网关。 <br/> **Internet**。 表示由 Azure 基础结构提供的默认 Internet 网关。 <br/> **虚拟设备**。 表示已添加到 Azure 虚拟网络的虚拟设备。 <br/> **无**。 表示黑洞。 转发到黑洞的数据包根本就不会进行转发。 |请考虑使用“无”  类型，以停止将数据包流动到给定目标。 |
| 下一跃点地址 |下一跃点地址包含应将数据包转发到的 IP 地址。 下一跃点值只允许在下一跃点类型为 *虚拟设备*的路由中使用。 |并且必须是在应用用户定义路由的虚拟网络中可以访问的 IP 地址。 |如果 IP 地址表示 VM，请确保在 Azure 中为 VM 启用“IP 转发” [](#IP-forwarding) 。 |

在 Azure PowerShell 中，某些“NextHopType”值具有不同名称：

* 虚拟网络是 VnetLocal
* 虚拟网络网关是 VirtualNetworkGateway
* 虚拟设备是 VirtualAppliance
* Internet 是“Internet”
* 无是“无”

### <a name="system-routes"></a>系统路由
在虚拟网络中创建的每个子网都会自动与其中包含以下系统路由规则的路由表关联：

* **本地 VNet 规则**：将为虚拟网络中的每个子网自动创建此规则。 它指定在 VNet 中的 VM 之间存在直接链接，其间没有下一跃点。
* **本地规则**：此规则适用于要发送到本地地址范围的所有流量，并使用 VPN 网关作为下一跃点目标。
* **Internet 规则**：此规则处理要发送到公共 Internet（地址前缀 0.0.0.0/0）的所有流量，并使用基础结构 Internet 网关作为要发送到 Internet 的所有流量的下一跃点。

### <a name="user-defined-routes"></a>用户定义路由
对于大多数环境，将仅需要已由 Azure 定义的系统路由。 不过，你可能需要创建路由表并在特定情况下创建一个或多个路由，例如：

* 强制通过本地网络以隧道方式连接到 Internet。
* 在 Azure 环境中使用虚拟设备。

在上面的方案中，你将需要创建一个路由表并向其添加用户定义的路由。 你可以有多个路由表，而同一个路由表则可与一个或多个子网相关联。 每个子网只能与一个路由表相关联。 子网中的所有 VM 和云服务都使用与该子网关联的路由表。

在将路由表关联到子网之前，子网依赖于系统路由。 建立关联以后，将根据最长前缀匹配 （LPM） 在用户定义的路由和系统路由之间进行路由。 如果有多个路由的 LPM 匹配情况相同，则按以下顺序根据路由源来选择路由：

1. 用户定义路由
2. BGP 路由（当使用 ExpressRoute 时）
3. 系统路由

若要了解如何创建用户定义路由，请参阅 [如何在 Azure 中创建路由和启用 IP 转发](virtual-network-create-udr-arm-template.md)。

> [!IMPORTANT]
> 用户定义路由仅适用于 Azure VM 和云服务。 例如，如果你想要在本地网络和 Azure 之间添加防火墙虚拟设备，则需为 Azure 路由表创建用户定义路由，以便将目标为本地地址空间的所有流量转发到虚拟设备。 还可以将用户定义的路由 (UDR) 添加到网关子网，以便通过虚拟设备将所有流量从本地转发到 Azure。 这是最近添加的功能。
> 
> 

### <a name="bgp-routes"></a>BGP 路由
如果在本地网络和 Azure 之间存在 ExpressRoute 连接，则可通过 BGP 将路由从本地网络传播到 Azure。 在每个 Azure 子网中，这些 BGP 路由的使用方式与系统路由和用户定义路由相同。 有关详细信息，请参阅 [ExpressRoute 简介](../expressroute/expressroute-introduction.md)。

> [!IMPORTANT]
> 你可以将 Azure 环境配置为使用强制方式通过隧道来连接你的本地网络，即为子网 0.0.0.0/0 创建一个用户定义路由，而该子网则使用 VPN 网关作为下一跃点。 但是，仅在使用 VPN 网关而非 ExpressRoute 的情况下，此方法才起作用。 对于 ExpressRoute，强制隧道是通过 BGP 配置的。
> 
> 

## <a name="ip-forwarding"></a>IP 转发
如上所述，之所以要创建用户定义路由，其中一个主要原因是为了将流量转发到虚拟设备。 虚拟设备只是一个 VM，该 VM 所运行的应用程序用于通过某种方式（例如防火墙或 NAT 设备）处理网络流量。

此虚拟设备 VM 必须能够接收不发送给自身的传入流量。 若要允许 VM 接收发送到其他目标的流量，必须为该 VM 启用 IP 转发。 这是 Azure 设置，不是来宾操作系统中的设置。

## <a name="next-steps"></a>后续步骤
* 了解如何 [在 Resource Manager 部署模型中创建路由](virtual-network-create-udr-arm-template.md) 并将路由关联到子网。 
* 了解如何 [在经典部署模型中创建路由](virtual-network-create-udr-classic-ps.md) 并将路由关联到子网。




<!--HONumber=Nov16_HO2-->


