---
title: 連接程式庫-適用於 MySQL 的 Azure 資料庫
description: 本文將列出用戶端程式在連接到適用於 MySQL 的 Azure 資料庫時可使用的每個程式庫或驅動程式。
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: conceptual
ms.date: 8/3/2020
ms.openlocfilehash: 9d4d862389a607b18ee5f2440069fa2eb6cf7bc4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "87553031"
---
# <a name="connection-libraries-for-azure-database-for-mysql"></a>適用於 MySQL 的 Azure 資料庫的連線庫
本文將列出用戶端程式在連接到適用於 MySQL 的 Azure 資料庫時可使用的每個程式庫或驅動程式。

## <a name="client-interfaces"></a>用戶端介面
MySQL 提供標準的資料庫驅動程式連線，以搭配應用程式和工具使用 MySQL，其可與 ODBC 和 JDBC 的業界標準相容。 任何使用 ODBC 或 JDBC 的系統都可使用 MySQL。

| **語言** | **平台** | **其他資源** | **下載** |
| :----------- | :------------| :-----------------------| :------------|
| PHP | Windows、Linux | [適用於 PHP 的 MySQL 原生驅動程式 - mysqlnd (英文)](https://dev.mysql.com/downloads/connector/php-mysqlnd/) | [下載](https://secure.php.net/downloads.php) |
| ODBC | Windows、Linux、Mac OS X 和 Unix 平台 | [MySQL 連接器/ODBC 開發人員指南 (英文)](https://dev.mysql.com/doc/connector-odbc/en/) | [下載](https://dev.mysql.com/downloads/connector/odbc/) |
| ADO.NET | Windows | [MySQL 連接器/Net 開發人員指南 (英文)](https://dev.mysql.com/doc/connector-net/en/) | [下載](https://dev.mysql.com/downloads/connector/net/) |
| JDBC | 平台獨立 | [MySQL 連接器/J 5.1 開發人員指南 (英文)](https://dev.mysql.com/doc/connector-j/5.1/en/) | [下載](https://dev.mysql.com/downloads/connector/j/) |
| Node.js | Windows、Linux、Mac OS X | [sidorares/node-mysql2 (英文)](https://github.com/sidorares/node-mysql2/tree/master/documentation) | [下載](https://github.com/sidorares/node-mysql2) |
| Python | Windows、Linux、Mac OS X | [MySQL 連接器/Python 開發人員指南 (英文)](https://dev.mysql.com/doc/connector-python/en/) | [下載](https://dev.mysql.com/downloads/connector/python/) |
| C++ | Windows、Linux、Mac OS X | [MySQL 連接器/C++ 開發人員指南 (英文)](https://dev.mysql.com/doc/connector-cpp/en/) | [下載](https://dev.mysql.com/downloads/connector/python/) |
| C | Windows、Linux、Mac OS X | [MySQL Connector/C 開發人員指南](https://dev.mysql.com/doc/c-api/8.0/en/) | [下載](https://dev.mysql.com/downloads/connector/c/)
| Perl | Windows、Linux、Mac OS X 和 Unix 平台 | [DBD::MySQL](https://metacpan.org/pod/DBD::mysql) | [下載](https://metacpan.org/pod/DBD::mysql) |


## <a name="next-steps"></a>後續步驟
請閱讀這些快速入門，以了解如何使用您選擇的語言來連線及查詢適用於 MySQL 的 Azure 資料庫：

- [PHP](./connect-php.md)
- [Java](./connect-java.md)
- [.NET (C#)](./connect-csharp.md)
- [Python](./connect-python.md)
- [Node.JS](./connect-nodejs.md)
- [Ruby](./connect-ruby.md)
- [C++](connect-cpp.md)
- [Go](./connect-go.md)
