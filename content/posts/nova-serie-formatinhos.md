---
title: "Nova série de posts: Formatinhos"
date: "2022-08-04T20:57:41-03:00"
author: "Arthur Mendes"
authorTwitter: "usrbinarthur"
showFullContent: false

tags: ["short"]
keywords: ["formatinhos", "serie"]
archive: "2022@08"
---

Nesse mês, eu comecei a pensar em alguns jeitos de movimentar esse blog um pouco mais.

Um desses jeitos foi criar uma espécie de "série" de postagens, várias postagens sobre um mesmo 
tema, que ensinassem algo interessante de aprender.

Depois de pensar bastante, cheguei numa ideia.

A série **Formatinhos**.

Uma vez por semana, eu vou trazer aqui um formato de arquivo diferente. Irei ensinar o que significa
cada coisa dentro do formato enquanto nós fazemos um *parser* para ele e o mostramos.\
Como ele vai ser mostrado depende do arquivo.

<!--more--> 

Alguns formatos que eu já estou pensando em falar aqui (sem uma ordem específica):
 - ZIP
 - TAR.GZ (são 2 formatos diferentes mas é um arquivo só)
 - ISO
 - BMP
 - WAV (PCM, A-Law, µ-Law, ADPCM)
 - o sistema de arquivos FAT (12, 16 e 32)

-----

Além disso, você também pode sugerir novos formatos, embora existam algumas regras:

 - **Nada que dependa apenas de libs**: sim, eu sei que eu posso usar alguma biblioteca pra abrir. Mas qual
   é a graça?\
   As unicas libs que serão permitidas serão bibliotecas de compressão (zlib, gzip...) ou criptografia
   (openssl, libressl...), porque:
   1. implementar essas duas coisas na mão é só pra quem tem muito tempo livre ou muita vontade.
   2. não é o foco do formato, o foco do formato são as estruturas de dados dentro dele.

 - **Se pelo menos uma pessoa usar, tá válido**: e essa pessoa inclui você!\
   Se você criou um formato e quer que ele apareça aqui, você pode sugerir.

 - **Tamanho não importa, mas tenha noção**: Se ficar muito grande, eu posso dividir em várias 
   partes, mas tenha noção de que eu quero pelo menos poder mostrar algo na parte 1, nem que só seja
   um "Hello World", um pixel ou um simples segundo de áudio.\
   Se precisar de dez mil páginas só pra mostrar um "OI" na tela, aí vai ser f#da.

 - **Complexidade não importa, mas tenha noção**: Formatos complexos que exigem um conhecimento 
   matemático decente, como JPEG e MP3, podem ser sugeridos, mas eles vão ficar num limbo até eu
   entender, porque a minha faculdade não me deu esses conhecimentos.\
   E eu aposto que várias pessoas tiveram uma faculdade ruim. Alguns *devs* nem fizeram faculdade
   ou têm uma base matemática decente. Isso significa que eu precisaria explicar tudo ou linkar
   para um site que explique certos conceitos como transformada de Fourier de uma forma facilitada.

E é isso.

O primeiro post sai na próxima sexta, à noite.