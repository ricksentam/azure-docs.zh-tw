---
title: Azure 春季雲端的服務方案和配額
description: 瞭解 Azure 春季雲端的服務配額和服務方案
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 496f2e812a102e85fea92a535552daaaadf5f31e
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2020
ms.locfileid: "92093424"
---
# <a name="quotas-and-service-plans-for-azure-spring-cloud"></a>Azure 春季雲端的配額和服務方案

**本文適用於：** ✔️ Java ✔️ C#

所有 Azure 服務都會設定資源和功能的預設限制和配額。   Azure 春季雲端提供兩種定價層：基本和標準。 本文將詳細說明這兩個層級的限制。

## <a name="azure-spring-cloud-service-tiers-and-limits"></a>Azure 春季雲端服務層和限制

| 資源 | 基本 | 標準
------- | ------- | -------
vCPU | 每個服務實例1個 | 每個服務實例4個
記憶體 | 每個服務實例 2 GB | 每個服務實例 8 GB
Azure Spring Cloud 服務執行個體 (每個訂用帳戶在每個區域) | 10 | 10
總應用程式執行個體 (每個 Azure Spring Cloud 服務執行個體) | 25 | 500
永續性磁碟區 | 1 GB/app x 10 個應用程式 | 50 GB/app x 10 應用程式

## <a name="next-steps"></a>後續步驟

某些預設限制可以增加。 如果您的安裝程式需要增加，請 [建立支援要求](../azure-portal/supportability/how-to-create-azure-support-request.md)。