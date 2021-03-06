---
title: Datadrivna felsökning i Azure Stream Analytics
description: Den här artikeln beskriver hur du felsöker Azure Stream Analytics-jobbet med jobb diagram och mått i Azure-portalen.
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/01/2017
ms.openlocfilehash: 3d50f96f3dea3646bb32a3a42d0248957dabf9f0
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/18/2018
---
# <a name="data-driven-debugging-by-using-the-job-diagram"></a>Datadrivna felsökning med hjälp av jobb-diagram

Jobbet diagram på den **övervakning** bladet i Azure-portalen kan hjälpa dig att visualisera dina jobb pipeline. Den visar indata, utdata och frågesteg. Du kan använda jobb diagrammet för att undersöka mätvärden för varje steg för att isolera snabbt källan för ett problem när du felsöker problem.

## <a name="using-the-job-diagram"></a>Med hjälp av jobb-diagram

I Azure-portalen när den är i ett Stream Analytics-jobb **stöd + felsökning**väljer **jobbet diagram**:

![Jobbet diagram med - plats](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-1.png)

Markera varje fråga steget för att se motsvarande avsnitt i en fråga Redigera fönstret. Ett mått diagram för steg visas i en längst ned på sidan.

![Jobbet diagram med - grundläggande jobb](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-2.png)

Om du vill se partitionerna i Azure Event Hubs indata, Välj **...** En snabbmeny visas. Du kan också se inkommande fusion.

![Jobbet diagram med - Utöka partition](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-3.png)

Om du vill se mått diagram för endast en partition markerar du partitionen-nod. Mätvärdena visas längst ned på sidan.

![Jobbet diagram med - flera mått](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-4.png)

Välj noden fusion mått diagram för en fusion visas. Följande diagram visar att inga händelser släpptes eller justeras.

![Jobbet diagram med - rutnätet](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-5.png)

Om du vill se information om värde och tid, pekar du på diagrammet.

![Jobbet diagram med - hovra](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-6.png)

## <a name="troubleshoot-by-using-metrics"></a>Felsöka med mått

Den **QueryLastProcessedTime** mått som anger när ett specifikt steg tog emot data. Genom att titta på topologi arbeta du bakåt från utdata processorn för att se vilka steg tar inte emot data. Om ett steg inte blir data, går du till frågesteg innan. Kontrollera om det föregående steget för frågan har ett tidsfönster och om tillräckligt med tid har förflutit för att överföra data. (Observera då windows är fäst på en timme.)
 
Om föregående frågesteg är en inkommande processor Använd inkommande mått för att besvara följande riktade frågor. De kan hjälpa dig att avgöra om ett jobb hämtar data från dess indatakällor. Granska varje partition om frågan är partitionerad.
 
### <a name="how-much-data-is-being-read"></a>Hur mycket data läses?

*   **InputEventsSourcesTotal** är antalet enheter läsa. Till exempel antal blobbar.
*   **InputEventsTotal** är antalet händelser som läsa. Det här måttet är tillgängligt per partition.
*   **InputEventsInBytesTotal** är antalet lästa byte.
*   **InputEventsLastArrivalTime** uppdateras med tiden för alla mottagna händelser i kö.
 
### <a name="is-time-moving-forward-if-actual-events-are-read-punctuation-might-not-be-issued"></a>Är tiden går vidare? Om faktiska händelser läses kanske inte skiljetecken utfärdas.

*   **InputEventsLastPunctuationTime** anger när ett skiljetecken utfärdades för att se till att tiden går framåt. Om skiljetecken inte är utfärdat kan dataflöde blockeras.
 
### <a name="are-there-any-errors-in-the-input"></a>Finns det några fel i indata?

*   **InputEventsEventDataNullTotal** antal händelser som har null-data.
*   **InputEventsSerializerErrorsTotal** antal händelser som inte gick att deserialisera korrekt.
*   **InputEventsDegradedTotal** antal händelser som har haft problem än med deserialisering.
 
### <a name="are-events-being-dropped-or-adjusted"></a>Är händelser tas bort eller justeras?

*   **InputEventsEarlyTotal** är antalet händelser som har ett Programtidsstämpel innan den övre storleksgränsen.
*   **InputEventsLateTotal** är antalet händelser som har ett Programtidsstämpel efter den övre storleksgränsen.
*   **InputEventsDroppedBeforeApplicationStartTimeTotal** är antalet händelser tas bort innan jobbets starttid.
 
### <a name="are-we-falling-behind-in-reading-data"></a>Vi sjunker vid läsning av data?

*   **Inkommande händelser eftersläpande (totalt)** får du reda på hur många fler meddelanden behöver läsas för Event Hubs och Azure IoT Hub-indata. När det här antalet är större än 0, innebär det att jobbet inte kan bearbeta data så fort det kommer. I det här fallet kan du behöva öka antalet enheter för strömning och/eller se till att jobbet kan paralleliserad. Du kan se mer information om detta på den [frågan parallellisering sidan](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization). 


## <a name="get-help"></a>Få hjälp
Mer hjälp, försök vår [Azure Stream Analytics-forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics). 

## <a name="next-steps"></a>Nästa steg
* [Introduktion till Stream Analytics](stream-analytics-introduction.md)
* [Kom igång med Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Skala Stream Analytics-jobb](stream-analytics-scale-jobs.md)
* [Språkreferens för Stream Analytics-fråga](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Strömma Analytics management REST API-referens](https://msdn.microsoft.com/library/azure/dn835031.aspx)
