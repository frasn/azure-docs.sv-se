---
title: Azure N-serien drivrutinsinstallation för Linux | Microsoft Docs
description: Hur du ställer in NVIDIA GPU drivrutiner för N-serien virtuella datorer som kör Linux i Azure
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: d91695d0-64b9-4e6b-84bd-18401eaecdde
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/29/2018
ms.author: danlep
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 807af10c0655d9d1728a80a47d1f8f9c2a16fb84
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 06/01/2018
ms.locfileid: "34654291"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-linux"></a>Installera drivrutiner för NVIDIA GPU på N-serien virtuella datorer som kör Linux

Om du vill dra nytta av funktionerna i Azure N-serien virtuella datorer som kör Linux GPU installeras grafik drivrutinerna. Den här artikeln innehåller drivrutinen konfigurationsstegen när du distribuerar en virtuell dator i N-serien. Inställningsinformation för drivrutinen är också tillgängligt för [virtuella Windows-datorer](../windows/n-series-driver-setup.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

N-serien VM specifikationerna, lagringskapacitet, och diskinformation finns [GPU Linux VM-storlekar](sizes-gpu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

[!INCLUDE [virtual-machines-n-series-linux-support](../../../includes/virtual-machines-n-series-linux-support.md)]

## <a name="install-cuda-drivers-on-n-series-vms"></a>Installera CUDA drivrutiner på N-serien virtuella datorer

Här följer stegen för att installera drivrutiner för CUDA från NVIDIA CUDA Toolkit på N-serien virtuella datorer. 


C och C++ utvecklare kan välja att installera fullständig Toolkit för att skapa GPU-snabbare program. Mer information finns i [CUDA installationsguiden](http://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html).

Om du vill installera CUDA drivrutiner skapar du en SSH-anslutning på varje virtuell dator. Kontrollera att systemet har en CUDA-kompatibla GPU genom att köra följande kommando:

```bash
lspci | grep -i NVIDIA
```
Du kommer se utdata som liknar följande exempel (visar ett NVIDIA Tesla K80-kort):

![lspci kommandoutdata](./media/n-series-driver-setup/lspci.png)

Sedan kör installationskommandon som är specifika för din distribution.

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

1. Hämta och installera drivrutiner för CUDA.
  ```bash
  CUDA_REPO_PKG=cuda-repo-ubuntu1604_9.1.85-1_amd64.deb

  wget -O /tmp/${CUDA_REPO_PKG} http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG} 

  sudo dpkg -i /tmp/${CUDA_REPO_PKG}

  sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub 

  rm -f /tmp/${CUDA_REPO_PKG}

  sudo apt-get update

  sudo apt-get install cuda-drivers

  ```

  Installationen kan ta flera minuter.

2. Vill du installera komplett CUDA verktyg skriver du:

  ```bash
  sudo apt-get install cuda
  ```

3. Starta om den virtuella datorn och fortsätta med att verifiera installationen.

#### <a name="cuda-driver-updates"></a>CUDA drivrutinsuppdateringar

Vi rekommenderar att du regelbundet uppdatera CUDA drivrutiner efter distributionen.

```bash
sudo apt-get update

sudo apt-get upgrade -y

sudo apt-get dist-upgrade -y

sudo apt-get install cuda-drivers

sudo reboot
```

### <a name="centos-or-red-hat-enterprise-linux-73-or-74"></a>CentOS eller Red Hat Enterprise Linux 7.3 eller 7.4

1. Uppdatera kernel.

  ```
  sudo yum install kernel kernel-tools kernel-headers kernel-devel
  
  sudo reboot

2. Install the latest [Linux Integration Services for Hyper-V and Azure](https://www.microsoft.com/download/details.aspx?id=55106).

  ```bash
  wget https://aka.ms/lis
 
  tar xvzf lis
 
  cd LISISO
 
  sudo ./install.sh
 
  sudo reboot
  ```
 
3. Anslut till den virtuella datorn och fortsätta installationen med följande kommandon:

  ```bash
  sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

  sudo yum install dkms

  CUDA_REPO_PKG=cuda-repo-rhel7-9.1.85-1.x86_64.rpm

  wget http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/${CUDA_REPO_PKG} -O /tmp/${CUDA_REPO_PKG}

  sudo rpm -ivh /tmp/${CUDA_REPO_PKG}

  rm -f /tmp/${CUDA_REPO_PKG}

  sudo yum install cuda-drivers
  ```

  Installationen kan ta flera minuter. 

4. Vill du installera komplett CUDA verktyg skriver du:

  ```bash
  sudo yum install cuda
  ```

5. Starta om den virtuella datorn och fortsätta med att verifiera installationen.

### <a name="verify-driver-installation"></a>Verifiera installation av drivrutiner

Fråga Enhetsstatus GPU SSH till den virtuella datorn och kör den [nvidia smi](https://developer.nvidia.com/nvidia-system-management-interface) kommandoradsverktyg som installeras med drivrutinen. 

Om drivrutinen är installerad visas utdata som liknar följande. Observera att **GPU-Util** visar 0% om du använder en GPU arbetsbelastning på den virtuella datorn. Din version av drivrutinen och information GPU kan skilja sig från de som visas.

![NVIDIA Enhetsstatus](./media/n-series-driver-setup/smi.png)

## <a name="rdma-network-connectivity"></a>RDMA-nätverksanslutning

RDMA-nätverksanslutning kan aktiveras på RDMA-kompatibla N-serien virtuella datorer som NC24r distribueras i samma tillgänglighetsuppsättning eller i en enda placering grupp i en skaluppsättning för virtuell dator. RDMA-nätverket har stöd för Message Passing Interface (MPI) trafik för program som körs med Intel MPI 5.x eller en senare version. Ytterligare krav som följer:

### <a name="distributions"></a>Distributioner

Distribuera RDMA-kompatibla N-serien virtuella datorer från en av avbildningarna i Azure Marketplace som stöder RDMA-anslutningar på N-serien virtuella datorer:
  
* **Ubuntu 16.04 LTS** – konfigurera RDMA drivrutiner på den virtuella datorn och registrera med Intel hämta Intel MPI:

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **CentOS-baserade 7.4 HPC** -RDMA drivrutiner och Intel MPI 5.1 är installerade på den virtuella datorn.

## <a name="install-grid-drivers-on-nv-series-vms"></a>Installera drivrutiner för RUTNÄTET på NV-serien virtuella datorer

Göra en SSH-anslutning för varje virtuell dator för att installera drivrutiner för NVIDIA RUTNÄTET på NV-serien virtuella datorer, och följ anvisningarna för Linux-distribution. 

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

1. Kör den `lspci` kommando. Kontrollera att NVIDIA M60 kort eller kort visas som PCI-enheter.

2. Installera uppdateringar.

  ```bash
  sudo apt-get update

  sudo apt-get upgrade -y

  sudo apt-get dist-upgrade -y

  sudo apt-get install build-essential ubuntu-desktop -y
  ```
3. Inaktivera kernel-drivrutinen Nouveau som inte är kompatibel med NVIDIA-drivrutinen. (Endast använda NVIDIA drivrutinen på NV virtuella datorer.) Det gör du genom att skapa en fil i `/etc/modprobe.d `med namnet `nouveau.conf` med följande innehåll:

  ```
  blacklist nouveau

  blacklist lbm-nouveau
  ```


4. Starta om den virtuella datorn och återansluta. Avsluta X-server:

  ```bash
  sudo systemctl stop lightdm.service
  ```

5. Hämta och installera drivrutinen RUTNÄTET:

  ```bash
  wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=849941  

  chmod +x NVIDIA-Linux-x86_64-grid.run

  sudo ./NVIDIA-Linux-x86_64-grid.run
  ``` 

6. När du tillfrågas om du vill köra verktyget nvidia xconfig Välj om du vill uppdatera konfigurationsfilen X **Ja**.

7. När installationen är klar, kopiera /etc/nvidia/gridd.conf.template till en ny fil gridd.conf på plats/etc/nvidia /

  ```bash
  sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
  ```

8. Lägg till följande `/etc/nvidia/gridd.conf`:
 
  ```
  IgnoreSP=TRUE
  ```
9. Starta om den virtuella datorn och fortsätta med att verifiera installationen.


### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS eller Red Hat Enterprise Linux 

1. Uppdatera kernel och DKMS.
 
  ```bash  
  sudo yum update
 
  sudo yum install kernel-devel
 
  sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
 
  sudo yum install dkms
  ```

2. Inaktivera kernel-drivrutinen Nouveau som inte är kompatibel med NVIDIA-drivrutinen. (Endast använda NVIDIA drivrutinen på NV virtuella datorer.) Det gör du genom att skapa en fil i `/etc/modprobe.d `med namnet `nouveau.conf` med följande innehåll:

  ```
  blacklist nouveau

  blacklist lbm-nouveau
  ```
 
3. Starta om den virtuella datorn, återansluta och installera senaste [Linux integreringstjänsterna för Hyper-V och Azure](https://www.microsoft.com/download/details.aspx?id=55106).
 
  ```bash
  wget https://aka.ms/lis

  tar xvzf lis

  cd LISISO

  sudo ./install.sh

  sudo reboot

  ```
 
4. Återansluta till den virtuella datorn och kör den `lspci` kommando. Kontrollera att NVIDIA M60 kort eller kort visas som PCI-enheter.
 
5. Hämta och installera drivrutinen RUTNÄTET:

  ```bash
  wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=849941  

  chmod +x NVIDIA-Linux-x86_64-grid.run

  sudo ./NVIDIA-Linux-x86_64-grid.run
  ``` 
6. När du tillfrågas om du vill köra verktyget nvidia xconfig Välj om du vill uppdatera konfigurationsfilen X **Ja**.

7. När installationen är klar, kopiera /etc/nvidia/gridd.conf.template till en ny fil gridd.conf på plats/etc/nvidia /
  
  ```bash
  sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
  ```
  
8. Lägg till följande `/etc/nvidia/gridd.conf`:
 
  ```
  IgnoreSP=TRUE
  ```
9. Starta om den virtuella datorn och fortsätta med att verifiera installationen.

### <a name="verify-driver-installation"></a>Verifiera installation av drivrutiner


Fråga Enhetsstatus GPU SSH till den virtuella datorn och kör den [nvidia smi](https://developer.nvidia.com/nvidia-system-management-interface) kommandoradsverktyg som installeras med drivrutinen. 

Om drivrutinen är installerad visas utdata som liknar följande. Observera att **GPU-Util** visar 0% om du använder en GPU arbetsbelastning på den virtuella datorn. Din version av drivrutinen och information GPU kan skilja sig från de som visas.

![NVIDIA Enhetsstatus](./media/n-series-driver-setup/smi-nv.png)
 

### <a name="x11-server"></a>X11 server
Om du behöver en X11 server för fjärranslutningar till en NV VM [x11vnc](http://www.karlrunge.com/x11vnc/) rekommenderas eftersom det tillåter maskinvaruacceleration för grafik. BusID för M60 enheten måste läggas till manuellt i filen xconfig (`etc/X11/xorg.conf` på Ubuntu 16.04 LTS `/etc/X11/XF86config` på CentOS 7.3 eller Red Hat Enterprise Server 7.3). Lägg till en `"Device"` avsnitt som liknar följande:
 
```
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla M60"
    BusID          "your-BusID:0:0:0"
EndSection
```
 
Dessutom uppdatera din `"Screen"` avsnittet om du vill använda den här enheten.
 
Decimaltecknet BusID hittar du genom att köra

```bash
echo $((16#`/usr/bin/nvidia-smi --query-gpu=pci.bus_id --format=csv | tail -1 | cut -d ':' -f 1`))
```
 
BusID kan ändras när en virtuell dator hämtar omfördelats startas om. Därför kan du kanske vill skapa ett skript för att uppdatera BusID i X11 konfiguration när en virtuell dator startas. Till exempel skapa ett skript som heter `busidupdate.sh` (eller ett annat namn som du väljer) med följande innehåll:

```bash 
#!/bin/bash
BUSID=$((16#`/usr/bin/nvidia-smi --query-gpu=pci.bus_id --format=csv | tail -1 | cut -d ':' -f 1`))

if grep -Fxq "${BUSID}" /etc/X11/XF86Config; then     echo "BUSID is matching"; else   echo "BUSID changed to ${BUSID}" && sed -i '/BusID/c\    BusID          \"PCI:0@'${BUSID}':0:0:0\"' /etc/X11/XF86Config; fi
```

Skapa sedan en post för ditt skript för att uppdatera i `/etc/rc.d/rc3.d` så anropas skriptet som rot på Start.

## <a name="troubleshooting"></a>Felsökning

* Du kan ange beständiga med hjälp av `nvidia-smi` så resultatet av kommandot är snabbare när du behöver fråga kort. Om du vill ange beständiga läge kan köra `nvidia-smi -pm 1`. Observera att om den virtuella datorn har startats om inställningen för läge försvinner. Du kan alltid skript inställningen för läge för att köra vid start.

## <a name="next-steps"></a>Nästa steg

* För att avbilda en Linux-VM-avbildning med din installerade drivrutinerna finns [så att generalisera och avbilda en virtuell Linux-dator](capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
