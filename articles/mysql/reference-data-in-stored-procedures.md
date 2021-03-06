---
title: Azure-databas för MySQL Data i replikering lagrade procedurer
description: Den här artikeln introducerar alla lagrade procedurer som används för Data i replikering.
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.topic: article
ms.date: 05/18/2018
ms.openlocfilehash: 8683aacb2ae41cb4c57aa70b375eca9db6106492
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34655667"
---
# <a name="azure-database-for-mysql-data-in-replication-stored-procedures"></a>Azure-databas för MySQL Data i replikering lagrade procedurer

Data i replikering kan du synkronisera data från en MySQL-server som kör lokalt, i virtuella datorer eller databastjänster hos andra molntjänstleverantörer till Azure-databas för MySQL-tjänst.

Följande lagrade procedurer för att ställa in eller ta bort Data i replikeringen mellan en primär och repliken.

|**Namnet på lagrade proceduren**|**Indataparametrar**|**Utdataparametrar**|**Observera användning**|
|-----|-----|-----|-----|
|*MySQL.az_replication_change_primary*|master_host<br/>master_user<br/>master_password<br/>master_port<br/>master_log_file<br/>master_log_pos<br/>master_ssl_ca|Gäller inte|Skicka i kontexten för CA-certifikatet till parametern master_ssl_ca för att överföra data med SSL-läge. </br><br>Skicka in en tom sträng i master_ssl_ca-parameter för att överföra data utan SSL.|
|*_starta MySQL.az_replication*|Gäller inte|Gäller inte|Startar replikering.|
|*MySQL.az_replication _stop*|Gäller inte|Gäller inte|Stoppar replikering.|
|*MySQL.az_replication _remove_primary*|Gäller inte|Gäller inte|Tar bort replikeringsrelationen mellan den primära servern och repliken.|
|*MySQL.az_replication_skip_counter*|Gäller inte|Gäller inte|Hoppar över en replikeringsfel.|

Om du vill konfigurera data i replikering mellan en primär och en replik i Azure-databas för MySQL avser [hur du konfigurerar replikering data i](howto-data-in-replication.md).