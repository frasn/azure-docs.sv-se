---
title: Hur cachelagring fungerar | Microsoft Docs
description: Cachelagring är en process för att lagra data lokalt så att framtida begäranden om att data kan nås snabbare.
services: cdn
documentationcenter: ''
author: dksimpson
manager: akucer
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/30/2018
ms.author: v-deasim
ms.openlocfilehash: bb0824995972b49febdb1695e41f45fbd0966cd1
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2018
---
# <a name="how-caching-works"></a>Så här fungerar cachelagring

Den här artikeln innehåller en översikt över allmänna cachelagring begrepp och hur [Azure Content Delivery Network (CDN)](cdn-overview.md) använder cachelagring för att förbättra prestanda. Om du vill ha mer information om hur du anpassar cachelagringsbeteendet på CDN-slutpunkten finns [kontroll Azure CDN cachelagring av frågesträngar med cachelagring regler](cdn-caching-rules.md) och [kontroll Azure CDN cachelagring av frågesträngar med frågesträngar](cdn-query-string.md).

## <a name="introduction-to-caching"></a>Introduktion till cachelagring

Cachelagring är en process för att lagra data lokalt så att framtida begäranden om att data kan nås snabbare. I den vanligaste typen av cachelagring, web webbläsare cachelagring, en webbplats lagrar webbläsaren kopior av statiska data lokalt på en lokal hårddisk. Med hjälp av cachelagring kan webbläsaren undvika att göra flera turer till servern och i stället åtkomst till samma data lokalt, vilket sparar tid och resurser. Cachelagring lämpar sig väl för att lokalt hantera liten, statiska data, till exempel statiska bilder, CSS-filer och JavaScript-filer.

På liknande sätt används cachelagring av ett nätverk för innehållsleverans på edge servrar nära användaren för att undvika begäranden reser tillbaka till ursprunget och minska svarstiden för slutanvändaren. Till skillnad från en webbserver webbläsarens cache som används endast för en enskild användare, har CDN delade cachelagrade. I en delad CDN-cache, kan en fil som har begärts av en användare nås senare av andra användare, vilket avsevärt minskar antalet begäranden till den ursprungliga servern.

Dynamiska resurser som ändras ofta eller är unika för en enskild användare kan inte cachelagras. Dessa typer av resurser, kan dock dra nytta av dynamisk webbplats acceleration (DSA) optimering i Azure Content Delivery Network för bättre prestanda.

Cachelagring kan ske på flera nivåer mellan den ursprungliga servern och användaren:

- Webbserver: använder en delad cache (för flera användare).
- Nätverk för innehållsleverans: använder en delad cache (för flera användare).
- Internet-leverantör (ISP): använder en delad cache (för flera användare).
- Webbläsare: använder ett privat cache (för en användare).

Varje cacheminne hanterar en egen resurs dokumentens normalt och utför verifieringen när en fil är inaktuell. Det här beteendet är definierad i HTTP-specifikationen cachelagring [RFC 7234](https://tools.ietf.org/html/rfc7234).

### <a name="resource-freshness"></a>Resursen dokumentens

Eftersom en cachelagrad resurs kan vara inaktuella eller inaktuella (jämfört med motsvarande resurs på den ursprungliga servern), det är viktigt för alla cachelagringsmekanism för att styra när innehåll uppdateras. Om du vill spara tid och bandbredd förbrukning cachelagrade resursen är inte jämfört med versionen på den ursprungliga servern varje gång den används. I stället så länge en cachelagrad resurs anses vara ny, det antas vara den senaste versionen och skickas direkt till klienten. En cachelagrad resurs anses vara ny när dess ålder är mindre än det ålder eller den period som definieras av en cache-inställningen. Till exempel när en webbläsare laddar en webbsida, verifierar att alla cachelagrade resurser på din hårddisk är ny och läser in den. Om resursen inte är giltigt (inaktuell), en aktuell kopia läses från servern.

### <a name="validation"></a>Validering

Om en resurs betraktas som inaktuella, blir den ursprungliga servern ombedd att verifiera den, det vill säga, avgör om data i cacheminnet fortfarande matchar vad som finns på den ursprungliga servern. Om filen har ändrats på den ursprungliga servern, uppdateras cacheminnet sin version av resursen. Annars, om resursen är ny data levereras direkt från cachen utan att verifiera den först.

### <a name="cdn-caching"></a>CDN-cachelagring

Cachelagring är integrerad sätt en CDN fungerar för att påskynda överföringen och minska belastningen ursprung för statiska tillgångar, till exempel bilder, teckensnitt och videor. CDN cachelagrar, statiska resurser lagras selektivt på strategiskt monterade servrar som är mer lokala användare och ger följande fördelar:

- Eftersom de flesta Internet-trafik är statiska (till exempel bilder, teckensnitt och videor), minskar CDN cachelagring svarstiderna för nätverket genom att flytta innehåll närmare för användaren, vilket minskar avståndet som data skickas.

- Genom att avlasta arbetet till en CDN kan cachelagring minska nätverkstrafiken och belastningen på den ursprungliga servern. Detta minskar kostnader och kraven för programmet, även om det finns många användare.

Liknar hur cachelagring implementeras i en webbläsare, du kan styra hur cachelagring utförs i en CDN genom att skicka cache-direktiv huvuden. Cache-direktiv huvuden är HTTP-rubriker, vilket vanligtvis för att lägga till den ursprungliga servern. Även om de flesta av dessa huvuden ursprungligen var avsedda att adressera cachelagring i klientwebbläsare, används de nu också av alla mellanliggande cacheminnen, till exempel CDN-nät. 

Två huvuden kan användas för att definiera cache dokumentens: `Cache-Control` och `Expires`. `Cache-Control` är mer aktuell och företräde framför `Expires`om båda finns. Det finns två typer av huvuden som används för verifiering (kallas verifierare): `ETag` och `Last-Modified`. `ETag` är mer aktuell och företräde framför `Last-Modified`om båda anges.  

## <a name="cache-directive-headers"></a>Cache-direktiv rubriker

> [!IMPORTANT]
> Som standard en Azure CDN-slutpunkt som är optimerad för DSA ignorerar cache-direktiv sidhuvuden och kringgår cachelagring. För **Azure CDN Standard från Verizon** och **Azure CDN Standard från Akamai** profiler, du kan ändra hur en Azure CDN-slutpunkt behandlar dessa huvuden med hjälp av [cachelagringsregler för CDN](cdn-caching-rules.md)att aktivera cachelagring. För **Azure CDN Premium från Verizon** profiler kan du använda den [regelmotor](cdn-rules-engine.md) att aktivera cachelagring.

Azure CDN stöder följande HTTP-cache-direktiv huvuden, som definierar cachelagringens varaktighet och delning av cachen.

**Cache-Control:**
- Lanserades i HTTP 1.1 att ge web utgivare mer kontroll över deras innehåll och att adressera begränsningar i `Expires` huvud.
- Åsidosätter den `Expires` huvudet om både den och `Cache-Control` definieras.
- När den används i en HTTP-begäran från klienten till CDN-POP `Cache-Control` ignoreras av alla Azure CDN profiler som standard.
- När de används i ett HTTP-svar från klienten till CDN POP:
     - **Azure CDN Standard/Premium från Verizon** och **Azure CDN Standard från Microsoft** stöder alla `Cache-Control` direktiven.
     - **Azure CDN Standard från Akamai** stöder endast följande `Cache-Control` direktiven; alla övriga filer ignoreras:
         - `max-age`: En cache kan lagra innehållet för antalet sekunder som har angetts. Till exempel `Cache-Control: max-age=5`. Detta direktiv anger den maximala tidsperiod som innehållet ska anses vara ny.
         - `no-cache`: Cachelagrar innehållet, men Validera innehållet varje gång innan levereras från cacheminnet. Motsvarar `Cache-Control: max-age=0`.
         - `no-store`: Cachelagra aldrig innehållet. Ta bort innehåll om det tidigare sparade.

**Upphör att gälla:**
- Äldre huvud som introducerades i HTTP 1.0; stöds för bakåtkompatibilitet.
- Använder ett datumbaserat förfallotid med andra precision. 
- Liknar `Cache-Control: max-age`.
- Används när `Cache-Control` finns inte.

**Pragma:**
   - Hanteras inte av Azure CDN som standard.
   - Äldre huvud som introducerades i HTTP 1.0; stöds för bakåtkompatibilitet.
   - Används som ett huvud för begäran av klienten med följande direktiv: `no-cache`. Detta direktiv instruerar servern att skicka en ny version av resursen.
   - `Pragma: no-cache` motsvarar `Cache-Control: no-cache`.

## <a name="validators"></a>Systemhälsoverifierare

När cachen är inaktuella används HTTP cache verifierare för att jämföra den cachelagra versionen av en fil med versionen på den ursprungliga servern. **Azure CDN Standard/Premium från Verizon** stöder både `ETag` och `Last-Modified` verifierare som standard när **Azure CDN Standard från Microsoft** och **Azure CDN Standard från Akamai** stöder endast `Last-Modified` som standard.

**ETag:**
- **Azure CDN Standard/Premium från Verizon** stöder `ETag` som standard när **Azure CDN Standard från Microsoft** och **Azure CDN Standard från Akamai** inte.
- `ETag` definierar en sträng som är unik för varje fil och en version av en fil. Till exempel `ETag: "17f0ddd99ed5bbe4edffdd6496d7131f"`.
- Introducerades i HTTP 1.1 och är mer aktuell än `Last-Modified`. Användbar när den senaste ändringsdatum är svårt att avgöra.
- Stöder både stark verifiering och svaga validering; Azure CDN stöder dock bara stark verifiering. För stark verifiering två garantier måste vara byte för byte identiska. 
- En cache verifierar en fil som använder `ETag` genom att skicka en `If-None-Match` huvud med en eller flera `ETag` verifierare i begäran. Till exempel `If-None-Match: "17f0ddd99ed5bbe4edffdd6496d7131f"`. Om serverns version matchar en `ETag` verifieraren på listan skickar den statuskod 304 (inte ändrade) i sitt svar. Om versionen inte svarar servern med statuskod 200 (OK) och den uppdaterade resursen.

**Last Modified:**
- För **Azure CDN Standard/Premium från Verizon** endast `Last-Modified` används om `ETag` ingår inte i HTTP-svaret. 
- Anger det datum och tid som den ursprungliga servern har fastställt resursen senast ändrades. Till exempel `Last-Modified: Thu, 19 Oct 2017 09:28:00 GMT`.
- En cache verifierar en fil med hjälp av `Last-Modified` genom att skicka en `If-Modified-Since` huvud med ett datum och tid i begäran. Den ursprungliga servern Jämför detta datum med den `Last-Modified` rubriken för den senaste resursen. Om resursen inte har ändrats sedan den angivna tiden, returnerar servern statuskod 304 (inte ändrade) i sitt svar. Om resursen har ändrats, returnerar servern status code 200 (OK) och den uppdaterade resursen.

## <a name="determining-which-files-can-be-cached"></a>Bestämma vilka filer som kan cachelagras

Inte alla resurser som kan cachelagras. I följande tabell visas vilka resurser kan cachelagras, beroende på vilken typ av HTTP-svar. Resurser som levereras med HTTP-svar som inte uppfyller dessa villkor kan inte cachelagras. För **Azure CDN Premium från Verizon** , du kan använda regelmotor anpassa vissa av dessa villkor.

|                   | Azure CDN från Microsoft          | Azure CDN från Verizon | Azure CDN från Akamai        |
|-------------------|-----------------------------------|------------------------|------------------------------|
| Statuskoder för HTTP | 200, 203, 206, 300, 301, 410, 416 | 200                    | 200, 203, 300, 301, 302, 401 |
| HTTP-metoder      | GET, HEAD                         | HÄMTA                    | HÄMTA                          |
| Filstorleksbegränsningar  | 300 GB                            | 300 GB                 | -Allmänt web leveransoptimering: 1,8 GB<br />-Direktuppspelning optimeringar: 1,8 GB<br />-Stora filer optimering: 150 GB |

För **Azure CDN Standard från Microsoft** cachelagring för att fungera på en resurs på den ursprungliga servern måste ha stöd för något huvud och hämta HTTP-begäranden och innehållslängd värdena måste vara samma för alla HEAD och hämta HTTP-svar för tillgången. För en HEAD-begäran på den ursprungliga servern måste stödja HEAD-begäran och måste svara med samma sidhuvuden som om den fick en GET-begäran.

## <a name="default-caching-behavior"></a>Beteende för standardcachelagring

I följande tabell beskrivs standard cachelagring av frågesträngar för Azure CDN-produkter och deras optimeringar.

|    | Microsoft: Allmänt web leverans | Verizon: Allmänt web leverans | Verizon: DSA | Akamai: Allmänt web leverans | Akamai: DSA | Akamai: Stora Filhämtning | Akamai: Allmänt eller VOD-direktuppspelning |
|------------------------|--------|-------|------|--------|------|-------|--------|
| **Respektera ursprung**       | Ja    | Ja   | Nej   | Ja    | Nej   | Ja   | Ja    |
| **CDN cachelagringens varaktighet** | 2 dagar |7 dagar | Ingen | 7 dagar | Ingen | 1 dag | 1 år |

**Respektera ursprung**: Anger om ta hänsyn till den [stöds cache-direktiv huvuden](#http-cache-directive-headers) om de finns i HTTP-svaret från den ursprungliga servern.

**CDN cachelagringens varaktighet**: Anger hur lång tid som en resurs har cachelagrats på Azure CDN. Men om **respektera ursprung** är Ja och HTTP-svaret från den ursprungliga servern innehåller rubriken cache-direktiv `Expires` eller `Cache-Control: max-age`, Azure CDN använder duration-värde som anges av rubriken i stället. 

## <a name="next-steps"></a>Nästa steg

- Information om hur du anpassar och åsidosätta standardvärdet för cachelagring av frågesträngar på CDN genom cachelagring regler finns [kontroll Azure CDN cachelagring av frågesträngar med cachelagring regler](cdn-caching-rules.md). 
- Information om hur du använder frågesträngar att styra funktionssätt för cachelagring finns [kontroll Azure CDN cachelagring av frågesträngar med frågesträngar](cdn-query-string.md).



