---
title: Ställ in haveriberedskap för lokala Hyper-V virtuella datorer i VMM-moln till Azure med Azure Site Recovery | Microsoft Docs
description: Lär dig hur du ställer in haveriberedskap för lokala Hyper-V virtuella datorer i System Center VMM-moln till Azure med Azure Site Recovery-tjänsten.
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 05/02/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: a8bbbe0a5aca20222ff7385be9d0ecf0a4224d5c
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2018
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-in-vmm-clouds-to-azure"></a>Ställ in haveriberedskap för lokala Hyper-V virtuella datorer i VMM-moln till Azure

[Azure Site Recovery](site-recovery-overview.md)-tjänsten bidrar till din strategi för haveriberedskap genom att hantera och samordna replikering, redundans och återställning av fysiska servrar och virtuella Azure-datorer.

Den här självstudiekursen visar hur du konfigurerar haveriberedskap för lokala virtuella Hyper-V-datorer till Azure. Kursen gäller för Hyper-V virtuella datorer som hanteras av System Center Virtual Machine Manager (VMM). I den här guiden får du lära dig att:

> [!div class="checklist"]
> * Väljer replikeringskälla och mål.
> * Konfigurerar källmiljön för replikering, inklusive lokala Site Recovery-komponenter och målmiljön för replikeringen.
> * Konfigurera nätverksmappning, för att mappa mellan VMM VM-nätverk och virtuella Azure-nätverk.
> * Skapa replikeringsprincip
> * Aktivera replikering för en virtuell dator

Detta är den tredje självstudien i en serie. Självstudien förutsätter att du redan har slutfört uppgifterna i de föregående självstudierna:

1. [Förbereda Azure](tutorial-prepare-azure.md)
2. [Förbereda lokala Hyper-V](tutorial-prepare-on-premises-hyper-v.md)

Innan du börjar är det bra att [granska arkitekturen](concepts-hyper-v-to-azure-architecture.md) för detta haveriberedskapsscenario.



## <a name="select-a-replication-goal"></a>Välj ett replikeringsmål

1. I **alla tjänster** > **Recovery Services-valv**, klicka på valvnamnet vi använder i de här kurserna **ContosoVMVault**.
2. I **Komma igång** klickar du på **Site Recovery**. Klicka sedan på **Förbered infrastrukturen**.
3. I **Skyddsmål** > **Var finns dina datorer?** väljer du **Lokalt**.
4. I **Till vilken plats ska dina datorer replikeras?** väljer du **Till Azure**.
5. I **är dina datorer virtualiserade**väljer **Ja, med Hyper-V**.
6. I **använder du System Center VMM**väljer **Ja**. Klicka sedan på **OK**.

    ![Replikeringsmål](./media/hyper-v-vmm-azure-tutorial/replication-goal.png)



## <a name="set-up-the-source-environment"></a>Konfigurera källmiljön

När du ställer in källmiljön kan du installera Azure Site Recovery-providern och Azure Recovery Services-agenten och registrera lokala servrar i valvet. 

1. Klicka på **Källa** i **Förbereda infrastrukturen**.
2. I **Förbered källa** klickar du på **+ VMM** för att lägga till en VMM-server. I **Lägg till Server**, kontrollera att **System Center VMM-servern** visas i **servertyp**.
3. Hämta installationsprogrammet för Microsoft Azure Site Recovery-providern.
4. Ladda ned valvregistreringsnyckeln. Du behöver den när du kör installationsprogrammet för providern. Nyckeln är giltig i fem dagar efter att du har genererat den.
5. Hämta Recovery Services-agenten.

    ![Ladda ned](./media/hyper-v-vmm-azure-tutorial/download-vmm.png)

### <a name="install-the-provider-on-the-vmm-server"></a>Installera providern på VMM-servern

1. I installationsguiden för Azure Site Recovery-providern > **Microsoft Update** väljer du att använda Microsoft Update för att söka efter uppdateringar för providern.
2. I **Installation**accepterar standardinstallationsplatsen för providern och klickar på **installera**. 
3. Efter installationen i Registreringsguiden för Microsoft Azure Site Recovery > **Valvinställningar**, klickar du på **Bläddra**, och i **nyckelfilen**väljer valvnyckeln fil som du laddas ned.
4. Ange Azure Site Recovery-prenumerationen och valvnamnet (**ContosoVMVault**). Ange ett eget namn för VMM-servern att identifiera den i valvet.
5. I **Proxyinställningar** väljer du **Anslut direkt till Azure Site Recovery utan proxyserver**.
6. Acceptera standardplatsen för det certifikat som används för att kryptera data. Krypterade data dekrypteras när du redundansväxlar.
7. I **synkronisera molnmetadata**väljer **synka molnmetadata till Site Recovery-portalen**. Den här åtgärden behöver bara göras en gång på varje server. Klicka på **registrera**.
8. När servern är registrerad i valvet, klickar du på **Slutför**.

När registreringen är klar, hämtas metadata från servern av Azure Site Recovery och VMM-servern visas i **Site Recovery-infrastruktur**.

### <a name="install-the-recovery-services-agent"></a>Installera Recovery Services-agenten

Installera agenten på varje Hyper-V-värden som innehåller virtuella datorer du vill replikera.

1. I installationsguiden för Microsoft Azure Recovery Services-agenten > **Kravkontroll**, klickar du på **nästa**. Eventuella nödvändiga komponenter som saknas installeras automatiskt.
2. I **installationsinställningar**, acceptera installationsplatsen och cachelagringsplatsen. Cacheenhet behöver minst 5 GB lagringsutrymme. Vi rekommenderar en enhet med 600 GB eller mer ledigt utrymme. Klicka på **Installera**.
3. I **Installation**, när installationen är klar, klickar du på **Stäng** och slutföra guiden.

    ![Installera agent](./media/hyper-v-vmm-azure-tutorial/mars-install.png)
    

## <a name="set-up-the-target-environment"></a>Konfigurera målmiljön

1. Klicka på **Förbered infrastrukturen** > **Mål**.
2. Välj prenumerationen och resursgruppen (**ContosoRG**) i som virtuella Azure-datorer skapas efter växling vid fel.
3. Välj **Resource Manager**-distributionsmodellen.

Site Recovery kontrollerar att du har ett eller flera kompatibla Azure-lagringskonton och Azure-nätverk.


## <a name="configure-network-mapping"></a>Konfigurera nätverksmappning

1. Under **Site Recovery-infrastruktur** > **Nätverksmappningar** > **Nätverksmappning** klickar du på ikonen **+Nätverksmappning**.
2. I **Lägg till nätverksmappning**, Välj VMM-källservern. Välj **Azure** som mål.
3. Kontrollera prenumerationen och distributionsmodellen efter redundansväxling.
4. I **Källnätverk**, Välj källnätverket lokalt.
5. I **målnätverket**, Välj Azure-nätverk i vilka Azure-Replikdatorerna kommer att finnas när de skapas efter växling vid fel. Klicka sedan på **OK**.

    ![Nätverksmappning](./media/hyper-v-vmm-azure-tutorial/network-mapping-vmm.png)

## <a name="set-up-a-replication-policy"></a>Konfigurerar en replikeringsprincip

1. Klicka på **Förbered infrastruktur** > **Replikeringsinställningar** > **+Skapa och koppla**.
2. I **Princip för att skapa och koppla** anger du principnamnet **ContosoReplicationPolicy**.
3. Låt standardvärdet stå kvar och klicka på **OK**.
    - **Kopieringsfrekvens** anger att deltadata (efter den inledande replikeringen) replikeras var femte minut.
    - **Kvarhållning av återställningspunkt** anger att kvarhållningsfönstret för varje återställningspunkt är två timmar.
    - **Frekvens för appkonsekvent ögonblicksbild** anger att återställningspunkterna som innehåller appkonsekventa ögonblicksbilder skapas varje timme.
    - **Starttid för inledande replikering** anger att den inledande replikeringen ska börja direkt.
    - **Kryptera data lagrade på Azure** -standard **av** innebär att vilande data i Azure inte är krypterad.
4. Klicka på **OK** när principen har skapats. När du skapar en ny princip associeras den automatiskt med VMM-molnet.

## <a name="enable-replication"></a>Aktivera replikering

1. Klicka på **Källa** i **Replikera program**. 
2. I **källa**, Välj VMM-moln. Klicka sedan på **OK**.
3. I **mål**kontrollerar Azure som mål, valvet-prenumerationen och välj den **Resource Manager** modell.
4. Välj den **contosovmsacct1910171607** storage-konto och **ContosoASRnet** Azure-nätverk.
5. I **Virtuella datorer** > **Välj** väljer du den virtuella dator som du vill replikera. Klicka sedan på **OK**.

 Du kan följa förloppet för åtgärden **Aktivera skydd** under **Jobb** > **Site Recovery-jobb**. Efter den **Slutför skydd** jobbet är slutfört den inledande replikeringen är klar och den virtuella datorn är redo för redundans.


## <a name="next-steps"></a>Nästa steg
[Köra ett återställningstest](tutorial-dr-drill-azure.md)
