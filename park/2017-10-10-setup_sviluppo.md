---
layout: post
title: "Setup di sviluppo"
modified: 2017-10-05T00:00:00+02:00
categories: blog
excerpt: boottare da una microSD card non è sempre la cosa più comoda
tags: linux development embedded yocto debian stretch
keywords: linux development embedded yocto debian stretch
---

Agosto e Settembre sono stati due mesi poco fruttuosi per i post relativi allo sviluppo embedded. Tra ferie, impegni lavorativi e didattici ed un problema che ho avuto di incompatibilità di un plugin di Jekyll e Github pages, il risultato è stato non solo che non sono andato avanti con la pubblicazione degli articoli du GNU/Linux embedded, ma addirittura ne ho scritto uno che è andato perso per un `git push -f` che mi ha riscritto la storia del repository. Ecco quindi la riscrittura del post.

Nei post precedenti avevamo visto come fare una build di Yocto Pyro, installarla sulla microSD card e fare il boot sulla Beaglebone Black. Questo modo di lavorare funziona, ma non è comodo, sopratutto nelle fasi quando di ha a che fare con le parti a più basso sistema dove di deve reboottare spesso per cambiare kernel, script di init e altri componenti del sistema che richiedono midofiche a più file.

Per ovviare questo conviene evitare di sviluppare usando una microSD, ma è meglio appoggiarsi ai servizi TFTP ed NFS, tenendo la microSD solo per avviare lo u-boot. Il servizio Trivial FTP viene usato per trasferire un kernel dal PC host allo u-boot del target affinché lo esegua. Una volta avviato il kernel, questo monterà il RootFS non dalla partizione della microSD ma dal server NFS.

<div markdown="0">
    <div class="mermaid" id="unix_tree">
	graph LR
	H[Server NFS] -->|RootFS| G[Beaglebone Black]
	I[Server TFTP] -->|Kernel| G[Beaglebone Black]
    </div>
</div>

Il diagramma temporale delle azioni è il seguente:

<div markdown="0">
    <div class="mermaid" id="unix_tree">
        sequenceDiagram
            participant Beaglebone
            participant TFTP
            participant NFS
        
            loop MLO avvia u-boot
                Beaglebone ->>Beaglebone: Avvio u-boot
            end
        
            Beaglebone ->>TFTP: Richiesta kernel
            TFTP ->> Beaglebone: Invio kernel 
        
            loop U-boot avvia il kernel
                Beaglebone ->>Beaglebone: Avvio kernel
            end
        
	    Beaglebone ->>NFS: Richiesta mount RootFS
            NFS ->> Beaglebone: Esportazione RootFS

            loop Il kernel avvia init
                Beaglebone ->>Beaglebone: Avvio sistema
            end

    </div>
</div>

Per ottenere questo setup sono necessari tre passi:
* Configurare il servizio TFTP
* Configurare il servizio NFS
* Modificare la configurazione di u-boot

## Configurare il servizio TFTP

Installare il server TFTP in debian è immediato

```bash
root@debian:~# apt-get install tftpd
```

A questo punto il server TFTP è già pronto per servire i file che sono presenti in `/srv/tftp`. Se desiderate cambiare la configurazione del server TFTP dovete editare il file `/etc/inetd.conf`, perché tftpd viene eseguito con inetd e [TCP wrapper](https://en.wikipedia.org/wiki/TCP_Wrapper). I programmi `/usr/sbin/inetd` e `/usr/sbin/tcpd` servono per accettare le connessioni di rete e poi passarle ai rispettivi programmi, permettendo una serie di vantaggi:
* `/usr/sbin/tcpd` è un programma che può loggare in modo standard gli accessi ai servizi;
* mediante i file `/etc/hosts.allow` e `/etc/hosts.deny` è possibile definire delle regole per restringere l'accesso ai servizi ai soli indirizzi IP autorizzati;
* tutta la comunicazione di rete viene gestita da `/usr/sbin/inetd` che passa il file descriptor del socket al programma, per cui il programma che viene avviato dopo la connessione può operare su stdin e stdout ed ignorare tutta la programmazione di rete. &Egrave; meglio appoggiarsi ad inetd se non si è pratici nella programmazione di rete

Nel file `/etc/inetd.conf` vediamo le seguenti opzioni per il server TFTP:

```
# /etc/inetd.conf:  see inetd(8) for further informations.
#
# Internet superserver configuration database
#
#
# Lines starting with "#:LABEL:" or "#<off>#" should not
# be changed unless you know what you are doing!
#
# If you want to disable an entry so it isn't touched during
# package updates just comment it out with a single '#' character.
#
# Packages should modify this file by using update-inetd(8)
#
# <service_name> <sock_type> <proto> <flags> <user> <server_path> <args>
tftp dgram udp wait nobody /usr/sbin/tcpd /usr/sbin/in.tftpd /srv/tftp
```

Questa riga può essere spezzata in più parti
* `tftp` : tftpd deve essere seguito sulla porta standard che IANA ha riservato per il TFTP, ovvero la 69. 
* `dgram udp` : tftpd deve rispondere al protocollo UDP
* `nobody` : tftpd deve essere eseguito con utente nobody, ovvero con il minimo dei privilegi nel sistema
* `/usr/sbin/tcpd` : tftp viene eseguito con TCP wrapper
* `/usr/sbin/in.tftpd` : è il path del server TFTP
* `/srv/tftp` : è la directory dove cercare i file

Per servire il kernel mediante TFTP dobbiamo copiare due file nella directory `/srv/tftp`:

```bash
root@debian:/home/user/poky/bbb# cp tmp/deploy/images/beaglebone/zImage /srv/tftp
root@debian:/home/user/poky/bbb# cp tmp/deploy/images/beaglebone/zImage-bbb-nohdmi.dtb /srv/tftp
```

## Configurare il servizio NFS

Per installare il servizio NFS è necessario installre il pacchetto `nfs-kernel-server`:

```bash
root@debian:~# apt-get install nfs-kernel-server
```

Anche i questo caso il servizio deve essere configurato. Nello specifico, la ocnfigurazione del server NFS viene effettuata con il file `/etc/exports`:

```
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/srv/nfs        10.0.0.0/255.0.0.0(rw,sync,no_subtree_check,no_root_squash,no_all_squash)
```

Per similitudine con il servizio TFTP, di solito uso `/srv/nfs` per il servizio NFS. Anche in questo caso la riga di configurazione può essere spezzata in più parti per spiegare cosa faccia:
* `/srv/nfs` : è la directory radice del server NFS, tutto quello che viene esportato è solo lì dentro;
* `10.0.0.0/255.0.0.0` : indica per quali indirizzi IP la regola sia valida;
* `rw` : il volume viene esportato in read e write;
* `sync` : le scritture sul volume di rete sono tutte sincrone, cosa molto comoda nello sviluppo di piattaforme embedded, sopratutto in caso di crash;
* `no_subtree_check` : rilassa delle impostazioni di sicurezza del server NFS. Questo ha una implicazione pratica nello sviluppo embedded: permette di rinominare sul pc le directory anche se il target ha un file aperto all'interno della cartella rinominata. Non è una cosa che accade spesso, però non sperlo può essere causa di giornate perse;
* `no_root_squash` : disabilita la mappatura dell'utente root su nobody. Per lo sviluppo di applicazioni embedded questo va bene, ma per altri usi del server NFS in generale questa opzione è nociva. Quando un volume NFS viene montato da un target, tutti i controlli sui diritti e gli utenti vengono fatti dal client e non dal server. Questo vuol dire che l'utente root del client ha gli stessi diritti dell'utente root del server, ma non è detto che l'utente root sia lo stesso sui due computer. Per questo motivo è buona prassi rimappare l'utente root del client sull'utente nobody, per evitare che un utente malevolo possa far fuori tutti i file della share. Questo non va bene nello sviluppo di sistemi embedded, perché se l'utente root ha il minimo possibile dei diritti il boot dello userspace è destinato a fallire rovinosamente.
* `no_all_squash` : è come `no_root_squash` applicato a tutti gli utenti. Non è detto che serva sempre, fa comodo disabilitarlo in questo setup.

A questo punto dobbiamo mettere il rootfs in `/srv/nfs`:

```bash
root@debian:/home/user/poky/bbb# mkdir /srv/nfs/root
root@debian:/home/user/poky/bbb# cp tmp/deploy/images/beaglebone/core-image-minimal-beaglebone.tar.xz /srv/nfs/root
root@debian:/home/user/poky/bbb# cd /srv/nfs/root
root@debian:/srv/nfs/root# tar xfvJ core-image-minimal-beaglebone.tar.xz
root@debian:/srv/nfs/root# rm core-image-minimal-beaglebone.tar.xz
```

## Configurare la microSD
