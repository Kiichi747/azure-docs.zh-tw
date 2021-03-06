---
title: "使用 Azure CLI 建立 Azure Database for PostgreSQL | Microsoft Docs"
description: "本快速入門指南說明如何使用 Azure CLI (命令列介面) 建立及管理 Azure Database for PostgreSQL 伺服器。"
services: postgresql
author: sanagama
ms.author: sanagama
manager: jhubbard
editor: jasonh
ms.assetid: 
ms.service: postgresql-database
ms.custom: quick start create
ms.tgt_pltfrm: portal
ms.devlang: azurecli
ms.topic: hero-article
ms.date: 05/10/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: fa02abd9176fa17df3b2d7f396988b72c28a473c
ms.contentlocale: zh-tw
ms.lasthandoff: 05/10/2017

---
# <a name="create-an-azure-database-for-postgresql-using-the-azure-cli"></a>使用 Azure CLI 建立 Azure Database for PostgreSQL

Azure Database for PostgreSQL 是一個受管理的服務，可讓您在雲端執行、管理及調整高可用性 PostgreSQL 資料庫。 Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門說明如何使用 Azure CLI 在 [Azure 資源群組](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)中建立 Azure Database for PostgreSQL 伺服器。

若要完成本快速入門，請確定您已安裝最新的 [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli)。 

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="log-in-to-azure"></a>登入 Azure

使用 [az login](/cli/azure/#login) 命令登入 Azure 訂用帳戶並遵循畫面上的指示。
```azurecli
az login
```
如果您有多個訂用帳戶，請選擇資源計費的適當訂用帳戶。 使用 [az account set](/cli/azure/account#set) 命令在您的帳戶之下選取特定訂用帳戶識別碼。
```azurecli
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](../azure-resource-manager/resource-group-overview.md) 命令建立 [Azure 資源群組](/cli/azure/group#create)。 資源群組是在其中以群組方式部署與管理 Azure 資源的邏輯容器。 下列範例會在 `westus` 位置建立名為 `myresourcegroup` 的資源群組。
```azurecli
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-postgresql-server"></a>建立 Azure Database for PostgreSQL 伺服器

使用 **az postgres server create** 命令建立 [Azure Database for PostgreSQL 伺服器](overview.md)。 一個伺服器包含一組當作群組管理的資料庫。 

下列範例會使用伺服器管理員登入 `mylogin` 在 `myresourcegroup` 資源群組中建立名為 `mypgserver-20170401` 的伺服器。 伺服器的名稱會對應至 DNS 名稱，因此必須是 Azure 中全域唯一的。 將 `<server_admin_password>` 替換成您自己的值。
```azurecli
az postgres server create --resource-group myresourcegroup --name mypgserver-20170401  --location westus --admin-user mylogin --admin-password <server_admin_password> --performance-tier Basic --compute-units 50 --version 9.6
```

> [!IMPORTANT]
> 需要伺服器系統管理員登入以及您在此處指定的密碼，稍後才能在本快速入門中登入伺服器及其資料庫。 請記住或記錄此資訊，以供稍後使用。

根據預設，**postgres** 資料庫會建立在您的伺服器底下。 [postgres](https://www.postgresql.org/docs/9.6/static/app-initdb.html) 資料庫是要供使用者、公用程式及第三方應用程式使用的預設資料庫。 


## <a name="configure-a-server-level-firewall-rule"></a>設定伺服器層級防火牆規則

使用 **az postgres server firewall-rule create** 命令建立 Azure PostgreSQL 伺服器層級防火牆規則。 伺服器層級的防火牆規則允許外部應用程式 (例如 [psql](https://www.postgresql.org/docs/9.2/static/app-psql.html) 或 [PgAdmin](https://www.pgadmin.org/)) 穿過 Azure PostgreSQL 服務防火牆連線到您的伺服器。 

您可以設定一個防火牆規則，來涵蓋能夠從您網路連線的 IP 範圍。 下列範例使用 **az postgres server firewall-rule create** 建立 IP 位址範圍的防火牆規則 `AllowAllIps`。 若要開啟所有 IP 位址，請使用 0.0.0.0 作為起始 IP 位址，並使用 255.255.255.255 作為結束位址。
```azurecli
az postgres server firewall-rule create --resource-group myresourcegroup --server mypgserver-20170401 --name AllowAllIps --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```

> [!NOTE]
> Azure PostgreSQL 伺服器會透過連接埠 5432 進行通訊。 如果您嘗試從公司網路內進行連線，您網路的防火牆可能不允許透過連接埠 5432 的輸出流量。 若是如此，除非 IT 部門開啟連接埠 5432，否則您將無法連線到 Azure SQL Database 伺服器。
>

## <a name="get-the-connection-information"></a>取得連線資訊

若要連線到您的伺服器，您必須提供主機資訊和存取認證。
```azurecli
az postgres server show --resource-group myresourcegroup --name mypgserver-20170401
```

結果為 JSON 格式。 請記下 **administratorLogin** 和 **fullyQualifiedDomainName**。
```json
{
  "administratorLogin": "mylogin",
  "fullyQualifiedDomainName": "mypgserver-20170401.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mypgserver-20170401",
  "location": "westus",
  "name": "mypgserver-20170401",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 50,
    "family": null,
    "name": "PGSQLS2M50",
    "size": null,
    "tier": "Basic"
  },
  "sslEnforcement": null,
  "storageMb": 51200,
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"
}
```

## <a name="connect-to-postgresql-database-using-psql"></a>使用 psql 連線到 PostgreSQL 資料庫

如果您的用戶端電腦已安裝 PostgreSQL，您可以使用 [psql](https://www.postgresql.org/docs/9.6/static/app-psql.html) 的本機執行個體來連線到 Azure PostgreSQL 伺服器。 讓我們現在使用 psql 命令列公用程式來連線到 Azure PostgreSQL 伺服器。

1. 執行下列 psql 命令以連線到 Azure Database for PostgreSQL 伺服器
```bash
psql --host=<servername> --port=<port> --username=<user@servername> --dbname=<dbname>
```

  例如，下列命令可使用存取認證，連線到 PostgreSQL 伺服器 **mypgserver-20170401.postgres.database.azure.com** 上名為 **postgres** 的預設資料庫。 系統提示您輸入密碼時，請輸入您選擇的 `<server_admin_password>`。
  
  ```bash
psql --host=mypgserver-20170401.postgres.database.azure.com --port=5432 --username=mylogin@mypgserver-20170401 --dbname=postgres
```

2.  連線到伺服器後，請在提示字元建立空白資料庫。
```bash
CREATE DATABASE mypgsqldb;
```

3.  在提示字元，執行下列命令，將連線切換到新建立的資料庫 **mypgsqldb**：
```bash
\c mypgsqldb
```

## <a name="connect-to-postgresql-database-using-pgadmin"></a>使用 pgAdmin 連線到 PostgreSQL 資料庫

若要使用 GUI 工具_pgAdmin_ 連線到 Azure PostgreSQL 伺服器：
1.    在您的用戶端電腦上啟動 _pgAdmin_ 應用程式。 您可以從 http://www.pgadmin.org/ 安裝 _pgAdmin_。
2.    從 [快速連結] 功能表選擇 [新增伺服器]。
3.    在 [建立 - 伺服器] 對話方塊的 [一般] 索引標籤上，輸入伺服器的唯一易記名稱。 例如 **Azure PostgreSQL 伺服器**。
 ![pgAdmin 工具 - 建立 - 伺服器](./media/quickstart-create-server-database-azure-cli/1-pgadmin-create-server.png)
4.    在 [建立 - 伺服器] 對話方塊的 [連線] 索引標籤︰
    - 在 [主機名稱/位址] 方塊中輸入完整的伺服器名稱 (例如，**mypgserver-20170401.postgres.database.azure.com**)。 
    - 在 [連接埠] 方塊中輸入連接埠 5432。 
    - 在 [使用者名稱] 和 [密碼] 方塊中，分別輸入稍早在此快速入門中取得的**伺服器管理員登入 (user@mypgserver)** 以及在您建立伺服器時所輸入的密碼。
    - 將 [SSL 模式] 選取為 [必要]。 根據預設，建立所有 Azure PostgreSQL 伺服器時都會開啟 SSL 強制執行。 若要關閉 SSL 強制執行，請參閱[強制 SSL](./concepts-ssl-connection-security.md) 中的詳細資訊。

    ![pgAdmin - 建立 - 伺服器](./media/quickstart-create-server-database-azure-cli/2-pgadmin-create-server.png)
5.    按一下 [儲存] 。
6.    在瀏覽器的左側窗格中展開 [伺服器群組]。 選擇您的伺服器 **Azure PostgreSQL 伺服器**。
7.  選擇您連線到的 [伺服器]，然後選擇其下的 [資料庫]。 
8.    以滑鼠右鍵按一下 [資料庫] 以建立資料庫。
9.    選擇資料庫名稱 **mypgsqldb** 以及其擁有者作為伺服器管理員登入 **mylogin**。
10. 按一下 [儲存] 以建立空白資料庫。
11. 在 [瀏覽器] 中展開 [伺服器] 群組。 展開您所建立的伺服器，並請參閱其下的資料庫 **mypgsqldb**。
 ![pgAdmin - 建立 - 資料庫](./media/quickstart-create-server-database-azure-cli/3-pgadmin-database.png)


## <a name="clean-up-resources"></a>清除資源

刪除 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)，以清除您在快速入門中建立的所有資源。

> [!TIP]
> 此集合中的其他快速入門會以本快速入門為基礎。 如果您打算繼續進行後續的快速入門，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列步驟，在 Azure CLI 中刪除本快速入門所建立的所有資源。

```azurecli
az group delete --name myresourcegroup
```

如果您只想要刪除一個新建立的伺服器：
```azurecli
az postgres server delete --resource-group myresourcegroup --name mypgserver-20170401
```

## <a name="next-steps"></a>後續步驟
- 使用[匯出和匯入](./howto-migrate-using-export-and-import.md)或[傾印和還原](./howto-migrate-using-dump-and-restore.md)來移轉資料庫。
- 如需此服務的技術概觀，請參閱[什麼是 ？](overview.md)
