+++
title = "Primeiro post"
date = "2021-02-16T01:09:54-03:00"
author = "Arthur M"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["primeiro post", "hugo", "jekyllrb"]
keywords = ["talk"]
description = "Primeiro post, onde eu falo como eu fiz esse blog e do que eu vou falar"
showFullContent = false
+++

Depois de algum tempo prometendo, finalmente criei um blog!

Vou usar esse blog pra falar de várias coisas: coisas dos meus projetinhos pessoais, algo que eu 
achar que alguém queira saber, ou que eu acreditar que vai ser útil. Provavelmente vai ser de 
tecnologia.

Eu poderia criar uma página de teste, ou um "Olá mundo", mas vou falar sobre como eu consegui
fazer esse blog, pra ajudar quem quiser começar

## Como?

Ao invés de usar o medium, ou o dev.to, decidi usar uma plataforma que gera sites estáticos 
chamada [Hugo](https://gohugo.io/), feita em Go. 

Eu procurei algumas, e estava indeciso entre essa e o [Jekyll](https://jekyllrb.com/), mas o Jekyll
é feito em uma linguagem que eu usei muito pouco, e mal passei do tutorial; além disso eu iria
precisar instalar o Ruby.

Instalar o Hugo não foi difícil, basicamente é tão simples quanto instalar um programa qualquer.

Pra criar o site, você digita `hugo new site <nome>`, e ele vai criar um site dentro da pasta 
`<nome>`. Ele não cria um repositório git automaticamente, então você vai ter que fazer isso.

Existe um arquivo de configuração, que você vai ter que mexer pra colocar o que você quiser. O meu
(até o presente momento) é assim:

```toml
baseURL = "https://arthurmco.github.io/"
languageCode = "pt-br"
title = "Blog do Arthur M."
theme = "terminal"

[params]
  fullWidthTheme = false
  centerTheme = true
  themeColor = "green"
  enableGitInfo = true
```

(Maravilhoso que o syntax highlighting funciona)

Pra mudar o tema, você pode fazer duas coisas: mover os arquivos dentro da pasta `themes/<nomedotema>`, 
ou criar um submódulo git nessa pasta aí. Lembre-se de editar o `config.toml` e botar o tema lá (que
deve ser `<nomedotema>`).

Lembre-se que alguns temas possuem opções adicionais, algumas obrigatórias e outras não. Elas vão
dentro de `[params]`  
O que eu instalei (que eu achei bonito pra caramba, pra falar a verdade), possui essas duas opções
de cima obrigatórias, e as outras são opcionais

Pra criar um post, você digita ` hugo new posts/my-first-post.md`. Ele vai criar um arquivo 
`my-first-post.md` dentro de `posts/`, e é onde você vai criar sua postagem. Lembre-se de escolher
um nome que tenha a ver com o post.

O comando `hugo serve` te ajuda a visualizar como vai ficar teu site.

----

Eu ainda não coloquei o blog no Github Pages. Depois que eu colocar, eu vou fazer a Parte 2 desse
post.

**EDIT**: A parte 2 está [aqui]({{< ref "/posts/my-first-post-pt2" >}})

Até mais!