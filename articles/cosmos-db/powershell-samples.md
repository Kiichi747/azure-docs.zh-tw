---
title: "適用於 Azure Cosmos DB 的 Azure PowerShell 範例 | Microsoft Docs"
description: "Azure PowerShell 範例：協助您建立和管理 Azure Cosmos DB 帳戶的指令碼。"
services: cosmosdb
author: mimig1
manager: jhubbard
editor: 
tags: azure-service-management
ms.assetid: 
ms.service: cosmosdb
ms.custom: sample
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: database
ms.date: 05/10/2017
ms.author: mimig
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 8bf047bd19c5278bfff85cab63ea10a1838cc683
ms.contentlocale: zh-tw
ms.lasthandoff: 05/10/2017

---

# <a name="azure-powershell-samples-for-azure-cosmos-db"></a>適用於 Azure Cosmos DB 的 Azure PowerShell 範例

下表包含適用於 Azure Cosmos DB 之範例 Azure PowerShell 指令碼的連結。

| |  |
|---|---|
|**建立 Azure Cosmos DB 帳戶**||
|[建立 DocumentDB API 帳戶](scripts/create-database-account-powershell.md)| 建立單一 Azure Cosmos DB 帳戶以搭配 DocumentDB API 使用。 |
|**調整 Azure Cosmos DB**||
|[複寫多個區域中的 Azure Cosmos DB 帳戶和設定容錯移轉優先順序](scripts/scale-multiregion-powershell.md)|使用指定的容錯移轉優先順序，將帳戶資料複寫到全球多個區域中。|
|**保護 Azure Cosmos DB**||
| [取得帳戶金鑰](scripts/secure-get-account-key-powershell.md) | 取得帳戶主要和次要的主要 (master) 寫入金鑰，以及主要和次要唯讀金鑰。|
| [取得 MongoDB 連接字串](scripts/secure-mongo-connection-string-powershell.md) | 取得連接字串以將您的 MongoDB 應用程式連線到 Azure Cosmos DB 帳戶。|
|[重新產生帳戶金鑰](scripts/secure-regenerate-key-powershell.md)|重新產生帳戶的主要或唯讀金鑰。|
|[建立防火牆](scripts/create-firewall-powershell.md)| 建立輸入 IP 存取控制原則，以限制從核准的電腦集合和/或雲端服務存取帳戶。|
|**高可用性、災害復原、備份和還原**||
|[設定容錯移轉原則](scripts/ha-failover-policy-powershell.md)|針對要在其中複寫帳戶的每個區域，設定容錯移轉優先順序。|
|||
