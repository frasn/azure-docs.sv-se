---
title: Händelsehubbar i Azure för Kafka ekosystem | Microsoft Docs
description: Översikt och introduktion till Kafka aktiverat Azure Event Hubs
services: event-hubs
documentationcenter: .net
author: djrosanova
manager: timlt
ms.service: event-hubs
ms.topic: article
ms.date: 05/07/2018
ms.author: darosa
ms.openlocfilehash: 51e32737be4eaf3c61cdbe50b82a05c205800ac4
ms.sourcegitcommit: fc64acba9d9b9784e3662327414e5fe7bd3e972e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2018
---
# <a name="event-hubs-for-kafka-ecosystems"></a>Händelsehubbar för Kafka ekosystem

Händelsehubbar för Kafka ekosystem ger en Kafka slutpunkt som kan användas av din befintliga Kafka baserat program som ett alternativ till att köra Kafka klustret. Händelsehubbar för Kafka ekosystem stöder [Apache Kafka 1.0](https://kafka.apache.org/10/documentation.html) och nyare versioner av klienten och fungerar med din befintliga Kafka program, inklusive MirrorMaker. Ändra anslutningssträngen och starta strömning händelser från ditt program som använder protokollet Kafka i Händelsehubbar.

## <a name="what-does-event-hubs-for-kafka-ecosystems-provide"></a>Vad ger Händelsehubbar för Kafka ekosystem?

Händelsehubbar för Kafka ekosystem ger en protokollet head ovanpå Azure Event Hubs är binär kompatibel med Kafka versioner 1.0 och senare för att både läsa från och skriva till Kafka avsnitt. Begreppsmässigt Kafka och Händelsehubbar är nästan identisk: de är båda partitionerade loggar som skapats för strömning av data. I följande tabell mappar begrepp mellan Kafka och Händelsehubbar.

### <a name="kafka-and-event-hub-conceptual-mapping"></a>Kafka och Event Hub konceptuella mappning

| Kafka begrepp | Event Hubs begrepp|
| --- | --- |
| Kluster | Namnrymd |
| Avsnitt | Event Hubs |
| Partition | Partition|
| Konsumentgrupp | Konsumentgrupp |
| Offset | Offset|

### <a name="key-differences-between-kafka-and-event-hubs"></a>Viktiga skillnader mellan Kafka och Händelsehubbar

Medan [Apache Kafka](https://kafka.apache.org/) är programvara som du kan köra där du väljer, Händelsehubbar är en molntjänst som liknar Azure Blob Storage. Det finns inga servrar eller nätverk för att hantera och inga mäklare för att konfigurera. Du skapar ett namnområde, vilket är ett fullständigt domännamn som din avsnitt bor, och skapa Händelsehubbar eller avsnitt inom det här namnområdet. Mer information om Händelsehubbar och namnområden finns [vad är Händelsehubbar](event-hubs-what-is-event-hubs.md). Som en molntjänst använder Händelsehubbar en stabil virtuella IP-adress som slutpunkten, så att klienter inte behöver veta om mäklare eller datorer i ett kluster. 

Skalan i Händelsehubbar styrs av hur många genomflödesenheter du köper, med varje genomflödesenhet som ger rätt till 1 MB per sekund eller 1000 händelser per sekund för inkommande trafik. Som standard Händelsehubbar skalas upp enheter när du når gränsen med den [automatiskt öka](event-hubs-auto-inflate.md) funktionen; den här funktionen också fungerar med Händelsehubbar för Kafka ekosystem. 

### <a name="security-and-authentication"></a>Säkerhet och autentisering

Händelsehubbar i Azure kräver SSL eller TLS för all kommunikation och använder delad åtkomst signaturer (SAS) för autentisering. Det här kravet gäller även för en Kafka slutpunkt i Händelsehubbar. För kompatibilitet med Kafka använder Händelsehubbar SASL OFORMATERAD för autentisering och SASL SSL för transportsäkerhet. Mer information om säkerhet i Händelsehubbar finns [Händelsehubbar autentiseringen och säkerheten](event-hubs-authentication-and-security-model-overview.md).

## <a name="other-event-hubs-features-available-for-kafka"></a>Andra Händelsehubbar funktioner som är tillgängliga för Kafka

Händelsehubbar för Kafka ekosystem kan du skriva med ett protokoll och läsa med en annan, så att din aktuella Kafka producenter kan fortsätta publicera via Kafka, och du kan lägga till läsare med Händelsehubbar, till exempel Azure Stream Analytics eller Azure Functions. Dessutom Händelsehubbar funktioner såsom [avbilda](event-hubs-capture-overview.md) och [Geo katastrofåterställning](event-hubs-geo-dr.md) också fungera med Händelsehubbar för Kafka ekosystem.

## <a name="features-that-are-not-supported-in-the-preview"></a>Funktioner som inte stöds i preview

Följande Kafka funktioner stöds inte för en förhandsversion av Händelsehubbar för Kafka ekosystem integrering:

*   Idempotent producenten
*   Transaktionen
*   Komprimering
*   Storlek-baserade kvarhållning
*   Logga komprimering
*   Lägger till partitioner i ett befintligt ämne
*   HTTP-Kafka API-stöd
*   Kafka ansluta
*   Kafka dataströmmar

## <a name="next-steps"></a>Nästa steg

Den här artikeln föreskrivs en introduktion till Händelsehubbar Kafka ekosystem. Mer information finns i följande länkar:

* [Så här skapar du Kafka aktiverad Händelsehubbar](event-hubs-create-kafka-enabled.md)
* [Dataströmmen i Händelsehubbar från Kafka-program](event-hubs-quickstart-kafka-enabled-event-hubs.md)
* Kom igång med en [kurs om händelsehubbar](event-hubs-dotnet-standard-getstarted-send.md)
* [Vanliga frågor och svar om Event Hubs](event-hubs-faq.md)

 
 

