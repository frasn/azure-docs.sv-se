---
title: Skapa en virtuell dator i Azure portal | Microsoft Docs
description: Skapa en virtuell Windows-dator i Azure-portalen.
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 1871f823-ebd7-4eff-9a22-8e2411555595
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: cynthn
ms.openlocfilehash: 5fd2128ff436d3211f41c7dfdcc4c2b8aabd0eb0
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/06/2018
---
# <a name="create-a-virtual-machine-running-windows-in-the-azure-portal"></a>Skapa en virtuell dator som kör Windows i Azure-portalen
> [!div class="op_single_selector"]
> * [Azure Portal](tutorial.md)
> * [PowerShell: Klassisk distribution](create-powershell.md)
>
>

<br>

> [!IMPORTANT]
> Azure har två olika distributionsmodeller för att skapa och arbeta med resurser: [Resource Manager och klassisk](../../../resource-manager-deployment-model.md). Den här artikeln täcker den klassiska distributionsmodellen. Microsoft rekommenderar att de flesta nya distributioner använder Resource Manager-modellen. Lär dig hur du [utföra dessa steg med Resource Manager-distributionsmodellen](../../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) med hjälp av den **Azure-portalen**.
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

Den här kursen visar hur du skapar en Azure virtuell dator (VM) med Windows Azure-portalen. Vi använder en Windows Server-avbildning som exempel, men det är bara en av de många avbildningar som Azure erbjuder. Observera att avbildningsalternativ beror på din prenumeration. Till exempel vara Windows desktop-avbildningar tillgängliga för MSDN-prenumeranter.

Det här avsnittet visar hur du använder den **instrumentpanelen** i Azure portal för att välja och skapa den virtuella datorn.

Du kan också skapa virtuella datorer med [egna avbildningar](createupload-vhd.md). Läs om denna och andra metoder i [olika sätt att skapa en virtuell dator för Windows](../../virtual-machines-windows-creation-choices.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a id="createvirtualmachine"> </a>Skapa den virtuella datorn
[!INCLUDE [virtual-machines-create-WindowsVM](../../../../includes/virtual-machines-create-windowsvm.md)]

## <a name="next-steps"></a>Nästa steg
* Lär dig hur du [skapa en virtuell dator med hjälp av Resource Manager-distributionsmodellen](../../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) i Azure-portalen.
* Logga in på den virtuella datorn. Instruktioner finns i [logga in på en virtuell dator som kör Windows Server](connect-logon.md).
* Ansluta en disk för att lagra data. Du kan koppla både tom och diskar som innehåller data. Mer information finns i [ansluta en datadisk till en Windows-dator som skapats med den klassiska distributionsmodellen](attach-disk.md).
