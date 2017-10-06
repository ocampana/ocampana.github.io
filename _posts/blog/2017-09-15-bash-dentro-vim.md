---
layout: post
title: "Come avere la bash dentro Vim"
modified:
categories: blog
excerpt: per i fanatici di Vim come me
tags: linux vim bash
keywords: linux vim bash
---

Vi è mai capitato di dover copiare più di una schermata da una shell dentro vim? Ecco, smanettare con copia ed incolla è un po' frustrante

Per questo motivo ho cercato se è possibile eseguire una shell dentro vim, e mi sono imbatutto in [ConqueTerm](https://vim.sourceforge.io/scripts/script.php?script_id=2771) che sembra fare proprio quello che voglio. Basta scaricare il file [conqueterm_2.2.vmb](https://vim.sourceforge.io/scripts/download_script.php?src_id=16279) ed installarlo con Vim

```
:open conqueterm_2.2.vmb
:so %
```

I file `.vmb` sono file Vimball, ovvero dei file con i plugin per vim e gli script per automatizzarne l'installazione. Quindi basta aprirli ed eseguirli, come mostrato nel blocco precedente.

A questo punto riavviamo vim e proviamo il comando `:ConqueTermVSplit bash`. Voilla:

![conqueterm](/images/conqueterm.png)
