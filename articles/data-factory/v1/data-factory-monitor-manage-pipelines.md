---
title: Övervaka och hantera pipelines med hjälp av Azure portal och PowerShell | Microsoft Docs
description: Lär dig hur du använder Azure-portalen och Azure PowerShell för att övervaka och hantera Azure datafabriker och pipelines som du har skapat.
services: data-factory
documentationcenter: ''
author: sharonlo101
manager: craigg
ms.assetid: 9b0fdc59-5bbe-44d1-9ebc-8be14d44def9
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/30/2018
ms.author: shlo
robots: noindex
ms.openlocfilehash: b6cfe6ba510f1e7ed1b448d99fb8a71bb94053e8
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34620691"
---
# <a name="monitor-and-manage-azure-data-factory-pipelines-by-using-the-azure-portal-and-powershell"></a>Övervaka och hantera Azure Data Factory pipelines med hjälp av Azure portal och PowerShell
> [!div class="op_single_selector"]
> * [Med hjälp av Azure portal/Azure PowerShell](data-factory-monitor-manage-pipelines.md)
> * [Med hjälp av övervakning och Management-appen](data-factory-monitor-manage-app.md)

> [!NOTE]
> Den här artikeln gäller för version 1 av Data Factory, som är allmänt tillgänglig (GA). Om du använder version 2 av Data Factory-tjänsten, som finns i förhandsgranskningen, se [övervaka och hantera Data Factory pipelines i version 2](../monitor-visually.md).

Den här artikeln beskriver hur du övervaka, hantera och felsöka din pipelines med hjälp av Azure portal och PowerShell.

> [!IMPORTANT]
> Övervakning och hantering av programmet ger ett bättre stöd för att övervaka och hantera dina data pipelines och felsöka eventuella problem. Mer information om hur du använder programmet finns [övervaka och hantera Data Factory pipelines med hjälp av övervakning och hantering av appen](data-factory-monitor-manage-app.md). 

> [!IMPORTANT]
> Azure Data Factory version 1 nu använder den nya [Azure-Monitor aviseringar infrastruktur](../../monitoring-and-diagnostics/monitor-alerts-unified-usage.md). Den gamla aviseringsdata infrastrukturen är föråldrad. Därför konfigurerade aviseringarna befintliga för version 1 data fabriker inte längre att fungera. Befintliga aviseringar för v1 datafabriker migreras inte automatiskt. Du måste återskapa dessa aviseringar på nya aviseringar infrastruktur. Logga in på Azure portal och välj **övervakaren** skapa nya aviseringar om mått (till exempel misslyckade körs eller lyckade körs) för din version 1 datafabriker.

## <a name="understand-pipelines-and-activity-states"></a>Förstå pipelines och aktivitet tillstånd
Med hjälp av Azure portal, kan du:

* Visa din data factory som ett diagram.
* Visa aktiviteter i en pipeline.
* Visa inkommande och utgående datauppsättningar.

Det här avsnittet beskriver också hur en datamängdssektor övergångar från ett tillstånd till ett annat tillstånd.   

### <a name="navigate-to-your-data-factory"></a>Navigera till din data factory
1. Logga in på [Azure Portal](https://portal.azure.com).
2. Klicka på **datafabriker** på menyn till vänster. Om du inte ser det klickar du på **fler tjänster >**, och klicka sedan på **datafabriker** under den **INTELLIGENCE + analys** kategori.

   ![Bläddra igenom alla > datafabriker](./media/data-factory-monitor-manage-pipelines/browseall-data-factories.png)
3. På den **datafabriker** bladet välj datafabriken som du är intresserad av.

    ![Välja datafabrik](./media/data-factory-monitor-manage-pipelines/select-data-factory.png)

   Du bör se startsidan för datafabriken.

   ![Bladet Datafabrik](./media/data-factory-monitor-manage-pipelines/data-factory-blade.png)

#### <a name="diagram-view-of-your-data-factory"></a>Diagramvy i din data factory
Den **Diagram** vy av en datafabrik som tillhandahåller en och samma plats att övervaka och hantera data factory och dess tillgångar. Se den **Diagram** visa i din data factory, klickar du på **Diagram** på startsidan för data factory.

![Diagramvy](./media/data-factory-monitor-manage-pipelines/diagram-view.png)

Du kan zooma in, Zooma ut, Zooma in för att passa, Zooma till 100%, låsa layouten för diagrammet och placera pipelines och datauppsättningar automatiskt. Du kan också se informationen om härkomst (det vill säga visa överordnade och underordnade objekt av valda objekt).

### <a name="activities-inside-a-pipeline"></a>Aktiviteter i en pipeline
1. Högerklicka på pipeline och klicka sedan på **öppna pipeline** att se alla aktiviteter i pipelinen, tillsammans med indata- och datauppsättningar för aktiviteter. Den här funktionen är användbart när din pipeline innehåller mer än en aktivitet och du vill förstå operativa härkomst av en enkel rörledning.

    ![Menyn Öppna pipeline](./media/data-factory-monitor-manage-pipelines/open-pipeline-menu.png)     
2. I följande exempel visas en kopia aktivitet i pipelinen med indata och utdata. 

    ![Aktiviteter i en pipeline](./media/data-factory-monitor-manage-pipelines/activities-inside-pipeline.png)
3. Du kan gå tillbaka till startsidan för datafabriken genom att klicka på **datafabriken** länk i sökvägen i det övre vänstra hörnet.

    ![Gå tillbaka till data factory](./media/data-factory-monitor-manage-pipelines/navigate-back-to-data-factory.png)

### <a name="view-the-state-of-each-activity-inside-a-pipeline"></a>Visa status för varje aktivitet i en pipeline
Du kan visa det aktuella tillståndet för en aktivitet genom att visa status för alla datauppsättningar som produceras av aktiviteten.

Genom att dubbelklicka på den **OutputBlobTable** i den **Diagram**, du kan se alla segment som genereras av en annan aktivitet körs i en pipeline. Du kan se att kopieringsaktiviteten kördes har för de senaste åtta timmarna och producerade sektorer i den **klar** tillstånd.  

![Tillståndet för pipeline](./media/data-factory-monitor-manage-pipelines/state-of-pipeline.png)

Dataset-segment i datafabriken kan ha en av följande status:

<table>
<tr>
    <th align="left">Status</th><th align="left">Undertillstånd</th><th align="left">Beskrivning</th>
</tr>
<tr>
    <td rowspan="8">Väntar</td><td>ScheduleTime</td><td>Tiden har inte inne för att köra sektorn.</td>
</tr>
<tr>
<td>DatasetDependencies</td><td>Uppströmsberoendena är inte redo.</td>
</tr>
<tr>
<td>ComputeResources</td><td>Beräkningsresurserna är inte tillgängliga.</td>
</tr>
<tr>
<td>ConcurrencyLimit</td> <td>Alla aktivitetsinstanserna är upptagna med att köra andra sektorer.</td>
</tr>
<tr>
<td>ActivityResume</td><td>Aktiviteten har pausats och kan inte köra sektorerna förrän aktiviteten återupptas.</td>
</tr>
<tr>
<td>Försök igen</td><td>Aktivitetskörningen försöks.</td>
</tr>
<tr>
<td>Validering</td><td>Verifieringen har inte startat ännu.</td>
</tr>
<tr>
<td>ValidationRetry</td><td>Verifieringen väntar på att göras.</td>
</tr>
<tr>
<tr>
<td rowspan="2">Pågår</td><td>Verifierar</td><td>Verifiering pågår.</td>
</tr>
<td>-</td>
<td>Sektorn behandlas.</td>
</tr>
<tr>
<td rowspan="4">Misslyckad</td><td>För lång tid</td><td>Aktivitetskörningen tog längre tid än vad som tillåts av aktiviteten.</td>
</tr>
<tr>
<td>Avbrutna</td><td>Sektorn avbröts av en användare.</td>
</tr>
<tr>
<td>Validering</td><td>Verifieringen misslyckades.</td>
</tr>
<tr>
<td>-</td><td>Det gick inte att vara genereras och/eller verifiera sektorn.</td>
</tr>
<td>Klar</td><td>-</td><td>Sektorn är klar att förbrukas.</td>
</tr>
<tr>
<td>Hoppades över</td><td>Ingen</td><td>Sektorn är inte bearbetas.</td>
</tr>
<tr>
<td>Ingen</td><td>-</td><td>En sektor som brukade finnas med en annan status, men den har återställts.</td>
</tr>
</table>



Du kan visa information om en sektor genom att klicka på en post i sektorn på den **nyligen uppdaterat segment** bladet.

![Sektorn information](./media/data-factory-monitor-manage-pipelines/slice-details.png)

Om sektorn har körts flera gånger, ser du flera rader i den **aktiviteten körs** lista. Du kan visa information om en aktivitet som kör genom att klicka på Kör post i den **aktiviteten körs** lista. I listan visas alla loggfiler, tillsammans med ett felmeddelande om det finns en. Den här funktionen är användbar för att visa loggar och felsökningsloggar utan att behöva lämna din data factory.

![Aktivitetskörningsinformation](./media/data-factory-monitor-manage-pipelines/activity-run-details.png)

Om sektorn inte i den **klar** tillstånd, kan du se sektorer uppströms som inte är klar och blockerar aktuellt segment från att köras i den **sektorer uppströms som inte är redo** lista. Den här funktionen är användbart när din sektorn är i **väntar på** tillstånd och du vill förstå uppströmsberoendena sektorn väntar på.

![Sektorer uppströms som inte är klara](./media/data-factory-monitor-manage-pipelines/upstream-slices-not-ready.png)

### <a name="dataset-state-diagram"></a>DataSet tillståndsdiagram
När du distribuerar en datafabrik och pipelines har en ogiltig aktiv period, sektorer datauppsättningen övergång från ett tillstånd till en annan. För närvarande följande sektorn status i följande tillståndsdiagram:

![Tillståndsdiagram](./media/data-factory-monitor-manage-pipelines/state-diagram.png)

Dataset tillstånd övergången flödet i data factory är följande: väntar -> förlopp/i-pågående (verifierar) -> klar/misslyckades.

Sektorn startar i en **väntar på** tillstånd, väntar på villkor som måste uppfyllas innan den körs. Sedan aktiviteten startar körning och sektorn försätts i ett **pågående** tillstånd. Aktivitetskörningen kan lyckas eller misslyckas. Sektorn är markerad som **klar** eller **misslyckades**, baserat på resultatet av körningen.

Du kan återställa sektorn gå tillbaka från den **klar** eller **misslyckades** till den **väntar på** tillstånd. Du kan också markera segment läget till **hoppa över**, vilket förhindrar att aktiviteten körs och icke-bearbetning av sektorn.

## <a name="pause-and-resume-pipelines"></a>Pausa och återuppta pipelines
Du kan hantera dina pipelines med hjälp av Azure PowerShell. Du kan till exempel pausa och återuppta pipelines genom att köra Azure PowerShell-cmdlets. 

> [!NOTE] 
> Diagramvyn stöder inte pausa och återuppta pipelines. Använd övervaknings- och hantera program om du vill använda ett användargränssnitt. Mer information om hur du använder programmet finns [övervaka och hantera Data Factory pipelines med hjälp av övervakning och hantering av appen](data-factory-monitor-manage-app.md) artikel. 

Du kan Pausa/Pausa pipelines med hjälp av den **pausa AzureRmDataFactoryPipeline** PowerShell-cmdlet. Denna cmdlet är användbar när du inte vill köra din pipelines förrän problemet har åtgärdats. 

```powershell
Suspend-AzureRmDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Exempel:

```powershell
Suspend-AzureRmDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

När problemet har åtgärdats med pipeline, kan du återuppta pausade pipeline genom att köra följande PowerShell-kommando:

```powershell
Resume-AzureRmDataFactoryPipeline [-ResourceGroupName] <String> [-DataFactoryName] <String> [-Name] <String>
```
Exempel:

```powershell
Resume-AzureRmDataFactoryPipeline -ResourceGroupName ADF -DataFactoryName productrecgamalbox1dev -Name PartitionProductsUsagePipeline
```

## <a name="debug-pipelines"></a>Felsöka pipelines
Azure Data Factory ger omfattande funktioner för dig att felsöka pipelines med hjälp av Azure-portalen och Azure PowerShell.

> [!NOTE] 
> Det är mycket enklare att Felsökte fel med övervakning & Management-appen. Mer information om hur du använder programmet finns [övervaka och hantera Data Factory pipelines med hjälp av övervakning och hantering av appen](data-factory-monitor-manage-app.md) artikel. 

### <a name="find-errors-in-a-pipeline"></a>Sök efter fel i en pipeline
Om aktiviteten körs inte i en pipeline, är dataset som produceras av pipelinen i ett feltillstånd på grund av felet. Du kan felsöka och felsöka i Azure Data Factory med hjälp av följande metoder.

#### <a name="use-the-azure-portal-to-debug-an-error"></a>Använda Azure portal för att felsöka ett fel
1. På den **tabell** bladet på problemet sektor som har det **Status** inställd på **misslyckades**.

   ![Tabell bladet med problem](./media/data-factory-monitor-manage-pipelines/table-blade-with-error.png)
2. På den **datasektorn** bladet, klickar du på aktiviteten körs som misslyckades.

   ![Datasektorn med ett fel](./media/data-factory-monitor-manage-pipelines/dataslice-with-error.png)
3. På den **aktivitet köras information** bladet som du kan hämta de filer som är associerade med HDInsight-bearbetning. Klicka på **hämta** för Status/stderr att ladda ned felloggen som innehåller information om felet.

   ![Aktiviteten kör informationsbladet med fel](./media/data-factory-monitor-manage-pipelines/activity-run-details-with-error.png)     

#### <a name="use-powershell-to-debug-an-error"></a>Använd PowerShell för att felsöka ett fel
1. Starta **PowerShell**.
2. Kör den **Get-AzureRmDataFactorySlice** kommandot för att se sektorerna och deras status. Du bör se ett segment med statusen **misslyckades**.        

    ```powershell   
    Get-AzureRmDataFactorySlice [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime] <DateTime> [[-EndDateTime] <DateTime> ] [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```   
   Exempel:

    ```powershell   
    Get-AzureRmDataFactorySlice -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime 2014-05-04 20:00:00
    ```

   Ersätt **StartDateTime** med starttiden för din pipeline. 
3. Kör nu den **Get-AzureRmDataFactoryRun** för att hämta information om aktiviteten kör för sektorn.

    ```powershell   
    Get-AzureRmDataFactoryRun [-ResourceGroupName] <String> [-DataFactoryName] <String> [-DatasetName] <String> [-StartDateTime]
    <DateTime> [-Profile <AzureProfile> ] [ <CommonParameters>]
    ```

    Exempel:

    ```powershell   
    Get-AzureRmDataFactoryRun -ResourceGroupName ADF -DataFactoryName LogProcessingFactory -DatasetName EnrichedGameEventsTable -StartDateTime "5/5/2014 12:00:00 AM"
    ```

    Värdet för StartDateTime är starttiden för sektorn fel/problem som du antecknade från föregående steg. Datum och tid ska omges av dubbla citattecken.
4. Du bör se utdata med information om felet som liknar följande:

    ```   
    Id                      : 841b77c9-d56c-48d1-99a3-8c16c3e77d39
    ResourceGroupName       : ADF
    DataFactoryName         : LogProcessingFactory3
    DatasetName               : EnrichedGameEventsTable
    ProcessingStartTime     : 10/10/2014 3:04:52 AM
    ProcessingEndTime       : 10/10/2014 3:06:49 AM
    PercentComplete         : 0
    DataSliceStart          : 5/5/2014 12:00:00 AM
    DataSliceEnd            : 5/6/2014 12:00:00 AM
    Status                  : FailedExecution
    Timestamp               : 10/10/2014 3:04:52 AM
    RetryAttempt            : 0
    Properties              : {}
    ErrorMessage            : Pig script failed with exit code '5'. See wasb://        adfjobs@spestore.blob.core.windows.net/PigQuery
                                    Jobs/841b77c9-d56c-48d1-99a3-
                8c16c3e77d39/10_10_2014_03_04_53_277/Status/stderr' for
                more details.
    ActivityName            : PigEnrichLogs
    PipelineName            : EnrichGameLogsPipeline
    Type                    :
    ```
5. Du kan köra den **spara AzureRmDataFactoryLog** med ID-värde som du ser från utdata och hämta filerna med hjälp av den **- DownloadLogsoption** för cmdleten.

    ```powershell
    Save-AzureRmDataFactoryLog -ResourceGroupName "ADF" -DataFactoryName "LogProcessingFactory" -Id "841b77c9-d56c-48d1-99a3-8c16c3e77d39" -DownloadLogs -Output "C:\Test"
    ```

## <a name="rerun-failures-in-a-pipeline"></a>Kör fel i en pipeline

> [!IMPORTANT]
> Det är enklare att felsöka och kör misslyckade segment med hjälp av övervakning & Management-appen. Mer information om hur du använder programmet finns [övervaka och hantera Data Factory pipelines med hjälp av övervakning och hantering av appen](data-factory-monitor-manage-app.md). 

### <a name="use-the-azure-portal"></a>Använda Azure-portalen
När du felsöker och felsöka fel i en pipeline, kan du köra fel genom att gå till fel segment och klicka på den **kör** knappen i kommandofältet.

![Kör en misslyckad sektor](./media/data-factory-monitor-manage-pipelines/rerun-slice.png)

I fallet sektorn valideringen misslyckades på grund av en princip-fel (till exempel om data inte är tillgängligt), kan du åtgärda felet och validera igen genom att klicka på den **verifiera** knappen i kommandofältet.

![Åtgärda felen och validera](./media/data-factory-monitor-manage-pipelines/fix-error-and-validate.png)

### <a name="use-azure-powershell"></a>Använda Azure PowerShell
Du kan köra fel med hjälp av den **Set AzureRmDataFactorySliceStatus** cmdlet. Finns det [Set AzureRmDataFactorySliceStatus](https://msdn.microsoft.com/library/mt603522.aspx) avsnittet annan information om cmdlet och syntax.

**Exempel:**

I följande exempel anger status för alla segment för tabellen 'DAWikiAggregatedData' till väntar i Azure data factory 'WikiADF'.

Uppdateringstyp är inställd på 'UpstreamInPipeline', vilket innebär att statusen för varje segment för tabellen och alla beroende (överordnad) tabeller är inställt på ”väntande”. Andra möjliga värde för den här parametern är 'Person'.

```powershell
Set-AzureRmDataFactorySliceStatus -ResourceGroupName ADF -DataFactoryName WikiADF -DatasetName DAWikiAggregatedData -Status Waiting -UpdateType UpstreamInPipeline -StartDateTime 2014-05-21T16:00:00 -EndDateTime 2014-05-21T20:00:00
```
## <a name="create-alerts-in-the-azure-portal"></a>Skapa aviseringar i Azure-portalen

1.  Logga in på Azure portal och välj **övervakaren -> aviseringar** att öppna sidan aviseringar.

    ![Öppna sidan aviseringar.](media/data-factory-monitor-manage-pipelines/v1alerts-image1.png)

2.  Välj **+ ny varningsregeln** att skapa en ny avisering.

    ![Skapa en ny avisering](media/data-factory-monitor-manage-pipelines/v1alerts-image2.png)

3.  Definiera den **Varna villkoret**. (Se till att välja **datafabriker** i den **filtrera efter resurstyp** fält.) Du kan även ange värden för **dimensioner**.

    ![Definiera Aviseringstillståndet - väljer mål](media/data-factory-monitor-manage-pipelines/v1alerts-image3.png)

    ![Definiera villkoret för aviseringen – Lägg till aviseringsvillkoren](media/data-factory-monitor-manage-pipelines/v1alerts-image4.png)

    ![Definiera villkoret för aviseringen – Lägg till avisering logik](media/data-factory-monitor-manage-pipelines/v1alerts-image5.png)

4.  Definiera den **aviseringsinformation**.

    ![Definiera varningsinformationen](media/data-factory-monitor-manage-pipelines/v1alerts-image6.png)

5.  Definiera den **grupp**.

    ![Definiera åtgärd gruppen – skapa en ny grupp](media/data-factory-monitor-manage-pipelines/v1alerts-image7.png)

    ![Definiera åtgärd grupp - egenskaper](media/data-factory-monitor-manage-pipelines/v1alerts-image8.png)

    ![Definiera åtgärd grupp - ny grupp skapas](media/data-factory-monitor-manage-pipelines/v1alerts-image9.png)

## <a name="move-a-data-factory-to-a-different-resource-group-or-subscription"></a>Flytta en datafabrik till en annan resursgrupp eller prenumeration
Du kan flytta en datafabrik till en annan resursgrupp eller en annan prenumeration med hjälp av den **flytta** kommandot liggande knappen på startsidan i din data factory.

![Flytta data factory](./media/data-factory-monitor-manage-pipelines/MoveDataFactory.png)

Du kan också flytta alla relaterade resurser (till exempel aviseringar som är associerade med data factory), tillsammans med datafabriken.

![Flytta dialogrutan resurser](./media/data-factory-monitor-manage-pipelines/MoveResources.png)
