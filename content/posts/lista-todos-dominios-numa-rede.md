---
title: "Listando todos os domínios numa rede"
date: "2022-09-26T17:35:44-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["hint"]
keywords: ["nslookup", "dominios", "rede interna"]
archive: "2022@09"
description: |
  Quer descobrir o IP de um servidor específico da sua rede, mas não tem o domínio?

  Aqui tem a solução
---

**AVISO: Esse é um post que eu fiz faz um tempo, pra um outro projeto de blog, que eu achei quando**
**eu tava fazendo backup.**\
**Como eu tava achei legal, trouxe pra cá**\
**A data original é 26/09/2019**


Às vezes você quer uma lista de todos os domínios numa rede.

Sei lá, talvez você queira o nome de um servidor específico no seu
trabalho, e não sabe onde ver isso. Ou você não tem acesso a essa
informação.

Eu precisei dessa informação um tempo atrás. Um pequeno shell script
resolveu o problema

```bash
  #!/bin/bash

  # Troca A, B e C pelos três primeiros octetos da sua rede
  # Por exemplo, 192.168.1

  # Só funciona com redes /24, mas pode funcionar para outras redes
  # se você adicionar mais loops.

  for i in {1..254}; do
    nslookup A.B.C.$i | tr '\n' ' '
    echo " "
  done

```

Vai mostrar os domínios, linha por linha, e os IPs que não têm domínio

Não é perfeito, mas é o que salvou a minha vida :)
