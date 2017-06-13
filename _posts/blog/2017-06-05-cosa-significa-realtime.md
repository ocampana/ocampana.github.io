---
layout: post
title: "Cosa significa realtime?"
modified:
categories: blog
excerpt: si può realizzare un sistema realtime in GNU/Linux embedded?
tags: linux embedded realtime
keywords: linux embedded sistema realtime
modified: 2017-06-06T00:00:00+02:00
---

Ci sono argomenti nella programmazione dei sistemi embedded che sono ricorrenti. Uno di questi argomenti che a volte ritornano sono i sistemi operativi realtime e la discussione se sono implementabili su GNU/Linux embedded o meno.

Poiché mi hanno chiesto di spiegare alcuni concetti, colgo l'occasione per scrivere un post con alcune nozioni elementari.

Anzitutto, iniziamo con la definizione di Sistema Realtime. Un sistema in tempo reale è un sistema soggetto ad un vincolo temporale di esecuzione di un algoritmo. E' quindi un esempio di sistema realtime il circuito di controllo delle sospensioni attive di un'auto, che deve garantire di avere un anello di retroazione con una banda passante sufficiente ampia per permettere all'auto di adattarsi al fondo stradale sconnesso. Non può essere definito sistema realtime un algoritmo che deve ricevere dalla rete Ethernet dei dati ed applicare un controllo. Questo perché la Ethernet e IP sono protocollo _best effort_ che non danno alcuna garanzia al tempo necessario alla trasmissione e ricezione dei dati. Se si vuole una rete deterministica è necessario adottare altri standard di rete, quali per esempio [Ethercat](https://en.wikipedia.org/wiki/EtherCAT), che usa lo strato fisico a MAC di Ethernet, ma non prevede l'uso di IP.

Quando si parla di sistemi realtime ci si imbatte spesso nella seguente distinzione:

* I sistemi **Hard Realtime** sono quei sistemi in cui il sistema ha un output distruttivo se anche una sola iterazione non viene eseguita rispettando il vincolo temporale del sistema.

* I sistemi **Soft Realtime** sono sistemi in cui la qualità dell'output degrada in caso di violazione (anche più violazioni) del vincolo temporale ed un output degradato è ancora accettabile.

Un esempio di sistema Hard realtime è un sistema di riconoscimento dei pedoni sulla strada in un veicolo a guida autonoma, perché un ritardo nel riconoscimento può causare un incidente. Invece la parte del software di navigazione che si occupa di ricevere i dati dal GPS e verificare la posizione dell'auto lungo la rotta precalcolata è un sistema soft realtime. Né Android né IOS sono sistemi operativi realtime, però riusciamo a farci guidare facilmente dalle applicazioni di supporto alla guida.

Perché non ci si concentra solo sull'Hard Realtime? Perché questi sistemi hanno software sviluppato ad hoc per lo scenario di utilizzo ed in generale non possono essere implementati su qualsiasi tipo di processore. Quando viene eseguita l'iterazione di controllo del processo è infatti necessario mascherare tutti gli interrupt, mettendo di fatto in pausa il sistema operativo. Questo risulta abbastanza semplice su microprocessori elementari come il Motorola 68000, mentre su processori per uso desktop come per esempio l'architettura x86\_64 multicore la cosa è molto più complessa, perché il controller degli interrupt è diviso in più parti Local Advanced Programmable Interrupt Controller (LAPIC) ed I/O Advanced Programmable Interrupt Controller (IOAPIC). Forse qualche lettore si ricorderà che in passato per far funzionare alcuni schede correttamente su Linux era necesario passare `noapic` e `nolapic` come riga di comando, perché i driver ancora non erano in grado di gestire correttamente la gestione degli interrupt in sistemi a multiprocessore.

Per questo motivo, in tutti quei sistemi in cui il mancato rispetto del vincolo temporale di esecuzione non porta a risultati catastrofici si preferisce utilizzare sistemi Soft Realtime.

Esistono vari modi per implementare un sistema soft Realtime in GNU/Linux, in base al tipo di vincolo d'esecuzione:
* nel caso di sistemi con processori multipli è possibile dedicare un processore ad un solo processo. Per fare questo si può usare il comando `taskset` che fa parte di `util-linux` che lega un processo ad un insieme particolare di processori e il parametro `isolcpus=<CPU_ID>` da passare al boot al kernel, affinché su uno o più processori non vengano eseguiti processi, a meno che non siano esplicitamente assegnati con il comando `taskset`.
* usare una adeguata priorità e lo scheduler `SCHED_FIFO`, affinché un processo sia eseguito fino all'invocazione di un'operazione I/O bloccante, invochi esplicitamente `sched_yield` o venga messo in pausa dal sistema operativo per via di un processo con priorità maggiore. Quando la richiesta bloccante può essere evasa, il processo viene inserito in coda alla lista dei processi con la sua priorità e successivamente schedulato per l'esecuzione.
* usare una adeguata priorità e lo scheduler `SCHED_RR`. In questo caso il processo si blocca quando arriva ad un richiesta di I/O bloccante o quando finisce il suo intervallo di esecuzione.
* usare lo scheduler `SCHED_DEADLINE`, ma questo solo a patto di avere un kernel di versione >= 3.14. `SCHED_DEADLINE` è l'implementazione dello scheduler [Earliest Deadline First (EDF)](https://en.wikipedia.org/wiki/Earliest_deadline_first_scheduling) sul cui funzionamento scriverò un post dedicato in futuro, perché lo reputo la soluzione migliore per la maggior parte delle applicazioni realtime di GNU/Linux embedded.

Merita infine una citazione il progetto [RealTime Application Interface for Linux (RTAI)](https://www.rtai.org). RTAI è un progetto sviluppato dal Politecnico di Milano per realizzare sistemi Hard Realtime in GNU/Linux e permette di modificare la gestione degli interrupt per ottenere l'integrazione tra hardware e software citata nella descrizione iniziale dei sistemi Hard Realtime. Come si può vedere nell'homepage del progetto, supporta alcune architetture ma non tutti i processori supportati da Linux. Nel corso degli anni ho studiato con interesse RTAI, ma non ho mai avuto la necessità di utilizzarlo in un progetto reale, perché per quel poco di hard realtime che ho fatto, ho sempre demandato la parte realtime ad un microcontrollore. Se si utilizza RTAI è necessario porre particolare attenzione sui task implementati, perché un errore di programmazione può portare rapidamente ad un kernel panic ed al blocco del sistema e non c'è un processo di init (sysvinit, upstart, systemd o altro) che possa fare il respawn del processo.

### Alcuni esempi di C

Termino questo posto con qualche spezzone di codice C, utile per fissare qualche idea. Per prima cosa va fatto notare che è necessario usare funzioni diverse, in base all'uso dei thread o dei task. (Ricordiamoci che in Linux i thread sono mappati sui task, quindi tutti i discorsi fatti per i task si applicano ai thread, con l'unica differenza che per i task si invocano direttamente le syscall POSIX, mentre per i thread è necessario passare per la libpthread.)

| | task | thread |
| :- | :-: | :-: |
**Selezionare uno scheduler** | `sched_setscheduler` | `pthread_setschedparam` |
**Impostare la priorità di un processo** | `sched_setparam` | `pthread_attr_setschedpolicy` |
| | | `pthread_attr_setschedparam` |
**Leggere la priorità di un processo** | `sched_getparam` | `pthread_attr_getschedparam` |

Per avviare un thread con scheduler `SCHED_FIFO` a massima priorità è possibile usare questa funzione di appoggio:

```c
#include <pthread.h>

int start_thread_with_max_priority (pthread_t* thread,
                                    void* (*start_routine)(void*), void* arg)
{
  pthread_attr_t attr;
  struct sched_param param;

  pthread_attr_init (&attr);
  pthread_attr_setschedpolicy (&attr, SCHED_FIFO);
  param.sched_priority = sched_get_priority_max (SCHED_FIFO);
  pthread_attr_setschedparam (&attr, &param);

  return pthread_create (thread, &attr, start_routine, arg);
}
```

Come detto in precedenza, un task od un thread schedulato con `SCHED_FIFO` continua ad essere eseguito fino a quando non invoca una syscall bloccante o chiama esplicitamente `sched_yield`. Syscall bloccanti come `read` sono facili da implementare, invocare `sched_yield` pure, ma come si può implementare un ciclo che non fa uso di syscall bloccanti senza che consumi tutto il processore?

```c
#include <stdbool.h>
#include <string.h>
#include <time.h>

int
main (int argc, char *argv[])
{
  /* ... */

  while (true)
  {
    struct timespec req;

    do_something ();

    /* let's sleep for 1 ms */
    req.tv_sec = 0;
    req.tv_nsec = 1000000;
    while (nanosleep (&req, &req)) ;
  }

  /* ... */

  return 0;
}
```

La funzione `nanosleep` ha tre caratteristiche molto importanti:
* ha una risoluzione maggiore se confrontata con `sleep` ed `usleep` .
* le specifiche POSIX vietano di implementarla con segnali, quindi non c'è rischio di conflitti con `SIGALARM`, come per esempio con la funzione `sleep` .
* se il `nanosleep` termina correttamente ritorna 0, mentre se il processo viene risvegliato perché ha ricevuto un segnale, la funzione ritorna -1 e scrive nella variabile `req` quanto tempo rimane da dormire.

Quest'ultima caratteristica fa sì che il ciclo while termini quando il processo ha dormito per tutto il tempo richiesto. In caso di interruzione `nanosleep` aggiorna la variabile `req` con il tempo da rimane da dormire, la funzione ritorna -1, il ciclo while non esce perché la funzione non ha ritornato 0 ed il processo torna a dormire.
