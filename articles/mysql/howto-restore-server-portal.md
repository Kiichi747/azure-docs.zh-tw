---
title: "如何在適用於 MySQL 的 Azure 資料庫中還原伺服器 | Microsoft Docs"
description: "本文說明如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中還原伺服器。"
services: mysql
author: v-chenyh
ms.author: v-chenyh
manager: jhubbard
editor: jasonh
ms.assetid: 
ms.service: mysql-database
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: portal
ms.date: 05/10/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: 0603119da20e74b423072ce6afdb8c9f20830383
ms.contentlocale: zh-tw
ms.lasthandoff: 05/10/2017

---

# <a name="how-to-backup-and-restore-a-server-in-azure-database-for-mysql-using-the-azure-portal"></a>如何使用 Azure 入口網站，在適用於 MySQL 的 Azure 資料庫中備份和還原伺服器

## <a name="backup-happens-automatically"></a>備份會自動進行
使用適用於 MySQL 的 Azure 資料庫時，資料庫服務每隔 5 分鐘會自動備份一次服務。 

使用基本層時，備份會保留 7 天，而使用標準層時會保留 35 天。 如需詳細資訊，請參閱[適用於 MySQL 的 Azure 資料庫服務層](concepts-service-tiers.md)

透過這個自動備份功能，您可以將伺服器和其所有資料庫還原至新的伺服器，而且還原至更早的時間點。

## <a name="restore-in-the-azure-portal"></a>在 Azure 入口網站中還原
適用於 MySQL 的 Azure 資料庫可讓您將伺服器還原至過去的時間點，並還原至新的伺服器複本。 您可以使用這個新的伺服器復原資料。 

比方說，如果今天中午不小心卸除資料表，您可以還原至中午之前的時刻，然後從該新的伺服器複本擷取遺漏的資料表和資料。

下列步驟會將範例伺服器還原至某個時間點︰

1. 登入 [Azure 入口網站](https://portal.azure.com/)

2. 找出適用於 MySQL 的 Azure 資料庫伺服器。 在左窗格中，選取 [所有資源]，然後從清單中選取您的伺服器。

3.    在伺服器概觀刀鋒視窗頂端，按一下工具列上的 [還原]。 [還原] 刀鋒視窗隨即開啟。
![按一下 [還原] 按鈕](./media/howto-restore-server-portal/click-restore-button.png)

4. 在 [還原] 表單中填入必要資訊︰

- **還原點 (UTC)**︰使用日期選擇器和時間選擇器，選取要還原的時間點。 指定的時間是 UTC 格式，因此您可能需要將本地時間轉換成 UTC。
- **還原至新伺服器**︰提供新的伺服器名稱作為還原現有伺服器的目的地。
- **位置**︰區域選擇會自動填入來源伺服器區域，無法變更。
- **定價層**︰定價層選擇會自動填入與來源伺服器相同的定價層，在這裡無法變更。 
![PITR 還原](./media/howto-restore-server-portal/pitr-restore.png)

5. 按一下 [確定] 將伺服器還原至某個時間點。 

6. 還原完成之後，找出已建立的新伺服器，確認資料庫如預期般還原。

## <a name="next-steps"></a>後續步驟
- [「適用於 MySQL 的 Azure 資料庫」的連線庫](concepts-connection-libraries.md)
