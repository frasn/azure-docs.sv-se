---
title: ta med fil
description: ta med fil
services: active-directory
documentationcenter: dev-center-name
author: andretms
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/19/2018
ms.author: andret
ms.custom: include file
ms.openlocfilehash: cf6604a0e22ca72c8aabd0603e42469cc71c9680
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2018
---
# <a name="add-sign-in-with-microsoft-to-an-aspnet-web-app"></a>Lägga till inloggning med Microsoft till ett ASP.NET-webbprogram

Den här guiden visar hur du implementerar inloggning med Microsoft med hjälp av en ASP.NET MVC-lösning med en traditionell webbläsarbaserad webbapp med OpenID Connect. 

I slutet av den här guiden kommer programmet att kunna acceptera logga moduler av personliga konton (inklusive outlook.com och live.com) som fungerar och skolkonton från alla företag eller organisation som har integrerat med Azure Active Directory. 

> Den här guiden kräver Visual Studio 2015 Update 3 eller Visual Studio 2017.  Har det?  [Hämta Visual Studio 2017 gratis](https://www.visualstudio.com/downloads/)

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Så här fungerar sample-appen som genererats av den här guiden

![Hur den här guiden fungerar](media/active-directory-develop-guidedsetup-aspnetwebapp-intro/aspnetbrowsergeneral.png)

Det exempelprogram som skapats av den här guiden är baserad på ett scenario där en användare använder webbläsaren för att komma åt en ASP.NET-webbplats som begär en användare autentiseras via en knapp för inloggning. I det här scenariot inträffar mesta av arbetet ska renderas webbsidan på serversidan.

## <a name="libraries"></a>Bibliotek

Den här guiden använder följande bibliotek:

|Bibliotek|Beskrivning|
|---|---|
|[Microsoft.Owin.Security.OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/)|Mellanprogram som aktiverar ett program att använda OpenIdConnect för autentisering|
|[Microsoft.Owin.Security.Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies)|Mellanprogram som aktiverar ett program att underhålla användarsession med hjälp av cookies|
|[Microsoft.Owin.Host.SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb)|Gör att OWIN-baserade program körs på IIS med hjälp av ASP.NET förfrågnings-pipelinen|

