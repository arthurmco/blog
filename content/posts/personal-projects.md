+++
title = "Projetos pessoais"
date = "2021-02-18T03:57:10-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["projetos", "shissue", "familyline", "demangler-js", "annos"]
keywords = ["devlog"]
description = "Nesse post, eu conto um pouco sobre os programas que fiz, e que estou fazendo, fora de meu trabalho. Todos eles estão [no meu Github](https://github.com/arthurmco)"
showFullContent = false
+++

Um dos motivos pelos quais eu criei esse blog é registrar meus pensamentos e problemas que eu 
encontro enquanto desenvolvo, e como eu os solucionei.  

Alguns problemas eu encontro no trabalho e, como no trabalho eu uso tecnologias que mais pessoas
usam, os posts onde eu descrevo esses problemas podem ser mais úteis.  

Porém, os posts sobre meus projetos fora do trabalho são aqueles que serão mais interessantes 
para mim.  
E, como eu penso em fazer uma espécie de "devlog" pra eles, é bom eu falar quais eles
são, para que ninguém fique perdido quando eu começar a postar coisas sobre eles e, caso alguém 
fique, eu possa linkar a pessoa para esse post aqui.


# shissue

![Print do programa](https://camo.githubusercontent.com/742d63d3ef7d7449eb52614be002539757a32b15572ed7e18aad1340d4ee9059/68747470733a2f2f61736369696e656d612e6f72672f612f7144785764717a764f35564c6e426c704f54646e4e7a31496d2e706e67)
[Link](https://github.com/arthurmco/shissue)

O `shissue` é uma ferramenta que eu fiz que te permite ver os issues do seu repositório. Com ele, 
você conseguia ver o número do issue, se ele estava aberto ou fechado, o texto e os autores. Ele
funcionava para o Github e para o Gitlab. Até as labels ele mostrava

Eu estava pensando em permitir que você escrevesse issues por ele.

Ele foi o meu primeiro programa real em Go, e com ele eu aprendi a fazer requisições HTTP em Go.

No momento ele está abandonado, mas, se vocês acharem útil, eu posso dar uma refatorada nele (a
linguagem mudou muito desde 2018, que foi quando eu criei esse programa).


# annos

> ANNOS (Arthur's Not Named Operating System) is a OS, powered by a C++ monolythic kernel.

![Print dele rodando no QEMU](https://camo.githubusercontent.com/074f7f41d2f449a4b9965900e15bc4eb739be1c5a6167c82322b9cd6023670fe/68747470733a2f2f692e696d6775722e636f6d2f684141356148682e706e67)
[Link](https://github.com/arthurmco/annOS)

O annOS foi um kernel que eu fiz. Não faz muita coisa, mas me ensinou *bastante* sobre programação
low-level, sobre como um kernel real acessa o hardware, um pouco sobre como o protocolo PCI funciona,
como os sistemas operacionais de antigamente mostravam coisas na tela, como a MMU dos processadores
x86 funcionam...

No momento ele está parado. Eu o abandonei porque eu não encontrei um objetivo pra ele, e acabei 
desanimando.

Há um tempo atrás, eu comecei a fazer um kernel em Rust, mas ele ainda não está no Github. Nem nome
ele tem!


# Familyline

Praticamente o único projeto realmente ativo dessa lista

![Foto dele rodando](/blog/images/02-fayline.png)
[Link](https://github.com/arthurmco/familyline)

> It's made with a homebrew engine, for me to exclusively learn while bringing joy to its players. The major objective of this game is fun, to play and develop.

É um jogo de estratégia em tempo real, feito com uma engine própria, onde você acompanha a história
de uma família fictícia (daí o nome) desde o final do Neolítico até mais ou menos 30 d.C.

A história real ainda não está feita, nem a engine, mas já aprendi bastante. Tudo o que eu sei de 
C++, CMake e programação OpenGL foi graças a esse jogo.  
Eu também aprendi a usar o [Google Test](https://github.com/google/googletest) (uma suíte de teste
da Google) pra testar algumas partes da engine.

Foi o projeto que eu mais aprendi, e é o que provavelmente vai render mais conteúdo.

Recentemente, eu aprendi a usar o AddressSanitizer do GCC, pra resolver alguns problemas de
corrupção de memória.

No momento, eu estou adicionando um modo multiplayer, mais pra confirmar se a engine está rodando
no tempo certo, se cada "tick", ou seja, cada iteração da engine, está acontecendo em seu tempo,
e também pro jogo ficar um pouco mais interessante.

Ele tem uma GUI, feita usando C++ e uma biblioteca de desenho 2D chamada [Cairo](https://www.cairographics.org/).

No futuro, eu quero fazer algumas coisas:

- suporte a Vulkan

- permitir a você adicionar entidades (as construções e as unidades) usando [Scheme](https://groups.csail.mit.edu/mac/projects/scheme/).

- refatorar a GUI (o código está meio feio)

- melhorar o renderer: adicionar um suporte melhor a luzes, renderizar sombras, normal maps, até um 
  ambient occlusion.

- mais tipos de terreno, especialmente um com água.
    
- refatorar o pathfinder.


Ele está *bem* ativo, com commits pelo menos toda semana.

# demangler-js

[Link](https://github.com/arthurmco/demangler-js)

Um demangler de funções C++.

Quando você compila um programa em C++, você tem um problema: C++ tem overloading de tipos nos 
argumentos. Em C, duas funções com o mesmo nome não podem existir, mesmo se elas tiverem argumentos
diferentes. Em C++, elas podem. O _mangling_ é feito pelo compilador, então, para encodar essas 
informações de tipos (além de outras, como namespace) no nome da função, para que, na hora de
debugar, ou qualquer outra hora que você precise saber o nome dessa função, você consiga saber
os tipos dela.

O demangler simplesmente pega esse nome estranho e, a partir dele, obtém o nome real da função e
os tipos de seus argumentos.

Esse projeto está meio parado. Ele também só suporta nomes mangleados pelo GCC