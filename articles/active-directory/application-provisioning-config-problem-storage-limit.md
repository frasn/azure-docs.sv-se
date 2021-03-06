---
title: "Problem med att spara autentiseringsuppgifter vid konfigurering av användarförsörjning till ett program för Azure AD-galleriet | Microsoft Docs"
description: "Felsökning av vanliga problem inför när Konfigurera användaretablering till ett program som redan visas i Azure AD Application Gallery"
services: active-directory
documentationcenter: 
author: asmalser-msft
manager: mtillman
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/21/2018
ms.author: asmalser
ms.openlocfilehash: 6617345c8923b1fc8081b01ddfe8b4bedf10b6ea
ms.sourcegitcommit: 088a8788d69a63a8e1333ad272d4a299cb19316e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 02/27/2018
---
# <a name="problem-saving-administrator-credentials-while-configuring-user-provisioning-to-an-azure-active-directory-gallery-application"></a>Problem med att spara autentiseringsuppgifter vid konfigurering av användarförsörjning till ett program i Azure Active Directory-galleriet 

När du använder Azure-portalen för att konfigurera [automatisk användaretablering](active-directory-saas-app-provisioning.md) för företagets program kan du stöta på en situation där:

* Den **administratörsautentiseringsuppgifter** angetts för programmet är giltiga, och **Testanslutningen** knappen fungerar. Men autentiseringsuppgifterna går inte att spara och Azure-portalen returnerar ett allmänt felmeddelande.

Om SAML-baserade enkel inloggning har konfigurerats för samma program, den troligaste orsaken till felet är att Azure AD interna, programspecifika lagringsgränsen för certifikat och autentiseringsuppgifterna har överskridits.

Azure AD för närvarande har en maximal lagringskapacitet på en kilobyte för alla certifikat, hemliga token, autentiseringsuppgifter och relaterade konfigurationsdata som är associerade med en enda instans av ett program (kallas även en service principal post i Azure AD).

När SAML-baserade enkel inloggning har konfigurerats, används för att signera SAML-token lagras här och ofta förbrukar över 50 procent av utrymmet.

Alla hemliga token, URI: er, notification e-postadresser, användarnamn och lösenord som får anges under installationen av användaretablering kan orsaka lagringsgränsen överskrids.

## <a name="how-to-work-around-this-issue"></a>Hur du löser problemet 

Det finns två möjliga sätt att komma runt problemet idag:

1. **Använda två galleriet programinstanser, en för enkel inloggning och en för användaretablering** -tar programmet galleriet [LinkedIn höjer](active-directory-saas-linkedinelevate-tutorial.md) exempelvis kan du lägga till LinkedIn höjer från galleriet och konfigurera den för enkel inloggning. Lägga till en annan instans av LinkedIn höjer från appgalleriet för Azure AD för att etablera, och ge den namnet ”LinkedIn utöka (etablering)”. Den här andra instansen, konfigurera [etablering](active-directory-saas-linkedinelevate-provisioning-tutorial.md), men inte enkel inloggning. När du använder den här lösningen, samma användare och grupper måste vara [tilldelade](active-directory-coreapps-assign-user-azure-portal.md) till båda programmen. 

2. **Minska mängden konfigurationsdata lagrad** -alla data som angetts i den [administratörsautentiseringsuppgifter](active-directory-saas-app-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) på fliken etablering lagras på samma plats som SAML-certifikat. När det inte möjligt att minska längden på all data, vissa fält i valfri konfiguration som den **e-postmeddelande** kan tas bort.

## <a name="next-steps"></a>Nästa steg
[Konfigurera användare etablering och avetablering för SaaS-program](active-directory-saas-app-provisioning.md)
