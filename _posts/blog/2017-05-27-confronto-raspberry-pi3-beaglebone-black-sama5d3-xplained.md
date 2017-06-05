---
layout: post
title: "Raspberry Pi3, Beagle Bone Black o SamA5D3 Xplained?"
modified:
categories: blog
share: true
comments: true
excerpt: Valutazione su quale scheda basare il libro.
---

Nell'ottica di scrivere il libro devo aggiornare i miei appunti, perché buona parte sono pensati per la scheda [Small Form Factor Software Defined Radio di Lyrtech](http://www.ti.com/tool/TMDSSFFSDR), una scheda costosa ma soprattutto obsoleta e difficilmente acquisibile.

Nell'ottica di un libro didattico, conviene orientarsi verso una scheda pensata per i makers dal costo contenuto. Per questo motivo le schede possibili sono:
* [Raspberry PI3 Model B](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/)
* [BeagleBone Black](https://beagleboard.org/black)
* [BeagleBone Black Wireless](https://beagleboard.org/black-wireless)
* [BeagleBone Blue](https://beagleboard.org/blue)
* [SamA5D3 Xplained](http://www.atmel.com/tools/ATSAMA5D3-XPLD.aspx)

Le prime due schede hanno una caratteristica che è molto importante nello sviluppo di sistemi embedded: la presenza di una scheda di rete cablata. Questo permette di eseguire una sistema linux montando il sistema embedded via NFS e di risparmiare il tempo di dover riprogrammare la memoria MicroSD ogni volta che si vuole riportare il sistema ad uno stato noto. Nello sviluppo dei sistemi embedded accade molto frequentemente, soprattutto nelle prime fasi dei progetti. La scheda Atmel ha un supporto Linux molto interessante, ma schede di espansione è compatibile con gli shield di Arduino R3, ha meno risorse computazionali degli altri ed è quindi più simile alle applicazioni embedded vere e proprie, ma costa di più e temo che possa di più difficile approvvigionamento.

| | Raspberry Pi3 Model B | BeagleBone Black | SamA5D3 Xplained |
| :- | :-: | :-: | :-: |
| **Processore** | Cortex A-53 64bit quad-core @ 1.2GHz | Cortex A-8 32bit single-core @ 1GHz | Cortex A-5 32bit single core @ 536 MHz |
| | | 2 x 32bit PRU Microcontrollers @ 200 MHz | |
| **Memoria RAM** | 1 GB | 512 MB | 256 MB|
| **Ethernet** | 10/100 Mbit/s | 10/100 Mbit/s | 1 x 10/100/1000 Mbit/s|
| | | | 1 x 10/100 Mbit/s |
| **Wifi** | 802.11n | ✖️ | ✖️ |
| **Bluetooth** | 4.1 e Bluetooth Low Energy | ✖️ | 
| **Porte USB** | 4 | 2 | 3 |
| **Pin GPIO** | 40 | 2x46 | 78 |
| **Output HDMI** | ✔️ | ✔️ | ✖️ |
| **Output audio** | Jack audio stereo 3.5mm | ✖️ | ✖️ |
| **Slot MicroSD** | ✔️ | ✔️ | ✖️ |
| **Slot SD** | ✖️ | ✖️ | ✔️ |
| **eMMC** | ✖️ | 4GB | ✖️ |
| **NAND Flash** | ✖️ | ✖️ | 256 MB |
| **Link** | <a target="_blank" href="https://www.amazon.it/gp/product/B01CD5VC92/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=B01CD5VC92&linkCode=as2&tag=ocampana07-21&linkId=3ebb44bd333da3430f0b16bbd6ab3f0d">Raspberry PI 3 Model B su Amazon</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=B01CD5VC92" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> | <a target="_blank" href="https://www.amazon.it/gp/product/B00KM6YTN6/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=B00KM6YTN6&linkCode=as2&tag=ocampana07-21&linkId=bde101a2757bf59bd1b3138a39a2a598">BeagleBone Black su Amazon</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=B00KM6YTN6" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> | <a target="_blank" href="https://www.amazon.it/gp/product/B00KG1ZXPA/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=B00KG1ZXPA&linkCode=as2&tag=ocampana07-21&linkId=490a236862b74ff47c23420c91dbae45">ATSAMA5D3-XPLD su Amazon</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=B00KG1ZXPA" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> |


A favore della scheda Raspberry Pi3 Model B giocano sicuramente il processore molto potente, l'abbondanza di RAM e di interfacce di connettività.

La BeagleBone Black costa un po' di più, ha meno RAM ed ha un processore meno potente. Tuttavia include a bordo una eMMC, che libera le persone dall'uso della MicroSD ed include due microcontrollori a 200MHz che possono essere usati per implementare un sistema di Asymmetric Multi Processing. Ha inoltre un altro vantaggio interessante, perché è alimentabile con un connettore 5V e non solo mediante la porta microUSB, che è la causa più frequente di malfunzionamenti dei miei progetti hobbistici.

Il pinout della Raspberry Pi3 Model B è il seguente

{: style="text-align: center;"}
![Raspberry Pi3 Model B pinout](https://openclipart.org/download/264607/gpiopinsv2.svg)

La scheda BeagleBone Black porta la flessibilità dei pin all'estremo. Infatti il processore AM3358 di Texas Instruments offre fino ad 8 modalità di funzionamento diverse per i pin dei connettori P8 e P9. 


La configurazione predefinita dei pin è questa

{: style="text-align: center;"}
![Beaglebone Black default pinout](https://beagleboard.org/static/images/cape-headers.png)

Le combinazioni complete sono troppe per poter essere riportate in una unica immagine, però sono riassunte in un [foglio di calcolo](/assets/files/BBB_Pins.xlsx).

C'è un dettaglio che rende la scheda non trascurabile: a differenza delle altre, esiste la [versione industriale](https://www.element14.com/community/docs/DOC-78671), con componenti che vanno da -40°C a +85°C. La versione a range di temperatura esteso, unita ad una forma compatta ed ai connettori che rendono semplice l'ancoraggio della scheda, anche munita di espansione, ad una struttura, può già essere usata in qualche progetto industriale reale. Questo non è possibile con la Raspberry, che è molto più debole meccanicamente, ed è un po' tirato anche per la scheda SamA5D3 Xplained.

A favore della scheda Atmel SamA5D3 Xplained ci sono di sicuro le due schede di rete, che permettono sviluppare dispositivi di routing o sniffing. Anche nel caso di questa scheda, alcuni pin possono essere mappati su delle funzioni alternative, come spiegato nella [user guide](/assets/files/Atmel-11269-32-bit-Cortex-A5-Microcontroller-SAMA5D3-Xplained_User-Guide.pdf).

Per mantenere la scheda compatibile con gli shield arduino, solo i pin dei connettori J19, J20 e J21 possono essere riassegnati:

{: style="text-align: center; width:100%"}
![SamA5D3 connettore J19](/images/sama5d3xplained_j19.png)

{: style="text-align: center; width:100%"}
![SamA5D3 connettore J20](/images/sama5d3xplained_j20.png)

{: style="text-align: center; width:100%"}
![SamA5D3 connettore J21](/images/sama5d3xplained_j21.png)

Tenendo conto di costo e flessibilità, sarei tentato di basare tutti gli esempi sulla beaglebone. Vedo nella Raspberry Pi3 una scheda che ricorda più un PC che un sistema embedded, e non a caso esistono porting di distribuzioni desktop per questo hardware. La scheda SamA5D3 è probabilmente la scheda che meglio si avvicina ai vincoli del mondo embedded con cui combattono i softwaristi nel mondo reale, ma è più difficile da approvvigionare e costa sensibilmente di più e questo potrebbe essere un grosso problema per chi volesse provare a replicare i futuri esempi.

E voi cosa ne pensate? Mi farebbe piacere avere raccogliere l'opinione anche di altre persone.
