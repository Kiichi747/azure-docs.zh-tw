---
title: "從 Azure 入口網站建立您的第一個函式 | Microsoft Docs"
description: "了解如何使用 Azure 入口網站來建立您的第一個 Azure 函式以進行無伺服器執行。"
services: functions
documentationcenter: na
author: ggailey777
manager: erikre
editor: 
tags: 
ms.assetid: 96cf87b9-8db6-41a8-863a-abb828e3d06d
ms.service: functions
ms.devlang: multiple
ms.topic: hero-article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 05/10/2017
ms.author: glenga
ms.translationtype: Human Translation
ms.sourcegitcommit: 5e92b1b234e4ceea5e0dd5d09ab3203c4a86f633
ms.openlocfilehash: abd508631787ba5f839a4ae2ea82e76c4bfab425
ms.contentlocale: zh-tw
ms.lasthandoff: 05/10/2017


---
# <a name="create-your-first-function-in-the-azure-portal"></a>在 Azure 入口網站中建立您的第一個函式

Azure Functions 可讓您在無伺服器環境中執行程式碼，而不需要先建立 VM 或發佈 Web 應用程式。 在本主題中，請學習如何使用 Functions 在 Azure 入口網站中建立「hello world」函式。

![在 Azure 入口網站中建立函式應用程式](./media/functions-create-first-azure-function/function-app-in-portal-editor.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

完成本主題中的所有步驟需要不到 5 分鐘的時間。

## <a name="log-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-function-app"></a>建立函數應用程式

您必須擁有函式應用程式以便主控函式的執行。 函式應用程式可讓您將多個函式群組為邏輯單位，以方便您管理、部署和共用資源。 

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

![已成功建立函式應用程式。](./media/functions-create-first-azure-function/function-app-create-success.png)

[!INCLUDE [functions-portal-favorite-function-apps](../../includes/functions-portal-favorite-function-apps.md)]

接下來，您要在新的函式應用程式中建立函式。

## <a name="create-function"></a>建立由 HTTP 觸發的函式

1. 展開新的函式應用程式，然後按一下 [Functions] 旁的 **+** 按鈕。

2.  在 [即刻開始使用] 頁面上，按一下 [WebHook + API]，並選擇您的函式語言，然後按一下 [建立此函式]。 
   
    ![Azure 入口網站中的 Functions 快速入門。](./media/functions-create-first-azure-function/function-app-quickstart-node-webhook.png)

系統隨即會使用由 HTTP 觸發之函式的範本，以您所選的語言來建立函式。 您可以藉由傳送 HTTP 要求來執行新的函式。

## <a name="test-the-function"></a>測試函式

1. 在新的函式中，按一下 [</> 取得函式 URL]，然後複製**函式 URL**。 

    ![從 Azure 入口網站複製函式 URL](./media/functions-create-first-azure-function/function-app-develop-tab-testing.png)

2. 將 HTTP 要求的 URL 貼到瀏覽器的網址列。 將查詢字串 `&name=<yourname>` 附加至此 URL 並執行要求。 下圖顯示瀏覽器中對於函式傳回之 GET 要求所做出的回應︰

    ![瀏覽器中的函式回應。](./media/functions-create-first-azure-function/function-app-browser-testing.png)

    要求 URL 預設會包含所需金鑰，以便透過 HTTP 存取您的函式。   

## <a name="view-the-function-logs"></a>檢視函式記錄 

當函式執行時，系統會將追蹤資訊寫入到記錄中。 若要查看上次執行的追蹤輸出，請在入口網站中返回您的函式，然後按一下畫面底部的向上箭號來展開**記錄**。 

![Azure 入口網站中的函式記錄檢視器。](./media/functions-create-first-azure-function/function-view-logs.png)

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [Clean up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>後續步驟

您已使用簡單的 HTTP 觸發函式建立了函式應用程式。  

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]

如需詳細資訊，請參閱 [Azure Functions HTTP 和 webhook 繫結](functions-bindings-http-webhook.md)。




