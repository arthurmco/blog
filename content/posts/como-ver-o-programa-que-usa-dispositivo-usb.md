---
title: "Como ver o programa que não deixa você tirar seu dispositivo USB"
date: "2022-09-26T17:41:23-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["tips", "simple"]
keywords: ["usb", "removal", "howto"]
archive: "2022@09"
description: |
    Eu já fechei todos os programas, mas o Windows não quer remover o meu pendrive!

    E agora, o que eu faço?
---

**AVISO: Esse é um post que eu fiz faz um tempo, pra um outro projeto de blog, que eu achei quando**
**eu tava fazendo backup.**\
**Como eu tava achei legal, trouxe pra cá**\
**A data original é 26/09/2019**

Ás vezes, você está no Windão e quer remover seu dispositivo USB, mas
aparece aquela mensagem dizendo que o dispositivo está sendo usado

Pra ver o programa que está usando o dispositivo, faça o seguinte

 - Botão direito no símbolo do Windows
 - Clique em "Visualizador de Eventos"
 - Na lista à esquerda, clique em Sistema

{{< figure src="/blog/images/event-log-device-removal.png" caption="O visualizador de eventos" >}}

 - Na lista à esquerda, vão aparecer alguns avisos. Clique no primeiro
   cuja "Fonte" for "Kernel-PnP". PnP significa "Plug and Play", o
   que, obviamente, seu dispositivo USB é.
 - Na aba "Geral", vai estar escrito algo como:

   ```
   O aplicativo 
   \Device\HarddiskVolume3\Program Files\Transmission\transmission-qt.exe 
   com a id de processo 6664 
   parou a remoção ou a ejeção para o dispositivo USB\VID_0BC2&PID_2322\MSFT30NA8H1DX3.
   ```

 No meu caso, o Transmission estava usando um arquivo que estava na
 minha HD externa. Quando eu o fechei, eu consegui remover! 
