---
title: Använda Azure Policy för att skapa och hantera principer för organisationsefterlevnad
description: Använd Azure Policy för att genomdriva standarder, efterleva krav på regelefterlevnad och granskning, kontrollera kostnader, behålla en konsekvent säkerhet och prestanda och tillämpa företagsomfattande principer.
services: azure-policy
author: DCtheGeek
ms.author: dacoulte
ms.date: 05/07/2018
ms.topic: tutorial
ms.service: azure-policy
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: 1d13c5c0099b8e4c9ed83221edf6a2ee92fc3c8e
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/16/2018
ms.locfileid: "34194877"
---
# <a name="create-and-manage-policies-to-enforce-compliance"></a>Skapa och hantera principer för att använda kompatibilitet

Det är viktigt att du förstår hur du skapar och hanterar principer i Azure för att du ska fortsätta att efterleva företagets standarder och servicenivåavtal. I den här självstudien lär du dig att använda Azure Policy för att utföra några av de vanligaste uppgifterna som att skapa, tilldela och hantera principer i organisationen, exempelvis:

> [!div class="checklist"]
> - Tilldela en princip för att genomdriva ett villkor för resurser som du skapar i framtiden
> - Skapa och tilldela en initiativdefinition för att spåra efterlevnad för flera resurser
> - Lösa en icke-kompatibel eller nekad resurs
> - Implementera en ny princip i en organisation

Om du vill tilldela en princip för att identifiera dina befintliga resursers efterlevnadstillstånd går artiklarna i snabbstarten igenom det. Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt konto](https://azure.microsoft.com/free/) innan du börjar.

## <a name="assign-a-policy"></a>Tilldela en princip

Det första steget för att tillämpa efterlevnad av Azure Policy är att tilldela en principdefinition. En principdefinition anger enligt vilket villkor en princip tillämpas och vilken åtgärd som ska vidtas. I det här exemplet ska du tilldela en inbyggd principdefinition som heter *Kräv SQL Server version 12.0*, för att tillämpa villkoret att alla SQL Server-databaser måste vara v12.0 för att vara kompatibla.

1. Starta Azure Policy-tjänsten i Azure Portal genom att klicka på **Alla tjänster** och sedan söka efter och välja **Princip**.

   ![Sök efter princip](media/create-manage-policy/search-policy.png)

2. Välj **Tilldelningar** till vänster på sidan Azure Policy. En tilldelning är en princip som tilldelats ett specifikt område.
3. Välj **Tilldela princip** längst upp på sidan **Princip – Tilldelningar**.

   ![Tilldela en principdefinition](media/create-manage-policy/select-assign-policy.png)

4. På sidan **Tilldela princip** väljer du **Omfång** genom att klicka på ellipsen och välja en prenumeration (krävs) och en resursgrupp (valfritt). En omfattning avgör vilka resurser eller grupper med resurser som principtilldelningen används på.  Klicka sedan på **Välj** längst ned på sidan **Omfång**.

   I det här exemplet används **Contoso-prenumerationen**. Din prenumeration skiljer sig.

5. Om du vill undanta en eller flera resursgrupper (om du bara använder en prenumeration i omfånget) eller specifika resurser inom en resursgrupp (något av omfångsalternativen), kan du konfigurera **Undantag** från principtilldelningen. Lämna det tomt just nu.

6. Välj ellipsen **Principdefinition** för att öppna listan med tillgängliga definitioner. Du kan filtrera principdefinitionen **Typ** som *Inbyggt* om du vill se alla och läsa deras beskrivningar.

7. Välj **Kräv SQL Server version 12.0**. Om du inte hittar det genast skriver du **kräv sql server** i sökrutan och trycker sedan på RETUR eller klickar utanför sökrutan. Klicka på **Välj** längst ned på sidan **Tillgängliga definitioner** när du har hittat och valt principdefinitionen.

   ![Hitta en princip](media/create-manage-policy/select-available-definition.png)

8. **Tilldelningsnamn** fylls i automatiskt med namnet på principen som du valde, men du kan ändra det om du vill. I det här exemplet låter du *Kräv SQL Server version 12.0* vara kvar. Du kan också lägga till en valfri **Beskrivning**. Beskrivningen innehåller information om den här principtilldelningen.

9. Klicka på **Tilldela**.

## <a name="implement-a-new-custom-policy"></a>Implementera en ny anpassad princip

Nu när du har tilldelat en inbyggd principdefinition kan du göra mer med Azure Policy. Sedan ska du skapa en ny anpassad princip för att spara kostnader genom att se till att de virtuella datorer du har skapat i din miljö inte kan vara i G-serien. På så sätt nekas varje gång en begäran från en användare i organisationen som försöker skapa en virtuell dator i G-serien.

1. Välj **Definitioner** under **REDIGERING** till vänster på sidan Azure Policy.

   ![Definition under redigering](media/create-manage-policy/definition-under-authoring.png)

2. Välj **+ Principdefinition** överst på sidan. Sidan **Principdefinition** öppnas.
3. Ange följande:

   - Hanteringsgruppen eller prenumerationen där principdefinitionen sparas. Välj **Definitionens plats** med ellipsen.

     > [!NOTE]
     > Om du tänker tillämpa denna principdefinition på flera prenumerationer, måste platsen vara en hanteringsgrupp som innehåller de prenumerationer som du vill tilldela principen till. Detsamma gäller för en initiativdefinition.

   - Namnet på principdefinitionen – *Kräv VM SKU:er som är mindre än G-serien*
   - Beskrivning av vad principdefinitionen är avsedd att göra – *Den här principdefinitionen tvingar alla virtuella datorer som är skapade i omfånget att ha SKU:er som är mindre än G-serien för att minska kostnaderna.*
   - Välj bland befintliga alternativ eller skapa en ny kategori för principdefinitionen.
   - Kopiera följande json-kod och uppdatera den sedan efter dina behov med:
      - Principparametrarna.
      - Principens regler/villkor, i det här fallet – VM SKU-storlek som motsvarar G-serien
      - Principens effekt, i det här fallet – **Neka**.

    Så här ska din json se ut. Klistra in den reviderade koden i Azure-portalen.

    ```json
    {
        "policyRule": {
            "if": {
                "allOf": [{
                        "field": "type",
                        "equals": "Microsoft.Compute/virtualMachines"
                    },
                    {
                        "field": "Microsoft.Compute/virtualMachines/sku.name",
                        "like": "Standard_G*"
                    }
                ]
            },
            "then": {
                "effect": "deny"
            }
        }
    }
    ```

    Värdet i egenskapen *Fält* i principregeln måste vara något av följande: Namn, Typ, Plats, Taggar eller ett alias. Ett exempel på ett alias kan vara `"Microsoft.Compute/VirtualMachines/Size"`.

    Fler Azure-principexempel finns i [Mallar för Azure Policy](json-samples.md).

4. Välj **Spara**.

## <a name="create-a-policy-definition-with-rest-api"></a>Skapa en principdefinition med REST API

Du kan skapa en princip med REST API för principdefinitioner. Med REST API kan du skapa och ta bort principdefinitioner och få information om befintliga definitioner.
Om du vill skapa en principdefinition använder du följande exempel:

```http-interactive
PUT https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}
```

Inkludera en begärantext som liknar följande exempel:

```json
{
    "properties": {
        "parameters": {
            "allowedLocations": {
                "type": "array",
                "metadata": {
                    "description": "The list of locations that can be specified when deploying resources",
                    "strongType": "location",
                    "displayName": "Allowed locations"
                }
            }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
            "if": {
                "not": {
                    "field": "location",
                    "in": "[parameters('allowedLocations')]"
                }
            },
            "then": {
                "effect": "deny"
            }
        }
    }
}
```

## <a name="create-a-policy-definition-with-powershell"></a>Skapa en principdefinition med PowerShell

Innan du fortsätter med PowerShell-exemplet ska du se till att du har den senaste versionen av Azure PowerShell installerad. Principparametrar lades till i version 3.6.0. Om du har en tidigare version returnerar exemplen ett fel som anger att parametern inte går att hitta.

Du kan skapa en principdefinition med cmdleten `New-AzureRmPolicyDefinition`.

Om du vill skapa en principdefinition från en fil skickar du sökvägen till filen. Använd följande exempel för en extern fil:

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition `
    -Name 'denyCoolTiering' `
    -DisplayName 'Deny cool access tiering for storage' `
    -Policy 'https://raw.githubusercontent.com/Azure/azure-policy-samples/master/samples/Storage/storage-account-access-tier/azurepolicy.rules.json'
```

Använd följande exempel för en lokal fil:

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition `
    -Name 'denyCoolTiering' `
    -Description 'Deny cool access tiering for storage' `
    -Policy 'c:\policies\coolAccessTier.json'
```

Om du vill skapa en principdefinition med en infogad regel använder du följande exempel:

```azurepowershell-interactive
$definition = New-AzureRmPolicyDefinition -Name 'denyCoolTiering' -Description 'Deny cool access tiering for storage' -Policy '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "not": {
                    "field": "Microsoft.Storage/storageAccounts/accessTier",
                    "equals": "cool"
                }
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

Utdata lagras i ett `$definition`-objekt som används under principtilldelningen.
I följande exempel skapas en principdefinition som innehåller parametrarna:

```azurepowershell-interactive
$policy = '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "not": {
                    "field": "location",
                    "in": "[parameters(''allowedLocations'')]"
                }
            }
        ]
    },
    "then": {
        "effect": "Deny"
    }
}'

$parameters = '{
    "allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of locations that can be specified when deploying storage accounts.",
            "strongType": "location",
            "displayName": "Allowed locations"
        }
    }
}'

$definition = New-AzureRmPolicyDefinition -Name 'storageLocations' -Description 'Policy to specify locations for storage accounts.' -Policy $policy -Parameter $parameters
```

### <a name="view-policy-definitions-with-powershell"></a>Visa principdefinitioner med PowerShell

Om du vill visa alla principdefinitioner i din prenumeration använder du följande kommando:

```azurepowershell-interactive
Get-AzureRmPolicyDefinition
```

Den returnerar alla tillgängliga principdefinitioner, inklusive inbyggda principer. Varje princip returneras i följande format:

```
Name               : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceId         : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceName       : e56962a6-4747-49cd-b67b-bf8b01975c4c
ResourceType       : Microsoft.Authorization/policyDefinitions
Properties         : @{displayName=Allowed locations; policyType=BuiltIn; description=This policy enables you to
                     restrict the locations your organization can specify when deploying resources. Use to enforce
                     your geo-compliance requirements.; parameters=; policyRule=}
PolicyDefinitionId : /providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c
```

## <a name="create-a-policy-definition-with-azure-cli"></a>Skapa en principdefinition med Azure CLI

Du kan skapa en principdefinition med Azure CLI med kommandot för principdefinition.
Om du vill skapa en principdefinition med en infogad regel använder du följande exempel:

```azurecli-interactive
az policy definition create --name 'denyCoolTiering' --description 'Deny cool access tiering for storage' --rules '{
    "if": {
        "allOf": [{
                "field": "type",
                "equals": "Microsoft.Storage/storageAccounts"
            },
            {
                "field": "kind",
                "equals": "BlobStorage"
            },
            {
                "not": {
                    "field": "Microsoft.Storage/storageAccounts/accessTier",
                    "equals": "cool"
                }
            }
        ]
    },
    "then": {
        "effect": "deny"
    }
}'
```

### <a name="view-policy-definitions-with-azure-cli"></a>Visa principdefinitioner med Azure CLI

Om du vill visa alla principdefinitioner i din prenumeration använder du följande kommando:

```azurecli-interactive
az policy definition list
```

Den returnerar alla tillgängliga principdefinitioner, inklusive inbyggda principer. Varje princip returneras i följande format:

```json
{
    "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements.",
    "displayName": "Allowed locations",
    "id": "/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "name": "e56962a6-4747-49cd-b67b-bf8b01975c4c",
    "policyRule": {
        "if": {
            "not": {
                "field": "location",
                "in": "[parameters('listOfAllowedLocations')]"
            }
        },
        "then": {
            "effect": "Deny"
        }
    },
    "policyType": "BuiltIn"
}
```

## <a name="create-and-assign-an-initiative-definition"></a>Skapa och tilldela en initiativdefinition

Med en initiativdefinition kan du gruppera flera principdefinitioner för att uppnå ett övergripande mål. Du skapar en initiativdefinition för att se till att resurser inom definitionens omfång efterlever principdefinitionerna som utgör initiativdefinitionen.  Se [Översikt över Azure Policy](azure-policy-introduction.md) för mer information om initiativdefinitioner.

### <a name="create-an-initiative-definition"></a>Skapa en initiativdefinition

1. Välj **Definitioner** under **REDIGERING** till vänster på sidan Azure Policy.

   ![Välja definitioner](media/create-manage-policy/select-definitions.png)

2. Välj **+ Initiativdefinition** överst på sidan för att öppna sidan **Initiativdefinition**.

   ![Initiativdefinition](media/create-manage-policy/initiative-definition.png)

3. Använd ellipsen **Definitionens plats** för att välja en prenumeration där definitionen ska lagras.

4. Ange **Namn** och **Beskrivning** för initiativet.

   I det här exemplet ser vi till att resurserna efterlever principdefinitionernas säkerhet. Namnet på initiativet blir därför **Get Secure** och beskrivningen blir: **This initiative has been created to handle all policy definitions associated with securing resources** (Det här initiativet har skapats för att hantera alla principdefinitioner som är associerade med att skydda resurser).

5. Välj bland befintliga alternativ eller skapa en ny kategori i **Kategori**.

6. Bläddra igenom listan **Tillgängliga definitioner** (högre halvan av sidan **Initiativdefinition**) och välj de principdefinitioner som du vill lägga till i initiativet. För initiativet **Get Secure** lägger du till följande inbyggda principdefinitioner genom att klicka på **+** bredvid principdefinitionens information eller klicka på en rad för principdefinitionen och sedan på alternativet **+ Lägg till** på informationssidan:
   - Kräv SQL Server version 12.0
   - [Preview]: Monitor unprotected web applications in Security Center.
   - [Preview]: Monitor permissive network across in Security Center.
   - [Preview]: Monitor possible app Whitelisting in Security Center.
   - [Preview]: Monitor unencrypted VM Disks in Security Center.

   När du har valt principdefinitionen i listan läggs den till under **PRINCIPER OCH PARAMETRAR**.

   ![Initiativdefinitioner](media/create-manage-policy/initiative-definition-2.png)

7. Klicka på **Spara**.

### <a name="assign-an-initiative-definition"></a>Tilldela en initiativdefinition

1. Välj **Definitioner** under **REDIGERING** till vänster på sidan Azure Policy.
2. Leta upp initiativdefinitionen **Get Secure** som du skapade tidigare och markera den.
3. Välj **Tilldela** överst på sidan för att öppna sidan **Get Secure: Assign Initiative**.

   ![Tilldela en definition](media/create-manage-policy/assign-definition.png)

   Du kan också högerklicka på den markerade raden eller vänsterklicka på ellipsen i slutet av raden för att få upp en snabbmeny.  Välj sedan **Tilldela**.

   ![Högerklicka på en rad](media/create-manage-policy/select-right-click.png)

4. Fyll i sidan **Get Secure: Assign Initiative** genom att ange följande exempelinformation. Du kan använda din egen information.

   - Omfång: Den prenumeration där du sparade initiativet blir standardprenumerationen.  Du kan ändra omfång för att tilldela initiativet till en resursgrupp på prenumerationens lagringsplats.
   - Undantag: Konfigurera några resurser inom omfånget för att förhindra att initiativtilldelningen tillämpas på dem.
   - Initiativdefinition och tilldelningsnamn: Get Secure (förifylld med namnet på initiativet som tilldelats).
   - Beskrivning: Initiativtilldelningen är skräddarsydd för att tillämpa den här gruppen med principdefinitioner.

5. Klicka på **Tilldela**.

## <a name="exempt-a-non-compliant-or-denied-resource-using-exclusion"></a>Undanta en icke-kompatibel eller nekad resurs med Exkludering

När du har angett att principdefinitionen kräver version 12.0 av SQL Server skulle en SQL-server som har skapats med en annan version än 12.0 nekas, precis som i exemplet ovan. I det här avsnittet går vi igenom hur du hanterar ett nekat försök att skapa en SQL-server genom att skapa ett undantag på en enskild resursgrupp. Undantaget förhindrar att principen (eller initiativet) tillämpas på resursen. I följande exempel tillåts alla SQL-serverversioner i en enskild resursgrupp. Ett undantag kan tillämpas på en resursgrupp, eller så kan du begränsa undantaget till enskilda resurser.

En distribution som har förhindrats på grund av en tilldelad princip eller initiativ kan visas på två platser:

- I den resursgrupp som är mål för distributionen: Välj **Distributioner** till vänster på sidan och klicka på **Distributionsnamn** för den misslyckade distributionen. Resursen som nekades visas med statusen _Förbjuden_. För att fastställa den princip, det initiativ och den tilldelning som har nekat resursen, klickar du på **Misslyckades. Klicka här för information ->** på sidan Distributionsöversikt. Ett fönster öppnas till höger på sidan med felinformationen. Under **Felinformation** finns GUID för de relaterade principobjekten.

   ![Distributionen nekas av principtilldelningen](media/create-manage-policy/rg-deployment-denied.png)

- På sidan Azure Policy: Välj **Efterlevnad** till vänster på sidan och klicka på principen **Kräv SQL Server version 12.0**. På sidan som öppnas visas en ökning i antalet för **Neka**. Under fliken **Händelser** visas också vem som försökte utföra distributionen som nekades av principen.

   ![Efterlevnadsöversikt för en tilldelad princip](media/create-manage-policy/compliance-overview.png)

I det här exemplet har Trent Baker, en av Contosos virtualiseringsspecialister, utfört ett arbete som krävdes. Vi måste bevilja honom ett undantag, men vi vill inte ha SQL-servrar som inte är av version 12.0 i någon resursgrupp. Vi har skapat den nya resursgruppen **SQLServers_Excluded** och kommer nu att bevilja ett undantag till den för principtilldelningen.

### <a name="update-assignment-with-exclusion"></a>Uppdatera tilldelning med undantag

1. Välj **Tilldelningar** under **REDIGERING** till vänster på sidan Azure Policy.
2. Bläddra igenom alla principtilldelningar och öppna tilldelningen *Kräv SQL Server version 12.0*.
3. Ange **Undantag** genom att klicka på ellipsen och välj den resursgrupp som ska exkluderas, vilket är *SQLServers_Excluded* i det här exemplet.

   ![Begär exkludering](media/create-manage-policy/request-exclusion.png)

   > [!NOTE]
   > Beroende på principen och dess effekt skulle undantaget också kunna beviljas till specifika resurser i en resursgrupp inom omfånget för tilldelningen. Eftersom **Neka** har använts i den här självstudien, gör det ingen skillnad att ange undantag för en specifik resurs som redan finns.

4. Klicka på **Välj** och sedan på **Spara**.

I det här avsnittet har du hanterat ett nekat försök att skapa en förbjuden version av SQL-servern genom att skapa ett undantag för en enskild resursgrupp.

## <a name="clean-up-resources"></a>Rensa resurser

Om du är klar med att arbeta med resurser i den här självstudien kan du använda följande steg för att ta bort tilldelningar eller definitioner som skapades ovan:

1. Välj **Definitioner** (eller **Tilldelningar** om du ska ta bort en tilldelning) under **REDIGERING** till vänster på sidan Azure Policy.
2. Sök efter den nya initiativ- eller principdefinition (eller tilldelning) som du vill ta bort.
3. Högerklicka på raden eller välj ellipserna i slutet av definitionen (eller tilldelningen) och välj **Ta bort definition** (eller **Ta bort tilldelning**).

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du gjort följande:

> [!div class="checklist"]
> - Tilldelat en princip för att genomdriva ett villkor för resurser som du skapar i framtiden
> - Skapat och tilldelat en initiativdefinition för att spåra efterlevnad för flera resurser
> - Löst en icke-kompatibel eller nekad resurs
> - Implementerat en ny princip i en organisation

Mer information om principdefinitionernas strukturer finns i den här artikeln:

> [!div class="nextstepaction"]
> [Azure Policy-definitionsstruktur](policy-definition.md)