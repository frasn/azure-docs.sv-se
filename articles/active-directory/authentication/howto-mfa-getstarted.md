---
title: Kom igång Azure MFA i molnet | Microsoft Docs
description: Det här är sidan om Microsoft Azure Multi-Factor Authentication som beskriver hur du kommer igång med Azure MFA i molnet.
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: get-started-article
ms.date: 06/24/2017
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: richagi
ms.openlocfilehash: 0a822d55e8d7bd0d503eb7d77f96dc9e60e1a4ba
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2018
---
# <a name="getting-started-with-azure-multi-factor-authentication-in-the-cloud"></a>Komma igång med Azure Multi-Factor Authentication i molnet
I den här artikeln beskrivs hur du kommer igång med Azure Multi-Factor Authentication i molnet.

> [!NOTE]
> Följande dokumentation innehåller information om hur du aktiverar användare med hjälp av **Azure-portalen**. Information om hur du konfigurerar Azure Multi-Factor Authentication för O365-användare finns i [Konfigurera Multi-Factor Authentication för Office 365.](https://support.office.com/article/Set-up-multi-factor-authentication-for-Office-365-users-8f0454b2-f51a-4d9c-bcde-2c48e41621c6?ui=en-US&rs=en-US&ad=US)

![MFA i molnet](./media/howto-mfa-getstarted/mfa_in_cloud.png)

## <a name="prerequisite"></a>Krav
[Registrera dig för en Azure-prenumeration](https://azure.microsoft.com/pricing/free-trial/) – Om du inte redan har en Azure-prenumeration måste du registrera dig för en. Om du precis börjat använda Azure MFA kan du använda en utvärderingsprenumeration.

## <a name="enable-azure-multi-factor-authentication"></a>Aktivera Azure Multi-Factor Authentication
Så länge användarna har licenser som inkluderar Azure Multi-Factor Authentication behöver du inte göra något för att aktivera Azure MFA. Du kan börja med att kräva tvåstegsverifiering för enskilda användare. Följande licenser aktiverar Azure MFA:
- Azure Multi-Factor Authentication
- Azure Active Directory Premium
- Enterprise Mobility + Security

Om du saknar någon av dessa tre licenser, eller om du inte har tillräckligt med licenser för alla användare, är det också ok. Du behöver bara utföra ett extra steg och [Skapa en Multi-Factor Authentication-provider](concept-mfa-authprovider.md) i din katalog.

## <a name="turn-on-two-step-verification-for-users"></a>Aktivera tvåstegsverifiering för användare

Använd något av tillvägagångssätten som beskrivs i avsnittet om [hur du kräver tvåstegsverifiering för en användare eller grupp](howto-mfa-userstates.md) för att börja använda Azure MFA. Du kan välja att tillämpa tvåstegsverifiering för alla inloggningar eller du kan skapa principer för villkorlig åtkomst för att kräva tvåstegsverifiering endast när det är viktigt för dig.

## <a name="next-steps"></a>Nästa steg
Nu när du har konfigurerat Multi-Factor Authentication i molnet kan du konfigurera din distribution. Se [Konfigurera Azure Multi-Factor Authentication](howto-mfa-mfasettings.md) för mer information.

