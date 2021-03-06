---
title: Översikt över acceleratorer för Azure IoT-lösning | Microsoft Docs
description: En beskrivning av acceleratorer för Azure IoT-lösningen och deras arkitektur med länkar till ytterligare resurser.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 01/17/2018
ms.author: dobett
ms.openlocfilehash: b961217317cea65b05cd58d66e64f0e4f012131e
ms.sourcegitcommit: 6cf20e87414dedd0d4f0ae644696151e728633b6
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/06/2018
ms.locfileid: "34808561"
---
# <a name="what-are-the-iot-solution-accelerators"></a>Vad är Azure IoT-lösningsacceleratorer?

_Lösningsacceleratorer_ för Azure IoT är en uppsättning lösningar som:

* Distribuerar på några minuter
* Hjälper dig att komma igång snabbt
* Du kan anpassa för att uppfylla dina specifika krav

Alla lösningsacceleratorer är utformade enligt samma principer och mål.

Följande videoklipp visar en översikt över fjärråtkomst övervakning solution accelerator:

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Meet-the-new-Remote-Monitoring-accelerator-for-Azure-IoT/Player]

## <a name="solution-accelerators-overview"></a>Översikt över lösningsacceleratorer

En lösningsaccelerator är en implementering med öppen källkod av ett vanligt IoT-lösningsmönster som du kan distribuera till Azure via din prenumeration. Varje lösningsaccelerator kombinerar anpassad kod och Azure-tjänster för att implementera ett specifikt IoT-scenario eller scenarier. Du kan anpassa alla dessa scenarion för att uppfylla dina specifika krav. Några vanliga scenarier:

* Visualisera data på en omfattande instrumentpanel för djupa insikter och lösningsstatus.
* Konfigurera regler och larm för live-telemetri från IoT-enheter.
* Schemalägga enhetshanteringsjobb, t.ex uppdateringar av programvara och konfigurationer.
* Etablera egna anpassade fysiska eller simulerade enheter.
* Felsök och åtgärda problem i dina IoT-enhetsgrupper.

Varje lösningsaccelerator är en fullständig implementering från slutpunkt till slutpunkt som kan använda simulerade eller fysiska enheter för att generera telemetri. Du kan använda lösningsacceleratorer för att lösa problem snabbare för att:

* Tillhandahålla en startpunkt för dina egna IoT-lösningar.
* Lära dig mer om vanliga mönster i utformningen och utvecklingen av IoT-lösningar.

Tre lösningsacceleratorer är tillgängliga idag:

* [Fjärrövervakning](iot-accelerators-remote-monitoring-explore.md)
* [Förebyggande underhåll](iot-accelerators-predictive-overview.md)
* [Ansluten fabrik](iot-accelerators-connected-factory-overview.md)

Följande tabell visar hur lösningarna mappar till specifika IoT-funktioner:

| Lösning | Datainhämtning | Enhetsidentitet | Enhetshantering | Edge-bearbetning | Kommando och kontroll | Regler och åtgärder | Förutsägelseanalys |
| ------------------------------------------------------------ | -- | -- | -- | -- | -- | -- | -- |
| [Fjärrövervakning](iot-accelerators-remote-monitoring-explore.md)  |Ja |Ja |Ja |-   |Ja |Ja |-   |
| [Förebyggande underhåll](iot-accelerators-predictive-overview.md)   |Ja |Ja |-   |-   |Ja |Ja |Ja |
| [Ansluten fabrik](iot-accelerators-connected-factory-overview.md) |Ja |- |- |Ja |Ja |Ja |-   |

* *Datainhämtning*: Inhämtning av data till molnet i hög skala.
* *Enhetsidentitet*: Hantera unika enhetsidentiteter och kontrollera enhetsåtkomst till lösningen.
* *Enhetshantering*: Hantera enhetsmetadata och utför åtgärder som omstarter av enheten och uppgraderingar av den inbyggda programvaran.
* *Kommando och kontroll*: Skicka meddelanden till en enhet från molnet för att utlösa en åtgärd på enheten.
* *Regler och åtgärder*: Lösningens serverdel använder regler för att agera på specifika data från enheten till molnet.
* *Förutsägelseanalys*: Lösningens backend-server analyserar data från enheten till molnet för att förutsäga när specifika åtgärder ska äga rum. Lösningen kan till exempel analysera telemetri från en flygplansmotor för att fastställa när motorunderhåll krävs.

> [!NOTE]
> Besök [Acceleratorer för Microsoft Azure IoT-lösning](https://www.azureiotsolutions.com/) om du vill distribuera en lösningsaccelerator och lära dig mer om hur du anpassar den.

## <a name="azure-services"></a>Azure-tjänster

När du distribuerar en lösningsaccelerator konfigurerar etableringsprocessen ett antal Azure-tjänster. I följande tabell visas de tjänster som används i lösningsacceleratorer:

|                      | Fjärrövervakning  | Förutsägande underhåll | Ansluten fabrik |
| -------------------- | ------------------ | ---------------------- | ----------------- |
| IoT Hub              | Ja                | Ja                    | Ja               |
| Event Hubs           |                    | Ja                    |                   |
| Time Series Insights |                    |                        | Ja               |
| Container Services   | Ja                |                        |                   |
| Stream Analytics     | Ja                | Ja                    |                   |
| Web Apps             | Ja                | Ja                    | Ja               |
| Cosmos DB            | Ja                | Ja                    |                    |
| Azure Storage         |                    | Ja                    | Ja               |

> [!NOTE]
> Mer information om resurser som har distribuerats i Fjärrövervaknings solution accelerator finns [Fjärrövervaknings arkitektur val](iot-accelerators-remote-monitoring-architectural-choices.md).

* [Azure IoT Hub](../iot-hub/index.yml). Den här tjänsten tillhandahåller funktioner för meddelandehantering från ”enhet till moln” och från ”moln till enhet” och fungerar som en gateway till molnet och andra viktiga lösningsacceleratortjänster. Tjänsten gör att du kan ta emot meddelanden från dina enheter i hög skala och skicka kommandon till dina enheter. Med tjänsten kan du även [hantera dina enheter](../iot-hub/iot-hub-device-management-overview.md). Du kan till exempel konfigurera, starta om eller utföra en fabriksåterställning på en eller flera enheter som är anslutna till hubben.
* [Azure Event Hubs](../active-directory-domain-services/index.yml). Tjänst för inmatning av stora händelsevolymer. Se [Comparison of Azure IoT Hub and Azure Event Hubs](../iot-hub/iot-hub-compare-event-hubs.md) (Jämförelse mellan Azure IoT Hub och Azure Event Hubs).
* [Azure Time Series Insights](../time-series-insights/index.yml). Lösningsacceleratorerna använder den här tjänsten för att analysera och visa telemetridata från dina enheter.
* [Azure Container Service](../container-service/index.yml). Den här tjänsten är värd för och hanterar mikrotjänster i lösningsacceleratorer.
* [Azure Cosmos DB](../cosmos-db/index.yml) och [Azure Storage](../storage/index.yml) för datalagring.
* [Azure Stream Analytics](../stream-analytics/index.yml). Förutsägande Underhåll solution accelerator använder den här tjänsten för att bearbeta inkommande telemetri, utföra aggregering och identifiera händelser. Den här förkonfigurerade lösningen använder också dataströmsanalys för att bearbeta informationsmeddelanden som innehåller data, till exempel metadata eller kommandosvar från enheter.
* [Azure Web Apps](../app-service/index.yml) som värd för den anpassade programkoden i de förkonfigurerade lösningarna.

En översikt över arkitekturen i en typisk IoT-lösning finns i [Microsoft Azure och Sakernas Internet (IoT)](iot-accelerators-what-is-azure-iot.md).

## <a name="whats-new-in-solution-accelerators"></a>Nyheter för lösningsacceleratorer

Microsoft uppdaterar lösningsacceleratorerna till en ny arkitektur baserad på mikrotjänster. I följande tabell visas den nuvarande statusen för lösningsacceleratorerna:

| Lösningsaccelerator | Arkitektur  | Språk     |
| ---------------------- | ------------- | ------------- |
| Fjärrövervakning      | Mikrotjänster | Java och .NET |
| Förutsägande underhåll | MVC           | .NET          |
| Ansluten fabrik      | MVC           | .NET          |

Följande avsnitt beskriver vad som är nytt i de lösningsacceleratorerna som är baserade på mikrotjänster:

### <a name="microservices"></a>Mikrotjänster

Den nya versionen av Fjärrövervaknings solution accelerator använder en arkitektur för mikrotjänster. Den här lösningsacceleratorn består av flera mikrotjänster som en *IoT Hub-hanterare* och en *Lagringshanterare*. Både Java- och .NET-versioner av varje mikrotjänst är tillgänglig för nedladdning tillsammans med relaterad utvecklardokumentation. Läs mer om mikrotjänsterna i [Remote monitoring architecture](iot-accelerators-remote-monitoring-sample-walkthrough.md) (Arkitektur för fjärrövervakning).

Den här arkitekturen för mikrotjänster är ett beprövat mönster för molnlösningar som:

* Är skalbar.
* Aktiverar utökningsbarhet.
* Är lätt att förstå.
* Gör det möjligt att byta ut enskilda tjänster mot alternativa tjänster.

> [!TIP]
> Läs mer om arkitekturer för mikrotjänster i [.NET Application Architecture](https://www.microsoft.com/net/learn/architecture) (.NET-programarkitektur) och [Microservices: An application revolution powered by the cloud](https://azure.microsoft.com/blog/microservices-an-application-revolution-powered-by-the-cloud/) (Mikrotjänster: En programrevolution som drivs av molnet).

När du distribuerar den nya versionen av Fjärrövervaknings måste du välja något av följande distributionsalternativ för:

* **Basic:** Version till reducerad kostnad för en demonstration eller för att testa en distribution. Alla mikrotjänster distribueras till en enda virtuell Azure-dator.
* **Standard:** Expanderad distribution av infrastruktur för att utveckla en produktionsdistribution. Azure Container Service distribuerar mikrotjänsterna till flera virtuella Azure-datorer. Kubernetes orkestrerar Docker-behållare som värdar för enskilda mikrotjänster.

### <a name="language-choices-java-and-net"></a>Språkval: Java och .NET

Implementeringar av var och en av mikrotjänsterna är tillgängliga i både Java- och .NET. Precis som .NET-koden är Java-källkoden öppen källkod och finns tillgänglig för dig så du kan anpassa den efter dina specifika krav:

* [.NET GitHub-lagringsplatsen för fjärrövervakning](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet)
* [Java GitHub-lagringsplatsen för fjärrövervakning](https://github.com/Azure/azure-iot-pcs-remote-monitoring-java)

Om du vill se andra språkimplementeringar lägger du till en begäran om det i [Azure IoT UserVoice](https://feedback.azure.com/forums/321918-azure-iot).

### <a name="react-user-interface-framework"></a>Ramverk för användargränssnittet React

Användargränssnittet skapas med javascript-biblioteket [React](https://facebook.github.io/react/). Källkoden är öppen källkod och finns tillgänglig för dig att ladda ned och anpassa.

## <a name="next-steps"></a>Nästa steg

Nu när du har en översikt över lösningsacceleratorerna föreslår vi följande nästa steg för varje lösningsaccelerator:

* [Utforska lösningen för fjärrövervakning](iot-accelerators-remote-monitoring-explore.md).
* [Översikt över lösningsaccelerator för förutsägande underhåll](iot-accelerators-predictive-overview.md)
* [Kom igång med lösningsacceleratorn Ansluten fabrik](iot-accelerators-connected-factory-overview.md).

Mer information om IoT-lösningsarkitekturer finns i [Microsoft Azure IoT-tjänster: referensarkitektur](https://aka.ms/iotrefarchitecture).
