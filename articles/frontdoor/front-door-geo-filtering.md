---
title: 在 Azure Front Door 的網域上進行地區篩選 | Microsoft Docs
description: 在本文中，您會了解 Azure Front Door 的地區篩選原則
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2020
ms.author: duau
ms.reviewer: tyao
ms.openlocfilehash: 42697a57d39f4a34eee4866b67e2cde947db1ff5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "91449256"
---
# <a name="geo-filtering-on-a-domain-for-azure-front-door"></a>針對 Azure Front Door 的網域進行地區篩選

根據預設，Azure Front Door 會回應所有使用者要求，而不論要求的來源位置為何。 在某些情況下，您可能會想要依國家/地區限制對您 web 應用程式的存取。 Front Door 中 (WAF) service 的 Web 應用程式防火牆可讓您使用自訂存取規則，針對端點上的特定路徑來定義原則，以允許或封鎖來自指定國家/地區的存取。 

WAF 原則包含一組自訂規則。 這項規則包含比對條件、動作和優先順序。 在比對條件中，您可以定義比對變數、運算子和 match 值。 若為地區篩選規則，則會 REMOTE_ADDR 比對變數、運算子為 GeoMatch，而值為感興趣的兩個字母國家/地區代碼。 您可以結合 GeoMatch 條件和 REQUEST_URI 字串比對條件，來建立以路徑為基礎的地區篩選規則。

您可以使用 [Azure PowerShell](front-door-tutorial-geo-filtering.md) 或使用 [快速入門範本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering)，為您的 Front Door 設定地區篩選原則。

## <a name="countryregion-code-reference"></a>國家/地區碼參考

|國家/地區碼 | 國家/地區名稱 |
| ----- | ----- |
| AD | 安道爾 |
| AE | 阿拉伯聯合大公國|
| AF | 阿富汗|
| AG | 安地卡及巴布達|
| AL | 阿爾巴尼亞|
| AM | 亞美尼亞|
| AO | 安哥拉|
| AR | 阿根廷|
| AS | 美屬薩摩亞|
| AT | 奧地利|
| AU | 澳大利亞|
| AZ | 亞塞拜然|
| BA | 波士尼亞赫塞哥維納|
| BB | 巴貝多|
| BD | 孟加拉|
| BE | 比利時|
| BF | 布吉納法索|
| BG | 保加利亞|
| BH | 巴林|
| BI | 蒲隆地|
| BJ | 貝南|
| BL | 聖巴瑟米|
| BN | 汶萊達魯薩蘭|
| BO | 玻利維亞|
| BR | 巴西|
| BS | 巴哈馬|
| BT | 不丹|
| BW | 波札那|
| BY | 白俄羅斯|
| BZ | 貝里斯|
| CA | Canada|
| CD | 剛果民主共和國|
| CF | 中非共和國|
| CH | 瑞士|
| CI | 象牙海岸|
| CL | 智利|
| CM | 喀麥隆|
| CN | 中國|
| CO | 哥倫比亞|
| CR | 哥斯大黎加|
| CU | 古巴|
| CV | 維德角|
| CY | 賽浦路斯|
| CZ | 捷克共和國|
| DE | 德國|
| DK | 丹麥|
| DO | 多明尼加共和國|
| DZ | 阿爾及利亞|
| EC | 厄瓜多|
| EE | 愛沙尼亞|
| EG | 埃及|
| ES | 西班牙|
| ET | 衣索比亞|
| FI | 芬蘭|
| FJ | 斐濟|
| FM | 密克羅尼西亞聯邦|
| FR | 法國|
| GB | United Kingdom|
| GE | 喬治亞|
| GF | 法屬圭亞那|
| GH | 迦納|
| GN | 幾內亞|
| GP | 哥德洛普|
| GR | 希臘|
| GT | 瓜地馬拉|
| GY | 蓋亞那|
| HK | 香港特別行政區|
| HN | 宏都拉斯|
| HR | 克羅埃西亞|
| HT | 海地|
| HU | 匈牙利|
| ID | 印尼|
| IE | 愛爾蘭|
| IL | 以色列|
| IN | 印度|
| IQ | 伊拉克|
| IR | 伊朗伊斯蘭共和國|
| IS | 冰島|
| IT | 義大利|
| JM | 牙買加|
| JO | 約旦|
| JP | 日本|
| KE | 肯亞|
| KG | 吉爾吉斯|
| KH | 柬埔寨|
| KI | 吉里巴斯|
| KN | 聖克里斯多福及尼維斯|
| KP | 朝鮮民主主義人民共和國|
| KR | 大韓民國|
| KW | 科威特|
| KY | 開曼群島|
| KZ | 哈薩克|
| LA | 寮人民民主共和國|
| LB | 黎巴嫩|
| LI | 列支敦斯登|
| LK | 斯里蘭卡|
| LR | 賴比瑞亞|
| LS | 賴索托|
| LT | 立陶宛|
| LU | 盧森堡|
| LV | 拉脫維亞|
| LY | 利比亞 |
| MA | 摩洛哥|
| MD | 摩爾多瓦共和國|
| MG | 馬達加斯加|
| MK | 北馬其頓|
| ML | 馬利|
| MM | 緬甸|
| MN | 蒙古|
| MO | 澳門特別行政區|
| MQ | 馬丁尼克|
| MR | 茅利塔尼亞|
| MT | 馬爾他|
| MV | 馬爾地夫|
| MW | 馬拉威|
| MX | 墨西哥|
| MY | 馬來西亞|
| MZ | 莫三比克|
| NA | 納米比亞|
| NE | 尼日|
| NG | 奈及利亞|
| NI | 尼加拉瓜|
| NL | 荷蘭|
| 否 | 挪威|
| NP | 尼泊爾|
| NR | 諾魯|
| NZ | 紐西蘭|
| OM | 阿曼|
| PA | 巴拿馬|
| PE | 祕魯|
| PH | 菲律賓|
| PK | 巴基斯坦|
| PL | 波蘭|
| PR | 波多黎各|
| PT | 葡萄牙|
| PW | 帛琉|
| PY | 巴拉圭|
| QA | 卡達|
| RE | 留尼旺|
| RO | 羅馬尼亞|
| RS | 塞爾維亞|
| RU | 俄羅斯聯邦|
| RW | 盧安達|
| SA | 沙烏地阿拉伯|
| SD | 蘇丹|
| SE | 瑞典|
| SG | 新加坡|
| SI | 斯洛維尼亞|
| SK | 斯洛伐克|
| SN | 塞內加爾|
| SO | 索馬利亞|
| SR | 蘇利南|
| SS | 南蘇丹|
| SV | 薩爾瓦多|
| SY | 敘利亞阿拉伯共和國|
| SZ | 史瓦濟蘭|
| TC | 土克斯及開科斯群島|
| TG | 多哥|
| 二四 | 泰國|
| TN | 突尼西亞|
| TR | 土耳其|
| TT | 千里達及托巴哥|
| TW | 台灣|
| TZ | 坦尚尼亞聯合共和國|
| UA | 烏克蘭|
| UG | 烏干達|
| US | 美國|
| UY | 烏拉圭|
| UZ | 烏茲別克|
| VC | 聖文森及格瑞那丁|
| VE | 委內瑞拉|
| VG | 英屬維京群島|
| VI | 美屬維京群島|
| VN | 越南|
| ZA | 南非|
| ZM | 尚比亞|
| ZW | 辛巴威|

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 深入瞭解如何 [設定地區篩選 WAF 原則](front-door-tutorial-geo-filtering.md)。
