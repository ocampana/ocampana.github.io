---
layout: post
title: "Installare Armbian sulla scheda nanopi neo 2"
modified:
categories: blog
excerpt: cosa è Armbian?
tags: linux embedded armbian
keywords: linux embedded armbian
---

Mi è stato richiesto di aiutare ad installare linux sulla scheda [NanoPi neo 2](http://www.friendlyarm.com/index.php?route=product/product&product_id=180) di cui non conoscevo l'esistenza. A dire il vero non conoscevo nemmeno il progetto [Armbian](https://www.armbian.com), così ho accettato di smanettarci su. Armbian è una distribuzione per schede embedded con processore ARM basata sui pacchetti di [Debian](https://www.debian.org) e di [Ubuntu](https://www.ubuntu.com).

Per iniziare è necessario scaricare [Etcher](https://ethcer.io), il programma consigliato da Armbian per flashare le schede microSD. Il passo successivo consiste nello scaricare da Armbian l'[immagine del sistema operativo desiderato](https://www.armbian.com/nanopi-neo-2/). Ad oggi Armbian offre per la scheda in questione due immagini per server, una basata su Debian ed una basata su Ubuntu.

Al primo login ai accede con root/1234 ed il sistema chiede di cambiare la password. L'IP è in DHCP.

Essendo una Debian, la prima cosa che vien voglia di fare è smanettare con `apt-get`, ma per avere un sistema debina funzionante è necessario lanciare il comando  `update-command-not-found` per generare i file do configurazione di apt. Successivamente, come da ordinaria amministrazione, è necssario lanciare `apt-get update` aggiornare l'elenco dei pacchetti.

Il punto però che mi ha stupito di più è stato il fatto che in questa scheda è possibile installare sia pacchetti con archiettura `arm64` che con architettura `armhf`. GNU/Debian supporta il [multiarch](https://wiki.debian.org/Multiarch), è abbastanza comune la presenza di librerie x86 su sistemi amd64, ma è la prima volta che mi sono imbattuto nel multiarch su piattaforma ARM.
