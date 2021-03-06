---
title: 'Självstudier: Skapa en offentlig enkel belastningsutjämnare – Azure Portal | Microsoft Docs'
description: Den här självstudiekursen beskriver hur du skapar en intern, enkel belastningsutjämnare med hjälp av Azure Portal.
services: load-balancer
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines within a specific zone in a region.
ms.assetid: aa9d26ca-3d8a-4a99-83b7-c410dd20b9d0
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: kumud
ms.custom: mvc
ms.openlocfilehash: a16e9ad5b72d87614f5d3630e24e6aa36def8c51
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/03/2018
ms.locfileid: "32779581"
---
# <a name="tutorial-load-balance-internal-traffic-with-basic-load-balancer-to-vms-using-the-azure-portal"></a>Självstudier: Belastningsutjämna intern trafik till virtuella datorer med en enkel belastningsutjämnare med hjälp av Azure Portal

Med belastningsutjämning får du högre tillgänglighet och skala genom att inkommande förfrågningar sprids över flera virtuella datorer. Du kan använda Azure Portal för att belastningsutjämna intern trafik till virtuella datorer med en enkel belastningsutjämnare. Den här kursen beskriver hur du skapar nätverksresurser, backend-servrar och en enkel intern belastningsutjämnare.

Om du vill kan du slutföra den här självstudien med [Azure CLI](load-balancer-get-started-ilb-arm-cli.md) eller [Azure PowerShell](load-balancer-get-started-ilb-arm-ps.md).

Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar. 

## <a name="sign-in-to-the-azure-portal"></a>Logga in på Azure Portal

Logga in på Azure Portal på [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-virtual-network"></a>Skapa ett virtuellt nätverk
1. Klicka på **Nytt** > **Nätverk** > **Virtuella nätverk** överst till vänster på skärmen och ange följande värden för det virtuella nätverket:
    - *myVnet* – för det virtuella nätverkets namn.
    - *myResourceGroupILB* – namnet på den befintliga resursgruppen
    - *myBackendSubnet* – för undernätsnamnet.
2. Skapa det virtuella nätverket genom att klicka på **Skapa**.

## <a name="create-a-basic-load-balancer"></a>Skapa en grundläggande belastningsutjämnare
Skapa en intern enkel belastningsutjämnare med hjälp av portalen.

1. Längst upp till vänster på skärmen klickar du på **Skapa en resurs** > **Nätverk** > **Belastningsutjämnare**.
2. Ange dessa värden för belastningsutjämnaren på sidan **Skapa en belastningsutjämnare**:
    - *myLoadBalancer* – namnet på belastningsutjämnaren.
    - **Internal** – typen av belastningsutjämnare.
    - **Basic** – SKU-versionen.
    - **10.1.0.7** – den statiska privata IP-adressen.
    - *myVNet* – det virtuella nätverk som du väljer i listan över befintliga nätverk.
    - *mySubnet* – det undernät som du väljer från listan med befintliga undernät.
    - *myResourceGroupILB* – namnet på den nya resursgrupp som du skapar.
3. Skapa belastningsutjämnaren genom att klicka på **Skapa**.
   
    ![Skapa en belastningsutjämnare](./media/tutorial-load-balancer-basic-internal-portal/1-load-balancer.png)


## <a name="create-backend-servers"></a>Skapa serverdelsservrar

I det här avsnittet skapar du två virtuella datorer för belastningsutjämnarens backend-pool och installerar sedan IIS på de virtuella datorerna. IIS gör det lättare att testa belastningsutjämnaren.

### <a name="create-virtual-machines"></a>Skapa virtuella datorer

1. Klicka på **Skapa en resurs** > **Compute** > **Windows Server 2016 Datacenter** överst till vänster på skärmen och ange följande värden för den virtuella datorn:
    - *myVM1* – för den virtuella datorns namn.        
    - *azureuser* – för administratörens användarnamn.   
    - *myResourceGroupILB* – för **Resursgrupp**väljer du **Använd befintlig** och väljer sedan *myResourceGroupILB*.
2. Klicka på **OK**.
3. Välj **DS1_V2** som storlek på den virtuella datorn och klicka på **Välj**.
4. Ange dessa värden för VM-inställningarna:
    - *myAvailabilitySet* – för namnet på den nya tillgänglighetsuppsättning som du skapar.
    -  *myVNet* – se till att detta har markerats som det virtuella nätverket.
    - *myBackendSubnet* – kontrollera att det har markerats som undernätet.
    - *myNetworkSecurityGroup* – namnet på den nya nätverkssäkerhetsgrupp (brandvägg) som du måste skapa.
5. Inaktivera startdiagnostikinställningar genom att klicka på **Inaktiverad**.
6. Klicka på **OK**, granska inställningarna på sammanfattningssidan och klicka sedan på **Skapa**.
7. Skapa, med hjälp av steg 1-6, en andra virtuell dator med namnet *VM2* med *myAvailabilityset* som tillgänglighetsuppsättning, *myVnet* som virtuellt nätverk, *myBackendSubnet* som undernät och *myNetworkSecurityGroup* som dess nätverkssäkerhetsgrupp. 

### <a name="install-iis-and-customize-the-default-web-page"></a>Installera IIS och anpassa standardwebbsidan

1. Klicka på **Alla resurser** på den vänstra menyn och klicka sedan i resurslistan på **myVM1** som finns i resursgruppen *myResourceGroupILB*.
2. Klicka på **Anslut** på sidan **Översikt** och anslut RDP till den virtuella datorn.
3. Logga in på den virtuella datorn.
4. Navigera till **Windows Administrationsverktyg**>**Serverhanteraren** på serverdatorn.
5. Starta Windows PowerShell på VM1 och kör följande kommandon för att installera IIS-servern och uppdatera standard-HTML-filen.
    ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```
5. Stäng RDP-anslutningen med *myVM1*.
6. Upprepa steg 1–5 med *myVM2* för att installera IIS och anpassa standardwebbsidan.

## <a name="create-nsg-rules"></a>Skapa nätverkssäkerhetsgruppsregler

I det här avsnittet skapar du nätverkssäkerhetsgruppsregler som tillåter att inkommande anslutningar använder HTTP och RDP.

1. Klicka på **Alla resurser** på den vänstra menyn och klicka sedan i resurslistan på **myNetworkSecurityGroup** som finns i resursgruppen **myResourceGroupLB**.
2. Klicka på **Ingående säkerhetsregler** under **Inställningar** och klicka sedan på **Lägg till**.
3. Ange dessa värden för den ingående säkerhetsregeln *myHTTPRule* så att inkommande HTTP-anslutningar som använder port 80 tillåts:
    - *Tjänstetagg* – för **Källa**.
    - *Internet* – för **Källtjänsttagg**
    - *80* – för **målportsintervall**
    - *TCP* – för **Protokoll**
    - *Tillåt* – för **Åtgärd**
    - *100* för **Prioritet**
    - *myHTTPRule* för namn
    - *Tillåt HTTP* – för beskrivning
4. Klicka på **OK**.
 
5. Skapa, genom att upprepa steg 2 till 4, en annan regel med namnet *myRDPRule* som tillåter att en inkommande RDP-anslutning använder port 3389 med följande värden:
    - *Tjänstetagg* – för **Källa**.
    - *Internet* – för **Källtjänsttagg**
    - *3389* – för **målportsintervall**
    - *TCP* – för **Protokoll**
    - *Tillåt* – för **Åtgärd**
    - *200* för **Prioritet**
    - *myRDPRule* för namn
    - *Tillåt HTTP* – för beskrivning

## <a name="create-basic-load-balancer-resources"></a>Skapa en grundläggande belastningshanterare

I det här avsnittet ska du konfigurera belastningsutjämningsinställningarna för en serverdelsadresspool och en hälsoavsökning, och ange belastningsutjämnare och NAT-regler.


### <a name="create-a-backend-address-pool"></a>Skapa en serverdelsadresspool

För att distribuera trafik till de virtuella datorerna finns en adresspool på serverdelen som innehåller IP-adresserna för de virtuella nätverkskort som är anslutna till belastningsutjämnaren. Skapa serverdelsadresspoolen *myBackendPool* så att den omfattar *VM1* och *VM2*.

1. Klicka på **Alla resurser** i den vänstra menyn och klicka sedan på **myLoadBalancer** i resurslistan.
2. Klicka på **Serverdelspooler** under **Inställningar** och klicka sedan på **Lägg till**.
3. Gör följande på sidan **Lägg till en serverdelspool**:
    - Ange * myBackEndPool som namn på serverdelspoolen.
    - Klicka på **Tillgänglighetsuppsättning** för **Associerad till** i den nedrullningsbara menyn
    - Klicka på **myAvailabilitySet** för **Tillgänglighetsuppsättning**.
    - Klicka på **Lägg till en målnätverks-IP-konfiguration** och lägg till varje virtuell dator (*myVM1* & *myVM2*) som du skapade till serverdelspoolen.
    - Klicka på **OK**.

        ![Lägga till i backend-adresspoolen (serverdelspoolen) ](./media/tutorial-load-balancer-basic-internal-portal/3-load-balancer-backend-02.png)

3. Kontrollera att inställningen för serverdelspoolens belastningsutjämnare visar båda de virtuella datorerna **VM1** och **VM2**.

### <a name="create-a-health-probe"></a>Skapa en hälsoavsökning

Om du vill att den grundläggande belastningsutjämnaren ska övervaka status för din app kan du använda en hälsoavsökning. Hälsoavsökningen lägger till eller tar bort virtuella datorer dynamiskt från belastningsutjämnarens rotation baserat på deras svar på hälsokontroller. Skapa en hälsoavsökning *myHealthProbe* så att du kan övervaka de virtuella datorernas hälsotillstånd.

1. Klicka på **Alla resurser** i den vänstra menyn och klicka sedan på **myLoadBalancer** i resurslistan.
2. Klicka på **Hälsoavsökningar** under **Inställningar** och klicka sedan på **Lägg till**.
3. Använd följande värden när du skapar hälsoavsökningen:
    - *myHealthProbe* – för hälsoavsökningens namn.
    - **HTTP** – för protokolltypen.
    - *80* – för portnumret.
    - *15* – för antalet **intervall** i sekunder mellan avsökningsförsöken.
    - *2* – för antalet **tröskelvärden för ohälsosamt värde** eller antalet avsökningsfel i följd som måste inträffa innan en virtuell dator anses vara felaktig.
4. Klicka på **OK**.

   ![Lägga till en avsökning](./media/tutorial-load-balancer-basic-internal-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Skapa en belastningsutjämningsregel

En belastningsutjämningsregel används för att definiera hur trafiken ska distribueras till de virtuella datorerna. Du definierar IP-konfigurationen på klientdelen för inkommande trafik och IP-poolen på serverdelen för att ta emot trafik samt nödvändig käll- och målport. Skapa belastningsutjämningsregeln *myLoadBalancerRuleWeb* så att du kan lyssna på port 80 i klientdelen *LoadBalancerFrontEnd* och skicka belastningsutjämnad nätverkstrafik till serverdelsadresspoolen *myBackEndPool* som också använder port 80. 

1. Klicka på **Alla resurser** i den vänstra menyn och klicka sedan på **myLoadBalancer** i resurslistan.
2. Klicka på **Belastningsutjämningsregler** under **Inställningar** och klicka sedan på **Lägg till**.
3. Använd följande värden när du konfigurerar belastningsutjämningsregeln:
    - *myHTTPRule* – för belastningsutjämningsregelns namn.
    - **TCP** – för protokolltypen.
    - *80* – för portnumret.
    - *80* – för serverdelsporten.
    - *myBackendPool* – för serverdelspoolens namn.
    - *myHealthProbe* – för hälsoavsökningens namn.
4. Klicka på **OK**.
    
    ![Lägga till en belastningsutjämningsregel](./media/tutorial-load-balancer-basic-internal-portal/5-load-balancing-rules.png)

## <a name="create-a-virtual-machine-to-test-the-load-balancer"></a>Skapa en virtuell dator för att testa belastningsutjämnaren
För att testa den interna belastningsutjämnaren måste du skapa en virtuell dator som finns i samma virtuella nätverk som de virtuella datorerna på backend-servern.
1. Klicka på **Skapa en resurs** > **Compute** > **Windows Server 2016 Datacenter** överst till vänster på skärmen och ange följande värden för den virtuella datorn:
    - *myVMTest* – den virtuella datorns namn.        
    - *myResourceGroupILB* – för **Resursgrupp**väljer du **Använd befintlig** och väljer sedan *myResourceGroupILB*.
2. Klicka på **OK**.
3. Välj **DS1_V2** som storlek på den virtuella datorn och klicka på **Välj**.
4. Ange dessa värden för VM-inställningarna:
    -  *myVNet* – se till att detta har markerats som det virtuella nätverket.
    - *myBackendSubnet* – kontrollera att det har markerats som undernätet.
5. Inaktivera startdiagnostikinställningar genom att klicka på **Inaktiverad**.
6. Klicka på **OK**, granska inställningarna på sammanfattningssidan och klicka sedan på **Skapa**.

## <a name="test-the-load-balancer"></a>Testa belastningsutjämnaren
1. På Azure Portal hämtar du den privata-IP-adressen för belastningsutjämnaren på skärmen **Översikt**. Gör så här: a. Klicka på **Alla resurser** i den vänstra menyn och klicka sedan på **myLoadBalancer** i resurslistan.
    b. På detaljsidan för **Översikt** kopierar du den privata IP-adressen (i det här exemplet är det 10.1.0.7).

2. Skapa en fjärranslutning till *myVMTest* så här: a. Klicka på **Alla resurser** på den vänstra menyn och klicka sedan i resurslistan på **myVMTest** som finns i resursgruppen *myResourceGroupILB*.
2. Starta en fjärrsession med den virtuella datorn genom att klicka på **Anslut** på sidan **Översikt**.
3. Logga in på *myVMTest*.
3. Klistra in den privata IP-adressen i adressfältet i webbläsaren i *myVMTest*. IIS-webbserverns standardsida visas i webbläsaren.

      ![IIS-webbserver](./media/tutorial-load-balancer-basic-internal-portal/9-load-balancer-test.png)

Om du vill se hur belastningsutjämnaren distribuerar trafik över båda virtuella datorer som kör din app, kan du framtvinga uppdatering av webbläsaren.

## <a name="clean-up-resources"></a>Rensa resurser

Ta bort resursgruppen, belastningsutjämnaren och alla relaterade resurser när de inte längre behövs. Gör det genom att markera den resursgrupp som innehåller belastningsutjämnaren och sedan klicka på **Ta bort**.

## <a name="next-steps"></a>Nästa steg

I den här självstudiekursen skapade du en resursgrupp, nätverksresurser och backend-servrar. Sedan använde du dessa resurser för att skapa en intern belastningsutjämnare som belastningsutjämnar intern trafik till virtuella datorer. Lär dig hur du [belastningsutjämnar virtuella datorer mellan tillgänglighetszoner](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
