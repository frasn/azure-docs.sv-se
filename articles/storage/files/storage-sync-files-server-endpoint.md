---
title: Lägg till/ta bort en slutpunkt för server filsynkronisering i Azure (förhandsversion) | Microsoft Docs
description: Lär dig vad du bör tänka på när du planerar för distribution av en Azure-filer.
services: storage
documentationcenter: ''
author: wmgries
manager: aungoo
editor: tamram
ms.assetid: 297f3a14-6b3a-48b0-9da4-db5907827fb5
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2018
ms.author: wgries
ms.openlocfilehash: 93331dd936a6d7b30ca18743d2079900421b2620
ms.sourcegitcommit: c722760331294bc8532f8ddc01ed5aa8b9778dec
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34738487"
---
# <a name="addremove-an-azure-file-sync-preview-server-endpoint"></a>Lägg till/ta bort en Azure-filsynkronisering (förhandsgranskning) serverslutpunkt
Med Azure File Sync (förhandsversionen) kan du centralisera din organisations filresurser i Azure Files med samma flexibilitet, prestanda och kompatibilitet som du får om du använder en lokal filserver. Det gör du genom att omvandla dina Windows-servrar till en snabb cache med Azure-filresursen. Du kan använda alla protokoll som är tillgängliga på Windows Server för att komma åt data lokalt (inklusive SMB, NFS och FTPS) och du kan ha så många cacheminnen som du behöver över hela världen.

En *serverslutpunkt* representerar en specifik plats på en *registrerad server*, till exempel en mapp på en server-volym eller i roten på volymen. Flera server-slutpunkter kan finnas på samma volym om deras namnområden inte överlappar (till exempel F:\sync1 och F:\sync2). Du kan konfigurera molnet lagringsnivåer principer individuellt för varje serverslutpunkt. Om du lägger till en plats med en befintlig uppsättning filer på servern som en serverslutpunkt i en grupp för synkronisering av slås filerna samman med andra filer redan på andra slutpunkter i gruppen synkronisering.

Se [hur du distribuerar Azure filsynkronisering (förhandsgranskning)](storage-sync-files-deployment-guide.md) information om hur du distribuerar Azure filsynkronisering slutpunkt till slutpunkt.

## <a name="prerequisites"></a>Förutsättningar
För att skapa en serverslutpunkt för, måste du först se till att följande villkor är uppfyllda: 
- Servern filsynkronisering för Azure-agenten har installerats och registrerats. Anvisningar för att installera Azure File Sync-agenten finns i den [registrera/avregistrera en server med Azure filsynkronisering (förhandsgranskning)](storage-sync-files-server-registration.md) artikel. 
- Se till att en lagring Sync-tjänsten har distribuerats. Se [hur du distribuerar Azure filsynkronisering (förhandsgranskning)](storage-sync-files-deployment-guide.md) mer information om hur du distribuerar en tjänst för synkronisering av lagring. 
- Se till att en grupp för synkronisering har distribuerats. Lär dig hur du [skapar du en synkronisering grupp](storage-sync-files-deployment-guide.md#create-a-sync-group).
- Kontrollera att servern är ansluten till internet och att Azure är tillgänglig. Vi använder port 443 för all kommunikation mellan servern och vår tjänst.

## <a name="add-a-server-endpoint"></a>Lägg till en serverslutpunkt
Om du vill lägga till en serverslutpunkt för, navigerar du till gruppen önskade synkronisering och välj ”Lägg till server endpoint”.

![Lägg till en ny serverslutpunkt i fönstret sync grupp](media/storage-sync-files-server-endpoint/add-server-endpoint-1.png)

Följande information krävs under **Lägg till serverslutpunkt**:

- **Registrerad server**: namnet på servern eller klustret att skapa Serverslutpunkten på.
- **Sökvägen**: sökväg på Windows Server som ska synkroniseras som en del av gruppen synkronisering.
- **Cloud Tiering**: en växel för att aktivera eller inaktivera molnet skiktning. När aktiverad molnet lagringsnivåer kommer *nivå* filer till Azure-filresurser. Detta konverterar lokala filresurser till en cache i stället för en fullständig kopia av datauppsättningen, som hjälper dig att hantera effektivt diskutrymme på servern.
- **Ledigt utrymme på volymen**: mängden ledigt utrymme som ska reserveras på den volym som finns i Serverslutpunkten. Till exempel om volymens lediga utrymme anges till 50% på en volym med en enskild serverslutpunkt ska ungefär hälften av data vara nivåer till Azure Files. Oavsett om molnet skiktning är aktiverat, Azure-filresursen har alltid en fullständig kopia av data i gruppen synkronisering.

Välj **skapa** att lägga till Serverslutpunkten. Filerna i ett namnområde för en grupp för synkronisering kommer nu att synkroniseras. 

## <a name="remove-a-server-endpoint"></a>Ta bort en serverslutpunkt för
Om du vill fortsätta att använda Azure filsynkronisering för en viss server-slutpunkt, kan du ta bort Serverslutpunkten. 

> [!Warning]  
> Försök inte att felsöka problem med synkronisering, molnet skiktning eller någon aspekt av Azure filen Sync genom att ta bort och återskapa Serverslutpunkten om du inte uttryckligen har angett till av en Microsoft-tekniker. Ta bort en serverslutpunkt för är en destruktiva åtgärd och nivåindelade filer i Serverslutpunkten kommer inte ”återanslutas” till deras platser på Azure-filresursen när Serverslutpunkten återskapas, vilket innebär synkroniserade fel. Observera också, skiktad filer som finns utanför servern endpoint namnområde kan vara förlorade permanent. Nivåindelad filer kan finnas i din server-slutpunkt, även om molnet skiktning aktiverades aldrig.

Inaktivera molnet skiktning på Serverslutpunkten för att säkerställa att alla nivåindelade filer hämtas innan du tar bort Serverslutpunkten, och köra följande PowerShell-cmdlet om du vill återkalla alla nivåindelade filer i namnområdet server slutpunkt:

```PowerShell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint>
```

> [!Note]  
> Om den lokala volymen som värd för servern inte har tillräckligt med ledigt utrymme för att återställa alla nivåindelade data i `Invoke-StorageSyncFileRecall` cmdlet misslyckas.  

Ta bort Serverslutpunkten:

1. Gå till lagring synkroniseringstjänsten där servern är registrerad.
2. Navigera till den önskade sync-gruppen.
3. Ta bort serverslutpunkt som du önskar i gruppen synkronisering i synkroniseringstjänsten för lagring. Detta kan åstadkommas genom att högerklicka på relevant Serverslutpunkten i fönstret sync grupp.

    ![Ta bort en serverslutpunkt för från en grupp för synkronisering](media/storage-sync-files-server-endpoint/remove-server-endpoint-1.png)

## <a name="next-steps"></a>Nästa steg
- [Registrera/avregistrera en server med Azure filsynkronisering (förhandsgranskning)](storage-sync-files-server-registration.md)
- [Planera för distribution av en Azure filsynkronisering](storage-sync-files-planning.md)