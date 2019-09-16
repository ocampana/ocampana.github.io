---
layout: post
title: "La densit&agrave; di pixel e la norma EN/IEC 62676-4"
excerpt: "Il successore al criterio di Johnson per comparare i sistemi di videosorveglianza"
categories: articles
comments: true
tags: cctv
keyworks: criterio johnson cctv
share: true
---


## Calcolatore

Per gli impazienti, ecco un calcolatore per determinare le prestazioni di un sistema di visione, secondo la norma EN/IEC 62676-4.

<div markdown="0">
<table>
  <tr>
    <th>Telecamera</th>
    <td>
      <select id="telecamera" name="telecamera" style="width:100%;">
        <option value="videotec_vcmhd30x01">Videotec VCMHD30X01</option>
        <option value="sony_fcb_ev7100">Sony FCB-EV7100</option>
        <option value="sony_fcb_ev7300">Sony FCB-EV7300</option>
        <option value="sony_fcb_ev7500">Sony FCB-EV7500</option>
        <option value="sony_fcb_ev7520">Sony FCB-EV7520</option>
        <option value="sony_fcb_se600">Sony FCB-SE600</option>
        <option value="sony_fcb_ex1020p">Sony FCB-EX1020P</option>
        <option value="sony_fcb_ex1020">Sony FCB-EX1020</option>
        <option value="flir_46336013HSPNLX">Flir Tau2 336x256 13mm</option>
        <option value="flir_46640013HSPNLX">Flir Tau2 640x512 13mm</option>
        <option value="flir_46336019HSPNLX">Flir Tau2 336x256 19mm</option>
        <option value="flir_46640019HSPNLX">Flir Tau2 640x512 19mm</option>
        <option value="flir_46336025HSPNLX">Flir Tau2 336x256 25mm</option>
        <option value="flir_46640025HSPNLX">Flir Tau2 640x512 25mm</option>
        <option value="flir_46336035HSPNLX">Flir Tau2 336x256 35mm</option>
        <option value="flir_46640035HSPNLX">Flir Tau2 640x512 35mm</option>
        <option value="flir_46336050HSPNLX">Flir Tau2 336x256 50mm</option>
        <option value="flir_46640050HSPNLX">Flir Tau2 640x512 50mm</option>
        <option value="flir_46633660HSPNLX">Flir Tau2 336x256 60mm</option>
        <option value="flir_46640060HSPNLX">Flir Tau2 640x512 60mm</option>
        <option value="flir_lepton_50">Flir Lepton 80x60 50mm</option>
        <option value="flir_lepton_25">Flir Lepton 80x60 25mm</option>
        <option value="flir_lepton3_50">Flir Lepton3 160x120 50mm</option>
      </select>
    </td>
    <th><div style="overflow: hidden; white-space: nowrap;">Monitor (m)</div></th>
    <td><input id="monitoring" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Risoluzione orizzontale</div></th>
    <td><input type="text" id="risoluzione_orizzontale" name="risoluzione_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Detect (m)</div></th>
    <td><input id="detection" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele orizzontale</div></th>
    <td><input type="text" id="angolo_tele_orizzontale" name="angolo_tele_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Observe (m)</div></th>
    <td><input id="observation" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele verticale</div></th>
    <td><input type="text" id="angolo_tele_verticale" name="angolo_tele_verticale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Recognise (m)</div></th>
    <td><input id="recognition" type="text" /></td>
  </tr>
  <tr>
    <th></th>
    <td></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Identify (m)</div></th>
    <td><input id="identification" type="text" /></td>
  </tr>
  <tr>
    <th></th>
    <td></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Inspect (m)</div></th>
    <td><input id="inspect" type="text" /></td>
  </tr>
</table>

<script>
function aggiorna_distanze ()
{
  var res = document.querySelector('#risoluzione_orizzontale').value;
  var angolo = document.querySelector('#angolo_tele_orizzontale').value;

  var d_monitoring = Math.round ( (res / (2 * 12) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#monitoring').value = d_monitoring;

  var d_detection = Math.round ( (res / (2 * 25) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#detection').value = d_detection;

  var d_observation = Math.round ( (res / (2 * 62) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#observation').value = d_observation;

  var d_recognition = Math.round ( (res / (2 * 125) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#recognition').value = d_recognition;

  var d_identification = Math.round ( (res / (2 * 250) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#identification').value = d_identification;

  var d_inspect = Math.round ( (res / (2 * 1000) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#inspect').value = d_inspect;
}

function aggiorna_telecamera (aggiorna_dati)
{
  if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7520')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.3;
    document.querySelector('#angolo_tele_orizzontale').value = 2.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7500')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.3;
    document.querySelector('#angolo_tele_orizzontale').value = 2.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7300')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.85;
    document.querySelector('#angolo_tele_orizzontale').value = 3.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7100')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 4.725;
    document.querySelector('#angolo_tele_orizzontale').value = 7.6;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_se600')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 18;
    document.querySelector('#angolo_tele_orizzontale').value = 32;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ex1020p')
  {
    document.querySelector('#risoluzione_orizzontale').value = 720;
    document.querySelector('#angolo_tele_verticale').value = 1.275;
    document.querySelector('#angolo_tele_orizzontale').value = 1.7;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ex1020')
  {
    document.querySelector('#risoluzione_orizzontale').value = 720;
    document.querySelector('#angolo_tele_verticale').value = 1.275;
    document.querySelector('#angolo_tele_orizzontale').value = 1.7;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336013HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 9.5;
    document.querySelector('#angolo_tele_orizzontale').value = 12.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640013HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 18.5;
    document.querySelector('#angolo_tele_orizzontale').value = 22.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336019HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 6.5;
    document.querySelector('#angolo_tele_orizzontale').value = 8.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640019HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 13;
    document.querySelector('#angolo_tele_orizzontale').value = 16;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336025HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 5;
    document.querySelector('#angolo_tele_orizzontale').value = 6.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640025HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 10;
    document.querySelector('#angolo_tele_orizzontale').value = 12.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336035HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 3.55;
    document.querySelector('#angolo_tele_orizzontale').value = 4.65;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640035HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 7;
    document.querySelector('#angolo_tele_orizzontale').value = 9;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336050HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 2.5;
    document.querySelector('#angolo_tele_orizzontale').value = 3.25;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640050HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 4.95;
    document.querySelector('#angolo_tele_orizzontale').value = 6.2;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336060HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 2.1;
    document.querySelector('#angolo_tele_orizzontale').value = 2.75;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640060HSPNLX')
  {
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 4.15;
    document.querySelector('#angolo_tele_orizzontale').value = 5.2;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton_50')
  {
    document.querySelector('#risoluzione_orizzontale').value = 60;
    document.querySelector('#angolo_tele_verticale').value = 25.5;
    document.querySelector('#angolo_tele_orizzontale').value = 18.75;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton_25')
  {
    document.querySelector('#risoluzione_orizzontale').value = 60;
    document.querySelector('#angolo_tele_verticale').value = 12.5;
    document.querySelector('#angolo_tele_orizzontale').value = 9.37;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton3_50')
  {
    document.querySelector('#risoluzione_orizzontale').value = 120;
    document.querySelector('#angolo_tele_verticale').value = 28;
    document.querySelector('#angolo_tele_orizzontale').value = 21;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'videotec_vcmhd30x01')
  {
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.32;
    document.querySelector('#angolo_tele_orizzontale').value = 2.36;
  }

  if (aggiorna_dati)
      aggiorna_distanze ();
}

document.addEventListener('DOMContentLoaded', function() {

  els = document.querySelector('select[name="telecamera"] option[value="sony_fcb_ev7520"]');

  if(els)
  {
    els.selected = true;
  }

  aggiorna_telecamera (false);

  aggiorna_distanze ();

  document.getElementById('telecamera').addEventListener('change', function() {
      aggiorna_telecamera (true);
  });
}, false);

</script>
</div>

## Spiegazione della norma EN/IEC 62676-4

La norma EN/IEC 62676-4 raccoglie i requisiti e le raccomandazioni per la corretta progettazione, installazione, verifica e manutenzione di un Video Surveillance System (VSS).

Come tutte le normative più moderne, anche la 62676-4 &egrave; una norma basata sull'analisi dei rischi. L'analisi deve considerare:

* il costo delle perdite
* l'ubicazione del sito
* le restrizioni di accesso al sito
* gli eventi pregressi

L'output di tale analisi confluir&agrave; nel Documento dei Requisiti Operativi (DRO) e a ciascun elemento da proteggere verr&agrave; assegnato il grado di rischio, come definito nella EN/IEC 62676-1-1. Il grado di rischio pu&ograve; assumere i seguenti valori:

* **1**, ovvero _low risk_, che identifica un sistema o sito che non necessita del controllo delle funzioni perch&eacute; non esposto al rischio di manomissioni
* **2**, ovvero _low to medium risk_, che identifica un sistema o sito in cui non viene monitorato il funzionamento perch&eacute; dispone di protezioni semplici contro le manomissioni. In un impianto di videosorveglianza per un obiettivo di questo tipo, questo si traduce nel fatto che le telecamere non sono accessibili dall'esterno.
* **3**, ovvero _medium to high risk_, che identifica un sistema o sito in cui vengono controllati sia il funzionamento che le manomissioni. A questo livello di rischio corrisponde anche una preprazione tecnica di base dell'aggressore, che potrebbe sapere come manomettere il sistema di videosorveglianza.
* **4**, ovvero _high risk_, che identifica un sistema o sito con elevate protezioni contro le manomissioni in cui viene garantito anche il controllo della funzionalità. A questo livello di rischio sono associate un'alta competenza tecnica degli intrusori e non &egrave; ammesso perdere le registrazioni.

In base al grado rischio di ciascun elemento da proteggere potr&agrave essere usata una telecamera per verificare il corretto funzionamento del bene. In base all'oggetto da proteggere, la camera potr&agrave; effettuare una delle azioni previste dal modello **MDORII**:

* **M**onitor che richiede 12 pixel/metro sul bersaglio
* **D**etect che richiede 25 pixel/metro sul bersaglio
* **O**bserve che richiede 62 pixel/metro sul bersaglio
* **R**ecognise che richiede 125 pixel/metro sul bersaglio
* **I**dentify che richiede 250 pixel/metro sul bersaglio
* **I**nspect che richiede 1000 pixel/metro sul bersaglio

Come accadeva nel criterio di Johnson, la massima distanza a cui una telecamera pu&ograve; garantire la risoluzione deciderata dipende dalla lente e dalle carettaristiche del sensore.

## Un po' di formule

La distanza basata sulla EN/IEC 62676-4 &egrave; facilmente ricavabile con la trigonometria dei triangoli rettangoli.

\\[ d = \frac{res}{2} . \frac{1}{pixel per metro} . \frac{1}{\tan( \alpha_{tele, hor}) } \\]

Dove con &alpha; si indica l'angolo di visione, deifnito come nella [pagina del criterio di Johnson](../criterio-di-johnson).
