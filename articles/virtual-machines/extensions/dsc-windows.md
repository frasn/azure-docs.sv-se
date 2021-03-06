---
title: Azure Desired State Configuration-tillägget hanterare | Microsoft Docs
description: Ladda upp och tillämpa en PowerShell DSC-konfiguration på en virtuell Azure-dator med hjälp av DSC-tillägg
services: virtual-machines-windows
documentationcenter: ''
author: eshaparmar
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: windows
ms.workload: ''
ms.date: 03/26/2018
ms.author: esparmar
ms.openlocfilehash: b34314951980f7dbe2269119883dec52a90a0587
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/10/2018
---
# <a name="powershell-dsc-extension"></a>PowerShell DSC-tillägg

## <a name="overview"></a>Översikt

PowerShell DSC-tillägg för Windows är publicerad och stöds av Microsoft. Tillägget överför och tillämpar en PowerShell DSC-konfigurationen på en virtuell dator i Azure. DSC-tillägg anropar i PowerShell DSC att införa mottagna DSC-konfigurationen på den virtuella datorn. Det här dokumentet beskriver de plattformar som stöds, konfigurationer och distributionsalternativ för DSC-tillägget för virtuell dator för Windows.

## <a name="prerequisites"></a>Förutsättningar

### <a name="operating-system"></a>Operativsystem

DSC-tillägg stöder följande OS

Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 SP1, Windows Client 7/8.1

### <a name="internet-connectivity"></a>Internetanslutning

DSC-tillägg för Windows kräver att den virtuella måldatorn är ansluten till internet. 

## <a name="extension-schema"></a>Tilläggsschema

Följande JSON visar schemat för den inställningar delen av DSC-tillägg i en Azure Resource Manager-mall. 

```json
{
  "type": "Microsoft.Compute/virtualMachines/extensions",
  "name": "Microsoft.Powershell.DSC",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "properties": {
    "publisher": "Microsoft.Powershell",
    "type": "DSC",
    "typeHandlerVersion": "2.73",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "wmfVersion": "latest",
        "configuration": {
            "url": "http://validURLToConfigLocation",
            "script": "ConfigurationScript.ps1",
            "function": "ConfigurationFunction"
        },
        "configurationArguments": {
            "argument1": "Value1",
            "argument2": "Value2"
        },
        "configurationData": {
            "url": "https://foo.psd1"
        },
        "privacy": {
            "dataCollection": "enable"
        },
        "advancedOptions": {
            "forcePullAndApply": false
            "downloadMappings": {
                "specificDependencyKey": "https://myCustomDependencyLocation"
            }
        } 
    },
    "protectedSettings": {
        "configurationArguments": {
            "parameterOfTypePSCredential1": {
                "userName": "UsernameValue1",
                "password": "PasswordValue1"
            },
            "parameterOfTypePSCredential2": {
                "userName": "UsernameValue2",
                "password": "PasswordValue2"
            }
        },
        "configurationUrlSasToken": "?g!bber1sht0k3n",
        "configurationDataUrlSasToken": "?dataAcC355T0k3N"
    }
  }
}
```

### <a name="property-values"></a>Egenskapsvärden

| Namn | Värdet / exempel | Datatyp |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | datum |
| Publisher | Microsoft.Powershell.DSC | sträng |
| typ | DSC | sträng |
| typeHandlerVersion | 2,73 | int |

### <a name="settings-property-values"></a>Inställningar för egenskapsvärden

| Namn | Datatyp | Beskrivning
| ---- | ---- | ---- |
| settings.wmfVersion | sträng | Anger versionen av Windows Management Framework som ska installeras på den virtuella datorn. Den här egenskapen till ”senaste” installerar den senaste uppdaterade versionen av WMF. Endast aktuella möjliga värden för den här egenskapen är ”4.0', '5.0' och 'senaste'. Dessa möjliga värden är regleras av uppdateringar. Standardvärdet är ”senaste”. |
| settings.configuration.url | sträng | Anger URL-plats som du vill hämta DSC-konfiguration zip-filen. Om den URL som kräver en SAS-token för åtkomst och behöver du ange egenskapen protectedSettings.configurationUrlSasToken till värdet för SAS-token. Den här egenskapen är obligatorisk om settings.configuration.script och/eller settings.configuration.function definieras.
| Settings.Configuration.Script | sträng | Anger namnet på det skript som innehåller definitionen av DSC-konfigurationen. Det här skriptet måste vara i rotmappen på zip-filen som hämtats från den URL som anges av egenskapen configuration.url. Den här egenskapen är obligatorisk om settings.configuration.url och/eller settings.configuration.script definieras.
| settings.configuration.function | sträng | Anger namnet på DSC-konfigurationen. Konfigurationen med namnet måste finnas i skriptet som definieras av configuration.script. Den här egenskapen är obligatorisk om settings.configuration.url och/eller settings.configuration.function definieras.
| settings.configurationArguments | Samling | Definierar de parametrar som du vill skicka DSC-konfigurationen. Den här egenskapen kommer inte krypteras.
| settings.configurationData.url | sträng | Anger den URL som du vill hämta din konfigurationsdatafilen (.pds1) ska användas som indata för DSC-konfigurationen. Om den URL som kräver en SAS-token för åtkomst och behöver du ange egenskapen protectedSettings.configurationDataUrlSasToken till värdet för SAS-token.
| settings.privacy.dataEnabled | sträng | Aktiverar eller inaktiverar telemetri samling. Endast möjliga värden för den här egenskapen är 'Aktivera', inaktivera ”,, eller $null. Om du lämnar den här egenskapen är tomt eller null aktiverar telemetri
| settings.advancedOptions.forcePullAndApply | bool | Aktiverar DSC-tillägg att uppdatera och tillämpar DSC-konfigurationer när uppdateringen är Pull.
| settings.advancedOptions.downloadMappings | Samling | Definierar andra platser för att hämta beroenden, till exempel WMF och .NET

### <a name="protected-settings-property-values"></a>Skyddade inställningarna egenskapsvärden

| Namn | Datatyp | Beskrivning
| ---- | ---- | ---- |
| protectedSettings.configurationArguments | sträng | Definierar de parametrar som du vill skicka DSC-konfigurationen. Den här egenskapen krypteras. |
| protectedSettings.configurationUrlSasToken | sträng | Anger SAS-token för att komma åt den URL som definierats av configuration.url. Den här egenskapen krypteras. |
| protectedSettings.configurationDataUrlSasToken | sträng | Anger SAS-token för att komma åt den URL som definierats av configurationData.url. Den här egenskapen krypteras. |


## <a name="template-deployment"></a>Malldistribution

Azure VM-tillägg kan distribueras med Azure Resource Manager-mallar. Mallar är perfekt när du distribuerar en eller flera virtuella datorer som kräver post distributionskonfiguration. En Resource Manager-mall som innehåller OMS-agenten VM-tillägget kan hittas på den [Azure Quick Start-galleriet](https://github.com/Azure/azure-quickstart-templates/tree/052db5feeba11f85d57f170d8202123511f72044/dsc-extension-iis-server-windows-vm). 

JSON-konfiguration för ett tillägg för virtuell dator kan kapslas i den virtuella datorresursen eller placeras i roten eller översta nivån i en Resource Manager JSON-mall. Placeringen av JSON-konfigurationen påverkar värdet av resursens namn och typen. 

När kapsla resursen tillägget JSON placeras i den `"resources": []` objekt av den virtuella datorn. När du monterar tillägget JSON i roten på mallen resursnamnet innehåller en referens till den överordnade virtuella datorn och typen visar kapslade konfigurationen.  


## <a name="azure-cli-deployment"></a>Azure CLI-distribution

Azure CLI kan användas för att distribuera OMS-agenten VM-tillägget till en befintlig virtuell dator. Ersätt OMS-nyckel och OMS-ID med de från din OMS-arbetsyta. 

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name Microsoft.Powershell.DSC \
  --publisher Microsoft.Powershell \
  --version 2.73 --protected-settings '{}' \
  --settings '{}'
```

## <a name="troubleshoot-and-support"></a>Felsöka och stöd

### <a name="troubleshoot"></a>Felsöka

Data om tillståndet för distributioner av tillägget kan hämtas från Azure-portalen och genom att använda Azure CLI. Om du vill se distributionsstatusen för tillägg för en viss virtuell dator, kör du följande kommando med hjälp av Azure CLI.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

Tilläggspaket hämtas och distribueras till den här platsen på den virtuella Azure-datorn
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}
```

Tilläggsfilen status innehåller sub status och slutfört/fel statuskoder tillsammans med detaljerade fel och beskrivning för varje tillägg som körs.
```
C:\Packages\Plugins\{Extension_Name}\{Extension_Version}\Status\{0}.Status  -> {0} being the sequence number
```

Tillägget utdataloggar loggas till följande katalog:

```
C:\WindowsAzure\Logs\Plugins\{Extension_Name}\{Extension_Version}
```

### <a name="error-codes-and-their-meanings"></a>Felkoder och deras innebörd

| Felkod | Betydelse | Möjlig åtgärd |
| :---: | --- | --- |
| 1000 | Allmänt fel | Meddelandet för det här felet kommer från särskilda undantag i tillägget loggar |
| 52 | Tillägget installera fel | Meddelandet för det här felet finns i det här undantaget |
| 1002 | Installera WMF fel | Fel uppstod när installera WMF. |
| 1004 | Ogiltig Zip-paketet | Ogiltig zip; Packa upp zip fel |
| 1100 | Argumentfel | Anger ett problem i indata som angetts av användaren. Meddelande för felet kommer från det här undantaget|


### <a name="support"></a>Support

Om du behöver mer hjälp när som helst i den här artikeln kan du kontakta Azure-experter på den [MSDN Azure och Stack Overflow-forum](https://azure.microsoft.com/support/forums/). Alternativt kan du lagra en incident i Azure-supporten. Gå till den [Azure supportwebbplats](https://azure.microsoft.com/support/options/) och välja Get support. Information om hur du använder Azure stöder finns i [vanliga frågor om Microsoft Azure-supporten](https://azure.microsoft.com/support/faq/).