---
title: Azure VPN 閘道： configuration settings
description: 瞭解 Resource Manager 部署模型中建立之虛擬網路的 VPN 閘道資源和設定。
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: cherylmc
ms.openlocfilehash: 94ad0a05dafe2c405b1b9cb62242675aa54c4432
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424347"
---
# <a name="about-vpn-gateway-configuration-settings"></a>關於 VPN 閘道組態設定

VPN 閘道是一種虛擬網路閘道，可透過公用連線在您的虛擬網路和內部部署位置之間傳送加密流量。 您也可以使用 VPN 閘道，透過 Azure 骨幹，在虛擬網路之間傳送流量。

VPN 閘道連線依賴多個資源的設定，每一個都包含可設定的設定值。 本文各節討論在 Resource Manager 部署模型中所建立之虛擬網路 VPN 閘道相關的資源和設定。 您可以在[關於 VPN 閘道](vpn-gateway-about-vpngateways.md) 一文中找到每個連線解決方案的描述和拓撲圖。

本文中的值適用於 VPN 閘道 (使用 -GatewayType Vpn 的虛擬網路閘道)。 本文並未涵蓋所有閘道類型或區域備援閘道。

* 如需適用於 -GatewayType 'ExpressRoute' 的值，請參閱 [ExpressRoute 的虛擬網路閘道](../expressroute/expressroute-about-virtual-network-gateways.md)。

* 如需了解區域備援閘道，請參閱[關於區域備援閘道](about-zone-redundant-vnet-gateways.md)。

* 如需了解虛擬 WAN，請參閱[關於虛擬 WAN](../virtual-wan/virtual-wan-about.md)。

## <a name="gateway-types"></a><a name="gwtype"></a>閘道類型

每個虛擬網路只能有一個各類型的虛擬網路閘道。 建立虛擬網路閘道時，您必須確定組態的閘道類型是正確的。

-GatewayType 的可用值為：

* Vpn
* ExpressRoute

VPN 閘道需要 `-GatewayType` *Vpn*。

範例：

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased
```

## <a name="gateway-skus"></a><a name="gwsku"></a>閘道 SKU

[!INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

### <a name="configure-a-gateway-sku"></a>設定閘道 SKU

**Azure 入口網站**

如果您使用 Azure 入口網站來建立 Resource Manager 虛擬網路閘道，可以使用下拉式清單選取閘道 SKU。 您看到的選項對應於您選取的閘道類型和 VPN 類型。

**PowerShell**

下列 PowerShell 範例將 `-GatewaySku` 指定為 VpnGw1。 使用 PowerShell 來建立閘道時，您必須先建立 IP 組態，然後使用變數來參考它。 在此範例中，組態變數是 $gwipconfig。

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'US East' -IpConfigurations $gwipconfig -GatewaySku VpnGw1 `
-GatewayType Vpn -VpnType RouteBased
```

**Azure CLI**

```azurecli
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWPIP --resource-group TestRG1 --vnet VNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait
```

###  <a name="resizing-or-changing-a-sku"></a><a name="resizechange"></a>調整大小或變更 SKU

如果您擁有 VPN 閘道，且您想要使用不同的閘道 SKU，您可選擇調整您的閘道 SKU，或是變更為另一個 SKU。 當您變更為另一個閘道 SKU 時，會完全刪除現有的閘道並建立一個新的閘道。 建立閘道最多可能需要45分鐘的時間。 相較之下，當您調整閘道 SKU 的大小時，因為您不需要刪除和重建閘道，所以不會有太多停機時間。 如果您可以選擇調整您閘道 SKU 的大小，而不必加以變更，建議您這麼做。 不過，關於調整大小有一些規則：

1. 除了基本 SKU 之外，您還可以將 VPN 閘道 SKU 的大小調整為相同世代 (Generation1.xml 或 Generation2) 中的另一個 VPN 閘道 SKU。 例如，Generation1.xml 的 VpnGw1 可以調整大小為 VpnGw2 的 Generation1.xml，而不是 VpnGw2 Generation2。
2. 使用舊式閘道 SKU 時，您可以在基本、標準和高效能 SKU 之間調整大小。
3. 您 **無法** 從基本/標準/HighPerformance sku 調整為 VpnGw sku。 您必須改為[變更](#change)為新的 SKU。

#### <a name="to-resize-a-gateway"></a><a name="resizegwsku"></a>調整閘道大小

**Azure 入口網站**

[!INCLUDE [Resize a SKU - portal](../../includes/vpn-gateway-resize-gw-portal-include.md)]

**PowerShell**

[!INCLUDE [Resize a SKU](../../includes/vpn-gateway-gwsku-resize-include.md)]

####  <a name="to-change-from-an-old-legacy-sku-to-a-new-sku"></a><a name="change"></a>從舊式 (舊版) SKU 變更為新的 SKU

[!INCLUDE [Change a SKU](../../includes/vpn-gateway-gwsku-change-legacy-sku-include.md)]

## <a name="connection-types"></a><a name="connectiontype"></a>連線類型

在 Resource Manager 部署模型中，每個組態皆需要特定的虛擬網路閘道連線類型。 `-ConnectionType` 的可用 Resource Manager PowerShell 值為：

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

在下列 PowerShell 範例中，我們會建立需要 *IPsec*連線類型的 S2S 連線。

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

## <a name="vpn-types"></a><a name="vpntype"></a>VPN 類型

當您為 VPN 閘道組態建立虛擬網路閘道時，必須指定 VPN 類型。 您所選擇的 VPN 類型取決於您想要建立的連線拓撲。 例如，P2S 連線需要 RouteBased VPN 類型。 VPN 類型也取決於您使用的硬體。 S2S 組態需要 VPN 裝置。 有些 VPN 裝置僅支援特定 VPN 類型。

您選取的 VPN 類型必須滿足您想建立的解決方案的所有連線需求。 例如，如果您想為相同的虛擬網路建立 S2S VPN 閘道連線和 P2S VPN 閘道連線，您會使用 VPN 類型 *RouteBased* ，因為 P2S 需要 RouteBased VPN 類型。 您也必須確認您的 VPN 裝置支援 RouteBased VPN 連線。 

一旦建立虛擬網路閘道，就無法變更 VPN 類型。 您必須刪除虛擬網路閘道，然後再建立新的。 有兩種 VPN 類型：

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

下列 PowerShell 範例將 `-VpnType` 指定為 *RouteBased*。 在建立閘道時，您必須確定用於組態的 -VpnType 是正確的。

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig `
-GatewayType Vpn -VpnType RouteBased
```

## <a name="gateway-requirements"></a><a name="requirements"></a>閘道需求

[!INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

## <a name="gateway-subnet"></a><a name="gwsub"></a>閘道子網路

建立 VPN 閘道之前，您必須先建立閘道子網路。 閘道子網路包含虛擬網路閘道 VM 與服務所使用的 IP 位址。 建立虛擬網路閘道時，會將閘道 VM 部署到閘道子網路，並為 VM 設定必要的 VPN 閘道設定。 絕不部署任何其他 (例如，其他 Vm) 至閘道子網。 此閘道子網路必須命名為 'GatewaySubnet' 才能正常運作。 將閘道子網路命名為 'GatewaySubnet' 可讓 Azure 知道這是要用來部署虛擬網路閘道 VM 和服務的子網路。

>[!NOTE]
>[!INCLUDE [vpn-gateway-gwudr-warning.md](../../includes/vpn-gateway-gwudr-warning.md)]
>

當您建立閘道子網路時，您可指定子網路包含的 IP 位址數目。 閘道子網路中的 IP 位址會配置給閘道 VM 和閘道服務。 有些組態需要的 IP 位址比其他組態多。 

當您規劃閘道子網大小時，請參閱您打算建立的設定檔。 例如，ExpressRoute/VPN 閘道並存設定需要較大的閘道子網，而不是大部分的其他設定。 此外，您可能會想要確定閘道子網路包含足夠的 IP 位址，以因應未來可能的額外組態需求。 雖然您可以建立小至/29 的閘道子網，但建議您建立/27 或更大 (/27、/26 等閘道子網，) 是否有可用的位址空間。 這會容納大部分的設定。

下列 Resource Manager PowerShell 範例顯示名為 GatewaySubnet 的閘道子網路。 您可以看到 CIDR 標記法指定 /27，這可提供足以供大多數現有組態使用的 IP 位址。

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="local-network-gateways"></a><a name="lng"></a>區域網路閘道

區域網路閘道與虛擬網路閘道並不相同。 建立 VPN 閘道設定時，局域網路閘道通常代表您的內部部署網路和對應的 VPN 裝置。 在傳統部署模型中，區域網路閘道被稱為本機站台。

您可以為局域網路閘道指定內部部署 VPN 裝置的名稱、公用 IP 位址或完整功能變數名稱 (FQDN) ，並指定位於內部部署位置的位址首碼。 Azure 會查看網路流量的目的地位址首碼、查閱您為區域網路閘道指定的組態，然後根據這些來路由傳送封包。 如果您在 VPN 裝置上使用邊界閘道協定 (BGP) ，您將會提供 VPN 裝置的 BGP 對等 IP 位址，以及內部部署網路 (ASN) 的自發系統編號。 您也可以針對使用 VPN 閘道連線的 VNet 對 VNet 組態指定區域網路閘道。

下列 PowerShell 範例會建立新的區域網路閘道︰

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

有時，您會需要修改區域網路閘道設定。 例如，當您新增或修改位址範圍時，或 VPN 裝置的 IP 位址變更時。 請參閱[使用 PowerShell 修改區域網路閘道設定](vpn-gateway-modify-local-network-gateway.md)。

## <a name="rest-apis-powershell-cmdlets-and-cli"></a><a name="resources"></a>REST API、PowerShell Cmdlet、CLI

使用 REST API、PowerShell Cmdlet 或 Azure CLI 來設定 VPN 閘道組態時，如需其他技術資源和特定語法需求，請參閱下列頁面：

| **傳統** | **Resource Manager** |
| --- | --- |
| [PowerShell](/powershell/module/az.network/#networking) |[PowerShell](/powershell/module/az.network#vpn) |
| [REST API](https://msdn.microsoft.com/library/jj154113) |[REST API](/rest/api/network/virtualnetworkgateways) |
| 不支援 | [Azure CLI](/cli/azure/network/vnet-gateway)|

## <a name="next-steps"></a>後續步驟

如需有關可用連線組態的詳細資訊，請參閱[關於 VPN 閘道](vpn-gateway-about-vpngateways.md)。
