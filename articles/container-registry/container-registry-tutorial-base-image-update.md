---
title: 教學課程 - 在基底映像更新時觸發映像建置
description: 在本教學課程中，您將了解如何設定 Azure Container Registry 工作，以在相同登錄中有基礎映像更新時自動觸發雲端中的容器映像建置。
ms.topic: tutorial
ms.date: 01/22/2020
ms.custom: seodec18, mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: f6dfc5b6cfe3312f2b3b4663ab5211cc60309eb6
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/27/2020
ms.locfileid: "92741110"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-an-azure-container-registry"></a>教學課程：在 Azure Container Registry 中更新基底映像時自動執行容器映像建置 

ACR 工作支援在容器的[基礎映像更新時](container-registry-tasks-base-images.md)自動建置容器映像，例如，當您在其中一個基礎映像中修補作業系統或應用程式架構時。 

在本教學課程中，您將了解如何建立 ACR 工作，以在將容器的基礎映像推送至相同登錄時，在雲端中觸發建置。 您也可以嘗試使用教學課程來建立 ACR 工作，以在基礎映像推送至[另一個 Azure 容器登錄](container-registry-tutorial-private-base-image-update.md)時觸發映像建置。 

本教學課程內容：

> [!div class="checklist"]
> * 建置基底映像
> * 在相同登錄中建立應用程式映像以追蹤基礎映像 
> * 更新基底映像以觸發應用程式映像工作
> * 顯示已觸發的工作
> * 確認更新的應用程式映像

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您想要在本機使用 Azure CLI，您必須安裝 Azure CLI **2.0.46** 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI][azure-cli]。

## <a name="prerequisites"></a>Prerequisites

### <a name="complete-the-previous-tutorials"></a>完成先前的教學課程

本教學課程假設您已完成本系列前兩個教學課程中的步驟，其間您完成了下列作業：

* 建立 Azure Container Registry
* 派生範例存放庫
* 複製範例存放庫
* 建立 GitHub 個人存取權杖

如果您尚未完成，請先完成下列教學課程，再繼續進行：

[使用 Azure Container Registry 工作在雲端中建置容器映像](container-registry-tutorial-quick-task.md)

[使用 Azure Container Registry 工作自動執行容器映像建置](container-registry-tutorial-build-task.md)

### <a name="configure-the-environment"></a>建立環境

請在這些殼層環境變數中填入您的環境適用的值。 此步驟並不是必要動作，但可簡化在本教學課程中執行多行 Azure CLI 命令的作業。 若未填入這些環境變數，則必須手動取代命令範例中出現的每個值。

[![內嵌啟動](https://shell.azure.com/images/launchcloudshell.png "啟動 Azure Cloud Shell")](https://shell.azure.com)

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```


### <a name="base-image-update-scenario"></a>基底映像更新案例

本教學課程會逐步引導您進行基礎映像更新案例，以在單一登錄中維護基礎映像和應用程式映像。 

[程式碼範例][code-sample]包含兩個 Dockerfile：一個應用程式映像，及其指定作為基底的映像。 在以下幾節中，您會建立一項 ACR 工作，以在基底映像的新版本推送至相同容器登錄時自動觸發應用程式映像的建置。

* [Dockerfile-app][dockerfile-app]：一個小型 Node.js Web 應用程式，會呈現一個靜態網頁，顯示它所依據的 Node.js 版本。 系統會模擬版本字串：它會顯示基底映像中定義的環境變數 `NODE_VERSION` 的內容。

* [Dockerfile-base][dockerfile-base]：`Dockerfile-app` 指定作為其基底的映像。 它本身會以[節點][base-node]映像為基礎，且包含 `NODE_VERSION` 環境變數。

在以下幾節中，您會建立工作、更新基底映像 Dockerfile 中的 `NODE_VERSION` 值，然後使用 ACR 工作來建置基底映像。 當 ACR 工作將新的基底映像推送至您的登錄時，它會自動觸發應用程式映像的建置。 您可以選擇性地在本機執行應用程式容器映像，以查看已建置的映像中不同的版本字串。

在本教學課程中，您的 ACR 工作會建置並推送在 Dockerfile 中指定的應用程式容器映像。 ACR 工作也可執行[多步驟工作](container-registry-tasks-multi-step.md)，使用 YAML 檔案來定義相關步驟，以建置、推送並選擇性地測試多個容器。

## <a name="build-the-base-image"></a>建置基底映像

首先請使用 [az acr build][az-acr-build]，透過 ACR 工作的快速工作  來建置基礎映像。 如本系列的[第一個教學課程](container-registry-tutorial-quick-task.md)所討論的，此程序不僅會建置映像，也會在建置成功時將映像推送至您的容器登錄。

```azurecli-interactive
az acr build --registry $ACR_NAME --image baseimages/node:9-alpine --file Dockerfile-base .
```

## <a name="create-a-task"></a>建立工作

接著，請使用 [az acr task create][az-acr-task-create] 建立工作：

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --arg REGISTRY_NAME=$ACR_NAME.azurecr.io \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT
```

此工作類似於[上一個教學課程](container-registry-tutorial-build-task.md)中建立的工作。 它會指示 ACR 工作在認可推送至 `--context` 所指定的存放庫時觸發映像建置。 在上一個教學課程中用來建置映像的 Dockerfile 會指定公用基底映像 (`FROM node:9-alpine`)，而此工作中的 Dockerfile [Dockerfile-app][dockerfile-app]，則是會指定相同登錄中的基底映像：

```dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:9-alpine
```

此組態可以讓稍後在本教學課程中的基底映像模擬架構修補程式變得容易。

## <a name="build-the-application-container"></a>建置應用程式容器

請使用 [az acr task run][az-acr-task-run] 手動觸發工作，並建置應用程式映像。 必須執行此步驟，工作才會在基礎映像上追蹤應用程式映像的相依性。

```azurecli-interactive
az acr task run --registry $ACR_NAME --name taskhelloworld
```

工作完成後，如果您想要完成下列選擇性步驟，請記下 **回合識別碼** (例如 "da6")。

### <a name="optional-run-application-container-locally"></a>選擇性：在本機執行應用程式容器

如果您在本機工作 (而不是在 Cloud Shell 中)，且您已安裝 Docker，請先執行容器以檢視在網頁瀏覽器中呈現的應用程式，再重建其基底映像。 如果您使用 Cloud Shell，請略過本節 (Cloud Shell 不支援 `az acr login` 或 `docker run`)。

首先，使用 [az acr login][az-acr-login] 驗證您的容器登錄：

```azurecli
az acr login --name $ACR_NAME
```

現在，請使用 `docker run` 在本機執行容器。 請將 **\<run-id\>** 取代為在上一個步驟的輸出中找到的回合識別碼 (例如 "da6")。 此範例將容器命名為 `myapp`，並且包含 `--rm` 參數以在您停止容器時將其移除。

```bash
docker run -d -p 8080:80 --name myapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

在瀏覽器中瀏覽至 `http://localhost:8080`，您應該會看到 Node.js 版本號碼呈現於網頁中，如下所示。 在後續步驟中，您可以在版本字串中加上 "a"，以變更版本。

![此螢幕擷取畫面顯示瀏覽器中呈現的範例應用程式。][base-update-01]

若要停止和移除容器，請執行下列命令：

```bash
docker stop myapp
```

## <a name="list-the-builds"></a>列出組建

接著，使用 [az acr task list-runs][az-acr-task-list-runs] 命令，列出 ACR 工作已為登錄完成的工作回合：

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

如果您已完成上一個教學課程 (且未刪除登錄)，您應該會看到如下的輸出。 請記下工作回合數目和最新的回合識別碼，以便在下一節更新基底映像後比較輸出。

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER     STARTED               DURATION
--------  --------------  ----------  ---------  ----------  --------------------  ----------
da6       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual      2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit  2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual      2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual      2018-09-17T22:29:59Z  00:00:57
```

## <a name="update-the-base-image"></a>更新基底映像

在此您將模擬基底映像中的架構修補程式。 請編輯 **Dockerfile-base** ，並在 `NODE_VERSION` 中定義的版本號碼後面加上 "a"：

```dockerfile
ENV NODE_VERSION 9.11.2a
```

執行快速工作，以建置經過修改的基底映像。 請記下輸出中的 **回合識別碼** 。

```azurecli-interactive
az acr build --registry $ACR_NAME --image baseimages/node:9-alpine --file Dockerfile-base .
```

當建置完成，且 ACR 工作已將新的基底映像推送至您的登錄後，它會觸發應用程式映像的建置。 您先前建立的工作可能需要一些時間才能觸發應用程式映像建置，因為它必須偵測最新建置並推送的基底映像。

## <a name="list-updated-build"></a>列出更新的組建

現在您已更新基底映像，接下來可以再次列出您的工作回合，將其與先前的清單比較。 若起初輸出並無差異，請定期執行命令，以查看清單中出現的新工作回合。

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

輸出大致如下。 最後執行之建置的 TRIGGER 應為 "Image Update"，表示工作是由基底映像的快速工作所起始。

```output
Run ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
da8       taskhelloworld  Linux       Succeeded  Image Update  2018-09-17T23:11:50Z  00:00:33
da7                       Linux       Succeeded  Manual        2018-09-17T23:11:27Z  00:00:35
da6       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T23:07:22Z  00:00:38
da5                       Linux       Succeeded  Manual        2018-09-17T23:06:33Z  00:00:31
da4       taskhelloworld  Linux       Succeeded  Git Commit    2018-09-17T23:03:45Z  00:00:44
da3       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:55:35Z  00:00:35
da2       taskhelloworld  Linux       Succeeded  Manual        2018-09-17T22:50:59Z  00:00:32
da1                       Linux       Succeeded  Manual        2018-09-17T22:29:59Z  00:00:57
```

如果您想要執行下列選擇性步驟以執行新建置的容器，並查看更新的版本號碼，請記下映像更新觸發之組建的 **RUN ID** 值 (在上述輸出中，其值為 "da8")。

### <a name="optional-run-newly-built-image"></a>選擇性：執行新建置的映像

如果您在本機工作 (而不是在 Cloud Shell 中)，且您已安裝 Docker，請在新的應用程式映像建置完成後執行該映像。 請將 `<run-id>` 取代為您在上一個步驟中取得的 RUN ID。 如果您使用 Cloud Shell，請略過本節 (Cloud Shell 不支援 `docker run`)。

```bash
docker run -d -p 8081:80 --name updatedapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

在瀏覽器中瀏覽至 http://localhost:8081 ，您應該會在網頁中看到更新的 Node.js 版本號碼 (附有 "a")：

![呈現在瀏覽器中的範例應用程式的螢幕擷取畫面][base-update-02]

務必留意的是，您是使用新的版本號碼更新 **基底** 映像，但最後建置的 **應用程式** 映像會顯示新版本。 ACR 工作會取用您對基底映像的變更，並自動重建您的應用程式映像。

若要停止和移除容器，請執行下列命令：

```bash
docker stop updatedapp
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用工作，在映像的基底映像有所更新時自動觸發容器映像建置。 現在，請繼續進行下一個教學課程，了解如何根據定義的排程觸發工作。

> [!div class="nextstepaction"]
> [依照排程執行工作](container-registry-tasks-scheduled.md)

<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-task-list-runs]: /cli/azure/acr
[az-acr-task]: /cli/azure/acr

<!-- IMAGES -->
[base-update-01]: ./media/container-registry-tutorial-base-image-update/base-update-01.png
[base-update-02]: ./media/container-registry-tutorial-base-image-update/base-update-02.png
