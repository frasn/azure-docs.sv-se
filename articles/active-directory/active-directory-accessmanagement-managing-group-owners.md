---
title: Nästa steg för hantering med hjälp av grupper | Microsoft Docs
description: Avancerade hur-för hantering av säkerhetsgrupper och hur du använder dessa grupper för att hantera åtkomst till en resurs.
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: article
ms.date: 09/12/2017
ms.author: curtand
ms.custom: it-pro
ms.openlocfilehash: 0f379c22f1bad522a2d9bff399bbf7006f7c4920
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34713287"
---
# <a name="managing-owners-for-a-group"></a>Hantera ägare för en grupp
När en resursägare har tilldelats till en resurs till en Azure AD-grupp, hanteras medlemskap i gruppen av gruppägare. Effektivt resursägaren behörighet att tilldela användare till resursen till ägare av gruppen.

## <a name="add-an-owner-to-a-group"></a>Lägg till en ägare till en grupp

1. I den [administrationscentret för Azure AD](https://aad.portal.azure.com)väljer **användare och grupper**.
2. Välj **alla grupper**, och sedan öppna den grupp som du vill lägga till ägare till.
3. Välj **lägga till ägare**.
4. På den **lägga till ägare** markerar du den användare som du vill lägga till som ägare av den här gruppen och kontrollera att namnet läggs till i **valda** fönstret.

## <a name="remove-an-owner-from-a-group"></a>Tar bort en ägare från en grupp

1. I den [administrationscentret för Azure AD](https://aad.portal.azure.com)väljer **användare och grupper**.
2. Välj **alla grupper**, och sedan öppna den grupp som du vill ta bort ägare.
3. Välj den **ägare** fliken.
4. Välj en ägare som du vill ta bort från den här gruppen och välj sedan **ta bort**.

## <a name="additional-information"></a>Ytterligare information
Dessa artiklar innehåller ytterligare information om Azure Active Directory.

* [Hantera åtkomst till resurser med Azure Active Directory-grupper](active-directory-manage-groups.md)
* [Azure Active Directory-cmdletar för att konfigurera gruppinställningar](active-directory-accessmanagement-groups-settings-cmdlets.md)
* [Artikelindex för programhantering i Azure Active Directory](active-directory-apps-index.md)
* [Vad är Azure Active Directory?](active-directory-whatis.md)
* [Integrera dina lokala identiteter med Azure Active Directory](active-directory-aadconnect.md)
