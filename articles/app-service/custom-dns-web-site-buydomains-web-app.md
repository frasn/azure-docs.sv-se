---
title: Köp ett anpassat domännamn för Azure Web Apps
description: Lär dig hur du köper ett anpassat domännamn med en webbapp i Azure App Service.
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: 70fb0e6e-8727-4cca-ba82-98a4d21586ff
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/24/2017
ms.author: cephalin
ms.openlocfilehash: 16a6d8e34bdec6e597c5f04faf421d387a877bff
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34714892"
---
# <a name="buy-a-custom-domain-name-for-azure-web-apps"></a>Köp ett anpassat domännamn för Azure Web Apps

Apptjänst-domäner (förhandsversion) är toppnivådomäner som hanteras direkt i Azure. De gör det lättare att hantera anpassade domäner för [Azure Web Apps](app-service-web-overview.md). Den här kursen visar hur du köper en Apptjänst-domän och tilldela Azure Web Apps DNS-namn.

Den här artikeln är Azure App Service (Web Apps, API Apps, Mobilappar, Logic Apps). Azure Storage eller Azure VM finns [tilldela Apptjänst domän till Azure VM eller Azure Storage](https://blogs.msdn.microsoft.com/appserviceteam/2017/07/31/assign-app-service-domain-to-azure-vm-or-azure-storage/). Cloud Services, se [konfigurera ett anpassat domännamn för en Azure-molntjänst](../cloud-services/cloud-services-custom-domain-name-portal.md).

## <a name="prerequisites"></a>Förutsättningar

För att slutföra den här självstudien behöver du:

* [Skapa en App Service-app](/azure/app-service/), eller använd en app som du har skapat för en annan kurs.
* [Ta bort utgiftsgränsen för din prenumeration](../billing/billing-spending-limit.md#remove). Du kan inte köpa Apptjänst domäner med ledigt prenumerationskrediter.

## <a name="prepare-the-app"></a>Förbereda appen

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

Att använda anpassade domäner i Azure Web Apps ditt webbprograms [programtjänstplanen](https://azure.microsoft.com/pricing/details/app-service/) måste vara en betald nivå (**delade**, **grundläggande**, **Standard**, eller **Premium**). I det här steget kan se du till att webbappen är i det prisnivån.

### <a name="sign-in-to-azure"></a>Logga in på Azure

Öppna [Azure Portal](https://portal.azure.com) och logga in med ditt Azure-konto.

### <a name="navigate-to-the-app-in-the-azure-portal"></a>Navigera till appen i Azure Portal

Välj **App Services** på menyn till vänster och välj sedan appens namn.

![Portalnavigering till Azure-app](./media/app-service-web-tutorial-custom-domain/select-app.png)

Du ser hanteringssidan för App Service-appen.  

### <a name="check-the-pricing-tier"></a>Kontrollera prisnivån

I det vänstra navigeringsfältet på appsidan bläddrar du till avsnittet **Inställningar** och väljer **Skala upp (App Service-plan)**.

![Skala upp-menyn](./media/app-service-web-tutorial-custom-domain/scale-up-menu.png)

Appens aktuell nivå markeras med en blå kantlinje. Kontrollera att appen inte är i den **F1** nivå. Anpassad DNS stöds inte i den **F1** nivå. 

![Kontrollera prisnivå](./media/app-service-web-tutorial-custom-domain/check-pricing-tier.png)

Om App Service-plan inte är i den **F1** nivån, Stäng av **skala upp** sidan och gå vidare till [köpa domänen](#buy-the-domain).

### <a name="scale-up-the-app-service-plan"></a>Skala upp App Service-planen

Välj någon av de icke kostnadsfria nivåerna (**D1**, **B1**, **B2**, **B3**, eller någon tjänstnivån i den **produktion** kategori). För ytterligare alternativ klickar du på **se ytterligare alternativ**.

Klicka på **Använd**.

![Kontrollera prisnivå](./media/app-service-web-tutorial-custom-domain/choose-pricing-tier.png)

När du ser följande meddelande har skalningsåtgärden slutförts.

![Bekräftelse av skalningsåtgärd](./media/app-service-web-tutorial-custom-domain/scale-notification.png)

## <a name="buy-the-domain"></a>Köpa domänen

### <a name="sign-in-to-azure"></a>Logga in på Azure
Öppna [Azure Portal](https://portal.azure.com/) och logga in med ditt Azure-konto.

### <a name="launch-buy-domains"></a>Starta köp domäner
I den **Web Apps** klickar du på namnet på ditt webbprogram, Välj **inställningar**, och välj sedan **anpassade domäner**
   
![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

I den **anpassade domäner** klickar du på **köpa domän**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-1.png)

> [!NOTE]
> Om du inte ser den **App Service domäner** avsnitt, du måste ta bort utgiftsgränsen på ditt Azure-konto (se [krav](#prerequisites)).
>
>

### <a name="configure-the-domain-purchase"></a>Konfigurera domänen köpet

I den **Service tillämpningsdomän** sidan den **Sök efter domän** skriver domännamnet som du vill köpa och skriv `Enter`. Föreslagna tillgängliga domäner visas under textrutan. Välj en eller flera domäner som du vill köpa.
   
![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-2.png)

> [!NOTE]
> Följande [toppnivådomäner](https://wikipedia.org/wiki/Top-level_domain) stöds av Apptjänst domäner: _com_, _net_, _co.uk_, _org_, _nl_, _i_, _biz_, _org.uk_, och _co.in_.
>
>

Klicka på den **kontaktuppgifter** och fylla i domänens kontaktinformation formulär. När du är klar klickar du på **OK** att återgå till sidan App Service-domän.
   
Det är viktigt att du fyller i alla obligatoriska fält med så mycket noggrannhet som möjligt. Felaktiga data kontaktinformation kan resultera i misslyckande med att köpa domäner. 

Välj därefter önskade alternativ för din domän. Se följande tabell förklaringar:

| Inställning | Föreslaget värde | Beskrivning |
|-|-|-|
|Sekretesskydd | Aktivera | Välja att ”sekretesskydd”, som ingår i inköpspriset _gratis_. Vissa toppnivådomäner hanteras av registratorer som inte stöder sekretesskydd och de visas på den **sekretesskydd** sidan. |
| Tilldela standard värdnamn | **www** och **@** | Välj önskad värdnamnsbindningar om så önskas. När domänen köp åtgärden är slutförd måste kan ditt webbprogram nås på den valda värdnamnen. Om webbappen bakom [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/), visas alternativet att tilldela rotdomänen (@), eftersom Traffic Manager har inte stöd för A-poster. Du kan ändra värdnamnet tilldelningar efter domän köpet har slutförts. |

### <a name="accept-terms-and-purchase"></a>Acceptera villkoren och köpa

Klicka på **juridiska villkor** att granska villkoren och kostnader och klicka sedan på **köpa**.

> [!NOTE]
> Apptjänst domäner använda DNS för Azure som värd för domänerna. Förutom domän registreringsavgift gäller användningskostnader för Azure DNS. Mer information finns i [priser för Azure DNS](https://azure.microsoft.com/pricing/details/dns/).
>
>

I den **Service tillämpningsdomän** klickar du på **OK**. Medan åtgärden pågår kan se du följande meddelanden:

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-validate.png)

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-purchase-success.png)

### <a name="test-the-hostnames"></a>Testa värdnamnen

Om du har tilldelat ditt webbprogram standard värdnamn se du också en lyckad avisering för varje vald värdnamn. 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

Du också se valda värdnamn i den **anpassade domäner** sidan den **anpassad värdnamn** avsnitt. 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added.png)

Gå till listan värdnamn i webbläsaren om du vill testa värdnamnen. I exemplet i den föregående skärmbilden försök gå till _kontoso.net_ och _www.kontoso.net_.

## <a name="assign-hostnames-to-web-app"></a>Tilldela webbprogrammet värdnamn

Om du inte väljer att tilldela en eller flera standard värdnamn till ditt webbprogram under inköpsprocessen, eller om måste du tilldela ett värdnamn som inte finns, kan du tilldela ett värdnamn på när som helst.

Du kan också tilldela värdnamn i programdomänen för tjänsten till ett annat webbprogram. Stegen är beroende av om App Service-domänen och webbprogrammet tillhöra samma prenumeration.

- Annan prenumeration: mappa anpassade DNS-poster från domänen App Service till webbprogrammet som en externt köpta domän. Mer information om att lägga till anpassade DNS-namn till en App Service-domän finns [hantera anpassade DNS-poster](#custom). Om du vill mappa en extern köpta domän till en webbapp, se [mappa ett befintligt anpassade DNS-namn till Azure Web Apps](app-service-web-tutorial-custom-domain.md). 
- Samma prenumeration: Använd följande steg.

### <a name="launch-add-hostname"></a>Starta Lägg till värdnamn
I den **Apptjänster** markerar du namnet på ditt webbprogram som du vill tilldela värdnamn, Välj **inställningar**, och välj sedan **anpassade domäner**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

Kontrollera att dina köpta domän visas i den **App Service domäner** avsnittet, men inte välja den. 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

> [!NOTE]
> Alla App Service-domäner i samma prenumeration som visas i webbappens **anpassade domäner** sidan. Om din domän finns i webbappens prenumeration, men du kan se den i webbapp **anpassade domäner** och prova att öppna den **anpassade domäner** sidan eller uppdatera webbsida. Kontrollera också notification bell överst i Azure-portalen för pågår eller skapa fel.
>
>

Välj **Lägg till värddatornamn**.

### <a name="configure-hostname"></a>Konfigurera värdnamnet
I den **lägga till värdnamnet** dialogrutan Skriv fullständigt kvalificerade domännamnet för din App Service-domän eller en underdomän. Exempel:

- kontoso.net
- www.kontoso.net
- abc.kontoso.net

När du är klar väljer **verifiera**. Hostname-posttypen väljs automatiskt för dig.

Välj **Lägg till värddatornamn**.

När åtgärden är klar, kan du se ett fungerande meddelande för tilldelade värdnamnet.  

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-bind-success.png)

### <a name="close-add-hostname"></a>Stäng lägga till värdnamnet
I den **lägga till värdnamnet** anger andra värdnamnet för ditt webbprogram efter behov. När du är klar stänger du den **lägga till värdnamnet** sidan.

Du bör nu se nytilldelade hostname(s) i din app **anpassade domäner** sidan.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostnames-added2.png)

### <a name="test-the-hostnames"></a>Testa värdnamnen

Gå till listan värdnamn i webbläsaren. I exemplet i den föregående skärmbilden försök gå till _abc.kontoso.net_.

## <a name="renew-the-domain"></a>Förnya domänen

Apptjänst-domän som du har köpt är giltig i ett år från tidpunkten då du köper. Som standard konfigureras domänen för att förnya automatiskt genom att dra betalningsmetoden för nästa år. Om du vill inaktivera automatisk förnyelse eller om du vill manuellt förnya din domän anvisningarna här.

I den **Web Apps** klickar du på namnet på ditt webbprogram, Välj **inställningar**, och välj sedan **anpassade domäner**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-6.png)

I den **App Service domäner** väljer du den domän som du vill konfigurera.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-select-domain.png)

I det vänstra navigeringsfönstret i domänen, väljer du **domänförnyelse**. Om du vill stoppa automatiskt förnya din domän, Välj **av**, och sedan **spara**. 

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-autorenew.png)

Om du vill manuellt förnya din domän, Välj **förnya domänen**. Men är den här knappen inte aktiv förrän 90 dagar före domänens förfallodatum.

<a name="custom"></a>

## <a name="manage-custom-dns-records"></a>Hantera anpassade DNS-poster

I Azure, DNS-poster för en tillämpningsdomän Service hanteras med hjälp av [Azure DNS](https://azure.microsoft.com/services/dns/). Du kan lägga till, ta bort, och uppdatera DNS-poster, precis som för en externt köpta domän.

### <a name="open-app-service-domain"></a>Öppna App Service-domän

Välj i Azure-portalen från den vänstra menyn **alla tjänster** > **App Service domäner**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

Välj domänen som ska hantera. 

### <a name="access-dns-zone"></a>Åtkomst till DNS-zonen

Välj domänens vänstra menyn **DNS-zonen**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-dns-zone.png)

Den här åtgärden öppnar den [DNS-zonen](../dns/dns-zones-records.md) sidan i din App Service-domän i Azure DNS. Information om hur du redigerar DNS-poster finns [hur du hanterar DNS-zoner i Azure portal](../dns/dns-operations-dnszones-portal.md).

## <a name="cancel-purchase-delete-domain"></a>Avbryt inköp (ta bort domän)

När du har köpt tjänsten tillämpningsdomänen har fem dagar att häva köpet för en full återbetalning. Efter fem dagar du kan ta bort domänen App Service, men det går inte att få en återbetalning.

### <a name="open-app-service-domain"></a>Öppna App Service-domän

Välj i Azure-portalen från den vänstra menyn **alla tjänster** > **App Service domäner**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-access.png)

Välj domänen du vill avbryta eller ta bort. 

### <a name="delete-hostname-bindings"></a>Ta bort värdnamnsbindningar

Välj domänens vänstra menyn **värdnamnsbindningar**. Värdnamnsbindningar från alla Azure-tjänster i den här listan.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-hostname-bindings.png)

Du kan inte ta bort tjänsten tillämpningsdomän förrän alla värdnamnsbindningar har tagits bort.

Ta bort varje värdnamn bindning genom att välja **...**   >  **Ta bort**. När alla bindningar tas bort, Välj **spara**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-delete-hostname-bindings.png)

### <a name="cancel-or-delete"></a>Avbryt eller ta bort

Välj domänens vänstra menyn **översikt**. 

Om den annullering på domänen köpta inte har gått, väljer **Avbryt inköp**. Annars kan du se en **ta bort** knappen i stället. Om du vill ta bort domänen utan bidrag, Välj **ta bort**.

![](./media/custom-dns-web-site-buydomains-web-app/dncmntask-cname-buydomains-cancel.png)

Bekräfta åtgärden genom att markera **Ja**.

När åtgärden är klar, är domänen utgivna från prenumerationen och är tillgängliga för alla att köpa igen. 

## <a name="direct-default-url-to-a-custom-directory"></a>Dirigera standard-URL:en till en anpassad katalog

Som standard dirigerar App Service webbegäranden till rotkatalogen för din appkod. Att dirigera dem till en underkatalog som `public`, se [direkt standard-URL till en anpassad katalog](app-service-web-tutorial-custom-domain.md#virtualdir).

## <a name="more-resources"></a>Fler resurser

[Vanliga frågor och svar: App Service-domän (förhandsversion) och anpassade domäner](https://blogs.msdn.microsoft.com/appserviceteam/2017/08/08/faq-app-service-domain-preview-and-custom-domains/)