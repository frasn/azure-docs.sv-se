---
title: Översikt över lösningen Ansluten fabrik – Azure | Microsoft Docs
description: En beskrivning av lösningsacceleratorn Ansluten fabrik i Azure IoT.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: dobett
ms.openlocfilehash: 1dbaf4978bd23058d37239ba97bfe8583dc1f2c9
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34627099"
---
# <a name="get-started-with-the-connected-factory-solution-accelerator"></a>Kom igång med lösningsacceleratorn Ansluten fabrik

Azure [IoT -lösningsacceleratorer][lnk-preconfigured-solutions] kombinerar flera Azure IoT-tjänster och levererar lösningar från slutpunkt till slutpunkt som implementerar vanliga IoT-företagsscenarier. Lösningsacceleratorn *Ansluten fabrik* ansluter till och övervakar dina industriella enheter. Med lösningen kan du analysera dataströmmar från dina enheter och öka produktiviteten och lönsamheten i verksamheten.

Den här självstudien visar hur du etablerar lösningsacceleratorn Ansluten fabrik. Vi går också igenom de grundläggande funktionerna i lösningsacceleratorn. Du har åtkomst till många av dessa funktioner från lösningens *instrumentpanel* som distribueras som en del av lösningsacceleratorn:

![Instrumentpanel för lösningsacceleratorn Ansluten fabrik][img-cf-home]

Du behöver en aktiv Azure-prenumeration för att kunna utföra stegen i den här självstudiekursen.

> [!NOTE]
> Om du inte har något konto kan du skapa ett kostnadsfritt utvärderingskonto på bara några minuter. Mer information finns i [kostnadsfri utvärderingsversion av Azure][lnk_free_trial].

## <a name="provision-the-solution"></a>Etablera lösningen

1. Logga in på azureiotsuite.com med din Azure-kontoinformation och skapa en lösning genom att klicka på **+**.
2. Klicka på **Välj** i panelen **Ansluten fabrik**.
3. Ange ett **Lösningsnamn** för lösningsacceleratorn Ansluten fabrik.
4. Välj den **prenumeration** och **region** som du vill använda för att etablera lösningen.
5. Klicka på **Skapa lösning** för att påbörja etableringen. Den här processen tar normalt flera minuter.

### <a name="while-you-wait-for-the-provisioning-process-to-complete"></a>Medan du väntar på att etableringsprocessen ska slutföras

1. Klicka på ikonen för din lösning med statusen **Etablerar**.
2. Observera **etableringsstatusen** när Azure-tjänsterna distribueras i din Azure-prenumeration.
3. När etableringen har slutförts ändras statusen till **Klar**.
4. Klicka på ikonen så ser du informationen om din lösning i den högra rutan.

> [!NOTE]
> Om det uppstår problem när du distribuerar lösningsacceleratorn kan du läsa [Behörigheter på webbplatsen azureiotsuite.com][lnk-permissions] och [Vanliga frågor och svar om Ansluten fabrik](iot-accelerators-faq-cf.md). Om problemen kvarstår så skapa en tjänstbiljett på [portalen][lnk-portal].

Finns det något som du förväntar dig att se men som inte visas för din lösning? Lämna förslag på funktioner i [User Voice](https://feedback.azure.com/forums/321918-azure-iot).

## <a name="scenario-overview"></a>Scenarioöversikt

När du distribuerar lösningsacceleratorn Ansluten fabrik innehåller den redan resurser som hjälper dig genom ett vanligt industriscenario. I det här scenariot rapporterar flera fabriker (som är anslutna till lösningen) de datavärden som krävs för att beräkna utrustningseffektivitet (Overall Equipment Efficiency, OEE) och KPI: er (Key Performance Indicators). I avsnitten nedan får du information om följande:

* Övervaka fabrik, produktionslinjer, station OEE och KPI-värden
* Analysera telemetridata som genereras av dessa enheter med hjälp av Azure Time Series Insights
* Åtgärda larm för att lösa problem

En viktig egenskap i detta scenario är att du kan utföra alla dessa åtgärder via fjärranslutning från instrumentpanelen med lösningar. Du behöver inte fysisk åtkomst till enheterna.

## <a name="view-the-solution-dashboard"></a>Visa instrumentpanelen för lösningen

På instrumentpanelen för lösningen kan du hantera den distribuerade lösningen. Det här är en hierarkisk representation av en global fabrikskonfiguration. Du kan till exempel se OEE och KPI:er, publicera nya noder för telemetri och åtgärdslarm.

1. När etableringen har slutförts och panelen för lösningsacceleratorn visar **Klar**, klickar du på **Starta**. Lösningsportalen Ansluten fabrik öppnas då på en ny flik.

    ![Starta lösningsacceleratorn][img-launch-solution]

1. Som standard visar lösningsportalen *instrumentpanelen*. Du kan navigera till andra delar av portalen via menyn till vänster på sidan.

    ![Instrumentpanel för lösningsacceleratorn Ansluten fabrik][cf-img-menu]

Följande information visas på instrumentpanelen:

* En panel med **fabriksplatser** som visar status, plats och aktuell produktionskonfiguration i lösningen. Första gången du kör lösningen finns det fyra simulerade enheter. Produktionslinjesimuleringen består av tre verkliga OPC UA-servrar per produktionslinje. Dessa utför simulerade uppgifter och delar data. Mer information om OPC UA finns i [Vanliga frågor och svar om Ansluten fabrik](iot-accelerators-faq-cf.md).
* En **karta** visar platsen för alla enheter som är kopplade till lösningen. Lösningen kan använda Bing Maps-API:t för att rita information på kartan. Om Bing Maps Enterprise API är aktiverat för din prenumeration används den här funktionen automatiskt. I annat fall kan du se [vanliga frågor och svar][lnk-faq] för information om hur du gör kartan dynamisk.
* En panel för **larm** som visar de larm som genereras när ett telemetri- eller OEE/KPI-värde överskrider ett visst tröskelvärde.
* En panel för **OEE** (Overall Equipment Efficiency) som visar OEE-värdena för hela företaget eller den fabrik, produktionslinje eller station du tittar på. Det här värdet sammanställs från stationsvyn till företagsnivån. OEE-bilden och dess beståndsdelar kan analyseras ytterligare.
* Panelen för **KPI:er** som visar antalet enheter som producerats och energi som förbrukas av hela företaget eller den fabrik, produktionslinje eller station som du visar. Dessa värden sammanställs från stationsvyn till företagsnivån.

## <a name="view-factories"></a>Visa fabriker

Panelen för *fabriksplatser* visar den geografiska platsen för alla fabriker i lösningen, deras status och den aktuella produktionskonfigurationen. Från listan över platser kan du navigera till andra nivåer i lösningshierarkin. Raderna i listan är hyperlänkar till information om produktionslinjerna på den platsen. Du kan söka vidare i informationen om produktionslinjen ned till vyn på stationsnivå. Du kan också använda ett filter för listan.

![Fabriker för lösningsacceleratorn Ansluten fabrik][cf-img-factories]

1. På **fabrikspanelen** visas fabrikslistan för lösningen.

2. I fabrikslistan visas inledningsvis sex fabriker som skapades vid etableringsprocessen. Du kan lägga till ytterligare simulerade och fysiska enheter till lösningen.

3. Om du vill visa information om en fabrik klickar du på raden i fabrikslistan.

4. Om du vill visa information om en produktionslinje klickar du på raden i listan.

5. Om du vill visa publicerade OPC UA-noder för en station i produktionslinjen klickar du på raden i listan.

6. Om du vill visa information om en viss nod i stationen klickar du på raden i listan. Den här åtgärden öppnar en kontextpanel med Time Series Insights-visualiseringar. Klicka på dessa diagram om du vill göra vidare analyser i Time Series Insights-utforskarmiljön.

## <a name="view-map"></a>Visa karta

Om din prenumeration ger åtkomst till Bing Maps-API:t visar *fabrikskartan* geografisk plats och status för alla fabriker i lösningen. Klicka på platserna som visas på kartan om du vill visa mer detaljerad information om platsen.

![Karta för lösningsacceleratorn Ansluten fabrik][cf-img-map]

## <a name="view-alarms"></a>Visa larm

**Larmpanelen** visar de larm som genererats på grund av ett rapporterat värde eller ett beräknat OEE/KPI-värde som överstiger det konfigurerade tröskelvärdet. Den här panelen visar larm på varje nivå i hierarkin, från vyn på stationsnivå till vyn på global nivå. Larmen innehåller en beskrivning av larmet, datum, tid, plats och antal förekomster. Du kan få information om vad som orsakat larmet med hjälp av Time Series Insights-data. Time Series Insights-data visualiseras i larmen (om tillämpligt). Om du är administratör kan du vidta standardåtgärder för larm, till exempel:

* Stäng larmet.
* Bekräfta larmet.

Du kan också kan du utföra mer komplexa åtgärder. För Pressure OPC UA-noden i sammansättningen kan du till exempel:

* Visa mer information på en webbsida i ett nytt webbläsarfönster.
* Förebygg orsaken till larmet genom att anropa en OPC UA-metod på enheten.
* Välja att standardåtgärderna inte ska vara tillgängliga.

    ![Larm för lösningsacceleratorn Ansluten fabrik][cf-img-alerts]

> [!NOTE]
> Larmen genereras av regler som anges i en konfigurationsfil i lösningsacceleratorn. Dessa regler kan generera larm när OEE- eller KPI-värden eller OPC UA-nodvärden överskrider de angivna tröskelvärdena.

1. **Larmpanelen** visar de larm som genererats i den här lösningen.

2. Om du vill visa information om ett larm klickar du på larmet i larmpanelen.

3. Du kan analysera larminformationen mer i detalj genom att klicka på diagrammet på larmpanelen för att öppna Time Series Insights-utforskarmiljön.

4. På larmpanelen finns flera åtgärder som du kan vidta för larmet. Välj ett lämpligt alternativ och klicka på knappen för att utföra åtgärden.

## <a name="view-overall-equipment-efficiency"></a>Visa OEE (Overall Equipment Efficiency)

OEE är nyckeltal för att mäta produktionseffektivitet. OEE är ett branschstandardmått som beräknas genom att multiplicera tillgänglighet, anläggningsutnyttjande och kvalitetsutbyte: OEE = tillgänglighet x anläggningsutnyttjande x kvalitetsutbyte.

![OEE för lösningsacceleratorn Ansluten fabrik][cf-img-oee]

1. Om du vill visa OEE för någon nivå i hierarkin navigerar du till den vy som du vill visa. OEE för vyn visas på panelen tillsammans med de delar som utgör OEE-procentvärdet.

2. Om du vill analysera OEE för en nivå i hierarkin i mer detalj klickar du på procentvärdet för OEE, tillgänglighet, prestanda eller kvalitet. En kontextpanel visas med Time Series Insights-visualiseringar som visar data från den senaste timmen, de senaste 24 timmarna och de senaste sju dagarna.

    ![TSI-visualisering för lösningsacceleratorn Ansluten fabrik][cf-img-tsi-visualization]

3. Om du vill analysera larmdata mer i detalj klickar du på diagrammet på larmpanelen. Den här åtgärden öppnar Time Series Insights-utforskarmiljön.

    ![TSI-utforskare för lösningsacceleratorn Ansluten fabrik][cf-img-tsi-explorer]

## <a name="view-key-performance-indicators"></a>Visa KPI: er

Lösningen innehåller två KPI:er: *units per hour* (enheter per timme) och *energy used in kWh* (energi som används i kWh).

![KPI för lösningsacceleratorn Ansluten fabrik][cf-img-kpi]

1. Om du vill visa enheter per timme eller energi som används för någon nivå i hierarkin navigerar du till den vy som du vill visa. Enheter per timme och energi som används visas på panelen.

2. Om du vill analysera enheter per timme eller förbrukad energi för en nivå i hierarkin klickar du på mätaren på panelen **KPI:er**. En kontextpanel visas med Time Series Insights-visualiseringar som visar data från den senaste timmen, de senaste 24 timmarna och de senaste sju dagarna.

## <a name="scenario-review"></a>Scenariogranskning

I det här scenariot övervakade du OEE- och KPI-värden på instrumentpanelen. Sedan använde du Time Series Insights för att visa mer information och detaljerade telemetridata för OEE och KPI:er för att kunna identifiera avvikelser. Du använde också larmpanelen för att visa problem med dina fabriker och du använde tillgängliga alternativ för att åtgärda orsaken till larmet.

## <a name="other-features"></a>Andra funktioner

I följande avsnitt beskrivs några fler funktioner i lösningen Ansluten fabrik som inte finns med i föregående scenario.

## <a name="apply-filters"></a>Använda filter

1. Klicka på **tratten** för att visa en lista över tillgängliga filter på panelen med fabriksplatser eller på larmpanelen.

2. Filterpanelen visas.

    ![Filter för lösningsacceleratorn Ansluten fabrik][cf-img-alert-filter]

3. Välj önskat filter. Du kan också skriva fritext i filterfälten.

4. Sedan tillämpas filtret. Filterläget visas i instrumentpanelen med hjälp av en tratt som visas i tabellerna för fabriker och larm.

    ![Filter för lösningsacceleratorn Ansluten fabrik][cf-img-alert-filter-funnel]

    > [!NOTE]
    > Ett aktivt filter påverkar inte de OEE och KPI-värden som visas. Det filtrerar bara listinnehållet.

5. Om du vill ta bort ett filter klickar du på tratten och på filtret på kontextpanelen för filtret. Texten **Alla** visas i tabellerna för fabriker och larm.

## <a name="browse-an-opc-ua-server"></a>Navigera i en OPC UA-serverläsare

När du distribuerar lösningsacceleratorn etableras automatiskt simulerade OPC UA-servrar som du kan navigera i via lösningens webbläsare. Dessa servrar är *simulerade OPC UA-servrar*. Med simulerade servrar kan du enkelt experimentera med lösningsacceleratorn utan att du behöver distribuera verkliga, fysiska servrar. Om du vill ansluta en verklig OPC UA-server till lösningen kan du använda självstudien [Ansluta OPC UA-enheten till lösningsacceleratorn Ansluten fabrik][lnk-connect-cf].

1. Klicka på **bläddringsikonen** i navigeringsfältet i instrumentpanelen.

    ![Serverwebbläsare för lösningsacceleratorn Ansluten fabrik][cf-img-server-browser]

2. Välj en av servrarna i den förkonfigurerade listan. Listan visar de servrar som är distribuerade åt dig i lösningsacceleratorn.

    ![Serverurval för lösningsacceleratorn Ansluten fabrik][cf-img-server-choice]

3. Klicka på **Connect** (Anslut) för att visa en dialogruta för säkerhet. För simuleringen är det säkert att klicka på **Proceed** (Fortsätt).

4. Om du vill expandera någon av noderna i serverträdet klickar du på noden. Noder som publicerar telemetri har ett skalstreck bredvid sig.

    ![Serverträd för lösningsacceleratorn Ansluten fabrik][cf-img-server-tree]

5. Högerklicka på ett objekt om du vill läsa, skriva, publicera eller anropa noden. Vilka åtgärder som är tillgängliga beror på dina behörigheter och nodens attribut. Läsningsalternativet visar en kontextpanel med värdet för den specifika noden. Skrivalternativet visar en kontextpanel där du kan ange ett nytt värde. Anropsalternativet visar en nod där du kan ange parametrar för anropet.

## <a name="publish-a-node"></a>Publicera en nod

När du navigerar i en *simulerad OPC UA-server* kan du välja att publicera nya noder. Du kan analysera telemetri från dessa noder i lösningen. Med dessa *simulerade OPC UA-servrar* kan du enkelt experimentera med lösningsacceleratorn utan att distribuera verkliga fysiska enheter.

1. Bläddra till en nod i OPC UA-serverläsarträdet som du vill publicera.

2. Högerklicka på noden.

3. Välj **Publish** (Publicera).

    ![Ansluten fabrik publicerar noden][cf-img-publish-node]

4. En kontextpanel visas som anger att publiceringen är klar. Noden visas i vyn på stationsnivå med en bock bredvid den.

    ![Lyckad publicering av den förkonfigurerade lösningen Ansluten fabrik][cf-img-publish-success]

## <a name="command-and-control"></a>Kommando och kontroll

Med Ansluten fabrik kan du styra dina industriella enheter direkt från molnet. Du kan använda funktionen till att vidta åtgärder vid larm som genereras av enheten. Du kan till exempel skicka ett kommando till enheten från molnet. Tillgängliga kommandon finns i noden **StationCommands** i OPC UA-serverläsarträdet. I det här scenariot öppnar du en övertrycksventil på en monteringsstation i en produktionslinje i München. För att kunna styra funktioner måste du ha rollen **Administratör** för distributionen av lösningsacceleratorn.

1. Bläddra till noden **StationCommands** i OPC UA-serverläsarträdet.

2. Välj det kommando som du vill använda.

3. Högerklicka på noden.

4. Välj **Call** (Anropa).

    ![Anropskommando för lösningsacceleratorn Ansluten fabrik][cf-img-call-command]

5. En kontextpanel visas som anger vilken metod du anropar och eventuell parameterinformation.

6. Välj **Call** (Anropa).

    ![Anropskontext för lösningsacceleratorn Ansluten fabrik][cf-img-call-context]

7. Kontextpanelen uppdateras för att informera dig att metodanropet har slutförts. Du kan verifiera att anropet har slutförts genom att läsa av värdet för trycknoden som uppdaterades av anropet.

    ![Lyckat anrop för lösningsacceleratorn Ansluten fabrik][cf-img-call-success]

## <a name="behind-the-scenes"></a>I bakgrunden

När du distribuerar en lösningsaccelerator skapar distributionsprocessen flera resurser i den valda Azure-prenumerationen. Du kan visa dessa resurser på Azure-[portalen][lnk-portal]. Under distributionsprocessen skapas en **resursgrupp** med ett namn som baseras på det namn som du valde för lösningsacceleratorn :

![Lösningsaccelerator i Azure Portal][img-cf-portal]

Du kan visa inställningarna för varje resurs genom att välja resursen i listan över resurser i resursgruppen.

Du kan också visa källkoden för lösningsacceleratorn. Källkoden för lösningsacceleratorn Ansluten fabrik finns i GitHub-lagringsplatsen [azure-iot-connected-factory][lnk-cfgithub]:

När du är klar kan du ta bort lösningsacceleratorn från Azure-prenumerationen på webbplatsen [azureiotsuite.com][lnk-azureiotsuite]. På den här webbplatsen kan du enkelt ta bort alla resurser som etablerades när du skapade lösningsacceleratorn.

> [!NOTE]
> För att vara säker på att du tar bort allt som hör till lösningsacceleratorn tar du bort den från webbplatsen [azureiotsuite.com][lnk-azureiotsuite]. Ta inte bort resursgruppen i portalen.

## <a name="next-steps"></a>Nästa steg

Nu när du har distribuerat en fungerande lösningsaccelerator kan du fortsätta lära dig mer om IoT-lösningsacceleratorer genom att läsa följande artiklar:

* [Genomgång av lösningsacceleratorn Ansluten fabrik][lnk-rm-walkthrough]
* [Ansluta enheten till lösningsacceleratorn Ansluten fabrik][lnk-connect-cf]
* [Behörigheter på webbplatsen azureiotsuite.com][lnk-permissions]

[img-cf-home]:media/iot-accelerators-connected-factory-overview/cf-dashboard.png
[img-launch-solution]: media/iot-accelerators-connected-factory-overview/launch-cf.png
[cf-img-menu]: media/iot-accelerators-connected-factory-overview/cf-dashboard-menu.png
[cf-img-factories]:media/iot-accelerators-connected-factory-overview/cf-dashboard-factory.png
[cf-img-map]:media/iot-accelerators-connected-factory-overview/cf-dashboard-map.png
[cf-img-alerts]:media/iot-accelerators-connected-factory-overview/cf-dashboard-alerts.png
[cf-img-oee]:media/iot-accelerators-connected-factory-overview/cf-dashboard-oee.png
[cf-img-kpi]:media/iot-accelerators-connected-factory-overview/cf-dashboard-kpi.png
[cf-img-tsi-visualization]:media/iot-accelerators-connected-factory-overview/cf-dashboard-tsi.png
[cf-img-tsi-explorer]:media/iot-accelerators-connected-factory-overview/tsi-explorer.png
[cf-img-server-browser]: media/iot-accelerators-connected-factory-overview/cf-dashboard-browser.png
[cf-img-server-choice]: media/iot-accelerators-connected-factory-overview/cf-select-server.png
[cf-img-server-tree]:media/iot-accelerators-connected-factory-overview/cf-server-tree.png
[cf-img-publish-node]:media/iot-accelerators-connected-factory-overview/cf-publish-node1.png
[cf-img-publish-success]:media/iot-accelerators-connected-factory-overview/cf-publish-success.png
[cf-img-call-command]:media/iot-accelerators-connected-factory-overview/cf-command-and-control-call.png
[cf-img-call-context]:media/iot-accelerators-connected-factory-overview/cf-command-and-control-call-button.png
[cf-img-call-success]:media/iot-accelerators-connected-factory-overview/cf-command-and-control-succeed.png
[img-cf-portal]:media/iot-accelerators-connected-factory-overview/cf-resource-group.png
[cf-img-alert-filter]:media/iot-accelerators-connected-factory-overview/cf-filter.png
[cf-img-alert-filter-funnel]:media/iot-accelerators-connected-factory-overview/cf-filter-funnel.png

[lnk_free_trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-preconfigured-solutions]:iot-accelerators-what-are-solution-accelerators.md
[lnk-azureiotsuite]: https://www.azureiotsolutions.com
[lnk-portal]: http://portal.azure.com/
[lnk-cfgithub]: https://github.com/Azure/azure-iot-connected-factory
[lnk-rm-walkthrough]:iot-accelerators-connected-factory-sample-walkthrough.md
[lnk-connect-cf]:iot-accelerators-connected-factory-gateway-deployment.md
[lnk-permissions]: iot-accelerators-permissions.md
[lnk-faq]: iot-accelerators-faq.md
