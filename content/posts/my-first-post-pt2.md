+++
title = "Primeiro post parte 2"
date = "2021-02-16T02:04:31-03:00"
author = "Arthur M"
authorTwitter = "usrbinarthur" #do not include @
cover = ""
tags = ["primeiro post", "hugo", "jekyllrb", "github-pages"]
keywords = ["talk"]
description = "Parte 2 do meu primeiro post. Aqui eu vou mostrar como se coloca esse blog no ar"
showFullContent = false
+++

Na [primeira parte]({{< ref "/posts/my-first-post" >}}), eu ensinei como iniciar o blog, e hoje
eu vou mostrar como postar ele no github pages.

## Como botar

Eu usei [esse guia](https://gohugo.io/hosting-and-deployment/hosting-on-github/) pra me ajudar, e 
funcionou. Mas, resumindo: 

 - você precisa criar um repositório.
   - Caso você queria que o seu site do Github Pages seja seu blog, de cara (no caso daqueles sites 
   `fulano.github.io`, é basicamente isso), você cria um repositório chamado `fulano.github.io`, 
   assumindo que `fulano` seja seu username do github.  

   - Caso você queira criar uma subpágina (eu fiz isso), você cria um repositório com o nome da
   sua subpágina (no meu caso, eu chamei de `blog`, por isso que a URL é essa. Você pode ver o 
   repositório [aqui](https://github.com/arthurmco/blog))

 - dê um `mkdir -p .github/workflows` dentro do seu repositório, e dentro dessa pasta `workflows`,
   cole [esse arquivo aqui](https://github.com/arthurmco/blog/blob/main/.github/workflows/gh_pages.yml).

   - Ele vai criar um job no Github Actions (uma espécie de CI do Github) que vai gerar o HTML do site
     a partir do Markdown e colocar esse HTML dentro do branch `gh-pages`. O branch `gh-pages` é de onde
     o Github vai tirar o HTML do seu site.  
     Essa action demora um pouco pra rodar, então demora alguns segundos até seu site ser gerado.

 - Vá até as configurações do repositório (a engrenagem escrito "Settings"), clique em Options,
   depois desça até "Github Pages". Mude o branch para `gh-pages`, assim como está na imagem:

   ![Como deve ficar teu setup](/blog/images/01-ghpages-image.png)

## Outras coisas 

Existe um jeito bem legal de fazer referências a outras páginas, que eu pesquisei só pra não ter
que ficar colocando a URL na mão.

Um exemplo: o código do começo do post ficou assim: 

```markdown

Na [primeira parte]({{&lt; ref "/posts/my-first-post" &gt;}}), eu ensinei...

```

Esse `<ref "/posts/my-first-post" >` cria um link pro arquivo `posts/my-first-post`, que é onde eu salvei o primeiro 
post.

Além disso, pra botar imagens, é só você colocar dentro do diretório `static`, lá na raiz do 
repositório. Por exemplo, a imagem que eu usei está na pasta `static/images/01-ghpages-image.png`,
pra usá-la eu escrevi assim: 

```markdown

   depois desça até "Github Pages". Mude o branch para `gh-pages`, assim como 
   está na imagem

![Como deve ficar teu setup](/blog/images/01-ghpages-image.png)

```

Infelizmente, eu tive que botar o `blog/`. Parece que, se seu blog não fica na raiz da url, você 
precisa botar o caminho. (Bom, pelo menos eu não preciso colocar a URL inteira.)

-----

Então é isso.

Enquanto isso, eu vou ver como eu adiciono comentários.

Até mais!
