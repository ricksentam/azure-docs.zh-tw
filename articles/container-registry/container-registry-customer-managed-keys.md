---
title: 使用客戶管理的金鑰來加密登錄
description: 深入瞭解 Azure container registry 的待用加密，以及如何使用儲存在 Azure Key Vault 中客戶管理的金鑰來加密您的 Premium 登錄
ms.topic: article
ms.date: 09/30/2020
ms.custom: ''
ms.openlocfilehash: 6eaae5266277a6a65c7cecaa761b75e3a41ebe87
ms.sourcegitcommit: 541bb46e38ce21829a056da880c1619954678586
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2020
ms.locfileid: "91940662"
---
# <a name="encrypt-registry-using-a-customer-managed-key"></a>使用客戶管理的金鑰來加密登錄

當您在 Azure container registry 中儲存映像和其他成品時，Azure 會自動使用[服務管理金鑰](../security/fundamentals/encryption-models.md)來加密待用登錄內容。 您可以使用您在 Azure Key Vault (客戶管理的金鑰) 中建立和管理的金鑰，以額外的加密層補充預設加密。 本文將逐步引導您完成使用 Azure CLI 和 Azure 入口網站的步驟。

透過與 [Azure Key Vault](../key-vault/general/overview.md) 的整合，可支援使用客戶管理的金鑰進行伺服器端加密。 您可以建立自己的加密金鑰，然後將其儲存在金鑰保存庫中，或是使用 Azure Key Vault 的 API 來產生金鑰。 您也可以使用 Azure Key Vault 來稽核金鑰使用方式。

**進階**容器登錄服務層級中提供這項功能。 如需登錄服務層級和限制的相關資訊，請參閱 [Azure Container Registry 服務層級](container-registry-skus.md)。


## <a name="things-to-know"></a>須知事項

* 您目前只能在建立登錄時啟用客戶管理的金鑰。 啟用金鑰時，您可以設定 *使用者指派* 的受控識別來存取金鑰保存庫。
* 在登錄上使用客戶管理的金鑰啟用加密之後，您就無法停用加密。  
* 在以客戶管理的金鑰加密的登錄中，目前不支援[內容信任](container-registry-content-trust.md)。
* 在以客戶管理的金鑰加密的登錄中，執行 [ACR 工作](container-registry-tasks-overview.md)的記錄目前僅保留 24 小時。 如果您需要保留記錄較長的時間，請參閱[匯出和儲存工作執行記錄的指引](container-registry-tasks-logs.md#alternative-log-storage)。


> [!NOTE]
> 如果使用具有 [Key Vault 防火牆](../key-vault/general/network-security.md)的虛擬網路來限制存取 Azure key vault，則需要額外的設定步驟。 建立登錄並啟用客戶管理的金鑰之後，請使用登錄 *系統指派* 的受控識別來設定金鑰的存取權，並將登錄設定為略過 Key Vault 防火牆。 請先遵循本文中的步驟，使用客戶管理的金鑰來啟用加密，然後參閱本文稍後的「 [Advanced 案例的指引： Key Vault 防火牆](#advanced-scenario-key-vault-firewall) 」。

## <a name="prerequisites"></a>Prerequisites

若要使用本文中的 Azure CLI 步驟，請使用 Azure CLI 2.2.0 版或更新版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。

## <a name="enable-customer-managed-key---cli"></a>啟用客戶管理的金鑰 - CLI

### <a name="create-a-resource-group"></a>建立資源群組

如有需要，請執行 [az group create][az-group-create] 命令，以建立用來建立金鑰保存庫、容器登錄和其他必要資源的資源群組。

```azurecli
az group create --name <resource-group-name> --location <location>
```

### <a name="create-a-user-assigned-managed-identity"></a>建立使用者指派的受控識別

使用 [az identity create][az-identity-create] 命令，建立使用者指派的 [Azure 資源受控識別](../active-directory/managed-identities-azure-resources/overview.md)。 您登錄將使用此身分識別來存取 Key Vault 服務。

```azurecli
az identity create \
  --resource-group <resource-group-name> \
  --name <managed-identity-name>
```

在命令輸出中，記下下列值：`id` 和 `principalId`。 您在稍後的步驟中需要這些值，才能設定金鑰保存庫的登錄存取權。

```JSON
{
  "clientId": "xxxx2bac-xxxx-xxxx-xxxx-192cxxxx6273",
  "clientSecretUrl": "https://control-eastus.identity.azure.net/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myidentityname/credentials?tid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&oid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&aid=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myresourcegroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myresourcegroup",
  "location": "eastus",
  "name": "myidentityname",
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "resourceGroup": "myresourcegroup",
  "tags": {},
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

為了方便起見，請將這些值儲存在環境變數中：

```azurecli
identityID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'id' --output tsv)

identityPrincipalID=$(az identity show --resource-group <resource-group-name> --name <managed-identity-name> --query 'principalId' --output tsv)
```

### <a name="create-a-key-vault"></a>建立金鑰保存庫

使用 [az keyvault create][az-keyvault-create] 建立金鑰保存庫，以儲存用於登錄加密的客戶管理金鑰。

若要防止意外刪除金鑰或金鑰保存庫所造成的資料遺失，請啟用下列設定：虛 **刪除** 和 **清除保護**。 下列範例包含這些設定的參數：

```azurecli
az keyvault create --name <key-vault-name> \
  --resource-group <resource-group-name> \
  --enable-soft-delete \
  --enable-purge-protection
```

> [!NOTE]
> 從 Azure CLI 2.2 版， `az keyvault create` 預設會啟用虛刪除。

若要在稍後的步驟中使用，請取得金鑰保存庫的資源識別碼：

```azurecli
keyvaultID=$(az keyvault show --resource-group <resource-group-name> --name <key-vault-name> --query 'id' --output tsv)
```

### <a name="enable-key-vault-access"></a>啟用金鑰保存庫存取

設定金鑰保存庫的原則，讓身分識別可以存取。 在下列 [az keyvault set-policy][az-keyvault-set-policy] 命令中，您會傳遞先前所建立的受控識別主體識別碼，並儲存在環境變數中。 將金鑰權限設定為 **get**、**unwrapKey** 和 **wrapKey**。  

```azurecli
az keyvault set-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID \
  --key-permissions get unwrapKey wrapKey
```

或者，您也可以使用 [AZURE RBAC Key Vault 的](../key-vault/general/rbac-guide.md) (預覽版) 將許可權指派給身分識別，以存取金鑰保存庫。 例如，使用 [az role assign create](/cli/azure/role/assignment?view=azure-cli-latest#az-role-assignment-create) 命令將 Key Vault 加密服務加密角色指派給身分識別：

```azurecli 
az role assignment create --assignee $identityPrincipalID \
  --role "Key Vault Crypto Service Encryption (preview)" \
  --scope $keyvaultID
```

### <a name="create-key-and-get-key-id"></a>建立金鑰並取得金鑰識別元

執行 [az keyvault key create][az-keyvault-key-create] 命令來建立金鑰保存庫中的金鑰。

```azurecli
az keyvault key create \
  --name <key-name> \
  --vault-name <key-vault-name>
```

在命令輸出中，記下下列金鑰識別元：`kid`。 您在下一個步驟將會用到：

```JSON
[...]
  "key": {
    "crv": null,
    "d": null,
    "dp": null,
    "dq": null,
    "e": "AQAB",
    "k": null,
    "keyOps": [
      "encrypt",
      "decrypt",
      "sign",
      "verify",
      "wrapKey",
      "unwrapKey"
    ],
    "kid": "https://mykeyvault.vault.azure.net/keys/mykey/xxxxxxxxxxxxxxxxxxxxxxxx",
    "kty": "RSA",
[...]
```
為了方便起見，請將此值儲存在環境變數中：

```azurecli
keyID=$(az keyvault key show \
  --name <keyname> \
  --vault-name <key-vault-name> \
  --query 'key.kid' --output tsv)
```

### <a name="create-a-registry-with-customer-managed-key"></a>使用客戶管理的金鑰建立登錄

執行 [az acr create][az-acr-create] 命令以在進階服務層級中建立登錄，並啟用客戶管理的金鑰。 傳遞先前儲存在環境變數中的受控識別主體識別碼和金鑰識別元：

```azurecli
az acr create \
  --resource-group <resource-group-name> \
  --name <container-registry-name> \
  --identity $identityID \
  --key-encryption-key $keyID \
  --sku Premium
```

### <a name="show-encryption-status"></a>顯示加密狀態

若要顯示是否已啟用使用客戶管理金鑰的登錄加密，請執行 [az acr encryption show][az-acr-encryption-show] 命令：

```azurecli
az acr encryption show --name <registry-name>
```

輸出會類似：

```console
{
  "keyVaultProperties": {
    "identity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "keyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789...",
    "versionedKeyIdentifier": "https://myvault.vault.azure.net/keys/myresourcegroup/abcdefg123456789..."
  },
  "status": "enabled"
}
```

## <a name="enable-customer-managed-key---portal"></a>啟用客戶管理的金鑰 - 入口網站

### <a name="create-a-managed-identity"></a>建立受控識別

在 Azure 入口網站中建立使用者指派的 [Azure 資源受控識別](../active-directory/managed-identities-azure-resources/overview.md)。 如需步驟指示，請參閱[建立使用者指派的身分識別](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md#create-a-user-assigned-managed-identity)。

您會在稍後的步驟中使用身分識別的名稱。

![在 Azure 入口網站中建立使用者指派的受控識別](./media/container-registry-customer-managed-keys/create-managed-identity.png)

### <a name="create-a-key-vault"></a>建立金鑰保存庫

如需建立金鑰保存庫的步驟，請參閱[快速入門：使用 Azure 入口網站從 Azure Key Vault 設定及擷取祕密](../key-vault/secrets/quick-create-portal.md)。

為客戶管理的金鑰建立金鑰保存庫時，請在 [基本概念] 索引標籤中，啟用下列保護設定：**虛刪除**和**清除保護**。 這些設定可防止意外刪除金鑰或金鑰保存庫而造成的資料遺失。

![在 Azure 入口網站中建立金鑰保存庫](./media/container-registry-customer-managed-keys/create-key-vault.png)

### <a name="enable-key-vault-access"></a>啟用金鑰保存庫存取

設定金鑰保存庫的原則，讓身分識別可以存取。

1. 瀏覽至您的金鑰保存庫。
1. 選取 [設定] > [存取原則] > [+新增存取原則]。
1. 選取 [金鑰權限]，並選取 [取得]、[將金鑰取消換行] 和 [包裝金鑰]。
1. 選取 [選取主體]，然後選取使用者指派受控識別的資源名稱。  
1. 選取 [新增]，然後選取 [儲存]。

![建立金鑰保存庫存取原則](./media/container-registry-customer-managed-keys/add-key-vault-access-policy.png)

 或者，您也可以使用 [AZURE RBAC Key Vault 的](../key-vault/general/rbac-guide.md) (預覽版) 將許可權指派給身分識別，以存取金鑰保存庫。 例如，將 Key Vault 加密服務加密角色指派給身分識別。

1. 瀏覽至您的金鑰保存庫。
1. 選取 [**存取控制] (IAM) **  >  **+**[新增  >  **新增角色指派**]。
1. 在 [ **新增角色指派** ] 視窗中：
    1. 選取 **Key Vault 加密服務加密 (預覽) ** 角色。 
    1. 將存取權指派給 **使用者指派的受控識別**。
    1. 選取使用者指派的受控識別的資源名稱，然後選取 [ **儲存**]。

### <a name="create-key"></a>建立金鑰

1. 瀏覽至您的金鑰保存庫。
1. 選取 [設定] > [金鑰]。
1. 選取 [+產生/匯入]，然後輸入唯一的金鑰名稱。
1. 接受其餘預設值並選取 [建立]。
1. 建立之後，請選取金鑰，並記下目前的金鑰版本。

### <a name="create-azure-container-registry"></a>建立 Azure Container Registry

1. 選取 [建立資源] > [容器] > [容器登錄]。
1. 在 [基本概念] 索引標籤中，選取或建立資源群組，然後輸入登錄名稱。 在 **SKU** 中，選取 [進階]。
1. 在 [加密] 索引標籤的**客戶管理的金鑰**中，選取 [已啟用]。
1. 在**身分識別**中，選取您建立的受控識別。
1. 在**加密**中，選取 [從 Key Vault 選取]。
1. 在 [從 Azure Key Vault 選取金鑰] 視窗中，選取您在上一節中建立的金鑰保存庫、金鑰及和版本。
1. 在 [加密] 索引標籤中，選取 [檢閱+建立]。
1. 選取 [建立] 以部署登錄執行個體。

![在 Azure 入口網站中建立容器登錄](./media/container-registry-customer-managed-keys/create-encrypted-registry.png)

若要在入口網站中查看登錄的加密狀態，請瀏覽至您的登錄。 在 [設定] 下，選取 [加密]。

## <a name="enable-customer-managed-key---template"></a>啟用客戶管理的金鑰 - 範本

您也可以使用 Resource Manager 範本來建立登錄，並使用客戶管理的金鑰來啟用加密。

下列範本會建立新的容器登錄和使用者指派的受控識別。 將下列內容複寫到新的檔案，並使用檔案名 (例如 `CMKtemplate.json`) 儲存檔案。

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vault_name": {
      "defaultValue": "",
      "type": "String"
    },
    "registry_name": {
      "defaultValue": "",
      "type": "String"
    },
    "identity_name": {
      "defaultValue": "",
      "type": "String"
    },
    "kek_id": {
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.ContainerRegistry/registries",
      "apiVersion": "2019-12-01-preview",
      "name": "[parameters('registry_name')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Premium",
        "tier": "Premium"
      },
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]": {}
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "adminUserEnabled": false,
        "encryption": {
          "status": "enabled",
          "keyVaultProperties": {
            "identity": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').clientId]",
            "KeyIdentifier": "[parameters('kek_id')]"
          }
        },
        "networkRuleSet": {
          "defaultAction": "Allow",
          "virtualNetworkRules": [],
          "ipRules": []
        },
        "policies": {
          "quarantinePolicy": {
            "status": "disabled"
          },
          "trustPolicy": {
            "type": "Notary",
            "status": "disabled"
          },
          "retentionPolicy": {
            "days": 7,
            "status": "disabled"
          }
        }
      }
    },
    {
      "type": "Microsoft.KeyVault/vaults/accessPolicies",
      "apiVersion": "2018-02-14",
      "name": "[concat(parameters('vault_name'), '/add')]",
      "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name'))]"
      ],
      "properties": {
        "accessPolicies": [
          {
            "tenantId": "[subscription().tenantId]",
            "objectId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', parameters('identity_name')), '2018-11-30').principalId]",
            "permissions": {
              "keys": [
                "get",
                "unwrapKey",
                "wrapKey"
              ]
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "apiVersion": "2018-11-30",
      "name": "[parameters('identity_name')]",
      "location": "[resourceGroup().location]"
    }
  ]
}

```

依照之前章節中的步驟來建立下列資源：

* 金鑰保存庫，依名稱識別
* 金鑰保存庫，依金鑰識別元識別

執行下列 [az group deployment create][az-group-deployment-create] 命令，以使用先前的範本檔案來建立登錄。 在指示的位置，提供新的登錄名稱和受控識別名稱，以及您所建立的金鑰保存庫名稱和金鑰識別元。

```bash
az group deployment create \
  --resource-group <resource-group-name> \
  --template-file CMKtemplate.json \
  --parameters \
    registry_name=<registry-name> \
    identity_name=<managed-identity> \
    vault_name=<key-vault-name> \
    kek_id=<key-vault-key-id>
```

### <a name="show-encryption-status"></a>顯示加密狀態

若要顯示登錄加密的狀態，請執行 [az acr encryption show][az-acr-encryption-show] 命令：

```azurecli
az acr encryption show --name <registry-name>
```

## <a name="use-the-registry"></a>使用登錄

在登錄中啟用客戶管理的金鑰後，您可以執行在登錄中執行且未使用客戶管理的金鑰進行加密的相同登錄作業。 例如，您可以使用登錄進行驗證，並推送 Docker 映像。 請參閱[推送和提取映像](container-registry-get-started-docker-cli.md)中的範例命令。

## <a name="rotate-key"></a>輪替金鑰

根據您的合規性政策，輪替用於登錄加密的客戶管理金鑰。 建立新的金鑰，或更新金鑰版本，然後使用金鑰更新登錄來加密資料。 您也可以使用 Azure CLI 或在入口網站中執行這些步驟。

輪替金鑰時，通常會指定建立登錄時使用的相同身分識別。 (選用) 針對金鑰存取設定新的使用者指派身分識別，或啟用並指定登錄系統指派的身分識別。

> [!NOTE]
> 確定已針對您為金鑰存取所設定的身分識別設定必要的 [金鑰保存庫存取權](#enable-key-vault-access) 。

### <a name="azure-cli"></a>Azure CLI

使用 [az keyvault key][az-keyvault-key] 命令來建立或管理您的金鑰保存庫金鑰。 例如，若要建立新的金鑰版本或金鑰，請執行 [az keyvault key create][az-keyvault-key-create] 命令：

```azurecli
# Create new version of existing key
az keyvault key create \
  –-name <key-name> \
  --vault-name <key-vault-name>

# Create new key
az keyvault key create \
  –-name <new-key-name> \
  --vault-name <key-vault-name>
```

然後執行 [az acr encryption rotate-key][az-acr-encryption-rotate-key] 命令，傳遞新的金鑰識別元和要設定的身分識別：

```azurecli
# Rotate key and use user-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity <principal-id-user-assigned-identity>

# Rotate key and use system-assigned identity
az acr encryption rotate-key \
  --name <registry-name> \
  --key-encryption-key <new-key-id> \
  --identity [system]
```

### <a name="portal"></a>入口網站

使用登錄的**加密**設定來更新用於客戶管理金鑰的金鑰版本、金鑰、金鑰保存庫或身分識別設定。

例如，若要產生並設定新的金鑰版本：

1. 在入口網站中，瀏覽到您的登錄。
1. 在 [設定]下，選取 [加密] > [變更金鑰]。
1. 選取 [選取金鑰]

    ![在 Azure 入口網站中輪替金鑰](./media/container-registry-customer-managed-keys/rotate-key.png)
1. 在 [從 Azure Key Vault 中選取金鑰] 視窗中選取您先前設定的金鑰保存庫和金鑰，然後在 [版本] 中，選取 [建立新的]。
1. 在 [建立金鑰] 視窗中，選取 [產生]，然後選取 [建立]。
1. 完成金鑰選取，然後選取 [儲存]。

## <a name="revoke-key"></a>撤銷金鑰

藉由變更金鑰保存庫的存取原則或許可權，或藉由刪除金鑰，撤銷客戶管理的加密金鑰。 例如，使用 [az keyvault delete-policy][az-keyvault-delete-policy] 命令來變更登錄所使用之受控識別的存取原則：

```azurecli
az keyvault delete-policy \
  --resource-group <resource-group-name> \
  --name <key-vault-name> \
  --object-id $identityPrincipalID
```

撤銷金鑰會有效封鎖對所有登錄資料的存取，因為登錄無法存取加密金鑰。 如果已啟用金鑰的存取權，或已還原刪除的金鑰，您的登錄將會挑選金鑰，讓您可以再次存取加密的登錄資料。

## <a name="advanced-scenario-key-vault-firewall"></a>Advanced 案例： Key Vault 防火牆

如果您的 Azure 金鑰保存庫部署在具有 Key Vault 防火牆的虛擬網路中，請在您的登錄中啟用客戶管理的金鑰加密之後，執行下列額外步驟。

1. 將登錄加密設定為使用登錄系統指派的身分識別
1. 啟用登錄以略過 Key Vault 防火牆
1. 輪替客戶管理的金鑰

### <a name="configure-system-assigned-identity"></a>設定系統指派的身分識別

您可以設定登錄系統指派的受控識別，以存取加密金鑰的金鑰保存庫。 如果您不熟悉 Azure 資源的各種受控識別，請參閱[概觀](../active-directory/managed-identities-azure-resources/overview.md)。

若要在入口網站中啟用登錄的系統指派身分識別：

1. 在入口網站中，瀏覽到您的登錄。
1. 選取 [設定] >  [識別]。
1. 在 [系統指派] 下，將 [狀態] 設定為 [開啟]。 選取 [儲存]。
1. 複製身分識別的**物件識別碼**。

若要將 Key Vault 存取權授與該身分識別：

1. 瀏覽至您的金鑰保存庫。
1. 選取 [設定] > [存取原則] > [+新增存取原則]。
1. 選取 [金鑰權限]，並選取 [取得]、[將金鑰取消換行] 和 [包裝金鑰]。
1. 選取 [選取主體]，然後搜尋系統指派的受控識別物件識別碼，或登錄名稱。  
1. 選取 [新增]，然後選取 [儲存]。

若要更新登錄的加密設定以使用身分識別：

1. 在入口網站中，瀏覽到您的登錄。
1. 在 [設定]下，選取 [加密] > [變更金鑰]。
1. 在**身分識別**中，選取 [系統指派]，然後選取 [儲存]。

### <a name="enable-key-vault-bypass"></a>啟用金鑰保存庫略過

若要存取 Key Vault 防火牆設定的金鑰保存庫，登錄必須略過防火牆。 確認金鑰保存庫已設定為允許任何 [受信任的服務](../key-vault/general/overview-vnet-service-endpoints.md#trusted-services)存取。 Azure Container Registry 是其中一項受信任的服務。

1. 在入口網站中，流覽至您的金鑰保存庫。
1. 選取 [**設定**  >  **網路**]。
1. 確認、更新或新增虛擬網路設定。 如需詳細資訊，請參閱[設定 Azure Key Vault 防火牆和虛擬網路](../key-vault/general/network-security.md)。
1. 在 [ **允許 Microsoft 信任的服務略過此防火牆**] 中，選取 **[是]**。 

### <a name="rotate-the-customer-managed-key"></a>輪替客戶管理的金鑰

完成上述步驟之後，請將金鑰輪替至防火牆後面的金鑰保存庫中的新金鑰。 如需相關步驟，請參閱本文中的 [旋轉金鑰](#rotate-key) 。

## <a name="troubleshoot"></a>疑難排解

### <a name="removing-user-assigned-identity"></a>正在移除使用者指派的身分識別

如果您嘗試從用於加密的登錄中移除使用者指派的身分識別，您可能會看到類似下列的錯誤訊息：
 
```
Azure resource '/subscriptions/xxxx/resourcegroups/myGroup/providers/Microsoft.ContainerRegistry/registries/myRegistry' does not have access to identity 'xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx' Try forcibly adding the identity to the registry <registry name>. For more information on bring your own key, please visit 'https://aka.ms/acr/cmk'.
```
 
您也將無法變更加密金鑰)  (輪替。 如果發生此問題，請先使用錯誤訊息中顯示的 GUID 重新指派身分識別。 例如：

```azurecli
az acr identity assign -n myRegistry --identities xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx
```
        
然後，在變更金鑰並指派不同的身分識別之後，您可以移除原始使用者指派的身分識別。

## <a name="next-steps"></a>後續步驟

* 深入了解 [Azure 中的待用加密](../security/fundamentals/encryption-atrest.md)。
* 深入了解存取原則，以及如何[安全地存取金鑰保存庫](../key-vault/general/secure-your-key-vault.md)。


<!-- LINKS - external -->

<!-- LINKS - internal -->

[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-show]: /cli/azure/feature#az-feature-show
[az-group-create]: /cli/azure/group#az-group-create
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
[az-keyvault-create]: /cli/azure/keyvault#az-keyvault-create
[az-keyvault-key-create]: /cli/azure/keyvault/key#az-keyvault-key-create
[az-keyvault-key]: /cli/azure/keyvault/key
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-keyvault-delete-policy]: /cli/azure/keyvault#az-keyvault-delete-policy
[az-resource-show]: /cli/azure/resource#az-resource-show
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-encryption-rotate-key]: /cli/azure/acr/encryption#az-acr-encryption-rotate-key
[az-acr-encryption-show]: /cli/azure/acr/encryption#az-acr-encryption-show
