+++
title = "O terminal nativo do macOS tem uma limitação curiosa"
date = "2022-04-07T11:35:21-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["short"]
keywords = ["macos", "terminal", "ansi"]
description = "E suporta só 256 cores diferentes."
showFullContent = false
archive = "2022@04"
+++

O app de terminal nativo do macOS suporta só 256 cores diferentes.

Ele não suporta 24bits de cor.

Se você realmente necessita de 24 bits de cor num aplicativo não-gráfico, usa o iTerm2, 
que é o que a galera costuma usar.


[![Lado a lado: cores no terminal e no iTerm2](/blog/images/terminal-iterm2-compare.png)](/blog/images/terminal-iterm2-compare.png)

O terminal é a janela da esquerda, o iTerm2 é a da direita. Clica na imagem pra dar um zoom

As cores deveriam ser as mesmas, mas na janela do iterm2 elas estão mais bem definidas.

Acho engraçado que o terminal do Windows suporta 24 bits de cor, e esse suporte é bem recente.