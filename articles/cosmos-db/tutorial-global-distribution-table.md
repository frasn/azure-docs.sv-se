---
title: Självstudie för global distribution i Azure Cosmos DB för tabell-API | Microsoft Docs
description: Läs om hur du konfigurerar global distribution i Azure Cosmos DB med tabell-API.
services: cosmos-db
keywords: global distribution, Table
documentationcenter: ''
author: SnehaGunda
manager: kfile
ms.assetid: 8b815047-2868-4b10-af1d-40a1af419a70
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 12/13/2017
ms.author: sngun
ms.custom: mvc
ms.openlocfilehash: f877baa33d94dad07250da9a10209555dbca65c9
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/16/2018
---
# <a name="how-to-setup-azure-cosmos-db-global-distribution-using-the-table-api"></a>Så här konfigurerar du global distribution i Azure Cosmos DB med tabell-API

Den här artikeln beskriver följande uppgifter: 

> [!div class="checklist"]
> * Konfigurera global distribution med Azure Portal
> * Konfigurera global distribution med [tabell-API](table-introduction.md)

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]


## <a name="connecting-to-a-preferred-region-using-the-table-api"></a>Ansluta till en önskad region med hjälp av tabell-API

I syfte att dra nytta av den [globala distributionen](distribute-data-globally.md) kan klientprogrammen ange den beställda listan med inställningar för regioner som ska användas för att utföra dokumentåtgärder. Detta kan göras genom att ange egenskapen [TableConnectionPolicy.PreferredLocations](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.table.tableconnectionpolicy.preferredlocations?view=azure-dotnet#Microsoft_Azure_CosmosDB_Table_TableConnectionPolicy_PreferredLocations). Tabell-API:ns SDK i Azure Cosmos DB väljer den bästa slutpunkten för kommunikationen baserat på kontokonfiguration, aktuell regional tillgänglighet och listan med inställningar.

PreferredLocations ska innehålla en kommaavgränsad lista med önskade (flera värdar) platser för läsningar. Varje klientinstans kan ange en delmängd av dessa regioner i önskad ordning för läsningar med låg latens. Regionerna måste namnges med sina [visningsnamn](https://msdn.microsoft.com/library/azure/gg441293.aspx), till exempel `West US`.

Alla läsningar skickas till den första tillgängliga regionen i PreferredLocations-listan. Om begäran misslyckas går klienten nedåt i listan till nästa region osv.

SDK:erna försöker läsa från de regioner som anges i PreferredLocations. Det innebär att om databaskontot t.ex. är tillgängligt i tre regioner men klienten enbart anger två av de skrivskyddade regionerna för PreferredLocations, så hanteras inga läsningar från skrivregionen även om en redundans sker.

SDK:n skickar automatiskt alla skrivningar till den aktuella skrivregionen.

Om egenskapen PreferredLocations inte har angetts hanteras alla förfrågningar från den aktuella skrivregionen.

Och med detta är den här självstudiekursen klar. Mer information om hur du kan hantera ditt globalt replikerade kontos konsekvens finns i [Konsekvensnivåer i Azure Cosmos DB](consistency-levels.md). Mer information om hur global databasreplikering fungerar i Azure Cosmos DB finns i [Distribuera data globalt med Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du gjort följande:

> [!div class="checklist"]
> * Konfigurera global distribution med Azure Portal
> * Konfigurera global distribution med tabell-API:n i Azure Cosmos DB

