---
title: 'Azure Analysis Services självstudiekurs 8: Skapa perspektiv | Microsoft Docs'
description: Beskriver hur du skapar perspektiv i Azure Analysis Services-självstudieprojektet.
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/12/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 865959a5595de7b5ac65a9915e484182a820b5b4
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34598716"
---
# <a name="create-perspectives"></a>Skapa perspektiv

I den här lektionen skapar du ett perspektiv för Internetförsäljning. Ett perspektiv definierar en visningsbar delmängd av en modell som ger fokuserade, affärsspecifika eller programspecifika översiktsvyer. När en användare ansluter till en modell med hjälp av ett perspektiv visas endast de modellobjekt (tabeller, kolumner, mått, hierarkier och KPI:er), eller fält, som definierats i det perspektivet. Mer information finns i [Partitioner](https://docs.microsoft.com/sql/analysis-services/tabular-models/perspectives-ssas-tabular).
  
Perspektivet Internetförsäljning som du skapar i den här lektionen exkluderar tabellobjektet DimCustomer. När du skapar ett perspektiv som exkluderar visning av ett visst objekt existerar objektet fortfarande i modellen. Men det är inte synligt i fältlistan i rapporteringsklienten. Beräknade kolumner och mått, som antingen är inkluderade i ett perspektiv eller inte, kan fortfarande göra beräkningar från objektdata som exkluderas.  
  
I den här lektionen visar vi hur du skapar perspektiv och du får bekanta dig med tabellmodellens redigeringsverktyg. Om du senare väljer att utöka modellen med ytterligare tabeller kan du skapa ytterligare perspektiv för att definiera olika översiktsvyer för modellen, till exempel Lager och Försäljning.  
  
Uppskattad tidsåtgång för den här lektionen: **Fem minuter**  
  
## <a name="prerequisites"></a>Förutsättningar  
Det här avsnittet ingår i självstudiekursen för tabellmodellering som bör slutföras i rätt ordning. Innan du utför uppgifterna under den här lektionen måste du ha slutfört föregående lektion: [Lektion 7: Skapa KPI:er](../tutorials/aas-lesson-7-create-key-performance-indicators.md).  
  
## <a name="create-perspectives"></a>Skapa perspektiv  
  
#### <a name="to-create-an-internet-sales-perspective"></a>Så här skapar du ett perspektiv för Internetförsäljning  
  
1.  Klicka på **Modell**-menyn > **Perspektiv** > **Skapa och hantera**.  
  
2.  Klicka på **Nytt perspektiv** i dialogrutan **Perspektiv**.  
  
3.  Dubbelklicka på kolumnrubriken **Nytt perspektiv** och ändra sedan namnet till **Internetförsäljning**.  
  
4.  Markera alla tabeller *utom* **DimCustomer**.  
  
    ![aas-lesson8-perspectives](../tutorials/media/aas-lesson8-perspectives.png)
  
    I en senare lektion använder du funktionen Analysera i Excel för att testa perspektivet. Fältlistan för pivottabell i Excel innehåller alla tabeller utom tabellen DimCustomer.  

## <a name="whats-next"></a>Nästa steg
[Lektion 9: Skapa hierarkier](../tutorials/aas-lesson-9-create-hierarchies.md).
  
  
  
  
