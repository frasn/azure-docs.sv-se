---
title: Tvåstegsverifiering och AD FS – Azure MFA | Microsoft Docs
description: Det här är sidan om Azure Multi-Factor Authentication som beskriver hur du kommer igång med Azure MFA och AD FS.
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: get-started-article
ms.date: 08/25/2017
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: richagi
ms.openlocfilehash: 5ee4f3eefb23b16bb135cb7b29511c1c0324ec2c
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2018
---
# <a name="getting-started-with-azure-multi-factor-authentication-and-active-directory-federation-services"></a>Komma igång med Azure Multi-Factor Authentication och Active Directory Federation Services

<center>![Moln](./media/multi-factor-authentication-get-started-adfs/adfs.png)</center>

Om din organisation har federerat det lokala Active Directory med Azure Active Directory med hjälp av AD FS finns det två Azure Multi-Factor Authentication-alternativ tillgängliga.

* Skydda molnresurser med Azure Multi-Factor Authentication eller Active Directory Federation Services
* Skydda molnet och lokala resurser med Azure Multi-Factor Authentication Server

Följande tabell sammanfattar verifieringsupplevelsen när resurser skyddas med Azure Multi-Factor Authentication och AD FS

| Verifieringsupplevelse – webbläsarbaserade appar | Verifieringsupplevelse – appar som inte är webbläsarbaserade |
|:--- |:--- |:--- |
| Skydda Azure AD-resurser med hjälp av Azure Multi-Factor Authentication |<li>Det första verifieringssteget utförs lokalt med hjälp av AD FS.</li> <li>Det andra steget är en telefonbaserad metod som utförs med hjälp av molnautentisering.</li> |
| Skydda Azure AD-resurser med hjälp av Active Directory Federation Services |<li>Det första verifieringssteget utförs lokalt med hjälp av AD FS.</li><li>Det andra steget utförs lokalt genom att anspråket tillämpas.</li> |

Varningar med applösenord för federerade användare:

* Applösenord verifieras med molnautentisering och kringgår därför federation. Federation används endast aktivt när applösenorden konfigureras.
* Inställningar för lokal klientåtkomstkontroll respekteras inte av applösenord.
* Du kan inte använda lokal autentiseringsloggning med applösenord.
* Inaktiveringen eller borttagningen av konton kan ta upp till tre timmar för katalogsynkronisering, vilket försenar inaktiveringen eller borttagningen av applösenord i molnidentiteten.

Information om hur du konfigurerar Azure Multi-Factor Authentication eller Azure Multi-Factor Authentication Server med AD FS finns i följande artiklar:

* [Skydda molnresurser med hjälp av Azure Multi-Factor Authentication och AD FS](howto-mfa-adfs.md)
* [Skydda molnresurser och lokala resurser med Azure Multi-Factor Authentication Server med Windows Server 2012 R2 AD FS](howto-mfaserver-adfs-2012.md)
* [Skydda molnet och lokala resurser med Azure Multi-Factor Authentication Server med AD FS 2.0](howto-mfaserver-adfs-2.md)
