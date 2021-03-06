---
title: 設定 Linux Python 應用程式
description: 了解如何使用 Azure 入口網站和 Azure CLI，設定 Web 應用程式執行所在的 Python 容器。
ms.topic: quickstart
ms.date: 10/06/2020
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18, devx-track-python, devx-track-azurecli
ms.openlocfilehash: 935baef209811146d0b60f4fc02986818fd103a7
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/27/2020
ms.locfileid: "92743796"
---
# <a name="configure-a-linux-python-app-for-azure-app-service"></a>設定適用於 Azure App Service 的 Linux Python 應用程式

本文說明 [Azure App Service](overview.md) 會如何執行 Python 應用程式，以及要如何在需要時自訂 App Service 的行為。 您必須使用所有必要的 [pip](https://pypi.org/project/pip/) 模組來部署 Python 應用程式。

在您部署 [Git 存放庫](deploy-local-git.md)或 [zip 套件](deploy-zip.md)時，App Service 部署引擎會自動啟動虛擬環境，並為您執行 `pip install -r requirements.txt`。

本指南為在 App Service 中使用內建 Linux 容器的 Python 開發人員提供了重要概念和指示。 如果您從未使用過 Azure App Service，請先遵循 [Python 快速入門](quickstart-python.md)和 [Python with PostgreSQL 教學課程](tutorial-python-postgresql-app.md)。

您可以使用 [Azure 入口網站](https://portal.azure.com)或 Azure CLI 進行設定：

- **Azure 入口網站** ：使用應用程式的 [設定] > [組態] 頁面，如 [在 Azure 入口網站中設定 App Service 應用程式](configure-common.md)所述。

- **Azure CLI** ：您有兩個選項。

    - 在 [Azure Cloud Shell](../cloud-shell/overview.md) 中執行命令時，您可以使用程式碼區塊右上角的 [試用] 按鈕來開啟。
    - 藉由安裝最新版的 [Azure CLI](/cli/azure/install-azure-cli)，在本機執行命令，然後使用 [az login](/cli/azure/reference-index#az-login) 來登入 Azure。
    
> [!NOTE]
> Linux 目前是在 App Service 中執行 Python 應用程式的建議選項。 如需 Windows 選項的資訊，請參閱 [Windows風格 App Service 的 Python](/visualstudio/python/managing-python-on-azure-app-service)。

## <a name="configure-python-version"></a>設定 Python 版本

- **Azure 入口網站** ：使用 [組態] 頁面上的 [一般設定] 索引標籤，如針對 Linux 容器 [設定一般設定](configure-common.md#configure-general-settings)所述。

- **Azure CLI** ：

    -  使用 [az webapp config show](/cli/azure/webapp/config#az_webapp_config_show) 顯示目前的 Python 版本：
    
        ```azurecli-interactive
        az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
        ```
        
        以適合您 Web 應用程式的名稱取代 `<resource-group-name>` 和 `<app-name>`。
    
    - 使用 [az webapp config set](/cli/azure/webapp/config#az_webapp_config_set) 設定 Python 版本
        
        ```azurecli-interactive
        az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "PYTHON|3.7"
        ```
    
    - 使用 [az webapp list-runtimes](/cli/azure/webapp#az_webapp_list_runtimes)顯示 Azure App Service 中支援的所有 Python 版本：
    
        ```azurecli-interactive
        az webapp list-runtimes --linux | grep PYTHON
        ```
    
您可以改為建置您自己的容器映像，以執行不支援的 Python 版本。 如需詳細資訊，請參閱[使用自訂 Docker 映像](tutorial-custom-container.md?pivots=container-linux)。

<!-- <a> element here to preserve external links-->
<a name="access-environment-variables"></a>

## <a name="customize-build-automation"></a>自訂組建自動化

當您使用 Git 或 zip 套件部署應用程式時，App Service 的建置系統 (稱為 Oryx) 會執行下列步驟：

1. 如果自訂建置前指令碼由 `PRE_BUILD_COMMAND` 設定指定，請加以執行。
1. 執行 `pip install -r requirements.txt`。 requirements.txt 檔案必須存在於專案的根資料夾中。 否則，建至程序會報告錯誤：「找不到 setup.py 或 requirements.txt;未執行 pip 安裝。」
1. 如果在存放庫的根目錄中找到 *manage.py* (表示 Django 應用程式)，請執行 *manage.py collectstatic* 。 不過，如果 `DISABLE_COLLECTSTATIC` 設定為 `true`，則會略過此步驟。
1. 如果自訂建置後指令碼由 `POST_BUILD_COMMAND` 設定指定，請加以執行。

根據預設，`PRE_BUILD_COMMAND`、`POST_BUILD_COMMAND` 和 `DISABLE_COLLECTSTATIC` 設定是空的。 

- 若要停用在建置 Django 應用程式時執行 collectstatic 的功能，請將 `DISABLE_COLLECTSTATIC` 設定設為 true。

- 若要執行建置前命令，請將 `PRE_BUILD_COMMAND` 設定設定為包含命令 (例如 `echo Pre-build command`)，或相對於專案根目錄的指令檔案路徑，例如 `scripts/prebuild.sh`。 所有命令都必須使用專案根資料夾的相對路徑。

- 若要執行建置後命令，請將 `POST_BUILD_COMMAND` 設定設定為包含命令 (例如 `echo Post-build command`)，或相對於專案根目錄的指令檔案路徑，例如 `scripts/postbuild.sh`。 所有命令都必須使用專案根資料夾的相對路徑。

如需可自訂建置自動化的其他設定，請參閱 [Oryx 組態](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md)。 

若要深入了解 App Service 如何在 Linux 中執行和建置 Python 應用程式，請參閱 [Oryx 如何偵測和建置 Python 應用程式](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/python.md)。

> [!NOTE]
> `PRE_BUILD_SCRIPT_PATH` 和 `POST_BUILD_SCRIPT_PATH` 設定與 `PRE_BUILD_COMMAND` 和 `POST_BUILD_COMMAND` 相同，且支援用於舊版。
> 
> 名為 `SCM_DO_BUILD_DURING_DEPLOYMENT` 的設定如果包含 `true` 或 1，就會觸發部署期間發生的 Oryx 建置。 使用 git、Azure CLI 命令 `az webapp up`和 Visual Studio Code 進行部署時，此設定為 true。

> [!NOTE]
> 在所有的建置前和建置後指令碼中一律使用相對路徑，因為 Oryx 執行所在的建置容器與執行應用程式執行所在的執行階段容器不同。 絕對不要依賴應用程式專案資料夾在容器中的確切位置 (例如，其會放在 site/wwwroot 之下)。

## <a name="production-settings-for-django-apps"></a>Django 應用程式的生產環境設定

對於 Azure App Service 之類的生產環境，Django 應用程式應遵循 Django 的[部署檢查清單](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/) (djangoproject.com)。

下表描述與 Azure 相關的生產環境設定。 這些設定會定義於應用程式的 setting.py 檔案中。

| Django 設定 | Azure 的指示 |
| --- | --- |
| `SECRET_KEY` | 如[以環境變數的形式存取應用程式設定](#access-app-settings-as-environment-variables)所述，將值儲存在 App Service 設定中。 您也可以 [在 Azure Key Vault 中將值儲存為「秘密」](/azure/key-vault/secrets/quick-create-python)。 |
| `DEBUG` | 請在值為 0 (false) 的 App Service 上建立 `DEBUG` 設定，然後將此值載入為環境變數。 在您的開發環境中，使用值 1 (true) 建立 `DEBUG` 環境變數。 |
| `ALLOWED_HOSTS` | 在生產環境中，Django 會要求您在 settings.py 的 `ALLOWED_HOSTS` 陣列中包含應用程式的 URL。 您可以在執行階段使用程式碼 `os.environ['WEBSITE_HOSTNAME']`擷取此 URL。 App Service 會自動將 `WEBSITE_HOSTNAME` 環境變數設定為應用程式的 URL。 |
| `DATABASES` | 在資料庫連線的 App Service 中定義設定，並將其載入為環境變數以填入 [`DATABASES`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-DATABASES) 字典中。 或者，您可以將值 (尤其是使用者名稱和密碼) 儲存為 [Azure Key Vault 秘密](/azure/key-vault/secrets/quick-create-python)。 |

## <a name="container-characteristics"></a>容器的特性

若部署至 App Service，Python 應用程式會在 [App Service Python GitHub 存放庫](https://github.com/Azure-App-Service/python)中定義的 Linux Docker 容器內執行。 您可以在屬於特定版本的目錄內找到映像設定。

此容器具有下列特性︰

- 應用程式在執行時所使用的是 [Gunicorn WSGI HTTP 伺服器](https://gunicorn.org/)，並且會使用額外的引數 `--bind=0.0.0.0 --timeout 600`。
    - 您可以透過專案根目錄中的 *gunicorn.conf.py* 檔案來提供 Gunicorn 的組態設定，如 [Gunicorn 組態概觀](https://docs.gunicorn.org/en/stable/configure.html#configuration-file) (docs.gunicorn.org) 所述。 您也可以[自訂啟動命令](#customize-startup-command)。

    - 為了保護您的 Web 應用程式免於遭受意外或刻意的 DDOS 攻擊，Gunicorn 會在 Nginx 反向 Proxy 後面執行，如[部署 Gunicorn](https://docs.gunicorn.org/en/latest/deploy.html) (docs.gunicorn.org) 所述。

- 根據預設，基底容器映像只包含 Flask Web 架構，但容器也可支援其他符合 WSGI 標準且相容於 Python 3.6+ 的架構，例如 Django。

- 若要安裝其他套件 (例如 Django)，請在專案的根目錄中建立 [*requirements.txt*](https://pip.pypa.io/en/stable/user_guide/#requirements-files) 檔案，該指定可指定您的直接相依性。 然後 App Service 會在您部署專案時自動安裝這些相依性。

    requirements.txt 檔案「必須」位於專案根目錄中，才能安裝相依性。 否則，建至程序會報告錯誤：「找不到 setup.py 或 requirements.txt;未執行 pip 安裝。」 如果您遇到此錯誤，請檢查 requirements 檔案的位置。

- App Service 會使用 Web 應用程式的 URL (例如 `msdocs-hello-world.azurewebsites.net`)，自動定義名為 `WEBSITE_HOSTNAME` 的環境變數。 其也會使用您的應用程式名稱 (例如 `msdocs-hello-world`) 來定義 `WEBSITE_SITE_NAME`。 
   
## <a name="container-startup-process"></a>容器的啟動程序

在啟動期間，Linux 容器上的 App Service 會執行下列步驟：

1. 使用[自訂的啟動命令](#customize-startup-command) (如果有提供的話)。
2. 檢查 [Django 應用程式](#django-app)是否存在，如果偵測到，則為其啟動 Gunicorn。
3. 檢查 [Flask 應用程式](#flask-app)是否存在，如果偵測到，則為其啟動 Gunicorn。
4. 如果找不到其他應用程式，則請啟動容器內建的預設應用程式。

下列各節會提供每個選項的其他詳細資料。

### <a name="django-app"></a>Django 應用程式

針對 Django 應用程式，App Service 會尋找應用程式的程式碼內名為 `wsgi.py` 的檔案，然後使用下列命令執行 Gunicorn：

```bash
# <module> is the name of the folder that contains wsgi.py
gunicorn --bind=0.0.0.0 --timeout 600 <module>.wsgi
```

如果您想要更具體地控制啟動命令，請使用[自訂啟動命令](#customize-startup-command)，將 `<module>` 替換為包含 wsgi.py 的資料夾名稱，然後新增 `--chdir` 引數 (如果該模組不在專案根目錄中)。 例如，如果您的 wsgi.py 位於專案根目錄的 knboard/backend/config 之下，請使用引數 `--chdir knboard/backend config.wsgi`。

若要啟用生產環境記錄功能，請新增 `--access-logfile` 和 `--error-logfile` 參數，如[自訂啟動命令](#customize-startup-command)的範例所示。

### <a name="flask-app"></a>Flask 應用程式

針對 Flask，App Service 會尋找名為 application.py 或 app.py 的檔案，並啟動 Gunicorn，如下所示：

```bash
# If application.py
gunicorn --bind=0.0.0.0 --timeout 600 application:app

# If app.py
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

如果主要的應用程式模組放在不同檔案內，請為該應用程式物件使用不同的名稱，如果您想要對 Gunicorn 提供額外的引數，則請使用[自訂啟動命令](#customize-startup-command)。

### <a name="default-behavior"></a>預設行為

如果 App Service 沒找到自訂命令、Django 應用程式或 Flask 應用程式，則會執行位於 opt/defaultsite 資料夾中的預設唯讀應用程式。 預設應用程式看起來會像下面這樣：

![Linux 網頁上的預設 App Service](media/configure-language-python/default-python-app.png)

## <a name="customize-startup-command"></a>自訂啟動命令

如本文先前所述，您可透過專案根目錄中的 *gunicorn.conf.py* 檔案來提供 Gunicorn 的組態設定，如 [Gunicorn 組態概觀](https://docs.gunicorn.org/en/stable/configure.html#configuration-file)所述。

如果此種組態不足，您可以在啟動命令檔案中提供自訂啟動命令或多個命令，以控制容器的啟動行為。 啟動命令檔案可以使用您選擇的任何名稱，例如 startup.sh、startup.cmd、startup.txt 等等。

所有命令都必須使用專案根資料夾的相對路徑。

若要指定啟動命令或命令檔案：

- **Azure 入口網站** ：選取應用程式的 [組態] 頁面，然後選取 [一般設定]。 在 [啟動命令] 欄位中，放置啟動命令的全文或啟動命令檔案的名稱。 然後選取 [儲存] 以套用變更。 請參閱針對 Linux 容器[設定一般設定](configure-common.md#configure-general-settings)。

- **Azure CLI** ：使用 [az webapp config set](/cli/azure/webapp/config#az_webapp_config_set) 命令搭配 `--startup-file` 參數，以設定啟動命令或檔案：

    ```azurecli-interactive
    az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
    ```
        
    將 `<custom-command>`替換為您啟動命令的全文或啟動命令檔案的名稱。
        
App Service 會忽略在處理自訂啟動命令或檔案時所發生的任何錯誤，然後藉由尋找 Django 和 Flask 應用程式來繼續其啟動程序。 如果您看不到預期的行為，請檢查您的啟動命令或檔案是否沒有錯誤，以及是否已將啟動命令檔案部署至 App Service 與您的應用程式程式碼。 您也可以查看[診斷記錄](#access-diagnostic-logs)以取得其他資訊。 另外，在 [Azure 入口網站](https://portal.azure.com)上查看應用程式的 [診斷並解決問題] 頁面。

### <a name="example-startup-commands"></a>範例啟動命令

- **已新增 Gunicorn 引數** ：下列範例會將 `--workers=4` 新增至 Gunicorn 命令列，以便啟動 Django 應用程式： 

    ```bash
    # <module-path> is the relative path to the folder that contains the module
    # that contains wsgi.py; <module> is the name of the folder containing wsgi.py.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi
    ```    

    如需詳細資訊，請參閱[執行 Gunicorn](https://docs.gunicorn.org/en/stable/run.html) (docs.gunicorn.org)。

- **啟用 Django 的生產環境記錄功能** ：將 `--access-logfile '-'` 和 `--error-logfile '-'` 引數新增至命令列：

    ```bash    
    # '-' for the log files means stdout for --access-logfile and stderr for --error-logfile.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi --access-logfile '-' --error-logfile '-'
    ```    

    這些記錄會出現在 [App Service 記錄資料流](#access-diagnostic-logs)中。

    如需詳細資訊，請參閱 [Gunicorn 記錄](https://docs.gunicorn.org/en/stable/settings.html#logging) (docs.gunicorn.org)。
    
- **自訂 Flask 主要模組** ：根據預設，App Service 假設 Flask 應用程式的主要模組為 *application.py* 或 *app.py* 。 如果您的主要模組使用不同的名稱，您就必須自訂啟動命令。 例如，如果您擁有的 Flask 應用程式，而其主要模組是 hello.py，且該檔案中的 Flask 應用程式物件名為 `myapp`，則命令如下所示：

    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 hello:myapp
    ```
    
    如果您的主要模組位於子資料夾中 (例如 `website`)，請使用 `--chdir` 引數指定該資料夾：
    
    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 --chdir website hello:myapp
    ```
    
- **使用非 Gunicorn 伺服器** ：若要使用不同的 Web 伺服器 (例如 [aiohttp](https://aiohttp.readthedocs.io/en/stable/web_quickstart.html))，請使用適當的命令作為啟動命令或使用啟動命令檔案中的適當命令：

    ```bash
    python3.7 -m aiohttp.web -H localhost -P 8080 package.module:init_func
    ```

## <a name="access-app-settings-as-environment-variables"></a>以環境變數的形式存取應用程式設定

應用程式設定是特別為您的應用程式儲存在雲端中的值，如[設定應用程式設定](configure-common.md#configure-app-settings)所述。 這些設定可供您的應用程式程式碼作為環境變數並使用標準 [os.environ](https://docs.python.org/3/library/os.html#os.environ) 模式來存取。

例如，如果您已建立名為 `DATABASE_SERVER` 的應用程式設定，下列程式碼會擷取該設定的值：

```python
db_server = os.environ['DATABASE_SERVER']
```
    
## <a name="detect-https-session"></a>偵測 HTTPS 工作階段

在 App Service 中，[SSL 終止](https://wikipedia.org/wiki/TLS_termination_proxy) (wikipedia.org) 會在網路負載平衡器上發生，因此所有的 HTTPS 要求都會以未加密 HTTP 要求的形式進入您的應用程式。 如果您的應用程式邏輯需要檢查使用者要求是否有加密，請檢查 `X-Forwarded-Proto` 標頭。

```python
if 'X-Forwarded-Proto' in request.headers and request.headers['X-Forwarded-Proto'] == 'https':
# Do something when HTTPS is used
```

熱門的 Web 架構可讓您在標準的應用程式模式中存取 `X-Forwarded-*` 資訊。 在 [CodeIgniter](https://codeigniter.com/) 中，[is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) 預設會檢查 `X_FORWARDED_PROTO` 的值。

## <a name="access-diagnostic-logs"></a>存取診斷記錄

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

若要透過 Azure 入口網站存取記錄，請在應用程式的左側功能表上選取 [監視] > [記錄資料流]。

## <a name="open-ssh-session-in-browser"></a>在瀏覽器中開啟 SSH 工作階段

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

## <a name="troubleshooting"></a>疑難排解

- **您在部署自己的應用程式程式碼之後，卻看到預設應用程式。** 之所以出現預設應用程式，原因可能是您並未將應用程式程式碼部署至 App Service，也可能是 App Service 找不到應用程式程式碼，而改為執行預設應用程式。

    - 重新啟動 App Service，等候 15 到 20 秒，然後再檢查一次應用程式。
    
    - 請確定您使用的是適用於 Linux 的 App Service，而不是以 Windows 為基礎的執行個體。 從 Azure CLI 執行 `az webapp show --resource-group <resource-group-name> --name <app-name> --query kind` 命令，並替換對應的 `<resource-group-name>` 和 `<app-service-name>`。 輸出應該會是 `app,linux`；若非如此，請重新建立 App Service 並選擇 Linux。
    
    - 使用 SSH 或 Kudu 主控台直接連線到 App Service，並確認檔案有在 site/wwwroot 底下。 如果檔案不存在，請檢閱部署程序，並重新部署應用程式。
    
    - 如果檔案存在，App Service 卻無法識別特定的啟動檔案。 請檢查應用程式的結構是否符合 App Service 對 [Django](#django-app) 或 [Flask](#flask-app) 的預期，或者，您也可以使用[自訂啟動命令](#customize-startup-command)。

- **您在瀏覽器中看到「服務無法使用」訊息。** 瀏覽器在等候 App Service 的回應時逾時，這表示 App Service 已啟動 Gunicorn 伺服器，但用來指定應用程式程式碼的引數不正確。

    - 重新整理瀏覽器，如果您使用 App Service 方案中的最低定價層，更應該如此。 舉例來說，如果您使用免費層，應用程式可能需要更久的時間才能啟動，在您重新整理瀏覽器後，應用程式便會有回應。

    - 請檢查應用程式的結構是否符合 App Service 對 [Django](#django-app) 或 [Flask](#flask-app) 的預期，或者，您也可以使用[自訂啟動命令](#customize-startup-command)。

    - 檢查[記錄資料流](#access-diagnostic-logs)是否有任何錯誤訊息。

- **記錄資料流「找不到 setup.py 或 requirements.txt;未執行 pip 安裝。」** ：Oryx 建置程序找不到您的 requirements.txt 檔案。

    - 使用 SSH 或 Kudu 主控台直接連線到 App Service，並確認 requirements.txt 直接存在於 site/wwwroot 底下。 如果不存在，請將該檔案放在您的存放庫中，並包含在您的部署中。 如果其存在於不同資料夾中，請將其移至根目錄。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [教學課程：使用 PostgreSQL 的 Python 應用程式](tutorial-python-postgresql-app.md)

> [!div class="nextstepaction"]
> [教學課程：從私人容器存放庫部署](tutorial-custom-container.md?pivots=container-linux)

> [!div class="nextstepaction"]
> [App Service Linux 常見問題集](faq-app-service-linux.md)
