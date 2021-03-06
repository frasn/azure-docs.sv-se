---
title: Hantera domänanslutna HDInsight - kluster i Azure
description: Lär dig att hantera domänanslutna HDInsight-kluster
services: hdinsight
author: omidm1
manager: jhubbard
editor: cgronlun
ms.assetid: 6ebc4d2f-2f6a-4e1e-ab6d-af4db6b4c87c
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: omidm
ms.openlocfilehash: 31b58176a75918cf68cda695f27216031670493e
ms.sourcegitcommit: e14229bb94d61172046335972cfb1a708c8a97a5
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/14/2018
---
# <a name="manage-domain-joined-hdinsight-clusters"></a>Hantera domänanslutna HDInsight-kluster
Läs om användarna och roller för domänanslutna HDInsight och hur du hanterar domänanslutna HDInsight-kluster.

## <a name="use-vscode-to-link-to-domain-joined-cluster"></a>Använda VSCode för att länka till domänanslutna kluster

Du kan länka en normal kluster med Ambari hanteras användarnamn, även länka ett säkerhet hadoop-kluster med hjälp av användarnamn (exempel: user1@contoso.com).
1. Öppna paletten kommandot genom att välja **CTRL + SKIFT + P**, och ange sedan **HDInsight: länka ett kluster**.

   ![länken cluster kommando](./media/apache-domain-joined-manage/link-cluster-command.png)

2. Ange HDInsight kluster-URL -> indata Username -> lösenordet -> Välj typ av kluster -> den visar lyckad info om verifiering skickades.
   
   ![länka klustret dialog](./media/apache-domain-joined-manage/link-cluster-process.png)

   > [!NOTE]
   > Länkade användarnamn och lösenord används om klustret både inloggad i Azure-prenumeration och länka ett kluster. 
   
3. Du kan se ett länkade kluster med hjälp av kommandot **listan kluster**. Nu kan du skicka ett skript för att den här länkade klustret.

   ![länkade kluster](./media/apache-domain-joined-manage/linked-cluster.png)

4. Du kan också Avlänka ett kluster genom att mata in **HDInsight: Avlänka ett kluster** kommandot paletten.

## <a name="use-intellij-to-link-to-domain-joined-cluster"></a>Använda IntelliJ för att länka till domänanslutna kluster

Du kan länka en normal kluster med Ambari hanteras användarnamn, även länka ett säkerhet hadoop-kluster med hjälp av användarnamn (exempel: user1@contoso.com). 
1. Klicka på **länka ett kluster** från **Azure Explorer**.

   ![länken klustrets snabbmenyn](./media/apache-domain-joined-manage/link-a-cluster-context-menu.png)

2. Ange **klusternamn**, **användarnamn** och **lösenord**. Du måste kontrollera användarnamn och lösenord om du har fått autentisering misslyckades. Du kan också lägga till Lagringskontot, nyckeln för Säkerhetslagring, och välj sedan en behållare lagringsbehållaren. Storage-informationen är för Lagringsutforskaren i vänster träd
   
   ![länka klustret dialog](./media/apache-domain-joined-manage/link-a-cluster-dialog.png)

   > [!NOTE]
   > Vi använder länkade lagringsnyckel, användarnamn och lösenord om klustret både inloggad i Azure-prenumeration och länka ett kluster.
   > ![Lagringsutforskaren i IntelliJ](./media/apache-domain-joined-manage/storage-explorer-in-IntelliJ.png)

   
3. Du kan se ett länkade kluster i **HDInsight** nod om informationen om indata är rätt. Nu kan du skicka ett program till den här länkade klustret.

   ![länkade kluster](./media/apache-domain-joined-manage/linked-cluster-intellij.png)

4. Du kan också Avlänka ett kluster från **Azure Explorer**.
   
   ![Olänkade kluster](./media/apache-domain-joined-manage/unlink.png)

## <a name="use-eclipse-to-link-to-domain-joined-cluster"></a>Använda Eclipse för att länka till domänanslutna kluster

Du kan länka en normal kluster med Ambari hanteras användarnamn, även länka ett säkerhet hadoop-kluster med hjälp av användarnamn (exempel: user1@contoso.com).
1. Klicka på **länka ett kluster** från **Azure Explorer**.

   ![länken klustrets snabbmenyn](./media/apache-domain-joined-manage/link-a-cluster-context-menu.png)

2. Ange **klusternamnet**, **användarnamn** och **lösenord**, klicka på OK för att länka klustret. Du kan också ange Lagringskonto, Lagringsnyckel och välj sedan lagringsbehållare som Lagringsutforskaren ska fungera i den vänstra trädvyn
   
   ![länka klustret dialog](./media/apache-domain-joined-manage/link-cluster-dialog.png)
   
   > [!NOTE]
   > Vi använder länkade lagringsnyckel, användarnamn och lösenord om klustret både inloggad i Azure-prenumeration och länka ett kluster.
   > ![Lagringsutforskaren i Eclipse](./media/apache-domain-joined-manage/storage-explorer-in-Eclipse.png)

3. Du kan se ett länkade kluster i **HDInsight** nod när du klickar på OK-knappen, om den inkommande informationen är rätt. Nu kan du skicka ett program till den här länkade klustret.

   ![länkade kluster](./media/apache-domain-joined-manage/linked-cluster-intellij.png)

4. Du kan också Avlänka ett kluster från **Azure Explorer**.
   
   ![Olänkade kluster](./media/apache-domain-joined-manage/unlink.png)

## <a name="access-the-clusters-with-enterprise-security-package"></a>Åtkomst till kluster med Enterprise säkerhetspaketet.

Enterprise-säkerhetspaketet (tidigare kallat HDInsight Premium) ger flera användare åtkomst till klustret, där autentisering görs av Active Directory och auktorisering av Apache Ranger och lagring ACL: er (ADLS-ACL: er). Auktorisering tillhandahåller säker gränser mellan flera användare och gör att enbart behöriga användare har åtkomst till data baserat på auktoriseringsprinciper.

Säkerhets- och isolering är viktiga för ett HDInsight-kluster med Enterprise säkerhetspaketet. SSH-åtkomst till klustret med Enterprise-säkerhetspaketet har blockerats för att uppfylla kraven. I följande tabell visas de rekommenderade åtkomstmetoder för varje typ av kluster:

|Arbetsbelastning|Scenario|Åtkomstmetod|
|--------|--------|-------------|
|Hadoop|Hive – interaktiva jobb-frågor |<ul><li>[Beeline](#beeline)</li><li>[Hive-vyn](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Powerbi](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Spark|Interaktiva jobb/frågor PySpark interaktiva|<ul><li>[Beeline](#beeline)</li><li>[Zeppelin med Livy](../spark/apache-spark-zeppelin-notebook.md)</li><li>[Hive-vyn](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Powerbi](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Spark|Batch-scenarier – Spark skicka PySpark|<ul><li>[Livy](../spark/apache-spark-livy-rest-interface.md)</li></ul>|
|Interaktiva fråga (LLAP)|Interaktiv|<ul><li>[Beeline](#beeline)</li><li>[Hive-vyn](../hadoop/apache-hadoop-use-hive-ambari-view.md)</li><li>[ODBC/JDBC – Powerbi](../hadoop/apache-hadoop-connect-hive-power-bi.md)</li><li>[Visual Studio Tools](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)</li></ul>|
|Alla|Installera anpassade program|<ul><li>[Skriptåtgärder](../hdinsight-hadoop-customize-cluster-linux.md)</li></ul>|

   > [!NOTE]
   > Jupyter är inte installerat/stöds i Enterprise-säkerhetspaketet.

Om du använder standard-API: er kan från säkerhetsperspektiv. Dessutom kan få du följande fördelar:

1.  **Hantering av** – du kan hantera din kod och automatisera jobb med hjälp av standard-API: er – Livius HS2 osv.
2.  **Granska** – med SSH, det går inte att granska vilka användare SSH hade till klustret. Detta skulle vara fallet när jobb skapas via standard-slutpunkter som de kan verkställas i kontexten för användaren. 



### <a name="beeline"></a>Använd Beeline 
Installera Beeline på din dator och ansluter via det offentliga internet kan använda följande parametrar: 

```
- Connection string: -u 'jdbc:hive2://<clustername>.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2'
- Cluster login name: -n admin
- Cluster login password -p 'password'
```

Om du har installerat lokalt Beeline och ansluta via ett virtuellt Azure-nätverk, kan du använda följande parametrar: 

```
- Connection string: -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Använd informationen i de hantera HDInsight med Ambari REST API-dokumentet för att hitta det fullständigt kvalificerade domännamnet för en headnode.














## <a name="users-of-domain-joined-hdinsight-clusters"></a>Användare av domänanslutna HDInsight-kluster
Ett HDInsight-kluster som inte är ansluten till domänen har två konton som skapas när klustret skapas:

* **Ambari admin**: det här kontot kallas även *Hadoop användare* eller *HTTP användaren*. Det här kontot kan användas för att logga in på Ambari när https://&lt;klusternamn >. azurehdinsight.net. Det kan också användas för att köra frågor på Ambari-vyer, köra jobb via externa verktyg (till exempel PowerShell, Templeton, Visual Studio) och autentisera med ODBC-drivrutinen och BI-verktyg (till exempel Excel, PowerBI eller Tableau).

En domänansluten HDInsight-kluster har tre nya användare utöver Ambari Admin.

* **Ranger admin**: det här kontot är lokalt administratörskonto för Apache Ranger. Det är inte en användare för active directory-domän. Det här kontot kan användas för att konfigurera principer och göra andra användare, Administratörer eller delegerade administratörer (så att användarna kan hantera principer). Som standard är användarnamnet *admin* och lösenordet är detsamma som Ambari administratörslösenord. Lösenordet kan uppdateras från sidan Inställningar i Ranger.
* **Klustret admin domänanvändare**: det här kontot är en active directory-domänanvändare utses Hadoop-kluster administratören inklusive Ambari och Ranger. Du måste ange den här användarens autentiseringsuppgifter när klustret skapas. Den här användaren har följande behörigheter:

  * Ansluta datorer till domänen och placera dem i Organisationsenheten som du anger när klustret skapas.
  * Skapa tjänstens huvudnamn i Organisationsenheten som du anger när klustret skapas.
  * Skapa omvänd DNS-poster.

    Observera de AD-användarna ha dessa privilegier.

    Det finns några slutpunkter i klustret (till exempel Templeton) som inte hanteras av Ranger och därför inte är säker. De här slutpunkterna är låsta för alla användare utom klustret domän administratörsanvändare.
* **Vanliga**: du kan ange flera active directory-grupper när klustret skapas. Användare i dessa grupper synkroniseras till Ranger och Ambari. Dessa användare har åtkomst till endast hanterade Ranger slutpunkter (till exempel Hiveserver2) är domänanvändare. Alla RBAC principer och granskning kommer att användas på dessa användare.

## <a name="roles-of-domain-joined-hdinsight-clusters"></a>Roller för domänanslutna HDInsight-kluster
Domänanslutna HDInsight har följande roller:

* Klusteradministratören
* Kluster-operatorn
* Tjänstadministratör
* Tjänsten Operator
* Kluster-användare

**Se behörigheter för dessa roller**

1. Öppna hanteringsgränssnittet för Ambari.  Se [öppna Ambari hanteringsgränssnittet](#open-the-ambari-management-ui).
2. I den vänstra menyn klickar du på **roller**.
3. Klicka på det blå frågetecknet visas behörigheterna:

    ![Behörigheter för domänanslutna HDInsight-roller](./media/apache-domain-joined-manage/hdinsight-domain-joined-roles-permissions.png)

## <a name="open-the-ambari-management-ui"></a>Öppna hanteringsgränssnittet för Ambari

1. Logga in på [Azure Portal](https://portal.azure.com).
2. Öppna ditt HDInsight-kluster. Se [listan och visa](../hdinsight-administer-use-management-portal.md#list-and-show-clusters).
3. Klicka på **instrumentpanelen** på den översta menyn för att öppna Ambari.
4. Logga in på Ambari med klustrets administratör domänanvändarnamn och lösenord.
5. Klicka på den **Admin** listrutan från längst upp till höger hörnet och klicka sedan på **hantera Ambari**.

    ![Domänanslutna HDInsight hantera Ambari](./media/apache-domain-joined-manage/hdinsight-domain-joined-manage-ambari.png)

    Det ser ut så Användargränssnittet:

    ![Domänanslutna HDInsight Ambari hanteringsgränssnittet](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui.png)

## <a name="list-the-domain-users-synchronized-from-your-active-directory"></a>Visa en lista med domänanvändare som synkroniseras från din Active Directory
1. Öppna hanteringsgränssnittet för Ambari.  Se [öppna Ambari hanteringsgränssnittet](#open-the-ambari-management-ui).
2. I den vänstra menyn klickar du på **användare**. Du bör se alla användare som synkroniseras från din Active Directory till HDInsight-klustret.

    ![Domänanslutna HDInsight Ambari management UI listan användare](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-users.png)

## <a name="list-the-domain-groups-synchronized-from-your-active-directory"></a>Visa en lista över de domängrupperna som synkroniseras från din Active Directory
1. Öppna hanteringsgränssnittet för Ambari.  Se [öppna Ambari hanteringsgränssnittet](#open-the-ambari-management-ui).
2. I den vänstra menyn klickar du på **grupper**. Du bör se alla grupper som synkroniseras från din Active Directory till HDInsight-klustret.

    ![Domänanslutna HDInsight Ambari UI lista hanteringsgrupper](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-groups.png)

## <a name="configure-hive-views-permissions"></a>Konfigurera behörigheter för Hive-vyer
1. Öppna hanteringsgränssnittet för Ambari.  Se [öppna Ambari hanteringsgränssnittet](#open-the-ambari-management-ui).
2. I den vänstra menyn klickar du på **vyer**.
3. Klicka på **HIVE** att visa detaljerna.

    ![Domänanslutna HDInsight Ambari management UI Hive-vyer](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views.png)
4. Klicka på den **Hive-vy** länk för att konfigurera Hive vyer.
5. Rulla ned till den **behörigheter** avsnitt.

    ![Konfigurera behörigheter för domänanslutna HDInsight Ambari management UI Hive-vyer](./media/apache-domain-joined-manage/hdinsight-domain-joined-ambari-management-ui-hive-views-permissions.png)
6. Klicka på **Lägg till användare** eller **Lägg till grupp**, och sedan ange användare eller grupper som kan använda Hive vyer.

## <a name="configure-users-for-the-roles"></a>Konfigurera användare för roller
 Om du vill se en lista över roller och deras behörigheter finns [roller för domänanslutna HDInsight-kluster](#roles-of-domain---joined-hdinsight-clusters).

1. Öppna hanteringsgränssnittet för Ambari.  Se [öppna Ambari hanteringsgränssnittet](#open-the-ambari-management-ui).
2. I den vänstra menyn klickar du på **roller**.
3. Klicka på **Lägg till användare** eller **Lägg till grupp** att tilldela användare och grupper för olika roller.

## <a name="next-steps"></a>Nästa steg
* Om du vill konfigurera ett domänanslutet HDInsight-kluster kan du läsa i [Konfigurera domänanslutna HDInsight-kluster](apache-domain-joined-configure.md).
* Om du vill konfigurera Hive-principer och köra Hive-frågor kan du läsa i [Konfigurera Hive-principer för domänanslutna HDInsight-kluster](apache-domain-joined-run-hive.md).
