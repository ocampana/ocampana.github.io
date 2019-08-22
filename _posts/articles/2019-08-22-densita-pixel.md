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


## Calcolatore distanze

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
        <option value="flir_46640013HSPNLX">Flir Tau2 640x480 13mm</option>
        <option value="flir_46640060HSPNLX">Flir Tau2 640x480 60mm</option>
        <option value="flir_lepton_50">Flir Lepton 80x60 50mm</option>
        <option value="flir_lepton_25">Flir Lepton 80x60 25mm</option>
        <option value="flir_lepton3_50">Flir Lepton3 160x120 50mm</option>
      </select>
    </td>
    <th><div style="overflow: hidden; white-space: nowrap;">Orientation (m)</div></th>
    <td><input id="monitoring" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Risoluzione orizzontale</div></th>
    <td><input type="text" id="risoluzione_orizzontale" name="risoluzione_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Orientation (m)</div></th>
    <td><input id="observation" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele orizzontale</div></th>
    <td><input type="text" id="angolo_tele_orizzontale" name="angolo_tele_orizzontale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Recognition (m)</div></th>
    <td><input id="recognition" type="text" /></td>
  </tr>
  <tr>
    <th><div style="overflow: hidden; white-space: nowrap;">Angolo tele verticale</div></th>
    <td><input type="text" id="angolo_tele_verticale" name="angolo_tele_verticale" readonly /></td>
    <th><div style="overflow: hidden; white-space: nowrap;">Identification (m)</div></th>
    <td><input id="identification" type="text" /></td>
  </tr>
</table>

<script>
function aggiorna_distanze ()
{
  var res = document.querySelector('#risoluzione_orizzontale').value;
  var angolo = document.querySelector('#angolo_tele_orizzontale').value;

  var d_monitoring = Math.round ( (res / (2 * 12) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#monitoring').value = d_detection;

  var d_detection = Math.round ( (res / (2 * 25) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#detection').value = d_detection;

  var d_observation = Math.round ( (res / (2 * 62) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#observation').value = d_observation;

  var d_recognition = Math.round ( (res / (2 * 125) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#recognition').value = d_recognition;

  var d_identification = Math.round ( (res / (2 * 250) ) / (Math.tan (angolo * Math.PI / 180.00) ));
  document.querySelector('#identification').value = d_identification;
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

