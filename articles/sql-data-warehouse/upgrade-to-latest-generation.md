---
title: Uppgradera till den senaste generationen Azure SQL Data Warehouse | Microsoft Docs
description: Uppgradera Azure SQL Data Warehouse till senaste generationens Azure maskin- och storage-arkitektur.
services: sql-data-warehouse
author: kevinvngo
manager: craigg-msft
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 58d65ef05ed872bb357070de9866253baea5dc70
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2018
---
# <a name="optimize-performance-by-upgrading-sql-data-warehouse"></a>Optimera prestanda genom att uppgradera SQL Data Warehouse
Uppgradera Azure SQL Data Warehouse till senaste generationens Azure maskin- och storage-arkitektur.

## <a name="why-upgrade"></a>Varför uppgradera?
Du kan nu sömlöst uppgradera till SQL Data Warehouse Gen2 i Azure-portalen. Om du har ett Gen1 data warehouse, rekommenderas uppgradera. Genom att uppgradera, du kan använda den senaste generationen Azure maskinvara och förbättrad lagringsarkitektur. Du kan dra nytta av bättre prestanda och högre skalbarhet obegränsad kolumner lagring. 

## <a name="applies-to"></a>Gäller för
Den här uppgraderingen gäller Gen1 datalager.

## <a name="sign-in-to-the-azure-portal"></a>Logga in på Azure Portal

Logga in på [Azure-portalen](https://portal.azure.com/).

## <a name="before-you-begin"></a>Innan du börjar
> [!NOTE]
> Om ditt befintliga Gen1 data warehouse inte är i en region där Gen2 är tillgänglig, kan du [geo-återställning till Gen2](https://docs.microsoft.com/en-us/azure/sql-data-warehouse/sql-data-warehouse-restore-database-powershell#restore-from-an-azure-geographical-region) via PowerShell till en region som stöds.
> 
>

1. Om datalagret Gen1 som ska uppgraderas är pausad [återuppta datalagret](pause-and-resume-compute-portal.md).
2. Att förbereda för ett par minuter stillestånd. 



## <a name="start-the-upgrade"></a>Starta uppgraderingen

1. Gå till din Gen1 data warehouse i Azure-portalen och klicka på **uppgradera till Gen2**: ![Upgrade_1](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_1.png)

2. Som standard **markerar du den föreslagna prestandanivån** för datalagret baserat på din nuvarande prestandanivå på Gen1 med hjälp av mappningen nedan:
    
| Gen1 | Gen2 |
| :----------------------: | :-------------------: |
|      DW100 – DW1000      |        DW1000c        |
|          DW1200          |        DW1500c        |
|          DW1500          |        DW1500c        |
|          DW2000          |        DW2000c        |
|          DW3000          |        DW3000c        |
|          DW6000          |        DW6000c        |


3. Kontrollera din arbetsbelastning har slutförts körs och offline innan du uppgraderar. Du får driftstopp om en stund innan ditt data warehouse är online igen som ett Gen2 data warehouse. **Klicka på Uppgradera**. Priset för prestandanivån Gen2 är halva för närvarande inaktiverat i förhandsversionen:
    
    ![Upgrade_2](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_2.png)

4. **Övervaka uppgraderingen** genom att kontrollera statusen i Azure-portalen:

   ![Upgrade3](./media/sql-data-warehouse-upgrade-to-latest-generation/Upgrade_to_Gen2_3.png)
   
   Det första steget i uppgraderingsprocessen genomgår åtgärden (”uppgradera - Offline”) där avbryts alla sessioner och anslutningar kommer att tas bort. 
   
   Det andra steget i uppgraderingsprocessen är datamigrering (”uppgradera - Online”). Migrering av data är bakgrunden online takt som långsamt flyttas kolumner data från den gamla lagringsarkitekturen till den nya lagringsarkitekturen i genom att använda en lokal SSD-cache. Under den här tiden kommer ditt data warehouse vara online för frågor och lästes in. Alla dina data blir tillgängliga att fråga oavsett om den har migrerats eller inte. Migrering av data sker i olika takt beroende på datastorleken på din, prestandanivå och antalet columnstore-segment. 

5. **Hitta datalagret Gen2** med hjälp av SQL-databas Bläddra-bladet. 

> [!NOTE]
> Det finns ett problem där Gen2 data datalager inte visas i SQL data warehouse Bläddra bladet. Använd Bläddra-bladet SQL-databas för att hitta ditt uppgraderade Gen2 data warehouse. Vi arbetar aktivt på den här snabbkorrigeringen.
> 

6. **Valfria rekommendation:** för att påskynda bakgrund migrerar data, rekommenderas att tvinga dataflyttning omedelbart genom att köra [Alter Index rebuild](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-index) för alla columnstore-tabeller i en större SLO och resurs klass. Den här åtgärden är offline jämfört med bakgrunden takt; migrering av data kommer dock att mycket snabbare där du sedan kan dra nytta av den nya arkitekturen för Förbättrad lagring när slutfördes med hög kvalitet rowgroups. 

Följande frågan genererar krävs Alter Index Rebuild kommandona för att påskynda migrerar data:

```sql
SELECT 'ALTER INDEX [' + idx.NAME + '] ON [' 
       + Schema_name(tbl.schema_id) + '].[' 
       + Object_name(idx.object_id) + '] REBUILD ' + ( CASE 
                                                         WHEN ( 
                                                     (SELECT Count(*) 
                                                      FROM   sys.partitions 
                                                             part2 
                                                      WHERE  part2.index_id 
                                                             = idx.index_id 
                                                             AND 
                                                     idx.object_id = 
                                                     part2.object_id) 
                                                     > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              ELSE '' 
                                                       END ) + '; SELECT ''[' + 
              idx.NAME + '] ON [' + Schema_name(tbl.schema_id) + '].[' + 
              Object_name(idx.object_id) + '] ' + ( 
              CASE 
                WHEN ( (SELECT Count(*) 
                        FROM   sys.partitions 
                               part2 
                        WHERE 
                     part2.index_id = 
                     idx.index_id 
                     AND idx.object_id 
                         = part2.object_id) > 1 ) THEN 
              ' PARTITION = ' 
              + Cast(part.partition_number AS NVARCHAR(256)) 
              + ' completed'';' 
              ELSE ' completed'';' 
                                                    END ) 
FROM   sys.indexes idx 
       INNER JOIN sys.tables tbl 
               ON idx.object_id = tbl.object_id 
       LEFT OUTER JOIN sys.partitions part 
                    ON idx.index_id = part.index_id 
                       AND idx.object_id = part.object_id 
WHERE  idx.type_desc = 'CLUSTERED COLUMNSTORE'; 
```



## <a name="next-steps"></a>Nästa steg
Uppgraderade datalagret är online. Om du vill dra nytta av förbättrad arkitektur, se [resursklasser för hantering av arbetsbelastning](resource-classes-for-workload-management.md).
 
