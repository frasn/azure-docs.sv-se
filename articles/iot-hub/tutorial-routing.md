---
title: Konfigurera meddelandedirigering med Azure IoT Hub (.NET) | Microsoft Docs
description: Konfigurera meddelandedirigering med Azure IoT Hub
services: iot-hub
documentationcenter: .net
author: robinsh
manager: timlt
editor: tysonn
ms.assetid: ''
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/01/2018
ms.author: robinsh
ms.custom: mvc
ms.openlocfilehash: 0674ed033f77d7d2eca319d0b1e82171dfa4256d
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2018
---
# <a name="tutorial-configure-message-routing-with-iot-hub"></a>Självstudie: Konfigurera meddelandedirigering med IoT Hub

Meddelanderedigering gör det möjligt att skicka telemetridata från dina IoT-enheter för inbyggda Event Hub-kompatibla slutpunkter eller anpassade slutpunkter som Blob Storage, Service Bus Queue, Service Bus Topic och Event Hubs. När du konfigurerar meddelanderedigering kan du skapa hanteringsregler för att anpassa vägen så att den matchar en viss regel. Därefter dirigeras inkommande data automatiskt till slutpunkterna av IoT Hub. 

I den här självstudien får du lära dig att konfigurera och använda dirigeringsregler med IoT Hub. Du kommer att dirigera meddelanden från en IoT-enhet till en av flera tjänster, däribland Blob Storage och en Service Bus-kö. Meddelanden till Service Bus-kön hämtas av en logikapp och skickas via e-post. Meddelanden som inte har dirigering konfigurerat skickas till standardslutpunkten och visas i en PowerBI-visualisering.

I den här självstudien utför du följande åtgärder:

> [!div class="checklist"]
> * Konfigurera grundläggande resurser med Azure CLI eller PowerShell – en IoT-hubb, ett lagringskonto, en Service Bus-kö och en simulerad enhet.
> * Konfigurera slutpunkter och vägar i IoT Hub för lagringskontot och Service Bus-kön.
> * Skapa en logikapp som utlöses och skickar e-post när ett meddelande läggs till i Service Bus-kön.
> * Ladda ned och kör en app som simulerar en IoT-enhet som skickar meddelanden till hubben för de olika dirigeringsalternativen.
> * Skapa en PowerBI-visualisering för data som skickas till standardslutpunkten.
> * Visa resultatet ...
> * ...i Service Bus-kön och e-postmeddelandena.
> * ...i lagringskontot.
> * ...i PowerBI-visualiseringen.

## <a name="prerequisites"></a>Nödvändiga komponenter

- En Azure-prenumeration. Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt konto](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) innan du börjar.

- Installera [Visual Studio för Windows](https://www.visualstudio.com/). 

- Ett PowerBI-konto för att analysera standardslutpunktens Stream Analytics. ([Prova PowerBI kostnadsfritt](https://app.powerbi.com/signupredirect?pbi_source=web))

- Ett Office 365-konto för att skicka e-postmeddelanden. 

Du behöver antingen Azure CLI eller Azure PowerShell för att göra konfigurationsstegen för den här självstudien. 

Om du använder Azure CLI rekommenderar vi att du använder Azure Cloud Shell även om du kan installera Azure CLI lokalt. Azure Cloud Shell är ett kostnadsfritt, interaktivt gränssnitt som du kan använda för att köra Azure CLI-skript. Vanliga Azure-verktyg förinstalleras och konfigureras i Cloud Shell och kan användas med kontot, så du måste inte installera dem lokalt. 

För att använda PowerShell installerar du det lokalt med hjälp av instruktionerna nedan. 

### <a name="azure-cloud-shell"></a>Azure Cloud Shell

Det finns flera olika sätt att öppna Cloud Shell:

|  |   |
|-----------------------------------------------|---|
| Välj **Prova** i det övre högra hörnet av ett kodblock. | ![Cloud Shell i den här artikeln](./media/tutorial-routing/cli-try-it.png) |
| Öppna Cloud Shell i din webbläsare. | [![https://shell.azure.com/bash](./media/tutorial-routing/launchcloudshell.png)](https://shell.azure.com) |
| Välj knappen **Cloud Shell** på menyn längst upp till höger i [Azure Portal](https://portal.azure.com). |    ![Cloud Shell i portalen](./media/tutorial-routing/cloud-shell-menu.png) |
|  |  |

### <a name="using-azure-cli-locally"></a>Använda Azure CLI lokalt

Om du hellre använder CLI lokalt istället för att använda Cloud Shell måste du ha Azure CLI-modul version 2.0.30.0 eller senare. Kör `az --version` för att hitta versionen. Om du behöver installera eller uppgradera kan du läsa [Installera Azure CLI 2.0](/cli/azure/install-azure-cli). 

### <a name="using-powershell-locally"></a>Använda PowerShell lokalt

För den här självstudien krävs Azure PowerShell-modul version 5.7 eller senare. Kör `Get-Module -ListAvailable AzureRM` för att hitta versionen. Om du behöver installera eller uppgradera kan du läsa [Install Azure PowerShell module](/powershell/azure/install-azurerm-ps) (Installera Azure PowerShell-modul).

## <a name="set-up-resources"></a>Konfigurera resurser

För den är självstudien använder du en IoT-hubb, ett lagringskonto och en Service Bus-kö. Alla resurser kan skapas med Azure CLI eller Azure PowerShell. Använd samma resursgrupp och plats för alla resurser. I slutet kan du ta bort allt i ett steg genom att ta bort resursgruppen.

Följande avsnitt beskriver hur du utför stegen som krävs. Följ instruktionerna för CLI *eller* PowerShell.

1. Skapa en [resursgrupp](../azure-resource-manager/resource-group-overview.md). 

    <!-- When they add the Basic tier, change this to use Basic instead of Standard. -->

2. Skapa en IoT-hubb på S1-nivån. Lägg till en konsumentgrupp till din IoT-hubb. Konsumentgruppen används av Azure Stream Analytics när data hämtas.

3. Skapa ett V1-standardlagringskonto med Standard_LRS-replikering.

4. Skapa ett namnområde och en kö för Service Bus. 

5. Skapa en enhetsidentitet för den simulerade enheten som skickar meddelanden till din hubb. Spara nyckeln för testfasen.

### <a name="azure-cli-instructions"></a>Azure CLI-instruktioner

Det lättaste sättet att använda det här skriptet är att kopiera det och klistra in det i Cloud Shell. Förutsatt att du redan är inloggad körs skriptet på en rad i taget. 

```azurecli-interactive

# This is the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity. 
az extension add --name azure-cli-iot-ext

# Set the values for the resource names.
location=westus
resourceGroup=ContosoResources
iotHubConsumerGroup=ContosoConsumers
containerName=contosoresults
iotDeviceName=Contoso-Test-Device 

# These resource names must be globally unique.
# You might need to change these if they are already in use by someone else.
iotHubName=ContosoTestHub 
storageAccountName=contosoresultsstorage 
sbNameSpace=ContosoSBNamespace 
sbQueueName=ContosoSBQueue

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# Create the IoT hub.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --sku S1 --location $location

# Add a consumer group to the IoT hub.
az iot hub consumer-group create --hub-name $iotHubName \
    --name $iotHubConsumerGroup

# Create the storage account to be used as a routing destination.
az storage account create --name $storageAccountName \
    --resource-group $resourceGroup \
    --location $location \
    --sku Standard_LRS

# Get the primary storage account key. 
#    You need this to create the container.
storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroup \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"') 

# See the value of the storage account key.
echo "$storageAccountKey"

# Create the container in the storage account. 
az storage container create --name $containerName \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --public-access off 

# Create the Service Bus namespace.
az servicebus namespace create --resource-group $resourceGroup \
    --name $sbNameSpace \
    --location $location
    
# Create the Service Bus queue to be used as a routing destination.
az servicebus queue create --name $sbQueueName \
    --namespace-name $sbNameSpace \
    --resource-group $resourceGroup

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the information about the device identity, then copy the primary key to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show --device-id $iotDeviceName \
    --hub-name $iotHubName

```

### <a name="powershell-instructions"></a>PowerShell-instruktioner

Det lättaste sättet att använda det här skriptet är att öppna [PowerShell ISE](/powershell/scripting/core-powershell/ise/introducing-the-windows-powershell-ise.md), kopiera skriptet till Urklipp och sedan klistra in hela skriptet i skriptfönstret. Du kan därefter ändra värdena för resursnamnen (om du vill) och köra hela skriptet. 

```azurepowershell-interactive
# Log into Azure account.
Login-AzureRMAccount

# Set the values for the resource names.
$location = "West US"
$resourceGroup = "ContosoResources"
$iotHubConsumerGroup = "ContosoConsumers"
$containerName = "contosoresults"
$iotDeviceName = "Contoso-Test-Device"

# These resource names must be globally unique.
# You might need to change these if they are already in use by someone else.
$iotHubName = "ContosoTestHub"
$storageAccountName = "contosoresultsstorage"
$serviceBusNamespace = "ContosoSBNamespace"
$serviceBusQueueName  = "ContosoSBQueue"

# Create the resource group to be used  
#   for all resources for this tutorial.
New-AzureRmResourceGroup -Name $resourceGroup -Location $location

# Create the IoT hub.
New-AzureRmIotHub -ResourceGroupName $resourceGroup `
    -Name $iotHubName `
    -SkuName "S1" `
    -Location $location `
    -Units 1

# Add a consumer group to the IoT hub for the 'events' endpoint.
Add-AzureRmIotHubEventHubConsumerGroup -ResourceGroupName $resourceGroup `
  -Name $iotHubName `
  -EventHubConsumerGroupName $iotHubConsumerGroup `
  -EventHubEndpointName "events"

# Create the storage account to be used as a routing destination.
# Save the context for the storage account 
#   to be used when creating a container.
$storageAccount = New-AzureRmStorageAccount -ResourceGroupName $resourceGroup `
    -Name $storageAccountName `
    -Location $location `
    -SkuName Standard_LRS `
    -Kind Storage
$storageContext = $storageAccount.Context 

# Create the container in the storage account.
New-AzureStorageContainer -Name $containerName `
    -Context $storageContext

# Create the Service Bus namespace.
New-AzureRmServiceBusNamespace -ResourceGroupName $resourceGroup `
    -Location $location `
    -Name $serviceBusNamespace 

# Create the Service Bus queue to be used as a routing destination.
New-AzureRmServiceBusQueue -ResourceGroupName $resourceGroup `
    -Namespace $serviceBusNamespace `
    -Name $serviceBusQueueName 

```

Skapa sedan en enhetsidentitet och spara nyckeln för framtida bruk. Den är enhetsidentiteten används av simuleringsprogrammet för att skicka meddelanden till IoT-hubben. Den här funktionen finns inte i PowerShell, men du kan skapa enheten i [Azure Portal](https://portal.azure.com).

1. Öppna [Azure Portal](https://portal.azure.com) och logga in på ditt Azure-konto.

2. Klicka på **Resursgrupper** och välj din resursgrupp. I den här självstudien används **ContosoResources**.

3. I listan över resurser klickar du på din IoT-hubb. I självstudien används **ContosoTestHub**. Välj **IoT-enheter** från hubbfönstret.

4. Klicka på **+ Lägg till**. I rutan Lägg till enhet fyller du i enhets-ID. I den här självstudien används **Contoso-Test-Device**. Lämna nycklarna tomma och markera **Generera nycklar automatiskt**. Kontrollera att **Connect device to IoT hub** (Anslut enhet till IoT-hubb) är aktiverat. Klicka på **Spara**.

   ![Skärmbild som visar skärmen Lägg till enhet.](./media/tutorial-routing/add-device.png)

5. Nu när den har skapats klickar du på enheten för att se de genererade nycklarna. Klicka på ikonen Kopiera på primärnyckeln och spara den någonstans, till exempel i Anteckningar, för testfasen i den här självstudien.

   ![Skärmbild som visar information om enheten, som nycklarna.](./media/tutorial-routing/device-details.png)



## <a name="set-up-message-routing"></a>Konfigurera meddelanderedigering

Du kommer att dirigera meddelanden till olika resurser baserat på egenskaper som är anslutna till meddelandet av den simulerade enheten. Meddelanden som inte är dirigeringsanpassade skickas till standardslutpunkten (meddelanden/händelser). 

|värde |Resultat|
|------|------|
|level="storage" |Skriv till Azure Storage.|
|level="critical" |Skriv till en Service Bus-kö. En logikapp hämtar meddelandet från kön och använder Office 365 för att skicka meddelandet via e-post.|
|standard |Visa dessa data med PowerBI.|

### <a name="routing-to-a-storage-account"></a>Dirigera till ett lagringskonto 

Konfigurera nu routning för lagringskontot. Definiera en slutpunkt och konfigurera en väg för den slutpunkten. Meddelanden där **nivåegenskapen** är inställd på **lagring** skrivs till ett lagringskonto automatiskt.

1. I [Azure Portal](https://portal.azure.com) klickar du på **Resursgrupper** och väljer resursgruppen. I den här självstudien används **ContosoResources**. Klicka på IoT-hubben i listan över resurser. I självstudien används **ContosoTestHub**. Klicka på **Slutpunkter**. I rutan **Slutpunkter** klickar du på **+Lägg till**. Ange följande information:

   **Namn**: Ange ett namn på slutpunkten. I den här självstudien används **StorageContainer**.
   
   **Typ av slutpunkt**: Välj **Behållare för Azure Storage** från listrutan.

   Klicka på **Pick a container** (Välj en behållare) för att se en lista över lagringskonton. Välj ditt lagringskonto. I självstudien används **contosoresultsstorage**. Välj sedan behållaren. I självstudien används **contosoresults**. Klicka på **Välj**, vilket gör att du återgår till Lägg till slutpunkt. 
   
   ![Skärmbild som visar hur du lägger till en slutpunkt.](./media/tutorial-routing/add-endpoint-storage-account.png)
   
   Klicka på **OK** för att sluta att lägga till slutpunkten.
   
2. Klicka på **Vägar** på din IoT-hubb. Du ska skapa en hanteringsregel som dirigerar meddelanden till lagringsbehållaren du just lade till som slutpunkt. Klicka på **+Lägg till** överst i rutan Vägar. Fyll i fälten på skärmen. 

   **Namn**: Ange ett namn på routningsregeln. I den här självstudien används **StorageRule**.

   **Datakälla**: Välj **Enhetsmeddelanden** i listrutan.

   **Slutpunkt**: Välj den slutpunkt som du nyss skapade. I den här självstudien används **StorageContainer**. 
   
   **Frågesträng**: Ange `level="storage"` som frågesträng. 

   ![Skärmbild som visar hur du skapar en hanteringsregel för lagringskontot.](./media/tutorial-routing/create-a-new-routing-rule-storage.png)
   
   Klicka på **Spara**. När den är klar returneras den till rutan Vägar, där du kan se din nya hanteringsregel för lagring. Stäng rutan Vägar så att du kommer tillbaka till sidan Resursgrupp.

### <a name="routing-to-a-service-bus-queue"></a>Dirigera till en Service Bus-kö 

Konfigurera nu routning för Service Bus-kön. Definiera en slutpunkt och konfigurera en väg för den slutpunkten. Meddelanden där **nivåegenskapen** är inställd på **kritisk** är skrivna till Service Bus-kön, som utlöser en logikapp som i sin tur skickar ett e-postmeddelande med informationen. 

1. På sidan Resursgrupp klickar du på din IoT-hubb och sedan på **Slutpunkter**. I rutan **Slutpunkter** klickar du på **+Lägg till**. Ange följande information.

   **Namn**: Ange ett namn på slutpunkten. Den här självstudien använder **CriticalQueue**. 

   **Typ av slutpunkt**: Välj **Service Bus-kö** i listrutan.

   **Service Bus-namnrymd**: Välj Service Bus-namnrymden för den här självstudien i listrutan. I den här självstudien används **ContosoSBNamespace**.

   **Service Bus-kö**: Välj Service Bus-kön i listrutan. I den här självstudien används **contososbqueue**.

   ![Skärmbild som visar tillägg av en slutpunkt för Service Bus-kön.](./media/tutorial-routing/add-endpoint-sb-queue.png)

   Spara slutpunkten genom att klicka på **OK**. När det är klart stänger du rutan Slutpunkter. 
    
2. Klicka på **Vägar** på din IoT-hubb. Du ska skapa en hanteringsregel som dirigerar meddelanden till Service Bus-kön du just lade till som slutpunkt. Klicka på **+Lägg till** överst i rutan Vägar. Fyll i fälten på skärmen. 

   **Namn**: Ange ett namn på routningsregeln. I den här självstudien används **SBQueueRule**. 

   **Datakälla**: Välj **Enhetsmeddelanden** i listrutan.

   **Slutpunkt**: Välj den slutpunkt som du nyss skapade, **CriticalQueue**.

   **Frågesträng**: Ange `level="critical"` som frågesträng. 

   ![Skärmbild som visar hur du skapar en hanteringsregel för Service Bus-kön.](./media/tutorial-routing/create-a-new-routing-rule-sbqueue.png)
   
   Klicka på **Spara**. När den återgår till rutan Vägar ser du dina båda nya hanteringsregler som de visas här.

   ![Skärmbild som visar de vägar du just konfigurerade.](./media/tutorial-routing/show-routing-rules-for-hub.png)

   Stäng rutan Vägar så att du kommer tillbaka till sidan Resursgrupp.

## <a name="create-a-logic-app"></a>Skapa en logikapp  

Service Bus-kön ska användas för att ta emot meddelanden som har angetts som kritiska. Konfigurera en logikapp som ska övervaka Service Bus-kön och som skickar e-post när ett meddelande läggs till i Service Bus-kön. 

1. I [Azure Portal](https://portal.azure.com) klickar du på **+ Skapa en resurs**. Skriv **logikapp** i sökrutan och klicka på Retur. Från sökresultatet som visas väljer du Logikapp och klickar på **Skapa** för att fortsätta till rutan **Skapa en logikapp**. Fyll i fälten. 

   **Namn**: Det här fältet är logikappens namn. I självstudien används **ContosoLogicApp**. 

   **Prenumeration**: Välj din Azure-prenumeration.

   **Resursgrupper**: Klicka på **Använd befintlig** och välj din resursgrupp. I den här självstudien används **ContosoResources**. 

   **Plats**: Använd din plats. I den här självstudien används **USA, västra**. 

   **Log Analytics**: Den här växeln bör vara avstängd. 

   ![Skärmbild som visar skärmen Skapa en logikapp.](./media/tutorial-routing/create-logic-app.png)

   Klicka på **Skapa**.

4. Gå nu till logikappen. Det enklaste sättet att komma till logikappen är att klicka på **Resursgrupper**, välja din resursgrupp (den här självstudien använder **ContosoResources**) och sedan välja logikappen från listan över resurser. Sidan Logikappdesigner visas (du kanske måste rulla över till höger för att se hela sidan). På sidan Logikappdesigner rullar du ned tills du ser en ruta med texten **Tom logikapp +** och klickar på den. 

5. En lista med anslutningsprogram visas. Välj **Service Bus**. 

   ![Skärmbild som visar listan över anslutningsprogram.](./media/tutorial-routing/logic-app-connectors.png)

6. En lista med utlösare visas. Välj **Service Bus – När ett meddelande tas emot i en kö (komplettera automatiskt)**. 

   ![Skärmbild som visar listan över utlösare för Service Bus.](./media/tutorial-routing/logic-app-triggers.png)

6. Fyll i namnet på anslutningen på nästa skärm. I den här självstudien används **ContosoConnection**. 

   ![Skärmbild som visar konfiguration av anslutningen för Service Bus-kön.](./media/tutorial-routing/logic-app-define-connection.png)

   Klicka på Service Bus-namnområdet. I den här självstudien används **ContosoSBNamespace**. När du väljer namnrymd ber portalen Service Bus-namnrymden att hämta nycklarna. Välj **RootManageSharedAccessKey** och klicka på **Skapa**. 
   
   ![Skärmbild som visar slutförandet av konfigurationen av anslutningen.](./media/tutorial-routing/logic-app-finish-connection.png)

7. På nästa skärm väljer du namnet på kön (den här självstudien använder **contososbqueue**) i listrutan. Du kan använda standardvärdena för resten av fälten. 

   ![Skärmbild som visar köalternativen.](./media/tutorial-routing/logic-app-queue-options.png)

7. Konfigurera nu att åtgärden ska skicka ett e-postmeddelande när ett meddelande kommer till kön. I Logikappsdesignern klickar du på **+ Nytt steg** för att lägga till ett steg och sedan på **Lägg till en åtgärd**. I rutan **Välj en åtgärd** söker du upp och klickar på **Office 365 Outlook**. På skärmen med utlösare väljer du **Office 365 Outlook – Skicka ett e-postmeddelande**.  

   ![Skärmbild som visar Office365-alternativen.](./media/tutorial-routing/logic-app-select-outlook.png)

8. Logga sedan in på ditt Office 365-konto för att konfigurera anslutningen. Ange e-postadresser för mottagare av e-postmeddelanden. Ange även ämne och skriv vilket meddelande du vill att mottagaren ska se i brödtexten. För att testa fyller du i din egen e-postadress som mottagare.

   Klicka på **Lägg till dynamiskt innehåll** för att visa innehåll från meddelandet som du kan inkludera. Välj **Innehåll** – det inkluderar meddelandet i e-postmeddelandet. 

   ![Skärmbild som visar e-postalternativen för logikappen.](./media/tutorial-routing/logic-app-send-email.png)

9. Klicka på **Spara**. Logikappdesignern öppnas.

## <a name="set-up-azure-stream-analytics"></a>Konfigurera Azure Stream Analytics

Om du vill se data i PowerBI-visualiseringen konfigurerar du först ett Stream Analytics-jobb för att hämta data. Kom ihåg att enbart meddelandena där **nivå** är **normal** skickas till standardslutpunkten och hämtas av Stream Analytics-jobbet för PowerBI-visualiseringen.

### <a name="create-the-stream-analytics-job"></a>Skapa Stream Analytics-jobbet

1. I [Azure Portal](https://portal.azure.com) klickar du på **Skapa en resurs** > **Sakernas Internet** > **Stream Analytics-jobb**.

2. Ange följande information för jobbet.

   **Jobbnamn**: Jobbets namn. Namnet måste vara globalt unikt. I självstudien används **contosoJob**.

   **Resursgrupp**: Använd samma resursgrupp som användes av IoT-hubben. I den här självstudien används **ContosoResources**. 

   **Plats**: Använd samma plats som användes i installationsskriptet. I den här självstudien används **USA, västra**. 

   ![Skärmbild som visar hur du skapar Stream Analytics-jobbet.](./media/tutorial-routing/stream-analytics-create-job.png)

### <a name="add-an-input-to-the-stream-analytics-job"></a>Lägga till indata till Stream Analytics-jobbet

1. Under **Jobbtopologi** klickar du på **Indata**.

2. I rutan **Indata** klickar du på **Lägg till strömindata** och väljer IoT Hub. På skärmen som visas fyller du i följande fält:

   **Inmatat alias**: Den här självstudien använder **contosoinputs**.

   **Prenumeration**: Välj din prenumeration.

   **IoT Hub**: Välj IoT Hub. I självstudien används **ContosoTestHub**.

   **Slutpunkt**: Välj **Meddelanden**. (Om du väljer Åtgärdsövervakning får du telemetridata om IoT-hubben istället för de data du skickar.) 

   **Namn på princip för delad åtkomst**: Välj **iothubowner**. Portalen fyller i Nyckel för princip för delad åtkomst åt dig.

   **Konsumentgrupp**: Välj konsumentgruppen du skapade tidigare. I den här självstudien används **contosoconsumers**.
   
   För resten av fälten accepterar du standardvärdena. 

   ![Skärmbild som visar hur du konfigurerar indata för Stream Analytics-jobbet.](./media/tutorial-routing/stream-analytics-job-inputs.png)

5. Klicka på **Spara**.

### <a name="add-an-output-to-the-stream-analytics-job"></a>Lägga till utdata till Stream Analytics-jobbet

1. Under **Jobbtopologi** klickar du på **Utdata**.

2. I rutan **Utdata** klickar du på **Lägg till** och väljer **PowerBI**. På skärmen som visas fyller du i följande fält:

   **Utdataalias**: Utdatas unika alias. I självstudien används **contosooutputs**. 

   **Namn på datauppsättning**: Namnet på datauppsättningen som ska användas i PowerBI. I självstudien används **contosodataset**. 

   **Tabellnamn**: Namnet på tabellen som ska användas i PowerBI. I självstudien används **contosotable**.

   För resten av fälten accepterar du standardvärdena.

3. Klicka på **Auktorisera** och logga in på PowerBI-kontot.

   ![Skärmbild som visar hur du konfigurerar utdata för Stream Analytics-jobbet.](./media/tutorial-routing/stream-analytics-job-outputs.png)

4. Klicka på **Spara**.

### <a name="configure-the-query-of-the-stream-analytics-job"></a>Konfigurera frågan för Stream Analytics-jobbet

1. Under **Jobbtopologi** klickar du på **Fråga**.

2. Ersätt `[YourInputAlias]` med inmatat alias för jobbet. I självstudien används **contosoinputs**.

3. Ersätt `[YourOutputAlias]` med utdataalias för jobbet. I självstudien används **contosooutputs**.

   ![Skärmbild som visar hur du konfigurerar frågan för Stream Analytics-jobbet.](./media/tutorial-routing/stream-analytics-job-query.png)

4. Klicka på **Spara**.

5. Stäng rutan Fråga.

### <a name="run-the-stream-analytics-job"></a>Köra Stream Analytics-jobbet

I Stream Analytics-jobbet klickar du på **Starta** > **Nu** > **Starta**. När jobbet startar ändras jobbstatusen från **Stoppad** till **Körs**.

För att konfigurera PowerBI-rapporten behöver du data, så du måste konfigurera PowerBI när du har skapat enheten och kör enhetssimuleringsprogrammet.

## <a name="run-simulated-device-app"></a>Köra en simulerad enhetsapp

Tidigare i avsnittet om skriptkonfiguration konfigurerade du en enhet för att använda en IoT-enhet. I det här avsnittet ska du ladda ned en .NET-konsolapp som simulerar en enhet som skickar ”enhet till molnet”-meddelanden till en IoT Hub. Programmet skickar meddelanden för varje routningsmetod. 

Ladda ned lösningen för [IoT-enhetssimuleringen](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Då laddas en lagringsplats ned med flera program i sig. Lösningen du söker är i Tutorials/Routing/SimulatedDevice/.

Dubbelklicka på lösningsfilen (SimulatedDevice.sln) för att öppna koden i Visual Studio och öppna sedan Program.cs. Ersätt `{iot hub hostname}` med IoT-hubbens värdnamn. Formatet för IoT-hubbens värdnamn är **{iot-hub-name}.azure-devices.net**. För den här självstudien är hubbens värdnamn **ContosoTestHub.azure-devices.net**. Ersätt därefter `{device key}` med enhetsnyckeln du sparade tidigare när du konfigurerade den simulerade enheten. 

   ```csharp
        static string myDeviceId = "contoso-test-device";
        static string iotHubUri = "ContosoTestHub.azure-devices.net";
        // This is the primary key for the device. This is in the portal. 
        // Find your IoT hub in the portal > IoT devices > select your device > copy the key. 
        static string deviceKey = "{your device key here}";
   ```

## <a name="run-and-test"></a>Kör och testa 

Kör konsolprogrammet. Vänta några minuter. Du kan se meddelandena som skickas på konsolskärmen i programmet.

Appen skickar ett nytt enhet till molnet-meddelande till IoT-hubben varje sekund. Meddelandet innehåller ett JSON-serialiserat objekt med enhets-ID, temperatur, luftfuktighet och meddelandenivå, där standardinställningen är `normal`. Slumpmässigt tilldelas en nivå på `critical` eller `storage`, vilket orsakar att meddelandet dirigeras till lagringskontot eller till Service Bus-kön (som utlöser att logikappen skickar ett e-postmeddelande). Standardavläsningarna (`normal`) ska visas i BI-rapporten som du ställer in sedan.

Om allt är korrekt konfigurerat bör då se följande resultat:

1. Du börjar få e-postmeddelanden om kritiska meddelanden. 

   ![Skärmbild som visar resulterande e-postmeddelanden.](./media/tutorial-routing/results-in-email.png)

   Detta innebär följande:

   * Routningen till Service Bus-kön fungerar korrekt.
   * Logikappen som hämtar meddelandet från Service Bus-kön fungerar som den ska.
   * Logic App-anslutningsprogrammet till Outlook fungerar som det ska. 

2. I [Azure Portal](https://portal.azure.com) klickar du på **Resursgrupper** och väljer resursgruppen. I den här självstudien används **ContosoResources**. Välj lagringskontot, klicka på **Blobbar** och välj sedan behållaren. I självstudien används **contosoresults**. Du bör se en mapp, och du kan öka detaljnivån mellan kataloger tills du ser en eller flera filer. Öppna en av dessa filer. De innehåller poster som dirigeras till lagringskontot. 

   ![Skärmbild som visar resultatfilerna i lagringen.](./media/tutorial-routing/results-in-storage.png)

Detta innebär följande:

   * Routningen till lagringskontot fungerar korrekt.

Nu när programmet fortfarande körs konfigurerar du PowerBI-visualisering för att se om meddelandena skickas genom standardroutningen. 

## <a name="set-up-the-powerbi-visualizations"></a>Konfigurera PowerBI-visualiseringar

1. Logga in på ditt [PowerBI](https://powerbi.microsoft.com/)-konto.

2. Gå till **Arbetsytor** och välj arbetsytan som du angav när du skapade utdata för Stream Analytics-jobbet. Den här självstudien använder **Min arbetsyta**. 

3. Klicka på **Datauppsättningar**.

   Du bör se den listade datauppsättningen som du angav när du skapade utdata för Stream Analytics-jobbet. I självstudien används **contosodataset**. (Det kan ta 5–10 minuter innan datauppsättningen visas första gången.)

4. Under **ÅTGÄRDER** klickar du på den första ikonen för att skapa en rapport.

   ![Skärmbild som visar PowerBI-arbetsytan med Åtgärder och rapportikonen markerad.](./media/tutorial-routing/power-bi-actions.png)

5. Skapa ett linjediagram för att visa realtidstemperatur över tid.

   a. På sidan för skapande av rapport lägger du till ett linjediagram genom att klicka på linjediagramikonen.

   ![Skärmbild som visar visualiseringar och fält.](./media/tutorial-routing/power-bi-visualizations-and-fields.png)

   b. I rutan **Fält** expanderar du tabellen du angav när du skapade utdata för Stream Analytics-jobbet. I självstudien används **contosotable**.

   c. Dra **EventEnqueuedUtcTime** till **Axel** i fönstret **Visualiseringar**.

   d. Dra **temperatur** till **Värden**.

   Ett linjediagram skapas. X-axeln visar datum och tid i UTC-tidszonen. Y-axeln visar temperatur från sensorn.

7. Skapa ett annat linjediagram om du vill visa realtidsfuktighet över tid. För att konfigurera ett andra diagram följer du samma steg som ovan och placerar **EventEnqueuedUtcTime** på x-axeln och **fuktighet** på y-axeln.

   ![Skärmbild som visar den slutgiltiga PowerBI-rapporten med två diagram.](./media/tutorial-routing/power-bi-report.png)

8. Klicka på **Spara** för att spara rapporten.

Du bör kunna se data i båda diagrammen. Detta innebär följande:

   * Routningen till standardslutpunkten fungerar korrekt.
   * Azure Stream Analytics-jobbet strömmar som det ska.
   * PowerBI-visualiseringen är korrekt konfigurerad.

Du kan uppdatera diagrammen för att se de senaste data genom att klicka på uppdateringsknappen högst upp i PowerBI-fönstret. 

## <a name="clean-up-resources"></a>Rensa resurser 

Om du vill ta bort alla resurser du har skapat tar du bort hela resursgruppen. Detta tar även bort alla resurser som ingår i gruppen. I det här fallet tar det bort IoT-hubben, Service Bus-namnrymden och kön, logikappen, lagringskontot och själva resursgruppen. 

### <a name="clean-up-resources-in-the-powerbi-visualization"></a>Rensa resurser i PowerBI-visualiseringen

Logga in på ditt [PowerBI](https://powerbi.microsoft.com/)-konto. Gå till din arbetsyta. Den här självstudien använder **Min arbetsyta**. Om du vill ta bort PowerBI-visualiseringen går du till DataSets och klickar på papperskorgsikonen och tar bort datauppsättningen. I självstudien används **contosodataset**. När du tar bort datauppsättningen tas även rapporten bort.

### <a name="clean-up-resources-using-azure-cli"></a>Rensa resurser med hjälp av Azure CLI

Om du vill ta bort resursgruppen använder du kommandot [az group delete](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-delete).

```azurecli-interactive
az group delete --name $resourceGroup
```
### <a name="clean-up-resources-using-powershell"></a>Rensa resurser med hjälp av PowerShell

Om du vill ta bort resursgruppen använder du kommandot [Remove-AzureRmResourceGroup](https://docs.microsoft.com/en-us/powershell/module/azurerm.resources/remove-azurermresourcegroup). $resourceGroup ställdes in på **ContosoIoTRG1** i början av den här självstudien.

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name $resourceGroup
```


## <a name="next-steps"></a>Nästa steg

I den här självstudien har du lärt dig att använda meddelanderoutning för att dirigera IoT Hub-meddelanden till olika mål genom att utföra följande uppgifter.  

> [!div class="checklist"]
> * Konfigurera grundläggande resurser med Azure CLI eller PowerShell – en IoT-hubb, ett lagringskonto, en Service Bus-kö och en simulerad enhet.
> * Konfigurera slutpunkter och vägar i IoT Hub för lagringskontot och Service Bus-kön.
> * Skapa en logikapp som utlöses och skickar e-post när ett meddelande läggs till i Service Bus-kön.
> * Ladda ned och kör en app som simulerar en IoT-enhet som skickar meddelanden till hubben för de olika dirigeringsalternativen.
> * Skapa en PowerBI-visualisering för data som skickas till standardslutpunkten.
> * Visa resultatet ...
> * ...i Service Bus-kön och e-postmeddelandena.
> * ...i lagringskontot.
> * ...i PowerBI-visualiseringen.

Gå vidare till nästa självstudie där du får lära dig hur du hanterar tillstånd för en IoT-enhet. 

> [!div class="nextstepaction"]
[Kom igång med Azure IoT Hub-enhetstvillingar](iot-hub-node-node-twin-getstarted.md)

 <!--  [Manage the state of a device](./tutorial-manage-state.md) -->