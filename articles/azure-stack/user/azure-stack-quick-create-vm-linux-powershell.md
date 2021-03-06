---
title: Skapa en virtuell Linux-dator med hjälp av PowerShell i Azure-stacken | Microsoft Docs
description: Skapa en virtuell Linux-dator med PowerShell i Azure-stacken.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 03EE5929-4F05-47D7-B246-EA93D6FC47CD
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 04/24/2018
ms.author: mabrigg
ms.custom: mvc
ms.openlocfilehash: 1e2dbc6020dd317e96c4116811f8e3bf87680bfb
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/04/2018
---
# <a name="quickstart-create-a-linux-server-virtual-machine-by-using-powershell-in-azure-stack"></a>Snabbstart: skapa en virtuell dator Linux server med PowerShell i Azure-stacken

*Gäller för: Azure Stack integrerat system och Azure-stacken Development Kit*

Du kan skapa en virtuell Ubuntu Server 16.04 LTS-dator med hjälp av PowerShell för Azure-stacken. Följ stegen i den här artikeln för att skapa och använda en virtuell dator.  Den här artikeln kan du också stegen för att:

* Ansluta till den virtuella datorn med en fjärransluten klient.
* Installera NGINX-webbserver och visa standardstartsida.
* Rensa oanvända resurser.

## <a name="prerequisites"></a>Förutsättningar

* **En Linux-avbildning i Azure-stacken marketplace**

   Stacken för Azure marketplace innehåller inte en Linux-avbildning som standard. Hämta Azure Stack-operatorn för att ange den **Ubuntu Server 16.04 LTS** bilden som du behöver. Operatorn kan använda stegen som beskrivs i den [hämta marketplace-objekt från Azure till Azure-stacken](../azure-stack-download-azure-marketplace-item.md) artikel.

* Azure-stacken kräver en viss version av Azure PowerShell för att skapa och hantera resurser. Om du inte har konfigurerats för stacken Azure PowerShell, logga in på den [development kit](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop), eller en Windows-baserad extern klient om du är [anslutna via VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) och följ stegen för att [ installera](azure-stack-powershell-install.md) och [konfigurera](azure-stack-powershell-configure-user.md) PowerShell.

* En offentlig SSH-nyckel med namnet-id_rsa.pub sparas i katalogen .ssh på din Windows-användarprofil. Detaljerad information om hur du skapar SSH-nycklar finns [skapa SSH-nycklar i Windows](../../virtual-machines/linux/ssh-from-windows.md).

## <a name="create-a-resource-group"></a>Skapa en resursgrupp

En resursgrupp är en logisk behållare där du kan distribuera och hantera resurser i Azure-stacken. Kör följande kodblock om du vill skapa en resursgrupp från din development kit eller Azure-stacken integrerat system. Tilldelar värden för alla variabler i det här dokumentet kan du använda dessa värden eller tilldela nya värden.

```powershell
# Create variables to store the location and resource group names.
$location = "local"
$ResourceGroupName = "myResourceGroup"

New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $location
```

## <a name="create-storage-resources"></a>Skapa storage-resurser

Skapa ett lagringskonto och sedan skapa en lagringsbehållare för Ubuntu Server 16.04 LTS avbildningen.

```powershell
# Create variables to store the storage account name and the storage account SKU information
$StorageAccountName = "mystorageaccount"
$SkuName = "Standard_LRS"

# Create a new storage account
$StorageAccount = New-AzureRMStorageAccount `
  -Location $location `
  -ResourceGroupName $ResourceGroupName `
  -Type $SkuName `
  -Name $StorageAccountName

Set-AzureRmCurrentStorageAccount `
  -StorageAccountName $storageAccountName `
  -ResourceGroupName $resourceGroupName

# Create a storage container to store the virtual machine image
$containerName = 'osdisks'
$container = New-AzureStorageContainer `
  -Name $containerName `
  -Permission Blob
```

## <a name="create-networking-resources"></a>Skapa nätverksresurser

Skapa ett virtuellt nätverk, undernät och offentlig IP-adress. Dessa resurser används för att ge nätverksanslutning till den virtuella datorn.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name MyVnet `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"

```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>Skapa en nätverkssäkerhetsgrupp och en regel för nätverkssäkerhetsgrupp

Nätverkssäkerhetsgruppen skyddar den virtuella datorn med hjälp av regler för inkommande och utgående. Skapa en regel för inkommande trafik för port 3389 för att tillåta inkommande anslutningar till fjärrskrivbord och en inkommande regel för port 80 för att tillåta inkommande webbtrafik.

```powershell
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleSSH  -Protocol Tcp `
-Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 22 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
-Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location local `
-Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>Skapa ett nätverkskort för den virtuella datorn

Nätverkskortet ansluter den virtuella datorn till ett undernät, en nätverkssäkerhetsgrupp och offentlig IP-adress.

```powershell
# Create a virtual network card and associate it with public IP address and NSG
$nic = New-AzureRmNetworkInterface `
  -Name myNic `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>Skapa en virtuell dator

Skapa en virtuell datorkonfiguration. Den här konfigurationen innehåller de inställningar som används när du distribuerar den virtuella datorn. Till exempel: användarautentiseringsuppgifter, storlek och avbildningen av virtuella datorn.

```powershell
# Define a credential object.
$UserName='demouser'
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ($UserName, $securePassword)

# Create the virtual machine configuration object
$VmName = "VirtualMachinelatest"
$VmSize = "Standard_D1"
$VirtualMachine = New-AzureRmVMConfig `
  -VMName $VmName `
  -VMSize $VmSize

$VirtualMachine = Set-AzureRmVMOperatingSystem `
  -VM $VirtualMachine `
  -Linux `
  -ComputerName "MainComputer" `
  -Credential $cred

$VirtualMachine = Set-AzureRmVMSourceImage `
  -VM $VirtualMachine `
  -PublisherName "Canonical" `
  -Offer "UbuntuServer" `
  -Skus "16.04-LTS" `
  -Version "latest"

$osDiskName = "OsDisk"
$osDiskUri = '{0}vhds/{1}-{2}.vhd' -f `
  $StorageAccount.PrimaryEndpoints.Blob.ToString(),`
  $vmName.ToLower(), `
  $osDiskName

# Sets the operating system disk properties on a virtual machine.
$VirtualMachine = Set-AzureRmVMOSDisk `
  -VM $VirtualMachine `
  -Name $osDiskName `
  -VhdUri $OsDiskUri `
  -CreateOption FromImage | `
  Add-AzureRmVMNetworkInterface -Id $nic.Id

# Configure SSH Keys
$sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"

# Adds the SSH Key to the virtual machine
Add-AzureRmVMSshPublicKey -VM $VirtualMachine `
 -KeyData $sshPublicKey `
 -Path "/home/azureuser/.ssh/authorized_keys"

# Create the virtual machine.
New-AzureRmVM `
  -ResourceGroupName $ResourceGroupName `
 -Location $location `
  -VM $VirtualMachine
```

## <a name="connect-to-the-virtual-machine"></a>Ansluta till den virtuella datorn

När den virtuella datorn har distribuerats kan du konfigurera en SSH-anslutning för den virtuella datorn. Använd kommandot [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress?view=azurermps-4.3.1) för att returnera den offentliga IP-adressen för den virtuella datorn.

```powershell
Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

Från en klientdator med SSH installerat, använder du följande kommando för att ansluta till den virtuella datorn. Om du arbetar med Windows, kan du använda [Putty](http://www.putty.org/) att skapa anslutningen.

```
ssh <Public IP Address>
```

När du uppmanas, anger du azureuser som användarens inloggning. Om du använder en lösenfras när du har skapat SSH-nycklar, har du att ange lösenfrasen.

## <a name="install-the-nginx-web-server"></a>Installera NGINX-webbserver

Om du vill uppdatera paketet resurser och installera det senaste NGINX-paketet, kör du följande skript:

```bash
#!/bin/bash

# update package source
apt-get -y update

# install NGINX
apt-get -y install nginx
```

## <a name="view-the-nginx-welcome-page"></a>Visa NGINX-välkomstsidan

Med NGINX installerat, och port 80 som är öppna på den virtuella datorn, kan du komma åt webbservern med hjälp av den virtuella datorns offentliga IP-adressen. Öppna en webbläsare och gå till ```http://<public IP address>```.

![Välkomstsida för NGINX web server](./media/azure-stack-quick-create-vm-linux-cli/nginx.png)

## <a name="clean-up-resources"></a>Rensa resurser

Rensa de resurser som du inte behöver längre. Du kan använda den [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup?view=azurermps-4.3.1) kommando för att ta bort dessa resurser. Ta bort resursgruppen och alla dess resurser genom att köra följande kommando:

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Nästa steg

I den här snabbstarten distribuerade virtuell dator för grundläggande Linux-server. Om du vill veta mer om Azure-stacken virtuella datorer kan du gå till [överväganden för virtuella datorer i Azure-stacken](azure-stack-vm-considerations.md).
