---
title: "Konfigurera parametrar för tjänsten i Azure-databas för MySQL"
description: "Den här artikeln beskriver hur du konfigurerar parametrar för tjänsten i Azure-databas för MySQL med hjälp av kommandoradsverktyget Azure CLI."
services: mysql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.devlang: azure-cli
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 9caf6b164f2433ab3c1b701554f562211cc2de80
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 02/28/2018
---
# <a name="customize-server-configuration-parameters-by-using-azure-cli"></a>Anpassa parametrar för konfiguration av servern med hjälp av Azure CLI
Du kan visa, visa och uppdatera konfigurationsparametrar för en Azure-databas för MySQL-servern med hjälp av Azure CLI, Azure kommandoradsverktyget. En delmängd av motorkonfigurationer är exponerad på servernivå och kan ändras. 

## <a name="prerequisites"></a>Förutsättningar
Du behöver följande för att gå igenom den här instruktioner:
- [En Azure-databas för MySQL-server](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI 2.0](/cli/azure/install-azure-cli) -kommandoradsverktyget eller Använd Azure Cloud-gränssnittet i webbläsaren.

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>Lista server konfigurationsparametrar för Azure-databas för MySQL-server
Om du vill visa en lista med alla parametrar som kan ändras i en server och deras värden, kör den [az mysql server konfigurationslistan](/cli/azure/mysql/server/configuration#az_mysql_server_configuration_list) kommando.

Du kan ange serverns konfigurationsparametrar för servern **mydemoserver.mysql.database.azure.com** under resursgrupp **myresourcegroup**.
```azurecli-interactive
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```
Definition av var och en av parametrarna i listan finns i avsnittet MySQL referens på [Server systemvariabler](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html).

## <a name="show-server-configuration-parameter-details"></a>Visa serverkonfiguration parameterinformation
Om du vill visa information om en specifik konfigurationsparameter för en server som kör den [az mysql server configuration visa](/cli/azure/mysql/server/configuration#az_mysql_server_configuration_show) kommando.

Det här exemplet visas information om den **långsam\_frågan\_loggen** server konfigurationsparameter för server **mydemoserver.mysql.database.azure.com** under resursgrupp **myresourcegroup.**
```azurecli-interactive
az mysql server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-a-server-configuration-parameter-value"></a>Ändra ett parametervärde för server-konfiguration
Du kan också ändra värdet för en viss server konfigurationsparameter, som uppdaterar det underliggande Konfigurationsvärdet som för server-databasmotorn MySQL. Uppdatera konfigurationen med den [az mysql server configuration set](/cli/azure/mysql/server/configuration#az_mysql_server_configuration_set) kommando. 

Uppdatera den **långsam\_frågan\_loggen** server konfigurationsparameter Server **mydemoserver.mysql.database.azure.com** under resursgruppen  **myresourcegroup.**
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```
Om du vill återställa värdet för en konfigurationsparameter utelämna den valfria `--value` parameter och tjänsten används standardvärdet. Till exempel ovan ser ut som:
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
Den här koden återställer den **långsam\_frågan\_loggen** konfiguration till standardvärdet **OFF**. 

## <a name="next-steps"></a>Nästa steg
- Så här konfigurerar du [serverparametrar i Azure-portalen](howto-server-parameters.md)