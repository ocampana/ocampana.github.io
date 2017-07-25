---
layout: post
title: "Spiegazione dei parametri di u-boot"
modified:
categories: blog
excerpt: uEnv.txt contiene molti parametri, ma cosa fanno di preciso?
keywords: linux debian stretch yocto pyro beaglebone black boot u-boot mlo
---

Nel post precedente è stata preparata una scheda microSD con due partizioni, una VFAT per avviare u-boot ed una EXT2 da usare come rootfs.

U-boot è stato configurato con il file `uEnv.txt` il cui contenuto può risultare abbastanza criptico per i non addetti ai lavori. Per capirne il significato è meglio partire dalla shell di u-boot.

All'avvio della scheda, u-boot scrive il seguente output sulla seriale:


```
U-Boot SPL 2017.05-jumpnow (Jun 22 2017 - 19:16:50)
Trying to boot from MMC1
reading u-boot.img
reading u-boot.img


U-Boot 2017.05-jumpnow (Jun 22 2017 - 19:16:50 +0200)

CPU  : AM335X-GP rev 2.1
I2C:   ready
DRAM:  512 MiB
MMC:   OMAP SD/MMC: 0, OMAP SD/MMC: 1
Using default environment

<ethaddr> not set. Validating first E-fuse MAC
Net:   cpsw
Press SPACE to abort autoboot in 2 seconds
```

Se durante i due secondi viene premuta la barra spaziatrice il processo di boot si interrompe e u-boot propone la sua shell interattiva:


```
=> 
```

U-boot ha una serie di parametri predefiniti, nel caso della beaglebone black questi parametri sono un numero molto elevato. Per vederli è necessario lanciare il comando `printenv`:

```
=> printenv
arch=arm
args_mmc=run finduuid;setenv bootargs console=${console} ${optargs} root=PARTUUID=${uuid} rw rootfstype=${mmcrootfstype}
baudrate=115200
board=am335x
board_name=A335BNLT
board_rev=00C0
board_serial=2716BBBK2133
boot_a_script=load ${devtype} ${devnum}:${distro_bootpart} ${scriptaddr} ${prefix}${script}; source ${scriptaddr}
boot_efi_binary=load ${devtype} ${devnum}:${distro_bootpart} ${kernel_addr_r} efi/boot/bootarm.efi; if fdt addr ${fdt_addr_r}; then bootefi ${kernel_addr_r} ${fdt_addr_r};else bootefi ${kernel_addr_r} ${fdtcontroladdr};fi
boot_extlinux=sysboot ${devtype} ${devnum}:${distro_bootpart} any ${scriptaddr} ${prefix}extlinux/extlinux.conf
boot_fdt=try
boot_fit=0
boot_prefixes=/ /boot/
boot_script_dhcp=boot.scr.uimg
boot_scripts=boot.scr.uimg boot.scr
boot_targets=mmc0 legacy_mmc0 mmc1 legacy_mmc1 nand0 pxe dhcp
bootcmd=run envboot; setenv mmcdev 1; run envboot; setenv mmcdev 0; run findfdt; run distro_bootcmd
bootcmd_dhcp=if dhcp ${scriptaddr} ${boot_script_dhcp}; then source ${scriptaddr}; fi;setenv efi_fdtfile ${fdtfile}; if test -z "${fdtfile}" -a -n "${soc}"; then setenv efi_fdtfile ${soc}-${board}${boardver}.dtb; fi; setenv efi_old_vci ${bootp_vci};setenv efi_old_arch ${bootp_arch};setenv bootp_vci PXEClient:Arch:00010:UNDI:003000;setenv bootp_arch 0xa;if dhcp ${kernel_addr_r}; then tftpboot ${fdt_addr_r} dtb/${efi_fdtfile};if fdt addr ${fdt_addr_r}; then bootefi ${kernel_addr_r} ${fdt_addr_r}; else bootefi ${kernel_addr_r} ${fdtcontroladdr};fi;fi;setenv bootp_vci ${efi_old_vci};setenv bootp_arch ${efi_old_arch};setenv efi_fdtfile;setenv efi_old_arch;setenv efi_old_vci;
bootcmd_legacy_mmc0=setenv mmcdev 0; setenv bootpart 0:2 ; run mmcboot
bootcmd_legacy_mmc1=setenv mmcdev 1; setenv bootpart 1:2 ; run mmcboot
bootcmd_mmc0=setenv devnum 0; run mmc_boot
bootcmd_mmc1=setenv devnum 1; run mmc_boot
bootcmd_nand=run nandboot
bootcmd_pxe=dhcp; if pxe get; then pxe boot; fi
bootcount=2
bootdelay=2
bootdir=/boot
bootenvfile=uEnv.txt
bootfile=zImage
bootm_size=0x10000000
bootpart=0:2
bootscript=echo Running bootscript from mmc${mmcdev} ...; source ${loadaddr}
console=ttyO0,115200n8
cpu=armv7
dfu_alt_info_emmc=rawemmc raw 0 3751936
dfu_alt_info_mmc=boot part 0 1;rootfs part 0 2;MLO fat 0 1;MLO.raw raw 0x100 0x100;u-boot.img.raw raw 0x300 0x400;spl-os-args.raw raw 0x80 0x80;spl-os-image.raw raw 0x900 0x2000;spl-os-args fat 0 1;spl-os-image fat 0 1;u-boot.img fat 0 1;uEnv.txt fat 0 1
dfu_alt_info_ram=kernel ram 0x80200000 0x4000000;fdt ram 0x80f80000 0x80000;ramdisk ram 0x81000000 0x4000000
distro_bootcmd=for target in ${boot_targets}; do run bootcmd_${target}; done
efi_dtb_prefixes=/ /dtb/ /dtb/current/
envboot=mmc dev ${mmcdev}; if mmc rescan; then echo SD/MMC found on device ${mmcdev};if run loadbootenv; then echo Loaded env from ${bootenvfile};run importbootenv;fi;if test -n $uenvcmd; then echo Running uenvcmd ...;run uenvcmd;fi;fi;
eth1addr=04:a3:16:b1:10:d7
ethact=cpsw
ethaddr=04:a3:16:b1:10:d5
fdt_addr_r=0x88000000
fdtaddr=0x88000000
fdtfile=undefined
findfdt=if test $board_name = A335BONE; then setenv fdtfile am335x-bone.dtb; fi; if test $board_name = A335BNLT; then setenv fdtfile am335x-boneblack.dtb; fi; if test $board_name = BBBW; then setenv fdtfile am335x-boneblack-wireless.dtb; fi; if test $board_name = BBG1; then setenv fdtfile am335x-bonegreen.dtb; fi; if test $board_name = BBGW; then setenv fdtfile am335x-bonegreen-wireless.dtb; fi; if test $board_name = BBBL; then setenv fdtfile am335x-boneblue.dtb; fi; if test $board_name = A33515BB; then setenv fdtfile am335x-evm.dtb; fi; if test $board_name = A335X_SK; then setenv fdtfile am335x-evmsk.dtb; fi; if test $board_name = A335_ICE; then setenv fdtfile am335x-icev2.dtb; fi; if test $fdtfile = undefined; then echo WARNING: Could not determine device tree to use; fi;
finduuid=part uuid mmc ${bootpart} uuid
fit_bootfile=fitImage
fit_loadaddr=0x88000000
importbootenv=echo Importing environment from mmc${mmcdev} ...; env import -t ${loadaddr} ${filesize}
init_console=if test $board_name = A335_ICE; then setenv console ttyO3,115200n8;else setenv console ttyO0,115200n8;fi;
kernel_addr_r=0x82000000
load_efi_dtb=load ${devtype} ${devnum}:${distro_bootpart} ${fdt_addr_r} ${prefix}${efi_fdtfile}
loadaddr=0x82000000
loadbootenv=fatload mmc ${mmcdev} ${loadaddr} ${bootenvfile}
loadbootscript=load mmc ${mmcdev} ${loadaddr} boot.scr
loadfdt=load ${devtype} ${bootpart} ${fdtaddr} ${bootdir}/${fdtfile}
loadfit=run args_mmc; bootm ${loadaddr}#${fdtfile};
loadimage=load ${devtype} ${bootpart} ${loadaddr} ${bootdir}/${bootfile}
loadramdisk=load mmc ${mmcdev} ${rdaddr} ramdisk.gz
mmc_boot=if mmc dev ${devnum}; then setenv devtype mmc; run scan_dev_for_boot_part; fi
mmcboot=mmc dev ${mmcdev}; setenv devnum ${mmcdev}; setenv devtype mmc; if mmc rescan; then echo SD/MMC found on device ${mmcdev};if run loadimage; then if test ${boot_fit} -eq 1; then run loadfit; else run mmcloados;fi;fi;fi;
mmcdev=0
mmcloados=run args_mmc; if test ${boot_fdt} = yes || test ${boot_fdt} = try; then if run loadfdt; then bootz ${loadaddr} - ${fdtaddr}; else if test ${boot_fdt} = try; then bootz; else echo WARN: Cannot load the DT; fi; fi; else bootz; fi;
mmcrootfstype=ext4 rootwait
netargs=setenv bootargs console=${console} ${optargs} root=/dev/nfs nfsroot=${serverip}:${rootpath},${nfsopts} rw ip=dhcp
netboot=echo Booting from network ...; setenv autoload no; dhcp; run netloadimage; run netloadfdt; run netargs; bootz ${loadaddr} - ${fdtaddr}
netloadfdt=tftp ${fdtaddr} ${fdtfile}
netloadimage=tftp ${loadaddr} ${bootfile}
nfsopts=nolock
partitions=uuid_disk=${uuid_gpt_disk};name=rootfs,start=2MiB,size=-,uuid=${uuid_gpt_rootfs}
pxefile_addr_r=0x80100000
ramargs=setenv bootargs console=${console} ${optargs} root=${ramroot} rootfstype=${ramrootfstype}
ramboot=echo Booting from ramdisk ...; run ramargs; bootz ${loadaddr} ${rdaddr} ${fdtaddr}
ramdisk_addr_r=0x88080000
ramroot=/dev/ram0 rw
ramrootfstype=ext2
rdaddr=0x88080000
rootpath=/export/rootfs
scan_dev_for_boot=echo Scanning ${devtype} ${devnum}:${distro_bootpart}...; for prefix in ${boot_prefixes}; do run scan_dev_for_extlinux; run scan_dev_for_scripts; done;run scan_dev_for_efi;
scan_dev_for_boot_part=part list ${devtype} ${devnum} -bootable devplist; env exists devplist || setenv devplist 1; for distro_bootpart in ${devplist}; do if fstype ${devtype} ${devnum}:${distro_bootpart} bootfstype; then run scan_dev_for_boot; fi; done
scan_dev_for_efi=setenv efi_fdtfile ${fdtfile}; if test -z "${fdtfile}" -a -n "${soc}"; then setenv efi_fdtfile ${soc}-${board}${boardver}.dtb; fi; for prefix in ${efi_dtb_prefixes}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${efi_fdtfile}; then run load_efi_dtb; fi;done;if test -e ${devtype} ${devnum}:${distro_bootpart} efi/boot/bootarm.efi; then echo Found EFI removable media binary efi/boot/bootarm.efi; run boot_efi_binary; echo EFI LOAD FAILED: continuing...; fi; setenv efi_fdtfile
scan_dev_for_extlinux=if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}extlinux/extlinux.conf; then echo Found ${prefix}extlinux/extlinux.conf; run boot_extlinux; echo SCRIPT FAILED: continuing...; fi
scan_dev_for_scripts=for script in ${boot_scripts}; do if test -e ${devtype} ${devnum}:${distro_bootpart} ${prefix}${script}; then echo Found U-Boot script ${prefix}${script}; run boot_a_script; echo SCRIPT FAILED: continuing...; fi; done
scriptaddr=0x80000000
soc=am33xx
spiargs=setenv bootargs console=${console} ${optargs} root=${spiroot} rootfstype=${spirootfstype}
spiboot=echo Booting from spi ...; run spiargs; sf probe ${spibusno}:0; sf read ${loadaddr} ${spisrcaddr} ${spiimgsize}; bootz ${loadaddr}
spibusno=0
spiimgsize=0x362000
spiroot=/dev/mtdblock4 rw
spirootfstype=jffs2
spisrcaddr=0xe0000
static_ip=${ipaddr}:${serverip}:${gatewayip}:${netmask}:${hostname}::off
stderr=ns16550_serial
stdin=ns16550_serial
stdout=ns16550_serial
update_to_fit=setenv loadaddr ${fit_loadaddr}; setenv bootfile ${fit_bootfile}
vendor=ti
ver=U-Boot 2017.05-jumpnow (Jun 22 2017 - 19:16:50 +0200)

Environment size: 8250/131068 bytes
=>
```

Di tutti questi parametri ce ne sono alcuni che sono necessari per effettuare il boot del sistema compilato in precedenza, altri che servono per effettuare il boot di sistemi linux da altri dispositivi. In particolar modo i parametri più importanti sono:

* `bootdelay=2` è il numero di secondi che il bootloader attende prima di lanciare il comando di boot del sistema linux. Può essere utile aumentare il valore di questa variabile per dare un po' più di tempo all'operatore per interrompere il boot del sistema. Nel caso di prodotti finiti invece non è raro trovare sistemi configurati con `bootdelay=0`, che impedisce di lanciare una shell interattiva e fa subito lanciare il boot di sistema. Questo assieme ad altri accorgimenti viene usato per ridurre il tempo di boot.
* `bootpart=0:2` indica la partizione da cui effettuare il boot. E' della sintassi `x:y`, dove `x` indica il numero della microSD ed `y` è il numero della partizione. `x` vale 0 per indicare la microSD da cui è stato avviato u-boot, quindi nel caso in cui venga fatto il boot da microSD il valore 0 indica la microSD ed il valore 1 la eMMC, mentre se si avvia il sistema preinstallato nella memoria eMMC, ad `x=0` corrisponde la eMMC e a `x=1` corrisponde la microSD. Il valore di `y` indica invece l'indice della partizione dove cercare il kernel, che nel nostro caso è stato messo nella prima partizione VFAT, assieme ad MLO e u-boot.
* `bootdir=/boot` indica se il kernel deve essere cercato in una subdirectory o nella radice del filesystem. Nei sistemi GNU/Linux desktop tradizionalmente kernel e di file necessari all'avvio sono salvati nella cartella `/boot`, mentre nei sistemi embedded può essere vero o meno. Spesso la scelta di dove mettere kernel e bootloader è più dettata dalle strategie di firmware upgrade che motivi estetici.
* `console=ttyO0,115200n8` è uno dei parametri fondamentali: informa il kernel di redirigere il suo output della seriale. Solo in questo modo è possibile vedere come si comporta al boot e determinare quali sono i problemi che fanno fallire il boot del sistema. Per non sbagliare lo passo sempre, perché passarlo non costa nulla e non si sa mai come sia configurato un sistema. Non è questa variabile ad essere la diretta responsabile della redirezione dell'output del kernel sulla seriale, ma viene usata per formare i `bootargs`, ovvero la linea di comando che viene passata al kernel.
* `fdtaddr=0x88000000` specifica a quale indirizzo caricare in memoria il [Flattened Device Tree](https://www.devicetree.org/) (FDT), ovvero il file che indica come mappare tutte le periferiche del System-on-Chip.
* `fdtfile=undefined` specifica il nome di quale FDT utilizzare.
* `loadaddr=0x82000000` specifica invece a quale indirizzo di memoria caricare il kernel.
* `mmcroot=/dev/mmcblk0p2 ro` questa variabile viene usata per assemblare i `bootargs`, per indicare al kernel di linux dove troverà il rootfs.
* `mmcrootfstype=ext2 rootwait` anche questa varaibile viene usate per assemblare i `bootargs`, nello specifico per dire al kernel di linux che tipo di filesystem deve aspettarsi per il rootfs.
* `optargs=consoleblank=0` è una variabile che, sempre usata per assemblare i `bootargs`, dice al kernel di non mettere in risparmio energetico un eventuale monitor collegato alla porta HDMI della scheda.
* `nohdmi=bbb-nohdmi.dtb` indica il file contenente il Device Tree Database 
* `loadimage=load ${devtype} ${bootpart} ${loadaddr} ${bootdir}/${bootfile}` è il comando con cui u-boot carica il kernel.

Quando u-boot leggere il file `uEnv.txt` legge tutti i parametri impostati dall'utente e sovrascrive i valori di default. Nello specifico, vediamo cosa cambia con le variabili passate mediante il file `uEnv.txt` . Per semplificare la cosa le righe sono state riordinate


```
bootpart=0:1
bootdir=
bootfile=zImage
loadimage=load mmc ${bootpart} ${loadaddr} ${bootdir}/${bootfile}
```
Queste prime righe dicono di selezionare la prima partizione della scheda microSD e di caricare il file `zImage` direttamente dalla radice del filesystem.


```
console=ttyO0,115200n8
mmcroot=/dev/mmcblk0p2 ro
mmcrootfstype=ext2 rootwait
optargs=consoleblank=0
mmcargs=setenv bootargs console=${console} ${optargs} root=${mmcroot} rootfstype=${mmcrootfstype}
```

Di queste righe, le prime quattro vengono usate per specificare dei parametri usati per assemblare i `bootargs`. L'ultima riga definisce il comando `mmcargs`, che quando viene eseguito popola la variabile d'ambiente d'ambiente `bootargs` con tutti i parametri da passare al kernel.

```
fdtaddr=0x88000000
fdtfile=zImage-bbb-nohdmi.dtb
loadaddr=0x82000000
nohdmi=bbb-nohdmi.dtb
findfdtfile=if test -e mmc ${bootpart} ${bootdir}/nohdmi; then setenv fdtfile ${nohdmi}; fi;
loadfdt=run findfdtfile; load mmc ${bootpart} ${fdtaddr} ${bootdir}/${fdtfile}
```

Queste variabili ridefiniscono alcuni parametri necessari al caricamento del Device Tree Database da usare con la scheda

```
uenvcmd=if run loadfdt; then echo Loaded ${fdtfile}; if run loadimage; then run mmcargs; bootz ${loadaddr} - ${fdtaddr}; fi; fi;
```

Questa ultima riga definisce il comando predefinito che u-boot esegue quando spira il timeout `bootdelay`. Nello specifico, il comando esegue i seguenti passi:

1. Carica il device tree database
2. Carica il kernel
3. Esegue `mmcargs` per impostare i `bootargs`
4. Lancia il kernel

## Trivia

Dopo aver letto questa pagina dovrebbe essere chiaro che i `bootargs` giocano un ruolo fondamentale nel boot, perché cambiano non poco il comportamento del kernel. Talvolta per debug serve capire con quali bootargs il sistema sia partito avendo solo a disposizione solo lo GNU/Linux già funzionante.

Come si può vedere con quali `bootargs` è partito?

La risposta è leggendo `/proc/cmdline`:

```bash
root@beaglebone:~# cat /proc/cmdline
console=ttyO0,115200n8 consoleblank=0 root=/dev/mmcblk0p2 ro rootfstype=ext2 rootwait
root@beaglebone:~#
```
