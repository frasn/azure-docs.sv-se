---
title: 'Självstudier: Konfigurera DocuSign för automatisk användaretablering med Azure Active Directory | Microsoft Docs'
description: Lär dig hur du konfigurerar enkel inloggning mellan Azure Active Directory och DocuSign.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 294cd6b8-74d7-44bc-92bc-020ccd13ff12
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 644d511dc65370c2ec81b4677940bc115da76d29
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/20/2018
---
# <a name="tutorial-configure-docusign-for-automatic-user-provisioning"></a>Självstudier: Konfigurera DocuSign för automatisk användaretablering

Syftet med den här kursen är att visa de steg som du behöver göra i DocuSign och Azure AD för att automatiskt etablera och avetablera användarkonton från Azure AD till DocuSign.

## <a name="prerequisites"></a>Förutsättningar

Det scenario som beskrivs i den här kursen förutsätter att du redan har följande objekt:

*   En Azure Active directory-klient.
*   Aktivera prenumerationen en DocuSign enkel inloggning.
*   Ett användarkonto i DocuSign Team administratörsbehörigheter.

## <a name="assigning-users-to-docusign"></a>Tilldela användare till DocuSign

Azure Active Directory använder ett begrepp som kallas ”tilldelningar” för att avgöra vilka användare ska få åtkomst till valda appar. I samband med automatisk konto användaretablering, synkroniseras de användare och grupper som har ”tilldelats” till ett program i Azure AD.

Innan du konfigurerar och aktiverar tjänsten etablering, måste du bestämma vilka användare och/eller grupper i Azure AD representerar de användare som behöver åtkomst till appen DocuSign. När bestämt, kan du tilldela dessa användare i appen DocuSign genom att följa anvisningarna här:

[Tilldela en användare eller grupp till en enterprise-app](https://docs.microsoft.com/azure/active-directory/active-directory-coreapps-assign-user-azure-portal)

### <a name="important-tips-for-assigning-users-to-docusign"></a>Viktiga tips för att tilldela användare till DocuSign

*   Vi rekommenderar att en enda Azure AD-användare har tilldelats DocuSign för att testa allokering konfigurationen. Ytterligare användare kan tilldelas senare.

*   När du tilldelar en användare DocuSign, måste du välja en giltig användarroll. Rollen ”standard åtkomst” fungerar inte för etablering.

> [!NOTE]
> Azure AD stöder inte grupp etablering med programmet Docusign, enbart användare som kan etableras.

## <a name="enable-user-provisioning"></a>Aktivera etablering av användare

Det här avsnittet hjälper dig att ansluta din Azure AD till Docusigns användarkonto API-etablering och konfigurera tjänsten etablering för att skapa, uppdatera och inaktivera tilldelade användarkonton i DocuSign baserat på tilldelning av användare och grupper i Azure AD.

> [!Tip]
> Du kan också välja att aktivera SAML-baserade enkel inloggning för DocuSign, följer du instruktionerna som anges i [Azure-portalen](https://portal.azure.com). Enkel inloggning kan konfigureras oberoende av Automatisk etablering, även om dessa två funktioner komplettera varandra.

### <a name="to-configure-user-account-provisioning"></a>Konfigurera användaretablering för kontot:

Syftet med det här avsnittet är att beskriva hur du aktiverar användaretablering för Active Directory-användarkonton till DocuSign.

1. I den [Azure-portalen](https://portal.azure.com), bläddra till den **Azure Active Directory > Företagsappar > alla program** avsnitt.

2. Om du redan har konfigurerat DocuSign för enkel inloggning, söka efter din instans av DocuSign med hjälp av sökfältet. Annars väljer **Lägg till** och Sök efter **DocuSign** i programgalleriet. Välj DocuSign i sökresultatet och lägga till den i listan med program.

3. Välj din instans av DocuSign och sedan den **etablering** fliken.

4. Ange den **Etableringsläge** till **automatisk**. 

    ![etablering](./media/active-directory-saas-docusign-provisioning-tutorial/provisioning.png)

5. Under den **administratörsautentiseringsuppgifter** och ange följande inställningar:
   
    a. I den **administratörsanvändarnamnet** textruta typen en DocuSign kontonamn som har den **systemadministratören** profil i DocuSign.com som tilldelats.
   
    b. I den **adminlösenord** textruta skriver du lösenordet för det här kontot.

6. I Azure-portalen klickar du på **Testanslutningen** så Azure AD kan ansluta till din DocuSign app.

7. I den **e-postmeddelande** anger du den e-postadressen för en person eller grupp som ska ta emot meddelanden om etablering fel och markera kryssrutan.

8. Klicka på **spara.**

9. Välj under avsnittet mappningar **synkronisera Azure Active Directory-användare DocuSign.**

10. I den **attributmappning** avsnittet kan du granska användarattribut som synkroniseras från Azure AD till DocuSign. De attribut som valts som **matchande** egenskaper som används för att matcha användarkonton i DocuSign för uppdateringsåtgärder. Välj knappen Spara för att genomföra ändringarna.

11. Om du vill aktivera Azure AD-tjänsten för DocuSign-etablering, ändra den **Status för etablering** till **på** i avsnittet Inställningar

12. Klicka på **spara.**

Startar den första synkroniseringen av alla användare som tilldelats DocuSign i avsnittet användare och grupper. Den första synkroniseringen tar längre tid än efterföljande synkroniseringar som sker ungefär var 40 minuter så länge som tjänsten körs. Du kan använda den **synkroniseringsinformation** avsnittet för att övervaka förloppet och följ länkarna till att etablera aktivitetsloggar som beskriver alla åtgärder som utförs av tjänsten etablering DocuSign appen.

Mer information om hur du tolkar Azure AD-etablering loggar finns [rapportering om automatisk konto användaretablering](active-directory-saas-provisioning-reporting.md).

## <a name="additional-resources"></a>Ytterligare resurser

* [Hantera användare konto-etablering för företag-appar](active-directory-saas-tutorial-list.md)
* [Vad är programåtkomst och enkel inloggning med Azure Active Directory?](manage-apps/what-is-single-sign-on.md)
* [Konfigurera enkel inloggning](active-directory-saas-docusign-tutorial.md)