---
title: 監視公用基礎負載平衡器的作業、事件和計數器
titleSuffix: Azure Load Balancer
description: 了解如何啟用 Basic Load Balancer 的警示事件和探查健康情況狀態記錄
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/05/2020
ms.author: allensu
ms.openlocfilehash: 42ec5a661bd7b42ba5de5bfa99b3898291cc60fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935597"
---
# <a name="azure-monitor-logs-for-public-basic-load-balancer"></a>公用基礎負載平衡器的 Azure 監視器記錄

您可以在 Azure 中使用不同類型的記錄來管理 Basic Load Balancer，並對其進行疑難排解。 透過入口網站可以存取其中一些記錄。 記錄可以串流至事件中樞或 Log Analytics 工作區。 所有記錄都可以從 Azure blob 儲存體解壓縮，並以不同的工具（例如 Excel 和 Power BI）來查看。  您可以從下列清單進一步了解不同類型的記錄。

* **活動記錄：** 您可以使用 [View 活動記錄來監視資源的動作](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit) ，以查看提交至您的 Azure 訂用帳戶 () 的所有活動及其狀態。 預設會啟用活動記錄，並可在 Azure 入口網站中查看。
* **警示事件記錄：** 您可以使用此記錄來檢視負載平衡器所引發的警示。 系統每五分鐘會收集一次負載平衡器的狀態。 只有在引發負載平衡器警示事件時，才會寫入此記錄檔。
* **健康狀態探查記錄︰** 您可以使用此記錄來檢視健康狀態探查所偵測到的問題，例如後端集區中因為健康狀態探查失敗而未從負載平衡器接收要求的執行個體數目。 健康狀態探查狀態發生變更時會寫入此記錄。

> [!IMPORTANT]
> **健康情況探查事件記錄檔目前無法運作，而且會列在 [Azure Load Balancer 的已知問題](whats-new.md#known-issues)中。** 記錄僅適用於在資源管理員部署模型中部署的資源。 您無法將記錄使用於傳統部署模型中的資源。 如需這些部署模型的詳細資訊，請參閱[了解 Resource Manager 部署和傳統部署](../azure-resource-manager/management/deployment-models.md)。

## <a name="enable-logging"></a>啟用記錄

每個 Resource Manager 資源都會自動啟用活動記錄功能。 啟用事件和健康情況探查記錄，以開始收集可透過這些記錄取得的資料。 使用下列步驟以啟用記錄功能。

登入 [Azure 入口網站](https://portal.azure.com)。 如果您還沒有負載平衡器，請先 [建立負載平衡器](https://docs.microsoft.com/azure/load-balancer/quickstart-create-basic-load-balancer-portal) 再繼續。

1. 在入口網站中，按一下 [ **資源群組**]。
2. 選取 **\<resource-group-name>** 負載平衡器所在的位置。
3. 選取負載平衡器。
4. 選取 [**活動記錄**  >  **診斷設定**]。
5. 在 [ **診斷設定** ] 窗格中，選取 [ **診斷設定**] 下的 [ **+ 新增診斷設定**]。
6. 在 [**診斷設定**建立] 窗格的 [**名稱**] 欄位中，輸入**myLBDiagnostics** 。
7. **診斷設定**有三個選項。  您可以選擇其中一個、兩個或三個，並針對您的需求進行設定：
   * **封存至儲存體帳戶**
   * **串流至事件中樞**
   * **傳送至 Log Analytics**

    ### <a name="archive-to-a-storage-account"></a>封存至儲存體帳戶
    您將需要已為此進程建立的儲存體帳戶。  若要建立儲存體帳戶，請參閱 [建立儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal)

    1. 選取 [封存 **至儲存體帳戶**] 旁的核取方塊。
    2. 選取 [ **設定** ] 以開啟 [ **選取儲存體帳戶** ] 窗格。
    3. 在下拉式清單方塊中，選取您的儲存體帳戶建立所在的 **訂** 用帳戶。
    4. 在下拉式清單方塊中，選取 **儲存體帳戶** 底下的儲存體帳戶名稱。
    5. 選取 [確定]。

    ### <a name="stream-to-an-event-hub"></a>串流至事件中樞
    您將需要已經為此進程建立的事件中樞。  若要建立事件中樞，請參閱 [快速入門：使用 Azure 入口網站建立事件中樞](https://docs.microsoft.com/azure/event-hubs/event-hubs-create)

    1. 選取 [**串流至事件中樞**] 旁的核取方塊
    2. 選取 [ **設定** ] 以開啟 [ **選取事件中樞** ] 窗格。
    3. 在下拉式清單方塊中選取您的事件中樞建立所在的 **訂** 用帳戶。
    4. 在下拉式清單方塊中**選取 [事件中樞命名空間**]。
    5. 在下拉式清單方塊中**選取 [事件中樞原則名稱**]。
    6. 選取 [確定]。

    ### <a name="send-to-log-analytics"></a>傳送至 Log Analytics
    您必須已經為此程式建立並設定 log analytics 工作區。  若要建立 Log Analytics 工作區，請參閱[在 Azure 入口網站中建立 Log analytics 工作區](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace)。

    1. 選取 [ **傳送至 Log Analytics**] 旁的核取方塊。
    2. 在下拉式清單方塊中選取您的 Log Analytics 工作區所在的 **訂** 用帳戶。
    3. 在下拉式清單方塊中選取 **Log Analytics 工作區** 。


8. 在 [**診斷設定**] 窗格中的 [**記錄**] 區段底下，選取 [兩者] 旁的核取方塊：
   * **LoadBalancerAlertEvent**
   * **LoadBalancerProbeHealthStatus**

9.  在 [**診斷設定**] 窗格中的 [計量 **] 區段底下**，選取下列核取方塊：
   * **AllMetrics**

11. 確認一切看起來正確，然後按一下 [建立**診斷設定**] 窗格頂端的 [**儲存**]。

## <a name="activity-log"></a>活動記錄檔

預設會產生活動記錄。 記錄會在 Azure 的 [事件記錄] 存放區中保留 90 天。 若要深入瞭解這些記錄，請閱讀 [View 活動記錄以監視資源的動作](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit) 文章。

## <a name="archive-to-storage-account-logs"></a>封存至儲存體客戶紀錄

### <a name="alert-event-log"></a>警示事件記錄檔

您必須對每一個負載平衡器進行啟用，才會產生此記錄檔。 事件會以 JSON 格式記錄，並儲存在您啟用記錄時所指定的儲存體帳戶中。 下列範例是事件。

```json
{
    "time": "2016-01-26T10:37:46.6024215Z",
    "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
    "category": "LoadBalancerAlertEvent",
    "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
    "operationName": "LoadBalancerProbeHealthStatus",
    "properties": {
        "eventName": "Resource Limits Hit",
        "eventDescription": "Ports exhausted",
        "eventProperties": {
            "public ip address": "40.117.227.32"
        }
    }
}
```

JSON 輸出會顯示 *事件事件名稱* 屬性，其將描述負載平衡器建立警示的原因。 在此情況下，產生的警示是因為來源 IP NAT 限制 (SNAT) 所造成的 TCP 埠耗盡。

### <a name="health-probe-log"></a>健全狀況探查記錄檔

如果您已如上所述對每一個負載平衡器進行啟用，才會產生此記錄檔。 資料會儲存在您啟用記錄時所指定的儲存體帳戶中。 系統會建立名為 'insights-logs-loadbalancerprobehealthstatus' 的容器，並記錄下列資料：

```json
{
    "records":[
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 1,
            "healthPercentage": 50.000000
        }
    },
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 0,
            "healthPercentage": 100.000000
        }
    }]
}
```

JSON 輸出在屬性欄位中顯示了探查健全狀況狀態的基本資訊。 [ *DipDownCount* ] 屬性會顯示後端上的實例總數，因為探查回應失敗，所以不會接收網路流量。

### <a name="view-and-analyze-the-activity-log"></a>檢視和分析活動記錄檔

您可以使用下列任何方法，檢視和分析活動記錄檔資料：

* **Azure 工具：** 透過 Azure PowerShell、Azure 命令列介面 (CLI) 、Azure REST API 或 Azure 入口網站，從活動記錄中取出資訊。 [稽核作業與資源管理員](../azure-resource-manager/management/view-activity-logs.md) 一文會詳述每個方法的逐步指示。
* **Power BI︰** 如果還沒有 [Power BI](https:// .microsoft.com/pricing) 帳戶，您可以免費試用。 使用 [Power BI 的 Azure 稽核記錄內容套件](https:// .microsoft.com/documentation/ -content-pack-azure-audit-logs)，您可以使用預先設定的儀表板來分析資料，或根據您的需求自訂檢視。

### <a name="view-and-analyze-the-health-probe-and-event-log"></a>檢視和分析健全狀況探查與事件記錄檔

連接到您的儲存體帳戶，並取得事件和健康情況探查記錄的 JSON 記錄專案。 下載 JSON 檔案後，您可以將它們轉換成 CSV，並在 Excel、Power BI 或任何其他資料視覺化檢視中查看。

> [!TIP]
> 如果您熟悉 Visual Studio 以及在 C# 中變更常數和變數值的基本概念，您可以使用 GitHub 所提供的[記錄檔轉換器工具 (英文)](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

## <a name="stream-to-an-event-hub"></a>串流至事件中樞
當診斷資訊串流至事件中樞時，它可以用於具有 Azure 監視器整合的協力廠商 SIEM 工具中的集中式記錄分析。 如需詳細資訊，請參閱 [將 Azure 監視資料串流至事件中樞](../azure-monitor/platform/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration)

## <a name="send-to-log-analytics"></a>傳送至 Log Analytics
Azure 中的資源可以將其診斷資訊直接傳送至 Log Analytics 工作區，其中可針對疑難排解和分析資訊來執行複雜的查詢。  如需詳細資訊，請參閱 Azure 監視器中的 [收集 Log Analytics 工作區中的 Azure 資源記錄](https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-workspace)

## <a name="next-steps"></a>接下來的步驟

[了解負載平衡器偵查](load-balancer-custom-probe-overview.md)
