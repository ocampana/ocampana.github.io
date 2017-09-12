---
layout: post
title: "Come installare dotnet su GNU/Debian Stretch"
modified:
categories: blog
excerpt: Un piccolo esperimento, visot che tutto lo stack è stato portato su GNU/Linux
tags: linux c# dotnet development
keywords: linux c# dotnet development
---

Agli arbori di Linux, Microsoft era vista come un nemico. Dopo anni di lotta, Microsoft ha iniziato ad essere pià aperta verso altri sistemi operativi, in primis verso Linux.

Questa estate Microsoft ha rilasciato il [.Net Core 2.0](https://blogs.msdn.microsoft.com/dotnet/2017/08/14/announcing-net-core-2-0/). Il linguaggio mi ha sempre incuriosito, però non mi sono mai trovato bene con [Mono](http://www.mono-project.com/), il compilatore opensource per C# perché non supporta tutte le librerie di sistema di .Net. Con la nuova release di .Net e l'annuncio in grande stile del supporto della piattaforma Linux ho deciso di fare una prova.

Si inizia con l'installazione di un po' di pacchetti di supporto:

```bash
root@debian:~# apt-get update
root@debian:~# apt-get install curl libunwind8 gettext apt-transport-https
```

Successivamente è necessario aggiungere la chiave con cui Microsoft firma i propri pacchetti:

```bash
root@debian:~# curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg
```

Successivamente è necessario aggiunere l'archivio dei pacchetti per Debian Stretch:


```bash
root@debian:~# echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-debian-stretch-prod stretch main" > /etc/apt/sources.list.d/dotnetdev.list
```

A questo punto è possibile procedere con l'installazione dell'SDK .Net:

```bash
root@debian:~# apt-get update
root@debian:~# apt-get install dotnet-sdk-2.0.0
```

## Facciamo un hello world


A questo punto, per verificare che tutto funzioni, creiamo una applicazione i nconsole che saluti il mondo:

```bash
user@debian:~$ dotnet new console -o hello
user@debian:~$ cd hello
```

Come si può vedere il comando crea una seriel di file, tra cui `Program.cs` :

```c#
using System;

namespace hello
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Per compilare ed eseguire il programma basta lanciare il seguente comando:


```bash
user@debian:~/hello$ dotnet run
Hello world!
user@debian:~/hello$ 
```

Se si vuole solo eseguire il programma senza compilarlo è possibile usare il seguente comando:

```bash
user@debian:~/hello$ dotnet exec bin/Debug/netcoreapp2.0/hello.dll
Hello world!
user@debian:~/hello$ 
```

## E per i webservice?

Purtroppo nonostate il core di .Net sia stato portato su Linux, il framework .Net è ancora disponibile solo su Windows. Come riportato [qua](https://fizzylogic.nl/2016/07/19/using-wcf-in-combination-with-net-core-sdk/), esiste il pacchetto [WCF connected service](https://visualstudiogallery.msdn.microsoft.com/c3b3666e-a928-4136-9346-22e30c949c08) che permette di realizzare client di web service usando solo il core .Net . Purtroppo il generatore delle classi per accedere ai servizi funziona solo sotto windows, ma poi il codice è compilabile anche da Linux. Il vero problema è che WCF connected service supporta solo un sottoinsieme del suo equivalente sotto windows, ed è carente per tutte le estensioni WS-\*, il che vuol dire mancanza di supporto a meccanismi di autenticazione, WD-Addressing, e meccanismi di trasporto.

Peccato, speravo meglio. Senza libreria grafica e funzioni di rete standard, C# su Linux rimane secondo me ancora un giocattolo.
