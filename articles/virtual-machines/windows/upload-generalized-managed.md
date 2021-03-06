---
title: Skapa en hanterad Azure virtuell dator från en virtuell Hårddisk generaliserad lokalt | Microsoft Docs
description: Överför en generaliserad virtuell Hårddisk till Azure och använda den för att skapa nya virtuella datorer, i Resource Manager-distributionsmodellen.
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
ms.date: 03/26/2018
ms.author: cynthn
ms.openlocfilehash: 9ebe1f67c7c662af6d9e1888580149834a007200
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34657479"
---
# <a name="upload-a-generalized-vhd-and-use-it-to-create-new-vms-in-azure"></a>Överför en generaliserad virtuell Hårddisk och använda den för att skapa nya virtuella datorer i Azure

Det här avsnittet beskriver hur du använder PowerShell för att överföra en VHD från en generaliserad virtuell dator till Azure, skapa en avbildning från den virtuella Hårddisken och skapa en ny virtuell dator från den avbildningen. Du kan överföra en virtuell Hårddisk som exporteras från ett verktyg för virtualisering av lokalt eller från ett annat moln. Med hjälp av [hanterade diskar](managed-disks-overview.md) för den nya virtuella datorn förenklas VM och ger bättre tillgänglighet när den virtuella datorn placeras i en tillgänglighetsuppsättning. 

Om du vill använda ett exempelskript finns [exempel på skript för att överföra en virtuell Hårddisk till Azure och skapa en ny virtuell dator](../scripts/virtual-machines-windows-powershell-upload-generalized-script.md)

## <a name="before-you-begin"></a>Innan du börjar

- Du bör följa innan du laddar upp alla VHD till Azure [förbereda en Windows-VHD eller VHDX för att överföra till Azure](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
- Granska [planera för migrering till hanterade diskar](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks) innan du påbörjar migreringen till [hanterade diskar](managed-disks-overview.md).
- Den här artikeln kräver AzureRM Modulversion 5.6 eller senare. Kör ` Get-Module -ListAvailable AzureRM.Compute` för att hitta versionen. Om du behöver uppgradera kan du läsa [Install Azure PowerShell module](/powershell/azure/install-azurerm-ps) (Installera Azure PowerShell-modul).


## <a name="generalize-the-source-vm-using-sysprep"></a>Generalisera den Virtuella källdatorn med hjälp av Sysprep

Sysprep tar bland annat bort all din personliga kontoinformation och förbereder datorn så att den kan användas som en avbildning. Mer information om Sysprep finns i [Sysprep översikt](https://docs.microsoft.com/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Se till att serverroller som körs på datorn som stöds av Sysprep. Mer information finns i [Sysprep-stöd för serverroller](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> Om du kör Sysprep innan du laddar upp den virtuella Hårddisken till Azure för första gången, kontrollera att du har [förberett din virtuella dator](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) innan du kör Sysprep. 
> 
> 

1. Logga in på Windows-dator.
2. Öppna Kommandotolken som administratör. Ändra katalogen till **%windir%\system32\sysprep**, och kör sedan `sysprep.exe`.
3. Välj **Starta OOBE för systemet (Out-of-Box Experience)** i dialogrutan **Systemförberedelseverktyget** och kontrollera att kryssrutan **Generalisera** är markerad.
4. I **avstängningsalternativ**väljer **avstängning**.
5. Klicka på **OK**.
   
    ![Starta Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. När Sysprep har slutförts stängs den virtuella datorn av. Starta inte om den virtuella datorn.


## <a name="get-the-storage-account"></a>Hämta storage-konto

Du behöver ett lagringskonto i Azure för att lagra överförda VM-avbildning. Du kan använda ett befintligt lagringskonto eller skapa en ny. 

Om du kommer att använda den virtuella Hårddisken för att skapa en hanterad disk för en virtuell dator, lagringsplats för kontot måste vara samma plats där du skapar den virtuella datorn.

Om du vill visa tillgängliga storage-konton, skriver du:

```azurepowershell
Get-AzureRmStorageAccount | Format-Table
```

## <a name="upload-the-vhd-to-your-storage-account"></a>Överför den virtuella Hårddisken till ditt lagringskonto

Använd den [Lägg till AzureRmVhd](https://msdn.microsoft.com/library/mt603554.aspx) för att ladda upp den virtuella Hårddisken till en behållare i ditt lagringskonto. Det här exemplet överför filen *myVHD.vhd* från *”C:\Users\Public\Documents\Virtual hårddiskar\"*  till ett lagringskonto med namnet *mittlagringskonto* i den *myResourceGroup* resursgruppen. Filen placeras i behållare med namnet *minbehållare* och det nya namnet kommer att *myUploadedVHD.vhd*.

```powershell
$rgName = "myResourceGroup"
$urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd"
Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
    -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
```


Om det lyckas, kan du få ett svar som liknar denna:

```powershell
MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
MD5 hash calculation is completed.
Elapsed time for the operation: 00:03:35
Creating new page blob of size 53687091712...
Elapsed time for upload: 01:12:49

LocalFilePath           DestinationUri
-------------           --------------
C:\Users\Public\Doc...  https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd
```

Det här kommandot kan ta en stund att slutföra beroende på nätverksanslutningen och storleken på VHD-filen

### <a name="other-options-for-uploading-a-vhd"></a>Andra alternativ för att överföra en virtuell Hårddisk
 
Du kan också ladda upp en virtuell Hårddisk till ditt lagringskonto med hjälp av något av följande:

- [AzCopy](http://aka.ms/downloadazcopy)
- [Azure Storage kopiera Blob API](https://msdn.microsoft.com/library/azure/dd894037.aspx)
- [Azure Storage Explorer överför Blobbar](https://azurestorageexplorer.codeplex.com/)
- [Storage Import/Export Service REST API-referens](https://msdn.microsoft.com/library/dn529096.aspx)
-   Du rekommenderas att använda Import/Export Service om överföring tid är längre än 7 dagar. Du kan använda [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) att uppskatta storlek och överför dataenheten tidpunkt. 
    Import/Export kan användas för att kopiera till ett standardlagringskonto. Du måste kopiera från standardlagring till premium storage-konto med ett verktyg som AzCopy.

> [!IMPORTANT]
> Om du använder AzCopy överför den virtuella Hårddisken till Azure, kontrollerar du att du har angett [/BlobType:page](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy#blobtypeblock--page--append) överför innan du kör skriptet. Om målet är en blob och det här alternativet inte anges som standard skapas en blockblobb AzCopy.
> 
> 



## <a name="create-a-managed-image-from-the-uploaded-vhd"></a>Skapa en hanterad avbildning från den överförda virtuella Hårddisken 

Skapa en hanterad avbildning med hjälp av din generaliserad OS-VHD. Ersätt värdena med din egen information.


Ange först vissa parametrar:

```powershell
$location = "East US" 
$imageName = "myImage"
```

Skapa avbildningen med din generaliserad OS-VHD.

```powershell
$imageConfig = New-AzureRmImageConfig `
   -Location $location
$imageConfig = Set-AzureRmImageOsDisk `
   -Image $imageConfig `
   -OsType Windows `
   -OsState Generalized `
   -BlobUri $urlOfUploadedImageVhd `
   -DiskSizeGB 20
New-AzureRmImage `
   -ImageName $imageName `
   -ResourceGroupName $rgName `
   -Image $imageConfig
```


## <a name="create-the-vm"></a>Skapa den virtuella datorn

Nu när du har en avbildning kan du skapa en eller flera nya virtuella datorer från avbildningen. Det här exemplet skapar en virtuell dator med namnet *myVM* från den *myImage*i den *myResourceGroup*.


```powershell
New-AzureRmVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -ImageName $imageName `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNSG" `
    -PublicIpAddressName "myPIP" `
    -OpenPorts 3389
```


## <a name="next-steps"></a>Nästa steg

Logga in på den nya virtuella datorn. Mer information finns i [ansluta och logga in på en virtuell Azure-dator kör Windows](connect-logon.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 

