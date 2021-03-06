---
title: Konfigurera ett Azure cloud service-projekt i Visual Studio | Microsoft Docs
description: Lär dig hur du konfigurerar en Azure cloud service-projekt i Visual Studio, beroende på dina krav för projektet.
services: visual-studio-online
author: ghogen
manager: douge
assetId: 609d6965-05cc-47b1-82dc-c76a92d4f295
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/06/2017
ms.author: ghogen
ms.openlocfilehash: 0b2b39897407875d2722c7fbaebebc1698f931a9
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/23/2018
---
# <a name="configure-an-azure-cloud-service-project-with-visual-studio"></a>Konfigurera ett Azure cloud service-projekt i Visual Studio
Du kan konfigurera ett Azure cloud service-projekt, beroende på dina krav för projektet. Du kan ange egenskaper för projektet i följande kategorier:

- **Publicera en tjänst i molnet till Azure** -du kan ange en egenskap för att se till att en befintlig molntjänst som distribueras till Azure inte tas bort av misstag.
- **Köra eller felsöka en molnbaserad tjänst på den lokala datorn** -du kan välja en tjänstkonfiguration och ange om du vill starta Azure storage-emulatorn.
- **Validera en cloud service-paket när den skapas** -du kan välja att behandla alla varningar som fel så att du kan se till att cloud service-paketet distribueras utan problem. 

## <a name="steps-to-configure-an-azure-cloud-service-project"></a>Steg för att konfigurera en Azure cloud service-projekt
1. Öppna eller skapa ett molntjänstprojekt i Visual Studio

1. I **Solution Explorer**, högerklicka på projektet och, från snabbmenyn, Välj **egenskaper**.
   
1. Välj i projektets egenskapssida i **Development** fliken.

    ![Projekt egenskaper-menyn](./media/vs-azure-tools-configuring-an-azure-project/solution-explorer-project-properties-menu.png)

1. Ange **fråga innan du tar bort en befintlig distribution** till **SANT**. Den här inställningen som hjälper dig för att kontrollera att du inte av misstag tar bort en befintlig distribution i Azure

1. Välj den önskade **tjänstkonfiguration** som anger vilken tjänstkonfiguration som du vill använda när du kör eller felsöka din molntjänst lokalt. Mer information om hur du ändrar en tjänstkonfiguration för en roll finns [hur du konfigurerar roller för en Azure-molntjänst med Visual Studio](./vs-azure-tools-configure-roles-for-cloud-service.md).

1. Ange **starta Azure storage-emulatorn** till **SANT** att starta Azure storage-emulatorn när du kör eller felsöka din molntjänst lokalt.

1. Ange **behandla varningar som fel** till **SANT** att se till att du inte kan publicera om det finns valideringsfel i paketet.

1. Ange **web project portar** till **SANT** se till att din webbroll använder samma port varje gång det startar lokalt i IIS Express.

1. I verktygsfältet i Visual Studio väljer **spara**.

## <a name="next-steps"></a>Nästa steg
- [Konfigurera en Azure-projekt med flera tjänstkonfiguration](vs-azure-tools-multiple-services-project-configurations.md)

