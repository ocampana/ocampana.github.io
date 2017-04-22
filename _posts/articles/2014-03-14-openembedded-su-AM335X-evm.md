---
layout: post
title: "OpenEmbedded su AM335X-evm"
excerpt: "Linux embedded"
categories: articles
comments: true
tags: [linux, embedded]
redirect_from: "/software/linux/openembedded-su-am335x-evm"
---

I System-On-Chip di Texas instruments mi sono piaciuti in passato. Ultimamente ho dovuto lavorare sul Sitara AM335X Starter Kit e accipicchia, ho trovato il supporto fornito da Texas Instruments confusionario. Siccome ho tribolato un bel po', riporto le mie note su come generarsi con OpenEmbedded una immagine funzionante e customizzabile per questo SoC.

I link da cui ho preso ispirazione:

* [Sitara bootcamp](https://onedrive.live.com/view.aspx?resid=AE45EAC569CA5E92!127&app=PowerPoint&authkey=!AGirsdPYWY9Q8cg)
* [Sitara Linux Training: Getting Started with Openembedded](http://processors.wiki.ti.com/index.php/Sitara_Linux_Training:_Getting_Started_with_Openembedded)
* [Setting Up Build Environment](http://arago-project.org/wiki/index.php/Setting_Up_Build_Environment)
* [How to Make 3 Partition SD Card](http://processors.wiki.ti.com/index.php/How_to_Make_3_Partition_SD_Card)


Iniziamo con lo scaricare la toolchain:

    $ wget --no-check-certificate \
        https://launchpad.net/linaro-toolchain-binaries/trunk/2013.03/+download/gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux.tar.bz2
    $ tar -jxvf gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux.tar.bz2 -C $HOME


Installiamo i pacchetti necessari

    $ sudo apt-get install git build-essential diffstat texinfo gawk chrpath

Se stiamo usando una versione di Debian a 64 bit, serve installare le librerie a 32 bit, perché la toolchain di linaro è solo a 32 bit

    $ sudo apt-get install ia32-libs

Riconfiguriamo dash, affinché `/bin/sh` punti a `/bin/bash`, altrimenti OpenEmbedded e Yocto si scassano:

    $ sudo dpkg-reconfigure dash

Compiliamo una minimal image:

    $ git clone git://arago-project.org/git/projects/oe-layersetup.git tisdk
    $ cd tisdk
    $ ./oe-layertool-setup.sh -f ./configs/amsdk/amsdk-06.00.00.0-config.txt
    $ cd build
    $ . conf/setenv
    $ export PATH=$HOME/gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux/bin:$PATH
    $ MACHINE=am335x-evm bitbake core-image-minimal

A questo punto dobbiamo preparare la SD card, partizionandola con il seguente script (es: ./mk2PartSDCard.sh /dev/sde):

    #! /bin/sh
    # mk2PartSDCard.sh v0.1
    # Licensed under terms of GPLv2

    DRIVE=$1

    dd if=/dev/zero of=$DRIVE bs=1024 count=1024

    SIZE=`fdisk -l $DRIVE | grep Disk | awk '{print $5}'`

    echo DISK SIZE - $SIZE bytes

    CYLINDERS=`echo $SIZE/255/63/512 | bc`

    sfdisk -D -H 255 -S 63 -C $CYLINDERS $DRIVE << EOF
    ,9,0x0C,*
    10,114,,,
    EOF

    mkfs.vfat -F 32 -n "boot" ${DRIVE}1
    umount ${DRIVE}1
    mkfs.ext3 -L "rootfs" ${DRIVE}2
    # end of file

Lo script crea due partizioni, una boot in vfat e una rootfs in ext3. Nella boot vanno copiati i file per boottare il kernel, mentre nella rootfs va esplosa la tarball generata dalla compilazione.Si tratta quindi di copiare:

* `tisdk/build/arago-tmp-external-linaro-toolchain/deploy/images/MLO` nella partizione di boot
* `tisdk/build/arago-tmp-external-linaro-toolchain/deploy/images/u-boot.img` nella partizione di boot
* `tisdk/build/arago-tmp-external-linaro-toolchain/deploy/images/uImage` nella partizione di boot
* `tisdk/build/arago-tmp-external-linaro-toolchain/deploy/images/core-image-minimal-am335x-evm.tar.gz` nella partizione di rootfs, da scopattare come utente root


La `core-image-minimal-am335x-evm` funziona perfettamente. È inoltre possibile compilare la `tisdk-rootfs-am335x-evm`, con tutti gli esempi di TI. Purtroppo l'immagine che si ottiene ha un problema nella gestione del touchscreen, perché le coordinate non vengono interpretate correttamente. Se si scarica il `AM335xSDK 06_00_00_00` c'è una immagine precompilata che sembra identica, ma per qualche motivo non lo è... 
