---
title: "Azure CLI 指令碼範例 - 在 Docker 容器中建立 ASP.NET Core Web 應用程式 | Microsoft Docs"
description: "Azure CLI 指令碼範例 - 在 Docker 容器中建立 ASP.NET Core Web 應用程式"
services: appservice
documentationcenter: appservice
author: syntaxc4
manager: erikre
editor: 
tags: azure-service-management
ms.assetid: 3a2d1983-ff7b-476a-ac44-49ec2aabb31a
ms.service: app-service
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: web
ms.date: 03/20/2017
ms.author: cfowler
ms.translationtype: Human Translation
ms.sourcegitcommit: 9568210d4df6cfcf5b89ba8154a11ad9322fa9cc
ms.openlocfilehash: 1876052306c5452dd0d183ce290ab6ff49e8fbae
ms.contentlocale: zh-tw
ms.lasthandoff: 05/15/2017

---

# <a name="create-an-aspnet-core-web-app-in-a-docker-container"></a>在 Docker 容器中建立 ASP.NET Core Web 應用程式

在此案例中，您會學習如何建立資源群組、Linux App Service 方案和 Web 應用程式，並使用 Docker 容器部署 ASP.NET Core 應用程式。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[主要](../../../cli_scripts/app-service/deploy-linux-docker/deploy-linux-docker.sh?highlight=6 "Linux Docker")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、Web 應用程式和所有相關資源。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意事項 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | 建立用來存放所有資源的資源群組。 |
| [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#create) | 建立 App Service 方案。 這就像是 Azure Web 應用程式的伺服器陣列。 |
| [az appservice web create](https://docs.microsoft.com/cli/azure/appservice/web#create) | 在 App Service 方案內建立 Azure Web 應用程式。 |
| [az appservice web config container update](https://docs.microsoft.com/cli/azure/appservice/web/config/container#update) | 設定 Azure Web 應用程式的 Docker 容器。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure/overview)。

您可以在 [Azure App Service 文件](../app-service-cli-samples.md)中找到其他的 App Service CLI 指令碼範例。

