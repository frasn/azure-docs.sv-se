---
title: Azure IoT-hubb moln till enhet alternativ | Microsoft Docs
description: Utvecklare guide - hjälp när du ska använda direkt metoder, enheten dubbla önskade egenskaper eller moln till enhet meddelanden för moln till enhet kommunikation.
author: fsautomata
manager: ''
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: elioda
ms.openlocfilehash: ff81be4bbf6d297c623c5d98b5dc22a540112fcc
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34634445"
---
# <a name="cloud-to-device-communications-guidance"></a>Vägledning för kommunikation moln till enhet
IoT-hubb innehåller tre alternativ för enhetsappar exponera funktionen till en backend-app:

* [Dirigera metoder] [ lnk-methods] för kommunikation som kräver omedelbar bekräftelse av resultatet. Direkt-metoder används ofta för interaktiv kontroll av enheter, till exempel aktivera fläktar.
* [Dubbla har önskade egenskaper] [ lnk-twins] för tidskrävande kommandon som är avsedda att placera enheten i en viss önskad tillstånd. Till exempel telemetri skicka intervallet till 30 minuter.
* [Moln till enhet meddelanden] [ lnk-c2d] för envägs-meddelanden till appen med enheten.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Här följer en detaljerad jämförelse av de olika alternativen för kommunikation molnet till enheten.

|  | Direkta metoder | Dubblas egenskaper | Meddelanden moln till enhet |
| ---- | ------- | ---------- | ---- |
| Scenario | Kommandon som kräver omedelbar bekräftelse, till exempel aktivera fläktar. | Långvariga kommandon som är avsedda att placera enheten i ett visst tillstånd. Till exempel telemetri skicka intervallet till 30 minuter. | Enkelriktade meddelanden till appen med enheten. |
| Dataflöde | Dubbelriktad. Enhetsapp svara direkt till metoden. Lösningens serverdel får resultatet sammanhang att begäran. | Enkelriktad. Enheten appen tar emot ett meddelande med egenskapsändringen. | Enkelriktad. Enheten appen tar emot meddelandet
| Hållbarhet | Frånkopplade enheter kontaktas inte. Lösningens serverdel meddelas att enheten inte är ansluten. | Egenskapsvärden bevaras i dubbla för enheten. Enheten läses den vid nästa återanslutning. Egenskapsvärden är strängfält med den [IoT-hubb frågespråket][lnk-query]. | Meddelanden kan behållas av IoT-hubb för upp till 48 timmar. |
| Mål | Enhet med hjälp av **deviceId**, eller flera enheter med hjälp av [jobb][lnk-jobs]. | Enhet med hjälp av **deviceId**, eller flera enheter med hjälp av [jobb][lnk-jobs]. | Enskild enhet av **deviceId**. |
| Storlek | Maximal direkta metoden nyttolastens storlek är 128 KB. | Maximalt önskade egenskaper storleken är 8 KB. | Upp till 64 KB meddelanden. |
| Frekvens | Hög. Mer information finns i [IoT-hubb begränsar][lnk-quotas]. | Medel. Mer information finns i [IoT-hubb begränsar][lnk-quotas]. | Låg. Mer information finns i [IoT-hubb begränsar][lnk-quotas]. |
| Protokoll | Tillgängligt med MQTT eller AMQP. | Tillgängligt med MQTT eller AMQP. | Tillgängligt på alla protokoll. Enheten måste avsöka när du använder HTTPS. |

Lär dig hur du använder direkta metoder, egenskaper och moln till enhet meddelanden i följande kurser:

* [Använda direkt metoder][lnk-methods-tutorial], för direkta metoder.
* [Använd önskade egenskaper för att konfigurera enheter][lnk-twin-properties]för enheten dubbla har önskade egenskaper. 
* [Skicka meddelanden moln till enhet][lnk-c2d-tutorial], för moln till enhet meddelanden.

[lnk-twins]: iot-hub-devguide-device-twins.md
[lnk-quotas]: iot-hub-devguide-quotas-throttling.md
[lnk-query]: iot-hub-devguide-query-language.md
[lnk-jobs]: iot-hub-devguide-jobs.md
[lnk-c2d]: iot-hub-devguide-messages-c2d.md
[lnk-methods]: iot-hub-devguide-direct-methods.md
[lnk-methods-tutorial]: iot-hub-node-node-direct-methods.md
[lnk-twin-properties]: iot-hub-node-node-twin-how-to-configure.md
[lnk-c2d-tutorial]: iot-hub-node-node-c2d.md
