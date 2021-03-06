---
title: Azure Web 應用程式防火牆 (WAF) 原則總覽
description: 本文概述 Web 應用程式防火牆 (WAF) 全域、每個網站和每個 URI 的原則。
services: web-application-firewall
ms.topic: article
author: winthrop28
ms.service: web-application-firewall
ms.date: 10/14/2020
ms.author: victorh
ms.openlocfilehash: 789a60c80d14301c107824d178a3566d358d287f
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056390"
---
# <a name="azure-web-application-firewall-waf-policy-overview"></a>Azure Web 應用程式防火牆 (WAF) 原則總覽

Web 應用程式防火牆原則包含所有的 WAF 設定和設定。 這包括排除專案、自訂規則、受控規則等等。 然後，這些原則會與應用程式閘道相關聯 (全域) 、每個網站)  (的接聽程式，或以路徑為基礎的規則 (每個 URI) ，以使其生效。

> [!NOTE]
> Azure Web 應用程式防火牆 (WAF) 每個 URI 原則都處於公開預覽狀態。
> 
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 特定功能可能不受支援、功能可能受限，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

您可以建立的原則數目沒有任何限制。 當您建立原則時，它必須與應用程式閘道相關聯才會生效。 它可以與應用程式閘道、接聽程式和路徑型規則的任何組合建立關聯。

## <a name="global-waf-policy"></a>全域 WAF 原則

當您全域建立 WAF 原則的關聯時，應用程式閘道 WAF 背後的每個網站都會受到相同的受控規則、自訂規則、排除專案及任何其他已設定的設定所保護。

如果您想要將單一原則套用至所有網站，您可以將該原則與應用程式閘道產生關聯。 如需詳細資訊，請參閱使用 Azure 入口網站 [建立應用程式閘道的 Web 應用程式防火牆原則](create-waf-policy-ag.md) ，以建立和套用 WAF 原則。 

## <a name="per-site-waf-policy"></a>每個網站的 WAF 原則

透過每個網站的 WAF 原則，您可以使用每個網站原則，在單一 WAF 背後保護安全性需求不同的多個網站。 例如，如果您的 WAF 背後有五個網站，您就可以有五個不同的 WAF 原則 (每個接聽程式各一個)，以自訂排除清單、自訂規則、受控規則集，以及每個網站的所有其他 WAF 設定。

假設您的應用程式閘道已套用全域原則。 然後，您將不同的原則套用至該應用程式閘道上的接聽程式。 該接聽程式的原則現在只會對該接聽程式生效。 應用程式閘道的全域原則仍然會套用至所有其他接聽程式，以及未獲派特定原則的路徑型規則。

## <a name="per-uri-policy"></a>每個 URI 原則

若要更進一步自訂 URI 層級，您可以將 WAF 原則與以路徑為基礎的規則建立關聯。 如果單一網站內有一些需要不同原則的頁面，您可以變更只會影響指定 URI 的 WAF 原則。 這可能適用于付款或登入頁面，或任何其他需要更明確 WAF 原則的 Uri，而不是 WAF 背後的其他網站。

如同每個網站的 WAF 原則，更明確的原則會覆寫較不特定的原則。 這表示 URL 路徑對應上的每個 URI 原則都會覆寫其上方的任何每個網站或全域 WAF 原則。

## <a name="example"></a>範例

假設您有三個網站： contoso.com、fabrikam.com 和 adatum.com 全都在相同的應用程式閘道後方。 您想要將 WAF 套用到這三個網站，但您需要使用 adatum.com 提高安全性，因為這是客戶造訪、流覽和購買產品的地方。

如有必要，您可以將全域原則套用至 WAF，並包含一些基本設定、排除專案或自訂規則，以避免封鎖流量的一些誤報。 在此情況下，不需要執行全域 SQL 插入式規則，因為 fabrikam.com 和 contoso.com 是靜態頁面，而且沒有 SQL 後端。 因此，您可以在全域原則中停用這些規則。

此全域原則適用于 contoso.com 和 fabrikam.com，但您必須更小心處理登入資訊和付款的處理。 您可以將每個網站的原則套用至 adatum 接聽程式，並讓 SQL 規則保持執行狀態。 也假設有一個 cookie 封鎖了某些流量，因此您可以建立該 cookie 的排除，以停止錯誤的肯定。 

Adatum.com/payments URI 是您需要小心的地方。 因此，請在該 URI 上套用另一個原則，並讓所有規則保持啟用狀態，並同時移除所有排除專案。

在此範例中，您有適用于兩個網站的全域原則。 您有一個適用于某個網站的每個網站原則，接著適用于一個特定路徑型規則的每個 URI 原則。 在此範例中，請參閱此處的 (插入連結（如果有的話）) 如何為對應的 PowerShell 建立每個網站和每個 URI 的原則。

## <a name="existing-waf-configurations"></a>現有的 WAF 設定

所有新的 Web 應用程式防火牆的 WAF 設定 (自訂規則、受控規則集設定、排除專案等等。 ) 存在於 WAF 原則中。 如果您有現有的 WAF，這些設定可能仍存在於您的 WAF 設定中。 如需移至新 WAF 原則的詳細資訊，請 [將 WAF 設定遷移至 WAF 原則](https://docs.microsoft.com/azure/web-application-firewall/ag/migrate-policy)。 


## <a name="next-steps"></a>後續步驟

- [使用 Azure PowerShell 建立每個網站和每個 URI 的原則](per-site-policies.md)。
