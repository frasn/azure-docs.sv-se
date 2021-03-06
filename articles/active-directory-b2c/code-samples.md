---
title: Azure Active Directory B2C-kodexempel | Microsoft Docs
description: Kodexempel för Azure Active Directory B2C-appar för mobil, skrivbord och enskild sida.
services: active-directory-b2c
author: davidmu1
ms.author: davidmu
ms.date: 01/23/2018
ms.custom: mvc
ms.topic: tutorial
ms.service: active-directory-b2c
ms.openlocfilehash: 42a9ce6f8a5a04f641ccf9471151334632c9b11e
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/18/2018
---
# <a name="azure-active-directory-b2c-code-samples"></a>Azure Active Directory B2C-kodexempel

Följande tabeller innehåller länkar till exempel för program, inklusive iOS, Android, .NET och Node.js.

## <a name="mobile-and-desktop-apps"></a>Mobilappar och skrivbordsappar

| Exempel | Beskrivning |
|--------| ----------- |
| [ios-swift-native-msal](https://github.com/Azure-Samples/active-directory-b2c-ios-swift-native-msal) | Ett iOS-exempel i Swift som autentiserar Azure AD B2C-användare och anropar ett API med OAuth 2.0 |
| [android-native-msal](https://github.com/Azure-Samples/active-directory-b2c-android-native-msal) | En enkel Android-app som visar hur du använder MSAL för att autentisera användare via Azure Active Directory B2C och få åtkomst till ett webb-API med resulterande token. |
| [ios-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-ios-native-appauth) | Ett exempel som visar hur du kan använda ett bibliotek från tredje part för att skapa ett iOS-program i Objective-C som autentiserar användare med Microsoft-identitet till vår Azure AD B2C-identitetstjänst. |
| [android-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-android-native-appauth) | Ett exempel som visar hur du kan använda ett bibliotek från tredje part för att skapa en Android-app som autentiserar användare med Microsoft-identitet till vår B2C-identitetstjänst och anropar ett webb-API med OAuth 2.0-åtkomsttoken. |
| [dotnet-desktop](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) | Ett exempel som visar hur ett Windows Desktop .NET-program (WPF-program) kan logga in en användare som använder Azure AD B2C, få ett åtkomsttoken med hjälp av MSAL.NET och anropa ett API. | 
| [xamarin-intern](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | En enkel Xamarin Forms-app som visar hur du använder MSAL för att autentisera användare via Azure Active Directory B2C och få åtkomst till ett webb-API med resulterande token. |

## <a name="web-apps-and-apis"></a>Webbappar och API:er

| Exempel | Beskrivning |
|--------| ----------- |
| [dotnet-webapp-and-webapi](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) | Ett kombinerat exempel för ett .NET-webbprogram som anropar ett .NET-webb-API, båda skyddade med Azure AD B2C. |
| [dotnetcore-webapp](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapp) | Ett ASP.NET Core-webbprogram som kan logga in en användare som använder Azure AD B2C, få ett åtkomsttoken med hjälp av MSAL.NET och anropa ett API. |
| [openidconnect nodejs](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-NodeJS) | En Node.js-app som ger ett snabbt och enkelt sätt att konfigurera ett webbprogram med Express med hjälp av OpenID Connect. |
| [javascript-nodejs-webapp](https://github.com/AzureADQuickStarts/active-directory-b2c-javascript-nodejs-webapp) | En node.js-server som ger ett snabbt och enkelt sätt att konfigurera en REST API-tjänst som använder protokollet OAuth2. |
| [javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) | Ett litet node.js-webb-API för Azure AD B2C som visar hur du skyddar ditt webb-api och accepterar B2C-åtkomsttoken med passport.js. |

## <a name="single-page-apps"></a>Appar med en sida

| Exempel | Beskrivning |
|--------| ----------- |
| [javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) | En ensidesapplikation (SPA) som anropar ett webb-API. Autentiseringen utförs med Azure AD B2C genom att utnyttja MSAL.js. | 
| [javascript-hellojs-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-hellojs-singlepageapp) | En ensidesapplikation (SPA) som implementeras med en ASP.NET Web API-serverdel som registrerar och loggar in användare med Azure AD B2C och anropar webb-API:et med OAuth 2.0-åtkomsttoken. |