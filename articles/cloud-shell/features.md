---
title: Bash i Azure Cloud Shell funktioner | Microsoft Docs
description: Översikt över funktionerna i Bash i Azure Cloud Shell
services: Azure
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/15/2017
ms.author: juluk
ms.openlocfilehash: 3e71f2307ab68b2bc4694c7782f7a0f0d43b159b
ms.sourcegitcommit: 4f9fa86166b50e86cf089f31d85e16155b60559f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34757050"
---
# <a name="features--tools-for-bash-in-azure-cloud-shell"></a>Funktioner och verktyg för Bash i Azure Cloud Shell

[!INCLUDE [features-introblock](../../includes/cloud-shell-features-introblock.md)]

> [!TIP]
> Funktioner och verktyg i [PowerShell](features-powershell.md) är också tillgänglig.

Bash i molnet Shell körs på `Ubuntu 16.04 LTS`.

## <a name="features"></a>Funktioner

### <a name="secure-automatic-authentication"></a>Säker automatisk autentisering

Bash i molnet Shell autentiserar på ett säkert sätt och automatiskt kontoåtkomst för Azure CLI 2.0.

### <a name="ssh-into-azure-linux-virtual-machines"></a>SSH i virtuella Azure Linux-datorer

Skapa en Linux VM från Azure CLI 2.0 kan skapa en standard SSH-nyckel och placera den i din `$Home` directory. Placera SSH-nycklar i `$Home` möjliggör SSH-anslutningar till virtuella Azure Linux-datorer direkt från molnet Shell. Nycklar förvaras i acc_<user>.img i din filresurs använder bästa metoder när du använder eller dela åtkomst till filresursen eller nycklar.

### <a name="home-persistence-across-sessions"></a>$Home beständiga mellan sessioner

För att bevara filer mellan sessioner vägleder dig genom att koppla en Azure-filresursen startas för första i molnet Shell.
När slutfört, koppla molnet Shell automatiskt din lagring (monterade som `$Home\clouddrive`) för alla framtida sessioner.
Dessutom i Bash i molnet Shell din `$Home` directory sparas som en .img i Azure-filresurs.
Filer utanför `$Home` och datorns tillstånd är inte beständiga mellan sessioner.

[Läs mer om spara filer i Bash i molnet Shell.](persisting-shell-storage.md)

### <a name="integration-with-open-source-tooling"></a>Integrering med öppen källkod verktygsuppsättning

Bash i molnet Shell innehåller förkonfigurerade autentisering för öppen källkod verktyg som Terraform, Ansible och InSpec Chef. Prova från exempel genomgång.

## <a name="tools"></a>Verktyg

|Kategori   |Namn   |
|---|---|
|Linux-verktyg            |Bash<br> del<br> tmux<br> gräva<br>               |
|Azure-verktyg            |[Azure CLI 2.0](https://github.com/Azure/azure-cli) och [1.0](https://github.com/Azure/azure-xplat-cli)<br> [AzCopy](https://docs.microsoft.com/azure/storage/storage-use-azcopy)<br> [Batch Shipyard](https://github.com/Azure/batch-shipyard) <br> [Service Fabric CLI](https://docs.microsoft.com/azure/service-fabric/service-fabric-cli) <br> [blobxfer](https://github.com/Azure/blobxfer#blobxfer) |
|Textredigerare           |VIM<br> nano<br> emacs       |
|Källkontrollen         |Git                    |
|Skapa verktyg            |Kontrollera<br> maven<br> npm<br> PIP         |
|Behållare             |[Docker CLI](https://github.com/docker/cli)/[Docker-dator](https://github.com/docker/machine)<br> [Kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)<br> [Helm](https://github.com/kubernetes/helm)<br> [DC/OS CLI](https://github.com/dcos/dcos-cli)         |
|Databaser              |MySQL-klient<br> PostgreSql-klient<br> [SQLCMD-verktyget](https://docs.microsoft.com/sql/tools/sqlcmd-utility)<br> [mssql-scripter](https://github.com/Microsoft/sql-xplat-cli) |
|Annat                  |iPython klienten<br> [Molnet Foundry CLI](https://github.com/cloudfoundry/cli)<br> [Terraform](https://www.terraform.io/docs/providers/azurerm/)<br> [Ansible](https://www.ansible.com/microsoft-azure)<br> [Chef InSpec](https://www.chef.io/inspec/)| 

## <a name="language-support"></a>Språkstöd

|Språk   |Version   |
|---|---|
|.NET       |2.0.0       |
|Go         |1.9        |
|Java       |1.8        |
|Node.js    |8.9.4      |
|PowerShell |[6.0.2](https://github.com/PowerShell/powershell/releases)       |
|Python     |2.7 och 3.5 (standard)|

## <a name="next-steps"></a>Nästa steg
[Bash i molnet Shell Snabbstart](quickstart.md) <br>
[Lär dig mer om Azure CLI 2.0](https://docs.microsoft.com/cli/azure/)
