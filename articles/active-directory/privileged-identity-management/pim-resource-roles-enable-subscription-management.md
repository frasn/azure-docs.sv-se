---
title: Privileged Identity Management för Azure-resurser - aktivera prenumerationshantering | Microsoft Docs
description: Lär dig hur globala administratörer kan hantera prenumerationer i klienten.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/27/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 22e735fc10f314e47c00795bf993b5502f639c84
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34699809"
---
# <a name="enable-subscription-management-in-your-tenant"></a>Aktivera prenumerationshantering av i din klient

Du kanske inte har åtkomst till alla prenumerationsresurser i din klient som en global administratör i din katalog. Den här artikeln beskrivs stegen för att ge åtkomst till alla prenumerationer i din klient. Det ger också en rekommenderad metod för återstående kompatibel med eventuella säkerhetsåtgärder som din organisation kräver när du har fått åtkomst.

## <a name="who-can-enable-management-of-subscriptions-in-my-directory"></a>Vem kan aktivera hantering av prenumerationer i min katalog?

Varje användare som har tilldelats rollen global administratör måste följa stegen nedan för att aktivera prenumerationshantering av. När du har aktiverat prenumerationshantering själv kan du lägga till andra globala administratörer som kan behöva resurs åtkomstnivåerna. Det finns ingen inställning för katalogen som ger åtkomst för alla medlemmar i rollen global administratör.

## <a name="sign-in-to-the-azure-portal"></a>Logga in på Azure Portal

Logga in på Azure-portalen med ett konto som är en berättigad eller aktiv medlem i rollen global administratör. Om kontot är berättigad global administratör, måste du först aktivera rollen innan du går vidare till nästa steg.

## <a name="access-the-azure-active-directory-admin-center"></a>Åtkomst till Azure Active Directory Administrationscenter

Nu när du är inloggad på Azure-portalen som global administratör kan du redigera inställningar som ger åtkomst till Azure-prenumerationer. Bläddra till Administrationscenter för Azure Active Directory (Azure AD) och välj **egenskaper**.

![Skärmbild av Azure AD Administrationscenter med egenskaper för markerat](media/azure-pim-resource-rbac/aad_properties.png)

I listan över egenskaper, under **Global administratör kan hantera Azure-prenumerationer**väljer **Ja**.

![Skärmbild av egenskapssida med växla till Ja](media/azure-pim-resource-rbac/aad_properties_save.png)

Ditt konto läggs nu automatiskt till användare åtkomst till rollen Administratör för varje prenumeration resurs i klienten.

## <a name="browse-to-azure-ad-pim"></a>Bläddra till Azure AD PIM

 Gå till Azure AD Privileged Identity Management (PIM) från den här. Under **hantera**väljer **Azure-resurser**.

![Skärmbild av PIM med Azure-resurser markerat](media/azure-pim-resource-rbac/aadpim_manage_azure_resources.png)

## <a name="manage-and-discover-resources"></a>Hantera och identifiera resurser

Om din organisation använder redan Azure AD PIM för att skydda administratörer i Azure AD, kan du se en lista över prenumerationer när bladet läses in.

![Skärmbild av PIM med listan över prenumerationer som visas i bladet](media/azure-pim-resource-rbac/aadpim_manage_azure_resource_some_there.png)

> [!NOTE]
> Om du inte ser några resurser, kontrollerar du att:
>- Din rollen global administratör har inte upphört att gälla. 
>- Din organisation har en Azure-prenumeration.

![Skärmbild av PIM med tom resurslistan](media/azure-pim-resource-rbac/aadpim_rbac_empty_resource_list.png)

## <a name="configure-assignments"></a>Konfigurera tilldelningar

Välj en prenumeration i listan och tilldela dig själv (eller en grupp som du är medlem i) som en berättigad ägare av resursen. 
[Läs mer om hur du tilldelar roller](pim-resource-roles-assign-roles.md).

Upprepa proceduren för varje resurs innan du fortsätter till nästa steg.

## <a name="clean-up-standing-access"></a>Rensa stående åtkomst

Nu när du har berättigade tilldelningar för viktiga prenumerationer i din organisation kan du rensa stående åtkomst genom att inaktivera alternativet i katalogegenskaperna för.

![Skärmbild av egenskapssida med Växla inställt på Nej](media/azure-pim-resource-rbac/aad_properties_no.png)

## <a name="next-steps"></a>Nästa steg

[Identifiera resurser](pim-resource-roles-discover-resources.md)

[Konfigurera rollinställningar](pim-resource-roles-configure-role-settings.md)








