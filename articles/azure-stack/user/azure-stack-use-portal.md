---
title: Med hjälp av Azure Stack-portalen | Microsoft Docs
description: Lär dig hur du använder användarportalen i Azure-stacken.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/01/2018
ms.author: mabrigg
ms.reviewer: efemmano
ms.openlocfilehash: 7ca29ee359349f69c3d5ff21bd9db3f93358206a
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34724320"
---
# <a name="use-the-azure-stack-portal"></a>Använd Azure Stack-portalen

*Gäller för: Azure Stack integrerat system och Azure-stacken Development Kit*

Du kan använda Azure Stack-portalen för att prenumerera på offentliga erbjudanden och använder de tjänster som dessa erbjudanden tillhandahåller. Om du har använt den globala Azure-portalen kan är du redan bekant med hur webbplatsen fungerar.

## <a name="access-the-portal"></a>Åtkomst till portalen

Azure Stack-operatorn (en tjänsteleverantör eller en administratör i din organisation), informerar dig om att få åtkomst till portalen rätt URL.

- För ett integrerat system URL: en varierar beroende på din operatorn region och externa domännamn och ska vara i formatet https://portal.&lt; *region*&gt;.&lt; *FQDN*&gt;.
- Om du använder Azure Stack Development Kit portal adressen är https://portal.local.azurestack.external.

![Skärmdump av Azure-stacken användarportalen](media/azure-stack-use-portal/UserPortal.png)

## <a name="customize-the-dashboard"></a>Anpassa instrumentpanelen

Instrumentpanelen innehåller en standarduppsättning paneler. Du kan välja **redigera instrumentpanel** ändra standardinstrumentpanelen eller välj **ny instrumentpanel** att skapa en anpassad instrumentpanel. Du kan enkelt anpassa en instrumentpanel genom att lägga till eller ta bort paneler. Om du vill lägga till en beräknings-panelen, väljer du exempelvis **ny**. Högerklicka på **Compute**, och välj sedan **fäst på instrumentpanelen**.

## <a name="create-subscription-and-browse-available-resources"></a>Skapa prenumeration och bläddra bland tillgängliga resurser

Om du inte redan har en prenumeration, är det första du behöver göra prenumererar på ett erbjudande. Sedan kan bläddra du tillgängliga resurser. Bläddra och skapa resurser genom att använda någon av följande metoder:

- Välj den **Marketplace** panelen på instrumentpanelen.
- På den **alla resurser** panelen, väljer **skapa resurser**.
- I det vänstra navigeringsfönstret, Välj **ny**.

## <a name="learn-how-to-use-available-services"></a>Lär dig hur du använder tillgängliga tjänster

Om du behöver vägledning för hur du använder tillgängliga tjänster kan finnas det olika alternativ tillgängliga för dig.

- Din organisation eller en tjänsteleverantör får sin egen dokumentation som normalt är fallet om de erbjuder anpassade tjänster eller program.
- Appar från tredje part har sin egen dokumentation.
- Konsekvent Azure services rekommenderar vi att du först granska dokumentationen för Azure-stacken. Välj ikonen Hjälp för att komma åt Azure Stack användardokumentation, och välj sedan **hjälp + support**.

    ![Hjälp och support alternativet i Användargränssnittet](media/azure-stack-use-portal/HelpAndSupport.png)

    I synnerhet föreslår vi att du läser igenom följande artiklar för att komma igång:

    - [Nyckeln överväganden: använda tjänster eller utveckla appar för Azure-Stack](azure-stack-considerations.md)
    - I den **använder tjänster** avsnittet av dokumentationen är en artikel i överväganden för varje tjänst. Sidan överväganden beskrivs skillnaderna mellan tjänsten erbjuds i Azure och samma tjänst som erbjuds i Azure-stacken. Ett exempel finns [VM överväganden](azure-stack-vm-considerations.md). Det kan finnas andra uppgifter i den **använder tjänster** avsnitt som är unik för Azure-stacken.

      Du kan använda i Azure-dokumentationen som allmän referens för en tjänst, men du måste vara medveten om dessa skillnader. Förstå att dokumentationen länkar på den **Quickstart självstudier** panelen pekar på Azure-dokumentationen.

## <a name="get-support"></a>Få support

Om du behöver support kontaktar du din organisation eller service provider för att få hjälp.

Om du använder Azure Stack Development Kit den [Azure Stack-forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack) är den enda källan för support.

## <a name="next-steps"></a>Nästa steg

[Nyckeln överväganden: använda tjänster eller utveckla appar för Azure-Stack](azure-stack-considerations.md)
