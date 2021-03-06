---
title: ta med fil
description: ta med fil
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: include
ms.date: 05/03/2018
ms.author: raynew
ms.custom: include file
ms.openlocfilehash: 2464fa41deaa1e9c43e5f53e1a900ca11b582040
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2018
---
**Konfigurationsprocessen/server-krav**

**Komponent** | **Krav** 
--- | ---
**MASKINVARA** | 
**CPU-kärnor** | 8 
**RAM-MINNE** | 16 GB
**Antal diskar** | 3, inklusive OS-disk, disk för processen server cache och kvarhållningsenhetens för återställning efter fel 
**Ledigt diskutrymme (processen servercache)** | 600 GB
**Ledigt diskutrymme (kvarhållningsdisken)** | 600 GB
**PROGRAMVARA** | 
**Operativsystem** | Windows Server 2012 R2 <br> Windows Server 2016
**Operativsystemets nationella inställningar** | Engelska (en-us)
**Windows Server-roller** | Aktivera inte dessa roller: <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V 
**Grupprinciper** | Aktivera inte dessa grupp-principer: <br> -Förhindra åtkomst till Kommandotolken. <br> -Förhindra åtkomst till verktyg för redigering av registret. <br> -Förtroende för bifogade filer. <br> -Aktivera körning av skript. <br> [Läs mer](https://technet.microsoft.com/library/gg176671(v=ws.10).aspx)
**IIS** | -Ingen redan befintliga standardwebbplatsen <br> -Ingen befintliga webbplats/programmet lyssnar på port 443 <br>-Aktivera [anonym autentisering](https://technet.microsoft.com/library/cc731244(v=ws.10).aspx) <br> -Aktivera [FastCGI](https://technet.microsoft.com/library/cc753077(v=ws.10).aspx) inställning 
**NÄTVERK** | 
**IP-adresstyp** | Statisk 
**Internet-åtkomst** | Servern behöver åtkomst till dessa URL: er (direkt eller via proxy) <br> - \*.accesscontrol.windows.net<br> - \*.backup.windowsazure.com <br>- \*.store.core.windows.net<br> - \*.blob.core.windows.net<br> - \*.hypervrecoverymanager.windowsazure.com <br> - https://cdn.mysql.com/archives/mysql-5.5/mysql-5.5.37-win32.msi (om du konfigurerar en server configuration) <br> - time.nist.gov <br> - time.windows.com 
**Portar** | 443 (kontrolkanalsorchestration)<br>9443 (dataöverföring) 
**VMWARE (när du ställer in konfigurationsprocessen/servern som en VMware VM)**
**NIC typen** | VMXNET3  
**VMware vSphere PowerCLI körs på Virtuella* | [PowerCLI version 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1 "PowerCLI 6.0")

**Konfigurationsprocessen/server storleksanpassa krav**

**CPU** | **Minne** | **Cache-disk** | **Dataändringshastighet** | **Replikerade datorer**
--- | --- | --- | --- | ---
8 vCPUs<br/><br/> 2 sockets * 4 kärnor @ 2,5 GHz | 16GB | 300 GB | 500 GB eller mindre | < 100 datorer
12 vCPUs<br/><br/> 2 socks * 6 kärnor @ 2,5 GHz | 18 GB | 600 GB | 500 GB - 1 TB | 100-150 datorer
16 vCPUs<br/><br/> 2 socks * 8 kärnor @ 2,5 GHz | 32 GB | 1 TB | 1 – 2 TB | 150-200 datorer

