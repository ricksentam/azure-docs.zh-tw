---
title: Azure 服務匯流排常見問題集 (FAQ) | Microsoft Docs
description: 本文提供一些常見問題的解答 (常見問題) Azure 服務匯流排。
ms.topic: article
ms.date: 09/16/2020
ms.openlocfilehash: ec79b6988fdbc78dc4f45e504f84179e617589cc
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518750"
---
# <a name="azure-service-bus---frequently-asked-questions-faq"></a>Azure 服務匯流排-常見問題 (常見問題) 

本文討論 Microsoft Azure 服務匯流排的一些常見問題解集。 您也可以造訪 [Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)，以取得一般的 Azure 價格和支援資訊。


## <a name="general-questions-about-azure-service-bus"></a>關於 Azure 服務匯流排的一般問題
### <a name="what-is-azure-service-bus"></a>什麼是 Azure 服務匯流排？
[Azure 服務匯流排](service-bus-messaging-overview.md) 是非同步訊息雲端平臺，可讓您在分離的系統之間傳送資料。 Microsoft 以服務方式提供這項功能，這表示您不需要裝載您自己的硬體來使用它。

### <a name="what-is-a-service-bus-namespace"></a>什麼是服務匯流排命名空間？
[命名空間](service-bus-create-namespace-portal.md)提供範圍容器，可在應用程式內定址服務匯流排資源。 必須建立命名空間才能使用服務匯流排，而且這也是開始使用的第一個步驟。

### <a name="what-is-an-azure-service-bus-queue"></a>什麼是 Azure 服務匯流排佇列？
[服務匯流排佇列](service-bus-queues-topics-subscriptions.md)是訊息儲存所在的實體。 如果您有多個應用程式，或多個需要彼此通訊的分散式應用程式部分，佇列很有用。 佇列和配送中心的類似之處在於，兩者都會接收多個產品 (訊息)，再從該處送出。

### <a name="what-are-azure-service-bus-topics-and-subscriptions"></a>什麼是 Azure 服務匯流排主題和訂用帳戶？
主題可視覺化為佇列，而且在使用多個訂用帳戶時，主題會變成更豐富的訊息模型；基本上是一對多的通訊工具。 此發佈/訂閱模型 (或「pub/sub」**) 可讓應用程式將訊息傳送至具有多個訂用帳戶的主題，以便讓多個應用程式接收該訊息。

### <a name="what-is-a-partitioned-entity"></a>什麼是分割的實體？
傳統的佇列或主題由單一訊息代理程式處理並儲存在一個訊息存放區中。 只有在基本和標準通訊層、 [分割的佇列或主題](service-bus-partitioning.md) 中才支援，它是由多個訊息代理程式處理，並儲存在多個訊息存放區中。 這項功能表示分割佇列或主題的整體輸送量不會再受到單一訊息代理程式或訊息存放區的效能所限制。 此外，訊息存放區暫時中斷也不會轉譯分割的佇列或主題無法使用。

使用分割實體時無法確保順序。 若分割無法使用，您仍可從其他分割傳送及接收訊息。

 [Premium SKU](service-bus-premium-messaging.md) 不再支援分割的實體。 

### <a name="where-does-azure-service-bus-store-customer-data"></a><a name="in-region-data-residency"></a>Azure 服務匯流排儲存客戶資料的位置為何？
Azure 服務匯流排儲存客戶資料。 服務匯流排會自動將此資料儲存在單一區域中，因此這項服務會自動滿足區域資料落地需求（包括 [信任中心](https://azuredatacentermap.azurewebsites.net/)內指定的需求）。

### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>我需要在防火牆上開啟哪些連接埠？ 
您可以使用下列通訊協定搭配 Azure 服務匯流排來傳送和接收訊息：

- 進階訊息佇列通訊協定 (AMQP)
- 服務匯流排傳訊通訊協定 (SBMP)
- HTTP

請參閱下表，了解您需要開啟哪些輸出連接埠，以使用這些通訊協定與 Azure 事件中樞進行通訊。 

| 通訊協定 | 連接埠 | 詳細資料 | 
| -------- | ----- | ------- | 
| AMQP | 5671 與 5672 | 請參閱 [AMQP 通訊協定指南](service-bus-amqp-protocol-guide.md) | 
| SBMP | 9350至9354 | 查看 [連接模式](/dotnet/api/microsoft.servicebus.connectivitymode?view=azure-dotnet&preserve-view=true) |
| HTTP、HTTPS | 80、443 | 

### <a name="what-ip-addresses-do-i-need-to-add-to-allow-list"></a>我需要在允許清單中新增哪些 IP 位址？
若要尋找要新增至連線允許清單的正確 IP 位址，請遵循下列步驟：

1. 從命令提示字元執行下列命令： 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. 記下 `Non-authoritative answer` 中傳回的 IP 位址。 

如果您的命名空間使用 **區域冗余** ，您需要執行一些額外的步驟： 

1. 首先，在命名空間上執行 nslookup。

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. 記下 [非授權回答] 區段中的名稱，其採用下列其中一種格式： 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. 針對尾碼為 s1、s2 和 s3 的每個名稱執行 nslookup，以取得三個執行個體全都在三個可用性區域執行的 IP 位址。 

    > [!NOTE]
    > 命令傳回的 IP 位址 `nslookup` 不是靜態 ip 位址。 不過，它會保持不變，直到基礎部署被刪除或移至不同的叢集為止。

### <a name="where-can-i-find-the-ip-address-of-the-client-sendingreceiving-messages-tofrom-a-namespace"></a>哪裡可以找到用戶端在命名空間中傳送/接收訊息的 IP 位址？ 
我們不會記錄在您的命名空間中傳送或接收訊息之用戶端的 IP 位址。 重新產生金鑰，如此一來，所有現有的用戶端就無法驗證及審核 [azure 角色型存取控制 (AZURE RBAC) ](authenticate-application.md#azure-built-in-roles-for-azure-service-bus)) 設定，以確保只有允許的使用者或應用程式可以存取命名空間。 

如果您使用 **premium** 命名空間，請使用 [IP 篩選](service-bus-ip-filtering.md)、 [虛擬網路服務端點](service-bus-service-endpoints.md)和 [私人端點](private-link-service.md) 來限制對命名空間的存取。 

## <a name="best-practices"></a>最佳作法
### <a name="what-are-some-azure-service-bus-best-practices"></a>Azure 服務匯流排的最佳做法有哪些？
請參閱[使用服務匯流排的效能改進最佳作法][Best practices for performance improvements using Service Bus] - 這篇文章說明如何在交換訊息時將效能最佳化。

### <a name="what-should-i-know-before-creating-entities"></a>建立實體前的須知事項為何？
佇列和主題的下列屬性是不可變的。 當您布建實體時，請考慮這項限制，因為這些屬性無法在不建立新的取代實體的情況下修改。

* 資料分割
* 工作階段
* 重複偵測
* 快速實體

## <a name="pricing"></a>定價
本節提供服務匯流排價格結構的一些常見問題解答。

[服務匯流排定價與計費](https://azure.microsoft.com/pricing/details/service-bus/)一文說明服務匯流排中的計費計量。 如需服務匯流排價格選項的特定資訊，請參閱[服務匯流排價格詳細資料](https://azure.microsoft.com/pricing/details/service-bus/)。

您也可以造訪 [Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)，以取得一般 Azure 價格資訊。 

### <a name="how-do-you-charge-for-service-bus"></a>服務匯流排的收費方式為何？
如需服務匯流排價格的完整資訊，請參閱[服務匯流排價格詳細資料][Pricing overview]。 除了註明的價格，您還需支付您的應用程式佈建所在資料中心外部的輸出相關資料傳輸費用。

### <a name="what-usage-of-service-bus-is-subject-to-data-transfer-what-isnt"></a>何種服務匯流排用法需支付資料傳輸費用？ 什麼不是？
指定的 Azure 區域內的任何資料傳輸都是免費提供，以及任何輸入的資料傳輸。 區域外部的資料傳送需要出口流量費用，可以在[這裡](https://azure.microsoft.com/pricing/details/bandwidth/)找到。

### <a name="does-service-bus-charge-for-storage"></a>服務匯流排是否會收取儲存體費用？
不可以。 服務匯流排不會收取儲存體費用。 不過，配額會限制每個佇列/主題可保存的最大資料量。 請參閱下一個常見問題。

### <a name="i-have-a-service-bus-standard-namespace-why-do-i-see-charges-under-resource-group-system"></a>我有服務匯流排標準命名空間。 為什麼我會在資源群組 ' $system ' 下看見費用？
Azure 服務匯流排最近升級計費元件。 由於這項變更，如果您有服務匯流排標準命名空間，您可能會在資源群組 ' $system ' 下看到資源 '/subscriptions/<azure_subscription_id>/resourceGroups/$system/providers/Microsoft.ServiceBus/namespaces/$system ' 的明細專案。

這些費用代表每個已布建服務匯流排標準命名空間的 Azure 訂用帳戶的基本費用。 

請務必注意，這些費用不是新的，也就是它們也存在於先前的計費模型中。 唯一的變更是它們現在列在 [$system] 下。 這是因為新計費系統中的條件約束，會將訂用帳戶層級費用（而不是系結至特定資源）在「$system」資源識別碼下進行分組。

## <a name="quotas"></a>配額

如需服務匯流排限制和配額的清單，請參閱[服務匯流排配額概觀][Quotas overview]。

### <a name="how-to-handle-messages-of-size--1-mb"></a>如何處理大小超過 1 MB 的訊息？
「服務匯流排」訊息服務 (佇列和主題/訂用帳戶) 可讓應用程式傳送大小最大達 256 KB (標準層) 或 1 MB (進階層) 的訊息。 如果您正在處理大小超過 1 MB 的訊息，請使用 [此 blog 文章](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern)中所述的宣告檢查模式。

## <a name="troubleshooting"></a>疑難排解
### <a name="why-am-i-not-able-to-create-a-namespace-after-deleting-it-from-another-subscription"></a>在從另一個訂用帳戶中刪除命名空間之後，我為何無法重新建立它？ 
在您從訂用帳戶刪除命名空間之後，必須先等候 4 個小時的時間，才能在另一個訂用帳戶中以相同的名稱重新建立它。 否則，您可能會收到下列錯誤訊息︰`Namespace already exists`。 

### <a name="what-are-some-of-the-exceptions-generated-by-azure-service-bus-apis-and-their-suggested-actions"></a>Azure 服務匯流排 API 所產生的例外狀況有哪些，其建議的動作為何？
如需可能的服務匯流排例外狀況，請參閱[例外狀況概觀][Exceptions overview]。

### <a name="what-is-a-shared-access-signature-and-which-languages-support-generating-a-signature"></a>什麼是共用存取簽章，何種語言可支援產生簽章？
共用存取簽章是以 SHA-256 安全雜湊或 URI 為基礎的驗證機制。 如需如何在 Node.js、PHP、JAVA、Python 和 c # 中產生您自己的簽章的詳細資訊，請參閱 [共用存取][Shared Access Signatures] 簽章文章。

## <a name="subscription-and-namespace-management"></a>訂用帳戶和命名空間管理
### <a name="how-do-i-migrate-a-namespace-to-another-azure-subscription"></a>如何將命名空間移轉到另一個 Azure 訂用帳戶？

您可以使用 [Azure 入口網站](https://portal.azure.com)或 PowerShell 命令，將命名空間從一個 Azure 訂用帳戶移到另一個訂用帳戶。 若要執行此作業，命名空間必須已在使用中。 執行命令的使用者必須同時是來源和目標訂用帳戶上的系統管理員。

#### <a name="portal"></a>入口網站

若要使用 Azure 入口網站將「服務匯流排」移到另一個訂用帳戶，請依照[這裡](../azure-resource-manager/management/move-resource-group-and-subscription.md#use-the-portal)的指示操作。 

#### <a name="powershell"></a>PowerShell

下列 PowerShell 命令序列會將命名空間從一個 Azure 訂用帳戶移到另一個訂用帳戶。 若要執行這項作業，命名空間必須已經是作用中，而且執行 PowerShell 命令的使用者必須是來源與目標訂用帳戶的系統管理員。

```powershell
# Create a new resource group in target subscription
Select-AzSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzResourceGroup -Name 'targetRG' -Location 'East US'

# Move namespace from source subscription to target subscription
Select-AzSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzResource -DestinationResourceGroupName 'targetRG' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## <a name="next-steps"></a>後續步驟
若要深入了解服務匯流排，請參閱下列文章：

* [Azure 服務匯流排進階簡介 (部落格文章)](https://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
* [Azure 服務匯流排進階簡介 (Channel9)](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)
* [服務匯流排總覽](service-bus-messaging-overview.md)
* [開始使用服務匯流排佇列](service-bus-dotnet-get-started-with-queues.md)

[Best practices for performance improvements using Service Bus]: service-bus-performance-improvements.md
[Best practices for insulating applications against Service Bus outages and disasters]: service-bus-outages-disasters.md
[Pricing overview]: https://azure.microsoft.com/pricing/details/service-bus/
[Quotas overview]: service-bus-quotas.md
[Exceptions overview]: service-bus-messaging-exceptions.md
[Shared Access Signatures]: service-bus-sas.md
