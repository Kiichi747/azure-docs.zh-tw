---
title: "Azure 自動化混合式 Runbook 背景工作 | Microsoft Docs"
description: "本文提供有關安裝和使用混合式 Runbook 背景工作的相關資訊，它是 Azure 自動化的一項功能，可讓您在本機資料中心內的機器上執行 Runbook。"
services: automation
documentationcenter: 
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: 06227cda-f3d1-47fe-b3f8-436d2b9d81ee
ms.service: automation
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/02/2017
ms.author: bwren
ms.translationtype: Human Translation
ms.sourcegitcommit: 97fa1d1d4dd81b055d5d3a10b6d812eaa9b86214
ms.openlocfilehash: a63778c300a3d215a2a0025824f6363e1d9e9675
ms.contentlocale: zh-tw
ms.lasthandoff: 05/11/2017

---

# <a name="automate-resources-in-your-data-center-with-hybrid-runbook-worker"></a>使用混合式 Runbook 背景工作將資料中心內的資源自動化
Azure 自動化中的 Runbook 無法存取您的本機資料中心中的資源，因為其在 Azure 雲端中執行。  Azure 自動化的 Hybrid Runbook Worker 功能可讓您在位於資料中心的機器上執行 Runbook，以管理本機資源。 Runbook 會儲存並在 Azure 自動化中管理，接著傳遞至一或多個內部部署機器。  

下圖說明這項功能：<br>  

![混合式 Runbook 背景工作概觀](media/automation-hybrid-runbook-worker/automation.png)

您可以指定您的資料中心中的一或多部電腦，以做為混合式 Runbook 背景工作，然後從 Azure 自動化執行 Runbook。  每一個背景工作角色都需要 Microsoft Management Agent 能夠連接 Microsoft Operations Management Suite 和 Azure 自動化 Runbook 環境。  Operations Management Suite 僅用來安裝及維護管理代理程式，以及監視背景工作角色的功能。  Runbook 的傳遞和執行的指示都是透過 Azure 自動化執行。

沒有輸入防火牆需求可支援混合式 Runbook 背景工作。 本機電腦上的代理程式會起始與雲端中 Azure 自動化的所有通訊。 啟動 Runbook 時，Azure 自動化會建立代理程式會擷取的指示。 代理程式接著會在執行之前取得 Runbook 和任何參數。  它也會從 Azure 自動化擷取 Runbook 使用的[資產](http://msdn.microsoft.com/library/dn939988.aspx) 。

> [!NOTE]
> 若要透過「預期狀態設定」(DSC) 管理支援 Hybrid Runbook Worker 之伺服器的設定，您需要將它們新增為 DSC 節點。  如需有關讓它們上線以透過 DSC 進行管理的詳細資訊，請參閱[讓機器上線以透過 Azure 自動化 DSC 進行管理](automation-dsc-onboarding.md)。           
><br>
>如果您啟用[更新管理解決方案](../operations-management-suite/oms-solution-update-management.md)，則任何連線到 OMS 工作區的 Windows 電腦都會自動設定為 Hybrid Runbook Worker，以支援此解決方案所包含的 Runbook。  不過，它不會向您已在自動化帳戶中定義的任何 Hybrid Worker 群組註冊。  您可以將電腦新增到您「自動化」帳戶中的 Hybrid Runbook Worker 群組來支援「自動化」Runbook，只要解決方案和 Hybrid Runbook Worker 群組成員資格兩者所用的帳戶相同即可。  此功能已新增至 Hybrid Runbook Worker 7.2.12024.0 版。  

## <a name="hybrid-runbook-worker-groups"></a>混合式 Runbook 背景工作群組
每一個混合式 Runbook 背景工作是您安裝代理程式時指定的混合式 Runbook 背景工作群組的成員。  群組可包含單一代理程式，但您可以在群組中安裝多個代理程式以獲得高可用性。

在 Hybrid Runbook Worker 上啟動 Runbook 時，您會指定要執行它的群組。  群組的成員決定由哪一個背景工作角色處理要求。  您無法指定特定的背景工作。

## <a name="hybrid-runbook-worker-requirements"></a>Hybrid Runbook Worker 的需求
您指定至少一台內部部署電腦來執行混合式 Runbook 作業。  這台電腦必須安裝下列軟體：

* Windows Server 2012 或更新版本
* Windows PowerShell 4.0 或更新版本。  我們建議在電腦上安裝 Windows PowerShell 5.0，以提升可靠性。 您可以從 [Microsoft 下載中心](https://www.microsoft.com/download/details.aspx?id=50395)下載最新版本
* 至少雙核心和 4 GB 的 RAM

針對混合式背景工作角色，請考慮下列建議：

* 在每個群組中指定多個混合式背景工作角色，以保留較高的可用性。  
* 混合式背景工作角色可以與 Service Management Automation 或 System Center Orchestrator Runbook 伺服器並存。
* 請考慮使用實際位於或接近自動化帳戶所在區域的電腦，因為當作業完成時，作業資料會送回到 Azure 自動化。

### <a name="configure-proxy-and-firewall-settings"></a>設定 Proxy 和防火牆設定
若要讓內部部署 Hybrid Runbook Worker 連線到 Microsoft Operations Management Suite (OMS) 服務並向其註冊，它必須能夠存取下述連接埠號碼和 URL。  這是除了 [Microsoft Monitoring Agent 所需的連接埠和 URL](../log-analytics/log-analytics-windows-agents.md) 以連線至 OMS。 如果您使用 Proxy 伺服器在代理程式和 OMS 服務之間進行通訊，您必須確保可以存取適當的資源。 如果您使用防火牆來限制網際網路存取，您需要設定防火牆以允許存取。

以下資訊列出要讓 Hybrid Runbook Worker 與自動化進行通訊所需的連接埠和 URL。

* 連接埠︰只需要 TCP 443 以便進行傳出網際網路存取
* 全域 URL：*.azure-automation.net

如果您已定義特定區域的自動化帳戶，而且您想要限制與該區域資料中心的通訊，下表提供每個區域的 DNS 記錄。

| **區域** | **DNS 記錄** |
| --- | --- |
| 美國中南部 |scus-jobruntimedata-prod-su1.azure-automation.net |
| 美國東部 2 |eus2-jobruntimedata-prod-su1.azure-automation.net |
| 美國中西部 | wcus-jobruntimedata-prod-su1.azure-automation.net |
| 西歐 |we-jobruntimedata-prod-su1.azure-automation.net |
| 北歐 |ne-jobruntimedata-prod-su1.azure-automation.net |
| 加拿大中部 |cc-jobruntimedata-prod-su1.azure-automation.net |
| 東南亞 |sea-jobruntimedata-prod-su1.azure-automation.net |
| 印度中部 |cid-jobruntimedata-prod-su1.azure-automation.net |
| 日本東部 |jpe-jobruntimedata-prod-su1.azure-automation.net |
| 澳大利亞東南部 |ase-jobruntimedata-prod-su1.azure-automation.net |
| 英國南部 | uks-jobruntimedata-prod-su1.azure-automation.net |
| 美國政府維吉尼亞州 | usge-jobruntimedata-prod-su1.azure-automation.us |

如需 IP 位址 (而非名稱) 的清單，請從「Microsoft 下載中心」下載並檢閱 [Azure 資料中心 IP 位址](https://www.microsoft.com/download/details.aspx?id=41653) xml 檔案。

> [!NOTE]
> 這個檔案包含 Microsoft Azure 資料中心使用的 IP 位址範圍 (包括計算、SQL 和儲存體範圍)。 每週會公佈已更新的檔案，以反映目前已部署的範圍及任何即將進行的 IP 範圍變更。 出現在檔案中的新範圍將至少有一週的時間不會在資料中心中使用。 請每週下載新的 xml 檔案，並在您的站台上執行必要的變更，以正確識別在 Azure 中執行的服務。 Express Route 使用者可能會注意到，在每個月的第一週會使用此檔案來更新 Azure 空間的 BGP 公告。
>

## <a name="installing-hybrid-runbook-worker"></a>安裝混合式 Runbook 背景工作

安裝和設定 Hybrid Runbook Worker 有兩種可用的方法。  建議的方法是使用自動化 Runbook，將設定 Windows 電腦所需的程序完全自動化。  第二種方法採取逐步程序來手動安裝和設定角色。  

### <a name="automated-deployment"></a>自動化部署

執行下列步驟，以自動安裝和設定混合式背景工作角色。  

1. 直接在執行 Hybrid Runbook Worker 角色的電腦上，從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker/1.0/DisplayScript)下載 *New-OnPremiseHybridWorker.ps1* 指令碼，或從環境中的另一部電腦下載，再複製到背景工作角色。  

    *New-OnPremiseHybridWorker.ps1* 指令碼在執行期間需要下列參數：

  * *AutomationAccountName* (必要) - 您的自動化帳戶名稱。  
  * *ResourceGroupName* (必要) - 與您的自動化帳戶相關聯的資源群組名稱。  
  * *HybridGroupName* (必要) - 針對支援此案例的 Runbook，您指定作為目標的 Hybrid Runbook Worker 群組名稱。
  *  *SubscriptionID* (必要) - 您的自動化帳戶所在的 Azure 訂用帳戶識別碼。
  *  *WorkspaceName* (選擇性) - OMS 工作區名稱。  如果您沒有 OMS 工作區，此指令碼會建立並設定一個 OMS 工作區。  

     > [!NOTE]
     > 目前，支援與 OMS 整合的自動化區域只包括 - **澳大利亞東南部**、**美國東部 2**、**東南亞**和**西歐**。  如果您的自動化帳戶不在其中一個區域，此指令碼會建立 OMS 工作區，但會警告您，指出無法將它們連結在一起。
     >
2. 在您的電腦上，從 [開始] 畫面以系統管理員模式啟動 **Windows PowerShell**。  
3. 從 PowerShell 命令列殼層，瀏覽至您下載的指令碼所在的資料夾，然後執行該指令碼，並變更參數 *-AutomationAccountName*、*-ResourceGroupName*、*-HybridGroupName*、*-SubscriptionId* 和 *-WorkspaceName* 的值。

     > [!NOTE]
     > 執行指令碼之後，您會收到向 Azure 進行驗證的提示。  您**必須**以訂用帳戶管理員角色成員和訂用帳戶共同管理員的帳戶登入。  
     >  

        .\New-OnPremiseHybridWorker.ps1 -AutomationAccountName <NameofAutomationAccount> `
        -ResourceGroupName <NameofOResourceGroup> -HybridGroupName <NameofHRWGroup> `
        -SubscriptionId <AzureSubscriptionId> -WorkspaceName <NameOfOMSWorkspace>

4. 系統會提示您同意安裝 **NuGet**，也會提示使用您的 Azure 認證進行驗證。<br><br> ![Execution of New-OnPremiseHybridWorker script](media/automation-hybrid-runbook-worker/new-onpremisehybridworker-scriptoutput.png)

5. 指令碼完成之後，[混合式背景工作角色群組] 刀鋒視窗會顯示新的群組和成員數目，或者，如果是現有的群組，則成員數目會遞增。  您可以在 [Hybrid Worker 群組] 刀鋒視窗從清單中選取群組，然後選取 [Hybrid Worker] 圖格。  在 [Hybrid Worker] 刀鋒視窗上，您會看到列出群組的每個成員。  

### <a name="manual-deployment"></a>手動部署
對您的自動化環境執行一次前兩個步驟，再對每一台背景工作角色電腦重複其餘步驟。

#### <a name="1-create-operations-management-suite-workspace"></a>1.建立 Operations Management Suite 工作區
如果您還沒有 Operations Management Suite 工作區，請使用[管理您的工作區](../log-analytics/log-analytics-manage-access.md)中的指示建立工作區。 如果您已經有工作區，可以使用現有的工作區。

#### <a name="2-add-automation-solution-to-operations-management-suite-workspace"></a>2.將自動化解決方案加入至 Operations Management Suite 工作區
解決方案會將功能加入至 Operations Management Suite。  自動化解決方案會增加 Azure 自動化的功能，包括支援 Hybrid Runbook Worker。  將解決方案新增至工作區時，它會自動將背景工作角色元件往下推送給您在下一步將安裝的代理程式電腦。

請依照 [使用解決方案資源庫新增解決方案](../log-analytics/log-analytics-add-solutions.md) 中的指示，將 **自動化** 解決方案新增至 Operations Management Suite 工作區。

#### <a name="3-install-the-microsoft-monitoring-agent"></a>3.安裝 Microsoft Monitoring Agent
Microsoft Monitoring Agent 可將電腦連線至 Operations Management Suite。  將代理程式安裝在內部部署電腦，並連接到您的工作區時，它會自動下載 Hybrid Runbook Worker 所需的元件。

請依照[將 Windows 電腦連接到 Log Analytics](../log-analytics/log-analytics-windows-agents.md) 中的指示，將代理程式安裝在內部部署電腦上。  您可以對多部電腦重複此程序，將多個背景工作角色加入至您的環境。

當代理程式成功連接到 Operations Management Suite 時，它會列在 Operations Management Suite [設定] 窗格的 [已連接的來源] 索引標籤上。  當 C:\Program Files\Microsoft Monitoring Agent\Agent 中出現 **AzureAutomationFiles** 資料夾時，就可確認代理程式已正確下載自動化解決方案。  若要確認 Hybrid Runbook Worker 版本，您可以瀏覽至 C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\，並記下 \\version 子資料夾。   

#### <a name="4-install-the-runbook-environment-and-connect-to-azure-automation"></a>4.安裝 Runbook 環境並連接到 Azure 自動化
將代理程式新增至 Operations Management Suite 時，自動化解決方案會往下推送包含 **Add-HybridRunbookWorker** Cmdlet 的 **HybridRegistration** PowerShell 模組。  您可以使用這個 Cmdlet 在電腦上安裝 Runbook 環境並向 Azure 自動化進行註冊。

以系統管理員模式開啟 PowerShell 工作階段，並執行下列命令來匯入模組。

    cd "C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\<version>\HybridRegistration"
    Import-Module HybridRegistration.psd1

然後使用下列語法執行 **Add-HybridRunbookWorker** Cmdlet：

    Add-HybridRunbookWorker –Name <String> -EndPoint <Url> -Token <String>

您可以從 Azure 入口網站的 [管理金鑰]  刀鋒視窗取得這個 Cmdlet 所需的資訊。  在您的自動化帳戶中，從 [設定] 刀鋒視窗選取 [金鑰]，以開啟此刀鋒視窗。

![混合式 Runbook 背景工作概觀](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

* **名稱** 是混合式 Runbook 背景工作群組的名稱。 如果自動化帳戶中已有這個群組，那麼會將目前的電腦加入。  如果尚不存在，則會加入。
* **EndPoint** 是 [管理金鑰] 刀鋒視窗中的 [URL] 欄位。
* **Token** 是 [管理金鑰] 刀鋒視窗中的 [主要存取金鑰]。  

在 **Add-HybridRunbookWorker** 中新增 **-Verbose** 參數可接收安裝的詳細資訊。

#### <a name="5-install-powershell-modules"></a>5.安裝 PowerShell 模組
Runbook 可以使用 Azure 自動化環境中安裝的模組中定義的任何活動和 Cmdlet。  不過，這些模組不會自動部署至內部部署機器，所以您必須手動安裝它們。  例外狀況是預設安裝的 Azure 模組，可提供 Azure 自動化所有 Azure 服務和活動的 Cmdlet 存取權。

由於 Hybrid Runbook Worker 功能的主要目的是要管理本機資源，您很可能必須安裝支援這些資源的模組。  您可以參考 [安裝模組](http://msdn.microsoft.com/library/dd878350.aspx) 來取得安裝 Windows PowerShell 模組的詳細資訊。  安裝的模組必須位於 PSModulePath 環境變數所參考的位置，才能由 Hybrid 背景工作角色自動匯入。  如需詳細資訊，請參閱[修改 PSModulePath 安裝路徑](https://msdn.microsoft.com/library/dd878326%28v=vs.85%29.aspx)。

## <a name="removing-hybrid-runbook-worker"></a>移除 Hybrid Runbook Worker
您可以移除群組中的一或多個 Hybrid Runbook 背景工作角色，或移除該群組，視您的需求而定。  若要從內部部署電腦中移除 Hybrid Runbook Worker，請執行下列步驟。

1. 在 Azure 入口網站中，瀏覽至您的自動化帳戶。  
2. 從 [設定] 刀鋒視窗中，選取 [金鑰]，記下欄位 [URL] 和 [主要存取金鑰] 的值。  下一個步驟需要此資訊。
3. 在系統管理員模式中開啟 PowerShell 工作階段，並執行下列命令 - `Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey>`。  使用 **-Verbose** 參數可取得移除程序的詳細記錄。

> [!NOTE]
> 這不會移除電腦上的 Microsoft Monitoring Agent，只會移除 Hybrid Runbook 背景工作角色的功能和組態。  

## <a name="remove-hybrid-worker-groups"></a>移除混合式背景工作角色群組
若要移除群組，您必須先使用先前所示的程序，從群組的每一部成員電腦中移除 Hybrid Runbook Worker，然後執行下列步驟移除群組。  

1. 在 Azure 入口網站中，開啟自動化帳戶。
2. 選取 [Hybrid 背景工作角色] 刀鋒視窗中的 [Hybrid 背景工作角色群組] 圖格，然後選取要刪除的群組。  選取特定的群組之後，[Hybrid 背景工作角色群組] 屬性刀鋒視窗隨即出現。<br> ![Hybrid Runbook 背景工作角色群組刀鋒視窗](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)   
3. 在所選群組的屬性刀鋒視窗中，按一下 [刪除]。  將會出現一則訊息要求您確認此動作，如果您確定要繼續，請選取 [是]。<br> ![建立群組確認對話方塊](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)<br> 此程序需要幾秒鐘才能完成，您可以在功能表的 [通知] 底下追蹤其進度。  

## <a name="starting-runbooks-on-hybrid-runbook-worker"></a>在混合式 Runbook 背景工作上啟動 Runbook
[在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md) 描述啟動 Runbook 的不同方法。  混合式 Runbook 背景工作加入了 **RunOn** 選項，您可以在其中指定混合式 Runbook 背景工作群組的名稱。  如果未指定群組，則會擷取 Runbook，且由該群組中的背景工作執行。  如果未指定此選項，則會正常在 Azure 自動化中執行。

在 Azure 入口網站中啟動 Runbook 時，您會看到 [執行於] 選項，您可以在此選取 [Azure] 或 [Hybrid Worker]。  如果選取 [Hybrid 背景工作角色]，則您可以從下拉式清單中選取群組。

使用 **RunOn** 參數。  您可以使用 Windows PowerShell 以下列命令在 Hybrid Runbook Worker 群組上啟動名為 Test-Runbook 的 Runbook。

    Start-AzureRmAutomationRunbook –AutomationAccountName "MyAutomationAccount" –Name "Test-Runbook" -RunOn "MyHybridGroup"

> [!NOTE]
> 0.9.1 版 Microsoft Azure PowerShell 的 **Start-AzureAutomationRunbook** Cmdlet 已新增 **RunOn** 參數。  如果您安裝較早的版本，您應該 [下載最新版本](https://azure.microsoft.com/downloads/) 。  您只需要在從 Windows PowerShell 啟動 Runbook 的工作站上安裝此版本。  您不需要將它安裝在背景工作電腦上，除非您想要從該電腦啟動 Runbook。  您目前無法在混合式 Runbook 背景工作上從另一個 Runbook 啟動 Runbook，因為這項作業需要在您的自動化帳戶中安裝最新版本的 Azure PowerShell。  Azure 自動化中會自動更新為最新版本，並且很快將其自動往下推送到背景工作角色。
>
>

## <a name="runbook-permissions"></a>Runbook 權限
在 Hybrid Runbook Worker 上執行的 Runbook 不能使用通常用於 Runbook 對 Azure 資源進行驗證的相同方法，因為它們會存取 Azure 外部的資源。  Runbook 可以提供自己的驗證給本機資源，或者您可以指定 RunAs 帳戶以對所有 Runbook 提供使用者內容。

### <a name="runbook-authentication"></a>Runbook 驗證
根據預設，Runbook 將在內部部署機器上的本機系統帳戶內容中執行，因此，它們必須對將會存取的資源提供自己的驗證。  

您可以在您的 Runbook 中使用[認證](http://msdn.microsoft.com/library/dn940015.aspx)和[憑證](http://msdn.microsoft.com/library/dn940013.aspx)資產，搭配可讓您指定認證的 Cmdlet，您就能夠向不同的資源驗證。  下列範例顯示會重新啟動電腦的 Runbook 的一部分。  它會從認證資產擷取認證和從變數資產擷取電腦的名稱，然後使用這些值搭配 Restart-Computer Cmdlet。

    $Cred = Get-AzureRmAutomationCredential -ResourceGroupName "ResourceGroup01" -Name "MyCredential"
    $Computer = Get-AzureRmAutomationVariable -ResourceGroupName "ResourceGroup01" -Name  "ComputerName"

    Restart-Computer -ComputerName $Computer -Credential $Cred

您也可以運用 [InlineScript](automation-powershell-workflow.md#inlinescript)，它可讓您利用 [PSCredential 一般參數](http://technet.microsoft.com/library/jj129719.aspx)指定的認證在另一部電腦上執行程式碼區塊。

### <a name="runas-account"></a>RunAs 帳戶
您不需要讓 Runbook 提供自己的驗證給本機資源，您可以針對混合式背景工作角色群組指定 **RunAs** 帳戶。  您指定具有本機資源存取權的 [認證資產](automation-credentials.md)，在群組中的 Hybrid Runbook Worker 執行時，所有 Runbook 會在這些認證下執行。  

認證的使用者名稱必須是下列格式之一：

* 網域\使用者名稱
* username@domain
* 使用者名稱 (適用於內部部署機器的本機帳戶)

使用下列程序以針對混合式背景工作角色群組指定 RunAs 帳戶：

1. 建立具有本機資源存取權的 [認證資產](automation-credentials.md) 。
2. 在 Azure 入口網站中，開啟自動化帳戶。
3. 選取 [Hybrid Worker 群組]  圖格，然後選取群組。
4. 選取 [所有設定]，然後選取 [Hybrid 背景工作角色群組設定]。
5. 將 [執行身分] 從 [預設] 變更為 [自訂]。
6. 選取認證，然後按一下 [儲存] 。

### <a name="automation-run-as-account"></a>自動化執行身分帳戶
在 Azure 中部署資源的自動化建置流程中，您可能需要詢問內部部署系統來支援部署序列中的工作或一組步驟。  若要支援使用「執行身分」帳戶向 Azure 驗證，您需要安裝「執行身分」帳戶憑證。  

下列 PowerShell Runbook *Export-RunAsCertificateToHybridWorker* 會從 Azure 自動化帳戶匯出執行身分憑證，並將它下載和匯入 Hybird 背景工作角色 (連線至相同帳戶) 的本機憑證存放區。  完成該步驟後，它會確認背景工作角色可以使用執行身分帳戶成功向 Azure 驗證。

    <#PSScriptInfo
    .VERSION 1.0
    .GUID 3a796b9a-623d-499d-86c8-c249f10a6986
    .AUTHOR Azure Automation Team
    .COMPANYNAME Microsoft
    .COPYRIGHT
    .TAGS Azure Automation
    .LICENSEURI
    .PROJECTURI
    .ICONURI
    .EXTERNALMODULEDEPENDENCIES
    .REQUIREDSCRIPTS
    .EXTERNALSCRIPTDEPENDENCIES
    .RELEASENOTES
    #>

    <#  
    .SYNOPSIS  
    Exports the Run As certificate from an Azure Automation account to a hybrid worker in that account.

    .DESCRIPTION  
    This runbook exports the Run As certificate from an Azure Automation account to a hybrid worker in that account.
    Run this runbook in the hybrid worker where you want the certificate installed.
    This allows the use of the AzureRunAsConnection to authenticate to Azure and manage Azure resources from runbooks running in the hybrid worker.

    .EXAMPLE
    .\Export-RunAsCertificateToHybridWorker

    .NOTES
    AUTHOR: Azure Automation Team
    LASTEDIT: 2016.10.13
    #>

    [OutputType([string])]

    # Set the password used for this certificate
    $Password = "YourStrongPasswordForTheCert"

    # Stop on errors
    $ErrorActionPreference = 'stop'

    # Get the management certificate that will be used to make calls into Azure Service Management resources
    $RunAsCert = Get-AutomationCertificate -Name "AzureRunAsCertificate"

    # location to store temporary certificate in the Automation service host
    $CertPath = Join-Path $env:temp  "AzureRunAsCertificate.pfx"

    # Save the certificate
    $Cert = $RunAsCert.Export("pfx",$Password)
    Set-Content -Value $Cert -Path $CertPath -Force -Encoding Byte | Write-Verbose

    Write-Output ("Importing certificate into local machine root store from " + $CertPath)
    $SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
    Import-PfxCertificate -FilePath $CertPath -CertStoreLocation Cert:\LocalMachine\My -Password $SecurePassword -Exportable | Write-Verbose

    # Test that authentication to Azure Resource Manager is working
    $RunAsConnection = Get-AutomationConnection -Name "AzureRunAsConnection"

    Add-AzureRmAccount `
      -ServicePrincipal `
      -TenantId $RunAsConnection.TenantId `
      -ApplicationId $RunAsConnection.ApplicationId `
      -CertificateThumbprint $RunAsConnection.CertificateThumbprint | Write-Verbose

    Select-AzureRmSubscription -SubscriptionId $RunAsConnection.SubscriptionID | Write-Verbose

    # List automation accounts to confirm Azure Resource Manager calls are working
    Get-AzureRmAutomationAccount | Select AutomationAccountName

將 *Export-RunAsCertificateToHybridWorker* Runbook 以 `.ps1` 副檔名儲存至您的電腦。  將它匯入您的自動化帳戶，並編輯 Runbook，將變數 `$Password` 的值變更為您自己的密碼。  發佈 Runbook，然後以使用「執行身分」帳戶執行和驗證 Runbook 的 Hybrid Worker 群組為對象，執行 Runbook。  作業資料流會報告將憑證匯入本機電腦存放區的嘗試，後面還會出現幾行，根據訂用帳戶中定義多少自動化帳戶和驗證是否成功而定。  

## <a name="creating-runbooks-for-hybrid-runbook-worker"></a>為混合式 Runbook 背景工作建立 Runbook
在 Azure 自動化和混合式 Runbook 背景工作上執行的 Runbook 結構中沒有任何差異。 您對兩者使用的 Runbook 可能會有顯著差異，因為 Hybrid Runbook Worker 的 Runbook 通常會管理資料中心內的本機資源，而 Azure 自動化中的 Runbook 通常管理 Azure 雲端中的資源。

您可以在 Azure 自動化中編輯混合式 Runbook 背景工作的 Runbook，但如果您嘗試在編輯器中測試 Runbook，可能會遇到困難。  存取本機資源的 PowerShell 模組可能未安裝在 Azure 自動化環境中，在此情況下，測試就會失敗。  如果您已安裝所需的模組，就會執行 Runbook，但它不能存取本機資源以進行完整測試。

## <a name="troubleshooting-runbooks-on-hybrid-runbook-worker"></a>在 Hybrid Runbook Worker 上進行 Runbook 疑難排解
[Runbook 輸出和訊息](automation-runbook-output-and-messages.md) 會從混合式背景工作角色傳送到 Azure 自動化，就像雲端中執行的 Runbook 工作一樣。  您也可以啟用詳細資訊和進度資料流，就像您在其他 Runbook 中的作法一樣。  

記錄檔儲存每一個混合式背景工作角色本機的 C:\ProgramData\Microsoft\System Center\Orchestrator\7.2\SMA\Sandboxes 中。

如果您的 Runbook 未成功完成，但作業摘要顯示 [暫止]  狀態，請檢閱疑難排解文章 [Hybrid Runbook Worker：Runbook 作業在暫止狀態下終止](automation-troubleshooting-hrw-runbook-terminates-suspended.md)。   

## <a name="relationship-to-service-management-automation"></a>與 Service Management Automation 的關聯性
[Service Management Automation (SMA)](https://technet.microsoft.com/library/dn469260.aspx) 可讓您在本機資料中心內執行 Azure 自動化支援的相同 Runbook。 SMA 通常與 Windows Azure Pack 一同部署，因為 Windows Azure Pack 包含 SMA 管理的圖形化介面。 不同於 Azure 自動化，SMA 需要本機安裝，安裝中包含了裝載 API 的 Web 伺服器、包含 Runbook 和 SMA 組態的資料庫、用以執行 Runbook 作業的 Runbook 背景工作角色。 Azure 自動化在雲端中提供這些服務，並只要求您在本機環境中維護混合式 Runbook 背景工作。

如果您是現有 SMA 使用者，可以將您的 Runbook 移至 Azure 自動化，以搭配混合式 Runbook 背景工作使用，而不必變更，假設他們已依 [為混合式 Runbook 背景工作建立 Runbook](#creating-runbooks-for-hybrid-runbook-worker)中所述，向資源進行驗證 。  SMA 中的 Runbook 會在背景工作伺服器上服務帳戶的內容中執行，其可能為 Runbook 提供該驗證。

您可以使用下列準則來判斷 Azure 自動化搭配混合式 Runbook 背景工作或 Service Management Automation 更適合您的需求。

* SMA 需要本機安裝其基礎元件，如果需要圖形化管理介面，這些元件會連接到 Windows Azure Pack。 相較於 Azure 自動化，這需要更多的本機資源和更高的維護成本；Azure 自動化只需要在本機 Runbook 背景工作角色上安裝代理程式。 代理程式由 Operations Management Suite 管理，進而減少您的維護成本。
* Azure 自動化會將其 Runbook 儲存在雲端中，並將它們提供給內部部署的 Hybrid Runbook 背景工作角色。 如果您的安全性原則不允許這種行為，您應該使用 SMA。
* SMA 隨附於 System Center，因此需要 System Center 2012 R2 授權。 Azure 自動化是採用分層式的訂用帳戶模型。
* Azure 自動化具備 SMA 中並未提供的圖形 Runbook 等進階功能。

## <a name="next-steps"></a>後續步驟
* 若要深入了解可用來啟動 Runbook 的不同方法，請參閱[在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md)。  
* 若要了解使用文字式編輯器在 Azure 自動化中使用 PowerShell 和 PowerShell 工作流程 Runbook 的不同程序，請參閱 [在 Azure 自動化中編輯 Runbook](automation-edit-textual-runbook.md)

