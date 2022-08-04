+++
title = "Criando um 'crash handler' Part 1"
date = "2021-04-26T21:32:29-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = "images/cover-crash-handler.png"
tags = ["gamedev", "c++", "crash-handler", "sysv", "abi"]
keywords = ["devlog", "talk"]
description = """Sabe aquelas telas que abarecem quando um programa ou jogo crasha e fecha? _BugSplat_, 
a do Firefox, do Chrome? Bem, descobri como elas foram feitas. Vou tentar fazer uma igual 
e mostrar pra vocês como eu fiz <br/> 

Nessa primeira parte eu vou mostrar como eu fiz isso no Linux
"""
showFullContent = false
archive = "2021@04"
+++

Sabe aquelas telas que abarecem quando um programa ou jogo crasha e fecha? _BugSplat_, 
a do Firefox, do Chrome? Bem, descobri como elas funcionam. Vou tentar fazer uma igual 
e mostrar pra vocês como eu fiz.

Uma coisa que eu descobri é que esse tipo de programa é extremamente dependente de arquitetura de 
processador e de sistema operacional; portanto nessa primeira parte eu vou mostrar o funcionamento,
e como fazer um desses _crash handlers_ no Linux

> Um aviso: Isso aqui só vai funcionar se o seu software não tiver nenhum tipo de runtime (ou seja, 
> Java e .NET...), porque iremos mexer diretamente com a pilha de execução do processador.

# Como um crash handler funciona

O crash handler é um programa que faz algo (a parte de _handle_) quando um programa _crasha_, daí o nome.

Geralmente esses programas pegam algumas informações importantes do processo (como a `call stack`, ou
pilha de chamadas, que é uma pilha onde a função no topo é a função mais recente, a função abaixo é a
 função que chamou a primeira, a terceira função é a que chamou a segunda...), e mandam para algum
 servidor para análise.

Um exemplo de call stack está abaixo. 
 ![Um exemplo de 'call stack' no GNU Debugger](/blog/images/03-call-stack-example.png)

Em azul estão os endereços de memória; no caso do primeiro da lista é o endereço de memória onde 
o processador estava (o famoso *program counter*, ou *instruction pointer*), no caso dos restantes
é o endereço de memória aonde o processador vai retornar quando ele sair da função acima. 

> Por exemplo, o endereço de memória do segundo item da lista é aonde o processador vai retornar quando 
> ele terminar de executar a primeira função, o endereço de memória do terceiro item é aonde o processador
> vai retornar quando terminar a segunda função, e assim por diante.

Em amarelo estão os nomes das funções, em verde os arquivos de código-fonte em que elas se encontram

Claro que a do GDB está mais bonita do que vamos gerar inicialmente, mas existe um jeito de você 
mostrar exatamente os nomes de cada função. Pretendo falar disso na parte 2.

Existem três jeitos de você carregar um "crash handler":
  1. como um serviço (ou daemon) rodando na máquina. O Google Chrome faz assim.  
     A Microsoft faz também (aquela janela que pede pra você enviar o relatório de erros pra 
     Microsoft é uma espécie de **crash handler**)
  2. como um launcher para o seu aplicativo ou jogo. O Minecraft faz desse jeito
  3. fazer o próprio aplicativo carregar o handler (no início, ou, caso seja possível, quando o 
     programa crashar). A maioria dos programas e jogos fazem assim.

O jeito que eu fiz foi o segundo jeito, como um launcher, porque eu achei mais fácil pra mim.  
Eu pretendo testar o terceiro jeito, porque eu acho que é o mais fácil para o usuário (ele não 
precisa carregar um programa externo pra carregar o programa que ele quer). 

O *crash handler* tem sua ação dividida em duas partes: a parte do software que crashou e a parte
do handler. O handler não faz nada até o programa monitorado crashar.

### No início

Alguns sistemas, dentre eles o Linux, lhe permitem executar funções em determinados eventos, chamados
de **sinais**. Essas funções são chamadas de *signal handlers* Para alterar essas funções, você 
deve usar uma função chamada [sigaction](https://www.man7.org/linux/man-pages/man2/sigaction.2.html).

> No Windows, o método é diferente. Veremos isso futuramente também.

Alguns desses sinais são executados quando o programa faz algo que não deveria. São eles os que 
queremos.

> A função que passarmos precisa ser uma função normal. Ela *não pode* estar dentro de nenhuma 
> instância, nem ser um `lambda` de C++, nem nada retornado pelo `std::bind`, por uma questão de 
> lifetimes, e porque você não deve confiar em nenhum trecho da memória que o seu aplicativo
> estava usando, afinal ele crashou!

O protótipo dessa função é:

```c

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);

```

 - signum é o número do sinal (embora geralmente você use um `#define` que já liga o nome ao número).  
   Uma lista de sinais está [disponível aqui](https://www.man7.org/linux/man-pages/man7/signal.7.html),
   na seção "Standard signals".
 - `sigaction` é uma estrutura que contém alguns valores, um deles é o ponteiro pra nossa função,
   que será executada durante o sinal que queremos
 - `act` e `oldact` são ponteiros para a estrutura acima: no primeiro você passa o valor que você
   quer setar, no segundo você passa um ponteiro de memória onde essa função vai escrever o valor antigo.


Você deve criar handlers para alguns sinais fatais, mas que podem ser capturados (`SIGSTOP` e 
`SIGKILL` não podem, afinal, geralmente, eles são causados pelo usuário).

Os que eu escolhi para esse tutorial foram `SIGSEGV` (o famoso Segmentation fault), `SIGABRT` 
(um "crash voluntário",  geralmente causado por exceções não tratadas em C++) e `SIGBUS` 
(um sinal causado para erros de acesso a memória que não são falhas de segmentação), porque eles 
são os mais comuns. 

Você pode (e deve) pegar todos os sinais fatais, eu só não vou pegar todos pro código ficar um 
pouco menor.

> Os sinais fatais são aqueles que, naquela lista de sinais, tem a **Action** igual a **Core**.
> Eles geralmente resultam no fim da execução do programa

Existem dois jeitos de iniciar:
 - o handler inicia o programa e pega o Process ID (ou ID do Processo)
 - o programa inicia, executa o handler e passa o próprio Process ID pra ele.

Nos dois casos, o handler precisa do process ID do programa que você quer monitorar.

Agora, o que o programa a ser crashado faz é, assim que ele é iniciado, configurar esses *handlers* 
para uma função que será executada na hora do crash.

O handler espera o processo terminar. [A função `waitpid`](https://linux.die.net/man/2/waitpid) lhe
permite fazer isso. Você passa pra ela o process ID e um ponteiro de memória pra guardar as informações
de retorno da função e ela só retorna quando o processo cujo PID você passou terminar.

## Na hora do crash

Na hora do crash, a função que passamos é executada

A assinatura da nossa função precisa ser essa daqui: 

```c

void handler(int signum, siginfo_t* siginfo, void* context) 
{
    /* ... */
}

```

 - `signum` é o número do sinal
 - `siginfo` é uma estrutura que contém informações sobre o sinal, como o que exatamente o causou.
   Por algum motivo que eu não sei, o valor do `instruction pointer` (ou o endereço da instrução
   que o processo estava quando o sinal aconteceu) não está aí, e isso me deixou um pouco puto.
 - `context` é o endereço pra uma estrutura onde os valores mais interessantes são dependentes
   da arquitetura

> Esse guia só funciona pra x86-64. Como eu tenho um Raspberry PI, eu prometo que vou fazer um
> post só com os específicos para ARM no Linux.


# TODO
 - falar do context, e de como pegar os valores dos registradores
 - falar como pega a call stack
 - falar como manda a call stack pro processo do crash handler