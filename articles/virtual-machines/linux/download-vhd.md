---
title: Hämta en Linux-VHD från Azure | Microsoft Docs
description: Hämta en Linux VHD med Azure CLI och Azure portal.
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 06/01/2018
ms.author: cynthn
ms.openlocfilehash: 1c6751d980a7bb28e58a3aa00514411959f515d7
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34725874"
---
# <a name="download-a-linux-vhd-from-azure"></a>Hämta en Linux-VHD från Azure

I den här artikeln får du lära dig hur du hämtar en [Linux virtuell hårddisk (VHD)](about-disks-and-vhds.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) filen från Azure med hjälp av Azure CLI och Azure-portalen. 

Om du inte redan har gjort det installerar [Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-az-cli2).

## <a name="stop-the-vm"></a>Stoppa den virtuella datorn

En virtuell Hårddisk kan inte hämtas från Azure om den är kopplad till en aktiv virtuell dator. Du måste stoppa den virtuella datorn för att ladda ned en virtuell Hårddisk. Om du vill använda en virtuell Hårddisk som en [bild](tutorial-custom-images.md) för att skapa andra virtuella datorer med nya diskar, måste du ta bort etableringen och generalisera operativsystemet finns i filen och stoppa den virtuella datorn. Om du vill använda den virtuella Hårddisken som en disk för en ny instans av en befintlig virtuell dator eller en datadisk, behöver du bara stoppa och ta bort den virtuella datorn.

Om du vill använda den virtuella Hårddisken som en bild för att skapa andra virtuella datorer, gör du följande:

1. Använda SSH, kontonamnet och offentliga IP-adressen för den virtuella datorn för att ansluta till den och ta bort etableringen av den. Du kan hitta den offentliga IP-adressen med [az nätverket offentliga ip-visa](https://docs.microsoft.com/cli/azure/network/public-ip#az-network-public-ip-show). Den + user-parameter tar också bort det senaste kontot för etablerad användare. Om du gräddning autentiseringsuppgifter i för att den virtuella datorn, lämnar ut detta + user-parameter. I följande exempel tar bort det senaste kontot för etablerad användare:

    ```bash
    ssh azureuser@<publicIpAddress>
    sudo waagent -deprovision+user -force
    exit 
    ```

2. Logga in på ditt Azure-konto med [az inloggningen](https://docs.microsoft.com/cli/azure/reference-index#az_login).
3. Stoppa och ta bort den virtuella datorn.

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

4. Generalisera den virtuella datorn. 

    ```azurecli
    az vm generalize --resource-group myResourceGroup --name myVM
    ``` 

Om du vill använda den virtuella Hårddisken som en disk för en ny instans av en befintlig virtuell dator eller en datadisk, gör du följande:

1.  Logga in på [Azure-portalen](https://portal.azure.com/).
2.  Klicka på **Virtual Machines** på navmenyn.
3.  Välj den virtuella datorn från listan.
4.  På bladet för den virtuella datorn klickar du på **stoppa**.

    ![Stoppa VM](./media/download-vhd/export-stop.png)

## <a name="generate-sas-url"></a>Generera en SAS-URL

Om du vill hämta VHD-filen måste du generera en [signatur för delad åtkomst (SAS)](../../storage/common/storage-dotnet-shared-access-signature-part-1.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) URL. När URL: en genereras tilldelas en förfallotid till URL: en.

1.  Klicka på menyn i bladet för den virtuella datorn, **diskar**.
2.  Välj disken som operativsystemet för den virtuella datorn och klicka sedan på **exportera**.
3.  Klicka på **generera URL**.

    ![Generera en URL](./media/download-vhd/export-generate.png)

## <a name="download-vhd"></a>Hämta VHD

1.  Klicka på Hämta VHD-filen under den URL som har genererats.

    ![Hämta VHD](./media/download-vhd/export-download.png)

2.  Du kan behöva klicka på **spara** i webbläsaren om du vill starta nedladdningen. Standardnamnet för VHD-filen är *abcd*.

    ![Klicka på Spara i webbläsaren](./media/download-vhd/export-save.png)

## <a name="next-steps"></a>Nästa steg

- Lär dig hur du [överför och skapa en Linux VM från anpassade disken med Azure CLI 2.0](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 
- [Hantera Azure-diskarna Azure CLI](tutorial-manage-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

