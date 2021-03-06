---
title: "逐步解說︰以 REST API 宣告交換作為 B2C 自訂原則中的步驟 | Microsoft Docs"
description: "Azure Active Directory B2C 自訂原則與 API 整合的主題"
services: active-directory-b2c
documentationcenter: 
author: rojasja
manager: krassk
editor: rojasja
ms.assetid: 
ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: article
ms.devlang: na
ms.date: 04/24/2017
ms.author: joroja
ms.translationtype: Human Translation
ms.sourcegitcommit: 2db2ba16c06f49fd851581a1088df21f5a87a911
ms.openlocfilehash: 27d066cc6f985565a765989837e4a2744387a628
ms.contentlocale: zh-tw
ms.lasthandoff: 05/09/2017

---

# <a name="walkthrough-integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-user-journeys-as-an-orchestration-step"></a>逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程中以作為協調流程步驟

構成 Azure AD B2C 基礎的**身分識別體驗架構** (IEF) 可讓身分識別開發人員於使用者旅程中整合與 RESTful API 的互動。  

在本逐步解說結束時，您將可以建立與 RESTful 服務互動的 Azure AD B2C 使用者旅程。

IEF 會在宣告中傳送資料，並在宣告中收到傳回的資料。  REST API 宣告交換可設計成協調流程步驟。

- 這可以觸發外部動作，例如，它可以在外部資料庫中記錄一筆事件。
- 這也可用來擷取值，並接著將它儲存在使用者資料庫中。
- 接收的宣告稍後可用來變更執行流程。

互動也可以設計成驗證設定檔。 如需這方面的相關資訊，請參閱[逐步解說︰將 REST API 宣告交換整合到 Azure AD B2C 使用者旅程中以作為使用者輸入的驗證](active-directory-b2c-rest-api-validation-custom.md)。

情況是，當使用者執行設定檔編輯時，我們會想要在外部系統中查閱使用者、取得該使用者的註冊城市，並將該屬性傳回作為回送給應用程式的宣告。

## <a name="prerequisites"></a>必要條件

- 如[開始使用](active-directory-b2c-get-started-custom.md)所述，設定為完成本機帳戶註冊/登入的 Azure AD B2C 租用戶。
- 要互動的 REST API 端點，本逐步解說使用非常簡單的 Azure 函式應用程式 WebHook 來作為範例
- **建議**︰完成 [REST API 宣告交換逐步解說以作為驗證步驟](active-directory-b2c-rest-api-validation-custom.md)。

## <a name="step-1---prepare-the-rest-api-function"></a>步驟 1 - 準備 REST API 函式

> [!NOTE]
> REST API 函式的設定不在本文討論範圍內。 [Azure 函式應用程式](https://docs.microsoft.com/azure/azure-functions/functions-reference)提供了絕佳的工具組，供您在雲端建立 RESTful 服務。

我們已設定了 Azure 函式，以接收宣告 `email`，並只傳回指派值為 `Redmond` 的宣告 `city`。 Azure 函式的範例在此︰[GitHub](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/AzureFunctionsSamples)

在此內容中，Azure 函式所傳回的 `userMessage` 宣告是選擇性的，IEF 會予以略過。  它有可能無法作為訊息，以在稍後傳遞給應用程式並向使用者呈現。

```csharp
if (requestContentAsJObject.email == null)
{
    return request.CreateResponse(HttpStatusCode.BadRequest);
}

var email = ((string) requestContentAsJObject.email).ToLower();

return request.CreateResponse<ResponseContent>(
    HttpStatusCode.OK,
    new ResponseContent
    {
        version = "1.0.0",
        status = (int) HttpStatusCode.OK,
        userMessage = "User Found",
        city = "Redmond"
    },
    new JsonMediaTypeFormatter(),
    "application/json");
```

**Azure 函式應用程式**可讓您輕鬆地取得函式 URL，此 URL 中包含特定函式的識別碼。  在此案例中，URL 是︰https://wingtipb2cfuncs.azurewebsites.net/api/LookUpLoyaltyWebHook?code=MQuG7BIE3eXBaCZ/YCfY1SHabm55HEphpNLmh1OP3hdfHkvI2QwPrw==，您可以用此 URL 來進行測試。

## <a name="step-2---configure-the-restful-api-claims-exchange-as-a-technical-profile-in-your-trustframeworextensionsxml-file"></a>步驟 2 - 在 TrustFrameworExtensions.xml 檔案中將 RESTful API 宣告交換設定為技術設定檔

技術設定檔是 RESTful 服務所需之交換的完整設定。 開啟 `TrustFrameworkExtensions.xml` 檔案，並將下列 XML 程式碼片段新增到 `<ClaimsProvider>` 元素內。

> [!NOTE]
> 請考慮使用以下所述的「Restful 提供者 1.0.0.0 版」來作為會與外部服務互動之函式的通訊協定。  您可以找到完整的結構描述定義 <!-- TODO: Link to RESTful Provider schema definition>-->

```XML
<ClaimsProvider>
    <DisplayName>REST APIs</DisplayName>
    <TechnicalProfiles>
        <TechnicalProfile Id="AzureFunctions-LookUpLoyaltyWebHook">
            <DisplayName>Check LookUpLoyalty Web Hook Azure Function</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
                <Item Key="ServiceUrl">https://wingtipb2cfuncs.azurewebsites.net/api/LookUpLoyaltyWebHook?code=MQuG7BIE3eXBaCZ/YCfY1SHabm55HEphpNLmh1OP3hdfHkvI2QwPrw==</Item>
                <Item Key="AuthenticationType">None</Item>
                <Item Key="SendClaimsIn">Body</Item>
            </Metadata>
            <InputClaims>
                <InputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="email" />
            </InputClaims>
            <OutputClaims>
                <OutputClaim ClaimTypeReferenceId="city" PartnerClaimType="city" />
            </OutputClaims>
            <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
    </TechnicalProfiles>
</ClaimsProvider>
```

`<InputClaims>` 元素會定義將從 IEF 傳送至 REST 服務的宣告。 在上述範例中，`givenName` 宣告的內容將會傳送至 REST 服務來作為 `email` 宣告。  

`<OutputClaims>` 元素會定義 IEF 預期要從 REST 服務收到的宣告。 不論收到多少宣告，IEF 只會使用這裡所識別的宣告。 在此範例中，以 `city` 形式所收到的宣告會對應到 IEF 宣告 `city`。

## <a name="step-3---add-a-new-claim-city-to-the-schema-of-your-trustframeworkextensionsxml-file"></a>步驟 3 - 將新的宣告 `city` 新增至 TrustFrameworkExtensions.xml 檔案的結構描述

宣告 `city` 並未在結構描述的其他地方另外定義。 因此，我們會在 `<BuildingBlocks>` 元素內新增定義，您可以在 `TrustFrameworkExtensions.xml` 檔案的開頭找到此元素。

```XML
<BuildingBlocks>
    <!--The claimtype city must be added to the TrustFrameworkPolicy-->
    <!-- You can add new claims in the BASE file Section III, or in the extensions file-->
    <ClaimsSchema>
        <ClaimType Id="city">
            <DisplayName>City</DisplayName>
            <DataType>string</DataType>
            <UserHelpText>Your city</UserHelpText>
            <UserInputType>TextBox</UserInputType>
        </ClaimType>
    </ClaimsSchema>
</BuildingBlocks>
```

## <a name="step-4---include-the-rest-service-claims-exchange-as-an-orchestration-step-in-your-profile-edit-user-journey-in-your-trustframeworkextensionsxml"></a>步驟 4 - 在 TrustFrameworkExtensions.xml 的設定檔編輯使用者旅程中，將 REST 服務宣告交換納入為協調流程步驟

我們已經決定要在使用者通過驗證之後新增設定檔編輯使用者旅程的步驟 (協調流程步驟 1-4 - 請見下面)，而且使用者已提供更新的設定檔資訊 (步驟 5)。

> [!NOTE]
> 有許多使用案例都可將 REST API 呼叫作為協調流程步驟。  作為協調流程步驟，它可在使用者成功完成工作 (例如首次註冊) 後作為外部系統的更新，或是作為設定檔更新以讓資訊保持同步。  在此情況下，它會用來加強設定檔編輯之後所提供給應用程式的資訊。

將設定檔編輯使用者旅程 XML 程式碼從 `TrustFrameworkBase.xml` 檔案複製到 `<UserJourneys>` 元素內的 `TrustFrameworkExtensions.xml` 檔案，然後在步驟 6 下進行修改。

```XML
<OrchestrationStep Order="6" Type="ClaimsExchange">
    <ClaimsExchanges>
        <ClaimsExchange Id="GetLoyaltyData" TechnicalProfileReferenceId="AzureFunctions-LookUpLoyaltyWebHook" />
    </ClaimsExchanges>
</OrchestrationStep>
```

> [!IMPORTANT]
> 如果操作順序不符合您的版本，請確定您是和該步驟一樣在 ClaimsExchange 類型 `SendClaims` 前插入

最後的 UserJourney XML 看起來應該像這樣︰

```XML
<UserJourney Id="ProfileEdit">
    <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
            <ClaimsProviderSelections>
                <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange" />
                <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSigninEmailExchange" />
            </ClaimsProviderSelections>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
                <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="3" Type="ClaimsExchange">
            <Preconditions>
                <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
                    <Value>authenticationSource</Value>
                    <Value>localAccountAuthentication</Value>
                    <Action>SkipThisOrchestrationStep</Action>
                </Precondition>
            </Preconditions>
            <ClaimsExchanges>
                <ClaimsExchange Id="AADUserRead" TechnicalProfileReferenceId="AAD-UserReadUsingAlternativeSecurityId" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="4" Type="ClaimsExchange">
            <Preconditions>
                <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
                    <Value>authenticationSource</Value>
                    <Value>socialIdpAuthentication</Value>
                    <Action>SkipThisOrchestrationStep</Action>
                </Precondition>
            </Preconditions>
            <ClaimsExchanges>
                <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="5" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="B2CUserProfileUpdateExchange" TechnicalProfileReferenceId="SelfAsserted-ProfileUpdate" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <!-- Add a step 6 to the user journey before the jwt token is created-->
        <OrchestrationStep Order="6" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="GetLoyaltyData" TechnicalProfileReferenceId="AzureFunctions-LookUpLoyaltyWebHook" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    </OrchestrationSteps>
    <ClientDefinition ReferenceId="DefaultWeb" />
</UserJourney>
```

## <a name="step-5---add-the-claim-city-to-your-relying-party-policy-file-so-the-claim-is-sent-to-your-application"></a>步驟 5 - 將宣告「city」新增至信賴憑證者原則檔案，以便宣告會傳送至您的應用程式

若要這樣做，請編輯您的 `ProfileEdit.xml` RP 檔案，並修改 `<TechnicalProfile Id="PolicyProfile">` 元素以新增下列項目︰`<OutputClaim ClaimTypeReferenceId="city" />`。

在新增宣告之後，TechnicalProfile 看起來像這樣︰

```XML
<DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="city" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
</TechnicalProfile>
```

## <a name="step-6---upload-your-changes-and-test"></a>步驟 6 - 上傳您的變更和測試

您將會覆寫現有的原則版本。

1.    (選擇性) 在繼續之前，請先儲存擴充檔案的現有版本 (透過下載)。  建議您不要上傳多個擴充檔案版本，以免一開始的複雜性太高。
2.    (選擇性) 您可以藉由變更 PolicyId="B2C_1A_TrustFrameworkProfileEdit"，來重新命名原則編輯檔案 PolicyId 的新版本
3.    上傳擴充檔案
4.    上傳原則編輯信賴憑證者檔案
5.    使用 [立即執行] 來測試原則。  檢閱 IEF 傳回給應用程式的權杖。

如果一切設定皆正確，權杖會包含新的宣告 `city`，且值為 `Redmond`。

```JSON
{
  "exp": 1493053292,
  "nbf": 1493049692,
  "ver": "1.0",
  "iss": "https://login.microsoftonline.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "sub": "a58e7c6c-7535-4074-93da-b0023fbaf3ac",
  "aud": "4e87c1dd-e5f5-4ac8-8368-bc6a98751b8b",
  "acr": "b2c_1a_trustframeworkprofileedit",
  "nonce": "defaultNonce",
  "iat": 1493049692,
  "auth_time": 1493049692,
  "city": "Redmond"
}
```

## <a name="next-steps"></a>後續步驟

[使用 REST API 來作為驗證步驟](active-directory-b2c-rest-api-validation-custom.md)

[如何修改設定檔編輯以從使用者收集其他資訊](active-directory-b2c-create-custom-attributes-profile-edit-custom.md)

