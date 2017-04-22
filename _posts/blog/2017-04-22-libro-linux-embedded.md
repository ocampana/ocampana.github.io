---
layout: post
title: "Un libro su GNU/Linux embedded"
modified:
categories: blog, linux
share: true
comments: true
excerpt: Il libro che non c'è
image:
  feature: Raspberry-Pi-3-top-down-web.jpg
  credit: Raspberry Pi Foundation
  creditlink: https://www.raspberrypi.org/
---

Per svariati motivi, lavorativi e non, sto valutando la possibilità di prendere tutte le mie vecchie presentazioni tenute ai vari LinuxDay, LinuxCafé e lezioni universitarie, aggiornare le parti vecchie e scrivere un libretto su GNU/Linux embedded.

L'idea nasce dal fatto che ci sono un sacco di programmatori software embedded, tutti con esperienza su microcontrollori, ma i programmatori esperti di GNU/Linux embedded sono rari. Sia con i colleghi che con persone che incontro nel tempo libero mi ritrovo relativamente spesso a discutere se ci sono dei libri su GNU/Linux embedded, per permettere ad un programmatore con esperienza di C e C++ sui microcontrollori di essere rapidamente operativo anche su sistemi a microprocessore con GNU/Linux. La mia risposta è che in generale un libro non c'è, di solito io suggerisco una mia lista di libri che compongono un adeguato bagaglio culturale, ma che non compongono un percorso formativo esaustivo:

* Kernigham, Ritchie, [Il linguaggio C. Principi di programmazione e manuale di riferimento](https://www.amazon.it/linguaggio-Principi-programmazione-manuale-riferimento/dp/887192200X), per chi non è familiare con il mio linguaggio di programmazione preferito.
* Bach, [The design of the Unix operating system](https://www.amazon.it/Design-Unix-Operating-System-Bach/dp/8120305167/), libro molto concreto sui sistemi operativi, purtroppo meno famoso di altri libri nonostante la qualità del contenuto.
* Stevens, [Unix Network Programming Vol 1](https://www.amazon.it/Unix-Network-Programming-Sockets-Networking/dp/0131411551/), libro fondamentale per la programmazione di rete. Di solito GNU/Linux viene usato proprio per interfacciare in rete dispositivi che prima erano isolati o connessi con seriali.
* Stevens, [Unix Network Programming Vol 2](https://www.amazon.it/UNIX-Network-Programming-Interprocess-Communications/dp/B01JXQNTX6/), questo è più orientato alle IPC, ovvero le Inter Process Communication. Probabilmente il libro meno importante di questa lista, bello per cultura personale.
* Stevens, [TCP/IP Illustrated, Vol 1](https://www.amazon.it/TCP-IP-Illustrated-Protocols-1/dp/0321336313), altro grande libro sul networking
* Purdie, [Yocto Project Reference Manual](http://www.yoctoproject.org/docs/latest/ref-manual/ref-manual.html) non so se esiste cartaceo. E' la bibbia di Yocto, uno degli strumenti più importanti per sviluppare sistemi embedded con GNU/Linux
* Corbet, Rubini, [Linux Device Drivers](https://www.amazon.it/Linux-Device-Drivers-Jonathan-Corbet/dp/0596005903), per chi deve sviluppare driver per periferiche o, se particolarmente "fortunato", fare un porting per una piattaforma.

La lista è lunga, i tomi sono tutti molto pesanti e danno una buona preparazione teorica ma ancora non sono sufficienti a rendere un programmatore operativo.

Esistono già diversi libri su Yocto, per esempio:
* Streif, [Embedded LinuxSystems With the Yocto Project](https://www.amazon.it/Embedded-Linux-Systems-Yocto-Project/dp/0133443248/)
* Salvador, Angolini, [Yocto for Embedded Linux Development Primer](Yocto for Embedded Linux Development Primer)

Tutti questi libri sono molto focalizzati sull'uso di Yocto, ma perdono di vista una serie di problemi concreti nello sviluppo di prodotti.


Per questo motivo avrei un mente il seguente sommario:

* Prima parte - organizzazione dello sviluppo su PC
  1. Uso di CMake per compilare e crosscompilare i progetti
  2. Come strutturare un progetto
* Seconda parte - portare il progetto sul target embedded
  3. Introduzione a Yocto
  4. Portare il progetto sul target e debug
* Parte terza - argomenti avanzati
  5. Realtime su GNU/Linux embedded
  6. Strategie di firmware upgrade

Un simile libro, per essere digerito rapidamente e non essere l'ennesimo mattone che allunga la lista iniziale, deve essere pensato in accoppiata con una scheda opensource, come per esempio la [BeagleBone](http://beagleboard.org/bone) o la [Raspberry](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/). Il costo dei dispositivi è simile, e per ciascuna scheda sono disponibili [capes](http://elinux.org/Beagleboard:BeagleBone_Capes) ed [hats](http://elinux.org/RPi_Expansion_Boards).

La mia idea consiste nello sviluppare un piccolo progetto basato su GNU/Linux embedded, sviluppando gli argomenti esposti nel sommario. Il maggior problema che vedo attualmente è la definizione di una applicazione che non sia in conflitto con il mio lavoro ma che allo stesso tempo sia sufficientemente interessante.

Per ora, l'unico scenario di progetto che potrei proporre è il seguente:

<div markdown="0">
    <div class="mermaid">
        sequenceDiagram
        PC->> Raspberry: REST API
        Raspberry->> Ricevitore Dali: Protocollo Dali
        Ricevitore Dali--> LED: Potenza
    </div>
</div>

Il problema più fastidioso è che il protocollo [Dali](https://en.wikipedia.org/wiki/Digital_Addressable_Lighting_Interface) si basa su una seriale differenziale optoisolata con tensioni più elevate della RS485 e non c'è nessun cape o hat che la implementi. Il che è un peccato, perché Dali è un protocollo semplice e che ben si presta alla didattica.

Mi piacerebbe ricevere qualche feedback, sia sulla lista degli argomenti che sullo scenario applicativo. Vi va di lasciare un commento?
