---
title: Distribuera SSIS-paket till Azure med PowerShell | Microsoft Docs
description: Den här artikeln förklarar hur du distribuerar SSIS-paket till Azure och skapar ett Azure-SSIS Integration Runtime med PowerShell.
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: hero-article
ms.date: 04/13/2018
ms.author: douglasl
ms.openlocfilehash: 3b127eded5e9f9bc921e5830975513c9e7233a2c
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/19/2018
---
# <a name="deploy-sql-server-integration-services-packages-to-azure-with-powershell"></a>Distribuera SQL Server Integration Services-paket till Azure med PowerShell
Den här självstudien innehåller steg för att distribuera en Azure SSIS Integration Runtime (IR) i Azure Data Factory. Sedan kan du använda SQL Server Data Tools (SSDT) eller SQL Server Management Studio (SSMS) för att distribuera SQL Server Integration Services-paket (SSIS) till den här körningen i Azure. I den här självstudien gör du följande:

> [!NOTE]
> I den här artikeln används Azure PowerShell till att etablera en Azure SSIS-integreringskörning. Om du vill använda Data Factory-användargränssnittet till att etablera en Azure SSIS-integritetskörning kan du läsa [Självstudie: Skapa en Azure SSIS-integreringskörning](tutorial-create-azure-ssis-runtime-portal.md). 

> [!div class="checklist"]
> * Skapa en datafabrik.
> * Skapa en Azure-SSIS Integration Runtime
> * Starta en Azure-SSIS Integration Runtime
> * Distribuera SSIS-paket
> * Granska det fullständiga skriptet

> [!NOTE]
> Den här artikeln gäller för version 2 av Data Factory, som för närvarande är en förhandsversion. Om du använder version 1 av Data Factory-tjänsten, som är allmänt tillgänglig, läser du [dokumentationen för Data Factory version 1](v1/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

## <a name="prerequisites"></a>Nödvändiga komponenter
- **Azure-prenumeration**. Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt](https://azure.microsoft.com/free/) konto innan du börjar. Begreppsrelaterad information om Azure-SSIS IR finns i [översikten över Azure SSIS Integration Runtime](concepts-integration-runtime.md#azure-ssis-integration-runtime).
- **Azure SQL Database-server**. Om du inte redan har en databasserver kan du skapa en i Azure-portalen innan du börjar. Den här servern är värd för SSISDB (SSIS Catalog Database). Vi rekommenderar att du skapar databasservern i samma Azure-region som Integration Runtime. Den här konfigurationen gör att Integration Runtimes skrivkörning loggas till SSISDB utan att korsa Azure-regioner. 
    - Bekräfta att inställningen **tillåt åtkomst till Azure-tjänster** är **På** för databasservern. Mer information finns i [säkra din Azure SQL-databas](../sql-database/sql-database-security-tutorial.md#create-a-server-level-firewall-rule-in-the-azure-portal). Om du vill aktivera den här inställningen med hjälp av PowerShell, se [New-AzureRmSqlServerFirewallRule](/powershell/module/azurerm.sql/new-azurermsqlserverfirewallrule?view=azurermps-4.4.1).
    - Lägg till IP-adressen för klientdatorn eller ett intervall med IP-adresser som innehåller IP-adressen för klientdatorn till klientens IP-adresslista i brandväggsinställningarna för databasservern. För mer information, se [Azure SQL Database-brandväggsregler på servernivå och databasnivå](../sql-database/sql-database-firewall-configure.md). 
    - Bekräfta att din Azure SQL Database-server inte har någon SSIS-katalog (SSIDB-databas). När du ska etablera Azure-SSIS IR kan du inte ha någon befintlig SSIS-katalog. 
- **Azure PowerShell**. Följ instruktionerna i [Så här installerar och konfigurerar du Azure PowerShell](/powershell/azure/install-azurerm-ps). Du använder PowerShell för att köra ett skript för att etablera en Azure-SSIS Integration Runtime som kör SSIS-paket i molnet. 

> [!NOTE]
> - Du kan skapa en datafabrik med version 2 i följande regioner: USA, östra; USA, östra 2; Asien, sydöstra och Europa, västra. 
> - Du kan skapa en integreringskörning för Azure-SSIS i följande regioner: USA, östra; USA, östra 2; USA, centrala; USA, västra 2; Europa, norra; Europa, västra; Storbritannien, södra och Australien, östra.

## <a name="launch-windows-powershell-ise"></a>Starta Windows PowerShell ISE
Starta **Windows PowerShell ISE** med administratörsprivilegier. 

## <a name="create-variables"></a>Skapa variabler
Kopiera och klistra in följande skript: Ange värden för variablerna. En lista över **prisnivåer** som stöds för Azure SQL Database finns i [Resursgränser för SQL-databas](../sql-database/sql-database-resource-limits.md).

```powershell
# Azure Data Factory version 2 information 
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$". 
$SubscriptionName = "<Azure subscription name>"
$ResourceGroupName = "<Azure resource group name>"
# Data factory name. Must be globally unique
$DataFactoryName = "<Data factory name>"
# You can create a data factory of version 2 in the following regions: East US, East US 2, Southeast Asia, and West Europe. 
$DataFactoryLocation = "EastUS" 

# Azure-SSIS integration runtime information. This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "<Specify a name for your Azure-SSIS IR>"
$AzureSSISDescription = "<Specify description for your Azure-SSIS IR"
# Azure-SSIS IR edition/license info: Standard or Enterprise 
$AzureSSISEdition = "Standard" # Enterprise Edition supports advanced/premium features

# You can create Azure-SSIS IR in the following regions: East US, East US 2, Central US, West US 2, North Europe, West Europe, UK South, Australia East. 
$AzureSSISLocation = "EastUS"
# In public preview, only Standard_A4_v2, Standard_A8_v2, Standard_D1_v2, Standard_D2_v2, Standard_D3_v2, Standard_D4_v2 are supported
$AzureSSISNodeSize = "Standard_D3_v2"
# In public preview, only 1-10 nodes are supported.
$AzureSSISNodeNumber = 2 
# For a Standard_D1_v2 node, 1-4 parallel executions per node are supported. For other nodes, it's 1-8.
$AzureSSISMaxParallelExecutionsPerNode = 2 

# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL: SAS URI of blob container where your custom setup script and its associated files are stored

# SSISDB info
$SSISDBServerEndpoint = "<Azure SQL server name>.database.windows.net"
$SSISDBServerAdminUserName = "<Azure SQL server - user name>"
$SSISDBServerAdminPassword = "<Azure SQL server - user password>"
# Remove the SSISDBPricingTier variable if you are using Azure SQL Managed Instance (Preview)
# This parameter applies only to Azure SQL Database. For the basic pricing tier, specify "Basic", not "B". For standard tiers, specify "S0", "S1", "S2", 'S3", etc.
$SSISDBPricingTier = "<pricing tier of your Azure SQL server. Examples: Basic, S0, S1, S2, S3, etc.>" 
```

## <a name="validate-the-connection-to-database"></a>Verifiera anslutningen till databasen
Lägg till följande skript för att verifiera Azure SQL Database-servern `<server>.database.windows.net`. 

```powershell
$SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID="+ $SSISDBServerAdminUserName +";Password="+ $SSISDBServerAdminPassword
$sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
Try
{
    $sqlConnection.Open();
}
Catch [System.Data.SqlClient.SqlException]
{
    Write-Warning "Cannot connect to your Azure SQL DB logical server/Azure SQL MI server, exception: $_"  ;
    Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
    $yn = Read-Host
    if(!($yn -ieq "Y"))
    {
        Return;
    } 
}
```

Se följande exempel för att skapa en Azure SQL-databas som en del av skriptet: 

Ange värden för de variabler som inte redan har definierats. Till exempel: FirewallIPAddress. 

```powershell
New-AzureRmSqlServer -ResourceGroupName $ResourceGroupName `
  -ServerName $SQLServerName `
    -Location $DataFactoryLocation `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $SQLServerAdmin, $(ConvertTo-SecureString -String $SQLServerPass -AsPlainText -Force))

New-AzureRmSqlServerFirewallRule -ResourceGroupName $ResourceGroupName `
    -ServerName $SQLServerName `
    -FirewallRuleName "ClientIPAddress_$today" -StartIpAddress $FirewallIPAddress -EndIpAddress $FirewallIPAddress

New-AzureRmSqlServerFirewallRule -ResourceGroupName $ResourceGroupName -ServerName $SQLServerName -AllowAllAzureIPs
```


## <a name="log-in-and-select-subscription"></a>Logga in och välj en prenumeration
Lägg till följande kod till skriptet för att logga in och välj din Azure-prenumeration: 

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $SubscriptionName
```

## <a name="create-a-resource-group"></a>Skapa en resursgrupp
Skapa en [Azure-resursgrupp](../azure-resource-manager/resource-group-overview.md) med kommandot [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup). En resursgrupp är en logisk behållare där Azure-resurser distribueras och hanteras som en grupp. I följande exempel skapas en resursgrupp med namnet `myResourceGroup` på platsen `westeurope`.

Om resursgruppen redan finns ska du inte kopiera den här koden i skriptet. 

```powershell
New-AzureRmResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
```

## <a name="create-a-data-factory"></a>Skapa en datafabrik
Skapa en datafabrik genom att köra följande kommandon:

```powershell
Set-AzureRmDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                        -Location $DataFactoryLocation `
                        -Name $DataFactoryName
```

## <a name="create-an-integration-runtime"></a>Skapa Integration Runtime
Kör följande kommando för att skapa en Azure-SSIS Integration Runtime som kör SSIS-paket i Azure: 

```powershell
$secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
$serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)
Set-AzureRmDataFactoryV2IntegrationRuntime  -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Type Managed `
                                            -CatalogServerEndpoint $SSISDBServerEndpoint `
                                            -CatalogAdminCredential $serverCreds `
                                            -CatalogPricingTier $SSISDBPricingTier `
                                            -Description $AzureSSISDescription `
                                            -Edition $AzureSSISEdition ` 
                                            -Location $AzureSSISLocation `
                                            -NodeSize $AzureSSISNodeSize `
                                            -NodeCount $AzureSSISNodeNumber `
                                            -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                            -SetupScriptContainerSasUri $SetupScriptContainerSasUri 
```

## <a name="start-integration-runtime"></a>Starta Integration Runtime
Kör följande kommando för att starta Azure-SSIS Integration Runtime: 

```powershell
write-host("##### Starting your Azure-SSIS integration runtime. This command takes 20 to 30 minutes to complete. #####")
Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")                                  
```
Det här kommandot tar mellan **20 till 30 minuter** att slutföra. 

## <a name="deploy-ssis-packages"></a>Distribuera SSIS-paket
Använd nu SQL Server Data Tools (SSDT) eller SQL Server Management Studio (SSMS) för att distribuera dina SSIS-paket till Azure. Anslut till Azure SQL-servern som är värd för SSIS-katalogen (SSISDB). Namnet på Azure SQL-servern är i formatet: `<servername>.database.windows.net` (för Azure SQL Database). 

Se följande artiklar från SSIS-dokumentationen: 

- [Distribuera, köra och övervaka ett SSIS-paket på Azure](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [Ansluta till SSIS-katalogen på Azure](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Schemalägga paketetkörning på Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)
- [Ansluta till lokala data med Windows-autentisering](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth) 

## <a name="full-script"></a>Fullständigt skript
I den här versionen måste du använda PowerShell för att etablera en instans för Azure-SSIS Integration Runtime som kör SSIS-paket i molnet. För närvarande går det inte att etablera den här körningen med hjälp av Azure-portalen. 

PowerShell-skriptet i det här avsnittet konfigurerar en instans av Azure-SSIS Integration Runtime i molnet som kör SSIS-paket. När du har kört det här skriptet kan du distribuera och köra SSIS-paket i Microsoft Azure-molnet med SSISDB i Azure SQL Database.

1. Starta Windows PowerShell ISE.
2. I ISE kör du följande kommando från kommandotolken.    
    ```powershell
    Set-ExecutionPolicy Unrestricted -Scope CurrentUser
    ```
3. Kopiera PowerShell-skriptet i det här avsnittet och klistra in det i ISE.
4. Ange lämpliga värden för skriptparametrarna i avsnittet med SSIS i Azure-specifikationer i början av skriptet. Dessa parametrar beskrivs i nästa avsnitt.
5. Kör skriptet. Kommandot `Start-AzureRmDataFactoryV2IntegrationRuntime` nära skriptets slut körs i **20 till 30 minuter**.

> [!NOTE]
> - Skriptet ansluts till din Azure SQL Database eller för att förbereda SSISDB. Skriptet konfigurerar också behörigheter och inställningar för ditt VNet, om det anges, och ansluter den nya instansen för Azure-SSIS Integration Runtime till VNet.
> - När du etablerar en instans av en Azure-SSIS IR installeras också Azure Feature Pack för SSIS och Access Redistributable. Med dessa komponenter upprättar du en anslutning till Excel- och Access-filer och till olika Azure-datakällor, utöver de datakällor som stöds av de inbyggda komponenterna. Du kan också installera ytterligare komponenter. Mer information finns i [Anpassad konfiguration för Azure-SSIS integreringskörning](how-to-configure-azure-ssis-ir-custom-setup.md).


En lista över **prisnivåer** som stöds för Azure SQL Database finns i [Resursgränser för SQL-databas](../sql-database/sql-database-resource-limits.md). 

En lista över regioner som stöds av Azure Data Factory V2 och Azure-SSIS Integration Runtime finns i [Tillgängliga produkter efter region](https://azure.microsoft.com/regions/services/). Expandera **Data och analys** för att se **Data Factory V2** och **SSIS Integration Runtime**.

```powershell
# Azure Data Factory version 2 information 
# If your input contains a PSH special character, e.g. "$", precede it with the escape character "`" like "`$". 
$SubscriptionName = "<Azure subscription name>"
$ResourceGroupName = "<Azure resource group name>"
# Data factory name. Must be globally unique
$DataFactoryName = "<Data factory name>" 
$DataFactoryLocation = "EastUS" 

# Azure-SSIS integration runtime information. This is a Data Factory compute resource for running SSIS packages
$AzureSSISName = "<Specify a name for your Azure-SSIS (IR)>"
$AzureSSISDescription = "<Specify description for your Azure-SSIS IR"
# Azure-SSIS IR edition/license info: Standard or Enterprise 
$AzureSSISEdition = "Standard" # Enterprise Edition supports advanced/premium features

$AzureSSISLocation = "EastUS" 
 # In public preview, only Standard_A4_v2, Standard_A8_v2, Standard_D1_v2, Standard_D2_v2, Standard_D3_v2, Standard_D4_v2 are supported
$AzureSSISNodeSize = "Standard_D3_v2"
# In public preview, only 1-10 nodes are supported.
$AzureSSISNodeNumber = 2 
# For a Standard_D1_v2 node, 1-4 parallel executions per node are supported. For other nodes, it's 1-8.
$AzureSSISMaxParallelExecutionsPerNode = 2 

# Custom setup info
$SetupScriptContainerSasUri = "" # OPTIONAL: SAS URI of blob container where your custom setup script and its associated files are stored

# SSISDB info
$SSISDBServerEndpoint = "<Azure SQL server name>.database.windows.net"
$SSISDBServerAdminUserName = "<Azure SQL server - user name>"
$SSISDBServerAdminPassword = "<Azure SQL server - user password>"
# Remove the SSISDBPricingTier variable if you are using Azure SQL Managed Instance (Preview)
# This parameter applies only to Azure SQL Database. For the basic pricing tier, specify "Basic", not "B". For standard tiers, specify "S0", "S1", "S2", 'S3", etc.
$SSISDBPricingTier = "<pricing tier of your Azure SQL server. Examples: Basic, S0, S1, S2, S3, etc.>" 

$SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID="+ $SSISDBServerAdminUserName +";Password="+ $SSISDBServerAdminPassword
$sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
Try
{
    $sqlConnection.Open();
}
Catch [System.Data.SqlClient.SqlException]
{
    Write-Warning "Cannot connect to your Azure SQL DB logical server/Azure SQL MI server, exception: $_"  ;
    Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
    $yn = Read-Host
    if(!($yn -ieq "Y"))
    {
        Return;
    } 
}

Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $SubscriptionName

Set-AzureRmDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                        -Location $DataFactoryLocation `
                        -Name $DataFactoryName

$secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
$serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)
Set-AzureRmDataFactoryV2IntegrationRuntime  -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Type Managed `
                                            -CatalogServerEndpoint $SSISDBServerEndpoint `
                                            -CatalogAdminCredential $serverCreds `
                                            -CatalogPricingTier $SSISDBPricingTier `
                                            -Description $AzureSSISDescription `
                                            -Edition $AzureSSISEdition ` 
                                            -Location $AzureSSISLocation `
                                            -NodeSize $AzureSSISNodeSize `
                                            -NodeCount $AzureSSISNodeNumber `
                                            -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                            -SetupScriptContainerSasUri $SetupScriptContainerSasUri

write-host("##### Starting your Azure-SSIS integration runtime. This command takes 20 to 30 minutes to complete. #####")
Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")
```

## <a name="join-azure-ssis-ir-to-a-vnet"></a>Anslut Azure-SSIS IR till ett virtuellt nätverk
Om du använder en Azure SQL-hanterad instans (förhandsversion) som värd för SQL Server Integration Services-katalogen (SSIS) i ett virtuellt nätverk (VNet) måste du även ansluta Azure-SSIS Integration Runtime till samma virtuella nätverk. Med Azure Data Factory version 2 (förhandsversion) kan du ansluta integreringskörningen för Azure-SSIS till ett virtuellt nätverk. Mer information finns i [Join Azure-SSIS runtime to a VNet](join-azure-ssis-integration-runtime-virtual-network.md) (Ansluta Azure-SSIS-körning till ett virtuellt nätverk).

Ett fullständigt skript för att skapa en Azure-SSIS-körning som ansluter till ett virtuellt nätverk finns i [Skapa en Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md).

## <a name="monitor-and-manage-azure-ssis-ir"></a>Övervaka och hantera Azure-SSIS IR
I följande artiklar finns information om att övervaka och hantera Azure-SSIS IR. 

- [Övervaka en Azure-SSIS Integration Runtime](monitor-integration-runtime.md#azure-ssis-integration-runtime)
- [Hantera en Azure-SSIS Integration Runtime](manage-azure-ssis-integration-runtime.md)

## <a name="next-steps"></a>Nästa steg
I den här självstudiekursen lärde du dig att: 

> [!div class="checklist"]
> * Skapa en datafabrik.
> * Skapa en Azure-SSIS Integration Runtime
> * Starta en Azure-SSIS Integration Runtime
> * Distribuera SSIS-paket
> * Granska det fullständiga skriptet

Fortsätt till följande självstudie och lär dig att kopiera data från en lokal plats till molnet: 

> [!div class="nextstepaction"]
>[Kopiera data i molnet](tutorial-copy-data-dot-net.md)
