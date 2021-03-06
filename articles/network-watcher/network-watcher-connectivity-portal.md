---
title: Felsöka anslutningar med Nätverksbevakaren Azure - Azure-portalen | Microsoft Docs
description: Lär dig hur du använder anslutningen felsöka möjligheterna för Nätverksbevakaren i Azure med Azure-portalen.
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/03/2017
ms.author: jdial
ms.openlocfilehash: cf7b71a49b63a95ed535210125120c6b76d9de8f
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2018
---
# <a name="troubleshoot-connections-with-azure-network-watcher-using-the-azure-portal"></a>Felsöka anslutningar med Nätverksbevakaren i Azure med Azure-portalen

> [!div class="op_single_selector"]
> - [Portal](network-watcher-connectivity-portal.md)
> - [PowerShell](network-watcher-connectivity-powershell.md)
> - [CLI 2.0](network-watcher-connectivity-cli.md)
> - [Azure REST-API](network-watcher-connectivity-rest.md)

Lär dig hur du använder anslutning felsöka för att kontrollera om en direkt TCP-anslutning från en virtuell dator till en viss slutpunkt kan upprättas.

## <a name="before-you-begin"></a>Innan du börjar

Den här artikeln förutsätter att du har följande resurser:

* En instans av Nätverksbevakaren i regionen som du vill felsöka en anslutning.
* Virtuella datorer för att felsöka anslutningar med.

> [!IMPORTANT]
> Felsökning av anslutning kräver att den virtuella datorn som du felsöker från den `AzureNetworkWatcherExtension` VM-tillägget installeras. Installera tillägget på en Windows VM finns [tillägg för virtuell dator i Azure Network Watcher Agent för Windows](../virtual-machines/windows/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) och för Linux VM besöka [tillägg för virtuell dator i Azure Network Watcher Agent för Linux](../virtual-machines/linux/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Tillägget krävs inte på målslutpunkten.

## <a name="check-connectivity-to-a-virtual-machine"></a>Kontrollera anslutningen till en virtuell dator

Det här exemplet kontrollerar anslutningen till en virtuell dator för målet via port 80.

Navigera till din Nätverksbevakaren och på **anslutning felsöka**. Välj den virtuella datorn för att kontrollera anslutningen mellan. I den **mål** väljer **väljer en virtuell dator** och välj rätt virtuell dator och port för att testa.

När du klickar på **Kontrollera**, anslutningar mellan virtuella datorer på den angivna porten är markerad. I det här exemplet målet VM kan inte nås, visas en lista över hopp.

![Resultat av anslutningen för en virtuell dator][1]

## <a name="check-remote-endpoint-connectivity"></a>Kontrollera att fjärrslutpunkten anslutning

Om du vill kontrollera anslutningen och fördröjning till en fjärrslutpunkt, Välj den **ange manuellt** alternativknapp i den **mål** avsnittet, ange URL-adressen och porten och klicka på **Kontrollera**.  Det här används för fjärråtkomst-slutpunkter som slutpunkter för webbplatser och lagring.

![Resultat av anslutningen för en webbplats][2]

## <a name="next-steps"></a>Nästa steg

Lär dig att automatisera insamlingar paket med virtuella aviseringar genom att visa [skapar en avisering utlösta paketinsamling](network-watcher-alert-triggered-packet-capture.md)

Hitta om vissa trafik tillåts i eller utanför den virtuella datorn genom att besöka [Kontrollera Kontrollera IP-flöde](diagnose-vm-network-traffic-filtering-problem.md)

[1]: ./media/network-watcher-connectivity-portal/figure1.png
[2]: ./media/network-watcher-connectivity-portal/figure2.png