---
title: Med hjälp av API-version profiler med Python i Azure-stacken | Microsoft Docs
description: Lär dig mer om hur du använder profiler för API-version med Python i Azure-stacken.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/30/2018
ms.author: mabrigg
ms.reviewer: sijuman
<!-- dev: viananth -->
ms.openlocfilehash: a4fe62ba8c0732745326831b977e8975e1210436
ms.sourcegitcommit: 6e43006c88d5e1b9461e65a73b8888340077e8a2
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/01/2018
---
# <a name="use-api-version-profiles-with-python-in-azure-stack"></a>Använd profiler för API-version med Python i Azure-stacken

*Gäller för: Azure Stack integrerat system och Azure-stacken Development Kit*

## <a name="python-samples-for-azure-stack"></a>Python-exempel för Azure-Stack 

Du kan använda följande kodexempel för att utföra vanliga hanteringsuppgifter för virtuella datorer i Azure-stacken.

Kodexempel visar du:

- Skapa virtuella datorer:
    - Skapa en virtuell Linux-dator
    - Skapa en virtuell Windows-dator
- Uppdatera en virtuell dator:
    - Expandera en enhet
    - Tagga en virtuell dator
    - Bifoga datadiskar
    - Koppla från datadiskar
- Använda en virtuell dator:
    - Starta en virtuell dator
    - Stoppa en virtuell dator
    - Starta om en virtuell dator
- Lista över virtuella datorer
- Ta bort en virtuell dator

Om du vill granska koden för att utföra dessa åtgärder, ta en titt på **run_example()** funktion i Python-skriptet **Hybrid/unmanaged-disks/example.py** i GitHub-Repo- [ virtuella datorer – python-hantera](https://github.com/viananth/virtual-machines-python-manage/tree/8643ed4bec62aae6fdb81518f68d835452872f88).

Varje åtgärd är tydligt märkt med en kommentar och en funktionen print.
Exemplen är inte nödvändigtvis i den ordning som visas i listan ovan.


## <a name="run-the-python-sample"></a>Kör Python-exempel

1.  Om du inte redan har det, [installera Python](https://www.python.org/downloads/).

    Det här exemplet (och SDK) är kompatibel med Python 2.7, 3.4, 3.5 och 3,6.

2.  Allmänna rekommendationer för utveckling av Python är att använda en virtuell miljö. 
    Mer information finns i https://docs.python.org/3/tutorial/venv.html
    
    Installera och starta den virtuella miljön med modulen ”venv” på Python 3 (du måste installera [virtuell miljö](https://pypi.python.org/pypi/virtualenv) för Python 2.7):

    ````bash
    python -m venv mytestenv # Might be "python3" or "py -3.6" depending on your Python installation
    cd mytestenv
    source bin/activate      # Linux shell (Bash, ZSH, etc.) only
    ./scripts/activate       # PowerShell only
    ./scripts/activate.bat   # Windows CMD only
    ````

3.  Klona lagringsplatsen.

    ````bash
    git clone https://github.com/Azure-Samples/virtual-machines-python-manage.git
    ````

4.  Installera beroenden med pip.

    ````bash
    cd virtual-machines-python-manage\Hybrid
    pip install -r requirements.txt
    ````

5.  Skapa en [tjänstens huvudnamn](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals) att arbeta med Azure-stacken. Kontrollera att tjänstens huvudnamn har [deltagare/ägarrollen](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals#assign-role-to-service-principal) på din prenumeration.

6.  Ange följande variabler och exportera dessa miljövariabler i din aktuella shell. 

`Note: provide an explanation of where these variables come from?`

    ````bash
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    export ARM_ENDPOINT={your AzureStack Resource Manager Endpoint}
    ```

7.  Observera att för att kunna köra det här exemplet, Ubuntu 16.04-LTS och Windows Server 2012 R2 Datacenter bilder måste finnas i Azure-stacken marknadsplatsen. Dessa kan vara antingen [ned från Azure](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-download-azure-marketplace-item) eller [till plattformen Avbildningslagringsplatsen](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-add-vm-image).


8. Kör exemplet.

    ```
    python unmanaged-disks\example.py
    ```

## <a name="notes"></a>Anteckningar

Du kanske tro att försöka hämta OS-disk för en virtuell dator med hjälp av `virtual_machine.storage_profile.os_disk`.
I vissa fall kan detta göra vad du vill att, men tänk på att den ger dig en `OSDisk` objekt.
För att uppdatera OS-diskens storlek som `example.py` , du behöver inte en `OSDisk` objektet men `Disk` objekt.
`example.py` hämtar den `Disk` objekt med följande:

```python
os_disk_name = virtual_machine.storage_profile.os_disk.name
os_disk = compute_client.disks.get(GROUP_NAME, os_disk_name)
```

## <a name="next-steps"></a>Nästa steg

- [Azure Python Development Center](https://azure.microsoft.com/develop/python/)
- [Dokumentation för Azure virtuella datorer](https://azure.microsoft.com/services/virtual-machines/)
- [Utbildningsväg för virtuella datorer](https://azure.microsoft.com/documentation/learning-paths/virtual-machines/)
- Om du inte har en Microsoft Azure-prenumeration kan du få ett kostnadsfritt utvärderingskonto [här](http://go.microsoft.com/fwlink/?LinkId=330212).
