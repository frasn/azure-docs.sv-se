---
title: Azure Search-tjänsten REST API-versionen 2017-11-11-Preview | Microsoft Docs
description: Azure Search Service REST API-versionen 2017-11-11-Preview innehåller experiment funktioner, till exempel synonymer och moreLikeThis sökningar.
services: search
author: HeidiSteen
manager: cgronlun
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: search
ms.date: 05/17/2018
ms.author: HeidiSteen
ms.openlocfilehash: ce5771777762414a0229cf83425c2f3601cb979a
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34655730"
---
# <a name="azure-search-service-rest-api-version-2017-11-11-preview"></a>Azure Search-tjänsten REST api-version 2017-11-11-Preview
Den här artikeln beskriver den `api-version=2017-11-11-Preview` versionen av Azure Search-tjänst, med experiment funktioner som inte ännu allmänt tillgänglig.

> [!NOTE]
> Förhandsgranskningsfunktioner finns tillgängliga för testning och experiment med målet att samla in feedback och kan komma att ändras. Vi starkt rekommenderar mot att använda Förhandsgranska API: er i program i produktion.


## <a name="new-in-2017-11-11-preview"></a>I 2017-11-11-Preview

[Kognitiva Sök](cognitive-search-concept-intro.md), en ny berikande funktion i Azure Search hittar Latenta information i icke-text källor och omvandla det till fulltext sökbara innehåll i Azure Search odifferentierad text.

Följande reources introduceras eller ändras i förhandsgranskningen REST API. Alla andra REST API: er är samma oavsett om du anropar den allmänt tillgängliga eller förhandsgranskning.

+ [Skapa kunskaper (api-version = 2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)

+ [Skapa indexerare (api-version = 2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)

+ [Fördefinierade kunskaper](cognitive-search-predefined-skills.md)

Alla andra REST API: er är samma oavsett hur du ställer in api-version. Till exempel `GET https://[service name].search.windows.net/indexes/hotels?api-version=2017-11-11-Preview` och `GET https://[service name].search.windows.net/indexes/hotels?api-version=2017-11-11` (utan `Preview`) är funktionellt likvärdiga.

## <a name="other-preview-features"></a>Andra funktioner för förhandsgranskning

Funktioner som presenterades i tidigare förhandsversioner är fortfarande i förhandsversion. Om du anropar en API med en tidigare förhandsversion api-version, kan du fortsätta att använda den här versionen eller växla till `2017-11-11-Preview` utan ändringar av förväntat beteende.

+ [CSV-filer i Azure Blob-indexering](search-howto-index-csv-blobs.md), introducerades i `api-version=2015-02-28-Preview`, förblir en förhandsversion av funktionen. Den här funktionen är en del av Azure Blob-indexering och anropas via en inställning för parametern. Varje rad i en CSV-fil indexeras som ett separat dokument.

+ [JSON-matriser i Azure Blob-indexering](search-howto-index-json-blobs.md), introducerades i `api-version=2015-02-28-Preview`, förblir en förhandsversion av funktionen. Den här funktionen är en del av Azure Blob-indexering och anropas via en inställning för parametern. där varje element i matrisen indexeras som ett separat dokument.

+ [moreLikeThis Frågeparametern](search-more-like-this.md) hittas dokument som är relevanta för ett visst dokument. Den här funktionen har varit i tidigare förhandsversioner. 


## <a name="how-to-call-a-preview-api"></a>Hur du anropar en API-förhandsgranskning

Äldre förhandsversioner fortfarande fungerar utan blir inaktuella över tid. Om koden anropar `api-version=2016-09-01-Preview` eller `api-version=2015-02-28-Preview`, anropen fortfarande är giltiga. Dock uppdateras bara den senaste förhandsversionen med förbättringar. 

Följande exempel illustrerar ett anrop till förhandsversionen API.

    GET https://[service name].search.windows.net/indexes/[index name]/docs?search=*&api-version=2017-11-11-Preview

Azure Search-tjänsten är tillgänglig i flera versioner. Mer information finns i [API-versioner](search-api-versions.md).
