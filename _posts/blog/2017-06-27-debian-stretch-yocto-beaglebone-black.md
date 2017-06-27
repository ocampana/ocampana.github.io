---
layout: post
title: "Compilare Yocto per la Beaglebone Black su Debian GNU/Linux Stretch"
modified:
categories: blog
excerpt:
tags: linux debian yocto beaglebone
keywords: linux debian stretch yocto beaglebone black
---

Essendo stata rilasciata la nuova Debian Stable è possibile scrivere una guida aggiornata su Yocto e la BeagleBone Black.

Tutti inizia scaricando il [CD Netinst](https://www.debian.org/CD/netinst/) di Debian Stretch. Di solito scarico questo CD perché è il CD più piccolo possibile con l'installer Debian, tutti gli altri pacchetti sono disponibili dagli archivi Debian. Se avete dubbi su come installare Debian, esiste un ricco [manuale di installazione](https://www.debian.org/releases/stable/installmanual). Nei computer dedicati allo sviluppo embedded io installo mediante tasksel solo i pacchetti

* server SSH
* utilità di sistema standard

mentre per tutto il resto uso quando serve `apt-get`.

Il mi editor preferito è Vim, e mi assicuro sempre che sia l'unico editor da riga di comando installato, affinché sia sempre richiamato automaticamente da tutti gli altri programmi, quali per esempio git:

```bash
root@debian:~# apt-get install vim
root@debian:~# echo "syntax on" > /etc/vim/vimrc.local
root@debian:~# dpkg --purge nano vim-tiny
```

Un altro punto fondamentale è cambiare la shell di default. In Debian infatti `/bin/sh` di default è un link a `/bin/dash`, ma per far funzionare correttamente Yocto è necessario fare in modo che `/bin/sh` punti a `/bin/bash` con il seguente comando

```bash
root@debian:~# dpkg-reconfigure dash
```

e rispondere no.

&Egrave; inoltre bene aggiungere il supporto al locale `en_US.UTF-8`, altrimenti si otterrà un errore al lancio della compilazione di Yocto:

```bash
root@debian:~# dpkg-reconfigure locales
```

va aggiunto il locale `en_US.UTF-8`, ma non è necessario che sia selezionato come predefinito.

Dash viene usata da Debian ed Ubuntu perché implementa un sottoinsieme delle funzionalità di Bash ed è più rapida da eseguire. Questo risulta essere molto vantaggioso negli script di avvio del sistema, perché riduce i tempi necessari all'ovvio dei processi e degli script. Purtroppo Yocto e bitbake richiedono alcune funzioni non presenti di Dash, quindi la compilazione è destinata a fallire se non si cambia la shell predefinita di sistema.

A questo punto è necessario installare alcuni pacchetti


```bash
root@debian:~# apt-get install gawk wget git-core diffstat unzip \
               texinfo gcc-multilib build-essential chrpath socat cpio \
	       python python3 python3-pip python3-pexpect xz-utils \
	       debianutils iputils-ping libsdl1.2-dev
```

Fatto questo non serve più essere root ed è possibile lavorare su Yocto come utente non privilegiato. Per prima cosa è necessario fare il checkout di Poky. Poky è la distribuzione del progetto Yocto, che include al suo interno bitbake ed una parte di OpenEmbedded, oltre a tutto un insieme di metadati specifici del progetto Yocto. Nel repository di Poky sono disponibili tutti i rami che corrispondono a tutte le versioni rilasciate. Per scaricare l'ultima versione stabile, la 2.3 al momento in cui questa pagina viene scritta, è necessario selezionare il ramo `pyro`.

```bash
user@debian:~$ git clone git://git.yoctoproject.org/poky
user@debian:~$ cd poky
user@debian:~/poky$ git checkout pyro
```

A questo punto dentro la directory `poky` sono disponibili alcuni file ed altre sottocartelle:

```bash
user@debian:~/poky$ ls
bitbake        meta-poky      meta-yocto-bsp            README.hardware
documentation  meta-selftest  oe-init-build-env         scripts
LICENSE        meta-skeleton  oe-init-build-env-memres
meta           meta-yocto     README
```

Le cartelle `meta-*` contengono i metadati, ovvero tutte le informazioni per compilare tutti i componenti software. Sono organizzati in _layer_, ovvero in gruppi di informazioni raccolte per temi. Per la beaglebone black per esempio è necessario aggiungere i layer chiamati `meta-bbb`, `meta-openembedded` e `meta-qt5`. Va notato che attualmente non esiste un branch pyro per meta-qt5, quindi viene fatto il checkout di master. (per completezza a fine pagina riporto i commit id dei vari repository, affinché possiate replicare la build anche nel caso in cui i branch vengano modificati)

```bash
user@debian:~/poky$ git clone -b pyro git://git.openembedded.org/meta-openembedded
user@debian:~/poky$ git clone -b pyro https://github.com/jumpnow/meta-bbb
user@debian:~/poky$ git clone https://github.com/meta-qt5/meta-qt5.git
```

Creiamo un ambiente di compilazione in `/home/user/poky/bbb`

```bash
user@debian:~/poky$ source oe-init-build-env bbb
```

A questo punto editiamo `~/poky/bbb/conf/local.conf` :


```bash
# Local configuration for meta-bbb images
# Yocto Project 2.3 Poky distribution [pyro] branch
# This is a sysvinit system
LICENSE_FLAGS_WHITELIST = "commercial"
DISTRO_FEATURES = "ext2 usbhost ${DISTRO_FEATURES_LIBC}"

# remove opengl so opencv builds cleanly, remove pulseaudio since
# it brings in x11
DISTRO_FEATURES_BACKFILL_CONSIDERED += "pulseaudio opengl"
PREFERRED_PROVIDER_jpeg = "libjpeg-turbo"
PREFERRED_PROVIDER_udev = "eudev"
VIRTUAL_RUNTIME_init_manager = "sysvinit"

# To change kernel version
PREFERRED_VERSION_linux-stable = "4.9.%"
MACHINE = "beaglebone"
#DL_DIR = "${HOME}/oe-sources"

DISTRO = "poky"
PACKAGE_CLASSES = "package_ipk"

# i686 or x86_64
SDKMACHINE = "x86_64"
EXTRA_IMAGE_FEATURES = "debug-tweaks"
USER_CLASSES = "image-mklibs image-prelink"
PATCHRESOLVE = "noop"
RM_OLD_IMAGE = "1"
INHERIT += "rm_work"
CONF_VERSION = "1"
```

ed il file `~/poky/bbb/conf/bblayers.conf` :


```bash
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  ${HOME}/poky/meta \
  ${HOME}/poky/meta-poky \
  ${HOME}/poky/meta-openembedded/meta-oe \
  ${HOME}/poky/meta-openembedded/meta-networking \
  ${HOME}/poky/meta-openembedded/meta-python \
  ${HOME}/poky/meta-qt5 \
  ${HOME}/poky/meta-bbb \
"
```

A questo punto si è pronti per lanciare la compilazione del primo sistema GNU/Linux embedded, usando bitbake:

```bash
user@debian:~/poky/bbb$ bitbake core-image-minimal
```

Lanciando il comando `bitbake` vengono pianificati ed eseguiti tutti i passaggi necessari alla realizzazione di un sistema minimale. Spesso le azioni da compiere sono ripetitive, infatti si tratta di scaricare i sorgenti, patcharli, configurarli con CMake o gli autotools, cross compilare i programmi, installarli nello staging environment ed infine creare l'immagine del rootfs. Per evitare di doverlo fare a mano (se non sapete cosa fare nel tempo libero, potete seguire il progetto [Linux From Scratch](http://www.linuxfromscratch.org/), ma questo progetto esula dallo scopo di questi post) è stato sviluppato il programma [BitBake](https://www.yoctoproject.org/docs/2.3/bitbake-user-manual/bitbake-user-manual.html) che automatizza tutti questi passaggi.

La durata della prima compilazione è in genere molto lunga e dipende dalle risorse del computer e dalla qualità delle connessione ad internet. Ciò nonostante, al termine del processo, dentro alla cartella `~/poky/bbb/tmp/deploy/images/beaglebone/` saranno disponibili svariati file:


```bash
user@debian:~/poky/bbb$ ls tmp/deploy/images/beaglebone/
core-image-minimal-beaglebone-20170622131019.rootfs.manifest
core-image-minimal-beaglebone-20170622131019.rootfs.tar.xz
core-image-minimal-beaglebone-20170622131019.testdata.json
core-image-minimal-beaglebone.manifest
core-image-minimal-beaglebone.tar.xz
core-image-minimal-beaglebone.testdata.json
MLO
MLO-beaglebone
MLO-beaglebone-2017.05-r1
modules--4.9.33-r30-beaglebone-20170622131019.tgz
modules-beaglebone.tgz
u-boot-beaglebone-2017.05-r1.img
u-boot-beaglebone.img
u-boot.img
zImage
zImage--4.9.33-r30-am335x-boneblack-20170622131019.dtb
zImage--4.9.33-r30-am335x-bonegreen-20170622131019.dtb
zImage--4.9.33-r30-am335x-bonegreen-wireless-20170622131019.dtb
zImage--4.9.33-r30-bbb-4dcape43t-20170622131019.dtb
zImage--4.9.33-r30-bbb-4dcape70t-20170622131019.dtb
zImage--4.9.33-r30-bbb-hdmi-20170622131019.dtb
zImage--4.9.33-r30-bbb-nh5cape-20170622131019.dtb
zImage--4.9.33-r30-bbb-nhd7cape-20170622131019.dtb
zImage--4.9.33-r30-bbb-nohdmi-20170622131019.dtb
zImage--4.9.33-r30-beaglebone-20170622131019.bin
zImage-am335x-boneblack.dtb
zImage-am335x-bonegreen.dtb
zImage-am335x-bonegreen-wireless.dtb
zImage-bbb-4dcape43t.dtb
zImage-bbb-4dcape70t.dtb
zImage-bbb-hdmi.dtb
zImage-bbb-nh5cape.dtb
zImage-bbb-nhd7cape.dtb
zImage-bbb-nohdmi.dtb
zImage-beaglebone.bin
user@debian:~/poky/bbb$ 
```

Molti di qeusti file sono link simbolici, per avviare il sistema nella beaglebone black saranno necessari i seguenti file:
* MLO
* u-boot.img
* zImage
* zImage-bbb-nohdmi.dtb
* core-image-minimal-beaglebone.tar.xz

A cosa servono e come usare questi file per popolare una microSD sarà l'aromento del prossimo post.

### Indici di revisione dei repository usati in questo esempio

|Repository|Nome branch|Commit ID|
| :- | :- | :- |
|poky|pyro|9074fb46bc2a6da92334df068dbcc8cf8efbb6dc|
|meta-bbb|pyro|b84b65a7adb76f9871731286db26a525ac8ae378|
|meta-qt5|master|3ae86cb32edd1449f702e0a094929ae9b21ce191|
|meta-openembedded|pyro|5e82995148a2844c6f483ae5ddd1438d87ea9fb7|
