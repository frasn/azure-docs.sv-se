---
title: Skapa och ladda upp en Red Hat Enterprise Linux VHD för användning i Azure | Microsoft Docs
description: Lär dig att skapa och ladda upp en Azure virtuell hårddisk (VHD) som innehåller en Red Hat Linux-operativsystem.
services: virtual-machines-linux
documentationcenter: ''
author: szarkos
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: 6c6b8f72-32d3-47fa-be94-6cb54537c69f
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 05/04/2018
ms.author: szark
ms.openlocfilehash: 9893ab83584e2fc93ea9cf29c0b74b957080d5ad
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2018
---
# <a name="prepare-a-red-hat-based-virtual-machine-for-azure"></a>Förbered en Red Hat-baserad virtuell dator för Azure
I den här artikeln får du lära dig hur du förbereder en virtuell dator för Red Hat Enterprise Linux (RHEL) för användning i Azure. De versioner av RHEL som beskrivs i den här artikeln är 6.7 + och 7.1 +. Hypervisorer för förberedelse som beskrivs i den här artikeln är Hyper-V, kernel-baserad virtuell dator (KVM) och VMware. Läs mer om behörighetskraven för deltagande i programmet för Red Hat Molnåtkomst [Red Hat Molnåtkomst webbplats](http://www.redhat.com/en/technologies/cloud-computing/cloud-access) och [kör RHEL på Azure](https://access.redhat.com/ecosystem/ccsp/microsoft-azure).

## <a name="prepare-a-red-hat-based-virtual-machine-from-hyper-v-manager"></a>Förbered en Red Hat-baserad virtuell dator från Hyper-V Manager

### <a name="prerequisites"></a>Förutsättningar
Det här avsnittet förutsätter att du redan har fått en ISO-fil från webbplatsen för Red Hat och installerat RHEL-avbildning till en virtuell hårddisk (VHD). Mer information om hur du använder Hyper-V Manager för att installera en operativsystemavbildning finns [installera Hyper-V-rollen och konfigurera en virtuell dator](http://technet.microsoft.com/library/hh846766.aspx).

**RHEL installationsinformation**

* Azure har inte stöd för VHDX-format. Azure stöder endast fast virtuell Hårddisk. Du kan använda Hyper-V Manager för att konvertera disken till VHD-format eller så kan använda cmdleten convert-vhd. Om du använder VirtualBox markerar **fast storlek** till skillnad från standardvärdet dynamiskt allokerade alternativet när du skapar disken.
* Azure stöder endast generering 1 för virtuella datorer. Du kan konvertera en virtuell dator i generation 1 från VHDX till VHD-format och dynamiskt expanderande till en disk med fast storlek. Du kan inte ändra en virtuell dator generation. Mer information finns i [bör jag skapa en virtuell dator i generation 1 eller 2 i Hyper-V?](https://technet.microsoft.com/windows-server-docs/compute/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v).
* Den maximala storleken som tillåts för den virtuella Hårddisken är 1,023 GB.
* Vi rekommenderar att du använder standard partitioner i stället för logisk volym Manager (LVM), vilket ofta är standard för många installationer när du installerar Linux-operativsystem. Denna praxis undviker LVM står i konflikt med den klonade virtuella datorer, särskilt om du behöver för att koppla en operativsystemdisk till en annan identisk virtuell dator för felsökning. [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) eller [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) får användas i datadiskar.
* Kernel-stöd för att montera Universal Disk (UDF) filsystem krävs. Vid första start på Azure skickar UDF-formaterad media som är kopplad till gästen etablering konfigurationen till den virtuella Linux-datorn. Azure Linux-agenten måste kunna montera UDF-filsystemet för att läsa konfigurationen och etablera den virtuella datorn.
* Versioner av Linux-kärnan som är äldre än 2.6.37 stöder inte icke-enhetlig minnesåtkomst (NUMA) på Hyper-V med större storlekar för virtuella datorer. Det här problemet påverkar huvudsakligen äldre distributioner som använder den överordnade Red Hat 2.6.32 kernel och åtgärdades i RHEL 6.6 (kernel-2.6.32-504). System som kör anpassade kärnor som är äldre än 2.6.37 eller RHEL-baserade kernlar som är äldre än 2.6.32-504 måste ange den `numa=off` starta parametern på kommandoraden kernel i grub.conf. Mer information finns i Red Hat [KB 436883](https://access.redhat.com/solutions/436883).
* Konfigurera inte en byte-partition på disken för operativsystemet. Linux-agenten kan konfigureras för att skapa en växlingsfil på tillfällig disken.  Mer information om detta finns i följande steg.
* Alla virtuella hårddiskar på Azure måste ha en virtuell storlek justeras till 1MB. Vid konvertering från en rå disk till virtuell Hårddisk måste du kontrollera att rådata diskens storlek är en multipel av 1MB innan konverteringen. Mer information finns i stegen nedan. Se även [Linux installationsinformation](create-upload-generic.md#general-linux-installation-notes) för mer information.

### <a name="prepare-a-rhel-6-virtual-machine-from-hyper-v-manager"></a>Förbered en RHEL 6 virtuell dator från Hyper-V Manager

1. Välj den virtuella datorn i Hyper-V Manager.

2. Klicka på **Anslut** att öppna ett konsolfönster för den virtuella datorn.

3. I RHEL 6 kan NetworkManager störa Azure Linux-agenten. Avinstallera paketet genom att köra följande kommando:
   
        # sudo rpm -e --nodeps NetworkManager

4. Skapa eller redigera den `/etc/sysconfig/network` filen och Lägg till följande text:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. Flytta (eller ta bort) udev-regler för att undvika att generera statiska regler för Ethernet-gränssnitt. Dessa regler orsaka problem när du klonar en virtuell dator i Microsoft Azure eller Hyper-V:

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
        
        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # sudo chkconfig network on

8. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

9. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

10. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Om du vill göra den här ändringen öppna `/boot/grub/menu.lst` i en textredigerare och se till att standard-kärnan innehåller följande parametrar:
    
        console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    
    Detta säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem.
    
    Dessutom rekommenderar vi att du tar bort följande parametrar:
    
        rhgb quiet crashkernel=auto
    
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten.  Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer. Den här konfigurationen kan vara problematiskt på mindre storlekar för virtuella datorer.

    >[!Important]
    RHEL 6.5 och tidigare måste också ange den `numa=off` kernel-parametern. Se Red Hat [KB 436883](https://access.redhat.com/solutions/436883).

11. Se till att secure shell (SSH)-server har installerats och konfigurerats för att starta när datorn startas, vilket vanligtvis är standard. Ändra /etc/ssh/sshd_config för att inkludera följande rad:

        ClientAliveInterval 180

12. Installera Azure Linux-agenten genom att köra följande kommando:

        # sudo yum install WALinuxAgent

        # sudo chkconfig waagent on

    Installera paketet WALinuxAgent tar bort NetworkManager och NetworkManager gör väldigt lätt paket om de inte har redan tagits bort i steg 3.

13. Skapa inte växlingsutrymme på operativsystemets disk.

    Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och att den kan tömmas när den virtuella datorn avetableras. Ändra följande parametrar i /etc/waagent.conf på rätt sätt när du har installerat Azure Linux-agenten i föregående steg:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

14. Avregistrera prenumerationen (vid behov) genom att köra följande kommando:

        # sudo subscription-manager unregister

15. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

16. Klicka på **åtgärd** > **Stäng** i Hyper-V Manager. Din Linux VHD är nu redo att överföras till Azure.


### <a name="prepare-a-rhel-7-virtual-machine-from-hyper-v-manager"></a>Förbered en RHEL 7 virtuell dator från Hyper-V Manager

1. Välj den virtuella datorn i Hyper-V Manager.

2. Klicka på **Anslut** att öppna ett konsolfönster för den virtuella datorn.

3. Skapa eller redigera den `/etc/sysconfig/network` filen och Lägg till följande text:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

4. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

5. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # sudo systemctl enable network

6. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Om du vill göra den här ändringen öppna `/etc/default/grub` i en textredigerare och redigera den `GRUB_CMDLINE_LINUX` parameter. Exempel:
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Detta säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem. Den här konfigurationen stänger också av nya RHEL 7 namnkonventionerna för nätverkskort. Vi rekommenderar dessutom att du tar bort följande parametrar:
   
        rhgb quiet crashkernel=auto
   
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten. Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer, vilket kan vara problematiskt på mindre storlekar för virtuella datorer.

8. När du är klar med redigering `/etc/default/grub`, kör följande kommando för att återskapa grub konfigurationen:

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

9. Se till att SSH-server har installerats och konfigurerats för att starta när datorn startas, vilket vanligtvis är standard. Ändra `/etc/ssh/sshd_config` till följande kommando:

        ClientAliveInterval 180

10. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

11. Installera Azure Linux-agenten genom att köra följande kommando:

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

12. Skapa inte växlingsutrymme på operativsystemets disk.

    Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och kan tömmas när den virtuella datorn avetableras. När du installerar Azure Linux-agenten i föregående steg, ändra följande parametrar i `/etc/waagent.conf` på rätt sätt:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. Om du vill att avregistrera prenumerationen, kör du följande kommando:

        # sudo subscription-manager unregister

14. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

15. Klicka på **åtgärd** > **Stäng** i Hyper-V Manager. Din Linux VHD är nu redo att överföras till Azure.


## <a name="prepare-a-red-hat-based-virtual-machine-from-kvm"></a>Förbered en Red Hat-baserad virtuell dator från KVM
### <a name="prepare-a-rhel-6-virtual-machine-from-kvm"></a>Förbered en virtuell dator för RHEL 6 från KVM

1. Ladda ned avbildningen KVM RHEL 6 från Red Hat-webbplats.

2. Ange ett rotlösenord.

    Generera ett krypterat lösenord och kopiera resultatet av kommandot:

        # openssl passwd -1 changeme

    Ange en rotlösenordet med guestfish:
        
        # guestfish --rw -a <image-name>
        > <fs> run
        > <fs> list-filesystems
        > <fs> mount /dev/sda1 /
        > <fs> vi /etc/shadow
        > <fs> exit

   Ändra rotanvändaren från andra fältet ””! till det krypterade lösenordet.

3. Skapa en virtuell dator i KVM qcow2-avbildningen. Ange typ av disk till **qcow2**, och enhetsmodell för virtuellt nätverk gränssnittet **virtio**. Sedan starta den virtuella datorn och logga in som rot.

4. Skapa eller redigera den `/etc/sysconfig/network` filen och Lägg till följande text:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

6. Flytta (eller ta bort) udev-regler för att undvika att generera statiska regler för Ethernet-gränssnitt. Dessa regler orsaka problem när du klonar en virtuell dator i Azure eller Hyper-V:

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

7. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # chkconfig network on

8. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # subscription-manager register --auto-attach --username=XXX --password=XXX

9. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Om du vill göra den här konfigurationen, öppna `/boot/grub/menu.lst` i en textredigerare och se till att standard-kärnan innehåller följande parametrar:
    
        console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    
    Detta säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem.
    
    Vi rekommenderar dessutom att du tar bort följande parametrar:
    
        rhgb quiet crashkernel=auto
    
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten. Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer, vilket kan vara problematiskt på mindre storlekar för virtuella datorer.

    >[!Important]
    RHEL 6.5 och tidigare måste också ange den `numa=off` kernel-parametern. Se Red Hat [KB 436883](https://access.redhat.com/solutions/436883).

10. Lägg till Hyper-V-modulerna initramfs:  

    Redigera `/etc/dracut.conf`, och Lägg till följande innehåll:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Återskapa initramfs:

        # dracut -f -v

11. Avinstallera molnet init:

        # yum remove cloud-init

12. Se till att SSH-server har installerats och konfigurerats för att starta när datorn startas:

        # chkconfig sshd on

    Ändra /etc/ssh/sshd_config för att inkludera följande rader:

        PasswordAuthentication yes
        ClientAliveInterval 180

13. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

14. Installera Azure Linux-agenten genom att köra följande kommando:

        # yum install WALinuxAgent

        # chkconfig waagent on

15. Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och kan tömmas när den virtuella datorn avetableras. När du installerar Azure Linux-agenten i föregående steg, ändra följande parametrar i **/etc/waagent.conf** på rätt sätt:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. Avregistrera prenumerationen (vid behov) genom att köra följande kommando:

        # subscription-manager unregister

17. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # waagent -force -deprovision

        # export HISTSIZE=0

        # logout

18. Stäng av den virtuella datorn i KVM.

19. Konvertera qcow2 bilden till VHD-format.

> [!NOTE]
> Det finns ett känt fel i qemu img versioner > = 2.2.1 som resulterar i en felaktigt formaterad virtuell Hårddisk. Problemet har åtgärdats i QEMU 2.6. Det rekommenderas att använda qemu img 2.2.0 eller lägre eller uppdatera till 2.6 eller senare. Referens: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f qcow2 -O raw rhel-6.9.qcow2 rhel-6.9.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-6.9.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-6.9.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.9.raw rhel-6.9.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-6.9.raw rhel-6.9.vhd

        
### <a name="prepare-a-rhel-7-virtual-machine-from-kvm"></a>Förbered en virtuell dator för RHEL 7 från KVM

1. Ladda ned avbildningen KVM RHEL 7 från Red Hat-webbplats. Den här proceduren använder RHEL 7 som exemplet.

2. Ange ett rotlösenord.

    Generera ett krypterat lösenord och kopiera resultatet av kommandot:

        # openssl passwd -1 changeme

    Ange en rotlösenordet med guestfish:

        # guestfish --rw -a <image-name>
        > <fs> run
        > <fs> list-filesystems
        > <fs> mount /dev/sda1 /
        > <fs> vi /etc/shadow
        > <fs> exit

   Ändra rotanvändaren från andra fältet ””! till det krypterade lösenordet.

3. Skapa en virtuell dator i KVM qcow2-avbildningen. Ange typ av disk till **qcow2**, och enhetsmodell för virtuellt nätverk gränssnittet **virtio**. Sedan starta den virtuella datorn och logga in som rot.

4. Skapa eller redigera den `/etc/sysconfig/network` filen och Lägg till följande text:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

5. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

6. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # sudo systemctl enable network

7. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # subscription-manager register --auto-attach --username=XXX --password=XXX

8. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Om du vill göra den här konfigurationen, öppna `/etc/default/grub` i en textredigerare och redigera den `GRUB_CMDLINE_LINUX` parameter. Exempel:
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Detta kommando säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem. Kommandot stänger också av nya RHEL 7 namnkonventionerna för nätverkskort. Vi rekommenderar dessutom att du tar bort följande parametrar:
   
        rhgb quiet crashkernel=auto
   
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten. Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer, vilket kan vara problematiskt på mindre storlekar för virtuella datorer.

9. När du är klar med redigering `/etc/default/grub`, kör följande kommando för att återskapa grub konfigurationen:

        # grub2-mkconfig -o /boot/grub2/grub.cfg

10. Lägg till Hyper-V-modulerna i initramfs.

    Redigera `/etc/dracut.conf` och lägga till innehåll:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Återskapa initramfs:

        # dracut -f -v

11. Avinstallera molnet init:

        # yum remove cloud-init

12. Se till att SSH-server har installerats och konfigurerats för att starta när datorn startas:

        # systemctl enable sshd

    Ändra /etc/ssh/sshd_config för att inkludera följande rader:

        PasswordAuthentication yes
        ClientAliveInterval 180

13. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

14. Installera Azure Linux-agenten genom att köra följande kommando:

        # yum install WALinuxAgent

    Aktivera tjänsten waagent:

        # systemctl enable waagent.service

15. Skapa inte växlingsutrymme på operativsystemets disk.

    Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och kan tömmas när den virtuella datorn avetableras. När du installerar Azure Linux-agenten i föregående steg, ändra följande parametrar i `/etc/waagent.conf` på rätt sätt:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

16. Avregistrera prenumerationen (vid behov) genom att köra följande kommando:

        # subscription-manager unregister

17. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

18. Stäng av den virtuella datorn i KVM.

19. Konvertera qcow2 bilden till VHD-format.

> [!NOTE]
> Det finns ett känt fel i qemu img versioner > = 2.2.1 som resulterar i en felaktigt formaterad virtuell Hårddisk. Problemet har åtgärdats i QEMU 2.6. Det rekommenderas att använda qemu img 2.2.0 eller lägre eller uppdatera till 2.6 eller senare. Referens: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-7.4.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd


## <a name="prepare-a-red-hat-based-virtual-machine-from-vmware"></a>Förbered en Red Hat-baserad virtuell dator från VMware
### <a name="prerequisites"></a>Förutsättningar
Det här avsnittet förutsätter att du redan har installerat en RHEL virtuell dator i VMware. Mer information om hur du installerar ett operativsystem i VMware finns [VMware gäst operativsystemet installationsguiden](http://partnerweb.vmware.com/GOSIG/home.html).

* När du installerar Linux-operativsystem, rekommenderar vi att du använder standard partitioner i stället för LVM, vilket ofta är standard för många installationer. Det här undviker LVM står i konflikt med den klonade virtuella datorn, särskilt om en operativsystemdisk någonsin måste vara kopplad till en annan virtuell dator för felsökning. LVM eller RAID kan användas på datadiskar om önskade.
* Konfigurera inte en byte-partition på disken för operativsystemet. Du kan konfigurera Linux-agenten för att skapa en växlingsfil på tillfällig disken. Du hittar mer information om detta i de steg som följer.
* När du skapar den virtuella hårddisken, Välj **Store virtuell disk som en enda fil**.

### <a name="prepare-a-rhel-6-virtual-machine-from-vmware"></a>Förbered en virtuell dator för RHEL 6 från VMware
1. I RHEL 6 kan NetworkManager störa Azure Linux-agenten. Avinstallera paketet genom att köra följande kommando:
   
        # sudo rpm -e --nodeps NetworkManager

2. Skapa en fil med namnet **nätverk** i/etc/sysconfig/katalogen som innehåller följande text:

        NETWORKING=yes
        HOSTNAME=localhost.localdomain

3. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no

4. Flytta (eller ta bort) udev-regler för att undvika att generera statiska regler för Ethernet-gränssnitt. Dessa regler orsaka problem när du klonar en virtuell dator i Azure eller Hyper-V:

        # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

        # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules

5. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # sudo chkconfig network on

6. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

7. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-6-server-extras-rpms

8. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Det gör du genom att öppna `/etc/default/grub` i en textredigerare och redigera den `GRUB_CMDLINE_LINUX` parameter. Exempel:
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0"
   
   Detta säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem. Vi rekommenderar dessutom att du tar bort följande parametrar:
   
        rhgb quiet crashkernel=auto
   
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten. Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer, vilket kan vara problematiskt på mindre storlekar för virtuella datorer.

9. Lägg till Hyper-V-modulerna initramfs:

    Redigera `/etc/dracut.conf`, och Lägg till följande innehåll:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Återskapa initramfs:

        # dracut -f -v

10. Se till att SSH-server har installerats och konfigurerats för att starta när datorn startas, vilket vanligtvis är standard. Ändra `/etc/ssh/sshd_config` till följande kommando:

    ClientAliveInterval 180

11. Installera Azure Linux-agenten genom att köra följande kommando:

        # sudo yum install WALinuxAgent

        # sudo chkconfig waagent on

12. Skapa inte växlingsutrymme på operativsystemets disk.

    Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och kan tömmas när den virtuella datorn avetableras. När du installerar Azure Linux-agenten i föregående steg, ändra följande parametrar i `/etc/waagent.conf` på rätt sätt:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

13. Avregistrera prenumerationen (vid behov) genom att köra följande kommando:

        # sudo subscription-manager unregister

14. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

15. Stäng av den virtuella datorn och omvandla VMDK-filen till en VHD-fil.

> [!NOTE]
> Det finns ett känt fel i qemu img versioner > = 2.2.1 som resulterar i en felaktigt formaterad virtuell Hårddisk. Problemet har åtgärdats i QEMU 2.6. Det rekommenderas att använda qemu img 2.2.0 eller lägre eller uppdatera till 2.6 eller senare. Referens: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f vmdk -O raw rhel-6.9.vmdk rhel-6.9.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-6.9.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-6.9.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.9.raw rhel-6.9.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-6.9.raw rhel-6.9.vhd


### <a name="prepare-a-rhel-7-virtual-machine-from-vmware"></a>Förbered en virtuell dator för RHEL 7 från VMware
1. Skapa eller redigera den `/etc/sysconfig/network` filen och Lägg till följande text:
   
        NETWORKING=yes
        HOSTNAME=localhost.localdomain

2. Skapa eller redigera den `/etc/sysconfig/network-scripts/ifcfg-eth0` filen och Lägg till följande text:
   
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no

3. Se till att nätverkstjänsten som startar när datorn startas genom att köra följande kommando:

        # sudo systemctl enable network

4. Registrera prenumerationen Red Hat för att aktivera installationen av paket från RHEL databasen genom att köra följande kommando:

        # sudo subscription-manager register --auto-attach --username=XXX --password=XXX

5. Ändra raden kernel Start i grub konfigurationen så att ytterligare kernel parametrar för Azure. Om du vill göra den här ändringen öppna `/etc/default/grub` i en textredigerare och redigera den `GRUB_CMDLINE_LINUX` parameter. Exempel:
   
        GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
   
   Den här konfigurationen säkerställer också att alla konsolmeddelanden skickas till den första seriella porten som kan hjälpa Azure support med felsökning av problem. Även stängs av nya RHEL 7 namnkonventionerna för nätverkskort. Vi rekommenderar dessutom att du tar bort följande parametrar:
   
        rhgb quiet crashkernel=auto
   
    Grafisk och tyst Start inte är användbara i en molnmiljö där vi vill att alla loggar som ska skickas till den seriella porten. Du kan lämna den `crashkernel` alternativet konfigureras om så önskas. Observera att den här parametern minskar mängden tillgängligt minne i den virtuella datorn med 128 MB eller mer, vilket kan vara problematiskt på mindre storlekar för virtuella datorer.

6. När du är klar med redigering `/etc/default/grub`, kör följande kommando för att återskapa grub konfigurationen:

        # sudo grub2-mkconfig -o /boot/grub2/grub.cfg

7. Lägg till Hyper-V-modulerna initramfs.

    Redigera `/etc/dracut.conf`, lägga till innehåll:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

    Återskapa initramfs:

        # dracut -f -v

8. Se till att SSH-server har installerats och konfigurerats för att starta när datorn startas. Den här inställningen är vanligtvis standard. Ändra `/etc/ssh/sshd_config` till följande kommando:

        ClientAliveInterval 180

9. Paketets WALinuxAgent `WALinuxAgent-<version>`, har aviserats i databasen för Red Hat-tillägg. Aktivera tillägg databasen genom att köra följande kommando:

        # subscription-manager repos --enable=rhel-7-server-extras-rpms

10. Installera Azure Linux-agenten genom att köra följande kommando:

        # sudo yum install WALinuxAgent

        # sudo systemctl enable waagent.service

11. Skapa inte växlingsutrymme på operativsystemets disk.

    Azure Linux-agenten kan automatiskt konfigurera växlingsutrymme med hjälp av den lokala resurs disk som är kopplad till den virtuella datorn när den virtuella datorn har etablerats på Azure. Observera att den lokala resursen är en tillfällig disk och kan tömmas när den virtuella datorn avetableras. När du installerar Azure Linux-agenten i föregående steg, ändra följande parametrar i `/etc/waagent.conf` på rätt sätt:

        ResourceDisk.Format=y
        ResourceDisk.Filesystem=ext4
        ResourceDisk.MountPoint=/mnt/resource
        ResourceDisk.EnableSwap=y
        ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.

12. Om du vill att avregistrera prenumerationen, kör du följande kommando:

        # sudo subscription-manager unregister

13. Kör följande kommandon för att ta bort etableringen av den virtuella datorn och förbereda den för att etablera i Azure:

        # sudo waagent -force -deprovision

        # export HISTSIZE=0

        # logout

14. Stäng den virtuella datorn och omvandla VMDK-filen till VHD-format.

> [!NOTE]
> Det finns ett känt fel i qemu img versioner > = 2.2.1 som resulterar i en felaktigt formaterad virtuell Hårddisk. Problemet har åtgärdats i QEMU 2.6. Det rekommenderas att använda qemu img 2.2.0 eller lägre eller uppdatera till 2.6 eller senare. Referens: https://bugs.launchpad.net/qemu/+bug/1490611.
>


    First convert the image to raw format:

        # qemu-img convert -f vmdk -O raw rhel-7.4.vmdk rhel-7.4.raw

    Make sure that the size of the raw image is aligned with 1 MB. Otherwise, round up the size to align with 1 MB:

        # MB=$((1024*1024))
        # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
          gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

        # rounded_size=$((($size/$MB + 1)*$MB))
        # qemu-img resize rhel-7.4.raw $rounded_size

    Convert the raw disk to a fixed-sized VHD:

        # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd

    Or, with qemu version **2.6+** include the `force_size` option:

        # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd


## <a name="prepare-a-red-hat-based-virtual-machine-from-an-iso-by-using-a-kickstart-file-automatically"></a>Förbereda en Red Hat-baserad virtuell dator från en ISO med hjälp av en kickstart fil automatiskt
### <a name="prepare-a-rhel-7-virtual-machine-from-a-kickstart-file"></a>Förbered en RHEL 7 virtuell dator från en kickstart-fil

1.  Skapa en kickstart-fil som innehåller följande innehåll och spara filen. Mer information om kickstart installation finns den [Kickstart installationsguiden](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html).

        # Kickstart for provisioning a RHEL 7 Azure VM

        # System authorization information
          auth --enableshadow --passalgo=sha512

        # Use graphical install
        text

        # Do not run the Setup Agent on first boot
        firstboot --disable

        # Keyboard layouts
        keyboard --vckeymap=us --xlayouts='us'

        # System language
        lang en_US.UTF-8

        # Network information
        network  --bootproto=dhcp

        # Root password
        rootpw --plaintext "to_be_disabled"

        # System services
        services --enabled="sshd,waagent,NetworkManager"

        # System timezone
        timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

        # Partition clearing information
        clearpart --all --initlabel

        # Clear the MBR
        zerombr

        # Disk partitioning information
        part /boot --fstype="xfs" --size=500
        part / --fstyp="xfs" --size=1 --grow --asprimary

        # System bootloader configuration
        bootloader --location=mbr

        # Firewall configuration
        firewall --disabled

        # Enable SELinux
        selinux --enforcing

        # Don't configure X
        skipx

        # Power down the machine after install
        poweroff

        %packages
        @base
        @console-internet
        chrony
        sudo
        parted
        -dracut-config-rescue

        %end

        %post --log=/var/log/anaconda/post-install.log

        #!/bin/bash

        # Register Red Hat Subscription
        subscription-manager register --username=XXX --password=XXX --auto-attach --force

        # Install latest repo update
        yum update -y

        # Enable extras repo
        subscription-manager repos --enable=rhel-7-server-extras-rpms

        # Install WALinuxAgent
        yum install -y WALinuxAgent

        # Unregister Red Hat subscription
        subscription-manager unregister

        # Enable waaagent at boot-up
        systemctl enable waagent

        # Disable the root account
        usermod root -p '!!'

        # Configure swap in WALinuxAgent
        sed -i 's/^\(ResourceDisk\.EnableSwap\)=[Nn]$/\1=y/g' /etc/waagent.conf
        sed -i 's/^\(ResourceDisk\.SwapSizeMB\)=[0-9]*$/\1=2048/g' /etc/waagent.conf

        # Set the cmdline
        sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

        # Enable SSH keepalive
        sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

        # Build the grub cfg
        grub2-mkconfig -o /boot/grub2/grub.cfg

        # Configure network
        cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
        DEVICE=eth0
        ONBOOT=yes
        BOOTPROTO=dhcp
        TYPE=Ethernet
        USERCTL=no
        PEERDNS=yes
        IPV6INIT=no
        NM_CONTROLLED=no
        EOF

        # Deprovision and prepare for Azure
        waagent -force -deprovision

        %end

2. Placera filen kickstart där installationen system kan komma åt den.

3. Skapa en ny virtuell dator i Hyper-V Manager. På den **Anslut virtuell hårddisk** väljer **koppla en virtuell hårddisk senare**, och slutför guiden Ny virtuell dator.

4. Öppna inställningarna för virtuella datorer:

    a.  Koppla en ny virtuell hårddisk till den virtuella datorn. Se till att välja **VHD-Format** och **med fast storlek**.

    b.  Koppla ISO installationen från DVD-enheten.

    c.  Ange BIOS för start från CD: N.

5. Startar den virtuella datorn. När installationsguiden visas trycker **fliken** konfigurera startalternativ för.

6. Ange `inst.ks=<the location of the kickstart file>` i slutet av startalternativ och tryck på **RETUR**.

7. Vänta på att installationen ska slutföras. När den är klar, stängs den virtuella datorn automatiskt. Din Linux VHD är nu redo att överföras till Azure.

## <a name="known-issues"></a>Kända problem
### <a name="the-hyper-v-driver-could-not-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>Hyper-V-drivrutinen kan inte ingå i inledande RAM-disk när du använder en icke-Hyper-V hypervisor

I vissa fall kanske Linux installationsprogram inte innehåller drivrutiner för Hyper-V på den första RAM-disken (initrd eller initramfs) såvida inte Linux upptäcker att den körs i en miljö med Hyper-V.

När du använder en annan virtualisering system (det vill säga Virtualbox, Xen osv.) för att förbereda avbildningen Linux kan du behöva återskapa initrd för att se till att minst kernel-modulerna hv_vmbus och hv_storvsc är tillgängliga på den första disken RAM-minne. Detta är ett känt problem minst på system som är baserade på den överordnade Red Hat-distributionen.

Lägg till Hyper-V-modulerna i initramfs och återskapa den för att lösa problemet:

Redigera `/etc/dracut.conf`, och Lägg till följande innehåll:

        add_drivers+="hv_vmbus hv_netvsc hv_storvsc"

Återskapa initramfs:

        # dracut -f -v

Mer information finns i informationen [återskapa initramfs](https://access.redhat.com/solutions/1958).

## <a name="next-steps"></a>Nästa steg
Nu är du redo att använda din virtuella hårddisk Red Hat Enterprise Linux för att skapa nya virtuella datorer i Azure. Om att du överföra VHD-filen till Azure Se [skapa ett Linux VM från en anpassad disk](upload-vhd.md#option-1-upload-a-vhd).

Mer information om hypervisorer som är certifierade för att köra Red Hat Enterprise Linux finns [Red Hat-webbplatsen](https://access.redhat.com/certified-hypervisors).
