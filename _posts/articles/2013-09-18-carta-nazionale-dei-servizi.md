---
layout: post
title: "Carta nazionale dei servizi"
excerpt: "Detta anche firma digitale"
categories: articles
tags: [linux, cns]
redirect_from: "/software/linux/cns"
---

Mi sono comprato una [penna usb per la firma digitale](http://www.vi.camcom.it/a_ITA_2551_1.html), perché avevo voglia di fare alcuni esperimenti. La camera di commercio rivende le penne per la firma digitale di Aruba, l'ho comprata in camera di commercio perché mi era più comodo che spedire documenti ed attendere la ricezione via posta della chiavetta. Un paio di anni fa mi era capitato di vedere che la chiavetta di Aruba aveva dei problemi, tuttavia questi sono stati poi risolti, almeno a leggere su internet.

La chiavetta di Aruba funziona abbastanza bene, perché ha una serie di binari che possono essere eseguiti direttamente dalla pennina, senza dover installare altre applicazioni. Questo può essere desiderabile per un utente normale, ma io volevo fare qualcosa di più e capirci qualcosa di più, quindi ho cercato di utilizzare la chiavetta con programmi esterni, installati sul computer. Qui riporto le mi osservazioni con la chiavetta.

**Premessa**: le mie prove sono fatte con Debian Wheezy 32 bit. All'inizio avevo provato con la 64bit, ma il multiarch mi dava qualche dubbi e ho preferito semplificare l'ambiente di prova. Questo non esclude che possa funzionare anche a 64, solo non ci sono ancora riuscito.

## Installazione

Quando colleghiamo il dispositivo alla porta USB del computer vedremo comparire 3 dispositivi usb:

    ottavio@pc:~$ lsusb 
    ...
    Bus 002 Device 003: ID 058f:6254 Alcor Micro Corp. USB Hub
    Bus 002 Device 004: ID 048d:1167 Integrated Technology Express, Inc. 
    Bus 002 Device 005: ID 2021:0002  
    ottavio@pc:~$

Quindi possiamo vedere che in verità è composto da tre cose:
* una hub usb cui collegare gli altri dispositivi (0x058f:0x6254)
* una memoria flash che è quella dove si possono salvare i file (0x048d:0x1167)
* un dispositivo anonimo (0x2021:0x0002)

Ebbene il dispositivo anonimo è un dispositivo HID che si occupa della crittografia relativa alla firma digitale. I dispositivi HID (Human Interface Device) sono una classe di dispositivi USB inizialmente pensata per tastiere, mouse e simili, ma poiché ha una interfaccia generica viene usato per implementare anche altri dispositivi.

Sotto Debian, la prima volta che il dispositivo viene collegato al computer appare come dispositivo `/dev/hidraw?`  e viene montato come una qualsiasi pennina USB. La chiavetta di Aruba ha una directory `ArubaKeyLinux/` in cui è installato tutto il software. All'interno di questa directory c'è il file `launcher.bat`, che quando viene eseguito lancia un eseguibile scritto in QT che provvede ad inizializzare il dispositivo.

L'inizializzazione del dispositivo è necessaria perché per funzionare la chiavetta ha bisogno di cambiare la propria modalità di funzionamento e che vengano "taroccati" i diritti di alcuni file in `/dev`. Come cambia il funzionamento la chiavetta?

**Appena inserita**

    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               1.10
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0 
      bDeviceProtocol         0 
      bMaxPacketSize0         8
      idVendor           0x2021 
      idProduct          0x0002 
      bcdDevice            1.00
      iManufacturer           1 
      iProduct                2 
      iSerial                 0 
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           41
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0 
        bmAttributes         0x80
          (Bus Powered)
        MaxPower              100mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass         3 Human Interface Device
          bInterfaceSubClass      0 No Subclass
          bInterfaceProtocol      0 None
          iInterface              0 
            HID Device Descriptor:
              bLength                 9
              bDescriptorType        33
              bcdHID               1.00
              bCountryCode            0 Not supported
              bNumDescriptors         1
              bDescriptorType        34 Report
              wDescriptorLength      34
             Report Descriptors: 
               ** UNAVAILABLE **
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval              10
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval              10

**Commutata**

    Device Descriptor:
      bLength                18
      bDescriptorType         1
      bcdUSB               1.10
      bDeviceClass            0 (Defined at Interface level)
      bDeviceSubClass         0 
      bDeviceProtocol         0 
      bMaxPacketSize0         8
      idVendor           0x2021 
      idProduct          0x0002 
      bcdDevice            1.00
      iManufacturer           1 AK910
      iProduct                2 HKey
      iSerial                 0 
      bNumConfigurations      1
      Configuration Descriptor:
        bLength                 9
        bDescriptorType         2
        wTotalLength           41
        bNumInterfaces          1
        bConfigurationValue     1
        iConfiguration          0 
        bmAttributes         0x80
          (Bus Powered)
        MaxPower              100mA
        Interface Descriptor:
          bLength                 9
          bDescriptorType         4
          bInterfaceNumber        0
          bAlternateSetting       0
          bNumEndpoints           2
          bInterfaceClass         3 Human Interface Device
          bInterfaceSubClass      0 No Subclass
          bInterfaceProtocol      0 None
          iInterface              0 
            HID Device Descriptor:
              bLength                 9
              bDescriptorType        33
              bcdHID               1.00
              bCountryCode            0 Not supported
              bNumDescriptors         1
              bDescriptorType        34 Report
              wDescriptorLength      34
              Report Descriptor: (length is 34)
                Item(Global): Usage Page, data= [ 0x00 0xff ] 65280
                                (null)
                Item(Local ): Usage, data= [ 0x01 ] 1
                                (null)
                Item(Main  ): Collection, data= [ 0x01 ] 1
                                Application
                Item(Local ): Usage, data= [ 0x01 ] 1
                                (null)
                Item(Global): Logical Minimum, data= [ 0x00 ] 0
                Item(Global): Logical Maximum, data= [ 0xff 0x00 ] 255
                Item(Global): Report Count, data= [ 0x40 ] 64
                Item(Global): Report Size, data= [ 0x08 ] 8
                Item(Main  ): Input, data= [ 0x00 ] 0
                                Data Array Absolute No_Wrap Linear
                                Preferred_State No_Null_Position Non_Volatile Bitfield
                Item(Local ): Usage, data= [ 0x01 ] 1
                                (null)
                Item(Global): Logical Minimum, data= [ 0x00 ] 0
                Item(Global): Logical Maximum, data= [ 0xff 0x00 ] 255
                Item(Global): Report Count, data= [ 0x40 ] 64
                Item(Global): Report Size, data= [ 0x08 ] 8
                Item(Main  ): Output, data= [ 0x00 ] 0
                                Data Array Absolute No_Wrap Linear
                                Preferred_State No_Null_Position Non_Volatile Bitfield
                Item(Main  ): End Collection, data=none
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x02  EP 2 OUT
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval              10
          Endpoint Descriptor:
            bLength                 7
            bDescriptorType         5
            bEndpointAddress     0x82  EP 2 IN
            bmAttributes            3
              Transfer Type            Interrupt
              Synch Type               None
              Usage Type               Data
            wMaxPacketSize     0x0040  1x 64 bytes
            bInterval              10
    Device Status:     0x0000
      (Bus Powered)

Come vediamo la pennina continua ad essere un dispositivo HID, ma adesso esporta tutte le sue funzionalità, cosa che prima non faceva. La parte interessante è che nel momento in cui viene avviato il `launcher.bat` la pennina smette di essere visibile come dispositivo `/dev/hidraw?` .

Cosa possiamo fare se non vogliamo dover eseguire il launcher, che richiede i diritti di diventare root, ogni volta che vogliamo usare la pennina? La possibilità è offerta dal software presente nella penna, solo che è imbucato. Avviato il launcher, selezionate la voce "Utilità" e da lì "Import certificato". Questa voce farà eseguire lo script `ArubaKeyLinux/Main_lnx/drivers/inst_hid.bat`, che farà due cose:

* installerà il binario `/usr/local/sbin/start_hid_ak`
* installerà la regola `/etc/udev/rukes.d/99-hid-ak-device.rules` che richiamerà il precedente programma e correggerà automaticamente i diritti

Il contenuto della regola è

    ATTRS{idVendor}=="2021", ATTRS{idProduct}=="0002", MODE="0666" 
    ACTION=="add", KERNEL=="hidraw*" ATTRS{idVendor}=="2021", ATTRS{idProduct}=="0002", RUN:="/usr/local/sbin/start_hid_ak", MODE="0666"


A questo punto, riavviato udev o fatto un reboot, la pennina si metterà in piedi senza dover rilanciare tutte le volte il launcher.

## Driver

I dispositivi HID prevedono una interfaccia abbastanza semplice perché demandano poi in userspace buona parte del loro funzionamento. Anche i security token come la pennina in questione non fanno eccezione. Quando si legge in giro dei driver per i security token spesso ci si riferisce proprio a questi componenti in userspace. In molti forum la gente si lamenta che il dispositivo non funziona perché ignora la parte precedente di commutazione del dispositivo.

La smartcard inserita nel mio securty token è una smartcard di tipo ST Incard. Sul [sito di Aruba](https://www.pec.it/Download.aspx) si cita anche un altro possibile tipo di smartcard, la Oberthur. In entrambi i casi il lettore della smartcard è fatto da bit4id, ma poiché ci sono due librerie diverse per i due tipi di smartcard bisogna fare attenzione. Per ST Incard, il driver giusto è `ArubaKeyLinux/Main_lnx/libbit4ipki.so` , mentre in altre pagine parla di `libbit4opki.so` . Quindi attenzione: libbit4**i**pki per Incard, libbit4**o**pki per Oberthur. Sempre in `ArubaKeyLinux/Main_lnx/` c'è un file `blob.bin` che puzza tanto da firmware da caricare sulla pennetta. Io ho provato a copiare il driver sul mio disco fisso senza copiare il blob.bin e non mi funziona, dovrò fare delle altre prove.

Avendo capito quale è il driver giusto possiamo passare al passaggio successivo, ovvero iniziare a smanettare con la pennina.

## Opensc

Iniziamo subito con un bel `sudo apt-get install opensc `. Vediamo per iniziare se il security token viene riconosciuto (ho taroccato i seriali, per ovvi motivi di riservatezza):

    ottavio@pc:~$ pkcs11-tool --module /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so -IOLM
    STARTUP: Closed open FD 3
    console log enabled
    STARTUP: arguments:
    0: '/home/ottavio/.hsvc/hsvc'
    1: '/tmp/pcsc_ak_socket'
    STARTUP: grace time: 30
    CRITICAL: connectSocket(): connect socket error: 2
    STARTUP: no service was running on '/tmp/pcsc_ak_socket' 
    DAEMONIZING pid=4160...
    Cryptoki version 2.20
    Manufacturer     bit4id srl
    Library          bit4id PKCS#11 (ver 1.2)
    Available slots:
    Slot 0 (0x0): DL-AK910 HKey 0
      token label:   CNS
      token manuf:   ST Incard
      token model:   T&S DS/2048 (LB)
      token flags:   rng, login required, PIN initialized, token initialized
      serial num  :  XXXXXXXXXXXXXXXX
    Using slot 0 with a present token (0x0)
    Supported mechanisms:
      RSA-PKCS, keySize={1024,2048}, hw, decrypt, sign
      RSA-X-509, keySize={1024,2048}, hw, decrypt, sign
      RSA-PKCS-KEY-PAIR-GEN, keySize={1024,2048}, hw, generate_key_pair
      SHA-1, digest
      SHA256, digest
      SHA384, digest
      SHA512, digest
      SHA384-RSA-PKCS, keySize={1024,2048}, hw, sign
      SHA256-RSA-PKCS, keySize={1024,2048}, hw, sign
      SHA512-RSA-PKCS, keySize={1024,2048}, hw, sign
      SHA1-RSA-PKCS, keySize={1024,2048}, hw, sign
    Certificate Object, type = X.509 cert
      label:      CNS User Certificate
      ID:         11111111
    Data object 1002
      label:          'PDATA'
      application:    'PDATA'
      app_id:         XXXXXXXXXX
      flags:           modifiable
    Public Key Object; RSA 1024 bits
      label:      CNS User Public Key
      ID:         11111111
      Usage:      encrypt, verify
    Certificate Object, type = X.509 cert
      label:      DS User Certificate0
      ID:         222222
    Public Key Object; RSA 1024 bits
      label:      DS User Public Key0
      ID:         222222
      Usage:      encrypt, verify
    ottavio@pc:~$

Vediamo che la carta supporta la gestione di chiavi RSA da 1024 e 2048 bit, ma che il certificato generato dalla camera di commercio è di 1024bit.

Come mai la carta mostra solo chiavi pubbliche? Perché la lettura dei dati presenti sulla carta è stata fatta senza effettuare prima il login. Rilanciamo il comando per elencare gli oggetti facendo prima il login alla smartcard:

    ottavio@pc:~$ pkcs11-tool --module /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so -O -l
    STARTUP: Closed open FD 3
    console log enabled
    STARTUP: arguments:
    0: '/home/ottavio/.hsvc/hsvc'
    1: '/tmp/pcsc_ak_socket'
    STARTUP: grace time: 30
    CRITICAL: connectSocket(): connect socket error: 2
    STARTUP: no service was running on '/tmp/pcsc_ak_socket' 
    DAEMONIZING pid=3434...
    Using slot 0 with a present token (0x0)
    Logging in to "CNS".
    Please enter User PIN: 
    Certificate Object, type = X.509 cert
      label:      CNS User Certificate
      ID:         11111111
    Data object 1002
      label:          'PDATA'
      application:    'PDATA'
      app_id:         XXXXXXXXXX
      flags:           modifiable
    Public Key Object; RSA 1024 bits
      label:      CNS User Public Key
      ID:         11111111
      Usage:      encrypt, verify
    Private Key Object; RSA 
      label:      CNS User Private Key
      ID:         11111111
      Usage:      decrypt, sign
    Certificate Object, type = X.509 cert
      label:      DS User Certificate0
      ID:         222222
    Private Key Object; RSA 
      label:      DS User Private Key 0
      ID:         222222
      Usage:      sign
      Access:     always authenticate
    Public Key Object; RSA 1024 bits
      label:      DS User Public Key0
      ID:         222222
      Usage:      encrypt, verify
    ottavio@pc:~$

Come vedete questa volta viene richiesto il PIN della smartcard e, autenticati con successo, possiamo vedere anche le chiavi private.

## OpenSSL

**Premessa**: sono riuscito a firmare file con openssl e a verificare con successo la firma con il software di Aruba presente sulla pennina USB. Per fare questo ho dovuto patchare openssl. A fine pagina c'è la patch che ho sviluppato. Ho creato anche un [repository su github](https://github.com/ocampana/openssl), per tenere aggiornata la patch con i sorgenti di openssl, sperando che venga inclusa.

La prima cosa da fare per firmare è estrarre il certificato pubblico della chiave. L'operazione di estrazione è prevista da tutti i security token e può essere effettuata con il seguente comando:

    ottavio@pc:~$ pkcs11-tool --module /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so -r --type cert -d 222222 > out.der

Per chi non lo sapesse, `der` è un formato di codifica di [ASN.1](http://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One), che serve per raccogliere una serie di valori. Questo formato non viene usato da openssl, quindi il passo successivo consiste obbligatoriamente nella sua conversione nel formato pem

    ottavio@pc:~$ openssl x509 -inform der -in out.der -out out.pem

Cosa c'è dentro questo file `out.pem`? C'è il formato x509 che descrive la chiave pubblica associata a quella privata che è salvata nella smartcard del security token. Vediamo cosa contiene (anche qua, ho nascosto alcuni dati):

    ottavio@pc:~$ openssl x509 -in out.pem -text

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number:
                XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: C=IT, O=ArubaPEC S.p.A., OU=Certification AuthorityC, CN=ArubaPEC S.p.A. NG CA 3
            Validity
                Not Before: XXX XX 00:00:00 XXXX GMT
                Not After : XXX XX 23:59:59 XXXX GMT
            Subject: C=IT, O=non presente, CN=CAMPANA OTTAVIO/serialNumber=IT:XXXXXXXXXXXXXXXX, GN=OTTAVIO, SN=CAMPANA/dnQualifier=XXXXXXXX
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (1024 bit)
                    Modulus:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
                        XX:XX:XX:XX:XX:XX:XX:XX:XX
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Key Usage: critical
                    Non Repudiation
                X509v3 Subject Key Identifier:
                    XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX
                X509v3 Certificate Policies: 
                    Policy: 1.3.6.1.4.1.29741.1.1.1
                      CPS: https://ca.arubapec.it/cps.html

                X509v3 CRL Distribution Points: 

                    Full Name:
                      URI:http://crl.arubapec.it/ArubaPECSpACertificationAuthorityC/LatestCRL.crl

                qcStatements: 
                    0!0......F..0......F.....0......F..
                X509v3 Authority Key Identifier: 
                    keyid:F0:C0:45:B1:B6:35:B4:EA:5F:29:FA:83:03:4A:DC:2F:F5:B3:7D:E8

                Authority Information Access: 
                    OCSP - URI:http://ocsp.arubapec.it

        Signature Algorithm: sha256WithRSAEncryption
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:XX:
             XX:XX:XX:XX
    -----BEGIN CERTIFICATE-----
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    -----END CERTIFICATE-----

Notiamo una cosa importante: la chiave deve avere l'estensione `X509v3 extensions: X509v3 Key Usage: critical Non Repudiation`. Se il vostro dump non vi mostra questo valore, vuol dire che avete estratto la chiave sbagliata (ricordate che nella CNS della camera di commercio ci sono due chiavi). Questa estensione serve per dare validità legale alla firma.

Procediamo con l'uso di OpenSSL per firmare un file prova.txt e generare il file prova.p7m:

    ottavio@pc:~$ openssl
    OpenSSL> engine -t dynamic -pre SO_PATH:/usr/lib/engines/engine_pkcs11.so -pre ID:pkcs11 -pre LIST_ADD:1 -pre LOAD -pre MODULE_PATH:/media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so
    (dynamic) Dynamic engine loading support
    [Success]: SO_PATH:/usr/lib/engines/engine_pkcs11.so
    [Success]: ID:pkcs11
    [Success]: LIST_ADD:1
    [Success]: LOAD
    [Success]: MODULE_PATH:/media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so
    Loaded: (pkcs11) pkcs11 engine
    STARTUP: Closed open FD 3
    console log enabled
    STARTUP: arguments:
    0: '/home/ottavio/.hsvc/hsvc'
    1: '/tmp/pcsc_ak_socket'
    STARTUP: grace time: 30
    CRITICAL: connectSocket(): connect socket error: 2
    STARTUP: no service was running on '/tmp/pcsc_ak_socket' 
    DAEMONIZING pid=5528...
         [ available ]
    OpenSSL> smime -md sha256 -nodetach -binary -outform DER -sign -signer out.pem -inkey id_222222 -keyform engine -in prova.txt -out prova.p7m -engine pkcs11
    STARTUP: Closed open FD 3
    console log enabled
    STARTUP: arguments:
    0: '/home/ottavio/.hsvc/hsvc'
    1: '/tmp/pcsc_ak_socket'
    STARTUP: grace time: 30
    CRITICAL: connectSocket(): connect socket error: 2
    STARTUP: no service was running on '/tmp/pcsc_ak_socket' 
    DAEMONIZING pid=5556...
    engine "pkcs11" set.
    PKCS#11 token PIN: 
    OpenSSL>

Vediamo che openssl con il primo comando carica l'engine pkcs11 per interfacciarsi con il security token, e poi per effettuare la firma chiede il pin per sbloccare la chiave privata presente sulla pennina.

**Cosa c'è che manca affinché tutto funzioni?** La delibera CNIPA 45/2009 richiede che nel pkcs#7 sia presente l'attributo `signingCertificateV2`, perché ha inglobato le specifiche [CMS Advanced Electronic Signatures (CAdES)](http://en.wikipedia.org/wiki/CAdES_%28computing%29), che coprono anche l'operazione di marcatura temporale. Per colmare questa lacuna ho scritto una patch (`cades.diff`) che ho mandato per revisione alla mailing list di openssl. Questa patch è in stato molto primordiale, ma dimostra che si può firmare i documenti con software 100% libero. Per inciso, facendo le prove, mi sa che ho trovato un baco nel software di verifica delle firme di Aruba. Se qualcuno di loro volesse contattarmi, glielo spiego.

## OpenSSH

    ottavio@pc:~$ ssh-keygen -D /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so

Con questo comando si estraggono le chiavi pubbliche, che vanno messe in ~`/.ssh/authorized_keys` e poi ci si potrà collegare al server con

    ottavio@pc:~$ ssh -I /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so 192.168.1.1


È inoltre possibile impostare il proprio `~/.ssh/config` affinché usi la CNS automaticamente per connettersi ai server che lo richiedono

     Host mio.host.it
     PKCS11Provider /media/2616-496E/ArubaKeyLinux/Main_lnx/libbit4ipki.so


ovviamente facendo attenzione ad impostare il path corretto verso lo shared object.

## Allegati

Patch openssl: [cades.diff](/assets/files/cades.diff)
