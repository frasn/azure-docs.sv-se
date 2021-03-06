---
title: Vad är gruppbaserade licensiering i Azure Active Directory? | Microsoft Docs
description: Beskrivning av Azure Active Directory gruppbaserade licensiering, hur det fungerar och bästa praxis
services: active-directory
keywords: Azure AD-licensiering
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: fundamentals
ms.topic: article
ms.workload: identity
ms.date: 05/21/2018
ms.author: curtand
ms.reviewer: piotrci
ms.custom: H1Hack27Feb2017;it-pro
ms.openlocfilehash: 6fe473d606df4ea0c37c2e6b2b1c9146056d0106
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34713481"
---
# <a name="group-based-licensing-basics-in-azure-active-directory"></a>Gruppbaserade licensiering grunderna i Azure Active Directory

Microsoft betald molntjänster, till exempel Office 365 Enterprise Mobility + Security, Dynamics 365 och andra liknande produkter kräver licenser. Dessa licenser har tilldelats till varje användare som behöver åtkomst till dessa tjänster. För att hantera licenser kan använda administratörer hanteringsportalerna (Office eller Azure) och PowerShell-cmdlets. Azure Active Directory (AD Azure) är den underliggande infrastruktur som stöder Identitetshantering för alla Microsoft-molntjänster. Azure AD lagrar information om tilldelning av licenstillstånd för användare.

Fram till nu kan licenser endast tilldelas på enskilda nivån, vilket kan göra storskaliga management svårt. Till exempel om du vill lägga till eller ta bort användarlicenser baserat på organisationens ändringar, till exempel användare ansluta till eller lämnar organisationen eller en avdelning måste administratör ofta skriva ett komplext PowerShell-skript. Det här skriptet anrop enskilda till Molntjänsten.

För att lösa dessa problem har innehåller nu Azure AD gruppbaserade licensiering. Du kan tilldela en eller flera licenser till en grupp. Azure AD säkerställer att licenserna som tilldelas alla medlemmar i gruppen. Alla nya medlemmar som kan ansluta till gruppen har tilldelats rätt licenser för. När de lämnar gruppen bort dessa licenser. Detta eliminerar behovet för att automatisera licenshantering via PowerShell så att ändringar i organisationen och avdelningsnivå struktur per användare.

>[!NOTE]
>Den här funktionen är för närvarande i förhandsversion. Var beredd på att återställa eller ta bort alla ändringar. Funktionen är tillgänglig i alla Azure Active Directory (Azure AD)-prenumeration under förhandsversion. När funktionen blir allmänt tillgänglig, kan vissa aspekter av funktionen kräver en eller flera Azure Active Directory Premium-licenser.

## <a name="features"></a>Funktioner

Här är de viktigaste funktionerna i gruppbaserade licensiering:

- Licenserna kan tilldelas till någon säkerhetsgrupp i Azure AD. Säkerhetsgrupper kan vara synkroniserade lokalt, med hjälp av Azure AD Connect. Du kan också skapa säkerhetsgrupper direkt i Azure AD (kallas även endast molnbaserad grupper) eller automatiskt via funktionen Azure AD dynamisk grupp.

- När en licens tilldelas till en grupp, kan administratören inaktivera en eller flera serviceplaner i produkten. Detta görs vanligtvis när organisationen inte ännu är redo att börja med en tjänst som ingår i en produkt. Administratören kan till exempel tilldela en avdelning Office 365 men tillfälligt inaktivera Yammer-tjänsten.

- Alla Microsoft-molntjänster som kräver användarnivå licensiering stöds. Detta omfattar alla produkter, Enterprise Mobility + Security och Dynamics 365 i Office 365.

- Gruppbaserade licensiering finns för närvarande endast via [Azure-portalen](https://portal.azure.com). Om du använder andra hanteringsportalerna främst för användare och grupp management, till exempel Office 365-portalen kan du kan fortsätta att göra detta. Men du bör använda Azure-portalen för att hantera licenser på gruppnivå.

- Licens-ändringar som uppstår när ändringar i gruppmedlemskap hanteras automatiskt i Azure AD. Licens ändringar normalt effektiva minuter efter en ändring av gruppmedlemskap.

- En användare kan vara medlem i flera grupper med licens principer som har angetts. En användare kan också ha vissa licenser som direkt har tilldelats, utanför grupper. Det resulterande användartillståndet är en kombination av alla tilldelade produkt och tjänstlicenser.

- I vissa fall kan inte licenser tilldelas till en användare. Exempelvis kanske det inte tillräckligt många tillgängliga licenser i klienten eller tjänster i konflikt kanske har tilldelats på samma gång. Administratörer har åtkomst till information om användare för vilka Azure AD inte kunde fullständigt bearbeta grupp licenser. De kan sedan vidta åtgärder baserat på informationen.

- Under förhandsversion krävs en betald eller utvärderingsversioner prenumeration för Azure AD Basic eller Premium-versioner i klienten att använda gruppbaserade licenshantering.

## <a name="your-feedback-is-welcome"></a>Din feedback är Välkommen!

Om du har feedback eller funktionen begäranden kan du dela dem med oss via [i Azure AD admin-forumet](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=162510).

## <a name="next-steps"></a>Nästa steg

Mer information om andra scenarier för licenshantering via gruppbaserade licensiering finns:

* [Kom igång med Azure Active Directory-licenser](active-directory-licensing-get-started-azure-portal.md)
* [Tilldela licenser till en grupp i Azure Active Directory](active-directory-licensing-group-assignment-azure-portal.md)
* [Identifiera och lösa eventuella problem med licens för en grupp i Azure Active Directory](active-directory-licensing-group-problem-resolution-azure-portal.md)
* [Hur du migrerar enskilda licensierade användare till gruppbaserade licensiering i Azure Active Directory](active-directory-licensing-group-migration-azure-portal.md)
* [Azure Active Directory gruppbaserade licensiering fler scenarier](active-directory-licensing-group-advanced.md)
