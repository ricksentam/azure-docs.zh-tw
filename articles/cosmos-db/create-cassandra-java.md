---
title: Java 應用程式與使用 Java 3.0 SDK 的 Azure Cosmos DB Cassandra API
description: 本快速入門示範如何使用 Azure Cosmos DB Cassandra API，以使用 Azure 入口網站和 Java 3.0 SDK 建立設定檔應用程式。
ms.service: cosmos-db
author: TheovanKraay
ms.author: thvankra
ms.subservice: cosmosdb-cassandra
ms.devlang: java
ms.topic: quickstart
ms.date: 05/18/2020
ms.custom: seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 59f3bf1cfda7bf26f63c1ec1352a5a231ee07995
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/15/2020
ms.locfileid: "90526777"
---
# <a name="quickstart-build-a-java-app-to-manage-azure-cosmos-db-cassandra-api-data-v3-driver"></a>建置 Java 應用程式來管理 Azure Cosmos DB Cassandra API 資料 (v3 驅動程式)

> [!div class="op_single_selector"]
> * [.NET](create-cassandra-dotnet.md)
> * [.NET Core](create-cassandra-dotnet-core.md)
> * [Java v3](create-cassandra-java.md)
> * [Java v4](create-cassandra-java-v4.md)
> * [Node.js](create-cassandra-nodejs.md)
> * [Python](create-cassandra-python.md)
>  

在本快速入門中，您會建立 Azure Cosmos DB Cassandra API 帳戶，並使用適用於 Java 的 [v3.x Apache Cassandra 驅動程式](https://github.com/datastax/java-driver/tree/3.x)，以及自 GitHub 複製的 Cassandra Java 應用程式來建立 Cassandra 資料庫和容器。 Azure Cosmos DB 是多模型的資料庫服務，可讓您快速建立及查詢具有全域散發和水平調整功能的文件、資料表、索引鍵/值及圖形資料庫。

## <a name="prerequisites"></a>必要條件

- 具有有效訂用帳戶的 Azure 帳戶。 [建立免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。 或[免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) (不需 Azure 訂用帳戶)。
- [Java 開發套件 (JDK) 8](https://www.azul.com/downloads/azure-only/zulu/?&version=java-8-lts&architecture=x86-64-bit&package=jdk) \(英文\)。 將 `JAVA_HOME` 環境變數指向 JDK 安裝所在的資料夾。
- [Maven 二進位檔封存](https://maven.apache.org/download.cgi)。 在 Ubuntu 上，執行 `apt-get install maven` 來安裝 Maven。
- [Git](https://www.git-scm.com/downloads)。 在 Ubuntu 上，執行 `sudo apt-get install git` 來安裝 Git。

## <a name="create-a-database-account"></a>建立資料庫帳戶

您必須先使用 Azure Cosmos DB 建立 Cassandra 帳戶，才可以建立文件資料庫。

[!INCLUDE [cosmos-db-create-dbaccount-cassandra](../../includes/cosmos-db-create-dbaccount-cassandra.md)]

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在讓我們切換為使用程式碼。 我們將從 GitHub 複製 Cassandra 應用程式、設定連接字串，然後加以執行。 您會看到，以程式設計方式來處理資料有多麼的容易。 

1. 開啟命令提示字元。 建立名為 `git-samples` 的新資料夾。 接著，關閉命令提示字元。

    ```bash
    md "C:\git-samples"
    ```

2. 開啟 git 終端機視窗 (例如 git bash)，並使用 `cd` 命令變更至要安裝範例應用程式的新資料夾。

    ```bash
    cd "C:\git-samples"
    ```

3. 執行下列命令來複製範例存放庫。 此命令會在您的電腦上建立範例應用程式副本。

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started.git
    ```

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 如果您有興趣了解程式碼如何建立資料庫資源，您可以檢閱下列程式碼片段。 或者也可以直接跳至[更新您的連接字串](#update-your-connection-string)。 這些程式碼片段摘錄自 *src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java* 檔案。  

* Cassandra 主機、連接埠、使用者名稱、密碼與 TLS/SSL 選項皆已設定。 連接字串資訊來自 Azure 入口網站的連接字串頁面。

   ```java
   cluster = Cluster.builder().addContactPoint(cassandraHost).withPort(cassandraPort).withCredentials(cassandraUsername, cassandraPassword).withSSL(sslOptions).build();
   ```

* `cluster` 會連線至 Azure Cosmos DB Cassandra API，並傳回要存取的工作階段。

    ```java
    return cluster.connect();
    ```

下列程式碼片段摘錄自 *src/main/java/com/azure/cosmosdb/cassandra/repository/UserRepository.java* 檔案。

* 建立新的 keyspace。

    ```java
    public void createKeyspace() {
        final String query = "CREATE KEYSPACE IF NOT EXISTS uprofile WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3' } ";
        session.execute(query);
        LOGGER.info("Created keyspace 'uprofile'");
    }
    ```

* 建立新的資料表。

   ```java
   public void createTable() {
        final String query = "CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)";
        session.execute(query);
        LOGGER.info("Created table 'user'");
   }
   ```

* 使用備妥的陳述式物件插入使用者實體。

    ```java
    public PreparedStatement prepareInsertStatement() {
        final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
        return session.prepare(insertStatement);
    }

    public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
    }
    ```

* 取得所有使用者資訊的查詢。

    ```java
   public void selectAllUsers() {
        final String query = "SELECT * FROM uprofile.user";
        List<Row> rows = session.execute(query).all();

        for (Row row : rows) {
            LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
        }
    }
    ```

* 取得單一使用者資訊的查詢。

    ```java
    public void selectUser(int id) {
        final String query = "SELECT * FROM uprofile.user where user_id = 3";
        Row row = session.execute(query).one();

        LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
    }
    ```

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。 連接字串詳細資料可讓您的應用程式與託管資料庫進行通訊。

1. 在 [Azure 入口網站](https://portal.azure.com/)中的 Azure Cosmos DB 帳戶中，選取 [連接字串]****。 

    :::image type="content" source="./media/create-cassandra-java/copy-username-connection-string-azure-portal.png" alt-text="從 Azure 入口網站 [連接字串] 頁面檢視及複製使用者名稱":::

2. 使用畫面右側的 :::image type="icon" source="./media/create-cassandra-java/copy-button-azure-portal.png"::: 按鈕來複製 CONTACT POINT 值。 

3. 從 C:\git-samples\azure-cosmosdb-cassandra-java-getting-started\java-examples\src\main\resources** 資料夾開啟 config.properties** 檔案。 

3. 從入口網站將 [連絡點] 值貼到 `<Cassandra endpoint host>` 的行 2。

    config.properties** 的第 2 行現在看起來應該類似於 

    `cassandra_host=cosmos-db-quickstart.cassandra.cosmosdb.azure.com`

3. 返回入口網站，並複製 [使用者名稱] 值。 從入口網站將 [使用者名稱] 值貼到 `<cassandra endpoint username>` 的行 4。

    config.properties** 的第 4 行現在看起來應該類似於 

    `cassandra_username=cosmos-db-quickstart`

4. 返回入口網站，並複製 [密碼] 值。 從入口網站將 [密碼] 值貼到 `<cassandra endpoint password>` 的行 5。

    config.properties** 的第 5 行現在看起來應該類似於 

    `cassandra_password=2Ggkr662ifxz2Mg...==`

5. 在第 6 行中，如果您想要使用特定的 TLS/SSL 憑證，請將 `<SSL key store file location>` 取代為 TLS/SSL 憑證的位置。 如果未提供值，會使用安裝在 <JAVA_HOME>/jre/lib/security/cacerts 的 JDK 憑證。 

6. 如果您變更行 6 為使用特定的 TLS/SSL 憑證，請更新行 7 以使用該憑證的密碼。 

7. 儲存 config.properties** 檔案。

## <a name="run-the-java-app"></a>執行 Java 應用程式

1. 在 git 終端機視窗中，`cd` 至 `azure-cosmosdb-cassandra-java-getting-started` 資料夾。

    ```git
    cd "C:\git-samples\azure-cosmosdb-cassandra-java-getting-started"
    ```

2. 在 git 終端機視窗中，使用下列命令產生 `cosmosdb-cassandra-examples.jar` 檔案。

    ```git
    mvn clean install
    ```

3. 在 git 終端機視窗中，執行下列命令以啟動 JAVA 應用程式。

    ```git
    java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
    ```

    終端機視窗會顯示 keyspace 和資料表已建立的通知。 它會接著選取並傳回資料表中的所有使用者並顯示輸出，然後依識別碼選取資料列並顯示值。  

    按 Ctrl+C 來停止執行程式，並關閉主控台視窗。

4. 在 Azure 入口網站中，開啟 [資料總管]**** 以查詢、修改及使用這個新資料。 

    :::image type="content" source="./media/create-cassandra-java/view-data-explorer-java-app.png" alt-text="從 Azure 入口網站 [連接字串] 頁面檢視及複製使用者名稱":::

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 Cassandra API 建立 Azure Cosmos DB 帳戶，並已執行可建立 Cassandra 資料庫和容器的 Cassandra Java 應用程式。 您現在可以將其他資料匯入 Azure Cosmos DB 帳戶中。 

> [!div class="nextstepaction"]
> [將 Cassandra 資料匯入到 Azure Cosmos DB](cassandra-import-data.md)
