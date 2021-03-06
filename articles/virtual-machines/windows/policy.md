---
title: Framtvinga säkerhet med principer på virtuella Windows-datorer i Azure | Microsoft Docs
description: Hur du använder en princip till en Azure Resource Manager Windows virtuell dator
services: virtual-machines-windows
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 0b71ba54-01db-43ad-9bca-8ab358ae141b
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 08/02/2017
ms.author: kasing
ms.openlocfilehash: fc95b57c244e855f6788b7f21fc9abc0eba3d44d
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34654716"
---
# <a name="apply-policies-to-windows-vms-with-azure-resource-manager"></a>Tillämpa principer för virtuella Windows-datorer med Azure Resource Manager
En organisation kan tillämpa olika konventioner och regler i hela företaget med hjälp av principer. Tillämpning av önskat beteende kan du minimera risken när bidrar till att organisationen. I den här artikeln beskriver vi hur du kan använda principer för Azure Resource Manager för att definiera önskat beteende för virtuella datorer i din organisation.

En introduktion till principer, se [vad är Azure principen?](../../azure-policy/azure-policy-introduction.md).

## <a name="permitted-virtual-machines"></a>Tillåtna virtuella datorer
För att säkerställa att virtuella datorer för din organisation är kompatibla med ett program, kan du begränsa de tillåtna operativsystem. I exemplet nedan principen Tillåt endast Windows Server 2012 R2 Datacenter virtuella datorer som ska skapas:

```json
{
  "if": {
    "allOf": [
      {
        "field": "type",
        "in": [
          "Microsoft.Compute/disks",
          "Microsoft.Compute/virtualMachines",
          "Microsoft.Compute/VirtualMachineScaleSets"
        ]
      },
      {
        "not": {
          "allOf": [
            {
              "field": "Microsoft.Compute/imagePublisher",
              "in": [
                "MicrosoftWindowsServer"
              ]
            },
            {
              "field": "Microsoft.Compute/imageOffer",
              "in": [
                "WindowsServer"
              ]
            },
            {
              "field": "Microsoft.Compute/imageSku",
              "in": [
                "2012-R2-Datacenter"
              ]
            },
            {
              "field": "Microsoft.Compute/imageVersion",
              "in": [
                "latest"
              ]
            }
          ]
        }
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

Du kan använda jokertecken för att ändra föregående princip för att tillåta alla Windows Server Datacenter-avbildning:

```json
{
  "field": "Microsoft.Compute/imageSku",
  "like": "*Datacenter"
}
```

Använd anyOf för att ändra föregående princip för att tillåta alla Windows Server 2012 R2 Datacenter eller högre avbildningen:

```json
{
  "anyOf": [
    {
      "field": "Microsoft.Compute/imageSku",
      "like": "2012-R2-Datacenter*"
    },
    {
      "field": "Microsoft.Compute/imageSku",
      "like": "2016-Datacenter*"
    }
  ]
}
```

Information om principfält finns [princip alias](../../azure-policy/policy-definition.md#aliases).

## <a name="managed-disks"></a>Hanterade diskar

Om du vill kräva användning av hanterade diskar, använder du följande princip:

```json
{
  "if": {
    "anyOf": [
      {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/virtualMachines"
          },
          {
            "field": "Microsoft.Compute/virtualMachines/osDisk.uri",
            "exists": true
          }
        ]
      },
      {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Compute/VirtualMachineScaleSets"
          },
          {
            "anyOf": [
              {
                "field": "Microsoft.Compute/VirtualMachineScaleSets/osDisk.vhdContainers",
                "exists": true
              },
              {
                "field": "Microsoft.Compute/VirtualMachineScaleSets/osdisk.imageUrl",
                "exists": true
              }
            ]
          }
        ]
      }
    ]
  },
  "then": {
    "effect": "deny"
  }
}
```

## <a name="images-for-virtual-machines"></a>Avbildningar för virtuella datorer

Av säkerhetsskäl bör kräva du att godkända anpassade avbildningar distribueras i din miljö. Du kan ange antingen resursgruppen som innehåller godkända bilder eller specifika godkända bilder.

I följande exempel kräver avbildningar från en godkänd resursgrupp:

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "in": [
                    "Microsoft.Compute/virtualMachines",
                    "Microsoft.Compute/VirtualMachineScaleSets"
                ]
            },
            {
                "not": {
                    "field": "Microsoft.Compute/imageId",
                    "contains": "resourceGroups/CustomImage"
                }
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
} 
```

I följande exempel anger godkända image-ID: N:

```json
{
    "field": "Microsoft.Compute/imageId",
    "in": ["{imageId1}","{imageId2}"]
}
```

## <a name="virtual-machine-extensions"></a>Tillägg för virtuell dator

Du kanske vill förbjuda användningen av vissa typer av tillägg. Till exempel kanske ett tillägg inte kompatibelt med vissa virtuella datoravbildningar. I följande exempel visas hur du blockerar ett specifikt filnamnstillägg. Används för utgivare och typ för att bestämma vilka tillägg som ska blockeras.

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "equals": "Microsoft.Compute/virtualMachines/extensions"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/publisher",
                "equals": "Microsoft.Compute"
            },
            {
                "field": "Microsoft.Compute/virtualMachines/extensions/type",
                "equals": "{extension-type}"

      }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```


## <a name="azure-hybrid-use-benefit"></a>Azure Hybrid-förmånen

När du har en licens för lokala sparar du licens avgift på virtuella datorer. När du inte har licensen som bör du förbjuda alternativet. Följande princip tillåter inte användning av Azure Hybrid Använd förmånen (AHUB):

```json
{
    "if": {
        "allOf": [
            {
                "field": "type",
                "in":[ "Microsoft.Compute/virtualMachines","Microsoft.Compute/VirtualMachineScaleSets"]
            },
            {
                "field": "Microsoft.Compute/licenseType",
                "exists": true
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}
```

## <a name="next-steps"></a>Nästa steg
* När du definierar en regel (som visas i föregående exempel) behöver du skapar principdefinitionen och kopplar den till ett omfång. Omfattningen kan vara en prenumeration, resursgrupp eller resurs. Om du vill tilldela principer finns [Använd Azure-portalen för att tilldela och hantera resursprinciper](../../azure-policy/assign-policy-definition.md), [Använd PowerShell för att tilldela principer](../../azure-policy/assign-policy-definition-ps.md), eller [Använd Azure CLI för att tilldela principer](../../azure-policy/assign-policy-definition-cli.md).
* En introduktion till resursprinciper finns [vad är Azure principen?](../../azure-policy/azure-policy-introduction.md).
* Vägledning för hur företag kan använda resurshanteraren för att effektivt hantera prenumerationer finns i [Azure enterprise scaffold - förebyggande prenumerationsåtgärder](/azure/architecture/cloud-adoption-guide/subscription-governance).