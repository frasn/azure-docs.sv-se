---
title: Om nätverksfunktionerna i Azure till Azure-haveriberedskap med hjälp av Azure Site Recovery | Microsoft Docs
description: Ger en översikt över nätverk för replikering av virtuella Azure-datorer med hjälp av Azure Site Recovery.
services: site-recovery
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 05/31/2018
ms.author: sujayt
ms.openlocfilehash: 7e717d06aaaef6031a0a3b26c5caf76f0c8c11df
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34715946"
---
# <a name="about-networking-in-azure-to-azure-replication"></a>Om nätverk i Azure till Azure-replikering



Den här artikeln innehåller vägledning för nätverk när du replikerar och återställa virtuella Azure-datorer från en region till en annan, med hjälp av [Azure Site Recovery](site-recovery-overview.md).

## <a name="before-you-start"></a>Innan du börjar

Lär dig hur Site Recovery tillhandahåller katastrofåterställning för [det här scenariot](azure-to-azure-architecture.md).

## <a name="typical-network-infrastructure"></a>Vanlig nätverksinfrastruktur

Följande diagram visar en typisk Azure miljö för program som körs på virtuella Azure-datorer:

![kund-miljö](./media/site-recovery-azure-to-azure-architecture/source-environment.png)

Om du använder Azure ExpressRoute eller en VPN-anslutning från ditt lokala nätverk till Azure, är miljön:

![kund-miljö](./media/site-recovery-azure-to-azure-architecture/source-environment-expressroute.png)

Oftast skyddas nätverk brandväggar och nätverkssäkerhetsgrupper (NSG: er). Brandväggar Använd URL eller IP-baserade vitlistning för att kontrollera nätverksanslutningen. NSG: er innehåller regler som använder IP-adressintervall för att kontrollera nätverksanslutningen.

>[!IMPORTANT]
> Med hjälp av en autentiserad proxyserver för att kontrollera nätverksanslutningen stöds inte av Site Recovery och det går inte att aktivera replikering.


## <a name="outbound-connectivity-for-urls"></a>Utgående anslutning för webbadresser

Om du använder en URL-baserade brandväggen proxy för att styra utgående anslutning Tillåt webbadresserna Site Recovery:


**URL** | **Detaljer**  
--- | ---
*.blob.core.windows.net | Krävs så att data kan skrivas till cache lagringskonto i regionen källa från den virtuella datorn.
login.microsoftonline.com | Krävs för autentiseringen och auktoriseringen till Site Recovery-webbadresser.
*.hypervrecoverymanager.windowsazure.com | Krävs för att kommunikation för Site Recovery-tjänsten kan ske från den virtuella datorn.
*.servicebus.windows.net | Krävs för att Site Recovery övervakning och diagnostik data kan skrivas från den virtuella datorn.

## <a name="outbound-connectivity-for-ip-address-ranges"></a>Utgående anslutning för IP-adressintervall

Om du använder en IP-baserade brandväggen proxy eller NSG-regler för att styra utgående anslutning, måste dessa IP-adressintervall som ska tillåtas.

- Alla IP-adressintervall som motsvarar storage-konton i källan region
    - Skapa en [Storage service taggen](../virtual-network/security-overview.md#service-tags) baserat NSG regeln för käll-region.
    - Tillåt adresserna så att data kan skrivas till cache storage-konto från den virtuella datorn.
- Alla IP-adressintervall som motsvarar Office 365 [autentisering och identitet IP V4-slutpunkter](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity).
    - Om nya adresser läggs till Office 365-adressintervall i framtiden, måste du skapa nya NSG-regler.
- Site Recovery-tjänsten endpoint IP-adresser – tillgängliga i en [XML-filen](https://aka.ms/site-recovery-public-ips) och beror på din målplats.
-  Du kan [hämtar och använder det här skriptet](https://aka.ms/nsg-rule-script), för att automatiskt skapa regler som krävs på NSG: N.
- Vi rekommenderar att du skapar nödvändiga NSG-regler på ett test NSG och kontrollera att det inte finns några problem innan du skapar regler på en NSG för produktion.


Site Recovery IP-adressintervall är följande:

   **mål** | **Site Recovery IP** |  **Site Recovery övervakning IP**
   --- | --- | ---
   Östasien | 52.175.17.132 | 13.94.47.61
   Sydostasien | 52.187.58.193 | 13.76.179.223
   Indien, centrala | 52.172.187.37 | 104.211.98.185
   Södra Indien | 52.172.46.220 | 104.211.224.190
   Norra centrala USA | 23.96.195.247 | 168.62.249.226
   Norra Europa | 40.69.212.238 | 52.169.18.8
   Västra Europa | 52.166.13.64 | 40.68.93.145
   Östra USA | 13.82.88.226 | 104.45.147.24
   Västra USA | 40.83.179.48 | 104.40.26.199
   Södra centrala USA | 13.84.148.14 | 104.210.146.250
   Centrala USA | 40.69.144.231 | 52.165.34.144
   Östra USA 2 | 52.184.158.163 | 40.79.44.59
   Östra Japan | 52.185.150.140 | 138.91.1.105
   Västra Japan | 52.175.146.69 | 138.91.17.38
   Södra Brasilien | 191.234.185.172 | 23.97.97.36
   Östra Australien | 104.210.113.114 | 191.239.64.144
   Sydöstra Australien | 13.70.159.158 | 191.239.160.45
   Centrala Kanada | 52.228.36.192 | 40.85.226.62
   Östra Kanada | 52.229.125.98 | 40.86.225.142
   Västra centrala USA | 52.161.20.168 | 13.78.149.209
   Västra USA 2 | 52.183.45.166 | 13.66.228.204
   Storbritannien, västra | 51.141.3.203 | 51.141.14.113
   Storbritannien, södra | 51.140.43.158 | 51.140.189.52
   Storbritannien, södra 2 | 13.87.37.4| 13.87.34.139
   Storbritannien, norra | 51.142.209.167 | 13.87.102.68
   Centrala Korea | 52.231.28.253 | 52.231.32.85
   Sydkorea | 52.231.298.185 | 52.231.200.144
   Frankrike, centrala | 52.143.138.106 | 52.143.136.55
   Frankrike, södra | 52.136.139.227 |52.136.136.62


## <a name="example-nsg-configuration"></a>NSG exempelkonfiguration

Det här exemplet visar hur du konfigurerar NSG-regler för en virtuell dator för att replikera.

- Om du använder NSG-regler för att styra utgående anslutning kan du använda ”Tillåt utgående HTTPS” regler för port: 443 för alla de nödvändiga IP-adressintervall.
- Exemplet förutsätter att källplatsen VM är ”Öst oss” och målplatsen är ”centrala USA”.

### <a name="nsg-rules---east-us"></a>NSG-regler - östra USA

1. Skapa ett utgående säkerhetsregel HTTPS (443) för ”Storage.EastUS” på NSG: N som visas i skärmbilden nedan.

      ![Storage-tagg](./media/azure-to-azure-about-networking/storage-tag.png)

2. Skapa regler för utgående HTTPS (443) för alla IP-adressintervall som motsvarar Office 365 [autentisering och identitet IP V4-slutpunkter](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity).
3. Skapa regler för utgående HTTPS (443) för Site Recovery IP-adresser som motsvarar målplatsen:

   **Plats** | **Site Recovery-IP-adress** |  **Site Recovery övervakning IP-adress**
    --- | --- | ---
   Centrala USA | 40.69.144.231 | 52.165.34.144

### <a name="nsg-rules---central-us"></a>NSG - regler i centrala USA

Reglerna krävs så att replikering kan aktiveras från målregionen som för den källa region postredundans:

1. Skapa ett utgående säkerhetsregel HTTPS (443) för ”Storage.CentralUS” på NSG: N.

2. Skapa regler för utgående HTTPS (443) för alla IP-adressintervall som motsvarar Office 365 [autentisering och identitet IP V4-slutpunkter](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity).

3. Skapa regler för utgående HTTPS (443) för Site Recovery IP-adresser som motsvarar källplatsen:

   **Plats** | **Site Recovery-IP-adress** |  **Site Recovery övervakning IP-adress**
    --- | --- | ---
   Centrala USA | 13.82.88.226 | 104.45.147.24

## <a name="network-virtual-appliance-configuration"></a>Virtuell installation nätverkskonfiguration

Om du använder virtuella nätverksinstallationer (NVAs) för att styra utgående nätverkstrafik från virtuella datorer kan hämta begränsas anordningen om replikeringstrafiken som passerar genom en NVA. Vi rekommenderar att du skapar en tjänstslutpunkt för nätverk i ditt virtuella nätverk för ”lagring” så att replikeringstrafiken inte går att en NVA.

### <a name="create-network-service-endpoint-for-storage"></a>Skapa nätverk tjänstslutpunkten för lagring
Du kan skapa en tjänstslutpunkt för nätverk i ditt virtuella nätverk för ”lagring” så att replikeringstrafiken inte lämnar Azure gräns.

- Markera din virtuella Azure-nätverket och klicka på Tjänsteslutpunkter

    ![Storage-slutpunkt](./media/azure-to-azure-about-networking/storage-service-endpoint.png)

- Klicka på ”Lägg till' och 'Lägga till Tjänsteslutpunkter” fliken öppnas
- Välj Microsoft.Storage om du under ”tjänst” och nödvändiga undernät under 'undernät-fält och klicka på ”Lägg till'

>[!NOTE]
>Begränsa inte åtkomst till virtuella nätverk till dina lagringskonton som används för automatisk Systemåterställning. Du måste tillåta åtkomst från ”alla nätverk”

### <a name="forced-tunneling"></a>Tvingad tunneltrafik

Du kan åsidosätta Azures system standardvägen för adressprefixet 0.0.0.0/0 med en [anpassade väg](../virtual-network/virtual-networks-udr-overview.md#custom-routes) och minska VM-trafiken till en lokal virtuell nätverksenhet (NVA), men den här konfigurationen rekommenderas inte för Site Recovery replikering. Om du använder anpassade vägar, bör du [skapa ett virtuellt nätverk tjänstslutpunkten](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) i ditt virtuella nätverk för ”lagring” så att replikeringstrafiken inte lämnar Azure gränsen.

## <a name="next-steps"></a>Nästa steg
- Börja skydda dina arbetsbelastningar av [replikering av Azure virtuella datorer](site-recovery-azure-to-azure.md).
- Lär dig mer om [IP-adress kvarhållning](site-recovery-retain-ip-azure-vm-failover.md) för redundans för virtuell dator i Azure.
- Mer information om återställning av [virtuella Azure-datorer med ExpressRoute ](azure-vm-disaster-recovery-with-expressroute.md).
