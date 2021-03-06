---
title: "Azure 中的使用者定義路由和 IP 轉送 | Microsoft Docs"
description: "了解如何在 Azure 中設定使用者定義的路由 (UDR) 和 IP 轉送，以將流量轉送至網路虛擬應用裝置。"
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: tysonn
ms.assetid: c39076c4-11b7-4b46-a904-817503c4b486
ms.service: virtual-network
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: jdial
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: b0c27ca561567ff002bbb864846b7a3ea95d7fa3
ms.openlocfilehash: 93920075a8ad8de4fd650d9cbbfd13b7bc18bf52
ms.lasthandoff: 04/25/2017


---
# <a name="user-defined-routes-and-ip-forwarding"></a>使用者定義的路由和 IP 轉送

當您在 Azure 中將虛擬機器 (VM) 新增到虛擬網路 (VNet) 時，您會發現 VM 能自動透過網路彼此通訊。 您不需要指定閘道，即使 VM 位於不同子網路。 VM 到公開網際網路的通訊同樣適用，甚至當存在 Azure 到您的資料中心的混合連線時，也適用於您的內部網路。

這個通訊流程是可能的，因為 Azure 會使用一連串系統路由來定義 IP 流量流動的方式。 系統路由控制下列案例的通訊流程：

* 從同一子網路。
* 從 VNet 中的子網路到另一個子網路。
* 從 VM 到網際網路。
* 透過 VPN 閘道從 VNet 到另一個 VNet。
* 透過 VNet 對等互連 (服務鏈結)，從 VNet 到另一個 VNet。
* 透過 VPN 閘道從 VNet 到您的內部網路。

下圖顯示 1 個 Vnet、2 個子網路和一些 VM，以及允許 IP 流量流動的系統路由的簡易安裝。

![Azure 中的系統路由](./media/virtual-networks-udr-overview/Figure1.png)

雖然使用系統路由可自動加速您部署的流量，但是也有一些您希望透過虛擬設備控制封包路由的情況。 您可以透過建立使用者定義的路由 (其指定流向指定子網路之封包的下個躍點，改為流向您的虛擬設備)，並啟用做為虛擬設備執行之 VM 的 IP 轉送。

下圖顯示使用者定義的路由和 IP 轉送的範例，強制由一個子網路傳送到另一個子網路的封包於第三個子網路通過虛擬應用裝置。

![Azure 中的系統路由](./media/virtual-networks-udr-overview/Figure2.png)

> [!IMPORTANT]
> 使用者定義的路由會套用至從子網路中的任何資源 (例如，連結至 VM 的網路介面) 離開子網路的流量。 例如，您無法建立路由來指定流量該如何從網際網路進入某個子網路。 接收轉送流量的應用裝置不得與流量來源的裝置位於相同的子網路中。 請永遠為您的應用裝置建立個別的子網路。 
> 
> 

## <a name="route-resource"></a>路由資源
根據在實體網路上的每個節點定義的路由表，封包是透過 TCP/IP 網路路由傳送。 路由表是個別路由的集合，用於決定根據目的地 IP 位址，在何處轉送封包。 路由是由下列項目所組成：

| 屬性 | 說明 | 條件約束 | 考量 |
| --- | --- | --- | --- |
| 位址首碼 |路由所套用的目的地 CIDR，例如 10.1.0.0/16。 |必須為代表公用網際網路、Azure 虛擬網路或內部部署資料中心上之位址的有效 CIDR 範圍。 |請確定 [位址首碼] 未包含 [下一個躍點位址] 的位址，否則您的封包將會進入來源與下一個躍點所形成的迴圈，而永遠不會抵達目的地。 |
| 下一個躍點類型 |要接收封包的 Azure 躍點的類型。 |必須為下列其中一個值： <br/> **虛擬網路**。 代表本機虛擬網路。 例如，如果您在相同的虛擬網路中有兩個子網路 (分別是 10.1.0.0/16 和 10.2.0.0/16)，則路由表中每個子網路的路由的下一個躍點值為「虛擬網路」 。 <br/> **虛擬網路閘道**。 代表 Azure S2S VPN 閘道。 <br/> **網際網路**。 代表 Azure 基礎結構所提供的預設網際網路閘道。 <br/> **虛擬應用裝置**。 代表您新增至 Azure 虛擬網路的虛擬應用裝置。 <br/> **無**。 代表黑洞。 轉送至黑洞的封包將完全不會被轉送。 |請考慮使用「虛擬應用裝置」將流量導向 VM 或 Azure Load Balancer 內部 IP 位址。  此類型允許使用如以下所述的 IP 位址規格。 請考慮使用 [無]  類型來阻止封包流向指定目的地。 |
| 下一個躍點位址 |下一個躍點位址包含應該要將封包轉送到哪個 IP 位址。 只有在下一個躍點類型為虛擬應用裝置 所在的路由中，才允許下一個躍點值。 |必須為當中套用使用者定義路由的虛擬網路內可連線的 IP 位址，而不需通過**虛擬網路閘道**。 IP 位址必須位於所套用的相同虛擬網路上，或位於對等互連的虛擬網路上。 |如果 IP 位址代表 VM，請確定您已在 Azure 中啟用 VM 的 [IP 轉送](#IP-forwarding) 。 如果 IP 位址代表 Azure Load Balancer 的內部 IP 位址，請確定您想要負載平衡的每個連接埠都有對應的負載平衡規則。|

在 Azure PowerShell 中，有些 "NextHopType" 值有不同的名稱︰

* 虛擬網路是 VnetLocal
* 虛擬網路閘道是 VirtualNetworkGateway
* 虛擬應用裝置是 VirtualAppliance
* 網際網路是 Internet
* 無是 None

### <a name="system-routes"></a>系統路由
虛擬網路中建立的每個子網路會自動與包含下列系統路由規則的路由表產生關聯：

* **本機 VNet 規則**：此規則會自動針對虛擬網路中的每個子網路建立。 它會指定 VNet 中 VM 之間的直接連結，沒有下一個中繼躍點。
* **內部部署規則**：此規則適用於目的地為內部部署位址範圍的所有流量，並使用 VPN 閘道作為下一個躍點目的地。
* **網際網路規則**：此規則處理所有公開網際網路 (位址首碼 0.0.0.0/0) 的流量，並使用基礎結構網際網路閘道做為所有網際網路流量的下一個躍點。

### <a name="user-defined-routes"></a>使用者定義的路由
對於大部分的環境，您只需要已經由 Azure 所定義的系統路由。 不過，您可能需要在特定情況下建立路由表，並新增一個或多個路由，例如：

* 透過內部部署網路到網際網路的強制通道。
* 在 Azure 環境中使用虛擬應用裝置。

在上述情況下，您必須建立一個路由表，並將使用者定義的路由新增到該路由表。 您可以擁有多個路由表，而且相同的路由表可以與一個或多個子網路產生關聯。 此外，每個子網路只能與單一路由資料表產生關聯。 子網路中的所有 VM 和雲端服務都使用與該子網路相關聯的路由表。

子網路會依賴系統路由，直到路由表與子網路產生關聯為止。 一旦關聯存在之後，在使用者定義的路由和系統路由之間，就會根據最長前置詞比對 (LPM) 完成。 如果有多個符合相同 LPM 的路由，則會根據其來源，以下列順序選取路由：

1. 使用者定義的路由
2. BGP 路由 (使用 ExpressRoute 時)
3. 系統路由

若要了解如何建立使用者定義的路由，請參閱 [如何在 Azure 中建立路由並啟用 IP 轉送](virtual-network-create-udr-arm-template.md)。

> [!IMPORTANT]
> 使用者定義的路由僅會套用至 Azure VM 和雲端服務。 例如，如果您想要在內部部署網路與 Azure 之間新增防火牆虛擬應用裝置，您必須為您的 Azure 路由表建立一個使用者定義的路由，此路由會將前往內部部署位址空間的所有流量轉送到虛擬應用裝置。 您也可以將使用者定義路由 (UDR) 加入 GatewaySubnet，以透過虛擬應用裝置將所有流量從內部部署轉送到 Azure。 這是近期內的新增功能。
> 
> 

### <a name="bgp-routes"></a>BGP 路由
如果您在內部部署網路與 Azure 之間有 ExpressRoute 連線，可以啟用 BGP，將路由從內部部署網路傳播至 Azure。 這些 BGP 路由的使用方式與系統路由相同，也與每個 Azure 子網路中的使用者定義路由相同。 如需詳細資訊，請參閱 [ExpressRoute 簡介](../expressroute/expressroute-introduction.md)。

> [!IMPORTANT]
> 您可以設定 Azure 環境透過內部部署網路使用強制通道，方法是，為使用 VPN 閘道做為下一個躍點的子網路 0.0.0.0/0 建立使用者定義的路由。 不過，這只有在您使用的是 VPN 閘道，而不是 ExpressRoute 時，才有作用。 若是 Expressroute，強制通道是透過 BGP 設定。
> 
> 

## <a name="ip-forwarding"></a>IP 轉送
如上所述，建立使用者定義的路由的主要原因之一是將流量轉送到虛擬應用裝置。 虛擬應用裝置無非就是一個 VM，可執行用來以某種方式處理網路流量的應用程式，例如防火牆或 NAT 裝置。

此虛擬應用裝置 VM 必須能夠接收未定址到本身的連入流量。 若要讓 VM 接收定址到其他目的地的流量，您必須針對 VM 啟用 IP 轉送。 這是 Azure 設定，不是客體作業系統中的設定。

## <a name="next-steps"></a>後續步驟
* 了解如何 [在資源管理員部署模型中建立路由](virtual-network-create-udr-arm-template.md) 並將其關聯至子網路。 
* 了解如何 [在傳統部署模型中建立路由](virtual-network-create-udr-classic-ps.md) 並將其關聯至子網路。


