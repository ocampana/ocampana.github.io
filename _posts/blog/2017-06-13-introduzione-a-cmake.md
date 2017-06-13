---
layout: post
title: "Introduzione a CMake"
modified:
categories: blog
excerpt: come organizzare i sorgenti di un progetto e avere un sistema di build robusto
tags: linux programmazione cmake
keywords: linux programmazione cmake programma libreria crosscompilare cross compilare
---

Come abbiamo visto nella [storia di Unix](/blog/storia-di-unix/), il proliferare di versioni diverse ha generato problemi di compatibilità tra i vari unix anche sulle stesse piattaforme.

Un esempio semplice è la funzione 

```c
#include <strings.h>

char *index (const char *s, int c);
```

Questa funzione è stata definita in 4.3BSD, è stata mantenuta in POSIX.1-2001 per retrocompatibilità ed è stata rimossa dalle specifiche POSIX.1-2008, rimpiazzata da

```c
#include <string.h>

char *strchr (const char *s, int c);
```

che fa parte dello standard C dalla revisione C89.

Sebbene molte implementazioni delle librerie C, quali per esempio la [GNU libc](https://www.gnu.org/software/libc/), siano molto ricche e mantengano le funzioni per garantire la massima retrocompatibilità, la presenza di queste funzioni rese obsolete non può essere data per certa. Per esempio la libreria [uClibc](https://www.uclibc.org/), spesso usata nei sistemi embedded, può essere compilata senza l'opzione `UCLIBC_SUSV3_LEGACY` ed in questo caso la funzione `index` non è disponibile e la compilazione del programma è destinata a fallire.

Allo stesso modo può accadere che cambiando sistema operativo gli strumenti di compilazione siano diversi: non si può dare per garantito che il compilatore sia sempre il gcc e gli strumenti di generazione delle librerie possono essere anche molto diversi tra loro.

La cosa si complica ancora di più nel caso di cross compilazione, operazione tipica delle attività di sviluppo di prodotti embedded: si sviluppa su uno unix con delle sue librerie, compilando per un dispositivo target che molto probabilmente ha delle librerie e compilatori diversi, all'interno di quello che viene chiamato uno *staging environment*, dove sono salvati tutti gli header file e le librerie necessarie alla compilazione del software per la scheda target.

Come si può fare per gestire tutte le possibili combinazioni? La risposta è ricorrendo ad uno strumento in grado di gestire il processo di compilazione del software che non abbia dipendenza dalla toolchain di compilazione.

Esistono vari software di questo tipo:
* Gli [Autotools](https://en.wikipedia.org/wiki/GNU_Build_System), conosciuti anche come lo Gnu Build System. Non è un unico programma, ma un insieme di strumenti tra cui `autoconf`, `automake` e `gettext` che permettono di adattare il codice sorgente all'ambiente di compilazione e di generare dei Makefile coerenti con il sistema di compilazione. &Egrave; lo strumento più diffuso per la generazione dei Makefile e per chi fosse interessato esiste un intero libro dedicato agli autotools, chiamato dagli smanettoni Autobook, che si trova sia su <a target="_blank" href="https://www.amazon.it/gp/product/1578701902/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=1578701902&linkCode=as2&tag=ocampana07-21&linkId=e8daf24101b94ec9f902aeab5028f35a">Amazon</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=1578701902" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" /> che [online](https://www.sourceware.org/autobook/autobook/autobook_toc.html).
* [Cmake](https://cmake.org), uno strumento più moderno, nato con lo scopo di semplificare la generazione dei Makefile. Anche per CMake esiste un libro, <a target="_blank" href="https://www.amazon.it/gp/product/1930934319/ref=as_li_tl?ie=UTF8&camp=3414&creative=21718&creativeASIN=1930934319&linkCode=as2&tag=ocampana07-21&linkId=f033ca90559ba681dc6eaf8004d2cb19">Mastering CMake</a><img src="//ir-it.amazon-adsystem.com/e/ir?t=ocampana07-21&l=am2&o=29&a=1930934319" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />.
* QMake, strumento integrato in Qt, simile a CMake.
* imake, citato per motivi storici, alla base del sistema di compilazione del Sistema a finestre X fino alla release X11R6.9 . Non conosco alcun software che ne faccia uso, ma per i curiosi la documentazione di imake è disponibile [qua](http://www.kitebird.com/imake-book/).

Poiché per sviluppare per embedded è necessario uno strumento di automazione del processo di compilazione, vediamo come sia possibile utilizzare CMake per realizzare un piccolo progetto.

Seguendo l'esempio iniziale con le funzioni `index` e `strchr`, vediamo come fare un programma helloworld in cui il binario invoca una funzione di una libreria dinamica che chiede il nome all'utente e lo saluta. Quando leggiamo il nome è presente il carattere di newline che può essere `\n`, `\r\n` o `\r` in base al sistema operativo e va rimosso.

I sorgenti sono così organizzati:

```
    .
    |
    +-- CMakeLists.txt
    +-- config.h.in
    +-- src
         |
         +-- CMakeLists.txt
         +-- libraries
         |    |
         |    +-- CMakeLists.txt
         |    +-- libhello
         |         |
         |         +-- CMakeLists.txt
         |         +-- hello.c
         |         +-- hello.h
         |
         +-- programs
              |
              +-- CMakeLists.txt
              +-- helloworld
                   |
                   +-- CMakeLists.txt
                   +-- main.c
```

Il file `./CMakeLists.txt` contiene:

```cmake
cmake_minimum_required (VERSION 2.6)
set (VERSION 0.1)
set (PACKAGE blog_intro_cmake)
project (${PACKAGE})

include (CheckIncludeFiles)
include (CheckFunctionExists)

check_include_files (strings.h HAVE_STRINGS_H)
check_include_files (string.h HAVE_STRING_H)
check_function_exists (index HAVE_INDEX)
check_function_exists (strchr HAVE_STRCHR)

configure_file(${CMAKE_SOURCE_DIR}/config.h.in ${CMAKE_SOURCE_DIR}/config.h)

add_subdirectory (src)

include (CPack)

```

In questo file oltre a definire il nome del pacchetto e la sua versione, vengono definiti i controlli per gli herder file `string.h` e `strings.h` e per le funzioni `index` e `strchr`. Nel file `./config.h.in` andiamo a definire le variabili di che cmake deve valorizzare, in base ai controlli definiti precedentemente:

```cmake
#cmakedefine HAVE_INDEX
#cmakedefine HAVE_STRCHR
#cmakedefine HAVE_STRINGS_H
#cmakedefine HAVE_STRING_H
```

I file `./src/CMakeLists.txt`, `./src/programs/CMakeLists.txt` e `./src/libraries/CMakeLists.txt` sono tutti molti simili, di fatto richiedono a cmake di scendere nello sottodirectory, quindi riporto solo il contenuto del primo:

```cmake
add_subdirectory (libraries)
add_subdirectory (programs)
```

La parte più interessante è il file per la generazione della libreria dinamica `libhello`. Iniziamo dal file `./src/libraries/libhello/CMakeLists.txt`:

```cmake
add_library (hello SHARED hello.c hello.h)

install(TARGETS hello DESTINATION lib)

file (GLOB HEADERS hello.h)
install(FILES ${HEADERS} DESTINATION include)

include_directories (${CMAKE_SOURCE_DIR})
```

Come si vede, in questo file viene richiesto di creare la libreria `libhello`, di tipo shared, ovvero dinamica e sono indicati i file sorgenti della libreria. Vengono inoltre specificati la directory di installazione e l'header file che deve essere incluso da tutti i programmi che vogliono usare la libreria. Per questo esempio non è necessario installare il file `hello.h`, tuttavia nel caso in cui avessimo più pacchetti software è necessario installare l'header file nello staging environment per poter effettuare la cross-compilazione del secondo pacchetto software.


Il file `./src/libraries/libhello/hello.h` è un semplice header file con la include barrier ed il prototipo della funzione:


```c
#ifndef _hello_h_
#define _hello_h_

void say_hello (void);

#endif /* _hello_h_ */
```

Nell file `./src/libraries/libhello/hello.c` avviene la gestione delle diverse funzioni:

```c
#include "hello.h"
#include "config.h"

#ifdef HAVE_STRINGS_H
#include <strings.h>
#endif

#ifdef HAVE_STRING_H
#include <string.h>
#endif

#include <stdio.h>

#define BUFSIZE 64

void
say_hello (void)
{
    char *c;
    char name[BUFSIZE];

    fprintf (stdout, "Inserisci il tuo nome: ");

    fgets (name, BUFSIZE, stdin);

    /*
     * dobbiamo rimuovere il newline finale in eccesso.
     * se vogliamo essere portabili dobbiamo gestire i casi
     * unix \n
     * windows \r\n
     * Mac OS <= 9 \r
     */

#if defined (HAVE_INDEX)
    c = index (name, '\n');
    if (c != NULL)
        *c = 0;

    c = index (name, '\r');
    if (c != NULL)
        *c = 0;
#elif defined (HAVE_STRCHR)
    c = strchr (name, '\n');
    if (c != NULL)
        *c = 0;

    c = strchr (name, '\r');
    if (c != NULL)
        *c = 0;
#else
#error "no function available"
#endif

    fprintf (stdout, "Ciao %s\n", name);
}
```

Per prima cosa si include il file `config.h` che contiene il risultato dei test di ricerca di funzioni ed header file, successivamente nella funzione `say_hello` vengono gestiti i tre casi:
* esiste la funzione index
* esiste la funzione strchr
* non esistono né la funzione index né la funzione strchr

Il terzo caso è improbabile, tuttavia quando si scrive software che gestisce piattaforme diverse è buona prassi fare in modo che la compilazione fallisca esplicitamente se l'ambiente di sviluppo non offre le funzioni richieste. In questo modo è possibile verificare a priori la criticità e non rischiare di dover gestire a posteriori un comportamento indefinito del software.

Giunto a questo punto è possibile scrivere il programma che fa uso della libreria appena sviluppata. La generazione del Makefile è fatta dal file `./src/programs/helloworld/CMakeLists.txt` :


```cmake
add_executable (helloworld main.c)

include_directories (../../libraries/libhello)
link_directories (${CMAKE_SOURCE_DIR}/src/libraries/libhello)

target_link_libraries (helloworld hello)

install (TARGETS helloworld RUNTIME DESTINATION bin)
```

Come si può vedere, viene definito il binario `helloworld` che a come sorgente il file `main.c` . Viene inoltre specificato dove trovare il file di intestazione hello.h e la librerie libello.


Per finire, il file`./src/programs/helloworld/main.c` è il più semplice di tutti:

```c
#include "hello.h"

int
main (int argc, char *argv[])
{
    say_hello ();

    return 0;
}
```

A questo punto è possibile verificare il corretto funzionamento di cmake e di tutti i suoi file di configurazione lanciando il seguente comando dalla radice dei sorgenti:

```bash
ottavio@debian:~/blog_intro_cmake$ cmake CMakeLists.txt 
-- The C compiler identification is GNU 4.9.2
-- The CXX compiler identification is GNU 4.9.2
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Looking for include file strings.h
-- Looking for include file strings.h - found
-- Looking for include file string.h
-- Looking for include file string.h - found
-- Looking for index
-- Looking for index - found
-- Looking for strchr
-- Looking for strchr - found
-- Configuring done
-- Generating done
-- Build files have been written to: /home/ottavio/blog_intro_cmake
ottavio@debian:~/blog_intro_cmake$ 
```

Come si può vedere la GNU libc è una libreria molto generosa e ricca di funzioni, per cui implementa sia `index` che `strchr`, ma come detto in precedente non è detto che questa condizione sia vera su altre combinazioni di software ed hardware.

Per compilare basta lanciare `make`, ma almeno le prime volte può essere più interessante eseguire `make VERBOSE=1`, per vedere tutti i comandi eseguiti automaticamente da cmake e che sono stati configurati in automatico nel passo precedente:

```bash
ottavio@debian:~/blog_intro_cmake$ make VERBOSE=1
/usr/bin/cmake -H/home/ottavio/blog_intro_cmake -B/home/ottavio/blog_intro_cmake --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/ottavio/blog_intro_cmake/CMakeFiles /home/ottavio/blog_intro_cmake/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: ingresso nella directory "/home/ottavio/blog_intro_cmake"
make -f src/libraries/libhello/CMakeFiles/hello.dir/build.make src/libraries/libhello/CMakeFiles/hello.dir/depend
make[2]: ingresso nella directory "/home/ottavio/blog_intro_cmake"
cd /home/ottavio/blog_intro_cmake && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/ottavio/blog_intro_cmake /home/ottavio/blog_intro_cmake/src/libraries/libhello /home/ottavio/blog_intro_cmake /home/ottavio/blog_intro_cmake/src/libraries/libhello /home/ottavio/blog_intro_cmake/src/libraries/libhello/CMakeFiles/hello.dir/DependInfo.cmake --color=
make[2]: uscita dalla directory "/home/ottavio/blog_intro_cmake"
make -f src/libraries/libhello/CMakeFiles/hello.dir/build.make src/libraries/libhello/CMakeFiles/hello.dir/build
make[2]: ingresso nella directory "/home/ottavio/blog_intro_cmake"
/usr/bin/cmake -E cmake_progress_report /home/ottavio/blog_intro_cmake/CMakeFiles 1
[ 50%] Building C object src/libraries/libhello/CMakeFiles/hello.dir/hello.c.o
cd /home/ottavio/blog_intro_cmake/src/libraries/libhello && /usr/bin/cc  -Dhello_EXPORTS -fPIC -I/home/ottavio/blog_intro_cmake    -o CMakeFiles/hello.dir/hello.c.o   -c /home/ottavio/blog_intro_cmake/src/libraries/libhello/hello.c
Linking C shared library libhello.so
cd /home/ottavio/blog_intro_cmake/src/libraries/libhello && /usr/bin/cmake -E cmake_link_script CMakeFiles/hello.dir/link.txt --verbose=1
/usr/bin/cc  -fPIC    -shared -Wl,-soname,libhello.so -o libhello.so CMakeFiles/hello.dir/hello.c.o 
make[2]: uscita dalla directory "/home/ottavio/blog_intro_cmake"
/usr/bin/cmake -E cmake_progress_report /home/ottavio/blog_intro_cmake/CMakeFiles  1
[ 50%] Built target hello
make -f src/programs/helloworld/CMakeFiles/helloworld.dir/build.make src/programs/helloworld/CMakeFiles/helloworld.dir/depend
make[2]: ingresso nella directory "/home/ottavio/blog_intro_cmake"
cd /home/ottavio/blog_intro_cmake && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/ottavio/blog_intro_cmake /home/ottavio/blog_intro_cmake/src/programs/helloworld /home/ottavio/blog_intro_cmake /home/ottavio/blog_intro_cmake/src/programs/helloworld /home/ottavio/blog_intro_cmake/src/programs/helloworld/CMakeFiles/helloworld.dir/DependInfo.cmake --color=
make[2]: uscita dalla directory "/home/ottavio/blog_intro_cmake"
make -f src/programs/helloworld/CMakeFiles/helloworld.dir/build.make src/programs/helloworld/CMakeFiles/helloworld.dir/build
make[2]: ingresso nella directory "/home/ottavio/blog_intro_cmake"
/usr/bin/cmake -E cmake_progress_report /home/ottavio/blog_intro_cmake/CMakeFiles 2
[100%] Building C object src/programs/helloworld/CMakeFiles/helloworld.dir/main.c.o
cd /home/ottavio/blog_intro_cmake/src/programs/helloworld && /usr/bin/cc   -I/home/ottavio/blog_intro_cmake/src/programs/helloworld/../../libraries/libhello    -o CMakeFiles/helloworld.dir/main.c.o   -c /home/ottavio/blog_intro_cmake/src/programs/helloworld/main.c
Linking C executable helloworld
cd /home/ottavio/blog_intro_cmake/src/programs/helloworld && /usr/bin/cmake -E cmake_link_script CMakeFiles/helloworld.dir/link.txt --verbose=1
/usr/bin/cc      CMakeFiles/helloworld.dir/main.c.o  -o helloworld -rdynamic ../../libraries/libhello/libhello.so -Wl,-rpath,/home/ottavio/blog_intro_cmake/src/libraries/libhello: 
make[2]: uscita dalla directory "/home/ottavio/blog_intro_cmake"
/usr/bin/cmake -E cmake_progress_report /home/ottavio/blog_intro_cmake/CMakeFiles  2
[100%] Built target helloworld
make[1]: uscita dalla directory "/home/ottavio/blog_intro_cmake"
/usr/bin/cmake -E cmake_progress_start /home/ottavio/blog_intro_cmake/CMakeFiles 0
ottavio@debian:~/blog_intro_cmake$ 
```

A questo punto, finalmente, possiamo apprezzare il frutto di tutto il lavoro svolto:

```bash
ottavio@debian:~/blog_intro_cmake$ src/programs/helloworld/helloworld 
Inserisci il tuo nome: Ottavio
Ciao Ottavio
ottavio@debian:~/blog_intro_cmake$ 
```

## Altre note

Come facciamo a verificare che il programma stia usando una libreria dinamica? Usando [ldd](https://linux.die.net/man/1/ldd)

```bash
ottavio@debian:~/blog_intro_cmake$ ldd src/programs/helloworld/helloworld 
	linux-vdso.so.1 (0x00007fff6e101000)
	libhello.so => /home/ottavio/blog_intro_cmake/src/libraries/libhello/libhello.so (0x00007ff68b691000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff68b2e6000)
	/lib64/ld-linux-x86-64.so.2 (0x00007ff68b892000)
ottavio@debian:~/blog_intro_cmake$ 
```

Come faccio a verificare che l'installazione dei programmi avvenga nelle directory corrette? Io di solito uso la variabile `CMAKE_INSTALL_PREFIX` per impostare come destinazione una directory temporanea. Questo è molto comodo, soprattutto quando ci sono programmi complessi, con alcuni binari compilati, altri sotto forma di script, librerie e un po' di file di configurazione.

```bash
ottavio@debian:~/blog_intro_cmake$ cmake -DCMAKE_INSTALL_PREFIX=/tmp/xxx CMakeLists.txt 
-- Configuring done
-- Generating done
-- Build files have been written to: /home/ottavio/blog_intro_cmake
ottavio@debian:~/blog_intro_cmake$ make install
[ 50%] Built target hello
[100%] Built target helloworld
Install the project...
-- Install configuration: ""
-- Installing: /tmp/xxx/lib/libhello.so
-- Installing: /tmp/xxx/include/hello.h
-- Installing: /tmp/xxx/bin/helloworld
-- Removed runtime path from "/tmp/xxx/bin/helloworld"
ottavio@debian:~/blog_intro_cmake$ 
```

Come si vede, al posto di `/` viene usato `/tmp/xxx/` e quindi si può verificare che il processo di installazione si corretto senza sporcare o danneggiare il sistema operativo. (Quando ero ancora studente ho avuto la _brillante_ idea di chiamare un programma `init`. Vi lascio immaginare cosa è successo quando ho dato `make install` senza riflettere...)

Tutto il codice di esempio è disponibile in [questo repository](https://bitbucket.org/ocampana/blog_intro_cmake/overview).
