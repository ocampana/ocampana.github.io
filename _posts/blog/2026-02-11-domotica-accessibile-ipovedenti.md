---
layout: post
title: "Domotica accessibile con l'AI"
modified:
categories: blog
excerpt: una soluzione vocale per ipovedenti (e non solo)
tags:
keywords:
---

Qualche giorno fa, durante una cena con un vecchio amico che non vedevo da anni, mi ha raccontato di un progetto a cui stava lavorando: automatizzare la domotica di casa sua per renderla controllabile tramite voce, con l'obiettivo di migliorare la qualità della vita di sua figlia, purtroppo ipovedente. Mentre ascoltavo le sue idee, mi sono reso conto che conoscevo già tutti i "mattoncini" necessari per costruire una soluzione del genere. Così, dopo averla testata a casa mia, ho deciso di scrivere questa guida per condividerla con lui e con chiunque altro si trovi ad affrontare una sfida simile.

### La soluzione in breve

L'idea è semplice: usare un sistema domotico open-source (Home Assistant) integrato con modelli di intelligenza artificiale per interpretare comandi vocali, elaborare risposte e fornire feedback audio. Il tutto senza dipendere da servizi cloud proprietari (come Alexa o Google Assistant), garantendo maggiore privacy e personalizzazione.

I componenti chiave sono:
 * Home Assistant (versione 2026.2.1) come hub centrale.
 * Due addon di Home Assistant per l’AI:
   * Local OpenAI LLM (1.3.1) per gestire il modello linguistico.
   * OpenAI Whisper Cloud (1.3.2) per il riconoscimento vocale.
 * [Mistral AI](https://mistral.ai/) come fornitore del modello linguistico (più economico e compliant con il GDPR e l'EU AI Act che diventerà cogente in Agosto 2026).
 * PicoTTS per la sintesi vocale di base (integrato nativamente in Home Assistant).

### Passo 1: Configurare Mistral AI

Prima di tutto, è necessario registrarsi su Mistral AI e generare una chiave API all’indirizzo [https://console.mistral.ai/home](https://console.mistral.ai/home). Per comodità, io l’ho chiamata `homeassistant`, ma potete usare il nome che preferite. Questa chiave servirà per collegare Home Assistant ai servizi di Mistral.

Attenzione: Mistral, come molti provider di inferenza, ha una offerta variegata. Offre un livello gratuito con crediti limitati (verificate le condizioni aggiornate sul loro sito), ha la possibilità di funzionare in abbonamento con un canone mensile e supporta una modalità di pagamento a consumo, chiamata **API**. In generale, poiché non voglio essere legato ad un tool di un venditore specifico, io opto sempre per la fatturazione a consumo dell'accesso mediante API. Questo tipo di accesso permette spesso di pagare di meno di un abbonamento. Ad esempio, il modello `mistral-small-latest` (quello che sto usando io) ha un prezzo accessibile e funziona egregiamente per questo scopo. Trovate tutte le informazioni sui prezzi dell'uso delle API [qua](https://mistral.ai/pricing#api)

### Passo 2: Installare e configurare gli addon in Home Assistant

#### OpenAI Whisper Cloud

Installate l’addon da HACS di Home Assistant. Nella configurazione, selezionate `Mistral` come fornitore del servizio. Incollate la chiave API generata in precedenza. In automatico il sistema vi proporrà di usare il modello `voxtral-mini-latest`.

#### Local OpenAI LLM

Anche qui, installate l’addon (versione 1.3.1 o successiva) ha HACS. *Attenzione*: versioni precedenti dell'addon non funzionano con Mistral (si veda questa [issue](https://github.com/skye-harris/hass_local_openai_llm/issues/22)) su Github. Nella configurazione, oltre alla chiave API, dovete specificare l’URL del servizio Mistral: `https://api.mistral.ai/v1/`.

Se l’installazione non crea automaticamente un *conversation agent*, aggiungetelo manualmente con l’apposito pulsante nella schermata delle integrazioni. E' possibile creare anche più di un agente di conversazione, basandosi su diversi modelli (e costi). Come vi dicevo, io uso `mistral-small-latest`.

#### Passo 3: Abilitare la sintesi vocale con PicoTTS

PicoTTS è un motore di text-to-speech (TTS) basilare ma funzionale per testare la soluzione, perché in questa applicazione ci interessa di più capire il linguaggio naturale dell'utente ed eseguire azioni, piuttosto che fare conversazione. PicoTTS ha una voce robotica e limitata perché è pensato per sistemi embedded. Per attivarlo, modificate il file `configuration.yaml` di Home Assistant aggiungendo:

```
tts:
    - platform: picotts
```

Riavviate Home Assistant, in modo da abilitare tutti i componenti.

#### Passo 4: Collegare i puntini in Home Assistant

Ora è il momento di unire tutto:
* Andate in `Impostazioni` > `Assistenti vocali`.
* Verificate che ci sia un assistente predefinito chiamato `Home Assistant`. Qualora non fosse presente si può aggiungerne uno
* Nella sezione `AI Agent`, scegliete quello che avete creato con il modello che avete selezionato configurando Local OpenAI LLM (`mistral-small-latest` in questo esempio) e disabilitate la gestione dei comandi in locale (altrimenti bypassereste l’AI).
* In `Riconoscimento vocale`, impostate `Mistral AI Whisper`.
* Infine, in `Sintesi vocale`, selezionate `PicoTTS`.

Dovreste ritrovarvi con una configurazione simile a questa:

![configurazione assistente vocale home assistant](/images/20260211-impostazioni-assistente-vocale.png)

#### Testare la soluzione

A questo punto, potete usare l'interfaccia web o la companion app di Home Assistant (disponibile per Android e iOS) per interagire con il sistema. In alto destra c'è la voce `Assist` (attenzione, nella app potrebbe essere collassata nell'icone dei re puntini) con cui sarà possibile avviare una conversazione. **Attenzione**: potrebbe accadere che su Firefox non vi funzioni il microfono. Questo accade perché siete connessi all'interfaccia web mediante HTTP e non HTTPS.

Qualora il sistema vi piaccia, potete automatizzare i comandi vocali con [Voice](https://www.home-assistant.io/voice-pe/), lo smart speaker ufficiale di Home Assistant.
