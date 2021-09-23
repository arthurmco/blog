+++
title = "Como usar o Webpack - Parte 2"
date = "2021-09-23T13:11:22-03:00"
author = "Arthur Mendes"
authorTwitter = "usrbinarthur" #do not include @
cover = "images/webpack-02-cover.png"
tags = ["series", "projetos"]
keywords = ["webpack", "javascript", "webpack5"]
description = """
Depois de muito tempo com medo de usar o Webpack, já que eu não
entendia muito bem, finalmente estou usando em um projeto pessoal, e
queria mostrar aquilo que eu aprendi para vocês.

Nessa segunda parte, eu vou mostrar como mudar o nome do arquivo de entrada,
criar uma página HTML automaticamente, e usar módulos loaders para você
poder importar qualquer arquivo no seu script
"""
showFullContent = false
+++

Depois de muito tempo com medo de usar o Webpack, já que eu não
entendia muito bem, finalmente estou usando em um projeto pessoal, e
queria mostrar aquilo que eu aprendi para vocês.

[Vimos na parte
1]({{<ref "/posts/webpack-howto-part-1">}}) como
o Webpack funciona, e que podemos usá-lo sem configuração. Porém, como
vimos, o uso sem configuração se torna bastante limitado, porque,
dentre muitas outras coisas, os nomes dos arquivos não podem ser
customizados,

Usando um arquivo de configuração, porém, você pode customizar várias
coisas, e usar vários **plugins** e **loaders** (eu expliquei o que são
esses dois na parte 1).

O nome do arquivo de configuração do webpack é o
`webpack.config.js`. Ele fica na raiz do seu projeto, no diretório
principal. Lembrando da estrutura de arquivos que tínhamos, na parte
1, incluindo esse arquivo, ela ficaria assim:

```
.
├── webpack.config.js
└── src
    ├── continhas.js
    └── index.js

```

De cara, eu vou introduzir algumas opções dele para você.

```javascript

const path = require('path');

module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
    }
}

```

Primeiro, dá pra perceber que ele é um arquivo javascript comum. A
configuração em si é só o que ele exporta.

A opção **mode** identifica o ambiente onde esse script vai rodar,
`development` pra desenvolvimento, `production` pra produção. Isso
você provavelmente deixaria em uma variável de ambiente, ou em outro
arquivo.

A opção **entry** identifica o arquivo de entrada. Lembrando o que eu
disse:

> O arquivo de entrada é chamado de _entry_, e o de saída é chamado de
> _bundle_.

Você pode definir mais de um arquivo de entrada assim:

```javascript

{
    ...
    entry: {
        arquivoum: './src/arquivo1.js',
        arquivodois: './src/arquivo2.js'
    }
}

```

Os nomes dos atributos não são muito importantes até onde eu sei, mas
os caminhos dos arquivos são onde esses arquivos estão.

A opção **output** define a saída, onde esses bundles serão
salvos. `output.path` define o caminho, e `output.filename` o nome do
arquivo.

Ele pode gerar um arquivo de output por cada arquivo de entrada,
usando a string de substituição `[name]`. Por exemplo:

```javascript

{
    ...
    entry: {
        arquivoum: './src/arquivo1.js',
        arquivodois: './src/arquivo2.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name]_out.js',
    }
}

```

geraria dois arquivos na pasta `dist`: `arquivoum_out.js` e
`arquivodois_out.js`

Usando esse arquivo super básico, você pode controlar os arquivos de
entrada e os de saída.

Mas existem mais coisas que você pode fazer:

# Módulos

Algumas bibliotecas lhe permitem importar um arquivo CSS dentro de um
arquivo Javascript, mais ou menos assim:

```javascript

import './style/style.css'
import './style/style.css'

```

Você pode fazer isso usando loaders, não só com CSS, mas com outros
tipos de arquivos também. Até com JSON, por mais engraçado que isso
possa parecer.

Usaremos dois loaders:

 - o **style-loader** serve como uma base para carregar estilos CSS
 - o **css-loader** serve para o style-loader reconhecer imports de
   CSS, geralmente usados para importar um arquivo CSS em outro.

Os dois são geralmente usados juntos.

Instale-os assim:

```
$ npm i --save-dev css-loader style-loader
```

E adicione essa linha abaixo de `output`, ainda dentro do `module.exports`

```json
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
            },
        ],
    },

```

Ele vai usar o css-loader e o style-loader em qualquer arquivo que
terminar com `.css`.

(aliás, quem quer um post sobre regex?)

# Plugins

Como eu disse antes, um plugin do webpack tem autoridade pra mexer nos
bundles. Um dos plugins mais usados é o `html-webpack-plugin`, e ele
serve para gerar uma página HTML que automaticamente inclui todos os
bundles, assim você não precisa criar nenhuma página

O HTML que ele gera é mais ou menos assim, nas configurações default:

```html
<html>
    <head>
        <meta charset="utf-8">
        <title>título</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <script defer="defer" src="bundle.js"></script>
    </head>
    <body>
    </body>
</html>
```

Sendo que o título você pode definir quando instancia o plugin.

Para instalar o `html-webpack-plugin`, você deve executar o seguinte
comando:

```
npm i --save-dev html-webpack-plugin
```

Dentro do webpack.config.js, insira essa linha, lá no começo

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

Isso vai incluir a classe responsável por instanciar o plugin.

Ao contrário dos loaders, os plugins devem ser requeridos.

Agora, insira essas linhas depois de `module`

```javascript
    plugins: [
        new HtmlWebpackPlugin({
            title: "título da sua página"
        })
    ]
```

O construtor dessa classe aceita mais atributos além de `title`:

 - **filename**: o path do arquivo para o qual você quer escrever a
   página. O padrão é index.html
 - **template**: um path para um template, cujo conteúdo HTML será
   copiado para o arquivo principal. [A documentação do
   plugin](https://github.com/jantimon/html-webpack-plugin#writing-your-own-templates)
   pode ajudar em como criar o template
 - **showErrors**: mostra erros (de compilação do bundle,
   provavelmente) na página HTML.

Para gerar mais de um arquivo HTML, você deve declarar esse plugin
mais de uma vez, e mudar o filename para outro arquivo, e o template
para outro arquivo também se necessário.

```javascript
    plugins: [
        new HtmlWebpackPlugin({
            title: "título da sua página"
        }),
        new HtmlWebpackPlugin({
            title: "outra página",
            filename: __dirname + "/outrapagina.html",
            template: "src/assets/outrotemplate.html"
        })
    ]
```

O seu webpack.config.js provavelmente vai ficar assim:

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
    },
    module: {
        rules: [
            {
                test: /\.css$/i,
                use: ['style-loader', 'css-loader'],
            },
        ],
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: "titulo da página",
            xhtml: true,
        })
    ],

};

```

<hr />

Então, por enquanto é isso. Já dá pra desenvolver alguma coisa

Na próxima parte falaremos de opções para desenvolvedores.

Além disso, falaremos sobre como usar o jest junto com o seu app,
porque você já deve ter percebido que, em 2021, o `import` não
é muito bem suportado no Node, só o `require`. O Webpack entende os
imports, mas o Node não. Mas os testes rodam no Node, então temos um
problema.  
Esse problema eu já resolvi nesse app que eu estou escrevendo, então
vou explicar como eu fiz isso pra vocês
