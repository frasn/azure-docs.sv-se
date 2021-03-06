---
title: Självstudie om Kubernetes i Azure – Distribuera program
description: Självstudie om AKS – Distribuera program
services: container-service
author: neilpeterson
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 02/22/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: e992d7ca455ad4d95d0f10a94c6c9ce8055f8286
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/11/2018
---
# <a name="tutorial-run-applications-in-azure-kubernetes-service-aks"></a>Självstudie: Köra program i Azure Kubernetes Service (AKS)

I den här självstudien, som är del fyra av åtta, distribuerar du ett exempelprogram till ett Kubernetes-kluster. Det här är några av stegen:

> [!div class="checklist"]
> * Uppdatera Kubernetes-manifestfiler
> * Köra programmet i Kubernetes
> * Testa programmet

I efterföljande självstudier skalar du ut programmet, uppdaterar det och konfigurerar Log Analytics för att övervaka Kubernetes-klustret.

I den här självstudien förutsätter vi att du har grundläggande kunskaper om vanliga Kubernetes-begrepp. Detaljerad information om Kubernetes finns i [Kubernetes-dokumentationen][kubernetes-documentation].

## <a name="before-you-begin"></a>Innan du börjar

I tidigare självstudier paketerades ett program i en behållaravbildning, avbildningen laddades upp till Azure Container Registry och ett Kubernetes-kluster skapades.

I den här självstudien behöver du Kubernetes-manifestfilen `azure-vote-all-in-one-redis.yaml` som skapats i förväg. Den här filen laddades ned med källkoden för programmet i en tidigare självstudie. Kontrollera att du har klonat lagringsplatsen och att du har ändrat katalogerna i den klonade lagringsplatsen.

Om du inte har gjort det här och vill följa med återgår du till [Självstudie 1 – Skapa behållaravbildningar][aks-tutorial-prepare-app].

## <a name="update-manifest-file"></a>Uppdatera manifestfilen

Azure Container Registry (ACR) har använts till att lagra en behållaravbildning i den här självstudien. Innan du kör programmet måste du uppdatera namnet på ACR-inloggningsservern i Kubernetes-manifestfilen.

Du hämtar namnet på ACR-inloggningsservern med kommandot [az acr list][az-acr-list].

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

Manifestfilen har skapats i förväg med inloggningsservern `microsoft`. Öppna filen med valfritt redigeringsprogram. I det här exemplet öppnas filen med `nano`.

```console
nano azure-vote-all-in-one-redis.yaml
```

Ersätt `microsoft` med namnet på ACR-inloggningsservern. Du hittar värdet på rad **47** i manifestfilen.

```yaml
containers:
- name: azure-vote-front
  image: microsoft/azure-vote-front:v1
```

Koden ovan blir då.

```yaml
containers:
- name: azure-vote-front
  image: <acrName>.azurecr.io/azure-vote-front:v1
```

Spara och stäng filen.

## <a name="deploy-application"></a>Distribuera programmet

Använd kommandot [kubectl apply] [ kubectl-apply] för att köra programmet. Det här kommandot parsar manifestfilen och skapar de definierade Kubernetes-objekten.

```azurecli
kubectl apply -f azure-vote-all-in-one-redis.yaml
```

Resultat:

```
deployment "azure-vote-back" created
service "azure-vote-back" created
deployment "azure-vote-front" created
service "azure-vote-front" created
```

## <a name="test-application"></a>Testa programmet

En [Kubernetes-tjänst][kubernetes-service] skapas som gör programmet synligt på internet. Den här processen kan ta ett par minuter.

Du kan övervaka förloppet genom att använda kommandot [kubectl get service][kubectl-get] med argumentet `--watch`.

```azurecli
kubectl get service azure-vote-front --watch
```

Till en början visas *EXTERNAL-IP* för *azure-vote-front*-tjänsten som *pending* (väntande).

```
azure-vote-front   10.0.34.242   <pending>     80:30676/TCP   7s
```

När *EXTERNAL-IP*-adressen har ändrats från *pending* till en *IP-adress* använder du `CTRL-C` för att stoppa kubectl-övervakningsprocessen.

```
azure-vote-front   10.0.34.242   52.179.23.131   80:30676/TCP   2m
```

Bläddra till den externa IP-adressen för att visa programmet.

![Bild av Kubernetes-kluster i Azure](media/container-service-kubernetes-tutorials/azure-vote.png)

Om det inte gick att läsa in programmet kan det bero på ett auktoriseringsproblem med avbildningsregistret.

Följ dessa steg om du vill [tillåta åtkomst via en Kubernetes-hemlighet](https://docs.microsoft.com/azure/container-registry/container-registry-auth-aks#access-with-kubernetes-secret).

## <a name="next-steps"></a>Nästa steg

I den här självstudien distribuerades programmet Azure Voting till ett Kubernetes-kluster i AKS. Det här är några av uppgifterna:

> [!div class="checklist"]
> * Ladda ned Kubernetes-manifestfiler
> * Köra programmet i Kubernetes
> * Testa programmet

Gå vidare till nästa självstudie om du vill lära dig om att skala ut både ett Kubernetes-program och den underliggande Kubernetes-infrastrukturen.

> [!div class="nextstepaction"]
> [Skala ut ett Kubernetes-program och tillhörande infrastruktur][aks-tutorial-scale]

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-documentation]: https://kubernetes.io/docs/home/
[kubernetes-service]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-scale]: ./tutorial-kubernetes-scale.md
[az-acr-list]: /cli/azure/acr#list
