---
title: Runbook-körningen i Azure Automation
description: Innehåller information om hur en runbook i Azure Automation bearbetas.
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 05/08/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: ff58e22f8b9b837ec272cd2cd6193da80a7b718e
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/16/2018
---
# <a name="runbook-execution-in-azure-automation"></a>Runbook-körningen i Azure Automation

När du startar en runbook i Azure Automation skapas ett jobb. Ett jobb är en enskild körningsinstans av en runbook. En Azure Automation arbetare ska köras varje jobb. När anställda delas av flera Azure-konton, är jobb från olika Automation-konton isolerade från varandra. Du kan inte har styra vilken Worker services-begäran för jobbet. En enstaka runbook kan ha flera jobb körs samtidigt. Körningsmiljön för jobb från samma Automation-kontot kan återanvändas. När du visar listan över runbooks i Azure-portalen visar status för alla jobb som har startats för varje runbook. Du kan visa listan över jobb för varje runbook för att kunna spåra statusen för var och en. En beskrivning av olika jobbstatus [jobbstatus](#job-statuses).

[!INCLUDE [gdpr-dsr-and-stp-note.md](../../includes/gdpr-dsr-and-stp-note.md)]

Följande diagram visar livscykeln för ett runbook-jobb för [grafiska runbook-flöden](automation-runbook-types.md#graphical-runbooks) och [PowerShell-arbetsflöde runbooks](automation-runbook-types.md#powershell-workflow-runbooks).

![Jobbstatus - PowerShell-arbetsflöde](./media/automation-runbook-execution/job-statuses.png)

Följande diagram visar livscykeln för ett runbook-jobb för [PowerShell runbooks](automation-runbook-types.md#powershell-runbooks).

![Jobbstatus - PowerShell-skript](./media/automation-runbook-execution/job-statuses-script.png)

Jobb har åtkomst till resurserna i Azure genom att göra en anslutning till din Azure-prenumeration. De har endast åtkomst till resurser i ditt datacenter om resurserna är tillgängliga från det offentliga molnet.

## <a name="job-statuses"></a>Jobbstatus

I följande tabell beskrivs de olika statuslägen som är möjliga för ett jobb.

| Status | Beskrivning |
|:--- |:--- |
| Slutfört |Jobbet har slutförts. |
| Misslyckad |För [grafisk och PowerShell-arbetsflöde runbooks](automation-runbook-types.md), det gick inte att kompilera runbook. För [PowerShell-skript runbooks](automation-runbook-types.md), det gick inte att starta runbook eller jobbet påträffade ett undantag. |
| Misslyckades, väntar på resurser |Jobbet misslyckades eftersom den nått den [fördelning](#fair-share) begränsa tre gånger och startas från samma kontrollpunkten eller från början av runbooken varje gång. |
| I kö |Jobbet väntar på resurser på en Automation arbetare blir tillgängliga så att den kan startas. |
| Startar |Jobbet har tilldelats en arbetare och systemet håller starta den. |
| Fortsätter |Systemet håller på att återuppta jobbet efter det pausades. |
| Körs |Jobbet körs. |
| Kör, väntar på resurser |Jobbet har inaktiverats eftersom den nådde den [fördelning](#fair-share) gränsen. Återupptas från den senaste kontrollpunkten inom kort. |
| Stoppad |Jobbet stoppades av användaren innan den slutfördes. |
| Stoppas |Systemet håller på att stoppa jobbet. |
| Tillfälligt avbruten |Jobbet pausades av användaren, datorn eller ett kommando i runbook. Ett jobb som pausats kan startas igen och återuppta från den senaste kontrollpunkten eller från början av runbooken om det inte har kontrollpunkter. Runbook är endast pausas av systemet när ett undantag inträffar. ErrorActionPreference är som standard **Fortsätt**, vilket innebär att jobbet körs på ett fel. Om inställningsvariabeln är **stoppa**, och sedan jobbet pausar på ett fel. Gäller för [grafisk och PowerShell-arbetsflöde runbooks](automation-runbook-types.md) endast. |
| Pausar |Systemet försöker att pausa arbetet på begäran av användaren. Runbooken måste nå nästa kontrollpunkt innan den kan pausas. Om den redan passerat den sista kontrollpunkten sedan är klar innan den kan pausas. Gäller för [grafisk och PowerShell-arbetsflöde runbooks](automation-runbook-types.md) endast. |

## <a name="viewing-job-status-from-the-azure-portal"></a>Visa jobbstatus från Azure-portalen

Du kan visa en sammanfattande status för alla runbook-jobb eller visa detaljer om en viss runbook-jobb i Azure-portalen eller genom att konfigurera integration med logganalys-arbetsytan att vidarebefordra dataströmmar för jobbet status och jobb för runbook. Mer information om hur du integrerar med Log Analytics finns [vidarebefordra jobbstatus och jobbet strömmar från Automation till logganalys](automation-manage-send-joblogs-log-analytics.md).

### <a name="automation-runbook-jobs-summary"></a>Automation runbook-jobb sammanfattning

Till höger om det valda Automation-kontot, visas en sammanfattning av alla runbook-jobb för ett valt Automation-konto under **jobbet statistik** panelen.

![Jobbet statistik sida vid sida](./media/automation-runbook-execution/automation-account-job-status-summary.png).

Den här panelen visar antalet och grafisk representation av jobbstatus för alla jobb som körs.

Klickar du på ikonen visas den **jobb** som innehåller en sammanfattande lista över alla jobb som körs, med status, jobbkörningen och start och slutförande gånger.

![Jobb-bladet för Automation-konto](./media/automation-runbook-execution/automation-account-jobs-status-blade.png)

Du kan filtrera listan över jobb genom att välja **filtrera jobb** och filtrera på en viss runbook jobbstatus, eller från den nedrullningsbara listan datum/tid-intervall för att söka i.

![Filtrera jobbstatus](./media/automation-runbook-execution/automation-account-jobs-filter.png)

Alternativt, du kan visa jobb sammanfattningsinformation för en viss runbook genom att välja den runbook från den **Runbooks** bladet i Automation-konto och välj sedan den **jobb** panelen. Detta innebär det **jobb** bladet, och därifrån kan du klicka på Jobbposten att visa detaljer och effekt.

![Jobb-bladet för Automation-konto](./media/automation-runbook-execution/automation-runbook-job-summary-blade.png)

### <a name="job-summary"></a>Jobbsammanfattning

Du kan visa en lista över alla jobb som har skapats för en viss runbook och deras senaste status. Du kan filtrera listan efter jobbstatus och datumintervallet för senaste ändring i jobbet. Om du vill visa detaljerad information och utdata, klickar du på namnet på ett jobb. Detaljerad vy av jobbet innehåller värden för runbookparametrar som gavs av jobbet.

Du kan använda följande steg för att visa jobb för en runbook.

1. Välj i Azure-portalen **Automation** och välj sedan namnet på ett Automation-konto.
2. Hubben, Välj **Runbooks** och klicka sedan på den **Runbooks** bladet Välj en runbook i listan.
3. Klicka på bladet för den valda runbooken i **jobb** panelen.
4. Klicka på ett av jobben i listan och på informationsbladet för runbook-jobb kan du visa detaljer och effekt.

## <a name="retrieving-job-status-using-windows-powershell"></a>Hämta jobbstatus med Windows PowerShell

Du kan använda den [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx) att hämta jobb som skapats för en runbook och detaljerna för ett specifikt jobb. Om du startar en runbook med Windows PowerShell med [Start AzureRmAutomationRunbook](https://msdn.microsoft.com/library/mt603661.aspx), och sedan returnerar det resulterande jobbet. Använd [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx)utdata för att få ett jobbs effekt.

Följande exempelkommandon hämtar det senaste jobbet för en exempel-runbook och visa dess status, de angivna värdena för runbook-parametrar och resultatet av jobbet.

```azurepowershell-interactive
$job = (Get-AzureRmAutomationJob –AutomationAccountName "MyAutomationAccount" `
–RunbookName "Test-Runbook" -ResourceGroupName "ResourceGroup01" | sort LastModifiedDate –desc)[0]
$job.Status
$job.JobParameters
Get-AzureRmAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAcct" -Id $job.JobId –Stream Output
```

I följande exempel hämtar utdata för ett specifikt jobb och returnerar varje post. I fallet skrivs att det uppstod ett undantag för en av posterna, undantaget i stället för värdet. Detta är användbart när undantag kan ge ytterligare information som kan inte loggas normalt under utdata.

```azurepowershell-interactive
$output = Get-AzureRmAutomationJobOutput -AutomationAccountName <AutomationAccountName> -Id <jobID> -ResourceGroupName <ResourceGroupName> -Stream "Any"
foreach($item in $output)
{
    $fullRecord = Get-AzureRmAutomationJobOutputRecord -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -JobId <jobID> -Id $item.StreamRecordId
    if ($fullRecord.Type -eq "Error")
    {
        $fullRecord.Value.Exception
    }
    else
    {
    $fullRecord.Value
    }
}
```

## <a name="get-details-from-activity-log"></a>Hämta information från aktivitetsloggen

Annan information, till exempel den person eller det konto som startade runbook kan hämtas från aktivitetsloggen för automation-kontot. Följande PowerShell-exempel innehåller den sista användaren för att köra runbook i fråga:

```powershell-interactive
$SubID = "00000000-0000-0000-0000-000000000000"
$rg = "ResourceGroup01"
$AutomationAccount = "MyAutomationAccount"
$RunbookName = "Test-Runbook"
$JobResourceID = "/subscriptions/$subid/resourcegroups/$rg/providers/Microsoft.Automation/automationAccounts/$AutomationAccount/jobs"

Get-AzureRmLog -ResourceId $JobResourceID -MaxRecord 1 | Select Caller
```

## <a name="fair-share"></a>Fördelning

För att kunna dela resurser med alla runbooks i molnet, inaktiveras Azure Automation tillfälligt jobb när den har körts under tre timmar. Under denna tid jobb för [PowerShell-baserade runbooks](automation-runbook-types.md#powershell-runbooks) stoppas och inte startas. Status visas i jobbet **stoppad**. Den här typen av runbook är alltid om från början, eftersom de inte stöder kontrollpunkter.

[PowerShell-Arbetsflödesbaserade runbooks](automation-runbook-types.md#powershell-workflow-runbooks) återupptas från deras senaste [kontrollpunkt](https://docs.microsoft.com/system-center/sma/overview-powershell-workflows#bk_Checkpoints). När du har kört tre timmar, runbook-jobbet kommer att inaktiveras av tjänsten och dess status visas **körs, väntar på resurser**. När en sandbox blir tillgänglig, startas automatiskt runbook Automation-tjänsten och återupptar från den senaste kontrollpunkten. Detta är normalt PowerShell-arbetsflöde att pausa/restart. Om runbook igen överskrider tre timmars körning kan processen upprepas, upp till tre gånger. Efter den tredje omstarten om runbook fortfarande inte har slutförts i tre timmar sedan runbook-jobbet misslyckades och jobbstatus visar **misslyckades, väntar på resurser**. I det här fallet visas följande undantag med felet.

*Jobbet kan inte fortsätta eftersom det upprepade gånger har avlägsnats från samma kontrollpunkten. Kontrollera att din Runbook utför långvariga åtgärder utan att spara sitt tillstånd.*

Detta är att skydda tjänsten från runbooks som körs på obestämd tid utan att slutföra, eftersom de inte kan göra det i nästa kontrollpunkt utan tas bort från minnet igen.

Om runbooken har inga kontrollpunkter eller jobbet hade inte nå den första kontrollpunkten innan tas bort från minnet, startas sedan om från början.

När du skapar en runbook bör du kontrollera att tid att köra några aktiviteter mellan två kontrollpunkter inte överskrider tre timmar. Du kan behöva lägga till kontrollpunkter till din runbook så att den inte nå den här gränsen tre timmars eller dela upp långa kör åtgärder. Din runbook kan exempelvis utföra en omindexera på en stor SQL-databas. Om den här enda åtgärden inte slutförs inom den rättmätiga del minnet jobbet och startas om från början. I det här fallet bör du dela upp omindexera igen i flera steg, till exempel startas en tabell i taget, och infoga en kontrollpunkt efter varje åtgärd så att jobbet återupptas efter den senaste åtgärden har slutförts.

## <a name="next-steps"></a>Nästa steg

* Läs mer om olika metoder som kan användas för att starta en runbook i Azure Automation i [starta en runbook i Azure Automation](automation-starting-a-runbook.md)
