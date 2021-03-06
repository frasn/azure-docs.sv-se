---
title: 'Självstudier: Azure Active Directory-integrering med GitHub | Microsoft Docs'
description: Lär dig hur du konfigurerar enkel inloggning mellan Azure Active Directory och GitHub.
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 8761f5ca-c57c-4a7e-bf14-ac0421bd3b5e
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2018
ms.author: jeedes
ms.openlocfilehash: 58e85dae3e6b39fc2976cdaa06a64a9073015a17
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34590043"
---
# <a name="tutorial-azure-active-directory-integration-with-github"></a>Självstudier: Azure Active Directory-integrering med GitHub

I kursen får lära du att integrera GitHub med Azure Active Directory (AD Azure).

Integrera GitHub med Azure AD ger dig följande fördelar:

- Du kan styra i Azure AD som har åtkomst till GitHub.
- Du kan aktivera användarna att automatiskt hämta loggat in på GitHub (Single Sign-On) med sina Azure AD-konton.
- Du kan hantera dina konton i en central plats - Azure-portalen.

Om du vill veta mer information om integrering av SaaS-app med Azure AD finns [vad är programåtkomst och enkel inloggning med Azure Active Directory](manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Förutsättningar

För att konfigurera Azure AD-integrering med GitHub, behöver du följande:

- En Azure AD-prenumeration
- En GitHub enkel inloggning aktiverad prenumeration

> [!NOTE]
> Om du vill testa stegen i den här kursen rekommenderar vi inte med hjälp av en produktionsmiljö.

Om du vill testa stegen i den här självstudiekursen, bör du följa dessa rekommendationer:

- Använd inte i produktionsmiljön, om det är nödvändigt.
- Om du inte har en utvärderingsversion Azure AD-miljö kan du [hämta en utvärderingsversion för en månad](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Scenariobeskrivning
I kursen får testa du Azure AD enkel inloggning i en testmiljö. Det scenario som beskrivs i den här kursen består av två huvudsakliga byggblock:

1. Att lägga till GitHub från galleriet
2. Konfigurera och testa Azure AD enkel inloggning

## <a name="adding-github-from-the-gallery"></a>Att lägga till GitHub från galleriet
Du måste lägga till GitHub från galleriet i listan över hanterade SaaS-appar för att konfigurera integrering av GitHub i Azure AD.

**Utför följande steg för att lägga till GitHub från galleriet:**

1. I den  **[Azure-portalen](https://portal.azure.com)**, klicka på den vänstra navigeringspanelen **Azure Active Directory** ikon. 

    ![Azure Active Directory-knappen][1]

2. Gå till **företagsprogram**. Gå till **alla program**.

    ![Bladet Enterprise program][2]
    
3. Om du vill lägga till nya programmet, klickar du på **nytt program** knappen överst i dialogrutan.

    ![Knappen Nytt program][3]

4. I sökrutan skriver **GitHub**väljer **GitHub** resultatet-panelen klickar **Lägg till** för att lägga till programmet.

    ![GitHub i resultatlistan](./media/active-directory-saas-github-tutorial/tutorial_github_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Konfigurera och testa Azure AD enkel inloggning

I det här avsnittet, konfigurera och testa Azure AD enkel inloggning med GitHub baserat på en testanvändare som kallas ”Britta Simon”.

Azure AD måste du känna till användaren i GitHub motsvarighet till en användare i Azure AD för enkel inloggning ska fungera. Med andra ord måste en länk mellan en Azure AD-användare och relaterade användaren i GitHub upprättas.

Om du vill konfigurera och testa Azure AD enkel inloggning med GitHub, måste du utföra följande byggblock:

1. **[Konfigurera Azure AD enkel inloggning](#configure-azure-ad-single-sign-on)**  - om du vill att användarna kan använda den här funktionen.
2. **[Skapa en Azure AD-testanvändare](#create-an-azure-ad-test-user)**  - om du vill testa Azure AD enkel inloggning med Britta Simon.
3. **[Skapa en testanvändare i GitHub](#create-a-github-test-user)**  – du har en motsvarighet för Britta Simon i GitHub som är kopplad till Azure AD-representation av användaren.
4. **[Tilldela Azure AD-testanvändare](#assign-the-azure-ad-test-user)**  - om du vill aktivera Britta Simon att använda Azure AD enkel inloggning.
5. **[Testa enkel inloggning](#test-single-sign-on)**  - om du vill kontrollera om konfigurationen fungerar.

### <a name="configure-azure-ad-single-sign-on"></a>Konfigurera Azure AD enkel inloggning

I det här avsnittet Aktivera Azure AD enkel inloggning i Azure-portalen och konfigurera enkel inloggning i ditt GitHub-program.

**Utför följande steg för att konfigurera Azure AD enkel inloggning med GitHub:**

1. I Azure-portalen på den **GitHub** integreringssidan för programmet, klickar du på **enkel inloggning**.

    ![Konfigurera enkel inloggning länk][4]

2. På den **enkel inloggning** markerar **läge** som **SAML-baserade inloggning** att aktivera enkel inloggning.
 
    ![Enkel inloggning dialogrutan](./media/active-directory-saas-github-tutorial/tutorial_github_samlbase.png)

3. På den **GitHub domän och URL: er** avsnittet, utför följande steg:

    ![URL: er och GitHub-domän med enkel inloggning information](./media/active-directory-saas-github-tutorial/tutorial_github_url.png)

    a. I den **inloggning URL** textruta Skriv en URL med följande mönster: `https://github.com/orgs/<entity-id>/sso`

    b. I den **identifierare (enhets-ID)** textruta Skriv en URL med följande mönster: `https://github.com/orgs/<entity-id>`

    > [!NOTE]
    > Observera att detta inte är verkliga värden. Du måste uppdatera dessa värden med den faktiska inloggning URL och identifierare. Här rekommenderar vi att du om du vill använda det unika värdet på strängen i identifieraren. Gå till GitHub-administratör att hämta dessa värden.

4. På den **användarattribut** väljer **användar-ID** som user.mail.

    ![Konfigurera enkel inloggning](./media/active-directory-saas-github-tutorial/tutorial_github_attribute_new01.png)

5. På den **SAML-signeringscertifikat** klickar du på **certifikat (Base64)** och spara certifikatfilen på datorn.

    ![Länken hämta certifikatet](./media/active-directory-saas-github-tutorial/tutorial_github_certificate.png) 

6. Klicka på **spara** knappen.

    ![Konfigurera enkel inloggning spara](./media/active-directory-saas-github-tutorial/tutorial_general_400.png)

7. På den **GitHub Configuration** klickar du på **konfigurera GitHub** att öppna **konfigurera inloggning** fönster. Kopiera den **Sign-Out URL, SAML enhets-ID och SAML enkel inloggning Tjänstwebbadress** från den **Snabbreferens avsnitt.**

    ![GitHub-konfiguration](./media/active-directory-saas-github-tutorial/tutorial_github_configure.png) 

8. Logga in på webbplatsen GitHub organisation som en administratör i en annan webbläsarfönster.

9. Gå till **inställningar** och på **säkerhet**

    ![Inställningar](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_03.png)

10. Kontrollera den **aktivera SAML-autentisering** rutan avslöja konfigurationsfält enkel inloggning. Använd sedan på enkel inloggning URL-adressen för att uppdatera den enda inloggnings-URL på Azure AD-konfiguration.

    ![Inställningar](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_13.png)

11. Konfigurera följande fält:

    a. I den **inloggning URL** textruta klistra in **SAML inloggning tjänst-URL för enkel** värde som du har kopierat från Azure-portalen.

    b. I den **utfärdaren** textruta klistra in **SAML enhets-ID** värde som du har kopierat från Azure-portalen.

    c. Öppna hämtat certifikat från Azure-portalen i anteckningar, klistra in innehållet i den **certifikat med offentlig** textruta.

    ![Inställningar](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_051.png)

12. Klicka på **Test SAML-konfiguration** Kontrollera att inga verifieringsfel eller fel under enkel inloggning.

    ![Inställningar](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_06.png)

13. Klicka på **Spara**

### <a name="create-an-azure-ad-test-user"></a>Skapa en testanvändare i Azure AD

Syftet med det här avsnittet är att skapa en testanvändare i Azure-portalen kallas Britta Simon.

   ![Skapa en testanvändare i Azure AD][100]

**Utför följande steg för att skapa en testanvändare i Azure AD:**

1. I Azure-portalen i den vänstra rutan klickar du på den **Azure Active Directory** knappen.

    ![Azure Active Directory-knappen](./media/active-directory-saas-github-tutorial/create_aaduser_01.png)

2. Om du vill visa en lista över användare, gå till **användare och grupper**, och klicka sedan på **alla användare**.

    ![”Användare och grupper” och ”alla användare” länkar](./media/active-directory-saas-github-tutorial/create_aaduser_02.png)

3. Öppna den **användare** dialogrutan klickar du på **Lägg till** överst i den **alla användare** dialogrutan.

    ![Knappen Lägg till](./media/active-directory-saas-github-tutorial/create_aaduser_03.png)

4. I den **användaren** dialogrutan utför följande steg:

    ![Dialogrutan användare](./media/active-directory-saas-github-tutorial/create_aaduser_04.png)

    a. I den **namn** skriver **BrittaSimon**.

    b. I den **användarnamn** Skriv användarens Britta Simon e-postadress.

    c. Välj den **visa lösenordet** kryssrutan och sedan skriva ned det värde som visas i den **lösenord** rutan.

    d. Klicka på **Skapa**.
 
### <a name="create-a-github-test-user"></a>Skapa en testanvändare i GitHub

Syftet med det här avsnittet är att skapa en användare som kallas Britta Simon i GitHub. GitHub har stöd för automatisk användaretablering, vilket är aktiverat som standard. Du hittar mer information [här](active-directory-saas-github-provisioning-tutorial.md) om hur du konfigurerar automatisk användaretablering.

**Om du behöver skapa användare manuellt gör du följande:**

1. Logga in på webbplatsen GitHub företag som administratör.

2. Klicka på **personer**.

    ![Personer](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_08.png "personer")

3. Klicka på **inbjudan medlem**.

    ![Bjuda in användare](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_09.png "bjuda in användare")

4. På den **inbjudan medlem** dialogrutan utför följande steg:

    a. I den **e-post** textruta skriver Britta Simon konto e-postadress.

    ![Bjuda in](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_10.png "bjuda in")

    b. Klicka på **skicka inbjudan**.

    ![Bjuda in](./media/active-directory-saas-github-tutorial/tutorial_github_config_github_11.png "bjuda in")

    > [!NOTE]
    > Innehavaren för Azure Active Directory-konto ett e-postmeddelande och länken för att bekräfta sina konton innan den aktiveras.

### <a name="assign-the-azure-ad-test-user"></a>Tilldela Azure AD-testanvändare

I det här avsnittet kan du aktivera Britta Simon att använda Azure enkel inloggning genom att bevilja åtkomst till GitHub.

![Tilldela rollen][200] 

**Om du vill tilldela GitHub Britta Simon utför du följande steg:**

1. Öppna vyn program i Azure-portalen och gå till vyn directory och gå till **företagsprogram** Klicka **alla program**.

    ![Tilldela användare][201]

2. Välj i listan med program **GitHub**.

    ![GitHub-länken i listan med program](./media/active-directory-saas-github-tutorial/tutorial_github_app.png)  

3. Klicka på menyn till vänster **användare och grupper**.

    ![Länken ”användare och grupper”][202]

4. Klicka på **Lägg till** knappen. Välj sedan **användare och grupper** på **Lägg uppdrag** dialogrutan.

    ![Fönstret Lägg till tilldelning][203]

5. På **användare och grupper** markerar **Britta Simon** på listan användare.

6. Klicka på **Välj** knappen på **användare och grupper** dialogrutan.

7. Klicka på **tilldela** knappen på **Lägg uppdrag** dialogrutan.
    
### <a name="test-single-sign-on"></a>Testa enkel inloggning

I det här avsnittet kan du testa Azure AD enkel inloggning konfigurationen med hjälp av panelen åtkomst.

När du klickar på GitHub-panelen på åtkomstpanelen du bör få automatiskt loggat in på ditt GitHub-program.
Läs mer om åtkomstpanelen [introduktion till åtkomstpanelen](active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Ytterligare resurser

* [Lista över självstudier om hur du integrerar SaaS-appar med Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Vad är programåtkomst och enkel inloggning med Azure Active Directory?](manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/active-directory-saas-github-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-github-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-github-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-github-tutorial/tutorial_general_04.png

[100]: ./media/active-directory-saas-github-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-github-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-github-tutorial/tutorial_general_201.png
[202]: ./media/active-directory-saas-github-tutorial/tutorial_general_202.png
[203]: ./media/active-directory-saas-github-tutorial/tutorial_general_203.png

