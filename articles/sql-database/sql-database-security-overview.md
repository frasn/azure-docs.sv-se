---
title: Säkerhetsöversikt för Azure SQL Database | Microsoft Docs
description: Läs mer om Azure SQL Database och SQL Server-säkerhet, inklusive skillnaderna mellan moln och SQL Server lokalt.
services: sql-database
author: giladm
manager: craigg
ms.reviewer: carlrab
ms.service: sql-database
ms.custom: security
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: giladm
ms.openlocfilehash: a40ca715c15540bf7048fae8b5dde152890eb1c1
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34648334"
---
# <a name="securing-your-sql-database"></a>Säkra din SQL Database

Den här artikeln går igen grunderna för att skydda datanivån hos ett program med Azure SQL Database. I synnerhet hjälper i den här artikeln dig att komma igång med resurser för att skydda data, styra åtkomst och proaktiv övervakning. 

En fullständig översikt över tillgängliga säkerhetsfunktioner på alla stilar av SQL finns i [Security Center för SQL Server Database Engine och Azure SQL Database](https://msdn.microsoft.com/library/bb510589). Mer information finns också i [Tekniskt faktablad om säkerhet och Azure SQL Database](https://download.microsoft.com/download/A/C/3/AC305059-2B3F-4B08-9952-34CDCA8115A9/Security_and_Azure_SQL_Database_White_paper.pdf) (PDF).

## <a name="protect-data"></a>Skydda data

### <a name="encryption"></a>Kryptering
SQL-databas skyddar dina data genom att tillhandahålla kryptering för data i rörelse med [Transport Layer Security](https://support.microsoft.com/kb/3135244), för data i vila med [Transparent datakryptering](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql), och för data som används med [ Always Encrypted](https://msdn.microsoft.com/library/mt163865.aspx). 

> [!IMPORTANT]
>Alla anslutningar till Azure SQL Database kräver filkryptering (SSL/TLS) hela tiden medan data är "under överföringen" till och från databasen. I anslutningssträngen för ditt program måste du ange parametrar för att kryptera anslutningen och *inte* ska lita på certifikatet (det gör du om du kopierar anslutningssträngen utanför Azure-portalen), annars den anslutningen verifiera inte identiteten för servern och är sårbara för ”man-in-the-middle”-attacker. För t.ex. ADO.NET-drivrutinen är dessa parametrar för anslutningssträngen **Encrypt=True** och **TrustServerCertificate=False**. Information om TLS- och anslutning finns [TLS-överväganden](sql-database-connect-query.md#tls-considerations-for-sql-database-connectivity)

För andra sätt att kryptera dina data, kan du överväga:

* [Kryptering på cellnivå](https://msdn.microsoft.com/library/ms179331.aspx) för att kryptera vissa kolumner eller även celler av data med olika krypteringsnycklar.
* Om du behöver en Hardware Security Module eller central hantering av krypteringsnyckelns hierarki, bör du använda [Azure Key Vault med SQL Server i en virtuell Azure-dator](http://blogs.technet.com/b/kv/archive/2015/01/12/using-the-key-vault-for-sql-server-encryption.aspx).

### <a name="data-discovery--classification"></a>Identifiering av data & klassificering
Identifiering av data och klassificering (för närvarande i förhandsversion) innehåller avancerade funktioner finns inbyggda i Azure SQL Database för identifiering, klassificering, etiketter och skydda känsliga data i databasen. Identifiera och klassificera dina största känsliga data (företag för finansiella, hälsovård, personligt identifierbar information, etc.) kan en nyckelroll i din organisation Information protection status. Den kan fungera som infrastrukturen för:

- Olika säkerhetsscenarier, till exempel övervakning (granskning) och aviseringar om avvikande tillgång till känsliga data.
- Kontrollera åtkomst till och Härdning av säkerheten för, databaser som innehåller mycket känslig data.
- Hjälper att uppfylla data sekretess normer och regelefterlevnad krav.

Mer information finns i [Kom igång med SQL DB Data Discovery & klassificering](sql-database-data-discovery-and-classification.md). 

## <a name="control-access"></a>Styr åtkomst
SQL Database skyddar dina data genom att begränsa åtkomsten till databasen med hjälp av brandväggsregler, autentiseringsmekanismer som kräver att användare bevisar sin identitet och auktorisering till data via rollbaserade medlemskap och behörigheter, såväl som säkerhet på radnivå och dynamisk datamaskering. En beskrivning av åtkomstkontrollfunktionerna i SQL Database finns i [Kontrollera åtkomst](sql-database-control-access.md).

> [!IMPORTANT]
> Hantering av databaser och logiska servrar inom Azure kontrolleras av din portalanvändarkontos rolltilldelningar. Mer information om den här artikeln finns [rollbaserad åtkomstkontroll i Azure-portalen](../role-based-access-control/overview.md).
>

### <a name="firewall-and-firewall-rules"></a>Brandvägg och brandväggsregler
För att hjälpa skydda dina data, förhindrar brandväggar all åtkomst till din databasserver tills du anger vilka datorer som har behörighet med hjälp av [brandväggsregler](sql-database-firewall-configure.md). Brandväggen ger åtkomst till databaser baserat på vilken IP-adress som varje begäran kommer från.

### <a name="authentication"></a>Autentisering
SQL-databasautentisering refererar till hur du styrker din identitet när du ansluter till databasen. SQL Database stöder två typer av autentisering:
* **SQL-autentisering**, som använder ett användarnamn och lösenord. När du skapade den logiska servern för databasen angav du en "serveradministratörsinloggning” med ett användarnamn och lösenord. Med dessa autentiseringsuppgifter kan du autentisera till en databas på servern som databasens ägare eller "dbo." 
* **Azure Active Directory-autentisering**, som använder identiteter som hanteras av Azure Active Directory och stöder hanterade och integrerade domäner. Använd Active Directory-autentisering (integrerad säkerhet) [närhelst det går](https://msdn.microsoft.com/library/ms144284.aspx). Om du vill använda Azure Active Directory-autentisering måste du skapa en annan serveradministratör som kallas "Azure AD-admin," som tillåts administrera Azure AD-användare och -grupper. Den här administratören kan också utföra alla åtgärder som en vanlig serveradministratören kan. Se [Ansluta till SQL Database genom att använda Azure Active Directory-autentisering](sql-database-aad-authentication.md) för en genomgång av hur du skapar en Azure AD-administratör för att aktivera Azure Active Directory-autentisering.

### <a name="authorization"></a>Auktorisering
Auktorisering hänvisar till vad en användare kan göra inom en Azure SQL-databas och det styrs av ditt användarkontos databas rollmedlemskap och behörigheter på objektnivå. Ett bra tips är att du ska ge användare så få behörigheter som möjligt. Serveradministratörskontot som du ansluter med är medlem i db_owner som har behörighet att göra vad som helst i databasen. Spara det här kontot för att distribuera schemauppgraderingar och andra hanteringsåtgärder. Använd kontot "ApplicationUser" med mer begränsade behörigheter för att ansluta från ditt program till databasen med den minsta behörigheten som krävs av programmet.

### <a name="row-level-security"></a>Säkerhet på radnivå
Säkerhet på radnivå ger kunder möjlighet att styra åtkomsten till rader i en databastabell baserat på egenskaperna för användaren som kör en fråga (t.ex. grupmedlemskap eller körningskontext). Mer information finns i [Säkerhet på radnivå](https://msdn.microsoft.com/library/dn765131).

### <a name="dynamic-data-masking"></a>Dynamisk datamaskning 
SQL-databas dynamisk datamaskning begränsar exponering av känsliga data genom att maskera till icke-privilegierade användare. Dynamisk datamaskering automatiskt identifierar potentiellt känsliga data i Azure SQL Database och innehåller tillämplig rekommendationer att maskera fälten med minimal påverkan på programnivån. Det fungerar genom att dölja känslig data i resultatuppsättningen för en fråga över angivna databasfält, medan data i databasen förblir oförändrad. Mer information finns i [Kom igång med SQL Database dynamisk datamaskning](sql-database-dynamic-data-masking-get-started.md).

## <a name="proactive-monitoring"></a>Proaktiv övervakning
SQL Database skyddar dina data genom att tillhandahålla funktioner för granskning och hotidentifiering. 

### <a name="auditing"></a>Granskning
SQL Database-granskning spårar databasaktiviteter och hjälper dig att upprätthålla regelefterlevnad, genom att registrera databashändelser till en granskningslogg på ditt Azure Storage-konto. Granskning låter dig förstå pågående databasaktiviteter, samt analysera och undersöka historiska aktiviteter för att identifiera potentiella hot eller misstänkt missbruk och säkerhetsöverträdelser. Ytterligare information finns i [Kom igång med SQL Database-granskning](sql-database-auditing.md).  

### <a name="threat-detection"></a>Hotidentifiering
Hotidentifiering kompletterar granskning genom att tillhandahålla ett extra lager av säkerhet för tillgångsinformation är inbyggda i Azure SQL Database-tjänsten som identifierar onormal och potentiellt skadliga försök att komma åt eller utnyttja databaser. Du meddelas om misstänkta aktiviteter, potentiella säkerhetsproblem och SQL injection attacker samt avvikande databasen åtkomstmönster. Hotidentifieringsaviseringar kan visas från [Azure Security Center](https://azure.microsoft.com/services/security-center/) och ange information om misstänkt aktivitet och rekommenderar åtgärd att undersöka och minska risken. Hotidentifiering kostnader $15/server/månad. Det är gratis för de första 60 dagarna. Mer information finns i [Kom igång med SQL Database Threat Detection](sql-database-threat-detection.md).
 
## <a name="compliance"></a>Efterlevnad
Förutom ovanstående egenskaper och funktioner som kan hjälpa ditt program uppfylla olika säkerhetskrav, Azure SQL Database även deltar i reguljära granskningar och har certifierats mot ett antal efterlevnadsstandarder. Mer information finns i [Microsoft Azure säkerhetscenter](https://azure.microsoft.com/support/trust-center/), där du hittar den senaste listan med [SQL Database-kompatibilitetscertifieringar](https://www.microsoft.com/trustcenter/compliance/complianceofferings/).


## <a name="security-management"></a>Säkerhetshantering

SQL-databas som hjälper dig att hantera din datasäkerhet genom att tillhandahålla databasen genomsökningar och en centraliserad säkerhet instrumentpanel med hjälp av [SQL Vulnerability Assessment](sql-vulnerability-assessment.md).

**Utvärdering av säkerhetsrisker**: [SQL Vulnerability Assessment](sql-vulnerability-assessment.md) (för närvarande i förhandsversion) är ett enkelt att konfigurera verktyget inbyggda i Azure SQL Database som kan hjälpa dig identifiera, spåra och åtgärda eventuella databas säkerhetsproblem. Bedömningen kör en genomsökning säkerhetsproblem på databasen och genererar en rapport som ger inblick i dina säkerhetstillstånd, inklusive tillämplig steg för att lösa säkerhetsproblem och förbättra säkerheten för din databas. Rapporten assessment kan anpassas för din miljö genom att ange en acceptabel baslinje för behörigheter, funktionen konfigurationer och inställningar. Detta kan hjälpa dig att:

- Uppfylla krav på databasen genomsökning rapporter. 

- Uppfyller data sekretess standarder. 

- Övervaka en dynamisk databas-miljö där ändringar har svårt att spåra.

Mer information finns i [SQL Vulnerability Assessment](sql-vulnerability-assessment.md).

## <a name="next-steps"></a>Nästa steg

- En beskrivning av åtkomstkontrollfunktionerna i SQL Database finns i [Kontrollera åtkomst](sql-database-control-access.md).
- En beskrivning av databasen granskning finns [SQL Database auditing](sql-database-auditing.md).
- En beskrivning av hotidentifiering finns [SQL-databas hotidentifiering](sql-database-threat-detection.md).
