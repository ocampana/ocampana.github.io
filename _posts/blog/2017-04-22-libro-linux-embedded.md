---
language: it
layout: post
title: "Un libro su GNU/Linux embedded"
modified:
categories: blog
share: true
comments: true
excerpt: Il libro che non c'è
keywords: libro libri linux embedded c networking sistemi operativi
image:
  feature: Raspberry-Pi-3-top-down-web.jpg
  credit: Raspberry Pi Foundation
  creditlink: https://www.raspberrypi.org/
---

Per svariati motivi, lavorativi e non, sto valutando la possibilità di prendere tutte le mie vecchie presentazioni tenute ai vari LinuxDay, LinuxCafé e lezioni universitarie, aggiornare le parti vecchie e scrivere un libretto su GNU/Linux embedded.

L'idea nasce dal fatto che ci sono un sacco di programmatori software embedded, tutti con esperienza su microcontrollori, ma i programmatori esperti di GNU/Linux embedded sono rari. Sia con i colleghi che con persone che incontro nel tempo libero mi ritrovo relativamente spesso a discutere se ci sono dei libri su GNU/Linux embedded, per permettere ad un programmatore con esperienza di C e C++ sui microcontrollori di essere rapidamente operativo anche su sistemi a microprocessore con GNU/Linux. La mia risposta è che in generale un libro non c'è, di solito io suggerisco una mia lista di libri che compongono un adeguato bagaglio culturale, ma che non compongono un percorso formativo esaustivo:

* Kernigham, Ritchie, <a target="_blank" href="https://www.amazon.it/gp/product/887192200X/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=887192200X&linkCode=as2&tag=ocampana07-21&linkId=1418fc6b112f98cfcaf6cb9929211d35">Il linguaggio C. Principi di programmazione e manuale di riferimento</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=887192200X" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, per chi non è familiare con il mio linguaggio di programmazione preferito.
* Bach, <a target="_blank" href="https://www.amazon.it/gp/product/B000M85BS6/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=B000M85BS6&linkCode=as2&tag=ocampana07-21&linkId=3551df8b8096e1fcd7223a33b793ce51">The Design of the UNIX Operating System</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=B000M85BS6" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, libro molto concreto sui sistemi operativi, purtroppo meno famoso di altri libri nonostante la qualità del contenuto.
* Stevens, <a target="_blank" href="https://www.amazon.it/gp/product/0131411551/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=0131411551&linkCode=as2&tag=ocampana07-21&linkId=456642eda049bb854eea85fa6a5cb813">Unix Network Programming Vol 1: The Sockets Networking Api</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=0131411551" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, libro fondamentale per la programmazione di rete. Di solito GNU/Linux viene usato proprio per interfacciare in rete dispositivi che prima erano isolati o connessi con serialindi è un must.
* Stevens, <a target="_blank" href="https://www.amazon.it/gp/product/0132974290/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=0132974290&linkCode=as2&tag=ocampana07-21&linkId=d7d68a3410f8401cc6c22465f8c61e78">Unix Network Programming Vol 2: Interprocess Communications</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=0132974290" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, questo è più orientato alle IPC, ovvero le Inter Process Communication. Probabilmente il libro meno importante di questa lista, bello per cultura personale.
* Stevens, [TCP/IP Illustrated, Vol 1](https://www.amazon.it/TCP-IP-Illustrated-Protocols-1/dp/0321336313/ref=as_li_ss_tl?_encoding=UTF8&qid=&sr=&linkCode=ll1&tag=ocampana07-21&linkId=65faf2a2fb8add9972373265a43ae356), altro grande libro sul networking
* Purdie, <a target="_blank" href="https://www.amazon.it/gp/product/9888381989/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=9888381989&linkCode=as2&tag=ocampana07-21&linkId=a0a62aa6485260ccc47bd656a3d2956b">Yocto Project Reference Manual</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=9888381989" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />. E' la bibbia di Yocto, uno degli strumenti più importanti per sviluppare sistemi embedded con GNU/Linux.
* Corbet, Rubini, <a target="_blank" href="https://www.amazon.it/gp/product/0596005903/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=0596005903&linkCode=as2&tag=ocampana07-21&linkId=314dced8dd42021d945ca6f11a3a4fbb">Linux Device Drivers</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=0596005903" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />, per chi deve sviluppare driver per periferiche o, se particolarmente "fortunato", fare un porting per una piattaforma.

La lista è lunga, i tomi sono tutti molto pesanti e danno una buona preparazione teorica ma ancora non sono sufficienti a rendere un programmatore operativo.

Esistono già diversi libri su Yocto, per esempio:
* Streif, <a target="_blank" href="https://www.amazon.it/gp/product/0133443248/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=0133443248&linkCode=as2&tag=ocampana07-21&linkId=a73951101b8d512532f7245c64dbf657">Embedded Linux Systems With the Yocto Project</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=0133443248" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> .
* Salvador, Angolini, <a target="_blank" href="https://www.amazon.it/gp/product/1783282339/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=1783282339&linkCode=as2&tag=ocampana07-21&linkId=0a7eccccd7743c368c0701aec84af029">Yocto for Embedded Linux Development Primer</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=1783282339" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> .

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
