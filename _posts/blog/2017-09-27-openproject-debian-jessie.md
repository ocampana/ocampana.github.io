---
layout: post
title: "Come installare OpenProject su GNU/Debian Jessie"
modified: 2017-10-05T00:00:00+02:00
categories: blog
excerpt: una alternativa opensource a Jira
tags: linux development
keywords: linux development
---

Sono un grande fan della suite [Altassian](https://www.atlassian.com/), soprattutto [Jira](https://www.atlassian.com/software/jira). Uso per alcuni miei progetti personali [Bitbucket](https://bitbucket.org/product), ma talvolta per altri progetti mi capita di aver bisogno di una soluzione simile alla piattaforma di Atlassian, ad un costo minore.

Dopo aver fatto diverse prove ho scoperto [OpenProject](https://www.openproject.org), che è meno stiloso di Jira ma è comunque molto completo. L'ho installato su un server baremetal di [Scaleway](https://www.scaleway.com/), riuscendo ad avere una piattaforma che posso condividere con i miei compagni di eMBA al prezzo di 3€/mese.

Le istruzioni online per installare OpenProject sono un po' sgangherate, quindi ho deciso di riassumere qui tutti i passaggi, nel caso in cui altre persone fossero interessate.

**Una premessa**: OpenProject è un programma opensource, che viene distribuito in pacchetti per chi lo vuole eseguire su un proprio server, altrimenti viene venduto __as a service__. I pacchetti per Debian sono disponibili ad oggi fino a Jessie, mentre per Stretch il software non è disponibile come già pacchettizzato e non è nemmeno agevole installarlo a mano, perché MySQL è diventato MariaDB e ci sono alcuni problemi con le versioni più aggiornate di Ruby. Ecco perché sul server di scaleway ho installato Debian Jessie e non Stretch. Per fortuna Debian fornisce gli aggiornamenti di sicurezza per oldstable, quindi non è un problema non avere l'ultima versione.

Iniziamo con l'importare la firma digitale del repository dei pacchetti di OpenProject per Debian Jessie:

```bash
root@debian:~# wget -qO- https://dl.packager.io/srv/opf/openproject-ce/key | apt-key add -
```

Per proseguire è necessario installare apt-transport-https, perché packager.io, il sito mediante il quale vengono distribuiti i pacchetti, fornisce il software solo mediante https e non http.

```bash
root@debian:~# apt-get install apt-transport-https
```

A questo punto è necessario aggiungere i repository per [Let's encrypt](https://letsencrypt.org/) ed OpenProject:

```bash
root@debian:~#  echo 'deb http://ftp.debian.org/debian jessie-backports main' | \
    tee /etc/apt/sources.list.d/backports.list

root@debian:~# wget -O /etc/apt/sources.list.d/openproject-ce.list \
    https://dl.packager.io/srv/opf/openproject-ce/stable/7/installer/debian/8.repo
```

Installiamo certbot per la generazione dei certificati

```bash
root@debian:~# apt-get update
root@debian:~# apt-get install python-certbot-apache -t jessie-backports
root@debian:~# certbot --apache -d nome.sito.it
```

A questo punto, se la procedura è andata a buon fine, dopo aver risposto ad alcune semplici domande, dovreste vedere questi file:

```bash
root@debian:~# ls /etc/letsencrypt/archive/nome.sito.it/
cert1.pem  chain1.pem  fullchain1.pem  privkey1.pem
root@debian:~#
```

I certificati di Let's Encrypt hanno una durata limitata di 90 giorni e devono essere rinnovati. I motivi di questa scelta sono spiegati [qui](https://letsencrypt.org/2015/11/09/why-90-days.html) e di fatto viene fatto perché si vuole fare in modo che se un certificato viene compromesso l'impatto sia limitato. Per non dover aggiornare a mano i certificati basta mettere una entry nella crontab:

```bash
root@debian:~# crontab -e
# m h  dom mon dow   command
* 7,19 * * * certbot -q renew
```

e dimenticarsi del rinnovo dei certificati. Fatto questo è il momento di installare OpenProject:

```bash
root@debian:~# apt-get install openproject
```

Questo comando provvederà ad installare tutte le dipendenze. Per la creazione del sito bisogna lanciare il seguente comando e prepararsi a rispondere ad una serie di domande:


```bash
root@debian:~# openproject configure
```

Il programma di configurazione è molto ricco, io di solito scelgo di:
* installare mysql
* installare apache2, che chiede il fully qualified domain name (FQDN) del sito, in questo caso _nome.sito.it_ , non specifico nessun server path prefix perché ho un server ad hoc per usare OpenProject ed abilito ssl
* come file del certificato ssl imposto `/etc/letsencrypt/archive/nome.sito.it/cert1.pem`
* come chiave del certificato imposto `/etc/letsencrypt/archive/nome.sito.it/privkey1.pem` come chiave ,
* non aggiungo alcuni CA bundle, perché in Debian c'è già il file `/etc/ssl/certs/ISRG_Root_X1.pem` , che è il certificato di Internet Security Research Group, ovvero la fondazione a capo di Let's Encrypt
* installare svn
* installare git
* installare sendmail
* installare memcached


A questo punto potrebbe essere rimasto attivo un disto di default di apache2, quindi per sicurezza basta lanciare questi due comandi

```bash
root@debian:~# a2dissite 000-default-le-ssl.conf
root@debian:~# /etc/init.d/apache2 restart
```

A questo punto, per iniziare ad usare il programma, basta aprire un browser e visitare il sito https://nome.sito.it . Il primo accesso avverrà come admin/admin, ma verrà immediatamente richiesto di impostare un'altra password.

## Controlliamo di non aver lasciato schifezze in giro

Il sito funziona, ma verifichiamo di non aver dimenticato in giro programmi indesiderati

```bash
root@debian:~# apt-get install nmap
```

Cosa possiamo esserci dimenticati?

```bash
root@debian:~# namp <ip pubblico>
Starting Nmap 6.47 ( http://nmap.org ) at 2017-09-27 14:36 CEST
Nmap scan report for aa.bb.cc.dd
Host is up (0.000011s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
80/tcp  open  http
111/tcp open  rpcbind
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 2.57 seconds
root@debian:~# 
```

La porta 22 aperta va bene, è per ssh, altrimenti come facciamo manutenzione?

La porta 25 aperta è di postfix, e non va assolutamente bene. &Egrave; vero che nel file `/etc/postfix/main.cf` c'è la riga:

```
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
```

che non permette di ricevere email se non dall'interfaccia di loopback, tuttavia esistono delle RBL che mettono in lista nera i server che hanno la porta 25 aperta. Per ovviare questo è necessario pubblicare dei recordi SPF e DKIM nella zona del proprio dominio. La cosa più semplice per uso hobbistico invece è modificare `/etc/postfix/main.cf` sostituendo la riga

```
inet_interfaces = all
```

con

```
inet_interfaces = loopback-only
```

e riavviare postfix. L'ultima porta da sistemare è la 111, occupata dal processo rcpbind. Questo è dovuto al fatto che Debian installa portmap di default, e per rimuoverlo basta dare il seguente comando:

```bash
root@debian:~# dpkg --purge rpcbind nfs-common
```

## Aggiornamento

&Egrave; stata rilasciata la versione 7.3.0 di OpenProject. La procedura di aggiornamento è semplice, ma richede un minimo di attenzione. Servono i seguenti comandi:

```bash
root@debian:~# apt-get update
root@debian:~# apt-get install --only-upgrade openproject
root@debian:~# apt-get openproject configure
```

In questo modo viene scompattato il pacchetto nuovo, ma per aggiornare il software viene usato il comando `openproject configure` che provvedere ad applicare tutte le migrazioni del database, se necessario.

Nella versione 7.3.0 di fine Settembre 2017 è stata introdotta la possibilità di citare gli utenti con `@nome` come nella suite Atlassian. Per fine Novembre 2017 è prevista la versione 7.4 che avrà altre evoluzioni minori, mentre per Marzo 2018 è previsto il rilascio di OpenProject 8.0 con un editor WYSIWYG per il wiki.

Per chi fosse interessato è possibile fare riferimento alla [roadmap](https://community.openproject.com/projects/openproject/timelines/62) ed alla [development timeline](https://community.openproject.com/projects/openproject/timelines/36) del progetto.
