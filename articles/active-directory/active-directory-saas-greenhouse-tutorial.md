---
title: "教學課程：Azure Active Directory 與 Greenhouse 整合 | Microsoft Docs"
description: "了解如何使用 Greenhouse 搭配 Azure Active Directory 來啟用單一登入、自動化佈建和更多功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 78ec1766-4f79-4f16-9a66-d5584c4b6151
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/10/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: ad2e712cc1bcb7aea82b2c56a8a4a0f79e2ce1e0
ms.openlocfilehash: cce21156d962c7678e2a32c0953586b786c1ad0f
ms.lasthandoff: 02/17/2017


---
# <a name="tutorial-azure-active-directory-integration-with-greenhouse"></a>教學課程：Azure Active Directory 與 Greenhouse 整合
本教學課程的目的是要示範 Azure 與 Greenhouse 的整合。  

本教學課程中說明的案例假設您已經具有下列項目：

* 有效的 Azure 訂閱
* Greenhouse 單一登入 (SSO) 的訂用帳戶

完成本教學課程之後，您指派給 Greenhouse 的 Azure AD 使用者就能夠單一登入您 Greenhouse 公司網站 (服務提供者起始登入) 的應用程式，或是使用 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。

本教學課程中說明的案例由下列建置組塊組成：

* 啟用 Greenhouse 的應用程式整合
* 設定單一登入 (SSO)
* 設定使用者佈建
* 指派使用者

![案例](./media/active-directory-saas-greenhouse-tutorial/IC790783.png "案例")

## <a name="enable-the-application-integration-for-greenhouse"></a>啟用 Greenhouse 的應用程式整合
本節的目的是要說明如何啟用 Greenhouse 的應用程式整合。

**若要啟用 Greenhouse 的應用程式整合，請執行下列步驟：**

1. 在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory] 。
   
   ![Active Directory](./media/active-directory-saas-greenhouse-tutorial/IC700993.png "Active Directory")
2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。
3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
   ![應用程式](./media/active-directory-saas-greenhouse-tutorial/IC700994.png "應用程式")
4. 按一下頁面底部的 [新增]  。
   
   ![新增應用程式](./media/active-directory-saas-greenhouse-tutorial/IC749321.png "新增應用程式")
5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。
   
   ![從資源庫新增應用程式](./media/active-directory-saas-greenhouse-tutorial/IC749322.png "從資源庫新增應用程式")
6. 在**搜尋方塊**中，輸入 **greenhouse**。
   
   ![應用程式資源庫](./media/active-directory-saas-greenhouse-tutorial/IC790784.png "應用程式資源庫")
7. 在結果窗格中，選取 [Greenhouse]，然後按一下 [完成] 來新增應用程式。
   
   ![Greenhouse](./media/active-directory-saas-greenhouse-tutorial/IC790785.png "Greenhouse")
   
## <a name="configure-single-sign-on"></a>設定單一登入

本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己的 Azure AD 帳戶驗證到 Greenhouse。

**若要設定單一登入，請執行下列步驟：**

1. 在 Azure 傳統入口網站的 [Greenhouse] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-greenhouse-tutorial/IC790786.png "設定單一登入")
2. 在 [要如何讓使用者登入 Greenhouse] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按一下 [下一步]。
   
   ![設定單一登入](./media/active-directory-saas-greenhouse-tutorial/IC790787.png "設定單一登入")
3. 在 [設定應用程式 URL] 頁面的 [登入 URL] 文字方塊中，使用下列模式輸入您的 URL："*https://company.greenhouse.io*"，然後按一下 [下一步]。
   
   ![設定應用程式 URL](./media/active-directory-saas-greenhouse-tutorial/IC790788.png "設定應用程式 URL")
4. 在 [設定在 Greenhouse 單一登入] 頁面上，按一下 [下載中繼資料]，然後將中繼資料檔儲存在您的本機電腦上。
   
   ![設定單一登入](./media/active-directory-saas-greenhouse-tutorial/IC790789.png "設定單一登入")
5. 將該中繼資料檔轉寄給 Greenhouse 支援小組。

>[!NOTE]
>單一登入必須由 Greenhouse 支援小組啟用。
>

6. 在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-greenhouse-tutorial/IC790790.png "設定單一登入")
   
## <a name="configure-user-provisioning"></a>設定使用者佈建

若要讓 Azure AD 使用者可以登入 Greenhouse，則必須將他們佈建至 Greenhouse。 在 Greenhouse 的情況下，需以手動的方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 **Greenhouse** 公司網站。
2. 在頂端的功能表中，按一下 [設定]，然後按一下 [使用者]。
   
   ![使用者](./media/active-directory-saas-greenhouse-tutorial/IC790791.png "使用者")
3. 按一下 [新使用者] 。
   
   ![新增使用者](./media/active-directory-saas-greenhouse-tutorial/IC790792.png "新增使用者")
4. 在 [新增使用者]  區段中，執行下列步驟：
   
   ![新增使用者](./media/active-directory-saas-greenhouse-tutorial/IC790793.png "新增使用者")
   1. 在 [輸入使用者電子郵件]  文字方塊中，輸入您要佈建之有效 Azure Active Directory 帳戶的電子郵件地址。
   2. 按一下 [儲存] 。    
   
      >[!NOTE]
      >Azure Active Directory 帳戶的持有者會收到一封包含連結的電子郵件，以在啟用帳戶前進行確認。
      >  

>[!NOTE]
>您可以使用任何其他的 Greenhouse 使用者帳戶建立工具或 Greenhouse 提供的 API 來佈建 AAD 使用者帳戶。 
> 

## <a name="assign-users"></a>指派使用者
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

**若要指派使用者給 Greenhouse，請執行下列步驟：**

1. 在 Azure 傳統入口網站中建立測試帳戶。
2. 在 [Greenhouse] 應用程式整合頁面上，按一下 [指派使用者]。
   
   ![指派使用者](./media/active-directory-saas-greenhouse-tutorial/IC790794.png "指派使用者")
3. 選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。
   
   ![是](./media/active-directory-saas-greenhouse-tutorial/IC767830.png "是")

如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。


