---
title: Begränsa nätverksåtkomst till PaaS-resurser – självstudie – Azure portal | Microsoft Docs
description: I den här självstudien får du lära dig att begränsa nätverksåtkomst till Azure-resurser, som Azure Storage och Azure SQL Database, med virtuella nätverksslutpunkter med Azure-portalen.
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: virtual-networ
ms.workload: infrastructure
ms.date: 03/14/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: f53544e756bde623a604513f17f9cc92c8efe42b
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/05/2018
---
# <a name="tutorial-restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-portal"></a>Självstudie: Begränsa nätverksåtkomst till PaaS-resurser med virtuella nätverksslutpunkter med Azure-portalen

Med virtuella nätverksslutpunkter kan du begränsa nätverksåtkomsten till vissa Azure-tjänsters resurser till ett undernät för virtuella datorer. Du kan också ta bort resursernas internetåtkomst. Tjänstslutpunkterna möjliggör direktanslutning från ditt virtuella nätverk till Azure-tjänster som stöds, så att du kan använda det privata adressutrymmet i det virtuella nätverket för åtkomst till Azure-tjänsterna. Trafik till Azure-resurser genom tjänstslutpunkterna finns alltid kvar i Microsoft Azure-stamnätverket. I den här guiden får du lära dig att:

> [!div class="checklist"]
> * Skapa ett virtuellt nätverk med ett undernät
> * Lägga till ett undernät och aktivera en tjänstslutpunkt
> * Skapa en Azure-resurs och tillåt nätverksåtkomst till den från enbart ett undernät
> * Distribuera en virtuell dator (VM) till varje undernät
> * Bekräfta åtkomst till en resurs från ett undernät
> * Bekräfta att åtkomst nekas för en resurs från ett undernät och internet

Om du vill kan du slutföra den här självstudien med [Azure CLI](tutorial-restrict-network-access-to-resources-cli.md) eller [Azure PowerShell](tutorial-restrict-network-access-to-resources-powershell.md).

Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar.

## <a name="log-in-to-azure"></a>Logga in på Azure 

Logga in på Azure Portal på http://portal.azure.com.

## <a name="create-a-virtual-network"></a>Skapa ett virtuellt nätverk

1. Klicka på **+ Skapa en resurs** längst upp till vänster på Azure Portal.
2. Välj **Nätverk** och välj därefter **Virtuellt nätverk**.
3. Välj eller ange följande information och välj **Skapa**:

    |Inställning|Värde|
    |----|----|
    |Namn| myVirtualNetwork |
    |Adressutrymme| 10.0.0.0/16|
    |Prenumeration| Välj din prenumeration|
    |Resursgrupp | Välj **Skapa ny** och skriv *myResourceGroup*.|
    |Plats| Välj **USA, östra** |
    |Namn på undernät| Offentligt|
    |Adressintervall för undernät| 10.0.0.0/24|
    |Tjänstslutpunkter| Disabled|

    ![Ange grundläggande information om ditt virtuella nätverk](./media/tutorial-restrict-network-access-to-resources/create-virtual-network.png)


## <a name="enable-a-service-endpoint"></a>Aktivera en tjänstslutpunkt

1. I rutan **Sök efter resurser, tjänster och dokument** högst upp i portalen skriver du *myVirtualNetwork.* När **myVirtualNetwork** visas i sökresultatet väljer du det.
2. Lägg till ett undernät i det virtuella nätverket. Under **INSTÄLLNINGAR** väljer du **Undernät** och väljer sedan **+ Undernät** som du ser på följande bild:

    ![Lägga till undernät](./media/tutorial-restrict-network-access-to-resources/add-subnet.png) 

3. Under **Lägg till undernät** väljer du eller anger följande information och väljer sedan **OK**:

    |Inställning|Värde|
    |----|----|
    |Namn| Privat |
    |Adressintervall| 10.0.1.0/24|
    |Tjänstslutpunkter| Välj **Microsoft.Storage** under **Tjänster**|

## <a name="restrict-network-access-for-a-subnet"></a>Begränsa nätverksåtkomst för ett undernät

1. Klicka på **+ Skapa en resurs** längst upp till vänster på Azure Portal.
2. Välj **Nätverk** och sedan **Nätverkssäkerhetsgrupp**.
Under **Skapa en nätverkssäkerhetsgrupp** anger eller väljer du följande information och väljer sedan **Skapa**:

    |Inställning|Värde|
    |----|----|
    |Namn| myNsgPrivate |
    |Prenumeration| Välj din prenumeration|
    |Resursgrupp | Välj **Använd befintlig** och sedan *myResourceGroup*.|
    |Plats| Välj **USA, östra** |

4. När nätverkssäkerhetsgruppen har skapats anger du *myNsgPrivate* i rutan **Sök efter resurser, tjänster och dokument** överst i portalen. När **myNsgPrivate** visas i sökresultaten väljer du det.
5. Under **INSTÄLLNINGAR** väljer du **Utgående säkerhetsregler**.
6. Välj **+ Lägg till**.
7. Skapa en regel som tillåter utgående åtkomst till de offentliga IP-adresser som tilldelats till Azure Storage-tjänsten. Välj eller ange följande information och välj **OK**:

    |Inställning|Värde|
    |----|----|
    |Källa| Välj **VirtualNetwork** |
    |Källportintervall| * |
    |Mål | Välj **Service Tag** (tjänsttagg)|
    |Måltjänsttagg | Välj **Lagring**|
    |Målportintervall| * |
    |Protokoll|Alla|
    |Åtgärd|Tillåt|
    |Prioritet|100|
    |Namn|Allow-Storage-All|
8. Skapa en regel som åsidosätter en standardsäkerhetsregel som tillåter åtkomst till alla offentliga IP-adresser. Slutför steg 6 och 7 igen med följande värden:

    |Inställning|Värde|
    |----|----|
    |Källa| Välj **VirtualNetwork** |
    |Källportintervall| * |
    |Mål | Välj **Service Tag** (tjänsttagg)|
    |Måltjänsttagg| Välj **Internet**|
    |Målportintervall| * |
    |Protokoll|Alla|
    |Åtgärd|Neka|
    |Prioritet|110|
    |Namn|Deny-Internet-All|

9. Under **INSTÄLLNINGAR** väljer du **Inkommande säkerhetsregel**.
10. Välj **+ Lägg till**.
11. Skapa en regel som tillåter ingående RDP-trafik (Remote Desktop Protocol) till undernätet varifrån som helst. Regeln åsidosätter en standardsäkerhetsregel som nekar all inkommande trafik från internet. Fjärrskrivbordsanslutningar tillåts i undernätet så att anslutningen kan testas i ett senare steg. Slutför steg 6 och 7 igen med följande värden:

    |Inställning|Värde|
    |----|----|
    |Källa| Alla |
    |Källportintervall| * |
    |Mål | Välj **Service Tag** (tjänsttagg)|
    |Måltjänsttagg| Välj **VirtualNetwork**|
    |Målportintervall| 3389 |
    |Protokoll|Alla|
    |Åtgärd|Tillåt|
    |Prioritet|120|
    |Namn|Allow-RDP-All|

12. Under **INSTÄLLNINGAR** väljer du **Undernät**.
13. Välj **+ Koppla**
14. Under **Koppla undernätverk** väljer du **Virtuellt nätverk** och sedan **myVirtualNetwork** under **Välj ett virtuellt nätverk**.
15. Under **Välj undernät** väljer du **Privat** och sedan **OK**.

## <a name="restrict-network-access-to-a-resource"></a>Begränsa nätverksåtkomst till en resurs

De steg som behövs för att begränsa nätverksåtkomsten till resurser som har skapats via Azure-tjänster som är aktiverade för tjänstslutpunkter varierar från tjänst till tjänst. Läs dokumentationen för enskilda tjänster för specifika åtgärder för varje tjänst. Resten av den här självstudiekursen innehåller steg för att begränsa nätverksåtkomsten för ett Azure Storage-konto, som ett exempel.

### <a name="create-a-storage-account"></a>skapar ett lagringskonto

1. Klicka på **+ Skapa en resurs** längst upp till vänster på Azure Portal.
2. Välj **Lagring** och sedan **Koppla undernät – blob, fil, tabell, kö**.
3. Ange eller välj följande information, acceptera standardinställningarna för återstående inställningar och välj sedan **Skapa**:

    |Inställning|Värde|
    |----|----|
    |Namn| Ange ett namn som är unikt i alla Azure-platser, mellan 3–24 tecken långt och som endast innehåller siffror och gemener.|
    |Typ av konto|StorageV2 (generell användning v2)|
    |Replikering| Lokalt redundant lagring (LRS)|
    |Prenumeration| Välj din prenumeration|
    |Resursgrupp | Välj **Använd befintlig** och sedan *myResourceGroup*.|
    |Plats| Välj **USA, östra** |

### <a name="create-a-file-share-in-the-storage-account"></a>Skapa en filresurs i lagringskontot

1. När lagringskontot är skapat anger du namnet på det i rutan **Sök efter resurser, tjänster och dokument** högst upp i portalen. När namnet på ditt lagringskonto visas i sökresultatet väljer du det.
2. Välj **Filer** enligt bilden nedan:

    ![Lagringskonto](./media/tutorial-restrict-network-access-to-resources/storage-account.png) 
3. Välj **+ Filresurs** under **Filtjänst**.
4. Skriv *my-file-share* under **Namn** och välj **OK**.
5. Stäng rutan **Filtjänst**.

### <a name="enable-network-access-from-a-subnet"></a>Aktivera nätverksåtkomst från ett undernät

Som standard godkänner lagringskonton nätverksanslutningar från klienter i alla nätverk. Om du vill tillåta åtkomst från endast ett specifikt undernät och neka åtkomst från alla andra nätverk utför du följande steg:

1. Under **INSTÄLLNINGAR** för lagringskontot väljer du **Brandväggar och virtuella nätverk**.
2. Under **Virtuella nätverk** väljer du **Valda nätverk**.
3. Välj **Lägg till befintligt virtuellt nätverk**.
4. Under **Lägg till nätverk** väljer du följande värden och sedan **Lägg till**:

    |Inställning|Värde|
    |----|----|
    |Prenumeration| Välj din prenumeration.|
    |Virtuella nätverk|Välj **myVirtualNetwork** under **Virtuella nätverk**|
    |Undernät| Välj **Privat** under **Undernät**|

    ![Brandväggar och virtuella nätverk](./media/tutorial-restrict-network-access-to-resources/storage-firewalls-and-virtual-networks.png) 

5. Välj **Spara**.
6. Stäng rutan **Brandväggar och virtuella nätverk**.
7. Under **INSTÄLLNINGAR** för lagringskontot väljer du **Åtkomstnycklar** enligt följande bild:

      ![Brandväggar och virtuella nätverk](./media/tutorial-restrict-network-access-to-resources/storage-access-key.png)

8. Lägg märke till **nyckelvärdet** eftersom du måste mata in det manuellt i ett senare steg när du mappar filresursen till en enhetsbeteckning i en virtuell dator.

## <a name="create-virtual-machines"></a>Skapa virtuella datorer

Om du vill testa nätverksåtkomsten till ett lagringskonto distribuerar du en virtuell dator till varje undernät.

### <a name="create-the-first-virtual-machine"></a>Skapa din första virtuella dator

1. Klicka på **+ Skapa en resurs** längst upp till vänster på Azure Portal.
2. Välj **Compute**, och välj sedan **Windows Server 2016 Datacenter**.
3. Välj eller ange följande information och välj **OK**:

    |Inställning|Värde|
    |----|----|
    |Namn| myVmPublic|
    |Användarnamn|Ange ett valfritt användarnamn.|
    |Lösenord| Ange ett valfritt lösenord. Lösenordet måste vara minst 12 tecken långt och uppfylla [de definierade kraven på komplexitet](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Prenumeration| Välj din prenumeration.|
    |Resursgrupp| Välj **Använd befintlig** och sedan **myResourceGroup**.|
    |Plats| Välj **USA, östra**.|

    ![Ange grundläggande information om en virtuell dator](./media/tutorial-restrict-network-access-to-resources/virtual-machine-basics.png)
4. Välj en storlek för den virtuella datorn och sedan **Välj**.
5. Under **Inställningar** väljer du **Nätverk** och sedan **myVirtualNetwork**. Välj sedan **Undernät** och därefter **Offentligt** enligt följande bild:

    ![Välja ett virtuellt nätverk](./media/tutorial-restrict-network-access-to-resources/virtual-machine-settings.png)
6. På **sammanfattningssidan** klickar du på **Skapa** för att starta distributionen av den virtuella datorn. Det tar några minuter att distribuera den virtuella datorn, men du kan fortsätta till nästa steg medan den virtuella datorn skapas.

### <a name="create-the-second-virtual-machine"></a>Skapa den andra virtuella datorn

Slutför steg 1–6 igen, men i steg 3 ska du ge den virtuella datorn namnet *myVmPrivate*, och i steg 5 väljer du undernätet **Privat**.

Det tar några minuter att distribuera den virtuella datorn. Fortsätt inte till nästa steg förrän skapandet är klart och dess inställningar öppnas i portalen.

## <a name="confirm-access-to-storage-account"></a>Bekräfta åtkomst till lagringskontot

1. När den virtuella datorn *myVmPrivate* har skapats öppnar Azure dess inställningar. Anslut till den virtuella datorn genom att välja knappen **Anslut** enligt följande bild:

    ![Ansluta till en virtuell dator](./media/tutorial-restrict-network-access-to-resources/connect-to-virtual-machine.png)

2. När du har valt knappen **Anslut** skapas en .rdp-fil (Remote Desktop Protocol) som laddas ned till datorn.  
3. Öppna den nedladdade rdp-filen. Om det händer väljer du **Anslut**. Ange användarnamnet och lösenordet du angav när du skapade den virtuella datorn. Du kan behöva välja **Fler alternativ** och sedan **Använd ett annat konto** för att ange autentiseringsuppgifterna du angav när du skapade den virtuella datorn. 
4. Välj **OK**.
5. Du kan få en certifikatvarning under inloggningen. Om du ser varningen väljer du **Ja** eller **Fortsätt** för att fortsätta med anslutningen.
6. På den virtuella datorn *myVmPrivate* mappar du Azure-fildelningen till enhet Z med PowerShell. Innan du kör kommandona som följer ska du ersätta `<storage-account-key>` och `<storage-account-name>` med värden som du angav och hämtade i [Skapa ett lagringskonto](#create-a-storage-account).

    ```powershell
    $acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
    New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
    ```
    
    PowerShell returnerar utdata som liknar följande exempelutdata:

    ```powershell
    Name           Used (GB)     Free (GB) Provider      Root
    ----           ---------     --------- --------      ----
    Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
    ```

    Azure-fildelningen har mappats till enhet Z.

7. Bekräfta att de virtuella datorerna inte har någon utgående anslutning till några andra offentliga IP-adresser från en kommandotolk:

    ```
    ping bing.com
    ```
    
    Du får inga svar eftersom nätverkssäkerhetsgruppen som är kopplad till det *privata* undernätet inte tillåter utgående åtkomst till offentliga IP-adresser förutom de adresser som är kopplade till Azure Storage-tjänsten.

8. Stäng fjärrskrivbordssessionen för den virtuella datorn *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Bekräfta att åtkomst till lagringskontot nekas

1. I rutan **Sök efter resurser, tjänster och dokument** högst upp i portalen skriver du *myVmPublic*.
2. När **myVmPublic** visas i sökresultaten väljer du det.
3. Slutför steg 1–6 i [Bekräfta åtkomst till lagringskontot](#confirm-access-to-storage-account) för den virtuella datorn *myVmPublic*.

    Åtkomst nekas och du ser felet `New-PSDrive : Access is denied`. Åtkomst nekas eftersom den virtuella datorn *myVmPublic* distribueras i det *offentliga* undernätet. Det *offentliga* undernätet har ingen tjänstslutpunkt aktiverad för Azure Storage, och lagringskontot tillåter endast nätverksåtkomst från det *privata* undernätet, inte det *offentliga*.

4. Stäng fjärrskrivbordssessionen för den virtuella datorn *myVmPublic*.

5. Från datorn bläddrar du till Azure [Portal](https://portal.azure.com).
6. Ange namnet på lagringskontot du skapade i rutan **Sök efter resurser, tjänster och dokument**. När namnet på ditt lagringskonto visas i sökresultatet väljer du det.
7. Välj **Filer**.
8. Du ser felet som visas i följande bild:

    ![Fel för nekad åtkomst](./media/tutorial-restrict-network-access-to-resources/access-denied-error.png)

    Åtkomsten nekas eftersom din dator inte är i det *privata* undernätet i det virtuella nätverket för *MyVirtualNetwork*.

## <a name="clean-up-resources"></a>Rensa resurser

Ta bort resursgruppen, skalningsuppsättningen och alla resurser som den innehåller:

1. Skriv *myResourceGroup* i **sökrutan** överst i portalen. När du ser **myResourceGroup** i sökresultatet väljer du det.
2. Välj **Ta bort resursgrupp**.
3. Skriv *myResourceGroup* där du uppmanas att **skriva resursgruppens namn:** (Skriv resursgruppens namn) och välj **Ta bort**.

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du aktiverat en tjänstslutpunkt för ett undernät för ett virtuellt nätverk. Du har lärt dig att tjänstslutpunkter kan aktiveras för resurser som distribueras med flera Azure-tjänster. Du har skapat ett Azure Storage-konto och begränsat nätverksåtkomst för lagringskontot till enbart resurser inom ett undernät för ett virtuellt nätverk. Om du vill veta mer om tjänstslutpunkter går du till [Översikt över tjänstslutpunkter](virtual-network-service-endpoints-overview.md) och [Hantera undernät](virtual-network-manage-subnet.md).

Om du har flera virtuella nätverk i ditt konto kanske du vill ansluta två virtuella nätverk så att resurserna i vart och ett kan kommunicera med varandra. Om du vill lära dig mer om att ansluta virtuella nätverk går du vidare till nästa självstudie.

> [!div class="nextstepaction"]
> [Ansluta virtuella nätverk](./tutorial-connect-virtual-networks-portal.md)