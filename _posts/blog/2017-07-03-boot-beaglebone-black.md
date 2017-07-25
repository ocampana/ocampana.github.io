---
layout: post
title: "Come funziona il boot della beaglebone black"
modified:
categories: blog
excerpt: Come MLO, u-boot, kernel e rootfs contribuiscono al boot della scheda
tags: linux debian yocto beaglebone boot
keywords: linux debian stretch yocto pyro beaglebone black boot u-boot mlo
---

Al termine del post precedente la compilazione della `core-image-minimal` aveva generato svariati file, tra cui:
* `MLO`
* `u-boot.img`
* `zImage`
* `zImage-bbb-nohdmi.dtb`
* `core-image-minimal-beaglebone.tar.xz`

Il processore della Beaglebone Black, un Texas Instruments Sitara AM3358 è un processore molto ricco e offre diverse modalità di boot. Il suo boot può essere suddiviso nelle seguenti fasi:
1. il processore esegue un microcodice fuso all'interno del System-on-Chip (SoC)  detto ROM Code o bootloader di primo livello, per cercare i dispositivi di boot
2. il ROM code carica il bootloader di secondo livello o  x-loader, chiamato anche x-loader, SPL o MLO (si veda sotto perché ha così tanti nomi).
3. MLO effettua una minima configurazione del sistema e carica u-boot
4. u-boot inizializza altre periferiche, carica il kernel, predispone i suoi parametri di boot ed avvia il kernel
5. il kernel completa l'inizializzazione dell'hardware, monta il rootfs e dà vita a init che porta a termine l'inizializzazione del sistema

## ROM code

Questo piccolo programma è fuso all'interno del chip, non è modificabile, e viene eseguito automaticamente all'accensione del sistema.

Il ROM code ha due funzioni principali:
1. Configurare ed inizializzare correttamente le periferiche primarie:
    * inizializzare lo stack
    * configurare il watchdog ad un intervallo di tre minuti
    * configurare i PLL ed i clock di sistema
2. Preparare il dispositivo per il bootloader di secondo livello:
    * verificare i dispositivi da cui caricare il bootloder successivo
    * copiare in RAM il codice del bootloader da eseguire

La lista dei dispositivi da utilizzare nella ricerca del bootloader di secondo livello è configurata mediante dei pin di strap-in, indicati nella documentazione tecnica del Sitara come `SYSBOOT[15:0]`.

Nel caso della beaglebone, il funzionamento di questi pin è illustrato nella pagina 6 degli [schematici](https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SCH.pdf?raw=true).

|`SYSBOOT[15:5]`|`SYSBOOT[4:0]`|Sequenza di boot|
|:-|:-:|:-|
|_fissi_|`11100b`|MMC1, MMC0, UART0, USB0|
|_fissi_|`11000b`|SPI0, MMC0, USB0, UART0|

Come si può vedere a pagina 8 degli schematici per la eMMC e a pagina 11 per la microSD, la eMMC è connessa ad MMC1 e la microSD è connessa a MMC0. Sempre a pagina 6 si può vedere come è connesso il bottone `uSD BOOT`. Quando viene premuto porta il bit `SYSBOOT[2]` a 0 e quindi viene eseguita la seconda sequenza di boot, che va a ricercare il bootloader di secondo livello nella MMC0, mentre se non viene premuto il bit `SYSBOOT[2]` varrà 1 e quindi verrà eseguita la prima sequenza di boot, ch darà priorità al boot ad eMMC rispetto a quello da microSD. Le beaglebone sono tutte in vendita con un sistema linux embedded già programmato nella eMMC, quindi a meno di non cancellare la eMMC, se il bottone `uSD BOOT` non viene premuto il sistema partirà sempre da eMMC. In questo caso, volendo avviare un sistema custom da microSD, sarà sempre necessario accendere la beaglebone black tenendo premuto il bottone di boot da microSD.

|![descrizione delle interfacce dell beaglebone black](/images/beaglebone-black-interfaces.jpg)|
|Il bottone `uSD BOOT` è il `boot button` in questa immagine.|

Nel caso di boot da microSD, il ROM code cerca la prima partizione formattata in FAT32 e carica il file MLO contenuto in essa. Il file MLO contiene il bootloader di secondo livello ed alcuni byte di intestazione che istruiscono il ROM code sulla dimensione del file e l'indirizzo di memoria dove questo programma deve essere caricato.

## x-loader, il bootloader di secondo livello detto anche Secondary Program Loader (SPL)

Per spiegare cosa è x-loader è necesario un piccolo preambolo:

> Fino a qualche anno fa la compilazione di u-boot aveva come output un singolo file, u-boot.bin o u-boot.img che doveva essere caricato nelle schede come primo bootloader. L'immagine di u-boot è "grande", che nel mondo embedded vuol dire che è oltre 200KB. Questo può essere un problema, perché al momento del caricamento del bootloader in RAM non è detto che la DDR sia inizializzata e funzionante. I processori di Texas Instruments, anche prima del Sitara, avevano della RAM statica (SRAM) al loro interno, che al boot viene usata per l'inizializzazione del SoC. Per questo motivo, quando Texas Instruments ha iniziato a produrre i SoC della linea OMAP ha preso una parte dei sorgenti di u-boot per realizzare x-loader, una versione minimale di u-boot, il cui scopo è inizializzare la DDR e caricare uno u-boot completo. L'idea di avere un bootloader che usasse la RAM statica interna dei SoC per inizializzare la DDR è stata successivamente ripresa anche da altri produttori, per cui i sorgenti di x-loader e di altri bootloader minimali sono stati unificati con u-boot e per avere un unico bootloader ed evitare una proliferazione di minibootloader realizzati di diversi produttori di SoC.

I compiti di questo bootloader sono:
1. inizializzare il pin muxing del SoC
2. inizializzare i clock e la memoria DDR
3. caricare lo u-boot vero e proprio in memoria ed eseguirlo

I processori Sitara sono molto ricchi di periferiche e sebbene abbiano un migliaio di pin non sono in grado di esportare tutte le periferiche sulla loro footprint. Per questo motivo ed anche perché alcune periferiche possono essere connesse a piedini diversi in base alle necessità di sbroglio, è necessario configurare correttamente il multiplexing delle interfacce. Questo viene realizzato configurando opportunamente dei registri chiamati `PINMUX`, ed una prima configurazione deve essere fatta dal bootloader, per poter eseguire correttamente u-boot.

Dopo aver inizializzato la RAM DDR, il bootloader di secondo livello cerca il file u-boot.img nella stessa partizione da cui è stato caricato. Una volta che lo ha caricato il bootloader di secondo livello termina facendo un salto senza ritorno all'entry point di u-boot.

## U-boot

A questo punto tutta la procedura di boot a basso livello è stata completata e si arriva alla shell di u-boot, abbastanza familiare per chi lavora con i sistemi embedded.

Lo scopo di u-boot è:
1. configurare le periferiche per caricare il kernel
2. caricare il Device Tree Blob (DTB) file in memoria per istruire il kernel su come configurare le periferiche
3. caricare il kernel in memoria
4. configurare i bootargs, variabili d'ambiente con cui si configurano alcune opzioni del kernel
5. terminare la propria esecuzione, saltando senza ritorno all'entry point del kernel in RAM

## Ultima parte del processo di boot

Una volta avvia il kernel effettua l'inizializzazione del sistema:
1. decompressione e riposizionamento del kernel in RAM
2. inizializzazione delle periferiche rimanenti
3. inizializzazione di tutte le strutture dati necessarie al sistema operativo
4. inizializzazione degli stack di rete
5. `fork()` ed esecuzione del processo di init

All'esecuzione di `fork()` viene creato il processo con PID 1. Questo è il primo, ed in partenza unico, processo in userspace. Questo processo viene eseguito come root e deve inizializzare tutto lo userland, far partire processi di getty sui pseudoterminali o sulle seriali, montare i filesystem e gestire la rete. Se per qualche motivo questo processo termina il sistema crasha, di solito con questo messaggio:

```
Kernel panic - not syncing: Attempted to kill init!
```

## Dopo la teoria, la pratica

Per prima cosa è necesario partizionare la scheda microSD. La configurazione minima richiede due partizioni:
* una partizione FAT32 dove mettere almeno `MLO`, `u-boot.img` ed `uEnv.txt`
* una partizione EXT2 dove estrarre l'immagine compilata precedentemente

Nel mio computer la microSD viene mappata su /dev/sdd, nel vostro potrebbe essere un altro device file, in base ai dispositivi connessi al vostro PC.

Il partizionamento seguito per questo esempio è il seguente e va bene per delle prime prove, ma come vedremo in futuro è troppo limitato se dobbiamo gestire anche il software upgrade.

```bash
root@debian:~# fdisk /dev/sdd

Command (m for help): p

Disk /dev/sdd: 15.9 GB, 15931539456 bytes
255 heads, 63 sectors/track, 1936 cylinders, total 31116288 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

   Device Boot      Start         End      Blocks   Id  System
/dev/sdd1   *          63      257039      128488+   c  W95 FAT32 (LBA)
/dev/sdd2          257040    31116287    15429624   83  Linux

Command (m for help):
```

Quando formattiamo la prima partizione della microSD è meglio assicurarsi di usare come dimensione 16 per le File Allocation Table (FAT), mentre per la seconda possiamo usare inizialmente un filesystem ext2 standard:

```bash
root@debian:~# mkfs.vfat -F 16 -n "boot" /dev/sdd1
root@debian:~# mkfs.ext2 /dev/sdd2
```

A questo punto copiamo all'interno della partizione FAT i seguenti file:
* `MLO`
* `u-boot.img`
* `zImage`
* `zImage-bbb-nohdmi.dtb`

Sempre all'interno di questa partizione creiamo il file `uEnv.txt` con il seguente contenuto:

```
bootpart=0:1
bootdir=
bootfile=zImage
console=ttyO0,115200n8
fdtaddr=0x88000000
fdtfile=zImage-bbb-nohdmi.dtb
loadaddr=0x82000000
mmcroot=/dev/mmcblk0p2 ro
mmcrootfstype=ext2 rootwait
optargs=consoleblank=0
nohdmi=bbb-nohdmi.dtb
mmcargs=setenv bootargs console=${console} ${optargs} root=${mmcroot} rootfstype=${mmcrootfstype}
findfdtfile=if test -e mmc ${bootpart} ${bootdir}/nohdmi; then setenv fdtfile ${nohdmi}; fi;
loadfdt=run findfdtfile; load mmc ${bootpart} ${fdtaddr} ${bootdir}/${fdtfile}
loadimage=load mmc ${bootpart} ${loadaddr} ${bootdir}/${bootfile}
uenvcmd=if run loadfdt; then echo Loaded ${fdtfile}; if run loadimage; then run mmcargs; bootz ${loadaddr} - ${fdtaddr}; fi; fi;

```

Il file `uEnv.txt` contiene la configurazione di u-boot. Se non è presente u-boot userà i parametri hard coded che gli sono stati passati durante la compilazione. La spiegazione dettagliata dei parametri presenti in questo file verrà esposta in un prossimo post. **ATTENZIONE**: onde evitare problemi vari di compatibilità con alcune versioni di uboot, il file `uEnv.txt` deve essere salvato in formato Unix e deve avere come ultima riga una riga vuota.

La preparazione della partizione ext2 è molto più semplice, perché basta copiare il file `core-image-minimal-beaglebone.tar.xz` dentro alla partizione montata, posizionarsi nella directory dove è il file e lanciare il comando `tar xJfv core-image-minimal-beaglebone.tar.xz` . Notate come file è compresso con `xz` e non con i classici `gzip` o `bzip2`, quindi il flag per decomprimere questo tipo di archivio è `J`.

Per debuggare il boot è necessario connettersi alla prima UART della beaglebone black. Per fare questo la soluzione migliore è usare un adattatore USB.

L'UART è esportata sul connettore J1 e per connettermi uso un cavo simile a <a target="_blank" href="https://www.amazon.it/gp/product/B00AU00BWC/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=B00AU00BWC&linkCode=as2&tag=ocampana07-21&linkId=da5561f73542f8272375ba12b79b4776">questo</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=B00AU00BWC" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />. E' un cavo che converte emula da un lato una seriale su USB e dall'altro esporta tutti i segnali con livelli logici TTL.

Per connettere il cavo alla scheda bisogna collegare
* la massa al pin J1 (quello a destra nell'immagine)
* l'RX al pin 4
* il TX al pin 5

come mostrato in figura:

![come connettersi all'UART della beaglebone black](/images/beaglebone_black_uart.jpg)

Dopo aver configurato tutto, è necessario alimentare la scheda ricordandosi di tener premuto il bottone per il boot da microSD e questo è il risultato:

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
switch to partitions #0, OK
mmc0 is current device
SD/MMC found on device 0
reading uEnv.txt
934 bytes read in 4 ms (227.5 KiB/s)
Loaded env from uEnv.txt
Importing environment from mmc0 ...
Running uenvcmd ...
reading /zImage-bbb-nohdmi.dtb
32741 bytes read in 8 ms (3.9 MiB/s)
Loaded zImage-bbb-nohdmi.dtb
reading /zImage
4068696 bytes read in 259 ms (15 MiB/s)
## Flattened Device Tree blob at 88000000
   Booting using the fdt blob at 0x88000000
   Loading Device Tree to 8fff5000, end 8fffffe4 ... OK

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.9.33-jumpnow (oe-user@oe-host) (gcc version 6.3.0 (GCC) ) #1 Thu Jun 22 18:36:49 CEST 2017
[    0.000000] CPU: ARMv7 Processor [413fc082] revision 2 (ARMv7), cr=10c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt:Machine model: TI AM335x BeagleBone Black
[    0.000000] cma: Reserved 16 MiB at 0x9f000000
[    0.000000] Memory policy: Data cache writeback
[    0.000000] CPU: All CPU(s) started in SVC mode.
[    0.000000] AM335X ES2.1 (sgx neon)
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 130048
[    0.000000] Kernel command line: console=ttyO0,115200n8 consoleblank=0 root=/dev/mmcblk0p2 ro rootfstype=ext2 rootwait
[    0.000000] PID hash table entries: 2048 (order: 1, 8192 bytes)
[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes)
[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes)
[    0.000000] Memory: 484508K/524288K available (6144K kernel code, 387K rwdata, 1972K rodata, 1024K init, 8008K bss, 23396K reserved, 16384K cma-reserved, 0K highmem)
[    0.000000] Virtual kernel memory layout:
[    0.000000]     vector  : 0xffff0000 - 0xffff1000   (   4 kB)
[    0.000000]     fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
[    0.000000]     vmalloc : 0xe0800000 - 0xff800000   ( 496 MB)
[    0.000000]     lowmem  : 0xc0000000 - 0xe0000000   ( 512 MB)
[    0.000000]     pkmap   : 0xbfe00000 - 0xc0000000   (   2 MB)
[    0.000000]     modules : 0xbf000000 - 0xbfe00000   (  14 MB)
[    0.000000]       .text : 0xc0008000 - 0xc0700000   (7136 kB)
[    0.000000]       .init : 0xc0a00000 - 0xc0b00000   (1024 kB)
[    0.000000]       .data : 0xc0b00000 - 0xc0b60e68   ( 388 kB)
[    0.000000]        .bss : 0xc0b60e68 - 0xc13330fc   (8009 kB)
[    0.000000] Running RCU self tests
[    0.000000] NR_IRQS:16 nr_irqs:16 16
[    0.000000] IRQ: Found an INTC at 0xfa200000 (revision 5.0) with 128 interrupts
[    0.000000] OMAP clockevent source: timer2 at 24000000 Hz
[    0.000014] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
[    0.000034] clocksource: timer1: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000074] OMAP clocksource: timer1 at 24000000 Hz
[    0.000356] clocksource_probe: no matching clocksources found
[    0.001076] Console: colour dummy device 80x30
[    0.001127] Lock dependency validator: Copyright (c) 2006 Red Hat, Inc., Ingo Molnar
[    0.001135] ... MAX_LOCKDEP_SUBCLASSES:  8
[    0.001142] ... MAX_LOCK_DEPTH:          48
[    0.001148] ... MAX_LOCKDEP_KEYS:        8191
[    0.001154] ... CLASSHASH_SIZE:          4096
[    0.001160] ... MAX_LOCKDEP_ENTRIES:     32768
[    0.001167] ... MAX_LOCKDEP_CHAINS:      65536
[    0.001173] ... CHAINHASH_SIZE:          32768
[    0.001179]  memory used by lock dependency info: 5167 kB
[    0.001186]  per task-struct memory footprint: 1536 bytes
[    0.001215] Calibrating delay loop... 996.14 BogoMIPS (lpj=4980736)
[    0.078819] pid_max: default: 32768 minimum: 301
[    0.079076] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes)
[    0.079089] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes)
[    0.081275] CPU: Testing write buffer coherency: ok
[    0.082342] Setting up static identity map for 0x80100000 - 0x80100058
[    0.086928] devtmpfs: initialized
[    0.116562] VFP support v0.3: implementor 41 architecture 3 part 30 variant c rev 3
[    0.117334] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.117396] futex hash table entries: 256 (order: 1, 11264 bytes)
[    0.118768] pinctrl core: initialized pinctrl subsystem
[    0.122777] NET: Registered protocol family 16
[    0.129307] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.166837] omap_hwmod: debugss: _wait_target_disable failed
[    0.222289] cpuidle: using governor ladder
[    0.222313] cpuidle: using governor menu
[    0.246782] OMAP GPIO hardware version 0.1
[    0.284400] No ATAGs?
[    0.284427] hw-breakpoint: debug architecture 0x4 unsupported.
[    0.355483] edma 49000000.edma: TI EDMA DMA engine driver
[    0.360199] usbcore: registered new interface driver usbfs
[    0.360372] usbcore: registered new interface driver hub
[    0.360536] usbcore: registered new device driver usb
[    0.361382] omap_i2c 44e0b000.i2c: could not find pctldev for node /ocp/l4_wkup@44c00000/scm@210000/pinmux@800/pinmux_i2c0_pins, deferring probe
[    0.361482] omap_i2c 4802a000.i2c: could not find pctldev for node /ocp/l4_wkup@44c00000/scm@210000/pinmux@800/i2c1_pins, deferring probe
[    0.361553] omap_i2c 4819c000.i2c: could not find pctldev for node /ocp/l4_wkup@44c00000/scm@210000/pinmux@800/i2c2_pins, deferring probe
[    0.389731] clocksource: Switched to clocksource timer1
[    0.463762] VFS: Disk quotas dquot_6.6.0
[    0.463874] VFS: Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
[    0.512171] NET: Registered protocol family 2
[    0.513750] TCP established hash table entries: 4096 (order: 2, 16384 bytes)
[    0.513837] TCP bind hash table entries: 4096 (order: 5, 147456 bytes)
[    0.514990] TCP: Hash tables configured (established 4096 bind 4096)
[    0.515145] UDP hash table entries: 256 (order: 2, 20480 bytes)
[    0.515312] UDP-Lite hash table entries: 256 (order: 2, 20480 bytes)
[    0.516016] NET: Registered protocol family 1
[    0.517487] hw perfevents: enabled with armv7_cortex_a8 PMU driver, 5 counters available
[    0.529063] workingset: timestamp_bits=30 max_order=17 bucket_order=0
[    0.535578] io scheduler noop registered
[    0.535603] io scheduler deadline registered
[    0.535677] io scheduler cfq registered (default)
[    0.538519] pinctrl-single 44e10800.pinmux: 142 pins at pa f9e10800 size 568
[    0.544084] Serial: 8250/16550 driver, 4 ports, IRQ sharing enabled
[    0.550640] omap_uart 44e09000.serial: no wakeirq for uart0
[    0.551199] 44e09000.serial: ttyO0 at MMIO 0x44e09000 (irq = 158, base_baud = 3000000) is a OMAP UART0
[    1.123670] console [ttyO0] enabled
[    1.128984] omap_uart 481a8000.serial: no wakeirq for uart4
[    1.135176] 481a8000.serial: ttyO4 at MMIO 0x481a8000 (irq = 159, base_baud = 3000000) is a OMAP UART4
[    1.146618] omap_uart 481aa000.serial: no wakeirq for uart5
[    1.152791] 481aa000.serial: ttyO5 at MMIO 0x481aa000 (irq = 160, base_baud = 3000000) is a OMAP UART5
[    1.165757] omap_rng 48310000.rng: OMAP Random Number Generator ver. 20
[    1.211514] brd: module loaded
[    1.237434] loop: module loaded
[    1.242392] mtdoops: mtd device (mtddev=name/number) must be supplied
[    1.259824] libphy: Fixed MDIO Bus: probed
[    1.264613] CAN device driver interface
[    1.339817] davinci_mdio 4a101000.mdio: davinci mdio revision 1.6
[    1.346205] davinci_mdio 4a101000.mdio: detected phy mask fffffffe
[    1.355078] libphy: 4a101000.mdio: probed
[    1.359284] davinci_mdio 4a101000.mdio: phy[0]: device 4a101000.mdio:00, driver SMSC LAN8710/LAN8720
[    1.370132] cpsw 4a100000.ethernet: Detected MACID = 04:a3:16:b1:10:d5
[    1.380580] usbcore: registered new interface driver asix
[    1.386354] usbcore: registered new interface driver ax88179_178a
[    1.392876] usbcore: registered new interface driver cdc_ether
[    1.399119] usbcore: registered new interface driver smsc95xx
[    1.405253] usbcore: registered new interface driver net1080
[    1.411285] usbcore: registered new interface driver cdc_subset
[    1.417570] usbcore: registered new interface driver zaurus
[    1.423567] usbcore: registered new interface driver cdc_ncm
[    1.436582] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    1.443498] ehci-omap: OMAP-EHCI Host Controller driver
[    1.449325] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    1.456013] usbcore: registered new interface driver cdc_wdm
[    1.462147] usbcore: registered new interface driver usbtest
[    1.471019] 47401300.usb-phy supply vcc not found, using dummy regulator
[    1.486585] 47401b00.usb-phy supply vcc not found, using dummy regulator
[    1.498303] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
[    1.510878] musb-hdrc musb-hdrc.1.auto: new USB bus registered, assigned bus number 1
[    1.520929] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[    1.528038] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    1.535626] usb usb1: Product: MUSB HDRC host driver
[    1.540830] usb usb1: Manufacturer: Linux 4.9.33-jumpnow musb-hcd
[    1.547197] usb usb1: SerialNumber: musb-hdrc.1.auto
[    1.556615] hub 1-0:1.0: USB hub found
[    1.561550] hub 1-0:1.0: 1 port detected
[    1.572957] mousedev: PS/2 mouse device common for all mice
[    1.584128] omap_rtc 44e3e000.rtc: rtc core: registered 44e3e000.rtc as rtc0
[    1.592181] i2c /dev entries driver
[    1.595900] Driver for 1-wire Dallas network protocol.
[    1.607206] omap_wdt: OMAP Watchdog Timer Rev 0x01: initial timeout 60 sec
[    1.617027] omap_hsmmc 48060000.mmc: Got CD GPIO
[    1.743311] ledtrig-cpu: registered to indicate activity on CPUs
[    1.750259] usbcore: registered new interface driver usbhid
[    1.756086] usbhid: USB HID core driver
[    1.761170] oprofile: using arm/armv7
[    1.765580] Initializing XFRM netlink socket
[    1.770263] NET: Registered protocol family 17
[    1.775105] NET: Registered protocol family 15
[    1.779787] can: controller area network core (rev 20120528 abi 9)
[    1.786382] NET: Registered protocol family 29
[    1.791060] can: raw protocol (rev 20120528)
[    1.795609] can: broadcast manager protocol (rev 20161123 t)
[    1.801861] Key type dns_resolver registered
[    1.806498] omap_voltage_late_init: Voltage driver support not added
[    1.813760] ThumbEE CPU extension supported.
[    1.867224] mmc0: host does not support reading read-only switch, assuming write-enable
[    1.881112] mmc0: new high speed SDHC card at address aaaa
[    1.892372] mmcblk0: mmc0:aaaa SL16G 14.8 GiB
[    1.899725] random: fast init done
[    1.909992]  mmcblk0: p1 p2
[    1.928880] tps65217 0-0024: TPS65217 ID 0xe version 1.2
[    1.935234] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
[    1.944317] omap_i2c 4802a000.i2c: bus 1 rev0.11 at 100 kHz
[    1.953101] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 100 kHz
[    1.961135] omap_rtc 44e3e000.rtc: setting system clock to 2000-01-01 00:00:01 UTC (946684801)
[    1.977073] mmc1: new high speed MMC card at address 0001
[    1.987339] mmcblk1: mmc1:0001 S10004 3.56 GiB
[    1.993067] VFS: Mounted root (ext2 filesystem) readonly on device 179:2.
[    2.001612] mmcblk1boot0: mmc1:0001 S10004 partition 1 4.00 MiB
[    2.009038] mmcblk1boot1: mmc1:0001 S10004 partition 2 4.00 MiB
[    2.017376] devtmpfs: mounted
[    2.022191] Freeing unused kernel memory: 1024K (c0a00000 - c0b00000)
[    2.031244]  mmcblk1: p1 p2
INIT: version 2.88 booting
Starting udev
[    2.597955] udevd[732]: starting version 3.2.1
[    2.663815] udevd[733]: starting eudev-3.2.1
[    3.816587] EXT2-fs (mmcblk0p2): warning: mounting unchecked fs, running e2fsck is recommended
Populating dev cache
Thu Jun 22 17:12:57 UTC 2017
INIT: Entering runlevel: 5
Configuring network interfaces... [    5.129363] net eth0: initializing cpsw version 1.12 (0)
[    5.230533] SMSC LAN8710/LAN8720 4a101000.mdio:00: attached PHY driver [SMSC LAN8710/LAN8720] (mii_bus:phy_addr=4a101000.mdio:00, irq=-1)
udhcpc (v1.24.1) started
Sending discover...
Sending discover...
Sending discover...
No lease, forking to background
done.
Starting syslogd/klogd: done

Poky (Yocto Project Reference Distro) 2.3 beaglebone /dev/ttyO0

beaglebone login:
```

Eccoci quindi arrivati al termine del primo boot di Yocto Pyro per Beaglebone Black!

Notiamo alcuni punti importanti:
* Il ROM code non emette alcun output
* All'avvio MLO emette il seguente output `U-Boot SPL 2017.05-jumpnow (Jun 22 2017 - 19:16:50)`
* Dopo essere stato avviato da MLO, U-boot emette `U-Boot 2017.05-jumpnow (Jun 22 2017 - 19:16:50 +0200)`
* All'avvio il kernel emette `Starting kernel`
* Quando il kernel è avviato, lo userspace prende vita con l'avvio del processo di init, in concomitanza dell'output `INIT: Entering runlevel: 5`
* Al termine del setup dello userspace viene avviato getty sulla seriale, che richiede il login.
