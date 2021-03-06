---
title: Nätverkskonfigurationen i Azure-Kubernetes (AKS)
description: Läs mer om grundläggande och avancerade nätverkskonfigurationen i Azure-Kubernetes (AKS).
services: container-service
author: mmacy
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 06/04/2018
ms.author: marsma
ms.openlocfilehash: d6f42a5f3ce907fdb759bef29ca25bdc7fe365d9
ms.sourcegitcommit: 4f9fa86166b50e86cf089f31d85e16155b60559f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/04/2018
ms.locfileid: "34757016"
---
# <a name="network-configuration-in-azure-kubernetes-service-aks"></a>Nätverkskonfigurationen i Azure-Kubernetes (AKS)

När du skapar ett kluster med Azure Kubernetes Service (AKS) du kan välja mellan två alternativ för nätverksfunktioner: **grundläggande** eller **Avancerat**.

## <a name="basic-networking"></a>Grundläggande nätverk

Den **grundläggande** nätverk alternativet är standardkonfigurationen för AKS klustret skapas. Nätverkskonfigurationen för klustret och dess skida hanteras helt av Azure och är lämplig för distributioner som inte kräver anpassad konfiguration av virtuellt nätverk. Du har inte kontroll över nätverkskonfigurationen som undernät eller IP-adressintervall som är tilldelade till klustret när du väljer grundläggande nätverk.

Noder i ett kluster som AKS är konfigurerad för enkla nätverk på [kubenet] [ kubenet] Kubernetes plugin-programmet.

## <a name="advanced-networking"></a>Avancerade nätverk

**Avancerade** nätverk placeras din skida i ett Azure Virtual Network (VNet) som du konfigurerar att ge dem automatisk anslutning till VNet resurser och integrering med omfattande uppsättning funktioner som Vnet-erbjudandet.
Avancerade nätverk är tillgänglig endast när du distribuerar AKS kluster i den [Azure-portalen] [ portal] eller med en Resource Manager-mall.

Noder i ett kluster som AKS är konfigurerad för avancerade nätverk den [Azure Container nätverk gränssnitt (CNI)] [ cni-networking] Kubernetes plugin-programmet.

![Diagram över två noder med bryggor ansluter dem till ett enda Azure VNet][advanced-networking-diagram-01]

## <a name="advanced-networking-features"></a>Avancerade nätverksfunktioner

Avancerade nätverk ger följande fördelar:

* Distribuera AKS-kluster till ett befintligt virtuellt nätverk eller skapa en ny VNet och undernät för klustret.
* Varje baljor i klustret tilldelas en IP-adress i virtuella nätverk och kan kommunicera direkt med andra skida i klustret och andra noder i virtuella nätverk.
* En baljor kan ansluta till andra tjänster i ett peered VNet och lokala nätverk via ExpressRoute och plats-till-plats (S2S) VPN-anslutningar. Skida kan även nås från lokalt.
* Exponera en Kubernetes tjänst externt eller internt via Azure belastningsutjämnare. Också en funktion i grundläggande nätverk.
* Skida i ett undernät som har aktiverat slutpunkter kan på ett säkert sätt ansluta till Azure-tjänster, till exempel Azure Storage- och SQL-databas.
* Använd användardefinierade vägar (UDR) att dirigera trafik från skida till en virtuell nätverksenhet.
* Skida kan komma åt resurser på Internet. Också en funktion i grundläggande nätverk.

> [!IMPORTANT]
> Varje nod i ett kluster för AKS som konfigurerats för avancerade nätverk kan vara värd för maximalt **30 skida**. Varje VNet som etablerats för användning med Azure CNI plugin-programmet är begränsad till **4096 konfigurerade IP-adresser**.

## <a name="advanced-networking-prerequisites"></a>Avancerade krav för nätverk

* Virtuella nätverk för AKS klustret måste tillåta utgående Internetanslutning.
* Skapa inte fler än ett AKS kluster i samma undernät.
* Avancerade nätverk för AKS stöder inte Vnet som använder Azure privata DNS-zoner.
* AKS kluster får inte använda `169.254.0.0/16`, `172.30.0.0/16`, eller `172.31.0.0/16` för Kubernetes service-adressintervall.
* Tjänstens huvudnamn för AKS klustret måste ha `Contributor` behörigheter till den resursgrupp som innehåller befintliga VNet.

## <a name="plan-ip-addressing-for-your-cluster"></a>Planera IP-adresser för klustret

Kluster som har konfigurerats med avancerade nätverk kräver ytterligare planering. Storleken på ditt VNet och dess undernät måste hantera både antalet skida som du planerar att köra samt antalet noder för klustret.

IP-adresser för skida och den klusternoder tilldelas från det angivna undernätet inom VNet. Varje nod har konfigurerats med en primär IP-adress, vilket är den IP-Adressen på noden och 30 ytterligare IP-adresser förkonfigurerade genom Azure CNI som har tilldelats skida schemalagda till noden. När du skalar upp ditt kluster, konfigureras varje nod på samma sätt med IP-adresser från undernätet.

IP-adress planen för en AKS klustret består av ett VNet, minst ett undernät för noderna och skida, och ett Kubernetes service-adressintervall.

| Adressintervall / Azure resurs | Gränser och storlek |
| --------- | ------------- |
| Virtuellt nätverk | Azure-VNet kan vara så stor som/8 som men kan endast ha 4096 konfigurerade IP-adresser. |
| Undernät | Måste vara tillräckligt stor för att hantera noder och skida. Att beräkna storleken på din minsta undernät: (antal noder) + (antalet noder * skida per nod). I ett kluster 50 noder: (50) + (50 * 30) = 1,550, ditt undernät måste vara en /21 eller större. |
| Kubernetes service-adressintervall | Det här intervallet bör inte används av nätverket element på eller ansluten till detta virtuella nätverk. Serviceadressen CIDR måste vara mindre än /12. |
| IP-adress för Kubernetes DNS-tjänsten | IP-adress inom Kubernetes service-adressintervall som ska användas av klustertjänstidentifiering (kube dns). |
| Docker bridge adress | IP-adress (CIDR-notation) som används som Docker-brygga IP-adress på noder. Standardvärdet för 172.17.0.1/16. |

Som nämnts tidigare varje VNet som etablerats för användning med Azure CNI plugin-programmet är begränsad till **4096 konfigurerade IP-adresser**. Varje nod i ett kluster som har konfigurerats för avancerade nätverk kan vara värd för maximalt **30 skida**.

## <a name="configure-advanced-networking"></a>Konfigurera avancerade nätverk

När du [skapa ett kluster som AKS](kubernetes-walkthrough-portal.md) i Azure-portalen följande parametrar kan konfigureras för avancerade nätverk:

**Virtuellt nätverk**: det VNet som du vill distribuera Kubernetes-kluster. Om du vill skapa ett nytt virtuellt nätverk för klustret, väljer *Skapa nytt* och följer anvisningarna i den *skapa virtuellt nätverk* avsnitt.

**Undernät**: undernätet inom VNet där du vill distribuera klustret. Om du vill skapa ett nytt undernät i VNet för klustret, väljer *Skapa nytt* och följer anvisningarna i den *skapa undernät* avsnitt.

**Kubernetes tjänsten adressintervallet**: IP-adressintervall för Kubernetes klustrets IP-adresser. Det här intervallet får inte vara inom VNet IP-adressintervall i klustret.

**IP-adress för Kubernetes DNS-tjänsten**: IP-adress för DNS-klustertjänsten. Den här adressen måste vara inom den *Kubernetes tjänsten adressintervallet*.

**Docker Bridge adress**: IP-adress och nätmask tilldelas till Docker-brygga. Denna IP-adress får inte vara inom VNet IP-adressintervall i klustret.

Följande skärmbild från Azure-portalen visar ett exempel på hur du konfigurerar de här inställningarna när AKS klustret skapas:

![Avancerade nätverkskonfigurationen i Azure-portalen][portal-01-networking-advanced]

## <a name="frequently-asked-questions"></a>Vanliga frågor och svar

Följande frågor och svar som gäller för den **Avancerat** nätverkskonfigurationen.

* *Kan jag konfigurera avancerade nätverk med Azure CLI?*

  Nej. Avancerade nätverk är tillgänglig endast när distribuera AKS kluster i Azure-portalen eller med en Resource Manager-mall.

* *Kan jag distribuera virtuella datorer i mitt undernät för klustret?*

  Nej. Distribuera virtuella datorer i undernätet som används av klustret Kubernetes stöds inte. Virtuella datorer kan distribueras i samma virtuella nätverk, men i ett annat undernät.

* *Kan jag konfigurera per baljor nätverksprinciper?*

  Nej. Per baljor nätverksprinciper stöds för närvarande inte.

* *Är det maximala antalet skida distribueras till en konfigurerbar nod?*

  Som standard kan varje nod vara värd för högst 30 skida. Du kan ändra det högsta värdet genom att ändra den `maxPods` egenskapen när du distribuerar ett kluster med en Resource Manager-mall.

* *Konfigurera ytterligare egenskaper för det undernät som jag har skapat när klustret skapas AKS? Till exempel tjänstens slutpunkter.*

  En fullständig lista över egenskaper för virtuellt nätverk och undernät som du skapar under AKS klustret har skapats kan konfigureras i standard VNet konfigurationssidan i Azure-portalen.

## <a name="next-steps"></a>Nästa steg

### <a name="networking-in-aks"></a>Nätverk i AKS

Mer information om nätverk i AKS i följande artiklar:

[Använda en statisk IP-adress med belastningsutjämnaren Azure Kubernetes Service (AKS)](static-ip.md)

[HTTPS-ingång på Azure-behållaren (AKS)](ingress.md)

[Använda en intern belastningsutjämnare med Azure Container Service (AKS)](internal-lb.md)

### <a name="acs-engine"></a>ACS-motorn

[Azure Container Service Engine (ACS Engine)] [ acs-engine] är ett projekt med öppen källkod som genererar Azure Resource Manager-mallar som du kan använda för att distribuera Docker-aktiverade kluster på Azure. Kubernetes, DC/OS, Swarm-läge och Swarm orchestrators kan distribueras med ACS-motorn.

Kubernetes kluster som skapas med ACS-motorn stöd för både den [kubenet] [ kubenet] och [Azure CNI] [ cni-networking] plugin-program. Därför stöds både grundläggande och avancerade nätverksscenarier av ACS-motorn.

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[acs-engine]: https://github.com/Azure/acs-engine
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[portal]: https://portal.azure.com

<!-- LINKS - Internal -->
[aks-ssh]: aks-ssh.md
