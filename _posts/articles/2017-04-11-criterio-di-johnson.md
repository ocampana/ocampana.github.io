---
layout: post
title: "Il criterio di Johnson"
excerpt: "Come valutare le prestazioni di un dispositivo di videosorveglianza intermini di distanza"
categories: articles
comments: true
tags: cctv
keyworks: criterio johnson cctv
share: true
modified: 2019-03-22T00:00:00+02:00
---


## Calcolatore

Per gli impazienti, ecco un calcolatore per determinare le prestazioni di un sistema di visione

<div markdown="0">
<table>
  <tr>
    <th>Bersaglio</th>
    <td>
      <select id="bersaglio" name="bersaglio" style="width:100%;">
        <option value="uomo">Uomo</option>
        <option value="tir">TIR</option>
      </select>
    </td>
    <th>Telecamera</th>
    <td>
      <select id="telecamera" name="telecamera" style="width:100%;">
        <option value="videotec_vcmhd30x01">Videotec VCMHD30X01</option>
        <option value="sony_fcb_ev7100">Sony FCB-EV7100</option>
        <option value="sony_fcb_ev7300">Sony FCB-EV7300</option>
        <option value="sony_fcb_ev7500">Sony FCB-EV7500</option>
        <option value="sony_fcb_ev7520">Sony FCB-EV7520</option>
        <option value="sony_fcb_se600">Sony FCB-SE600</option>
        <option value="flir_46336013HSPNLX">Flir Tau2 336x256 13mm</option>
        <option value="flir_46640013HSPNLX">Flir Tau2 640x480 13mm</option>
        <option value="flir_46640060HSPNLX">Flir Tau2 640x480 60mm</option>
        <option value="flir_lepton_50">Flir Lepton 80x60 50mm</option>
        <option value="flir_lepton_25">Flir Lepton 80x60 25mm</option>
        <option value="flir_lepton3_50">Flir Lepton3 160x120 50mm</option>
      </select>
    </td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Dim. verticale (m)</div></th>
    <td><input type="text" id="dimensione_verticale" name="dimensione_verticale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Dim. orizzontale (m)</div></th>
    <td><input type="text" id="dimensione_orizzontale" name="dimensione_orizzontale" readonly /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Risoluzione orizzontale</div></th>
    <td><input type="text" id="risoluzione_orizzontale" name="risoluzione_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Detection (m)</div></th>
    <td><input id="detection" type="detection" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Risoluzione verticale</div></th>
    <td><input type="text" id="risoluzione_verticale" name="risoluzione_verticale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Orientation (m)</div></th>
    <td><input id="orientation" type="orientation" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele orizzontale</div></th>
    <td><input type="text" id="angolo_tele_orizzontale" name="angolo_tele_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Recognition (m)</div></th>
    <td><input id="recognition" type="recognition" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele verticale</div></th>
    <td><input type="text" id="angolo_tele_verticale" name="angolo_tele_verticale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Identification (m)</div></th>
    <td><input id="identification" type="identification" /></td>
  </tr>
</table>

<script>
function aggiorna_distanze ()
{
  if (parseFloat(document.querySelector('#dimensione_verticale').value) >
      parseFloat(document.querySelector('#dimensione_orizzontale').value))
  {
    /* d_hor è critica */
    var d_crit = document.querySelector('#dimensione_orizzontale').value;
    var res = document.querySelector('#risoluzione_orizzontale').value;
    var angolo = document.querySelector('#angolo_tele_orizzontale').value;
  }
  else
  {
    /* d_ver è critica */
    var d_crit = document.querySelector('#dimensione_verticale').value;
    var res = document.querySelector('#risoluzione_verticale').value;
    var angolo = document.querySelector('#angolo_tele_verticale').value;
  }

  var d_detection = Math.round (((res / 2) * d_crit) / (Math.tan (angolo * Math.PI / 180.00) * 2));
  document.querySelector('#detection').value = d_detection;

  var d_orientation = Math.round (((res / 2) * d_crit) / (Math.tan (angolo * Math.PI / 180.00) * 3));
  document.querySelector('#orientation').value = d_orientation;

  var d_recognition = Math.round (((res / 2) * d_crit) / (Math.tan (angolo * Math.PI / 180.00) * 8));
  document.querySelector('#recognition').value = d_recognition;

  var d_identification = Math.round (((res / 2) * d_crit) / (Math.tan (angolo * Math.PI / 180.00) * 13));
  document.querySelector('#identification').value = d_identification;
}

function aggiorna_bersaglio (aggiorna_dati)
{
  if (document.querySelector('#bersaglio option:checked').value == 'uomo')
  {
    document.querySelector('#dimensione_verticale').value = 2;
    document.querySelector('#dimensione_orizzontale').value = 0.5;
  }
  else if (document.querySelector('#bersaglio option:checked').value == 'tir')
  {
    document.querySelector('#dimensione_verticale').value = 2.3;
    document.querySelector('#dimensione_orizzontale').value = 10;
  }
  else
  {
    document.querySelector('#dimensione_verticale').value = 1;
    document.querySelector('#dimensione_orizzontale').value = 1;
  }

  if (aggiorna_dati)
    aggiorna_distanze ();
}

function aggiorna_telecamera (aggiorna_dati)
{
  if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7520')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.3;
    document.querySelector('#angolo_tele_orizzontale').value = 2.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7500')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.3;
    document.querySelector('#angolo_tele_orizzontale').value = 2.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7300')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.85;
    document.querySelector('#angolo_tele_orizzontale').value = 3.3;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_ev7100')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 4.725;
    document.querySelector('#angolo_tele_orizzontale').value = 7.6;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'sony_fcb_se600')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 18;
    document.querySelector('#angolo_tele_orizzontale').value = 32;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46336013HSPNLX')
  {
    document.querySelector('#risoluzione_verticale').value = 256;
    document.querySelector('#risoluzione_orizzontale').value = 336;
    document.querySelector('#angolo_tele_verticale').value = 9.5;
    document.querySelector('#angolo_tele_orizzontale').value = 12.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640013HSPNLX')
  {
    document.querySelector('#risoluzione_verticale').value = 480;
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 18.5;
    document.querySelector('#angolo_tele_orizzontale').value = 22.5;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_46640060HSPNLX')
  {
    document.querySelector('#risoluzione_verticale').value = 480;
    document.querySelector('#risoluzione_orizzontale').value = 640;
    document.querySelector('#angolo_tele_verticale').value = 4.15;
    document.querySelector('#angolo_tele_orizzontale').value = 5.2;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton_50')
  {
    document.querySelector('#risoluzione_verticale').value = 80;
    document.querySelector('#risoluzione_orizzontale').value = 60;
    document.querySelector('#angolo_tele_verticale').value = 25.5;
    document.querySelector('#angolo_tele_orizzontale').value = 18.75;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton_25')
  {
    document.querySelector('#risoluzione_verticale').value = 80;
    document.querySelector('#risoluzione_orizzontale').value = 60;
    document.querySelector('#angolo_tele_verticale').value = 12.5;
    document.querySelector('#angolo_tele_orizzontale').value = 9.37;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'flir_lepton3_50')
  {
    document.querySelector('#risoluzione_verticale').value = 160;
    document.querySelector('#risoluzione_orizzontale').value = 120;
    document.querySelector('#angolo_tele_verticale').value = 28;
    document.querySelector('#angolo_tele_orizzontale').value = 21;
  }
  else if (document.querySelector('#telecamera option:checked').value == 'videotec_vcmhd30x01')
  {
    document.querySelector('#risoluzione_verticale').value = 1080;
    document.querySelector('#risoluzione_orizzontale').value = 1920;
    document.querySelector('#angolo_tele_verticale').value = 1.32;
    document.querySelector('#angolo_tele_orizzontale').value = 2.36;
  }

  if (aggiorna_dati)
      aggiorna_distanze ();
}

document.addEventListener('DOMContentLoaded', function() {
  var els = document.querySelector('select[name="bersaglio"] option[value="uomo"]');

  if(els)
  {
    els.selected = true;
  }

  aggiorna_bersaglio (false);

  els = document.querySelector('select[name="telecamera"] option[value="sony_fcb_ev7520"]');

  if(els)
  {
    els.selected = true;
  }

  aggiorna_telecamera (false);

  aggiorna_distanze ();

  document.getElementById('bersaglio').addEventListener('change', function() {
      aggiorna_bersaglio (true);
  });

  document.getElementById('telecamera').addEventListener('change', function() {
      aggiorna_telecamera (true);
  });
}, false);

</script>
</div>

## Storia e spiegazione del criterio di Johnson

Verso la fine degli anni 50, John Johnson ha proposto una metrica per caratterizzare la probabilità di riconoscere la presenza di un oggetto all'interno di un'immagine generata da un sistema di acquisizione video.

Johnson era un militare e lo scopo dei suoi studi consisteva nella ricerca di un modo per valutare le prestazioni di un sistema di visione per portare a termine dei compiti ben precisi. I compiti identificati dal criterio di Johnson sono:
* *Detection* - determinare se un oggetto è presente o meno sulla scena.
* *Orientation* - determinare l'orientamento di un oggetto, verticale o orizzontale, o valutare eventuali simmetrie.
* *Recognition* - riconoscere il tipo di oggetto, per esempio se il bersaglio è un uomo od un veicolo.
* *Identification* - discernere tra un uomo ed una donna o riconoscere il modello di un'auto.

Come è semplice da intuire, maggiore è il numero di pixel (o coppie di righe al tempo dei sistemi analogici) e maggiore è la probabilità di riuscire ad eseguire correttamente il compito desiderato.

Un concetto basilare nel criterio di Johnson è quello di *dimensione critica*. Tutti gli oggetti ripresi in una scienza da un sistema di visione possono essere inscritti in un rettangolo. In base al tipo di oggetto, il rettangolo potrà essere più alto che largo o viceversa. La dimensione critica è il valore minore tra altezza e larghezza. Per esempio, per un uomo la dimensione critica sarà la larghezza, mentre per un camion con rimorchio sarà l'altezza.

Il risultato degli studi di Johnson consiste nella determinazione del numero di pixel o coppie di righe che devono rappresentare la dimensione critica del bersaglio per eseguire uno dei quattro compiti precedenti con il 50% di probabilità di successo.

Compito        |Numero di pixel (sistemi digitali)|Coppie di righe (sistemi analogici)
---------------|----------------------------------|----------------------------------
Detection      | <center>2</center>               | <center>1</center>
Orientation    | <center>3</center>               | <center>1,4</center>
Recognition    | <center>8</center>               | <center>4</center>
Identification | <center>13</center>              | <center>6,4</center>

## Un po' di formule

Per fare un calcolo approssimato che non tenga conto della deformazioni dovute alla non idealità della lente possiamo ragionare nel modo seguente.

Supponiamo che la camera sia un pin-hole e consideriamo che il problema sia unidimensionale. Quest'ultima ipotesi non è limitativa perché il criterio di Johnson si basa sul concetto di dimensione critica, quindi riduce il problema ad una sola dimensione. Nell'immagine successiva la telecamera è centrata nel vertice del triangolo corrispondente all'angolo &alpha; . Poiché gli angoli delle telecamere vengono dati in gradi rispetto alla normale all'asse ottico, l'angolo a tutto tele &alpha; corrisponde a quello che viene inquadrato da metà sensore.

Per determinare la distanza del criterio di Johnson per effettuare uno dei compiti è necessario determinare a quale distanza un bersaglio verrà proiettato sul sensore col il numero di pixel necessari allo svolgimento del compito. Nell'immagine seguente si suppone per esempio che il sensore abbia risoluzione 30 pixel, per questo motivo i pixel marcati sul cateto L sono 15. Il bersaglio nell'immagine è l'ellisse rossa, che deve occupare due pixel per il compito di detection. Come ultima ipotesi supponiamo che la dimensione critica dell'ellisse sia 0,5m .

<div markdown="0">
<canvas id="triangolo" width="600" height="400" >
</canvas>
<script>
document.addEventListener('DOMContentLoaded', function() {
  var canvas = document.getElementById("triangolo");
  var context = canvas.getContext("2d");

  context.font = '20pt Arial';
  context.fillText('distanza d', 200, 380);
  context.fillText('\u03B1', 110, 340);

  context.save();
  context.translate(560, 100);
  context.rotate(Math.PI/2);
  context.textAlign = "center";
  context.fillText("Larghezza inquadrata L", 100, 0);
  context.restore();

  context.beginPath ();
  context.moveTo (50, 350);
  context.lineTo (550, 350);
  context.lineTo (550, 50);
  context.closePath ();
  context.stroke ();

  context.beginPath ();
  for (var i = 70 ; i < 350 ; i += 20)
  {
    context.moveTo (547, i);
    context.lineTo (553, i);
  }

  context.closePath ();
  context.stroke ();

  context.beginPath ();
  context.arc (50,350,50,11/6*Math.PI,2*Math.PI);
  context.stroke ();

  context.beginPath ();
  context.ellipse (550, 170, 9, 18, 0, 0, 2 * Math.PI);
  context.fillStyle = "red";
  context.fill ();
  context.stroke ();
});
</script>
</div>

Se l'ellisse larga mezzo metro deve essere ripresa in due pixel possiamo calcolare quanto lungo deve essere il cateto L con una semplice proporzione:

\\[ 2 : 0,5 = 15 : L \\]

che ha come risultato

\\[ L = \frac{15 . 0,5}{2} = \frac{15}{4} \\]

A questo punto la distanza d è la distanza tale per cui il lato L viene sotteso da &alpha;

\\[ d = \frac{15}{4} . \cot (\alpha)  = \frac{15}{4} . 24,89 = 93,3\\]

quindi la distanza di Johnson in questo esempio per effettuare la detection di un oggetto avente dimensione critica di 0,5m con un'ottica che ha angolo tele si 2,3° ed una risoluzione di 30 pixel è 93,3m .

Se invece dei valori numerici sostituiamo le variabili possiamo riscrivere le equazioni in questo modo:


\\[ p : d_{crit} = \frac{ res}{2} : L \\]

dove p è il numero di pixel necessari al compimento del compito. La proporzione può essere quindi scritta come


\\[ L = \frac{ d_{crit} . \frac{ res}{2} }{p} \\]

e quindi 

\\[ d = \frac{ d_{crit} . \frac{ res}{2} }{\tan(\alpha_{tele}) . p} \\]


Nel caso di dimensione critica orizzontale le distanze vanno determinate come:

\\[ d = \frac{ \frac{ res_{hor}}{2} . d_{crit}}{ \tan(\alpha_{tele, hor}) . p } \\]

Allo stesso modo, se la dimensione critica è verticale, la formula da usare è:

\\[ d = \frac{ \frac{ res_{ver}}{2} . d_ {crit}}{ \tan(\alpha_{tele, ver}) . p } \\]
