---
title: Självstudier – Bevilja åtkomst till ett Node.js-webb-API från en skrivbordsapp med Azure Active Directory B2C | Microsoft Docs
description: Självstudiekurs som visar hur du använder Active Directory B2C för att skydda ett Node.js-webb-API och anropar det från en .NET-skrivbordsapp.
services: active-directory-b2c
author: PatAltimore
ms.author: patricka
ms.reviewer: parja
ms.date: 3/01/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory-b2c
ms.openlocfilehash: ef92a7fbbc3b84195eff251b0224cb0fd51a6c88
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2018
---
# <a name="tutorial-grant-access-to-a-nodejs-web-api-from-a-desktop-app-using-azure-active-directory-b2c"></a>Självstudier: Bevilja åtkomst till ett Node.js-webb-API från en skrivbordsapp med Azure Active Directory B2C

I den här självstudiekursen ser du hur du kan anropa en Node.js-webb-API-resurs som skyddas av Azure Active Directory (Azure AD) B2C från en WPF-skrivbordsapp (Windows Presentation Foundation).

I den här guiden får du lära dig att:

> [!div class="checklist"]
> * Registrera ett webb-API i en Azure AD B2C-klientorganisation
> * Definiera och konfigurera omfång för ett webb-API
> * Tilldela behörighet till webb-API
> * Uppdatera exempelkod och använder Azure AD B2C för att skydda ett webb-API

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Nödvändiga komponenter

* Slutför [självstudiekursen för autentisering av användare med Azure Active Directory B2C i en skrivbordsapp](active-directory-b2c-tutorials-desktop-app.md).
* Installera [Visual Studio 2017](https://www.visualstudio.com/downloads/) med **.NET-skrivbordsutveckling** och arbetsbelastningarna **ASP.NET och webbutveckling**.
* Installera [Node.js](https://nodejs.org/en/download/)

## <a name="register-web-api"></a>Registrera webb-API

Webb-API-resurser måste vara registrerade i klientorganisationen innan de kan godkänna och svara på en [begäran från en skyddad resurs](../active-directory/develop/active-directory-dev-glossary.md#resource-server) från [klientprogram](../active-directory/develop/active-directory-dev-glossary.md#client-application) som använder en [åtkomsttoken](../active-directory/develop/active-directory-dev-glossary.md#access-token) från Azure Active Directory. Registrering skapar [programmet och tjänstens huvudnamnsobjekt](../active-directory/develop/active-directory-dev-glossary.md#application-object) i klientorganisationen. 

Logga in på [Azure Portal](https://portal.azure.com/) som global administratör för Azure AD B2C-klientorganisationen.

[!INCLUDE [active-directory-b2c-switch-b2c-tenant](../../includes/active-directory-b2c-switch-b2c-tenant.md)]

1. Välj **Azure AD B2C** i listan över tjänster i Azure Portal.

2. I B2C-inställningarna klickar du på **Program** och sedan på **Lägg till**.

    Registrera webb-API-exemplet i klientorganisationen med följande inställningar.
    
    ![Lägga till ett nytt API](media/active-directory-b2c-tutorials-desktop-app-webapi/web-api-registration.png)
    
    | Inställning      | Föreslaget värde  | Beskrivning                                        |
    | ------------ | ------- | -------------------------------------------------- |
    | **Namn** | My sample Node.js web API | Ange ett **Namn** som beskriver ditt webb-API för utvecklare. |
    | **Ta med webbapp/webb-API** | Ja | Välj **Ja** om det är ett webb-API. |
    | **Tillåt implicit flöde** | Ja | Välj **Ja** eftersom API:et använder [OpenID Connect-inloggning](active-directory-b2c-reference-oidc.md). |
    | **Svarswebbadress** | `http://localhost:5000` | Svarswebbadresser är slutpunkter där Azure AD B2C returnerar de token som API:et begär. I den här självstudien körs webb-API-exemplet lokalt (localhost) och lyssnar på port 5000. |
    | **URI för app-id** | demoapi | URI:n identifierar API:et i klientorganisationen. Det gör att flera API:er kan registreras per klientorganisation. [Omfång](../active-directory/develop/active-directory-dev-glossary.md#scopes) styr åtkomsten till den skyddade API-resursen och definieras med URI:n för app-ID. |
    | **Inbyggd klient** | Nej | Eftersom det här är ett webb-API och inte en intern klient väljer du Nej. |
    
3. Klicka på **Skapa** för att registrera API:et.

Registrerade API visas i programlistan för Azure AD B2C-klientorganisationen. Välj webb-API i listan. Webb-API:ets egenskapsruta visas.

![Egenskaper för webb-API](./media/active-directory-b2c-tutorials-web-api/b2c-web-api-properties.png)

Anteckna det **Programklients-id** som visas. Detta ID identifierar API:et och behövs när API:et konfigureras senare under självstudierna.

När webb-API:et registreras i Azure AD B2C skapas ett förtroende. Eftersom API:et är registrerat med B2C kan API:et nu lita på de B2C-åtkomsttoken det får från andra program.

## <a name="define-and-configure-scopes"></a>Definiera och konfigurera omfång

[Omfång](../active-directory/develop/active-directory-dev-glossary.md#scopes) är ett sätt att styra åtkomst till skyddade resurser. Omfång används av webb-API för att implementera omfångsbaserad åtkomststyrning. Vissa användare kan till exempel ha både läs- och skrivåtkomst medan andra bara har skrivskyddad åtkomst. I den här självstudiekursen definierar du läs- och skrivrättigheter för webb-API.

### <a name="define-scopes-for-the-web-api"></a>Definiera omfång för webb-API

Registrerade API visas i programlistan för Azure AD B2C-klientorganisationen. Välj webb-API i listan. Webb-API:ets egenskapsruta visas.

Klicka på **Publicerade områden (förhandsgranskning)**.

Konfigurera omfång för API:t genom att lägga till följande poster. 

![omfång definierade i webb-api](media/active-directory-b2c-tutorials-web-api/scopes-defined-in-web-api.png)

| Inställning      | Föreslaget värde  | Beskrivning                                        |
| ------------ | ------- | -------------------------------------------------- |
| **Omfång** | demo.read | Läsåtkomst till demo-API|

Klicka på **Spara**.

De publicerade omfången kan användas för att tilldela behörighet för webb-API till ett klientprogram.

### <a name="grant-app-permissions-to-web-api"></a>Tilldela appbehörighet till webb-API

Om du vill anropa ett skyddat webb-API från en app måste du ge appen åtkomst till API:t. I den här självstudiekursen används skrivbordsappen som skapades i [självstudien för autentisering av användare med Azure Active Directory B2C i en skrivbordsapp](active-directory-b2c-tutorials-desktop-app.md).

1. I Azure Portal väljer du **Azure AD B2C** från listan med tjänster och klickar på **Program** för att visa den registrerade applistan.

2. Välj **Min exempel-WPF-app** från applistan. Klicka på **API-åtkomst (förhandsversion)** och sedan på **Lägg till**.

3. I listrutan **Välj API** väljer du det registrerade web-API:et **My Sample Node.js web API**.

4. I listrutan **Välj områden** väljer du de omfång du definierade vid registreringen av webb-API:t.

    ![välja omfång för app](media/active-directory-b2c-tutorials-web-api/selecting-scopes-for-app.png)

5. Klicka på **OK**.

Din **My Sample Web App** är registrerad att anropa det skyddade **My Sample Web API**. En användare [autentiserar](../active-directory/develop/active-directory-dev-glossary.md#authentication) med Azure AD B2C för att använda WPF-skrivbordsappen. Skrivbordsappen får ett [auktoriseringsbeviljande](../active-directory/develop/active-directory-dev-glossary.md#authorization-grant) från Azure AD B2C som ger tillgång till det skyddade webb-API:et.

## <a name="update-web-api-code"></a>Uppdatera webb-API-koden

När webb-API:et är registrerat och har ett definierat omfång måste du konfigurera webb-API-koden så den använder din Azure AD B2C-klientorganisation. I den här självstudien får du konfigurera en Node.js-webbapp som du kan ladda ned från GitHub. 

[Ladda ned en zip-fil](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi/archive/master.zip) eller klona exempelwebbappen från GitHub.

```
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```
Exempelwebb-API:et för Node.js använder Passport.js-biblioteket för att aktivera Azure AD B2C att skydda anrop till API: et. 

### <a name="configure-the-web-api"></a>Konfigurera webb-API

1. Öppna filen `index.html` i Node.js-webb-API-exemplet.
2. Konfigurera exemplet med registreringsinformation för Azure AD B2C-klientorganisationen. Ändra följande rader med kod:

```nodejs
var tenantID = "<your-tenant-name>.onmicrosoft.com";
var clientID = "<Application ID for your Node.js Web API>";
var policyName = "B2C_1_SiUpIn";  // Sign-in / sign-up policy name
```

### <a name="configure-the-desktop-app"></a>Konfigurera skrivbordsappen

1. Öppna `active-directory-b2c-wpf`-lösningen från [självstudiekursen för autentisering av användare med Azure Active Directory B2C i en skrivbordsapp](active-directory-b2c-tutorials-desktop-app.md) i Visual Studio.

## <a name="run-the-sample"></a>Kör exemplet

Kör webb-API:et för Node.js:

1. Starta en kommandotolk för Node.js.
2. Gå till den katalog som innehåller Node.js-exemplet. Till exempel `cd c:\active-directory-b2c-javascript-nodejs-webapi`
3. Kör följande kommandon:
    ```
    npm install && npm update
    ```
    ```
    node index.js
    ```
Kör skrivbordsappen:

1. Tryck på **F5** för att köra skrivbordsappen.
2. Logga in med e-postadressen och lösenordet som skapades i [självstudien för autentisering av användare med Azure Active Directory B2C i en skrivbordsapp](active-directory-b2c-tutorials-desktop-app.md).
3. Klicka på knappen **Anropa API**. 

Skrivbordsappen gör en begäran till webb-API:et och får ett svar med den inloggade användarens visningsnamn. Den skyddade skrivbordsappen anropar det skyddade webb-API:et i Azure AD B2C-klientorganisationen.

## <a name="clean-up-resources"></a>Rensa resurser

Du kan använda Azure AD B2C-klientorganisationen om du vill prova andra självstudier för Azure AD B2C. När den inte längre behövs kan du ta bort [Azure AD B2C-klientorganisationen](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant).

## <a name="next-steps"></a>Nästa steg

Den här artikeln visade hur du kan skydda ett ASP.NET webb-API genom att registrera och definiera omfång i Azure AD B2C. Lär dig mer genom att titta på tillgängliga Azure AD B2C-kodexempel.

> [!div class="nextstepaction"]
> [Kodexempel för Azure AD B2C](https://azure.microsoft.com/resources/samples/?service=active-directory-b2c&sort=0)
