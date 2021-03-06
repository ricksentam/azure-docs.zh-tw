---
title: 使用 REST 來管理 ML 資源
titleSuffix: Azure Machine Learning
description: 如何使用 REST Api 來建立、執行和刪除 Azure ML 資源
author: lobrien
ms.author: laobri
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.date: 01/31/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: b733fbc44deefe46e3496e288ebad525346ef005
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91322303"
---
# <a name="create-run-and-delete-azure-ml-resources-using-rest"></a>使用 REST 建立、執行及刪除 Azure ML 資源



有數種方式可管理您的 Azure ML 資源。 您可以使用 [入口網站](https://portal.azure.com/)、 [命令列介面](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest&preserve-view=true)或 [Python SDK](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py&preserve-view=true)。 或者，您可以選擇 REST API。 REST API 會以標準方式使用 HTTP 動詞命令來建立、取出、更新和刪除資源。 REST API 適用于任何可發出 HTTP 要求的語言或工具。 REST 的簡單結構通常會讓它成為腳本環境及 MLOps 自動化的理想選擇。 

在本文中，您將學會如何：

> [!div class="checklist"]
> * 取出授權權杖
> * 使用服務主體驗證建立格式正確的 REST 要求
> * 使用 GET 要求取得 Azure ML 階層式資源的相關資訊
> * 使用 PUT 和 POST 要求來建立和修改資源
> * 使用刪除要求來清除資源 
> * 使用以金鑰為基礎的授權來評分已部署的模型

## <a name="prerequisites"></a>必要條件

- 您擁有系統管理許可權的 **Azure 訂** 用帳戶。 如果您沒有這類訂用帳戶，請試用[免費或付費的個人訂](https://aka.ms/AMLFree)用帳戶
- [Azure Machine Learning 工作區](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace)
- 系統管理 REST 要求使用服務主體驗證。 依照 [設定 Azure Machine Learning 資源和工作流程的驗證](https://docs.microsoft.com/azure/machine-learning/how-to-setup-authentication#set-up-service-principal-authentication) 中的步驟，在您的工作區中建立服務主體
- **捲曲**的公用程式。 [Windows 子系統 Linux 版](https://aka.ms/wslinstall/)或任何 UNIX 發佈都有提供**捲曲**的程式。 在 PowerShell 中，「 **捲曲** 」是 **WebRequest** 的別名，而且 `curl -d "key=val" -X POST uri` 會成為 `Invoke-WebRequest -Body "key=val" -Method POST -Uri uri` 。 

## <a name="retrieve-a-service-principal-authentication-token"></a>取得服務主體驗證權杖

系統管理 REST 要求會使用 OAuth2 隱含流程進行驗證。 此驗證流程會使用您訂用帳戶的服務主體所提供的權杖。 若要取得此權杖，您將需要：

- 您的租使用者識別碼 (識別訂用帳戶所屬的組織) 
- 您的用戶端識別碼 (將與建立的權杖相關聯) 
- 您應保護的用戶端密碼 () 

您應該要有這些值，以回應建立服務主體。 [Azure Machine Learning 資源和工作流程的設定驗證](https://docs.microsoft.com/azure/machine-learning/how-to-setup-authentication#set-up-service-principal-authentication)中會討論取得這些值。 如果您使用的是公司訂用帳戶，您可能沒有建立服務主體的許可權。 在這種情況下，您應該使用 [免費或付費的個人訂](https://aka.ms/AMLFree)用帳戶。

若要取出權杖：

1. 開啟終端機視窗
1. 在命令列中輸入下列程式碼
1. 以您自己的值取代 `{your-tenant-id}` 、 `{your-client-id}` 和 `{your-client-secret}` 。 在本文中，以大括弧括住的字串是您必須以適當的值取代的變數。
1. 執行命令

```bash
curl -X POST https://login.microsoftonline.com/{your-tenant-id}/oauth2/token \
-d "grant_type=client_credentials&resource=https%3A%2F%2Fmanagement.azure.com%2F&client_id={your-client-id}&client_secret={your-client-secret}" \
```

回應應提供一個小時的存取權杖：

```json
{
    "token_type": "Bearer",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1578523094",
    "not_before": "1578519194",
    "resource": "https://management.azure.com/",
    "access_token": "your-access-token"
}
```

請記下權杖，因為您將使用它來驗證所有後續的管理要求。 您將在所有要求中設定授權標頭來執行此作業：

```bash
curl -h "Authentication: Bearer {your-access-token}" ...more args...
```

請注意，值的開頭會是字串 "持有人"，包括單一空格，然後再新增權杖。

## <a name="get-a-list-of-resource-groups-associated-with-your-subscription"></a>取得與您的訂用帳戶相關聯的資源群組清單

若要取得與您的訂用帳戶相關聯的資源群組清單，請執行：

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups?api-version=2019-11-01 -H "Authorization:Bearer {your-access-token}"
```

在 Azure 中，會發佈許多 REST Api。 每個服務提供者會以自己的步調更新其 API，但不會中斷現有的程式。 服務提供者會使用 `api-version` 引數，以確保相容性。 自 `api-version` 變數會因服務而異。 例如，針對 Machine Learning 服務，目前的 API 版本為 `2019-11-01` 。 針對儲存體帳戶，它是 `2019-06-01` 。 針對金鑰保存庫，它是 `2019-09-01` 。 所有 REST 呼叫都應該將 `api-version` 引數設定為預期的值。 即使 API 持續演進，您也可以依賴指定版本的語法和語義。 如果您在沒有引數的情況下將要求傳送給提供者 `api-version` ，回應將會包含人們可讀取的支援值清單。 

上述呼叫會產生格式的壓縮 JSON 回應： 

```json
{
    "value": [
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG1",
            "name": "RG1",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "westus2",
            "properties": {
                "provisioningState": "Succeeded"
            }
        },
        {
            "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/RG2",
            "name": "RG2",
            "type": "Microsoft.Resources/resourceGroups",
            "location": "eastus",
            "properties": {
                "provisioningState": "Succeeded"
            }
        }
    ]
}
```


## <a name="drill-down-into-workspaces-and-their-resources"></a>深入瞭解工作區及其資源

若要在資源群組中取出工作區集合，請執行下列動作， `{your-subscription-id}` 並以、 `{your-resource-group}` 和取代 `{your-access-token}` ： 

```
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

同樣地，您將會收到 JSON 清單，這一次包含清單，其中每個專案都會詳細說明工作區：

```json
{
    "id": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourceGroups/DeepLearningResourceGroup/providers/Microsoft.MachineLearningServices/workspaces/my-workspace",
    "name": "my-workspace",
    "type": "Microsoft.MachineLearningServices/workspaces",
    "location": "centralus",
    "tags": {},
    "etag": null,
    "properties": {
        "friendlyName": "",
        "description": "",
        "creationTime": "2020-01-03T19:56:09.7588299+00:00",
        "storageAccount": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.storage/storageaccounts/myworkspace0275623111",
        "containerRegistry": null,
        "keyVault": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.keyvault/vaults/myworkspace2525649324",
        "applicationInsights": "/subscriptions/12345abc-abbc-1b2b-1234-57ab575a5a5a/resourcegroups/DeepLearningResourceGroup/providers/microsoft.insights/components/myworkspace2053523719",
        "hbiWorkspace": false,
        "workspaceId": "cba12345-abab-abab-abab-ababab123456",
        "subscriptionState": null,
        "subscriptionStatusChangeTimeStampUtc": null,
        "discoveryUrl": "https://centralus.experiments.azureml.net/discovery"
    },
    "identity": {
        "type": "SystemAssigned",
        "principalId": "abcdef1-abab-1234-1234-abababab123456",
        "tenantId": "1fedcba-abab-1234-1234-abababab123456"
    },
    "sku": {
        "name": "Basic",
        "tier": "Basic"
    }
}
```

若要使用工作區中的資源，您將從一般 **management.azure.com** 伺服器切換至工作區位置專屬的 REST API 伺服器。 請注意 `discoveryUrl` 上述 JSON 回應中的索引鍵值。 如果您收到該 URL，則會收到類似下列的回應：

```json
{
  "api": "https://centralus.api.azureml.ms",
  "catalog": "https://catalog.cortanaanalytics.com",
  "experimentation": "https://centralus.experiments.azureml.net",
  "gallery": "https://gallery.cortanaintelligence.com/project",
  "history": "https://centralus.experiments.azureml.net",
  "hyperdrive": "https://centralus.experiments.azureml.net",
  "labeling": "https://centralus.experiments.azureml.net",
  "modelmanagement": "https://centralus.modelmanagement.azureml.net",
  "pipelines": "https://centralus.aether.ms",
  "studiocoreservices": "https://centralus.studioservice.azureml.com"
}
```

回應的值 `api` 是您將用於其他要求之伺服器的 URL。 舉例來說，若要列出實驗，請傳送下列命令。 以 `regional-api-server` `api` 實例的回應 (值取代， `centralus.api.azureml.ms`) 。 也 `your-subscription-id` 請 `your-resource-group` `your-workspace-name` `your-access-token` 像往常一樣地取代、、和：

```bash
curl https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

同樣地，若要在您的工作區中取出已註冊的模型，請傳送：

```bash
curl https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

請注意，若要列出實驗，路徑的開頭為 `history/v1.0` 列出模型，路徑的開頭為 `modelmanagement/v1.0` 。 REST API 分成數個作業群組，每個都有不同的路徑。 

|區域|Path|
|-|-|
|Artifacts|/rest/api/azureml|
|資料存放區|/azure/machine-learning/how-to-access-data|
|超參數微調|hyperdrive/v1.0/|
|模型|modelmanagement/v1.0/|
|執行記錄|執行/v1.0/和歷程記錄/1.0 版/|

您可以使用的一般模式來探索 REST API：

|URL 元件|範例|
|-|-|
| https://| |
| 區域-api-伺服器/ | centralus.api.azureml.ms/ |
| 作業-路徑/ | 歷程記錄/v1.0/ |
| 訂用帳戶/{您的訂用帳戶 id}/ | 訂閱/abcde123-abab-abab-1234-0123456789abc/ |
| resourceGroups/{您的資源群組}/ | resourceGroups/MyResourceGroup/ |
| 提供者/作業-提供者/ | 提供者/Microsoft. MachineLearningServices/ |
| 提供者-資源-路徑/ | workspace/MLWorkspace/MyWorkspace/FirstExperiment/執行/1/ |
| 作業-端點/ | 構件/中繼資料/ |


## <a name="create-and-modify-resources-using-put-and-post-requests"></a>使用 PUT 和 POST 要求建立和修改資源

除了具有 GET 動詞的資源抓取之外，REST API 還支援建立定型、部署和監視 ML 解決方案所需的所有資源。 

定型和執行 ML 模型需要計算資源。 您可以使用下列內容列出工作區的計算資源： 

```bash
curl https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes?api-version=2019-11-01 \
-H "Authorization:Bearer {your-access-token}"
```

若要建立或覆寫指名的計算資源，您將使用 PUT 要求。 在下列中，除了現在熟悉的、、和的替代項之外，也會取代、、、、 `your-subscription-id` `your-resource-group` `your-workspace-name` `your-access-token` `your-compute-name` 和的值 `location` `vmSize` `vmPriority` `scaleSettings` `adminUserName` `adminUserPassword` 。 如 [Machine Learning Compute-建立或更新 SDK 參考](https://docs.microsoft.com/rest/api/azureml/workspacesandcomputes/machinelearningcompute/createorupdate)的參考中所指定，下列命令會建立專用的單一節點 Standard_D1 (基本的 CPU 計算資源) ，在30分鐘之後縮減：

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/computes/{your-compute-name}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{your-azure-location}",
    "properties": {
        "computeType": "AmlCompute",
        "properties": {
            "vmSize": "Standard_D1",
            "vmPriority": "Dedicated",
            "scaleSettings": {
                "maxNodeCount": 1,
                "minNodeCount": 0,
                "nodeIdleTimeBeforeScaleDown": "PT30M"
            }
        }
    },
    "userAccountCredentials": {
        "adminUserName": "{adminUserName}",
        "adminUserPassword": "{adminUserPassword}"
    }
}'
```

> [!Note]
> 在 Windows 終端機中，您可能必須在傳送 JSON 資料時，對雙引號符號進行換用。 也就是說，文字（例如） `"location"` 會變成 `\"location\"` 。 

成功的要求將會收到 `201 Created` 回應，但請注意，此回應只是表示已開始布建程式。 您必須輪詢 (，或使用入口網站) 來確認其成功完成。

### <a name="create-an-experimental-run"></a>建立實驗執行

若要在實驗內開始執行，您需要包含定型腳本和相關檔案的 ZIP 檔案夾，以及執行定義 JSON 檔案。 Zip 資料夾的根目錄中必須有 Python 專案檔。 例如，將簡單的 Python 程式（如下所示）壓縮成名為 **train.zip**的資料夾。

```python
# hello.py
# Entry file for run
print("Hello, REST!")
```

將此下一個程式碼片段儲存為 **definition.js開啟**。 確認「腳本」值符合您剛剛壓縮的 Python 檔案名。 確認「目標」值符合可用計算資源的名稱。 

```json
{
    "Configuration":{  
       "Script":"hello.py",
       "Arguments":[  
          "234"
       ],
       "SourceDirectoryDataStore":null,
       "Framework":"Python",
       "Communicator":"None",
       "Target":"cpu-compute",
       "MaxRunDurationSeconds":1200,
       "NodeCount":1,
       "Environment":{  
          "Python":{  
             "InterpreterPath":"python",
             "UserManagedDependencies":false,
             "CondaDependencies":{  
                "name":"project_environment",
                "dependencies":[  
                   "python=3.6.2",
                   {  
                      "pip":[  
                         "azureml-defaults"
                      ]
                   }
                ]
             }
          },
          "Docker":{  
             "BaseImage":"mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04"
          }
      },
       "History":{  
          "OutputCollection":true
       }
    }
}
```

使用下列內容將這些檔案張貼到伺服器 `multipart/form-data` ：

```bash
curl https://{regional-api-server}/execution/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-name}/startrun?api-version=2019-11-01 \
  -X POST \
  -H "Content-Type: multipart/form-data" \
  -H "Authorization:Bearer {your-access-token}" \
  -F projectZipFile=@train.zip \
  -F runDefinitionFile=@runDefinition.json
```

成功的 POST 要求將會產生 `200 OK` 狀態，其中包含所建立執行之識別碼的回應主體：

```json
{
  "runId": "my-first-experiment_1579642222877"
}
```

您可以使用應立即熟悉的 REST 規格模式來監視執行：

```bash
curl 'https://{regional-api-server}/history/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/experiments/{your-experiment-names}/runs/{your-run-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}'
```

### <a name="delete-resources-you-no-longer-need"></a>刪除您不再需要的資源

部分（但非全部）資源支援刪除動詞。 請先檢查 [API 參考](https://docs.microsoft.com/rest/api/azureml/) ，再對刪除使用案例的 REST API 進行認可。 舉例來說，若要刪除模型，您可以使用：

```bash
curl
  -X DELETE \
'https://{regional-api-server}/modelmanagement/v1.0/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/providers/Microsoft.MachineLearningServices/workspaces/{your-workspace-name}/models/{your-model-id}?api-version=2019-11-01' \
  -H 'Authorization:Bearer {your-access-token}' 
```

## <a name="use-rest-to-score-a-deployed-model"></a>使用 REST 對已部署的模型進行評分

雖然您可以部署模型，以便使用服務主體進行驗證，但大部分用戶端對應的部署都會使用以金鑰為基礎的驗證。 您可以在 Studio 的 [ **端點** ] 索引標籤內，于部署的頁面中找到適當的金鑰。 相同的位置會顯示您端點的評分 URI。 您模型的輸入必須模型化為名為的 JSON 陣列 `data` ：

```bash
curl 'https://{scoring-uri}' \
 -H 'Authorization:Bearer {your-key}' \
 -H 'Content-Type: application/json' \
  -d '{ "data" : [ {model-specific-data-structure} ] }
```

## <a name="create-a-workspace-using-rest"></a>使用 REST 建立工作區 

每個 Azure ML 工作區相依于其他四個 Azure 資源：已啟用系統管理的容器登錄、金鑰保存庫、Application Insights 資源和儲存體帳戶。 在這些資源存在之前，您無法建立工作區。 如需建立每個這類資源的詳細資訊，請參閱 REST API 參考。

若要建立工作區，請將類似下列的呼叫放入 `management.azure.com` 。 雖然此呼叫需要您設定大量變數，但它的結構與本文所討論的其他呼叫相同。 

```bash
curl -X PUT \
  'https://management.azure.com/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.MachineLearningServices/workspaces/{your-new-workspace-name}?api-version=2019-11-01' \
  -H 'Authorization: Bearer {your-access-token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "location": "{desired-region}",
    "properties": {
        "friendlyName" : "{your-workspace-friendly-name}",
        "description" : "{your-workspace-description}",
        "containerRegistry" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.ContainerRegistry/registries/{your-registry-name}",
        keyVault" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}\
/providers/Microsoft.Keyvault/vaults/{your-keyvault-name}",
        "applicationInsights" : "subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.insights/components/{your-application-insights-name}",
        "storageAccount" : "/subscriptions/{your-subscription-id}/resourceGroups/{your-resource-group}/\
providers/Microsoft.Storage/storageAccounts/{your-storage-account-name}"
    },
    "identity" : {
        "type" : "systemAssigned"
    }
}'
```

您應該會 `202 Accepted` 在傳回的標頭中收到回應，以及 `Location` URI。 您可以取得此 URI 以取得部署的相關資訊，包括有用的偵錯工具，如果您的其中一個相依資源發生問題 (例如，如果您忘記在容器登錄) 上啟用系統管理員存取權。 

## <a name="troubleshooting"></a>疑難排解

### <a name="resource-provider-errors"></a>資源提供者錯誤

[!INCLUDE [machine-learning-resource-provider](../../includes/machine-learning-resource-provider.md)]

### <a name="moving-the-workspace"></a>移動工作區

> [!WARNING]
> 不支援將 Azure Machine Learning 工作區移至不同的訂用帳戶，或將擁有的訂用帳戶移至新租用戶。 這麼做可能會導致錯誤。

### <a name="deleting-the-azure-container-registry"></a>刪除 Azure Container Registry

Azure Machine Learning 工作區會使用 Azure Container Registry (ACR) 進行某些作業。 它會在第一次需要 ACR 執行個體時自動建立一個。

[!INCLUDE [machine-learning-delete-acr](../../includes/machine-learning-delete-acr.md)]

## <a name="next-steps"></a>後續步驟

- 探索完整的 [AzureML REST API 參考](https://docs.microsoft.com/rest/api/azureml/)。
- 瞭解如何使用設計工具，透過 [設計工具預測汽車價格](https://docs.microsoft.com/azure/machine-learning/tutorial-designer-automobile-price-train-score)。
- 探索 [Jupyter 筆記本的 Azure Machine Learning](https://docs.microsoft.com/azure//machine-learning/samples-notebooks)。
