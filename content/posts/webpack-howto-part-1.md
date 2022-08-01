+++
title = "Como usar o Webpack - Parte 1"
date = "2021-09-11T23:15:56-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = "images/webpack-01-cover.png"
tags = ["series", "projetos"]
keywords = ["webpack", "javascript", "webpack5"]
description = """
Depois de muito tempo com medo de usar o Webpack, já que eu não
entendia muito bem, finalmente estou usando em um projeto pessoal, e
queria mostrar aquilo que eu aprendi para vocês.

Nessa primeira parte, eu vou mostrar mais ou menos como ele funciona, 
como instalar e como rodar ele.

"""
showFullContent = false
archive = "2021@09"
+++

Depois de muito tempo com medo de usar o Webpack, já que eu não
entendia muito bem, finalmente estou usando em um projeto pessoal, e
queria mostrar aquilo que eu aprendi para vocês.

Isso vai ser o começo uma série sobre o Webpack nesse blog, onde eu
vou explicar o que eu estou aprendendo.

O meu intuito é chegar no ponto de "criar" um projeto Vue (que é o que
esse projeto vai usar) do zero. Eu poderia ter usado Vue direto, mas
queria testar algumas coisas antes usando Javascript puro, e o Webpack
me proporcionou isso.

Vou ser bem honesto: hoje em dia é mais viável usar o Webpack direto
caso você queira fazer algo usando Javascript puro, com Javascript,
HTML e CSS. Caso você queira usar React ou Vue, eles já vêm com
ferramentas que abstraem o Webpack. Você não precisa configurar nada.

Eu vou usar o Webpack 5. Eu não usei o 4 porque eu queria usar a
versão mais nova, mas eu posso ver o 4 caso vocês estejam interessados.

# O que é?

Antes de tudo, vamos falar sobre o que é o Webpack?

Copiando da documentação deles:

> At its core, webpack is a static module bundler for modern
> JavaScript applications. When webpack processes your application, it
> internally builds a dependency graph from one or more entry points
> and then combines every module your project needs into one or more
> bundles, which are static assets to serve your content from.

Traduzindo:

> Em sua essência, o webpack é um empacotador de módulo estático para
> aplicativos JavaScript modernos. Quando o webpack processa seu
> aplicativo, ele constrói internamente um gráfico de dependência a
> partir de um ou mais pontos de entrada e, em seguida, combina cada
> módulo de que seu projeto precisa em um ou mais pacotes, que são
> ativos estáticos para servir seu conteúdo.

Resumindo: ele pega um arquivo de entrada (que você pode especificar
qual), identifica todas as dependências dele e junta tudo em um
arquivo de saída. O arquivo de entrada é chamado de _entry_, e o de
saída é chamado de _bundle_. Você pode ter vários arquivos de saída em
um projeto.

Esses arquivos de saída são o que você inclui na sua página HTML

```html

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Minha página</title>
    </head>
    <body>
        <script src="saidadowebpack.js"></script>
        <!-- várias coisas maneiras e supimpas -->
    </body>
</html>


```

Isso é muito bom para fazer deploy de bibliotecas Javascript nesses
CDNs, já que você pode separar tudo em vários arquivos no ambiente de
dev e gerar um arquivo só. Antes do Node se tornar popular,
ferramentas assim provavelmente eram bem usadas.

O Webpack tambem, usando **loaders** e **plugins**, pode converter
arquivos de uma maneira que eles sejam usáveis dentro do
Javascript. Os **loaders** simplesmente convertem arquivos em módulos
Javascript, enquanto os **plugins** podem fazer mais coisas além
disso, coisas que envolvem o bundle inteiro.

Por exemplo, o _css-loader_, que é um loader que você pode usar para
importar CSS dentro do Javascript, só precisa saber dos arquivos de
entrada, então ele pode ser um loader mesmo. Já o
**HtmlWebpackPlugin**, um plugin que gera páginas HTML para a
aplicação que usa o Webpack, precisa saber o nome final do bundle,
coisa que um loader não vai te fornecer, então ele só pode ser
implementado como um plugin mesmo

## Criação

Inicialmente, você vai precisar instalar três pacotes: `webpack`,
`webpack-cli` e `webpack-dev-server`

O **webpack** é o webpack em si, o **webpack-cli** é o pacote que contém a
ferramentea de linha de comando do webpack (quando você digita
`webpack` no console, você está chamando uma ferramenta do CLI), e o
**webpack-dev-server** é um web server de desenvolvimento do webpack,
que suporta várias coisas interessantes, como recarregar a página
automaticamente quando uma mudança é feita.

Para instalar os três:

```

npm install webpack webpack-cli webpack-dev-server


```


A estrutura de pastas padrão esperada pelo webpack é a seguinte:

```

.
├── dist
└── src
    └── index.js

```
O arquivo `src/index.js` é o arquivo de entrada padrão. Isso significa
que o webpack vai converter esse arquivo e todos os arquivos que ele
importar (usando o `import ... from ...` do Javascript) em um arquivo
de saída, ou bundle. O local padrão do bundle é `dist/main.js`, e esse
arquivo é criado automaticamente.

Você, então, pode incluir esse dist/main.js em qualquer página
HTML.

Dos arquivos importados, o webpack só vai incluir no bundle aquilo que
foi realmente usado: funções, variáveis, etc. Ou seja, se um módulo
define a função `soma` e a função `subtração`, mas você só usa a
função `soma`, apenas essa função será incluída no bundle.

Por exemplo, digamos que eu tenha dois arquivos (além do packages.json
e a pasta node_modules, que eu tirei pra focarmos apenas nos arquivos
importantes
)

```
.
└── src
    ├── continhas.js
    └── index.js

```

com o continhas.js sendo:

```javascript

/** Soma dois valores **/
function soma(a, b) {
	return a + b;
}

/** Subtrai dois valores **/
function subtracao(a, b) {
	return a - b
}

export {
	soma,
	subtracao
}

```

e o index.js (nossa entrada) sendo:

```javascript


import {soma} from './continhas.js'

let a = 30;

console.log("Hello world ", soma(10, 20));
console.log("Hello world ", soma(a, 20));


```

Lembre-se: o webpack começa pelo index.js, vai pegando os arquivos
que o index.js importa e, de certa forma, os incluindo (na verdade,
como você vai ver, ele faz mais do que isso)

Rode o seguinte comando:

```

$ npx webpack

```

Ele vai rodar o webpack.  
Algumas mensagens de WARNING podem aparecer, mas elas só aparecem
porque uma propriedade não foi definida no arquivo de configuração, e
esse arquivo nós ainda não criamos.

O arquivo dist/main.js vai ficar assim (na verdade, esse resultado
pode mudar um pouco de versão pra versão do Webpack):

```
(()=>{"use strict";function o(o,l){return o+l}console.log("Hello world ",o(10,20)),console.log("Hello world ",o(30,20))})();
```

Perceba que os `console.log`s ainda estão aqui. Mas algumas coisas
aconteceram além disso. Por exemplo:

 - **Todos os comentários foram embora**.  
   O Webpack remove os comentários no bundle final, já que o que ele
   faz é semelhante a uma compilação. Esse bundle é como um arquivo
   compilado
   
 - **A função de subtração sumiu.**  
   Como eu disse antes, funções não usadas são removidas. Isso faz o
   arquivo final ser bem menor que os arquivos-fonte
   
   ```
   [tuts@lula2022 webpack-test]$ ls -l dist/*.js
   -rw-rw-r--. 1 tuts tuts 124 Sep 11 22:31 dist/main.js
   [tuts@lula2022 webpack-test]$ ls -l src/*
   -rw-rw-r--. 1 tuts tuts 169 Sep 11 22:06 src/continhas.js
   -rw-rw-r--. 1 tuts tuts 155 Sep 11 22:30 src/index.js

   ```
   
   169 + 155 = 324  
   61% de redução do tamanho total do arquivo. Esse percentual pode
   variar, mas o arquivo final sempre vai ser menor que os fontes, já
   que ele é como se fosse um arquivo compilado. Ele não é feito pra
   ser lido por pessoas, como você pôde ver quando listamos o conteúdo
   dele.
   
   
 - **A função de soma sumiu.**  
   Na verdade, a função soma ainda está aqui. O webpack só deu outro
   nome pra ela:
   ```
   function o(o,l){return o+l}
   ```
   
   ```
   function soma(o, l) {return o+l;}
   ```
   ```
   function soma(a, b) {return a + b;}
   ```
   ```javascript
   /** Soma dois valores **/
   function soma(a, b) {
       return a + b;
   }
   ```
   
   Viu?
   A função `o(o, l)`, logo depois do "use strict", é a nossa função
   soma. O webpack mudou o nome dela pro arquivo final ficar menor.  
   Esse processo de remoção de coisas (como comentários e mudança de
   nome de variáveis) pro arquivo ficar menor é chamado de
   **minificação**. O webpack faz isso por padrão em modo de produção.  
   Usar o modo de desenvolvimento ou o de produção pode ser especificado
   no arquivo de configuração, mas, caso ele não exista, o modo de produção
   é usado por padrão.


Caso você inclua esse arquivo main.js numa página HTML, você vai ver esses 
`console.log()`s printando Hello World

Por exemplo, crie um arquivo index.html na mesma pasta do main.js, e
coloque isso aqui nele:

```html

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Teste Webpack</title>
    </head>
    <body>
        <script src="main.js"></script>
    </body>
</html>


```

Caso você abra no navegador, vai aparecer isso:

![O console dessa página HTML aí em cima](/blog/images/webpack-01-console.png)


Existem algumas perguntas que ficaram sem ser respondidas:
 - Como eu uso essas funções fora do bundle? Acessando elas da minha
   página HTML?
 - Como eu mudo o caminho do bundle de saída?
 - Como eu uso esse `webpack-dev-server` aí?
 
Essas coisas vão ficar pra [parte 2]({{<ref "/posts/webpack-howto-part-2">}}), onde eu mostro como criar esse
arquivo webpack.config.js, o arquivo de configuração do webpack, e o
que vem nele.
