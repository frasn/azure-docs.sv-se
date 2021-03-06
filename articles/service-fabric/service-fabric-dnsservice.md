---
title: Azure Service Fabric DNS-tjänsten | Microsoft Docs
description: Använda Service Fabric DNS-tjänsten för identifiering av mikrotjänster från i klustret.
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: vturecek
ms.assetid: 47f5c1c1-8fc8-4b80-a081-bc308f3655d3
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 7/27/2017
ms.author: msfussell
ms.openlocfilehash: 656aed1f1fbd3294c4318520058ace480fd2219c
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/16/2018
---
# <a name="dns-service-in-azure-service-fabric"></a>DNS-tjänsten i Azure Service Fabric
DNS-tjänsten är en valfri systemtjänst som du kan aktivera i klustret för att identifiera andra tjänster med hjälp av DNS-protokollet. 

Många tjänster, särskilt av tjänster, kan ha ett befintligt URL-namn och att kunna lösa dem med hjälp av DNS-standardprotokollet (i stället för protokollet Naming Service) är önskvärt, särskilt i ”lyfta och flytta” scenarier. DNS-tjänsten kan du mappa DNS-namn till ett namn och därför matcha IP-adresser för slutpunkt. 

Tjänsten DNS matchar DNS-namn till tjänstnamn som i sin tur kan matchas med namngivningstjänst att returnera tjänstslutpunkten. DNS-namn för tjänsten tillhandahålls vid tidpunkten för skapandet.

![slutpunkter](./media/service-fabric-dnsservice/dns.png)

Dynamiska portar stöds inte av DNS-tjänsten. Lös tjänster som exponeras av dynamiska portar med de [omvänd proxy-tjänsten](./service-fabric-reverseproxy.md).

## <a name="enabling-the-dns-service"></a>Aktivera DNS-tjänsten
När du skapar ett kluster med hjälp av portalen DNS-tjänsten är aktiverad som standard i den **inkluderar DNS-tjänsten** kryssrutan på den **klusterkonfigurationen** menyn:

![Aktivera DNS-tjänsten via portalen][2]

Om du inte använder portalen för att skapa klustret eller om du uppdaterar ett befintligt kluster, behöver du aktivera tjänsten DNS i en mall:

- Om du vill distribuera ett nytt kluster som du kan använda den [exempel mallar](https://github.com/Azure/azure-quickstart-templates/tree/master/service-fabric-secure-cluster-5-node-1-nodetype) eller skapa en egen Resource Manager-mall. 
- För att uppdatera ett befintligt kluster, kan du gå till resursgruppen på portalen och klicka på **Automatiseringsskriptet** att arbeta med en mall som motsvarar det aktuella tillståndet för klustret och andra resurser i gruppen. Läs mer i [Exportera mallen från resursgruppen](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-export-template#export-the-template-from-resource-group).

När du har en mall kan aktivera du DNS-tjänsten med följande steg:

1. Kontrollera att den `apiversion` är inställd på `2017-07-01-preview` eller senare för att den `Microsoft.ServiceFabric/clusters` resursen, och om inte, uppdatera det som visas i följande fragment:

    ```json
    {
        "apiVersion": "2017-07-01-preview",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        ...
    }
    ```

2. Nu aktivera DNS-tjänsten genom att lägga till följande `addonFeatures` avsnittet efter den `fabricSettings` avsnittet som visas i följande utdrag: 

    ```json
        "fabricSettings": [
        ...      
        ],
        "addonFeatures": [
            "DnsService"
        ],
    ```

3. När du har uppdaterat mallen för kluster med föregående ändringarna kan använda dem och låta uppgraderingen slutförts. När uppgraderingen är klar startar tjänsten DNS system körs i klustret. Tjänstnamnet är `fabric:/System/DnsService`, och du hittar den under den **System** avsnitt i Service Fabric explorer-tjänsten. 


## <a name="setting-the-dns-name-for-your-service"></a>Ange ett DNS-namn för tjänsten
När DNS-tjänsten körs i klustret, kan du ange ett DNS-namn för dina tjänster deklarativt för standardtjänster i antingen den `ApplicationManifest.xml` eller via PowerShell-kommandon.

DNS-namnet för din tjänst måste matchas i hela klustret. Vi rekommenderar att du använder en namngivningsschemat för `<ServiceDnsName>.<AppInstanceName>`, till exempel `service1.application1`. På så sätt att DNS-namnet i hela klustret unikt. Om ett program distribueras med Docker compose, tjänster tilldelas automatiskt med hjälp av den här namngivningsschemat DNS-namn.

### <a name="setting-the-dns-name-for-a-default-service-in-the-applicationmanifestxml"></a>Ange DNS-namnet för en standardtjänst i ApplicationManifest.xml
Öppna projektet i Visual Studio eller ditt favoritprogram redigerare och öppna den `ApplicationManifest.xml` filen. Gå till avsnittet standard tjänster och för varje tjänst lägga till den `ServiceDnsName` attribut. I följande exempel visas hur du anger DNS-namnet på tjänsten `service1.application1`

```xml
    <Service Name="Stateless1" ServiceDnsName="service1.application1">
    <StatelessService ServiceTypeName="Stateless1Type" InstanceCount="[Stateless1_InstanceCount]">
        <SingletonPartition />
    </StatelessService>
    </Service>
```
När programmet har distribuerats, service-instans i Service Fabric explorer visar DNS-namn för den här instansen, som visas i följande bild: 

![slutpunkter][1]

### <a name="setting-the-dns-name-for-a-service-using-powershell"></a>Ange DNS-namnet för en tjänst med Powershell
Du kan ange DNS-namnet för en tjänst när du skapar den med hjälp av den `New-ServiceFabricService` Powershell. I följande exempel skapas en ny tillståndslös tjänst med DNS-namn `service1.application1`

```powershell
    New-ServiceFabricService `
    -Stateless `
    -PartitionSchemeSingleton `
    -ApplicationName `fabric:/application1 `
    -ServiceName fabric:/application1/service1 `
    -ServiceTypeName Service1Type `
    -InstanceCount 1 `
    -ServiceDnsName service1.application1
```

## <a name="using-dns-in-your-services"></a>Med hjälp av DNS i dina tjänster
Om du distribuerar mer än en tjänst hittar du slutpunkter av andra tjänster kan kommunicera med genom att använda en DNS-namn. DNS-tjänsten kan bara användas för tillståndslösa tjänster, eftersom DNS-protokollet inte kan kommunicera med tillståndskänsliga tjänster. För tillståndskänsliga tjänster, kan du använda inbyggt [omvänd proxy-tjänsten](./service-fabric-reverseproxy.md) för http-anrop att anropa en viss tjänst partition. Dynamiska portar stöds inte av DNS-tjänsten. Du kan använda omvänd proxy för att lösa tjänster som använder dynamiska portar.

Följande kod visar hur du anropar en annan tjänst som ger helt enkelt en vanliga http-anropet du porten och en valfri sökväg som en del av URL: en.

```csharp
public class ValuesController : Controller
{
    // GET api
    [HttpGet]
    public async Task<string> Get()
    {
        string result = "";
        try
        {
            Uri uri = new Uri("http://service1.application1:8080/api/values");
            HttpClient client = new HttpClient();
            var response = await client.GetAsync(uri);
            result = await response.Content.ReadAsStringAsync();
            
        }
        catch (Exception e)
        {
            Console.Write(e.Message);
        }

        return result;
    }
}
```

## <a name="next-steps"></a>Nästa steg
Mer information om kommunikation inom klustret med [ansluta och kommunicera med tjänster](service-fabric-connect-and-communicate-with-services.md)

[0]: ./media/service-fabric-connect-and-communicate-with-services/dns.png
[1]: ./media/service-fabric-dnsservice/servicefabric-explorer-dns.PNG
[2]: ./media/service-fabric-dnsservice/DNSService.PNG
