---
title: Server begrepp i Azure-databas för PostgreSQL
description: Den här artikeln innehåller information och riktlinjer för att konfigurera och hantera Azure-databas för PostgreSQL-servrar.
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: article
ms.date: 05/22/2018
ms.openlocfilehash: f877f6df51cd7aed29260331d27d5c96f0584afc
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34640021"
---
# <a name="azure-database-for-postgresql-servers"></a>Azure Database for PostgreSQL-servrar
Den här artikeln innehåller information och riktlinjer för att arbeta med Azure-databas för PostgreSQL-servrar.

## <a name="what-is-an-azure-database-for-postgresql-server"></a>Vad är en Azure-databas för PostgreSQL-servern?
En Azure-databas för PostgreSQL-server är en central administrativ plats för flera databaser. Det är samma PostgreSQL server-konstruktion som du kanske känner till i den lokala världen. Mer specifikt tjänsten PostgreSQL hanteras, ger garanti vad gäller prestanda, exponerar åtkomst och funktioner på servernivå.

En Azure-databas för PostgreSQL-server:

- Skapas inom en Azure-prenumeration.
- Är den överordnade resursen för databaser.
- Ger ett namnområde för databaser.
- Är en behållare med starka livstid semantik - ta bort en server och tar bort de inneslutna databaserna.
- Collocates resurser i en region.
- Ger en Anslutningens slutpunkt för server och databas åtkomst (. postgresql.database.azure.com).
- Scope för av hanteringsprinciper som gäller för dess databaser: inloggning, brandvägg, användare, roller, konfigurationer och så vidare.
- Är tillgänglig i flera versioner. Mer information finns i [PostgreSQL-databas-versioner som stöds](concepts-supported-versions.md).
- Kan utökas av användare. Mer information finns i [PostgreSQL tillägg](concepts-extensions.md).

Du kan skapa en eller flera databaser i en Azure-databas för PostgreSQL-servern. Du kan välja att skapa en databas per server om du vill använda dig av samtliga resurser, eller skapa flera databaser som får dela på resurserna. Prissättning är strukturerade per server, baserat på konfigurationen av prisnivå, vCores, och lagringsutrymme (GB). Mer information finns i [prisnivåer](./concepts-pricing-tiers.md).

## <a name="how-do-i-connect-and-authenticate-to-an-azure-database-for-postgresql-server"></a>Hur gör ansluta och autentisera till en Azure-databas för PostgreSQL-servern?
Följande element att säkerställa säker åtkomst till databasen:

|||
|:--|:--|
| **Autentisering och auktorisering** | Azure-databas för PostgreSQL server stöder interna PostgreSQL-autentisering. Du kan ansluta och autentisera till servern med serverns administratör för inloggning. |
| **Protokoll** | Tjänsten stöder en message-baserat protokoll som används av PostgreSQL. |
| **TCP/IP** | Protokollet stöds via TCP/IP och över sockets för Unix-domän. |
| **Brandvägg** | För att skydda dina data, förhindrar en brandväggsregel all åtkomst till servern och dess databaser förrän du anger vilka datorer som har behörighet. Se [Azure-databas för PostgreSQL serverbrandväggsreglerna](concepts-firewall-rules.md). |

## <a name="managing-your-server"></a>Hantera servern
Du kan hantera Azure-databas för PostgreSQL-servrar med hjälp av den [Azure-portalen](https://portal.azure.com) eller [Azure CLI](/cli/azure/postgres).

När du skapar en server måste ställa du in autentiseringsuppgifterna för admin-användare. Administratören är den högsta privilegium användare som du har på servern. Det hör till rollen azure_pg_admin. Den här rollen har inte fullständiga superanvändare behörigheter. 

Attributet PostgreSQL superanvändare har tilldelats azure_superuser som tillhör hanterade tjänsten. Du har inte åtkomst till den här rollen.

En Azure-databas för PostgreSQL-servern har två standarddatabaser: 
- **postgres** -en standarddatabas som du kan ansluta till en gång servern skapas.
- **azure_maintenance** -databasen som används för att avgränsa de processer som tillhandahåller hanteringstjänster från användaråtgärder. Du har inte åtkomst till den här databasen.


## <a name="server-parameters"></a>Serverparametrar
Serverparametrar PostgreSQL Kontrollera konfigurationen av servern. I Azure-databas för PostgreSQL listan över parametrar som kan visas och redigeras med hjälp av Azure-portalen eller Azure CLI. 

Som en hanterad tjänst för Postgres konfigurerbara parametrarna i Azure-databas för PostgreSQL är en delmängd av parametrarna i en lokal instans av Postgres (Mer information om Postgres parametrar finns i [PostgreSQL dokumentationen](https://www.postgresql.org/docs/9.6/static/runtime-config.html)). Din Azure-databas för PostgreSQL server aktiveras med standardvärden för varje parameter skapas. Vissa parametrar som kräver en server startas om eller superanvändare åtkomst för att ändringarna ska börja gälla kan inte konfigureras av användaren.


## <a name="next-steps"></a>Nästa steg
- En översikt över tjänsten finns [Azure Database PostgreSQL översikt](overview.md).
- Information om specifik resurs kvoter och begränsningar baserat på din **tjänstnivån**, se [tjänstnivåer](concepts-pricing-tiers.md).
- Information om hur du ansluter till tjänsten finns [anslutningsbibliotek för Azure-databas för PostgreSQL](concepts-connection-libraries.md).
- Visa och redigera serverparametrar via [Azure-portalen](howto-configure-server-parameters-using-portal.md) eller [Azure CLI](howto-configure-server-parameters-using-cli.md).
