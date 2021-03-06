---
title: Rotera hemligheter i Azure-stacken | Microsoft Docs
description: Lär dig mer om att rotera din hemliga i Azure-stacken.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2018
ms.author: mabrigg
ms.reviewer: ppacent
ms.openlocfilehash: a3dfce6ce1b136e39047cfd47b336b2fb2a35af9
ms.sourcegitcommit: 96089449d17548263691d40e4f1e8f9557561197
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/17/2018
---
# <a name="rotate-secrets-in-azure-stack"></a>Rotera hemligheter i Azure-stacken

*Dessa anvisningar gäller bara för Azure-stacken integrerad System Version 1803 och senare. Försök inte hemliga rotation i pre-1802 Azure Stack versioner*

Azure-stacken använder olika hemligheter för att underhålla säker kommunikation mellan Azure Stack infrastrukturresurser och tjänster.

- **Internt hemligheter**  
Alla de certifikat, lösenord, säker strängar och nycklar som används av infrastrukturen i Azure-stacken utan åtgärd för Stack-operatorn Azure. 

- **Externa hemligheter**  
Infrastruktur tjänstcertifikat för externa tjänster som tillhandahålls av Azure Stack-operatorn. Detta inkluderar certifikat för följande tjänster: 
    - Administratörsportalen 
    - Offentlig Portal 
    - Administratören Azure Resource Manager 
    - Globala Azure Resource Manager 
    - Administratören Keyvault 
    - KeyVault 
    - ACS (inklusive blob-, tabell- och queue storage) 
    - ADFS<sup>*</sup>
    - Diagrammet<sup>*</sup>

    > <sup>*</sup> Gäller endast om den miljön identitetsleverantören är Active Directory Federation Services (AD FS).

> [!NOTE]
> Alla andra säkra nycklar och strängar, inklusive BMC och växla lösenord, användar- och kontolösenord uppdateras manuellt av administratören. 

Operatorer måste kunna regelbundet rotera sin infrastruktur hemligheter i frekvenser som är konsekventa med organisationens säkerhetskrav för att upprätthålla integriteten hos Azure Stack-infrastruktur.

### <a name="rotating-secrets-with-external-certificates-from-a-new-certificate-authority"></a>Rotera hemligheter med externa certifikat från en ny certifikatutfärdare

Azure-stacken stöder hemliga rotation med externa certifikat från en ny certifikat (CA) i följande kontexter:

|Installerade certifikat Certifikatutfärdare|Certifikatutfärdaren för att rotera till|Stöds|Azure Stack-versioner som stöds|
|-----|-----|-----|-----|-----|
|Från självsignerade|För företag|Stöds inte||
|Från självsignerade|Att självsignerat|Stöds inte||
|Från självsignerade|Allmänheten<sup>*</sup>|Stöds|1803 & senare|
|Från Enterprise|För företag|Så länge som kunder använder samma företagets Certifikatutfärdare som används vid distribution som stöds|1803 & senare|
|Från Enterprise|Att självsignerat|Stöds inte||
|Från Enterprise|Allmänheten<sup>*</sup>|Stöds|1803 & senare|
|Från allmänheten<sup>*</sup>|För företag|Stöds inte|1803 & senare|
|Från allmänheten<sup>*</sup>|Att självsignerat|Stöds inte||
|Från allmänheten<sup>*</sup>|Allmänheten<sup>*</sup>|Stöds|1803 & senare|

<sup>*</sup> Här är offentlig certifikatutfärdare som är en del av Windows Trusted Root Program. Du hittar en fullständig lista [Microsoft Trusted Root Certificate Program: deltagare (från och med 27 juni 2017)](https://gallery.technet.microsoft.com/Trusted-Root-Certificate-123665ca).

## <a name="alert-remediation"></a>Aviseringen reparation

När hemligheter är inom 30 dagar upphör att gälla, skapas följande aviseringar i Administratörsportalen: 

- Lösenordets giltighetstid för väntande tjänster konto 
- Väntande interna certifikatet upphör att gälla 
- Väntande externa certifikatet upphör att gälla 

Kör hemliga rotation med hjälp av anvisningarna nedan kommer att åtgärda dessa aviseringar.

## <a name="pre-steps-for-secret-rotation"></a>Före steg för hemliga rotation

1.  Meddela användarna av eventuella underhållsåtgärder. Schemalägga normal underhållsfönster, så mycket som möjligt, under tider utanför kontorstid. Underhåll kan påverka både användaren arbetsbelastningar och portalen åtgärder.

    > [!note]  
    > Nästa steg gäller endast när rotera Azure Stack externa hemligheter.

2.  Förbered en ny uppsättning utbyte av externa certifikat. Den nya uppsättningen matchar de specifikationer för certifikatet som anges i den [Azure Stack PKI-certifikatkrav](https://docs.microsoft.com/azure/azure-stack/azure-stack-pki-certs).
3.  Lagra en säkerhetskopiering till de certifikat som används för rotation i en säker plats för säkerhetskopiering. Om din rotation körs och misslyckas, ersätter du certifikaten i filresursen med säkerhetskopiorna innan du kör rotationen. Observera, säkerhetskopior i en säker plats för säkerhetskopiering.
3.  Skapa en filresurs som du kan komma åt från de virtuella datorerna ERCS. Filresursen måste vara läsbar och skrivbar för den **CloudAdmin** identitet.
4.  Öppna ett PowerShell ISE-konsolen från en dator där du har åtkomst till filresursen. Gå till din filresurs. 
5.  Kör **[CertDirectoryMaker.ps1](http://www.aka.ms/azssecretrotationhelper)** att skapa katalogerna som krävs för din externa certifikat.

## <a name="rotating-external-and-internal-secrets"></a>Rotera externa och interna hemligheter

Rotera både externa ett internt hemlighet:

1. I den nyligen skapade **certifikat** katalog som skapades i steg inför placera den nya uppsättningen ersättning externa certifikat i katalogstrukturen enligt det format som beskrivs i avsnittet obligatoriska certifikat i den [Azure Stack PKI-certifikatkrav](https://docs.microsoft.com/azure/azure-stack/azure-stack-pki-certs#mandatory-certificates).
2. Skapa en PowerShell-Session med den [Privilegierade Endpoint](https://docs.microsoft.com/azure/azure-stack/azure-stack-privileged-endpoint) med hjälp av den **CloudAdmin** konto och lagra sessionerna som en variabel. Du använder den här variabeln som parameter i nästa steg.

    > [!IMPORTANT]  
    > Ange sessionen inte, lagra sessionen som en variabel.
    
3. Kör  **[anropskommandot](https://docs.microsoft.com/powershell/module/microsoft.powershell.core/invoke-command?view=powershell-5.1)**. Skicka Privilegierade Endpoint PowerShell-sessionsvariabeln som den **Session** parameter. 
4. Kör **Start SecretRotation** med följande parametrar:
    - **PfxFilesPath**  
    Ange sökvägen till katalogen certifikat som skapades tidigare.  
    - **PathAccessCredential**  
    Ett PSCredential-objekt för autentiseringsuppgifter till resursen. 
    - **CertificatePassword**  
    En säker sträng för det lösenord som används för alla pfx-certifikat-filer skapas.
4. Vänta medan din hemliga rotera.  
När hemliga rotation har slutförts, visar konsolen **övergripande Åtgärdsstatus: lyckade**. 
5. Efter slutförande av hemliga rotation, ta bort certifikat från resursnamn som du skapade i steget innan och lagrar dem i sina säker plats för säkerhetskopiering. 

## <a name="walkthrough-of-secret-rotation"></a>Genomgång av hemliga rotation

PowerShell-exemplet nedan visar cmdlets och parametrar för att rotera din hemliga.

```powershell
#Create a PEP Session
winrm s winrm/config/client '@{TrustedHosts= "<IPofERCSMachine>"}'
$PEPCreds = Get-Credential 
$PEPsession = New-PSSession -computername <IPofERCSMachine> -Credential $PEPCreds -ConfigurationName PrivilegedEndpoint 

#Run Secret Rotation
$CertPassword = ConvertTo-SecureString "Certpasswordhere" -AsPlainText -Force
$CertShareCred = Get-Credential 
$CertSharePath = "<NetworkPathofCertShare>"
Invoke-Command -session $PEPsession -ScriptBlock { 
Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCred -CertificatePassword $using:CertPassword }
Remove-PSSession -Session $PEPSession
```
## <a name="rotating-only-internal-secrets"></a>Rotera endast internt hemligheter

Rotera endast Azure-stacken interna hemligheter:

1. Skapa en PowerShell-session med den [Privilegierade Endpoint](https://docs.microsoft.com/azure/azure-stack/azure-stack-privileged-endpoint).
2. I den privilegierade Endpoint-sessionen köras **Start SecretRotation** utan argument.

## <a name="start-secretrotation-reference"></a>Start-SecretRotation referens

Roterar hemligheter för ett Azure Stack-System. Körs bara mot Azure Stack Privilegierade slutpunkten.

### <a name="syntax"></a>Syntax

Sökvägen (standard)

```powershell
Start-SecretRotation [-PfxFilesPath <string>] [-PathAccessCredential] <PSCredential> [-CertificatePassword <SecureString>]  
```

### <a name="description"></a>Beskrivning

Cmdleten Start-SecretRotation roterar infrastruktur hemligheter för ett Azure Stack-system. Som standard den roteras alla hemligheter som är exponerad för intern infrastruktur för nätverk med användarindata roterar den också certifikat för alla externa nätverk infrastruktur slutpunkter. När du roterar externt nätverk infrastruktur slutpunkter, ska Start-SecretRotation köras via ett Invoke-Command-skriptblock med Azure Stack-miljön Privilegierade endpoint session skickades som parametern session.
 
### <a name="parameters"></a>Parametrar

| Parameter | Typ | Krävs | Position | Standard | Beskrivning |
| -- | -- | -- | -- | -- | -- |
| PfxFilesPath | Sträng  | False  | Med namnet  | Ingen  | Sökvägen till filresursen den **\Certificates** katalog som innehåller alla externa nätverk endpoint certifikat. Krävs endast när rotera interna och externa hemligheter. End katalogen måste vara **\Certificates**. |
| CertificatePassword | SecureString | False  | Med namnet  | Ingen  | Lösenordet för alla certifikat som anges i - PfXFilesPath. Obligatoriskt värde om PfxFilesPath tillhandahålls när både interna och externa hemligheter roteras. |
|

### <a name="examples"></a>Exempel
 
**Rotera endast intern infrastruktur hemligheter**

```powershell  
PS C:\> Start-SecretRotation  
```

Det här kommandot roterar alla infrastruktur-hemligheter som exponeras för Azure-stacken interna nätverket. Start-SecretRotation roterar alla stack-genererade hemligheter, men eftersom det inte finns några angivna certifikat, roteras inte externa slutpunkten certifikat.  

**Rotera interna och externa infrastruktur hemligheter**
  
```powershell
PS C:\> Invoke-Command -session $PEPSession -ScriptBlock { 
Start-SecretRotation -PfxFilesPath $using:CertSharePath -PathAccessCredential $using:CertShareCred -CertificatePassword $using:CertPassword } 
Remove-PSSession -Session $PEPSession
```

Det här kommandot roterar alla infrastruktur-hemligheter som exponeras för Azure-stacken interna nätverk samt TLS-certifikat som används för Azure-stacken externt nätverk infrastruktur slutpunkter. Start-SecretRotation roterar alla stack-genererade hemligheter och eftersom det finns certifikat, roteras även externa slutpunkten certifikat.  


## <a name="update-the-baseboard-management-controller-bmc-password"></a>Uppdatera lösenordet baseboard management controller (BMC)

Den hanteringsstyrenhet för baskort (BMC) övervakar tillståndet fysiska servrar. Specifikationer och instruktioner om hur du uppdaterar lösenordet för BMC variera beroende på maskinvaruleverantören OEM-tillverkaren (OEM). Du bör uppdatera ditt lösenord för Azure Stack-komponenter på en vanlig takt.

1. Uppdatera bmc-Styrenheten på fysiska servrar för Azure-stacken genom att följa OEM-instruktioner. Lösenord för varje BMC i din miljö måste vara samma.
2. Öppna en privilegierad slutpunkt i Azure Stack-sessioner. Anvisningar finns i avsnittet [med Privilegierade slutpunkten i Azure-stacken](azure-stack-privileged-endpoint.md).
3. När din PowerShell prompten har ändrats till **[IP-adress eller ERCS VM name]: PS >** eller **[azs ercs01]: PS >**, beroende på miljön, kör `Set-BmcPassword` genom att köra `invoke-command`. Skicka dina Privilegierade endpoint sessionsvariabeln som en parameter. Exempel:

    ```powershell
    # Interactive Version
    $PEip = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PECred = Get-Credential "<Domain>\CloudAdmin" -Message "PE Credentials" 
    $NewBMCpwd = Read-Host -Prompt "Enter New BMC password" -AsSecureString 

    $PEPSession = New-PSSession -ComputerName $PEip -Credential $PECred -ConfigurationName "PrivilegedEndpoint" 

    Invoke-Command -Session $PEPSession -ScriptBlock {
        Set-Bmcpassword -bmcpassword $using:NewBMCpwd
    }
    Remove-PSSession -Session $PEPSession
    ```
    
    Du kan också använda statiska PowerShell-version med lösenord som kodraderna:
    
    ```powershell
    # Static Version
    $PEip = "<Privileged Endpoint IP or Name>" # You can also use the machine name instead of IP here.
    $PEUser = "<Privileged Endpoint user for exmaple Domain\CloudAdmin>"
    $PEpwd = ConvertTo-SecureString "<Privileged Endpoint Password>" -AsPlainText -Force
    $PECred = New-Object System.Management.Automation.PSCredential ($PEUser, $PEpwd) 
    $NewBMCpwd = ConvertTo-SecureString "<New BMC Password>" -AsPlainText -Force 

    $PEPSession = New-PSSession -ComputerName $PEip -Credential $PECred -ConfigurationName "PrivilegedEndpoint" 

    Invoke-Command -Session $PEPSession -ScriptBlock {
        Set-Bmcpassword -bmcpassword $using:NewBMCpwd
    }
    Remove-PSSession -Session $PEPSession
    ```

## <a name="next-steps"></a>Nästa steg

[Lär dig mer om Azure Stack-säkerhet](azure-stack-security-foundations.md)
