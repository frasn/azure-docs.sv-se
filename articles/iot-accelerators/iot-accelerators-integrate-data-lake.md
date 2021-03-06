---
title: Integrera Fjärrövervaknings-lösning med Azure Data Lake Store | Microsoft Docs
description: Lär dig hur du integrerar Fjärrövervaknings-lösning med Azure Data Lake Store med hjälp av en Azure Stream Analytics-jobbet.
author: philmea
manager: timlt
ms.author: philmea
ms.date: 04/29/2018
ms.topic: conceptual
ms.service: iot-accelerators
services: iot-accelerators
ms.openlocfilehash: 3bd29e348fd067c12def8ca36fbdc1d7e35b2874
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34627594"
---
# <a name="integrate-the-remote-monitoring-solution-with-azure-data-lake-store"></a>Integrera Fjärrövervaknings-lösning med Azure Data Lake Store

Du kan ha avancerade analyser krav utöver vad som erbjuds i lösningen Fjärrövervaknings. Azure Data Lake Store är idealiskt för det här programmet eftersom det kan lagra data från massiv och skilda datauppsättningar samt integrera med Azure Data Lake Analytics att tillhandahålla på begäran.

I den här anvisningar använder du ett Azure Stream Analytics-jobb att dataströmmen data från IoT-hubben i din lösning för övervakning av fjärråtkomst till ett Azure Data Lake Store.

## <a name="prerequisites"></a>Förutsättningar

Du behöver följande för att slutföra den här anvisningar:

* [Distribuera fjärråtkomst övervakning solution accelerator](iot-accelerators-remote-monitoring-deploy.md).
  * Fjärrövervaknings lösningen ska distribuera IoT-hubb och Azure Stream Analytics-jobbet som används i den här artikeln i din Azure-prenumeration.
* [Distribuera ett Azure Data Lake Store](../data-lake-store/data-lake-store-get-started-portal.md)
  * Din Data Lake Store ska distribueras till samma region som din lösning för övervakning av fjärråtkomst.
  * [Skapa en mapp](../data-lake-store/data-lake-store-get-started-portal.md#createfolder) med namnet ”strömning” i ditt konto.

## <a name="create-a-consumer-group"></a>Skapa en konsumentgrupp

Skapa en dedikerad konsumentgrupp i IoT-hubben i Fjärrövervaknings lösningen. Detta kommer att användas av Stream Analytics-jobbet för strömmande data till Data Lake Store.

> [!NOTE]
> Konsumentgrupper används av program för att hämta data från Azure IoT Hub. Du bör skapa en ny konsumentgrupp för varje fem utdata konsumenter. Du kan skapa upp till 32 konsumentgrupper.

1. Logga in på Azure Portal.

1. I Azure-portalen klickar du på den **moln Shell** knappen.

    ![Starta företagsportal-ikon](./media/iot-accelerators-integrate-data-lake/portal-launch-icon.png)

1. Kör detta kommando för att skapa en ny konsumentgrupp:

```azurecli-interactive
az iot hub consumer-group create --hub-name contoso-rm30263 --name streamanalyticsjob --resource-group contoso-rm
```

> [!NOTE]
> Använd resursgrupp och IoT-hubb namn från din lösning för övervakning av fjärråtkomst.

## <a name="create-stream-analytics-job"></a>Skapa Stream Analytics-jobbet

Skapa ett Azure Stream Analytics-jobb för att strömma data från IoT-hubb till din Azure Data Lake store.

1. Klicka på **skapar du en resurs**Sakernas Internet från Marketplace och välj klickar du på **Stream Analytics-jobbet**.

    ![Nya Stream Analytics-jobbet](./media/iot-accelerators-integrate-data-lake/new-stream-analytics-job.png)

1. Ange ett Jobbnamn och välj lämplig prenumeration och resurs.

1. Välj en plats i den nära eller i samma region som din Data Lake Store. Här använder vi östra USA.

1. Se till att lämna värd-miljö som standard **moln**.

1. Klicka på **Skapa**.

    ![Skapa Stream Analytics-jobbet](./media/iot-accelerators-integrate-data-lake/create-stream-analytics-job.png)

## <a name="configure-the-stream-analytics-job"></a>Konfigurera Stream Analytics-jobbet

1. Gå till den **Stream Analytics-jobbet** i resursgruppen Fjärrövervaknings lösning.

1. På sidan Översikt **indata**.

    ![Översiktssidan](./media/iot-accelerators-integrate-data-lake/stream-analytics-overview.png)

1. Klicka på **lägga till strömindata** och välj **IoT-hubb** från listrutan.

    ![Lägga till indata](./media/iot-accelerators-integrate-data-lake/stream-analytics-add-input.png)

1. Ange ett indata-alias för på den nya inkommande fliken **IoTHub**.

1. Konsumenten grupp listrutan, Välj konsumentgrupp som du skapade tidigare. Här använder vi **streamanalyticsjob**.

    ![Välj indata](./media/iot-accelerators-integrate-data-lake/stream-analytics-new-input.png)

1. Klicka på **Spara**.

1. På sidan Översikt **utdata**.

    ![Lägg till Data Lake Store](./media/iot-accelerators-integrate-data-lake/stream-analytics-overview-2.png)

1. Klicka på **Lägg till** och välj **Datasjölager** från listrutan.

    ![Lägga till utdata](./media/iot-accelerators-integrate-data-lake/stream-analytics-output.png)

1. På fliken utdata, anger du ett kolumnalias för **DataLakeStore**.

1. Välj Data Lake Store-konto som du skapade i föregående steg och ange mappstrukturen dataströmmen data till arkivet.

1. Ange i formatfältet **/streaming/ {date} / {time}**. Lämna standardformatet för datum för åååå-MM/DD och tidsformat hh.

    ![Ange mappstruktur](./media/iot-accelerators-integrate-data-lake/stream-analytics-new-output.png)

1. Klicka på **auktorisera**.

    Du måste godkänna med Data Lake Store att ge skrivåtkomst för Stream analytics-jobbet till filsystemet.

    ![Auktorisera Stream Analytics till Data Lake Store](./media/iot-accelerators-integrate-data-lake/stream-analytics-out-authorize.png)

    Du ser ett popup-fönster och när popup-fönstret stängs auktorisera knappen är avmarkerad när auktoriseringen är klar.

    > [!NOTE]
    > Om du ser ett fel i popup-fönstret, öppna ett nytt webbläsarfönster i Incognito-läge och försök igen.

1. Klicka på **Spara**.

## <a name="edit-the-stream-analytics-query"></a>Redigera Stream Analytics-fråga

Azure Stream Analytics använder ett SQL-liknande frågespråk för att ange ingen källa som strömmas data, transformera data som önskad och utdata till en mängd olika lagrings- och mål.

1. På fliken Översikt **redigera frågan**.

    ![Redigera fråga](./media/iot-accelerators-integrate-data-lake/stream-analytics-edit-query.png)

1. I Redigeraren för frågan ersätter [YourOutputAlias] och [YourInputAlias] platshållarna med värdena som du definierade tidigare.

    ```sql
    SELECT
        *, System.Timestamp as time
    INTO
        DataLakeStore
    FROM
        IoTHub
    ```

    ![Stream Analytics-fråga](./media/iot-accelerators-integrate-data-lake/stream-analytics-query.png)

1. Klicka på **Spara**.
1. Klicka på **Ja** att godkänna ändringarna.

## <a name="start-the-stream-analytics-job"></a>Starta Stream Analytics-jobbet

1. På fliken Översikt **starta**.

    ![Starta Stream Analytics-jobbet](./media/iot-accelerators-integrate-data-lake/stream-analytics-start.png)

1. På fliken Start jobbet **anpassad**.

1. Ange anpassade tid att gå tillbaka flera timmar för att hämta data från när enheten har startats strömning.

1. Klicka på **starta**.

    ![Välj Anpassad datum](./media/iot-accelerators-integrate-data-lake/stream-analytics-start-custom.png)

    Vänta tills jobbet blir körs, om du får felmeddelanden som det kan vara från din fråga, se till att kontrollera att syntaxen är korrekt.

    ![Köra jobb](./media/iot-accelerators-integrate-data-lake/stream-analytics-running.png)

    Direktuppspelningsjobbet ska börja läsa data från IoT-hubb och lagra data i Data Lake Store. Det kan ta några minuter för att data ska börja ska visas i din Data Lake Store.

## <a name="explore-the-streaming-data"></a>Utforska strömmande data

1. Gå till din Data Lake Store.

1. På fliken Översikt **Data explorer**.

1. I Data explorer, öka detaljnivån till den **/ strömning** mapp. Mappar som skapas med formatet ÅÅÅÅ/MM/DD/HH visas.

    ![Utforska strömmande Data](./media/iot-accelerators-integrate-data-lake/data-lake-store-data-explorer.png)

    Ser du json-filer med en fil per timme.

    ![Utforska strömmande Data](./media/iot-accelerators-integrate-data-lake/data-lake-store-file-preview.png)

## <a name="next-steps"></a>Nästa steg

Azure Data Lake Analytics kan användas för att utföra stordataanalys på din Data Lake Store-datauppsättningar. Läs mer på den [dokumentation för Data Lake Analytics](https://docs.microsoft.com/en-us/azure/data-lake-analytics).
