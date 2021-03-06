---
title: Analysera videor med Azure Media Services | Microsoft Docs
description: Följ stegen i den här självstudien för att analysera videor med Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/09/2018
ms.author: juliako
ms.openlocfilehash: 0fdc8c6dc9fae96a79e2ab2b05b7db3012834c1e
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/20/2018
ms.locfileid: "34362302"
---
# <a name="tutorial-analyze-videos-with-azure-media-services"></a>Självstudie: Analysera videor med Azure Media Services 

Den här självstudien visar hur du analyserar videor med Azure Media Services. Det finns många scenarier där du kanske vill veta mer om inspelat video- eller ljudinnehåll. Organisationer kan till exempel för att uppnå högre kundnöjdhet köra tal-till-text-bearbetning och konvertera inspelningar från kundtjänst till en sökbar katalog med index och instrumentpaneler. Det blir då lättare att få fler insikter om verksamheten, till exempel en lista med vanliga klagomål, källor till sådana klagomål osv.

I den här självstudiekursen lär du dig att:    

> [!div class="checklist"]
> * Starta Azure Cloud Shell
> * Skapa ett Media Services-konto
> * Åtkomst till Media Services API
> * Konfigurera exempelappen
> * Granska exempelkoden i detalj
> * Kör appen
> * Granska utdatan
> * Rensa resurser

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Nödvändiga komponenter

Om du inte har Visual Studio installerat kan du hämta [Visual Studio Community 2017](https://www.visualstudio.com/thank-you-downloading-visual-studio/?sku=Community&rel=15).

## <a name="download-the-sample"></a>Hämta exemplet

Klona en GitHub-lagringsplats som innehåller .NET-exemplet till din dator med följande kommando:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

[!INCLUDE [media-services-cli-create-v3-account-include](../../../includes/media-services-cli-create-v3-account-include.md)]

[!INCLUDE [media-services-v3-cli-access-api-include](../../../includes/media-services-v3-cli-access-api-include.md)]

## <a name="examine-the-sample-code-in-detail"></a>Granska exempelkoden i detalj

Det här avsnittet går igenom funktionerna som definierades i filen [Program.cs](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/AnalyzeVideos/Program.cs) för projektet *AnalyzeVideos*.

### <a name="start-using-media-services-apis-with-net-sdk"></a>Börja använda API:er för Media Services med .NET SDK

Om du vill börja använda API:er för Media Services med .NET, måste du skapa ett **AzureMediaServicesClient**-objekt. När du skapar objektet måste du ange de autentiseringsuppgifter som krävs för att klienten ska kunna ansluta till Azure med hjälp av Azure AD. Först måste du hämta en token och sedan skapa ett **ClientCredential**-objekt från den token som returnerades. I den kod som du klonade i början av artikeln används objektet **ArmClientCredential** för att hämta token.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateMediaServicesClient)]

### <a name="create-an-output-asset-to-store-the-result-of-a-job"></a>Skapa en utdatatillgång där resultatet av ett jobb lagras 

[Utdatatillgången](https://docs.microsoft.com/rest/api/media/assets) lagrar resultatet av ditt jobb. Projektet definierar funktionen **DownloadResults** som laddar ner resultaten från utdatatillgången till mappen ”utdata”, så att du kan se vad du har fått.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CreateOutputAsset)]

### <a name="create-a-transform-and-a-job-that-analyzes-videos"></a>Skapa en transformering och ett jobb som analyserar videor

När kodningen eller bearbetningen av innehåll i Media Services görs, konfigureras vanligtvis kodningsinställningarna som ett recept. Du skickar sedan ett **Jobb** som tillämpar receptet på en video. Genom att skicka nya jobb för varje ny video, tillämpar du receptet på alla videor i biblioteket. Ett recept i Media Services kallas för en **Transformering**. Mer information finns i [Transformeringar och jobb](transform-concept.md). Det exempel som beskrivs i självstudien definierar ett recept som analyserar den angivna videon. 

#### <a name="transform"></a>Transformering

När du skapar en ny instans för en [Transformering](https://docs.microsoft.com/rest/api/media/transforms), måste du ange vilken utdata du vill att den ska skapa. Den obligatoriska parametern är objektet **TransformOutput** som visas i koden ovan. Varje **TransformOutput** innehåller en **Förinställning**. I **Förinställning** finns stegvisa anvisningar för den video- och/eller ljudbearbetning som ska användas för att generera önskad **TransformOutput**. I det här exemplet används förinställningen **VideoAnalyzerPreset** och språket (”en-US”) har överförts till konstruktorn. Med denna förinställning kan du extrahera flera ljud- och videoinsikter från en video. Du kan använda förinställningen **AudioAnalyzerPreset** om du vill extrahera flera ljudinsikter från en video. 

När du skapar en **Transformering** bör du först kontrollera om det redan finns en. Det gör du med metoden **Get** som visas i koden nedan.  I Media Services v3 returnerar **Get**-metoderna i entiteter **null** om entiteten inte finns (skiftlägesokänslig kontroll av namnet).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#EnsureTransformExists)]

#### <a name="job"></a>Jobb

Som nämns ovan är objektet [Transformering](https://docs.microsoft.com/rest/api/media/transforms) receptet och ett [Jobb](https://docs.microsoft.com/en-us/rest/api/media/jobs) är det faktiska begärandet till Media Services om att tillämpa **transformeringen** på en indatavideo eller ett ljudinnehåll. **Jobb** anger information som platsen för indatavideon och platsen för utdatan. Du kan ange platsen för din video med: HTTPS-webbadresser, SAS-URL:er eller tillgångar som finns på ditt Media Services-konto. 

I det här exemplet är jobbets indata en lokal video.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#SubmitJob)]

### <a name="wait-for-the-job-to-complete"></a>Vänta tills jobbet är klart

Jobbet tar en stund att slutföra och du meddelas när detta sker. Det finns olika alternativ för att få meddelanden om när [jobben](https://docs.microsoft.com/en-us/rest/api/media/jobs) är klara. Det enklaste alternativet (som visas här) är att använda avsökning. 

Vi rekommenderar inte att utföra avsökning av produktionsprogram på grund av att svarstiden kan bli lång. Avsökningen kan begränsas om den överanvänds på ett konto. Utvecklare bör i stället använda Event Grid.

Event Grid är utformat för hög tillgänglighet, konsekvent prestanda och dynamisk skalning. Med Event Grid kan dina appar lyssna efter och reagera på händelser från i princip alla Azure-tjänster, samt även från anpassade källor. Med enkel och HTTP-baserad reaktiv händelsehantering blir det lättare att skapa effektiva lösningar med hjälp av intelligent filtrering och dirigering av händelser. Se [Dirigera händelser till en anpassad webbslutpunkt](job-state-events-cli-how-to.md).

**Jobb** har vanligtvis följande tillstånd: **Schemalagd**, **I kö**, **Bearbetas**, **Slutförd** (slutlig status). Om jobbet har påträffat ett fel visas tillståndet **Fel**. Om jobbet avbryts visas **Avbryter** och **Avbruten** när det är klart.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#WaitForJobToFinish)]

### <a name="download-the-result-of-the-job"></a>Ladda ned jobbets resultat

Följande funktion laddar ned resultaten från [utdatatillgången](https://docs.microsoft.com/rest/api/media/assets) till mappen ”utdata”, så att du kan undersöka jobbresultatet. 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#DownloadResults)]

### <a name="clean-up-resource-in-your-media-services-account"></a>Rensa resurser på ditt Media Services-konto

Vanligtvis bör du rensa bort allt utom objekt som du tänker återanvända (om du t.ex. återanvänder transformeringar behåller du StreamingLocators osv.). Om du vill att ditt konto ska vara rensat efter experimentet, bör du ta bort de resurser som du inte tänker återanvända. Följande kod tar exempelvis bort Jobb.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/AnalyzeVideos/Program.cs#CleanUp)]

## <a name="run-the-sample-app"></a>Kör exempelappen

Tryck på Ctrl+F5 för att köra programmet *AnalyzeVideos*.

När vi kör programmet skapar jobbet miniatyrer för varje ansikte som hittas i videon. Det skapar också filen insights.json.

## <a name="examine-the-output"></a>Granska utdatan

Utdatafilen för videoanalysen kallas insights.json. Den här filen innehåller insikter om videon. Beskrivning av elementen i json-filen finns i artikeln [Medieintelligens](intelligence-concept.md).

## <a name="clean-up-resources"></a>Rensa resurser

Om du inte längre behöver någon av resurserna i resursgruppen, inklusive Media Services och de lagringskonton som du skapade för självstudien, tar du bort resursgruppen som du skapade tidigare. Du kan använda verktyget **CloudShell**.

Kör följande kommando i **CloudShell**:

```azurecli-interactive
az group delete --name amsResourceGroup
```

## <a name="multithreading"></a>Flertrådsteknik

SDK:erna i Azure Media Services v3 är inte trådsäkra. När du arbetar med flertrådade program bör du skapa ett nytt AzureMediaServicesClient-objekt per tråd.

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Självstudie: Ladda upp, koda och strömma filer](stream-files-tutorial-with-api.md)
