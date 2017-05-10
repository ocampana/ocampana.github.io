---
layout: post
title: "Storia di Unix"
modified:
categories: blog
share: true
comments: true
excerpt: Una complessa storia di famiglia.
---

Di solito, quando leggiamo un albero genealogico, siamo di fronte ad una struttura molto simile ad un albero. In qualche rara occasione è possibile trovare un incrocio con consanguinei, ma in generale tali alberi sono strutture planari.

L'albero genealogico di Unix racconta invece una genesi complessa e frazionata.

Nella seconda metà degli anni sessanta, i Bell Labs decisero di contribuire allo sviluppo di Multics (**MULT**iplexed **I**nformation and **C**omputing **S**ervice), un sistema operativo in time-sharing basato su memoria virtuale. Il progetto pose le base per i successivi sistemi operativi, tuttavia il suo sviluppo fu lento. Per questo motivo nel 1969 i Bell Labs decisero di abbandonare il progetto.

All'inizio degli anni 70 i Bell Labs diedero vita ad Unix, un sistema operativo multitasking e multiutente. Poco dopo AT&T, l'azienda controllante i Bell Labs, decise di concedere in licenza il sistema operativo ad altre aziende ed università. Ciascun licenziatario estese il sistema operativo originale, dando vita a tante diverse versioni, simili, ma spesso incompatibili tra di loro.

Le differenze sono riassumibili in:
* differenti layout del filesystem
* differenti funzioni e librerie disponibili
* differenti piattaforme HW supportate

Nella seguente immagine è possibile vedere come l'albero genealogico di Unix sia contorto e complesso.

<div markdown="0">
    <div class="mermaid" id="unix_tree">
        graph TD
        id1(Multics)

          subgraph Unix
            id2(Unix 1 - 4)
            id3(Unix 5 - 6)
            id4(PWB/Unix)
            id6(Unix 7)
            id7(Unix/32V)
            id11(Unix 8)
            id13(Unix 9 - 10)
          end

          subgraph BSD
            id5(BSD 1.0 - 2.0)
            id8(BSD 3.0 - 4.1)
            id9(BSD 4.2)
            id14(BSD 4.3)
            id15(BSD 4.3 Tahoe)
            id16(BSD 4.3 Reno)
            id43(BSD 4.4 - 4.4 lite2)
          end

          subgraph Sun OS
            id10(Sun OS  1 - 1.1)
            id12(Sun OS 1.2 - 3.0)
            id17(Sun OS 4.0)
          end

            id18(Xenix 1.0 - 2.3)
            id19(Xenix 3.0)
            id20(Sco Xenix)
            id27(Sco Xenix V/286)
            id28(Sco Xenix V/386)

        id21(System III)

          subgraph System V
            id22(System V R1 - R2)
            id23(System V R3)
            id24(System V R4)
          end

          subgraph AIX
            id25(AIX 1.0)
            id26(AIX 3.0 - 7.1)
          end

        id29(Sco Unix 3.2.4)
        id30(OpenServer 5.0 - 5.0.4)
        id31(OpenServer 5.0.5 - 5.0.7)
        id32(OpenServer 6.0)

          subgraph Unixware
            id33(UnixWare 1.x - 2.x)
            id34(UnixWare 7.x)
          end

          subgraph Solaris
            id35(Solaris 2.1 - 9)
            id36(Solaris 10)
            id37(Solaris 11)
          end

        id38(OpenSolaris)

          subgraph HP-UX
            id39(HP-UX 1.0 - 1.2)
            id40(HP-UX 2.0 - 3.0)
            id41(HP-UX 6 - 11)
            id42(HP-UX 11i - 11iv3)
          end

        id44(BSD NET/2)
        id45(386BSD)

          subgraph FreeBSD
            id46(FreeBSD 1.0 - 2.2.x)
            id53(FreeBSD 3.0 - 3.2)
            id54(FreeBSD 3.2 - 4.8)
            id58(FreeBSD 4.9 - 11.x)
          end

          subgraph NetBSD
            id47(NetBSD 0.8 - 1.0)
            id48(NetBSD 1.1 - 1.2)
            id51(NetBSD 1.3)
            id52(NetBSD 1.4 - 7.x)
          end

          subgraph OpenBSD
            id49(OpenBSD 1.0 - 2.2)
            id50(OpenBSD 2.3 - 6.x)
          end

        id55(NextStep/OpenStep 1.0 - 4.0)

          subgraph Mac OS X
            id56(Mac OS X Server)
            id57(Mac OS X 10.0 - 10.12.x)
          end

        id59(DragonFly BSD 1.0 - 4.x)

        id1-->id2
        id2-->id3
        id2-->id4
        id3-->id5
        id3-->id6
        id5-->id8
        id6-->id7
        id6-->id11
        id8-->id9
        id8-->id10
        id8-->id11
        id9-->id14
        id10-->id12
        id11-->id13
        id9-->id12
        id14-->id13
        id14-->id15
        id15-->id16
        id15-->id17
        id16-->id43
        id12-->id17
        id6-->id18
        id18-->id19
        id19-->id20
        id4-->id21
        id7-->id21
        id21-->id19
        id21-->id22
        id22-->id23
        id23-->id24
        id22-->id25
        id9-->id25
        id24-->id26
        id25-->id26
        id20-->id27
        id27-->id28
        id28-->id29
        id29-->id30
        id30-->id31
        id31-->id32
        id24-->id33
        id33-->id34
        id31-->id34
        id34-->id32
        id24-->id35
        id35-->id36
        id36-->id37
        id36-->id38
        id21-->id39
        id39-->id40
        id23-->id40
        id40-->id41
        id24-->id41
        id41-->id42
        id16-->id44
        id44-->id45
        id45-->id46
        id44-->id47
        id45-->id47
        id47-->id48
        id47-->id49
        id49-->id50
        id43-->id50
        id43-->id51
        id48-->id51
        id51-->id52
        id46-->id53
        id43-->id53
        id53-->id54
        id54-->id58
        id14-->id55
        id55-->id56
        id43-->id56
        id51-->id56
        id56-->id57
        id53-->id57
        id54-->id59

        id13-->id21
        id13-->id10
	id43-->id44
        id58-->id59
	id52-->id49
	id50-->id56 
	id43-->id10
	id17-->id24
	id58-->id47
	id11-->id5
        linkStyle 77 stroke-width:0px;
        linkStyle 78 stroke-width:0px;
        linkStyle 79 stroke-width:0px;
        linkStyle 80 stroke-width:0px;
        linkStyle 81 stroke-width:0px;
        linkStyle 82 stroke-width:0px;
        linkStyle 83 stroke-width:0px;
        linkStyle 84 stroke-width:0px;
        linkStyle 85 stroke-width:0px;

    </div>
</div>

In un mondo caratterizzato da una frammentazione così spinta e dalla fusione delle funzionalità tra release diverse, ciascuna azienda proponeva una propria versione di Unix, simile ma mai completamente compatibile con le offerte di altre software house. Le vittime di questa frammentazione erano gli sviluppatori e gli utenti finali, che non erano in grado di fornire e/o fruire degli stessi strumenti su tutte le piattaforme. Il grado di frustrazione dovuto a questa frammentazione portò gli utenti di Unix a coniare il nome *Unix Wars* per descrivere la situazione.

Al fine di superare questa impasse vennero lanciate diverse iniziative:
* Nel 1984 venne creato il consorzio X/Open
* Nel 1985 AT&T lanciò le specifiche System V Interface Definition (SVID)
* Nel 1988 IEEE rilasciò le specifiche Posix. Queste specifiche divennero rapidamente obbligatorie per i bandi dell'amministrazione pubblica americana e di conseguenza ebbero una ampia diffusione nel giro di pochi mesi.

Grazie all'adozione delle specifiche Posix il porting del software tra Unix System V e BSD divenne molto più semplice.

Ci si potrebbe chiedere perché Linux non è nell'albero genealogico. La risposta sta nel significato del nome stesso di Linux: **L**inux **I**s **N**ot **U**ni**X**. In una [intervista](https://www.abc.se/~m9339/linux/linuxdoc/linus.html), Linus Torvalds stesso afferma che se avesse potuto avere accesso a 386BSD prima di iniziare a sviluppare Linux, probabilmente non avrebbe sviluppato il suo sistema operativo:

> M: What is your opinion of 386BSD?
> 
> L: Actually, I have never even checked 386BSD out; when I started on Linux it wasn't available (although Bill Jolitz' series on it in Dr. Dobbs' Journal had started and were interesting), and when 386BSD finally came out, Linux was already in a state where it was so usable that I never really thought about switching. If 386BSD had been available when I started on Linux, Linux would probably never had happened.

Allo stesso modo, anche Minix è basato su dei sorgenti indipendenti dalla stria di Unix. Minix è un sistema operativo sviluppato dal professor Andrew Tanenbaum, per scopi didattici.

Sia Linux che Minix sono compatibili con le specifiche POSIX, anche se non sono ufficialmente certificati.

Come vedremo in un post futuro, questo ecosistema ricco di sistemi operativi simili ma non identici che vengono eseguiti su architetture hardware anche molto diverse tra di loro ha generato la necessità di strumenti in grado di riconoscere su quale combinazione di hardware e software si sta cercando di compilare il programma e di adattare dinamicamente il codice per superare queste diversità. Come vedremo, questi strumenti di portabilità giocano un ruolo fondamentale non solo nello sviluppo di applicazioni per computer, ma anche per lo sviluppo di sistemi embedded.
