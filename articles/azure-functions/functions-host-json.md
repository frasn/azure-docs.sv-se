---
title: Host.JSON referens för Azure Functions
description: I referensdokumentationen för Azure Functions host.json filen.
services: functions
author: tdykstra
manager: cfowler
editor: ''
tags: ''
keywords: ''
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 02/12/2018
ms.author: tdykstra
ms.openlocfilehash: d1dec6f2da4f6fcbeb38585fc6a1cfcd9d622c4a
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2018
---
# <a name="hostjson-reference-for-azure-functions"></a>Host.JSON referens för Azure Functions

Den *host.json* metadatafil innehåller globala konfigurationsalternativ som påverkar alla funktioner för en funktionsapp. Den här artikeln innehåller de inställningar som är tillgängliga. JSON-schema är på http://json.schemastore.org/host.

Det finns andra globala konfigurationsalternativ i [appinställningar](functions-app-settings.md) och i den [local.settings.json](functions-run-local.md#local-settings-file) fil.

## <a name="sample-hostjson-file"></a>Exempelfilen host.json

I följande exempel *host.json* filen har alla angivna möjliga alternativ.

```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    },
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    },
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    },
    "functions": [ "QueueProcessor", "GitHubWebHook" ],
    "functionTimeout": "00:05:00",
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    },
    "http": {
        "routePrefix": "api",
        "maxOutstandingRequests": 20,
        "maxConcurrentRequests": 10,
        "dynamicThrottlesEnabled": false
    },
    "id": "9f4ea53c5136457d883d685e57164f08",
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    },
    "queues": {
      "maxPollingInterval": 2000,
      "visibilityTimeout" : "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    },
    "serviceBus": {
      "maxConcurrentCalls": 16,
      "prefetchCount": 100,
      "autoRenewTimeout": "00:05:00"
    },
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    },
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    },
    "watchDirectories": [ "Shared" ],
}
```

I följande avsnitt i den här artikeln beskrivs varje översta egenskap. Alla är valfria om inget annat anges.

## <a name="aggregator"></a>Aggregator

Anger hur många funktionsanrop sammanställs när [beräkna mått för Application Insights](functions-monitoring.md#configure-the-aggregator). 

```json
{
    "aggregator": {
        "batchSize": 1000,
        "flushTimeout": "00:00:30"
    }
}
```

|Egenskap |Standard  | Beskrivning |
|---------|---------|---------| 
|batchSize|1000|Högsta antal begäranden ska aggregeras.| 
|flushTimeout|00:00:30|Maximal tid period ska aggregeras.| 

Funktionsanrop aggregeras när först av två begränsar har uppnåtts.

## <a name="applicationinsights"></a>applicationInsights

Kontroller av [provtagning funktion i Application Insights](functions-monitoring.md#configure-sampling).

```json
{
    "applicationInsights": {
        "sampling": {
          "isEnabled": true,
          "maxTelemetryItemsPerSecond" : 5
        }
    }
}
```

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------| 
|IsEnabled|true|Aktiverar eller inaktiverar provtagning.| 
|maxTelemetryItemsPerSecond|5|Tröskelvärdet på vilka provtagning börjar.| 

## <a name="durabletask"></a>durableTask

Konfigurationsinställningar för [varaktiga funktioner](durable-functions-overview.md).

```json
{
  "durableTask": {
    "HubName": "MyTaskHub",
    "ControlQueueBatchSize": 32,
    "PartitionCount": 4,
    "ControlQueueVisibilityTimeout": "00:05:00",
    "WorkItemQueueVisibilityTimeout": "00:05:00",
    "MaxConcurrentActivityFunctions": 10,
    "MaxConcurrentOrchestratorFunctions": 10,
    "AzureStorageConnectionStringName": "AzureWebJobsStorage",
    "TraceInputsAndOutputs": false,
    "EventGridTopicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
    "EventGridKeySettingName":  "EventGridKey"
  }
}
```

Hubb aktivitetsnamn måste börja med en bokstav och endast bestå av bokstäver och siffror. Om inget anges är standardnamnet för aktiviteten hubb för en funktionsapp **DurableFunctionsHub**. Mer information finns i [uppgift hubbar](durable-functions-task-hubs.md).

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------|
|hubName|DurableFunctionsHub|Alternativa [aktivitet hubb](durable-functions-task-hubs.md) namn kan användas för att isolera flera beständiga funktioner program från varandra, även om de använder samma lagringserverdel.|
|ControlQueueBatchSize|32|Antal meddelanden till pull från kontrollen kön i taget.|
|PartitionCount |4|Antalet partitioner för kö för kontrollen. Kan vara ett positivt heltal mellan 1 och 16.|
|ControlQueueVisibilityTimeout |5 minuter|Tidsgränsen för visning av meddelanden i kö togs bort från kön kontroll.|
|WorkItemQueueVisibilityTimeout |5 minuter|Tidsgränsen för visning av meddelanden i kö för objektet togs bort från kön arbete.|
|MaxConcurrentActivityFunctions |10 x antalet processorer på den aktuella datorn|Maximalt antal aktiviteten funktioner som kan bearbetas samtidigt på en enda värd-instans.|
|MaxConcurrentOrchestratorFunctions |10 x antalet processorer på den aktuella datorn|Maximalt antal aktiviteten funktioner som kan bearbetas samtidigt på en enda värd-instans.|
|AzureStorageConnectionStringName |AzureWebJobsStorage|Namnet på appinställningen med Azure Storage-anslutningssträngen som används för att hantera de underliggande resurserna i Azure Storage.|
|TraceInputsAndOutputs |false|Ett värde som anger om du vill spåra indata och utdata för funktionsanrop. Standardbeteendet när spårning funktionen körning händelser är att inkludera antalet byte i serialiserade indata och utdata för funktionsanrop. Detta ger minimalt information om hur indata och utdata ut utan svullen buk loggarna eller exponera känslig information till loggar oavsiktligt. Den här egenskapen till true kommer funktionen Standardloggning logga hela innehållet i funktionen indata och utdata.|
|EventGridTopicEndpoint ||URL till en Azure händelse rutnätet anpassade avsnittet slutpunkt. När den här egenskapen anges publiceras orchestration livscykel meddelandehändelser till den här slutpunkten.|
|EventGridKeySettingName ||Namnet på appinställningen som innehåller den nyckel som används för att autentisera med Azure händelse rutnätet anpassade artikeln i `EventGridTopicEndpoint`.

Många av dessa är för att optimera prestanda. Mer information finns i [prestanda och skalning](durable-functions-perf-and-scale.md).

## <a name="eventhub"></a>eventHub

Konfigurationsinställningar för [Event Hub-utlösare och bindningar](functions-bindings-event-hubs.md).

[!INCLUDE [functions-host-json-event-hubs](../../includes/functions-host-json-event-hubs.md)]

## <a name="functions"></a>functions

En lista över funktioner som värd för jobbet ska köras. En tom matris innebär att köra alla funktioner. Avsedd att användas endast när [körs lokalt](functions-run-local.md). I funktionen appar använda den *function.json* `disabled` egenskapen i stället för den här egenskapen i *host.json*.

```json
{
    "functions": [ "QueueProcessor", "GitHubWebHook" ]
}
```

## <a name="functiontimeout"></a>functionTimeout

Anger timeout-varaktighet för alla funktioner. Det giltiga intervallet är mellan 1 sekund till 10 minuter i förbrukning planer, och standardvärdet är 5 minuter. Det finns ingen gräns i App Service-planer, och standardvärdet är null, vilket indikerar ingen tidsgräns.

```json
{
    "functionTimeout": "00:05:00"
}
```

## <a name="healthmonitor"></a>healthMonitor

Konfigurationsinställningar för [värden hälsoövervakning](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Host-Health-Monitor).

```
{
    "healthMonitor": {
        "enabled": true,
        "healthCheckInterval": "00:00:10",
        "healthCheckWindow": "00:02:00",
        "healthCheckThreshold": 6,
        "counterThreshold": 0.80
    }
}
```

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------| 
|aktiverad|true|Om funktionen är aktiverad. | 
|healthCheckInterval|10 sekunder|Tidsintervallet mellan regelbunden hälsa kontrollerar. | 
|healthCheckWindow|2 minuter|Ett skjutfönster tid används tillsammans med den `healthCheckThreshold` inställningen.| 
|healthCheckThreshold|6|Maximalt antal gånger hälsotillståndskontroll kan misslyckas innan värden återvinning initieras.| 
|counterThreshold|0,80|Tröskelvärdet som en prestandaräknare betraktas som ohälsosamt.| 

## <a name="http"></a>http

Konfigurationsinställningar för [http-utlösare och bindningar](functions-bindings-http-webhook.md).

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

## <a name="id"></a>id

Unikt ID för en värd för jobbet. Vara kan en gemen GUID med bindestreck bort. Krävs när du kör programmet lokalt. När den körs i Azure Functions ID skapas automatiskt om `id` utelämnas.

Om du delar ett lagringskonto över flera funktionen appar, se till att varje funktionsapp har en annan `id`. Du kan hoppa över den `id` egenskapen eller manuellt ange varje funktionsapp `id` till ett annat värde. Timer-utlösaren använder lagring Lås så att endast en timer-instans när en funktionsapp skalas ut till flera instanser. Om två funktionen appar med samma `id` och varje använder en timer som utlösare, körs bara en timer.


```json
{
    "id": "9f4ea53c5136457d883d685e57164f08"
}
```

## <a name="logger"></a>Loggaren

Kontroller filtrering för loggar som skrivits av en [ILogger objekt](functions-monitoring.md#write-logs-in-c-functions) eller [context.log](functions-monitoring.md#write-logs-in-javascript-functions).

```json
{
    "logger": {
        "categoryFilter": {
            "defaultLevel": "Information",
            "categoryLevels": {
                "Host": "Error",
                "Function": "Error",
                "Host.Aggregator": "Information"
            }
        }
    }
}
```

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------| 
|categoryFilter|Saknas|Anger att filtrera efter kategori| 
|defaultLevel|Information|Alla kategorier som angetts i den `categoryLevels` kan skicka loggar på denna nivå och senare till Application Insights.| 
|categoryLevels|Saknas|En matris med kategorier som anger den minsta loggnivån som skickas till Application Insights för varje kategori. Den kategori som anges här styr alla kategorier som börjar med samma värde, och längre värden företräde. I föregående exempel *host.json* -fil, alla kategorier som börjar med ”Host.Aggregator”-loggen på `Information` nivå. Alla kategorier som börjar med ”värd”, till exempel ”Host.Executor” logga in på `Error` nivå.| 

## <a name="queues"></a>Köer

Konfigurationsinställningar för [lagring-utlösare och bindningar](functions-bindings-storage-queue.md).

[!INCLUDE [functions-host-json-queues](../../includes/functions-host-json-queues.md)]

## <a name="servicebus"></a>ServiceBus

Konfigurationsinställning för [Service Bus-utlösare och bindningar](functions-bindings-service-bus.md).

[!INCLUDE [functions-host-json-service-bus](../../includes/functions-host-json-service-bus.md)]

## <a name="singleton"></a>Singleton

Konfigurationsinställningar för Singleton låsa beteende. Mer information finns i [GitHub problemet om singleton-stöd](https://github.com/Azure/azure-webjobs-sdk-script/issues/912).

```json
{
    "singleton": {
      "lockPeriod": "00:00:15",
      "listenerLockPeriod": "00:01:00",
      "listenerLockRecoveryPollingInterval": "00:01:00",
      "lockAcquisitionTimeout": "00:01:00",
      "lockAcquisitionPollingInterval": "00:00:03"
    }
}
```

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------| 
|lockPeriod|00:00:15|Den period som funktionen nivån Lås används för. Lås förnyelse automatisk.| 
|listenerLockPeriod|00:01:00|Den period som lyssnare Lås används för.| 
|listenerLockRecoveryPollingInterval|00:01:00|Tidsintervall som används för lyssnare låsåterställning om inte det gick att hämta en lyssnare Lås vid start.| 
|lockAcquisitionTimeout|00:01:00|Längsta tid körningsmiljön kommer att försöka skaffa ett lås.| 
|lockAcquisitionPollingInterval|Saknas|Intervall mellan försök för anskaffning av Lås.| 

## <a name="tracing"></a>spårning

Konfigurationsinställningar för loggarna som du skapar med hjälp av en `TraceWriter` objekt. Se [C# loggning](functions-reference-csharp.md#logging) och [Node.js loggning](functions-reference-node.md#writing-trace-output-to-the-console). 

```json
{
    "tracing": {
      "consoleLevel": "verbose",
      "fileLoggingMode": "debugOnly"
    }
}
```

|Egenskap  |Standard | Beskrivning |
|---------|---------|---------| 
|consoleLevel|Info|Spårning-nivå för konsolen loggning. Alternativen är: `off`, `error`, `warning`, `info`, och `verbose`.|
|fileLoggingMode|debugOnly|Spårning-nivå för filen loggning. Alternativen är `never`, `always`, `debugOnly`.| 

## <a name="watchdirectories"></a>watchDirectories

En uppsättning [delade kod kataloger](functions-reference-csharp.md#watched-directories) som bör övervakas för ändringar.  Garanterar att när koden i de här katalogerna ändras ändringarna fångas upp av dina funktioner.

```json
{
    "watchDirectories": [ "Shared" ]
}
```

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Lär dig hur du uppdaterar filen host.json](functions-reference.md#fileupdate)

> [!div class="nextstepaction"]
> [Globala inställningar i miljövariabler](functions-app-settings.md)
